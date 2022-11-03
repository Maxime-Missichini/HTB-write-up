# Netmon - Easy

```java
Starting Nmap 7.92 ( https://nmap.org ) at 2022-10-31 13:53 EDT
Nmap scan report for 10.10.10.152
Host is up (0.15s latency).
Not shown: 997 closed tcp ports (conn-refused)
PORT    STATE SERVICE      VERSION
21/tcp  open  ftp          Microsoft ftpd
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| 02-03-19  12:18AM                 1024 .rnd
| 02-25-19  10:15PM       <DIR>          inetpub
| 07-16-16  09:18AM       <DIR>          PerfLogs
| 02-25-19  10:56PM       <DIR>          Program Files
| 02-03-19  12:28AM       <DIR>          Program Files (x86)
| 02-03-19  08:08AM       <DIR>          Users
|_02-25-19  11:49PM       <DIR>          Windows
| ftp-syst: 
|_  SYST: Windows_NT
135/tcp open  msrpc        Microsoft Windows RPC
445/tcp open  microsoft-ds Microsoft Windows Server 2008 R2 - 2012 microsoft-ds
Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled but not required
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-time: 
|   date: 2022-10-31T17:54:42
|_  start_date: 2022-10-31T17:53:22

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 55.28 seconds
```

le login anonyme est permis donc on peut récupérer le premier flag super facilement

pas possible d’upload sur le ftp

site web de dispo ?

eternalblue ne passe pas malgré SMB

tentative d’injection dans l’url mais rien

découverte des fichiers de configs disponible sur le FTP, recherche dans ceux-ci

<dbpassword>
<!-- User: prtgadmin -->
PrTg@dmin2018
</dbpassword>

en regardant les backups de config sur reddit

exploration de l’interface → rien de concret

utilisation d’un payload metasploit authentifié avec le compte admin (quelle aubaine)

autorité système, quelle aubaine !