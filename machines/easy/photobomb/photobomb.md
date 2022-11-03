# Photobomb - Easy

nmap → nginx découvert = site web

tentative d’accès par le navigateur → donne l’url qu’on rajoute à /etc/hosts

une fois ajouté on accède au site, on fait un dirb pour trouver les fichiers du site → rien de trouvé

un petit nikto pour trouver vuln → rien

analyse statique du site → trouvé les creds dans le javascript

arrivée sur une page pour dl des images → analyse des requetes avec burp

on peut inserer du code → [https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology and Resources/Reverse Shell Cheatsheet.md](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md)

transformation en URL pour que ça passe: [https://meyerweb.com/eric/tools/dencoder/](https://meyerweb.com/eric/tools/dencoder/)

[https://www.revshells.com/](https://www.revshells.com/) pour le listener

On trouver le premier flag facilement puis on fouille les fichiers

On check les droits sudo → droit sudo sur cleanup.sh

find est pas appelé en absolu donc on change l’env et on fait une commande find custom pour afficher le flag système