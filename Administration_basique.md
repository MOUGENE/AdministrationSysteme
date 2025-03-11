# Administration Basique

## Ubuntu

### Mise à jour et installation d'Apache2
```bash
sudo apt update
sudo apt install apache2
```

### Vérifier le fonctionnement du serveur web
```bash
sudo systemctl status apache2
```

Obtenir les adresses IP :
```bash
hostname -I
```
Accès au serveur web :
```
http://your_server_ip
```

### Gestion d'Apache
```bash
sudo systemctl stop apache2    # Stopper le service
sudo systemctl start apache2   # Démarrer le service
sudo systemctl restart apache2 # Redémarrer le service
sudo systemctl reload apache2  # Recharger la configuration sans interrompre les connexions
sudo systemctl disable apache2 # Désactiver le lancement automatique au démarrage
sudo systemctl enable apache2  # Réactiver le lancement automatique
```

### Installation de MySQL ou MariaDB
```bash
sudo apt install mysql-server     # Installer MySQL (version 8.0 par défaut)
sudo apt install mariadb-server   # Installer MariaDB (comme demandé dans le TP)
```

### Installation de PHP compatible avec MySQL et Apache2
```bash
sudo apt install php libapache2-mod-php php-mysql
```

### Connexion à MariaDB en tant que root
```bash
sudo mysql -u root -p
```

### Création d'un utilisateur et d'une base de données
```sql
CREATE USER "nom_utilisateur"@"IP_ou_localhost" IDENTIFIED BY "mot_de_passe";
CREATE DATABASE "nom_de_la_base";
GRANT ALL PRIVILEGES ON "nom_de_la_base".* TO 'nom_utilisateur'@'localhost';
FLUSH PRIVILEGES;
```

### Installation de phpMyAdmin
```bash
sudo apt install phpmyadmin
sudo ln -s /etc/phpmyadmin/apache.conf /etc/apache2/conf-available/phpmyadmin.conf
sudo a2enconf phpMyAdmin
sudo systemctl reload apache2
```

### Accès à phpMyAdmin
```
http://votre_adresse_ip/phpmyadmin
```

### Vérification des versions installées
```bash
apache2 -v
mysql --version
php -v
```

---

## Rocky Linux

### Mise à jour et installation d'Apache
```bash
sudo dnf update
sudo dnf install httpd -y
sudo dnf install less -y   # -y permet de répondre "oui" automatiquement
```

### Gestion d'Apache
```bash
sudo systemctl enable httpd  # Activer Apache au démarrage
sudo systemctl start httpd   # Démarrer Apache
sudo systemctl status httpd  # Vérifier l'état du service
sudo systemctl stop httpd    # Arrêter Apache
sudo systemctl restart httpd # Redémarrer Apache
```

### Configuration d'Apache (ajouter à `000-default.conf`)
```apache
<IfModule mod_userdir.c>
    UserDir public_html
    UserDir disabled root
    <Directory /home/*/public_html>
        AllowOverride FileInfo AuthConfig Limit Indexes
        Options Multiviews Indexes SymLinksIfOwnerMatch IncludesNoExec
        php_admin_flag engine on
    </Directory>
</IfModule>

<IfModule mod_php.c>
    php_flag display_errors Off
    php_flag log_errors On
    php_value error_log /var/log/php_errors/log
</IfModule>
```

### Installation d'un paquet personnalisé
```bash
sudo apt install ./*.deb
```

### Création d'un paquet Python
```bash
mkdir -p mypythonpackage/DEBIAN
mkdir -p mypythonpackage/usr/local/bin

cd mypythonpackage/usr/local/bin
nano request.py
```

Ajouter le contenu suivant dans `request.py` :
```python
#!/usr/bin/env python3
import requests
response = requests.get('https://api.github.com')
print(response.json())
```

Sauvegarder (`CTRL + X`, `Y`, `Enter`).

Rendre le script exécutable :
```bash
chmod 755 request.py
```

Créer le fichier `control` :
```bash
cd mypythonpackage/DEBIAN
nano control
```

Ajouter le contenu suivant dans `control` :
```
Package: mypythonpackage
Version: 1.0
Section: base
Priority: optional
Architecture: all
Maintainer: Votre Nom <votre.email@example.com>
Depends: python3, python3-requests
Description: Un paquet avec un script Python et des dépendances
```

Définir les permissions :
```bash
chmod 644 control
```

Se placer à la racine :
```bash
cd ~/
```

Construire le paquet :
```bash
dpkg-deb --build mypythonpackage
```

Installer le paquet :
```bash
sudo dpkg -i mypythonpackage.deb
```

Tester le script :
```bash
request.py
```

