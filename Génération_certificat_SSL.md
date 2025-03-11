# Génération de certificat SSL

## VM Ubuntu

```bash
ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519 -N ""
```
Permet de générer une paire de clés

## VM Rocky

```bash
sudo adduser info0603
```
Permet de créer un nouvel utilisateur

```bash
sudo passwd info0603
```
Permet de créer un mot de passe pour l'utilisateur `info0603`
Mot de passe : `Info$603$25`

```bash
su - info0603
```
Permet de se connecter à un autre utilisateur

```bash
ssh-keygen -t rsa -f ~/.ssh/id_rsa -N ""
```
Permet de créer une paire de clés

```bash
exit
```

## Depuis la VM Ubuntu

```bash
ssh-copy-id -i ~/.ssh/id_ed25519.pub info0603@10.11.29.26
```
(A remplacer par l'IP de la machine si ce n'est pas celle-ci)
Permet de copier la clé publique à destination de `info0603` sur la VM Rocky pour permettre l'accès sans mot de passe

## Tester la connexion sans mot de passe

```bash
ssh info0603@<IP_MACHINE>
```
Si cela connecte directement sans demander de mot de passe alors c'est réussi

Faire pareil avec la VM Rocky

## Générer un certificat auto-signé

```bash
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
-keyout /etc/ssl/private/apache-selfsigned.key \
-out /etc/ssl/certs/apache-selfsigned.crt \
-subj "/CN=ubuntu-zabbix/O=Info0603/C=fr"
```

## Créer un fichier de configuration SSL

```bash
sudo nano /etc/apache2/sites-available/zabbix-ssl.conf
```

Dedans mettre :

```apache
<VirtualHost *:443>
    ServerName ubuntu-zabbix
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/apache-selfsigned.crt
    SSLCertificateKeyFile /etc/ssl/private/apache-selfsigned.key
    # ... (autres directives pour Zabbix)
</VirtualHost>
```

## Activer le site et les modules

```bash
sudo a2enmod ssl
sudo a2ensite zabbix-ssl.conf
sudo systemctl restart apache2
```

## Accès en HTTPS à Zabbix

```plaintext
https://<@IP_MACHINE>/zabbix
```
On arrive sur la page de connexion sécurisée, demandez à continuer vers l'adresse pour accéder à Zabbix en HTTPS
