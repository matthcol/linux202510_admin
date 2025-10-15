pwd : où suis je

ls    : lister
ls -a : lister tout (y compris fichiers cachés)
ls -l : avec détail
ls -lh : détail et taille avec la bonne unité 
ls -al : 2 options cumulées
ls --help : aide de la commande

wc -l : compter des lignes

Visualisation e fichier (texte):
cat : voir le contenu d'un fichier
more : idem (défilement)
less : mieux que more
head : début du fichier
tail : fin de fichier

Editeur de texte
vi
vim
nano




grep sudo /etc/group        # recherche du mot sudo dans le fichier /etc/group
ls /usr/bin | grep ls       # recherche du mot ls dans le résultat de la commande précédente

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
sudo su -               # se reloguer en tant que root
sudo su - aurelien      # se reloguer en tant qu'aurélien
sudo su                 # changer d'identité => root (environnement et répertoire inchangé)
sudo su aurelien        # changer d'identité => aurelien (environnement et répertoire inchangé)

## Variables d'environnments

env                         # lister toutes les variables
echo $PATH                  # afficher le contenu d'une variable
export METEO_TMP_DIR=/tmp   # (re)définir une variable

## Installation de logiciel via la distribution
Gestionnaire pour debian, ubuntu: apt

apt update
apt install unzip

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
systemctl status tomcat
systemctl stop tomcat
systemctl start tomcat
systemctl enable tomcat
systemctl disable tomcat

### Journaux de services
journalctl -u ssh
journalctl -u ssh -n 20     # 20 dernières lignes

### Types de services
- simple (défaut)
- forking
- oneshot
- notify
- dbus
- idle

### Niveau de démarrage
Un fichier .target rassemble des services. L'OS démarre sur un fichier .target.

systemctl get-default               # niveau de démarrage
systemctl set-default multi-user    # changer le niveau de démarrage
systemctl isolate rescue            # changer à chaud le niveau

### Reboot

reboot
shutdown -r
systemctl isolate reboot
init 6  # compatibilité System V

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


