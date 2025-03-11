# Configuration du réseau et du DHCP

## Configuration du réseau

```bash
sudo vi /etc/netplan/10-lxc.yaml
```

```yaml
network:
  version: 2
  ethernets:
    eth0:
      dhcp4: true
      dhcp-identifier: mac
      addresses: [10.11.35.162/28] # A remplacer par l'adresse IP souhaitée
      routes:
        - to: default
          via: 10.11.35.174 # A remplacer par l'adresse IP de la passerelle
      nameservers:
        addresses: [1.1.1.3]
```

### Commandes `vi`

- `i` : Pour être en mode insertion
- `echap` : Pour quitter le mode insertion
- `:wq` : Sauvegarder et quitter

## Configuration du DHCP

```bash
sudo vi /etc/udhcpd.conf
```

### Définir la plage d'adresses IP à attribuer

```plaintext
start   192.168.10.2    # Première adresse IP de la plage DHCP
end     192.168.10.14   # Dernière adresse IP de la plage DHCP
```

### Interface réseau sur laquelle le serveur DHCP écoute

```plaintext
interface ens33         # Remplacez par le nom de votre interface réseau
```

### Masque de sous-réseau

```plaintext
option  subnet  255.255.255.240
```

### Passerelle par défaut

```plaintext
opt     router  192.168.10.1    # Remplacez par l'adresse IP de votre passerelle
```

### Serveurs DNS

```plaintext
opt     dns     192.168.10.2 192.168.10.10  # Remplacez par les adresses DNS de votre réseau
option  dns     129.219.13.81               # DNS supplémentaire
```

### Domaine local

```plaintext
option  domain  local
```

### Durée du bail DHCP (en secondes)

```plaintext
option  lease   864000          # 10 jours
```

### Fichier de boot PXE (pour le démarrage réseau)

```plaintext
opt     bootfile /amd64/pxelinux.0  # Chemin du fichier de boot sur le serveur TFTP
```

### Adresse IP du serveur TFTP

```plaintext
siaddr  192.168.10.2    # Remplacez par l'adresse IP de votre serveur TFTP
```

### Options supplémentaires (facultatives)

```plaintext
opt     wins    192.168.10.10   # Serveur WINS (si utilisé)
option  msstaticroutes  10.0.0.0/8 10.127.0.1           # Route statique
option  staticroutes    10.0.0.0/8 10.127.0.1, 10.11.12.0/24 10.11.12.1
option  0x08    01020304        # Option personnalisée en hexadécimal
option  14      "dumpfile"      # Option personnalisée en chaîne de caractères
```

### Appliquer la configuration et redémarrer le service

```bash
sudo systemctl restart udhcpd
sudo systemctl enable udhcpd
sudo systemctl status udhcpd
```

### Configuration du serveur TFTP

Pas eu besoin de configurer le serveur tftpd, la configuration existante était déjà suffisante.

```bash
sudo systemctl restart tftpd-hpa
```

## Ajout d'ArchLinux en tant que boot

```bash
sudo apt install syslinux -y
sudo cp /usr/lib/syslinux/modules/bios/* /srv/tftp/amd64/
sudo wget https://archlinux.org/static/netboot/jpxe-arch.5ee66f360339.pxe -P /srv/tftp/amd64
```

### Configuration du menu PXE

```bash
sudo nano /srv/tftp/amd64/pxelinux.cfg/default
```

```plaintext
DEFAULT menu.c32
PROMPT 0
MENU TITLE PXE Boot Menu

LABEL Ubuntu
  MENU LABEL Live Ubuntu
  KERNEL /amd64/ubuntu-installer/amd64/linux
  APPEND initrd=/amd64/ubuntu-installer/amd64/initrd.gz

LABEL Arch
  MENU LABEL Live Arch Linux
  KERNEL /amd64/jpxe-arch.5ee66f360339.pxe
```

### Activer les services

```bash
sudo systemctl enable udhcpd
sudo systemctl enable tftp-hpa
```

### Redémarrer le système

```bash
sudo reboot
```
