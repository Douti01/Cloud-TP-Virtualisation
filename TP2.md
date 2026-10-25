# TP2: DEUX MARCHINES VIRTUELLES EN RESEAU VIRTUALBOX 

## Partie 1 : créer les VM par clonage 

### 1.1 clonage de l'image et créer le réseau 

**1.1.1 extinction de ubuntu** 
  **commande**

  '''
      powershell

      $env:Path += ";C:\Program Files\Oracle\VirtualBox"
      VBoxManage controlvm ubuntu-srv acpipowerbutton     # ignorer l'erreur si elle est déjà éteinte
      VBoxManage list runningvms                          # doit être vide
'''
** sortie**
<img width="918" height="180" alt="CP1" src="https://github.com/user-attachments/assets/59b74b0a-5999-457d-8566-65dd912d8a09" />

**1.1.2 Création de web et client**
**commande**

'''Powershell

      Measure-Command { VBoxManage clonevm ubuntu-srv --name web --register } | Select-Object TotalSeconds
      VBoxManage clonevm ubuntu-srv --name client --register
      VBoxManage modifyvm web    --cpus 1 --memory 1024
      VBoxManage modifyvm client --cpus 1 --memory 1024
'''
**sortie**
<img width="956" height="306" alt="CP2" src="https://github.com/user-attachments/assets/dd62ff29-01ce-4bbc-8548-e7135ed76d38" />


**1.1.3 Creation du reseau NAT**

**commande**

'''Powershell

    VBoxManage natnetwork add --netname tp-cloud --network "10.10.0.0/24" --enable --dhcp off
    VBoxManage natnetwork modify --netname tp-cloud --port-forward-4 "ssh-web:tcp:[127.0.0.1]:2222:[10.10.0.10]:22"
    VBoxManage natnetwork modify --netname tp-cloud --port-forward-4 "ssh-client:tcp:[127.0.0.1]:2223:[10.10.0.20]:22"
    VBoxManage natnetwork modify --netname tp-cloud --port-forward-4 "http-web:tcp:[127.0.0.1]:8080:[10.10.0.10]:80"
    VBoxManage natnetwork list

'''
**sortie**

<img width="947" height="412" alt="CP3" src="https://github.com/user-attachments/assets/1e1ec2f9-679d-42fd-b9a2-82d2b20fe832" />

**1.1.4 Branchage de la carte reseau**

**commande**

'''Powershell

    VBoxManage modifyvm web    --nic1 natnetwork --nat-network1 tp-cloud
    VBoxManage modifyvm client --nic1 natnetwork --nat-network1 tp-cloud
    VBoxManage showvminfo web | Select-String "Memory size|Number of CPUs|NIC 1"
    
'''

**sortie**
<img width="947" height="248" alt="CP4" src="https://github.com/user-attachments/assets/105301ea-bb66-4c9e-a3dc-ea118a99feea" />


### 1.2 Personnalisation du clonnage 

**1.2.1 Demarage web**

**commande**

''' console 

    sudo hostnamectl set-hostname web
    sudo sed -i 's/ubuntu-srv/web/g' /etc/hosts
    sudo rm -f /etc/machine-id && sudo systemd-machine-id-setup
    sudo rm -f /etc/ssh/ssh_host_* && sudo ssh-keygen -A

'''

**sortie**

<img width="795" height="476" alt="CP5" src="https://github.com/user-attachments/assets/125346a7-d41b-46fb-aa9f-082e92430999" />




**1.2.2 Adressage**

''' console 

      sudo tee /etc/netplan/99-tp.yaml > /dev/null <<'EOF'
      network:
        version: 2
        ethernets:
          enp0s3:
            dhcp4: false
            addresses: [10.10.0.10/24]
            routes:
              - to: default
                via: 10.10.0.1
            nameservers:
              addresses: [10.10.0.1, 1.1.1.1]
      EOF
      sudo chmod 600 /etc/netplan/99-tp.yaml
      sudo netplan apply
      ip -4 addr show enp0s3
      sudo reboot
'''

**1.2.3 Refaire les etape 5 et 6 sur client**

**sortie**

<img width="1196" height="217" alt="CP7" src="https://github.com/user-attachments/assets/5c39d34a-661f-47e2-baf0-ad1ceb5cad27" />

<img width="1202" height="367" alt="CP8" src="https://github.com/user-attachments/assets/98b3e3a9-3d51-4cb7-a9a1-31bddc12a01f" />

**1.2.4 Se connecter aux deux VM en SSH**

**commande**

'''Powershell

      ssh-keygen -R "[127.0.0.1]:2222"     # oublie l'ancienne empreinte de ubuntu-srv
      ssh-keygen -R "[127.0.0.1]:2223"
      ssh -p 2222 etudiant@127.0.0.1 "hostnamectl; lsb_release -a; uname -r"
      ssh -p 2223 etudiant@127.0.0.1 "hostname; ip -4 -br addr"

  '''

  **sortie**

  <img width="951" height="416" alt="CP9" src="https://github.com/user-attachments/assets/6fd6a726-991b-4454-8375-c63676aae315" />



### Questions 



**Q1. Durée du clonage et comparaisons**

• Temps de clonage de web :  22  secondes 
• Comparaison avec l'installation d'Ubuntu : Le clonage est nettement plus rapide (quelques secondes contre 15 à 30 minutes pour une installation complète de l'OS via ISO). L'installation nécessite de décompresser, copier et configurer chaque paquet individuellement, tandis que le clonage effectue une copie bloc par bloc ou fichier par fichier d'un disque déjà installé.
• Comparaison avec un serveur physique : La mise en service d'un serveur physique prend plusieurs jours à plusieurs semaines (commande du matériel, livraison, montage dans la baie, câblage réseau/électrique, configuration du BIOS/RAID, puis installation de l'OS). Le virtuel offre ici une agilité immédiate.

**Q2. Pourquoi personnaliser un clone et risques SSH**

• Pourquoi personnaliser : Un clone étant une copie conforme, conserver les mêmes identifiants provoque des conflits majeurs :
	• Nom d'hôte (hostname) : Rend l'identification difficile sur le réseau et perturbe les scripts d'automatisation.
	• Identifiant de machine (machine-id) : Utilisé par systemd et pour l'attribution des baux DHCP. S'il est identique, plusieurs machines peuvent se voir attribuer la même adresse IP par un routeur.
	• Clés d'hôte SSH : Elles garantissent l'identité du serveur auprès des clients.
• Risque de garder les mêmes clés SSH sur 50 serveurs :
	• Attaque de l'homme du milieu (MitM) : Si un attaquant compromet un seul des 50 serveurs, il récupère la clé privée. Il peut ensuite intercepter le trafic des 49 autres serveurs et déchiffrer les communications sans déclencher d'alerte de sécurité chez les utilisateurs.

Q3. Caractéristiques de web et concept Cloud

• Caractéristiques de web :
	• Version d'Ubuntu : Ubuntu 26.04.1 LTS 
	• Noyau (Kernel) : 7.0.0-30-generic.
	• Nombre de vCPU : 1 vCPU .
	• RAM : 1024 Mo (1 Go) .



  ## Partie 2 : Relier les VM en reseau 

 ###Connexion

  **commande**

  '''Powershell
      ssh -p 2223 etudiant@127.0.0.1
  '''
  **sortie

  <img width="1324" height="651" alt="CP1" src="https://github.com/user-attachments/assets/a4bb104b-91f1-4fb8-9583-9c38eceecd47" />

**commande**

    '''console

        ping -c 4 10.10.0.10
        ping -c 2 10.10.0.1
        ping -c 2 1.1.1.1
        ip route
        resolvectl status | grep "DNS Servers"

    '''

**sortie**

<img width="1093" height="573" alt="CP2" src="https://github.com/user-attachments/assets/a47aba54-e612-4b3d-a8b8-85f8fecaa31f" />


### nommer web 

**commande**

'''Powershell

      echo "10.10.0.10 web" | sudo tee -a /etc/hosts
      ping -c 2 web
      exit

  '''

**sortie**

<img width="1066" height="400" alt="CP3" src="https://github.com/user-attachments/assets/371bf6c0-63c5-46b1-907d-b513c8f2a113" />

**commande**

'''Powershell

    ping -n 2 10.10.0.10
'''

**sortie**

<img width="901" height="331" alt="CP4" src="https://github.com/user-attachments/assets/b82eec4a-e665-4392-8d8c-3b06a3bfb451" />





**Q1. Appartenance au même sous-réseau**

• Oui, les deux VM sont dans le même sous-réseau.
• Justification : Le masque /24 (ou 255.255.255.0) indique que les 24 premiers bits (les 3 premiers octets) définissent l'adresse du réseau, soit 10.10.0.0.
	• L'adresse de web est 10.10.0.10.
	• L'adresse de client est 10.10.0.20.
	• Leurs trois premiers octets étant identiques (10.10.0), elles partagent la même plage réseau et peuvent communiquer directement entre elles sans routeur intermédiaire.

**Q2. Passerelle par défaut de client et équivalents Cloud**

• Passerelle par défaut : L'adresse IP est 10.10.0.1 
• Rôle : Elle sert de point de sortie pour tous les paquets réseau dont la destination n'est pas dans le sous-réseau local (comme le trafic vers Internet ou 1.1.1.1). C'est le routeur virtuel géré par VirtualBox.
• Équivalent Cloud : Elle correspond à une Internet Gateway (IGW) dans un VPC AWS, ou à une Route de passerelle par défaut (0.0.0.0/0 via Internet) associée à une passerelle de sous-réseau dans un Azure VNet.

**Q3. Temps de réponse du ping entre client et web**

• Temps de réponse moyen : 0.3ms.
• Pourquoi il est si faible : Le trafic ne transite par aucun câble physique, carte réseau réelle ou commutateur matériel. Tout est émulé en mémoire vive (RAM) sur le même processeur de la machine hôte. La vitesse de transmission dépend uniquement de la vitesse de calcul du système, rendant la latence quasi nulle.

**Q4. Ping depuis Windows et accès SSH**

• Le ping depuis Windows aboutit-il ? Non, le ping échoue (généralement une erreur du type Délai d'attente de la demande dépassé).
• Pourquoi ? Le réseau NAT de VirtualBox isole les VM dans un sous-réseau privé virtuel. Windows (l'hôte) ne possède pas de route pour atteindre directement le réseau 10.10.0.0/24. Pour lui, ces adresses IP sont invisibles et inaccessibles.
• Comment l'hôte arrive-t-il à joindre la VM en SSH ? Grâce aux redirections de ports (Port Forwarding) configurées à l'étape 3 du TP. VirtualBox écoute sur l'interface locale de l'hôte (127.0.0.1) sur le port 2222 (pour web) et le port 2223 (pour client). Quand l'hôte se connecte en SSH sur l'un de ces ports locaux, VirtualBox intercepte le trafic et le traduit vers le port 22 interne de la VM correspondante.


## Partie 3: Publication de page web et protection de celle-ci

### 3.1 Installer Nginx 

**3.1.1 connexion**

**commande**

'''Powershell
    ssh -p 2222 etudiant@127.0.0.1
'''
**commande**

'''console
    sudo apt update && sudo apt install -y nginx
    systemctl status nginx --no-pager
    echo "<h1>Binôme NOM1 - NOM2 : ma première VM en réseau</h1>" | sudo tee /var/www/html/index.html
    exit

'''
**sortie**

<img width="1259" height="634" alt="CP5" src="https://github.com/user-attachments/assets/8e04bd3a-9718-407a-8796-d0ee1c26a1cb" />

<img width="709" height="166" alt="CP5-1" src="https://github.com/user-attachments/assets/c705232d-e270-4824-b7a9-6ba8f2c2a4a0" />

<img width="1168" height="391" alt="CP5-2" src="https://github.com/user-attachments/assets/114df5f5-5c25-423c-a56f-4816343db746" />

**3.1.2 Affichage de la page depuis client**

**commande**

'''Powershell 
    ssh -p 2223 etudiant@127.0.0.1 "curl -s http://web"

'''

###3.2 Fermerture du port 80

**3.2.1 Activation du parfeu


**commande**

''' console
      sudo ufw allow OpenSSH
      sudo ufw enable            # répondre y
      sudo ufw status verbose

'''


**sortie**

<img width="1028" height="348" alt="CP7" src="https://github.com/user-attachments/assets/81fe7ad2-dfbb-4d5d-b003-aa551daccdbe" />




**3.2.2 Actiualisation de la page au niveau du client**

**commande**

''' Powershell 

      ssh -p 2223 etudiant@127.0.0.1 "curl -m 5 http://web; echo code retour: `$?"
'''

**sortie**

<img width="931" height="125" alt="image" src="https://github.com/user-attachments/assets/fb65184d-7e79-403f-833d-948a6fe197f9" />


**3.2.3 Ouverture du port ssh**

**commande**


''' Powershell

    sudo ufw allow 'Nginx HTTP'
    sudo ufw status numbered
    sudo ss -tlnp

'''

**sortie**

<img width="1269" height="567" alt="image" src="https://github.com/user-attachments/assets/72fa26ba-86f2-49c6-8274-524b040d164f" />


**commande**

''' Powershell
      ssh -p 2223 etudiant@127.0.0.1 "curl -s http://web"

'''

**sortie**

<img width="904" height="211" alt="image" src="https://github.com/user-attachments/assets/3c06addd-cb62-46aa-8007-630ca1571979" />






### Questions 


**Q1. Comportement de curl et du navigateur (Port 80 fermé)**

• Retour de curl :
	• Message : curl: (28) Connection timed out .
	• Code retour ($?) : 28 (indique un dépassement de délai).
• Pourquoi le navigateur reste bloqué :
  UFW applique par défaut une politique "DROP" lorsqu'il bloque un port. Cela signifie qu'il rejette les paquets de manière totalement silencieuse, sans renvoyer de réponse au client. Le navigateur (ou curl) ne sait pas que le port est fermé ; il attend indéfiniment que le serveur réponde, jusqu'à ce que son propre mécanisme de sécurité de temps d'attente (timeout) se déclenche. Si UFW avait utilisé un rejet de type "REJECT", le serveur aurait renvoyé un paquet RST (Reset) et l'erreur aurait été immédiate.

**Q2. Écoute de Nginx (Analyse de ss -tlnp)**

• Port et adresses d'écoute : Nginx écoute sur le port 80 (HTTP).
• Justification par ss -tlnp : Dans la sortie de cette commande, vous verrez une ligne concernant nginx avec l'adresse locale sous la forme 0.0.0.0:80 (pour IPv4) et/ou [::]:80 (pour IPv6).
	• Le symbole  [::] signifie que Nginx écoute sur toutes les interfaces réseau disponibles de la VM web (l'interface locale localhost ainsi que l'interface réseau connectée au réseau de vos autres VM).

**Q3. Comparaison : UFW vs Groupe de Sécurité Cloud**

• Deux ressemblances :
	1. Filtrage par règles : Les deux fonctionnent avec un système de liste blanche (autoriser par défaut le trafic spécifié comme OpenSSH ou Nginx HTTP, et bloquer le reste).
	2. Filtrage par ports et protocoles : Les deux permettent de restreindre l'accès en se basant sur le port de destination (ex: 22, 80) et le protocole réseau (TCP/UDP).
• Une différence :
	• Emplacement de l'exécution : UFW est un pare-feu logiciel interne à l'OS (il s'exécute directement dans la machine virtuelle et consomme ses ressources). Un groupe de sécurité du Cloud est un pare-feu au niveau de l'infrastructure réseau (externe) ; le trafic indésirable est bloqué par l'hyperviseur du fournisseur Cloud avant même d'atteindre la carte réseau de la machine virtuelle.

**Q4. Responsabilité IaaS (Modèle Shared Responsibility)**

Dans le modèle de service IaaS (Infrastructure as a Service), la démarcation se fait au niveau de la virtualisation.
• Couches fournies par VirtualBox (Le fournisseur de Cloud) :
	• Matériel / Hardware (Serveur physique, processeur, mémoire de l'hôte).
	• Réseau physique (Cartes réseaux et commutateurs physiques de l'hôte).
	• Stockage physique (Disque dur/SSD de l'hôte).
	• Virtualisation / Hyperviseur (Le logiciel VirtualBox lui-même qui simule le matériel).
• Couches gérées par vous-même (Le client) :
	• Système d'exploitation (OS) : L'installation, la configuration et la mise à jour d'Ubuntu sur la VM web.
	• Pare-feu / Sécurité : La configuration de l'outil UFW.
	• Middleware / Runtime : La configuration réseau fine et les services système.
	• Applications : L'installation et la gestion du serveur web Nginx.
	• Données : Les pages web et fichiers servis par Nginx.


## Partie 4 : Mesurer et Netoyer 

**4.1 Mesure des resource**

**commande**

'''  Pöwershell

      ssh -p 2222 etudiant@127.0.0.1 "free -h; df -h /; nproc"
      
'''

**sortie**

<img width="917" height="252" alt="image" src="https://github.com/user-attachments/assets/1aeb6842-25d5-48e9-b3f5-63b272d9b07b" />

**4.2 Comparaison avec ce que l'on voit**

**commande**

''' Powershell 

        Get-ChildItem "$HOME\VirtualBox VMs" -Directory | ForEach-Object { "{0} : {1:N2} Go" -f $_.Name, 
        ((Get-ChildItem $_.FullName -Recurse -File | Measure-Object Length -Sum).Sum / 1GB) }

'''

**sortie**

<img width="957" height="105" alt="image" src="https://github.com/user-attachments/assets/68712bd0-28bc-41e6-833a-d2dc8c14e2d6" />



**4.3 Redemarage**

**commande**

''' Powershell

        VBoxManage controlvm web acpipowerbutton
        VBoxManage controlvm client acpipowerbutton
        VBoxManage list runningvms
        VBoxManage startvm web --type headless
        VBoxManage controlvm web poweroff
        VBoxManage unregistervm web --delete
        VBoxManage unregistervm client --delete
        VBoxManage natnetwork remove --netname tp-cloud
        VBoxManage list vms                     # seule ubuntu-srv doit rester

'''

**sortie**

<img width="1350" height="495" alt="image" src="https://github.com/user-attachments/assets/d78f3167-5b24-4e26-ba61-6e344a577856" />





###Questions


**Q1. RAM vue par la VM vs l'hôte et mécanisme d'allocation**

• Ce que voit la VM : Non, la VM web ne voit pas exactement 1 Go de RAM (1024 Mo). La commande free -h affiche généralement autour de 960 Mo à 980 Mo. Une petite partie de la mémoire est réservée dès le démarrage par le noyau Linux (kernel) pour ses propres structures et pour le matériel virtuel simulé.
• Consommation réelle sur l'hôte : Dans le Gestionnaire des tâches, le processus VirtualBoxVM.exe correspondant consomme souvent un peu plus de 1 Go (par exemple 1,05 Go ou 1,1 Go).
• Explication de l'écart : L'hyperviseur (VirtualBox) a besoin de sa propre mémoire pour faire fonctionner la machine virtuelle : émuler les composants (carte réseau, carte graphique virtuelle, contrôleur disque) et maintenir la table de pagination de la mémoire (la correspondance entre la RAM de la VM et la RAM réelle). La mémoire consommée sur l'hôte est donc toujours égale à : RAM allouée à la VM + Coût de fonctionnement de l'hyperviseur.

**Q2. Taille du disque virtuel (Virtuelle vs Réelle)**

• Place réelle sur l'hôte : La commande PowerShell montre que le dossier du clone occupe en réalité entre 2 Go et 4 Go seulement sur le disque dur du PC hôte (au lieu des 25 Go vus par la VM).
• Pourquoi : Le disque virtuel a été créé au format dynamiquement alloué (Dynamically allocated). Lors de sa création, le fichier de la VM (souvent un .vdi ou .vmdk) est presque vide. Il ne grossit sur l'hôte qu'au fur et à mesure que vous écrivez de vrais fichiers à l'intérieur de la VM (comme l'installation d'Ubuntu et de Nginx). Les 25 Go représentent uniquement la taille maximale que le disque est autorisé à atteindre.

**Q3. Différence entre les commandes d'extinction et de suppression**

• acpipowerbutton : Envoie un signal d'extinction logiciel à la VM, identique à une pression sur le bouton physique d'un PC. C'est un arrêt propre : l'OS (Ubuntu) interrompt proprement ses services, démonte les disques et s'éteint sans perte de données.
• poweroff : Coupe instantanément le courant de la VM (comme débrancher la prise d'un PC physique). C'est un arrêt brutal. On ne l'utilise que si la VM est totalement plantée, car cela peut corrompre le système de fichiers.
• unregistervm --delete : Supprime la VM des registres de VirtualBox ET efface définitivement tous les fichiers associés stockés sur le disque de l'hôte (fichiers de configuration, fichiers de logs et surtout les disques virtuels .vdi). La VM est définitivement détruite.

**Q4. Facturation d'une VM arrêtée dans le Cloud**

Dans le cloud public (AWS, Azure, GCP), une VM arrêtée n'est plus facturée pour sa puissance de calcul (CPU et RAM), mais elle reste facturée pour ses ressources persistantes.
Voici pourquoi la facture ne tombe pas à zéro :
• Le stockage (Disque) : Même si la VM ne tourne pas, vos données et votre OS restent stockés sur les disques du fournisseur (les volumes type AWS EBS). Vous payez le coût de stockage au Go/mois tant que le volume n'est pas supprimé.
• L'adresse IP publique : Les adresses IPv4 publiques sont rares. Si vous conservez une IP publique statique (IP réservée) alors que votre VM est arrêtée, la plupart des fournisseurs du Cloud vous la facturent (parfois même plus cher lorsqu'elle est inutilisée) pour vous inciter à la libérer afin que d'autres clients puissent en profiter.










