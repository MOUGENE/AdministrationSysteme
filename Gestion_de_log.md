# Gestion des Logs avec Rsyslog

## Installation de Rsyslog

### Ubuntu
```bash
sudo apt install rsyslog -y
```

### CentOS / Rocky Linux
```bash
sudo dnf install rsyslog -y
```

## Manipulation de Rsyslog
```bash
sudo systemctl start rsyslog
sudo systemctl enable rsyslog
sudo systemctl status rsyslog
sudo systemctl restart rsyslog
```

## Changer le hostname sur CentOS et Rocky Linux
```bash
sudo hostnamectl set-hostname client-centos
sudo hostnamectl set-hostname client-rocky
```

Mettre à jour le fichier `hostname` :
```bash
echo "client-centos" | sudo tee /etc/hostname
echo "client-rocky"  | sudo tee /etc/hostname
```

Mettre à jour le fichier `hosts` :
```bash
sudo nano /etc/hosts
```
Modifier la ligne contenant `127.0.0.1` ou `127.0.1.1` avec le nouveau hostname :
```
client-centos
client-rocky
```

Appliquer les changements sans redémarrage :
```bash
sudo systemctl restart systemd-hostnamed
```

Vérifier la mise à jour du hostname :
```bash
hostnamectl
```

## Configuration de Rsyslog pour envoyer les logs vers un serveur Ubuntu

Éditer la configuration Rsyslog sur les clients (CentOS/Rocky) :
```bash
sudo nano /etc/rsyslog.conf
```
Ajouter à la fin du fichier :
```bash
# Envoi des logs au serveur Ubuntu via TCP
action(type="omfwd"
    queue.filename="forwardToUbuntu"  # Nom unique pour la queue
    queue.maxdiskspace="1g"           # Limite d'espace disque
    queue.saveonshutdown="on"         # Sauvegarde des logs en cas d'arrêt
    queue.type="LinkedList"           # Utilisation d'une queue en mémoire
    action.resumeRetryCount="-1"      # Tentatives de reconnexion illimitées
    Target="<IP_DU_SERVEUR_UBUNTU>"   # Adresse du serveur Ubuntu
    Port="514"                        # Port TCP 514
    Protocol="tcp")                   # Protocole TCP
```

Redémarrer Rsyslog :
```bash
sudo systemctl restart rsyslog
```

## Configuration de Rsyslog sur le serveur Ubuntu

Éditer la configuration :
```bash
sudo nano /etc/rsyslog.conf
```
Ajouter ou décommenter les lignes suivantes :
```bash
module(load="imudp")  # Activer le module UDP
module(load="imtcp")  # Activer le module TCP
input(type="imtcp" port="514")  # Écouter sur le port 514
input(type="imudp" port="514")
```

### Stockage des logs dans `/var/log/remote`
```bash
sudo nano /etc/rsyslog.conf
```
Ajouter à la fin du fichier :
```bash
# Stockage des logs
$template RemoteLogs, "/var/log/remote/%HOSTNAME%.log"
*.* ?RemoteLogs

# Règle pour CentOS (severity info et supérieure)
if $fromhost startswith 'client-centos' and $syslogseverity <= 6 then /var/log/remote/apache-centos.log
& stop

# Règle pour Rocky Linux (severity notice et supérieure)
if $fromhost startswith 'client-rocky' and $syslogseverity <= 5 then /var/log/remote/apache-rocky.log
& stop

# Stockage des logs d'authentification par hostname
$template AuthLogs, "/var/log/remote/%HOSTNAME%/auth.log"
auth.*,authpriv.* ?AuthLogs
```

Créer le répertoire de stockage des logs :
```bash
sudo mkdir -p /var/log/remote
sudo chown syslog:syslog /var/log/remote
```

Redémarrer Rsyslog :
```bash
sudo systemctl restart rsyslog
```

Vérifier que le serveur écoute sur le port 514 :
```bash
sudo ss -tuln | grep 514
```

## Vérification des logs sur les clients CentOS/Rocky

Envoyer les logs vers le serveur Ubuntu :
```bash
sudo tcpdump -i eth0 port 514
```

Vérifier la présence des fichiers de logs sur le serveur Ubuntu :
```bash
ls /var/log/remote/
```

Afficher le contenu des logs en temps réel :
```bash
sudo tail -f /var/log/remote/client-rocky.log
sudo tail -f /var/log/remote/client-centos.log
```

## Configuration d'Apache pour envoyer ses logs via Rsyslog

Éditer la configuration d'Apache :
```bash
sudo nano /etc/apache2/apache2.conf
```
Modifier la directive `ErrorLog` :
```bash
ErrorLog Syslog:local1
```

Redémarrer Apache :
```bash
# Pour CentOS/Rocky
sudo systemctl restart httpd

# Pour Ubuntu
sudo systemctl restart apache2
```

Configurer Rsyslog pour collecter les logs Apache :
```bash
sudo nano /etc/rsyslog.conf
```
Ajouter la ligne suivante :
```bash
local1.*@@<IP_SERVEUR_UBUNTU>:514
```

Redémarrer Rsyslog :
```bash
sudo systemctl restart rsyslog
```

## Vérification des logs Apache

Consulter les logs Apache en temps réel :
```bash
sudo tail -f /var/log/remote/apache-centos.log
sudo tail -f /var/log/remote/apache-rocky.log
```

Vérifier l'envoi des logs :
```bash
sudo tcpdump -i eth0 port 514
```

