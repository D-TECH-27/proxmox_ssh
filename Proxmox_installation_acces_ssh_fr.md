Pour installer un serveur Proxmox, vous devez avoir :

* une machine physique pour le serveur connecté au réseau (n'oubliez pas de formater le disque physique de la machine)
* une clé USB formatée
* une autre machine (avec un logiciel de formatage (balenaEtcher ou Rufus) et une connexion par câble)
* un fichier image disque (ISO) de Proxmox Virtual Environment

Étape 1 : Récupérez le fichier ISO de Proxmox VE via le site officiel : https://www.proxmox.com/en/downloads/proxmox-virtual-environment/iso

Dans ce cas, nous pouvons télécharger la dernière version (9.2)

Étape 2 : Insérez la clé USB formatée et ouvrez le logiciel balenaEtcher ou Rufus. Vous pouvez le télécharger sur les sites officiel :

* balenaEtcher : https://etcher.balena.io/#download-etcher
* Rufus : https://rufus.ie/fr/#download

Étape 3 :

* Sur balenaEtcher :
* Sélectionnez "Flash From File" et sélectionnez le fichier ISO :

<img width="799" height="512" alt="image" src="https://github.com/user-attachments/assets/6403afaa-7c79-4540-b023-e40ef993f524" />

* Puis, cliquez sur "Select target" pour choisir votre clé USB

<img width="792" height="509" alt="image" src="https://github.com/user-attachments/assets/5cf1d86e-7fd6-4bb4-9d04-2bc31b0b667f" />

* Enfin, cliquez sur "Flash" et attendez !

<img width="794" height="502" alt="image" src="https://github.com/user-attachments/assets/6ab79d2e-ec37-4efa-b1c4-19716e1fb85f" />

* Sur Rufus :
* Votre clé USB est sélectionné automatiquement. Si ce n'est pas le cas, sélectionnez votre clé USB dans le champ "Périphérique". Puis, cliquez sur "SÉLECTION" (pas la flèche) pour choisir votre fichier ISO
(Si vous ne trouvez pas le fichier, ouvrez votre Explorateur de Fichiers et glissez-déposez votre fichier. Un message apparaît mais vous pouvez l'ignorer en cliquant sur "OK").

Cliquez sur "DÉMARRER" et attendez !
<img width="501" height="535" alt="image" src="https://github.com/user-attachments/assets/21711c01-3be6-4ab1-b6f4-680be5216cd5" />

N'oubliez pas d'éjecter la clé USB quand cela est terminé !

Étape 4 : Sur l'autre machine (le serveur Proxmox), insérez la clé USB, démarrez la machine et appuyez sur le bouton indiqué ci-dessous (en fonction de la marque de votre machine) pendant que la machine est en train de démarrer pour accéder au BIOS :

* ASUS : F2
* HP : F10, F2, Échap ou F6
* DELL : F2
* Acer : F2
* Gigabyte : Suppr
* ASRock : F2
* Lenovo : F2

Recherchez le menu d'ordre de démarrage et changez la priorité de démarrage en sélectionnant votre clé USB et en la glissant vers le premier périphérique à démarrer. Sauvegardez et quittez !

Étape 5 : Sélectionnez "Proxmox VE (Graphical)" et appuyez sur Entrée

L'installation permet :

* Sélectionner le disque pour installer Proxmox (sur le SSD ou le HDD de la machine et non pas sur la clé USB, merci !)
* Sélectionner la langue et la localisation
* Insérer le mot de passe "root" (root est l'utilisateur par défaut ayant tous les droits sur un système)
* Sélectionner une carte réseau et insérer ses paramètres IP comme son adresse, son masque et sa passerelle (tout en statique)

Le système va redémarrer après l'installation. Quand la machine redémarre, retirez la clé USB

Étape 6 : Allez sur la prmière machine et ouvrez un navigateur web et insérez :

&#x20;     http://\[adresse IP de Proxmox]:8006


Si vous avez accès à votre Proxmox, connectez-vous en root !

Étape 7 : Sur le serveur Proxmox, insérez cette commande pour installer ces outils :

* vim : Éditeur de texte
* htop : Informations des composants
* net-tools : Informations réseau
* git : Importation de paquets Github
* nmap : Analyseur de trames
* wget : Importation de paquets

... et bien plus

&#x20;     apt install -y vim htop iotop iftop curl wget git net-tools dnsutils lsof tree tcpdump nmap iperf3 mtr smartmontools nvme-cli unattended-upgrades chrony -y


Si ces paquets ne sont pas disponibles, vous pouvez les retirer et appuyer sur Entrée. Ce sont juste des paquets qui n'impacteront pas le système en lui-même sur votre configuration.

Étape 8 : SSH

Pour générer une clé SSH (dans ce cas, avec un chiffrement ed25519) sur Linux, insérez cette commande :

&#x20;     ssh-keygen -o -a 256 -t ed25519 -C "$(hostname)-$(date +'%d-%m-%Y')"


Appuyez sur Entrée 3 fois (pas de mot de passe à saisir).

Étape 9 : Donnez tous les accès à la clé SSH à "root"

&#x20;     chmod 700 /root/.ssh
      chmod 400 /root/.ssh/id\_ed25519
      chown root:root /root/.ssh/id\_ed25519\*


Étape 10 : Sur votre première machine, ouvrez un Invite de Commande et insérez cette commande :

* Sur Windows : ssh-keygen
* Sur Linux : voir Étape 8
* Sur MacOS : ssh-keygen -t ed25519

Étape 11 : Importez la clé SSH sur le serveur Proxmox. Dans ce cas, j'utilise Windows PowerShell

&#x20;     type "%USERPROFILE%\\.ssh\\id\_ed25519.pub" | ssh root@\[Proxmox IP address] "cat >> .ssh/authorized\_keys"


Vous pouvez retirer le "%USERPROFILE%" le cas échéant.

Étape 12 : Vérifiez la clé importée sur le serveur en accédant au fichier "authorized\_keys" :

&#x20;     nano /.ssh/authorized\_keys


Étape 13 : Allez au fichier de configuration SSH :

&#x20;     nano /etc/ssh/sshd\_config


Et retirez le symbole faisant office de commentaire (#) en appliquant ou modifiant ces paramètres :

* Protocol 2
* Port 22
* AddressFamily inet
* AllowUsers root@\[adresse IP de la première machine]
* ListenAddress \[adresse IP du serveur Proxmox]
* HostKey /etc/ssh/ssh\_host\_ed25519\_key
* LoginGraceTime 2m
* PermitRootLogin yes
* MaxAuthTries 3
* MaxSessions 10
* PubkeyAuthentication yes
* AuthorizedKeysFile .ssh/authorized\_keys
* HostBasedAuthentication no
* IgnoreRHosts yes
* PasswordAuthentication no
* PermitEmptyPasswords no
* PrintMotd no
* PrintLastLog yes
* TCPKeepAlive yes
* ClientAliveInterval 1800
* ClientAliveCountMax 3
* UseDNS no
* MaxStartups 2

Enregistrez le fichier en appuyant sur Ctrl+O et quittez !

Étape 14 : Test de SSH

Insérez la commande sur l'Invite de Commande de Windows :

&#x20;     ssh root@\[Adresse IP de Proxmox]


(si le port par défaut a été modifié (22 par 443 par exemple), ajoutez "-p \[numéro de port]" à la fin).

