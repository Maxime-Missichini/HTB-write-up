# Jerry - Easy

Serveur apache qui tourne donc nikto + dirb, pas encore regardé les vulns, on tombe sur la page d’acceuil tomcat

```java
---- Scanning URL: http://10.10.10.95:8080/ ----
+ http://10.10.10.95:8080/com1 (CODE:200|SIZE:0)                               
+ http://10.10.10.95:8080/com2 (CODE:200|SIZE:0)                               
+ http://10.10.10.95:8080/com3 (CODE:200|SIZE:0)                               
+ http://10.10.10.95:8080/con (CODE:200|SIZE:0)                                
+ http://10.10.10.95:8080/docs (CODE:302|SIZE:0)                               
+ http://10.10.10.95:8080/examples (CODE:302|SIZE:0)                           
+ http://10.10.10.95:8080/favicon.ico (CODE:200|SIZE:21630)                    
+ http://10.10.10.95:8080/host-manager (CODE:302|SIZE:0)                       
+ http://10.10.10.95:8080/lpt1 (CODE:200|SIZE:0)                               
+ http://10.10.10.95:8080/lpt2 (CODE:200|SIZE:0)                               
+ http://10.10.10.95:8080/manager (CODE:302|SIZE:0)                            
+ http://10.10.10.95:8080/nul (CODE:200|SIZE:0)                                
+ http://10.10.10.95:8080/prn (CODE:200|SIZE:0)                                
                                                                               
-----------------
END_TIME: Sun Oct 30 18:03:28 2022
DOWNLOADED: 4612 - FOUND: 13

+ Server: Apache-Coyote/1.1
+ The anti-clickjacking X-Frame-Options header is not present.
+ No CGI Directories found (use '-C all' to force check all possible dirs)
+ Server leaks inodes via ETags, header found with file /favicon.ico, fields: 0xW/21630 0x1525691762000 
+ OSVDB-39272: favicon.ico file identifies this server as: Apache Tomcat
+ Allowed HTTP Methods: GET, HEAD, POST, PUT, DELETE, OPTIONS 
+ OSVDB-397: HTTP method ('Allow' Header): 'PUT' method could allow clients to save files on the web server.
+ OSVDB-5646: HTTP method ('Allow' Header): 'DELETE' may allow clients to remove files on the web server.
+ DEBUG HTTP verb may show server debugging information. See http://msdn.microsoft.com/en-us/library/e8z01xdh%28VS.80%29.aspx for details.
+ /examples/servlets/index.html: Apache Tomcat default JSP pages present.
+ Cookie JSESSIONID created without the httponly flag
+ OSVDB-3720: /examples/jsp/snp/snoop.jsp: Displays information about page retrievals, including other users.

+ /manager/html: Default Tomcat Manager interface found
+ 6544 items checked: 0 error(s) and 10 item(s) reported on remote host
+ End Time:           2022-10-30 18:05:57 (GMT-4) (342 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested
```

![Untitled](Jerry%20-%20Easy%20a1950556703442069a63ae25ab190f62/Untitled.png)

scanner/http/tomcat_mgr_login pour trouver le mdp par défaut tomcat:s3cret

utilisation de metasploit authentifié pour avoir un reverse shell

en fait on est admin direct donc on a les deux flags !