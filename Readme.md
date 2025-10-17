# Commandes Linux

## Système de fichiers
```
pwd : où suis je

ls    : lister
ls -a : lister tout (y compris fichiers cachés)
ls -l : avec détail
ls -lh : détail et taille avec la bonne unité 
ls -al : 2 options cumulées
ls --help : aide de la commande

wc -l : compter des lignes
```

Visualisation e fichier (texte):
```
cat : voir le contenu d'un fichier
more : idem (défilement)
less : mieux que more
head : début du fichier
tail : fin de fichier
```

Editeur de texte
```
vi
vim
nano
```


```
grep sudo /etc/group        # recherche du mot sudo dans le fichier /etc/group
ls /usr/bin | grep ls       # recherche du mot ls dans le résultat de la commande précédente
```
## Utilisateurs

Supervision
```
groups
id
whoami
```

Gestion des utilisateurs
```
useradd
usermod
userdel
```

Gestion des groupes
```
groupadd
groupmod
groupdel
```

Scenario 1:
```
useradd aurelien                # paramètres par défaut (shell=sh)
usermod -s /bin/bash aurelien   # changer le shell
userdel -r aurelien             # supprimer complète
```

Scenario 2: créer utilisateur en spécifiant le shell et en créant le home directory
```
sudo useradd -m -s /bin/bash aurelien
sudo useradd -m -s /bin/bash stephane
```

sudo une_commande : faire la commande en tant que root
```
sudo su -               # se reloguer en tant que root
sudo su - aurelien      # se reloguer en tant qu'aurélien
sudo su                 # changer d'identité => root (environnement et répertoire inchangé)
sudo su aurelien        # changer d'identité => aurelien (environnement et répertoire inchangé)
```

## Variables d'environnments
```
env                         # lister toutes les variables
echo $PATH                  # afficher le contenu d'une variable
export METEO_TMP_DIR=/tmp   # (re)définir une variable
```
## Installation de logiciel via la distribution
Gestionnaire pour debian, ubuntu: apt

```
apt update
apt install unzip
```

## Atelier
Installation de Tomcat manuelle en passant par le zip.

NB: il vaut mieux passer par le tar.gz, l'objectif est purement pédagogique ici.

### Téléchargement
En tant que matthias (admin)
```
cd
mkdir Installers
cd Installers
wget https://dlcdn.apache.org/tomcat/tomcat-11/v11.0.13/bin/apache-tomcat-11.0.13.zip
```

### Désarchivage
```
sudo apt update
sudo apt install unzip
unzip -t apache-tomcat-11.0.13.zip      # test archive

cd /opt
sudo unzip ~/Installers/apache-tomcat-11.0.13.zip
sudo unzip $HOME/Installers/apache-tomcat-11.0.13.zip
sudo unzip /home/matthias/Installers/apache-tomcat-11.0.13.zip

sudo apt install tree
tree
tree -L 2
tree -L 3

sudo ln -s apache-tomcat-11.0.13/ apache-tomcat
```

### Utiliser un user dédié
- Créer un user système tomcat (uid < 1000) dont la maison est /opt/apache-tomcat/
- Changer le propriétaire de l'arborescence tomcat (chown)

```
sudo useradd -r -d /opt/apache-tomcat -s /bin/bash tomcat
sudo chown -R tomcat:tomcat apache-tomcat-11.0.13/
sudo su - tomcat
```

En tomcat:
```
pwd
cd bin
ls -l    # il manque le droit x sur les .sh
chmod 755 *.sh
ls -l *.sh
```

### Installer Java
En admin:
```
sudo apt install openjdk-21-jre
java --version
```

### Lancer Tomcat
En tomcat:
```
cd bin
export JRE_HOME=/usr/lib/jvm/java-21-openjdk-amd64
echo $JRE_HOME
env
./startup.sh
```

Vérifier si tomcat tourne en arrière plan:
```
ps -aux | grep java
ps -aef | grep java
ss -plantu | grep LISTEN

sudo apt install net-tools
netstat -plan | grep -i listen    # -i = ignore case
```

## Mot de passes
- fichier /etc/shadow
- changement : passwd
- gestion durée : chage

## Services Systemd
### Gestion des services
```
systemctl status tomcat
systemctl stop tomcat
systemctl start tomcat
systemctl enable tomcat
systemctl disable tomcat
```

### Journaux de services
```
journalctl -u ssh
journalctl -u ssh -n 20     # 20 dernières lignes
```

### Types de services
- simple (défaut)
- forking
- oneshot
- notify
- dbus
- idle

### Niveau de démarrage
Un fichier .target rassemble des services. L'OS démarre sur un fichier .target.

```
systemctl get-default               # niveau de démarrage
systemctl set-default multi-user    # changer le niveau de démarrage
systemctl isolate rescue            # changer à chaud le niveau
```

### Reboot
```
reboot
shutdown -r
systemctl isolate reboot
init 6  # compatibilité System V
```

## Aptitude
Gestionnaire de paquets ubuntu (ou debian)

```
apt update                  # à faire régulièrement ou après install repo supplémentaire
apt search postgresql
apt install postgresql-18
apt remove xfce4
apt autoremove              # nettoyage après remove
```

## Alternatives
Editeur par défaut (visudo, git, ...):

```
sudo update-alternatives --config editor
```

Version de java:
```
sudo update-alternatives --config java
```

## Réseaux

### Ancienne recette

Le fichier /etc/network/interfaces

```
# /etc/network/interfaces
# Configuration réseau en DHCP

# Interface loopback (obligatoire)
auto lo
iface lo inet loopback

# Interface Ethernet principale en DHCP
auto enp0s3
iface enp0s3 inet dhcp

# Interface Ethernet principale en DHCP
auto enp0s8
iface enp0s8 inet dhcp
```

### Netplan (ubuntu)

Fichier /etc/netplan/50-cloud-init.yaml (ou autre dans ce répertoire)
en DHCP
```
network:
  version: 2
  ethernets:
    enp0s3:
      dhcp4: true
    enp0s8:
      dhcp4: true
      optional: true
```

En version statique:
```
network:
  version: 2
  ethernets:
    enp0s3:
      dhcp4: true
    enp0s8:
      dhcp4: no
      optional: true
      addresses:
        - 192.168.173.3/24
#      routes:
#        - to: 192.168.173.0/24
#          scope: link  
#          # better than gateway for host-only (via: 192.168.173.1)
```

```
sudo netplan generate
sudo netplan apply
```
ou 
```
sudo netplan try
````

### Nom d'hôte

```
/etc/hostname
/etc/hosts
```

### Résolution de noms et routage
Resolution DNS
```
nslookup github.com
dig github.com
ping github.com   # 1ère étape du ping
```
Routage:
```
ip route
ip route get 127.0.0.1
ip route get 140.82.121.3
ip route get 192.168.173.4
ip route get 192.168.173.5
```

## Ansible

Valider l'inventaire (fichier hosts)
```
ansible -k -i hosts -m ping all
```
Création d'un jeu de clé SSH pour le déploiement (id_rsa_deploy,  id_rsa_deploy.pub):
```
ssh-keygen -t rsa
```
Jouer le playbook Ansible:
```
ansible-playbook -k -K -i hosts playbook-deploy.yml
```
Agent SSH
```
ssh-agent   # copy-paste result
ssh-add -L  # liste des clés déverouillées
ssh-add ~/.ssh/id_rsa_deploy   # confier une clé

ssh deployer@192.168.173.4                                # session interactive
ssh deployer@192.168.173.4 sudo systemctl stop tomcat10   # commande à distance
scp /etc/passwd deployer@192.168.173.4:/tmp

ssh-add -D    # retirer toutes les clés  (-d 1 clé)
```

## Gestion de disques

GUI: gparted
```
df
df -h     # unité informatique
df -H     # unité internationale

lsblk
lsblk -f  # display FS

fdisk /dev/sdb  # partitionner (parted, gparted)

sudo mkfs.ext4 /dev/sdb1
sudo mkfs -t ext4 /dev/sdb2
```

Créer un gros fichier à partir /dev/zero ou /dev/random
```
dd bs=1024 count=1048576 if=/dev/zero of=video2.mp4
```
Créer plein de fichier
```
for i in $(seq 1 50000); do touch "cookie-$i.txt"; done
for i in $(seq 1 100); do  echo "contenu $i" > "cookie-$i.txt"; done
```
Si disque système monté en lecture seule, on peut tenter de le remonter en écriture
```
sudo mount -o remount,rw /
```

Changer l'UUID (ext4):
- tune2fs:
```
# UUID aléatoire
sudo tune2fs -U random /dev/mapper/vg-lv

# UUID spécifique
sudo tune2fs -U 12345678-1234-1234-1234-123456789abc /dev/mapper/vg-lv

# Voir l'UUID
lsblk -f
sudo tune2fs -l /dev/mapper/vg-lv | grep UUID
sudo dumpe2fs /dev/mapper/vg-lv | grep UUID
sudo blkid /dev/mapper/vg-lv
```

- alternative avec uuidgen
```
# Générer un UUID
NEW_UUID=$(uuidgen)
echo $NEW_UUID

# L'appliquer
sudo tune2fs -U $NEW_UUID /dev/mapper/vg-lv
```

- xfs : utiliser les outils xfs_admin et xfs_metadump

### Checker un disque
NB: se fait automatiquement au bout de plusieurs jours ou plusieurs boot:
- ext4
```
e2fsck -f /dev/sdb1
```

## LVM

Infos
```
pvs
vgs
lvs

pvdisplay
pvdisplay /dev/sda3

vgdisplay
vgdisplay ubuntu-vg

lvdisplay
lvdisplay /dev/ubuntu-vg/ubuntu-lv
```

Etendre un LV
```
lvextend -L +2G -r /dev/ubuntu-vg/ubuntu-lv
lvextend -l +100 -r /dev/ubuntu-vg/ubuntu-lv
```

Approvisionner un VG avec un nouveau PV
vgextend ubuntu-vg /dev/sdb3

Creer 3 niveaux:
```
vgcreate -f docker-vg /dev/sdc
vgs
lvcreate -n docker-lv -L 3G docker-vg
lvcreate -n volume-lv -L 3G docker-vg
lvs
```

Formattage:
```
mkfs -t ext4 /dev/docker-vg/docker-lv
mkfs -t ext4 /dev/docker-vg/volume-lv
```

Fichier fstab (extrait):
```
/dev/docker-vg/docker-lv /var/lib/docker ext4 defaults 0 1
/dev/docker-vg/volume-lv /var/lib/docker/volumes ext4 defaults 0 1
```

## Docker
### Installation
Ajouter du repot officiel de docker avec sa clé:
https://docs.docker.com/engine/install/ubuntu/

Attention à cause des droits restreints, faire sudo df -h pour voir les volumes LVM
associés à docker montés.

### Premières commandes
```
docker pull mysql:8       # télécharger une image
docker image ls           # lister les images
docker run --name mysql-dbski -e MYSQL_ROOT_PASSWORD=my-secret-pw -d mysql:8
docker exec -it mysql-dbski bash
    mysql -u root -p
        show databases;
```

### Composition docker
Exemple de composition avec base de données métier et mapping de port (3316 -> 3306)
```
cd ~/linux202510_admin/docker-mysql-dbmovie
docker compose up -d    # lance le(s) conteneurs décrit(s) dans le docker-compose.yml
docker ps
sudo ss -plantu         # montre le port 3316 ouvert sur ubuntu et redirigé vers docker
```

## Planification
2 outils:
- at : ponctuel
- cron : régulier

at -t 10171406 # saisi le job sur l'entree standard (vérifier le vocabulaire dispo sur votre version de at)
```
at now + 30 minutes
at now + 2 hours
at now + 3 days
at now + 1 week
at tomorrow
at 10:30 tomorrow
at 14:00 friday
at 09:00 next monday
at noon
at midnight
at 5:30 PM
```

atq       # liste de jobs en attente
atrm 5    # suppression job 5
at -c 5   # le detail du job 5

echo 'tar czf "formation-$(date +%Y%m%d%H%M).tgz" linux202510_admin' | at -t 10171500

## Cron
1 fichier crontab permet de planifier des tâches avec cron (service)

2 types de crontab:
- systeme : /etc/crontab + /etc/cron.d/*
- user

crontab -l   # voir sa crontab

Exemple de crontab système extra: /etc/cron.d/dumber
```
*/5 9-17 * * 1-5 matthias date >> /tmp/dumber.txt
```

NB: attention à la variable PATH, au current directory et pas d'affichage (stdout ou stderr)

## Archivage tar
- cvf : c (create) v (verbose) f (file)
- cvzf : create + gzip (ou 1 autre lettre pour un autre format)
- tvf : tester
- xvf : extraire


```
tar cvf formation.tar linux202510_admin/
tar cvfz formation.tar.gz linux202510_admin/
tar cvfz formation-2025101715000.tgz linux202510_admin/
```

Outils de (dé)compression:
- zip/unzip
- gzip/gunzip
- bzip2/bunzip2
- 7z