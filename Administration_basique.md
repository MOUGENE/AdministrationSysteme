# Ubuntu : 

sudo apt update

sudo apt install apache2

Pour vérifier le fonctionnement du serveur web :

sudo systemclt status apache2

hostname -I -> obtenir des addresses

http://your_server_ip

Gestion apache :

sudo systemctl stop apache2 -> pour stopper le service

sudo systemctl start apache2 -> pour démarrer le service

sudo systemctl restart apache2 -> pour redémarrer le service

sudo systemctl reload apache2 -> Si vous procédez uniquement à des modifications de configuration, il se peut qu’Apache recharge souvent sans interrompre les connexions. 

sudo systemctl disable apache2 -> Par défaut, Apache est configuré pour un lancement automatique au démarrage du serveur. Si ce n’est pas ce que vous souhaitez désactivez ce comportement

sudo systemctl enable apache2 -> Pour réactiver le service de lancement automatique au démarrage

sudo apt install mysql-server -> pour installer MySQL (normalement ce sera la version 8.0)

				OU
sudo apt install mariadb-server -> pour installer MariaDB comme demander dans le TP

sudo apt install php libapache2-mod-php php-mysql -> pour installer un php compatible MySQL et Apache2 (normalement version 7.4)

Se connecter a MariaDB en tant que root : 

sudo MySQL -u root -p

Créer un nouvel utilisateur et une BDD : 

CREATE USER "nom_d'utilisateur"@"IP_ou_localhost" IDENTIFIED BY "mettre_un_mdp";
CREATE DATABASE "nom_de_la_base";
GRANT ALL PRIVILEGES ON "nom_de_la_base".* TO 'username'@'localhost';
FLUSH PRIVILEGES;

Installer phpmyadmin :

sudo apt install phpmyadmin
sudo ln -s /etc/phpmyAdmin/apache.conf /etc/apache2/conf-available/phpmyadmin.conf
sudo a2enconf phpMyAdmin
sudo systemctl reload apache2

Accéder a phpmyadmin : 

http://votre_adresse_ip/phpmyadmin

Vérifier les versions avec : 

apache2 -v
mysql --version
php -v


Rocky :

sudo dnf update

sudo dnf install httpd -y

sudo dnf install less -y

A savoir l'option -y c'est pour répondre a oui a toute les questions qui sont posés durant l'installation

Gérer le serveur Apache :

sudo systemctl enable httpd -> Enable Apache to start at system boot automatically.

sudo systemctl start httpd -> Start the Apache webserver.

sudo systemctl status httpd -> Check the Apache webserver status and verify it's running.

sudo systemctl stop httpd -> Stop the Apache webserver.

sudo systemctl restart httpd -> Restart the Apache webserver.

sudo nano 000-default.conf ce qu'il faut mettre a la fin du fichier :

<IfModule mod_userdir.c>
        # Activer UserDir
        UserDir public_html
        UserDir disabled root

        # Autoriser l'exécution de PHP dans UserDir
        <Directory /home/*/public_html>
                AllowOverride FileInfo AuthConfig Limit Indexes
                Options Multiviews Indexes SymLinksIfOwnerMatch IncludesNoExec
                # Require method GET POST OPTIONS
                php_admin_flag engine on
        </Directory>
</IfModule>

<IfModule mod_php.c>
        # Masquer les erreurs PHP
        php_flag display_errors Off
        php_flag log_errors On
        php_value error_log /var/log/php_errors/log
</IfModule>

Pour installer le paquet perso faire la commande suivante : 
sudo apt install ./*.deb

Pour le paquet Python : 

mkdir -p mypythonpackage/DEBIAN
mkdir -p mypythonpackage/usr/local/bin

cd /myphytonpackage/usr/local/bin

nano request.py

Mettre ceci dans le fichier : 

#!/usr/bin/env python3
import requests
response = requests.get('https://api.github.com')
print(response.json())

CRTL + X save yes

Rendre le script executable : 

chmod 755 request.py

cd /mypythonpackake/DEBIAN

nano control

Mettre les droits du fichier control grâce a : 
chmod 644 control

Mettre ceci dans le fichier : 

Package: mypythonpackage
Version: 1.0
Section: base
Priority: optional
Architecture: all
Maintainer: Votre Nom <votre.email@example.com>
Depends: python3, python3-requests
Description: Un paquet avec un script Python et des dépendances

CRTL + X save yes

Se mettre a la racine donc : 

cd ~/

Construire le paquet : 

dpkg-deb --build mypythonpackage

Installer le paquet :

sudo dpkg -i mypythonpackage.deb

Tester le script :

request.py





