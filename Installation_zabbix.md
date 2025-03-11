# Installation de Zabbix

## 1. Ajout du dépôt Zabbix

```bash
wget https://repo.zabbix.com/zabbix/6.4/ubuntu/pool/main/z/zabbix-release/zabbix-release_6.4-1+ubuntu24.04_all.deb
sudo dpkg -i zabbix-release_6.4-1+ubuntu24.04_all.deb
sudo apt update
```

## 2. Installation des composants

```bash
sudo apt install zabbix-server-mysql zabbix-frontend-php zabbix-apache-conf zabbix-sql-scripts zabbix-agent mysql-server -y
```

## 3. Configuration de la BDD pour Zabbix

```sql
sudo mysql -u root -p

# Exécuter dans MySQL :
CREATE DATABASE <NOM_BD> CHARACTER SET utf8 COLLATE utf8_bin;
CREATE USER <NOM_USER>@<IP_OU_LOCALHOST> IDENTIFIED BY <MOT_DE_PASSE>;
GRANT ALL PRIVILEGES ON <NOM_BD>.* TO <NOM_USER>@<IP_OU_LOCALHOST>;
FLUSH PRIVILEGES;
EXIT;
```

## 4. Importation du schéma initial

```bash
sudo zcat /usr/share/zabbix-sql-scripts/mysql/server.sql.gz | mysql --default-character-set=utf8mb4 -u zabbix -p Zabbix
```

## 5. Configuration du serveur Zabbix

Éditer le fichier `/etc/zabbix/zabbix_server.conf` :

```ini
DBHost=<HOST_NAME>
DBName=<DB_NAME>
DBUser=<USER>
DBPassword=<PASSWORD>
```

## 6. Redémarrage des services

```bash
sudo systemctl restart zabbix-server zabbix-agent2 apache2
sudo systemctl enable zabbix-server zabbix-agent2 apache2
```

## 7. Installation des agents2 sur CentOS et Rocky

```bash
sudo rpm -Uvh https://repo.zabbix.com/zabbix/6.4/rhel/9/x86_64/zabbix-release-6.4-1.el9.noarch.rpm
sudo dnf install zabbix-agent2 -y
```

### Configuration de l'agent

Éditer le fichier `/etc/zabbix/zabbix_agent2.conf` :

```ini
Server=<IP_UBUNTU_ZABBIX>
ServerActive=<IP_UBUNTU_ZABBIX>
Hostname=<NOM_UNIQUE_DE_LA_VM>
```

## 8. Importation du modèle communautaire

```bash
wget https://git.zabbix.com/projects/ZBX/repos/zabbix/raw/templates/os/Linux_agent2.xml
```

## 9. Gestion des agents

Vérifier le statut :
```bash
sudo systemctl status zabbix-agent2
```

Si non activé :
```bash
sudo systemctl enable --now zabbix-agent2
```

## 10. Installation SNMPv2

```bash
sudo dnf install net-snmp net-snmp-utils -y
```

### Configuration SNMPv2

Éditer le fichier `/etc/snmp/snmpd.conf` :

```ini
rocommunity public 10.11.29.29  # Autoriser le serveur Zabbix
```

Démarrer et activer le service SNMP :
```bash
sudo systemctl start snmpd
sudo systemctl enable snmpd
```