# TP 1 — Installation de VirtualBox et création d'une VM Ubuntu

- **Cours :** Introduction au Cloud
- **Étudiant(s) :**
-                   SANOGO DOUTI AMADOU
-                   KARAMOKO ISSIAKA
-                   DAHOUE MAMADOU
- **Date :** 14/10/2026

---

## Partie 1 — Préparer l'hôte et installer VirtualBox

### Commandes 1 :  de verification de l'hyperviseur 
'''Powershell

		bcdedit /enum | findstr hypervisorlaunchtype
		systeminfo | findstr /C:"Hyper-V"
'''
### Sorties de la commande 1
'''Powershell

		>>
		hypervisorlaunchtype    Auto
		Configuration requise pour Hyper-V:         Un hyperviseur a ?t? d?tect?. Les fonctionnalit?s n?cessaires ? Hyper-V ne seront pas affich?es.
'''
### Commande 2 :  de verification de l'installation de virtual box .
''' Powershell

		$env:Path += ";C:\Program Files\Oracle\VirtualBox"
		VBoxManage --version
		VBoxManage list hostinfo
'''
### Sorties de la commande 2 
'''

		>>
		7.2.20r175154
		Host Information:
		
		Host time: 2026-10-13T16:00:05.188000000Z
		Processor online count: 8
		Processor count: 8
		Processor online core count: 4
		Processor core count: 4
		Processor supports HW virtualization: yes
		Processor supports PAE: yes
		Processor supports long mode: yes
		Processor supports nested paging: yes
		Processor supports unrestricted guest: no
		Processor supports nested HW virtualization: no
		Processor supports virt. vmsave/vmload: no
		Processor#0 speed: unknown
		Processor#0 description: Intel(R) Core(TM) i5-8250U CPU @ 1.60GHz
		Processor#1 speed: unknown
		Processor#1 description: Intel(R) Core(TM) i5-8250U CPU @ 1.60GHz
		Processor#2 speed: unknown
		Processor#2 description: Intel(R) Core(TM) i5-8250U CPU @ 1.60GHz
		Processor#3 speed: unknown
		Processor#3 description: Intel(R) Core(TM) i5-8250U CPU @ 1.60GHz
		Processor#4 speed: unknown
		Processor#4 description: Intel(R) Core(TM) i5-8250U CPU @ 1.60GHz
		Processor#5 speed: unknown
		Processor#5 description: Intel(R) Core(TM) i5-8250U CPU @ 1.60GHz
		Processor#6 speed: unknown
		Processor#6 description: Intel(R) Core(TM) i5-8250U CPU @ 1.60GHz
		Processor#7 speed: unknown
		Processor#7 description: Intel(R) Core(TM) i5-8250U CPU @ 1.60GHz
		Memory size: 16227 MByte
		Memory available: 5721 MByte
		Operating system: Windows 11
		Operating system version: 10.0.26200.9168
		PS C:\WINDOWS\system32>

'''
### Reponse aux questions 

1. **Hyperviseur de type 1 et type 2**
   **Hyperviseur de type 1**
   
   - **installation**: '''Il s'installe directement sur le matériel de la machine'''
   - **Performance** : '''**Excellente**, car IL a  un accès optimisé aux ressources physique (CPU, RAM)'''
   - **utlisation** : ''Environnement basé sur des serveurs (Data center , cloud )
   **Hyperviseur de type 2**
     - **installation**: '''Il ne s'installe pas  directement sur le matériel de la machine. Il fonctionne plutôt comme une application
                            dans un système d'exploitation '''
     - **Performance** :''' **Moindres**,  car pour chaque requête il doit parcourir le système hôte'''
     -  **utlisation** : ''' Pour une utilisation individuel '''

    **Hyperviseur de type 1 utilisé par un founisseur cloud**
     **Catégorie de VirtualBox** : '''Oracle VM VirtualBox se range dans la catégorie des hyperviseurs de Type 2. Il nécessite un
                                      système d'exploitation hôte pour fonctionner.'''
     **Hyperviseur de Type 1 en Cloud** : ''' **AWS** (Amazon Web Services) utilise majoritairement l'hyperviseur KVM
                                              (intégré au noyau Linux et fonctionnant comme un Type 1)'''

3. **Pourquoi VT-x / AMD-V :**  '''L'activation de VT-x (Intel) ou AMD-V (AMD) dans le BIOS/UEFI est indispensable pour faire tourner une machine virtuelle (VM) 64 bits pour plusieurs raisons techniques :
                                  • L'isolation des privilèges (Ring 0) : En architecture x86/x64, le système d'exploitation doit s'exécuter avec le plus haut niveau de privilège, appelé le Ring 0.
                                  	• Sur un hyperviseur de Type 2, c'est déjà le système d'exploitation de votre ordinateur qui occupe le Ring 0.
                                  	• Sans assistance matérielle, la VM 64 bits ne peut pas s'octroyer ces privilèges indispensables pour gérer sa propre mémoire et ses processus en 64 bits.
                                  • La création d'un "Ring -1" : Les technologies VT-x et AMD-V introduisent un nouveau mode d'exécution CPU (souvent vulgarisé en "Ring -1"). Cela permet à l'hyperviseur de se placer "en dessous" du système d'exploitation virtuel, donnant à la VM l'illusion qu'elle s'exécute directement sur le vrai processeur en Ring 0.
                                  • La limitation du mode 64 bits de base : Contrairement au mode 32 bits, l'architecture 64 bits (x86-64) supprime certaines fonctionnalités de segmentation de mémoire que les anciens hyperviseurs utilisaient pour "tricher" et isoler les VM par logiciel. La virtualisation du 64 bits requiert donc obligatoirement que le processeur physique prenne en charge cette isolation de manière matérielle.
                                  '''
4. **Version installée et résultat de `list hostinfo` :** 
   - **Version installée** : 7.2.20
   - **résultat de `list hostinfo` :**  **Processor core count**: 4,  et **Memory size** : 16227 MByte.

## Parties 2 - Télécharger et verifier l'image ISO ubuntu

### commandes 1 : Calcul 
'''Powershell 

		cd $HOME\Downloads
		Get-FileHash .\ubuntu-26.04-live-server-amd64.iso -Algorithm SHA256
		Select-String -Path .\SHA256SUMS -Pattern "live-server-amd64"
'''
### Sorties commande 1 

Algorithm       Hash                                                                   Path
---------       ----                                                                   ----
SHA256          CC8A95CDE20F6CED61A322420DE00F10CC3C90CED545DAA46CB9C1A117F1D927       C:\Users\sanog\Downloads\ubun...

### La taille du fichier ISO 
La taille est : 2,72 Go (2 927 861 760 octets)
### Questions 
1. Les deux empreintes sont-elles identiques ? Pourquoi vérifier ?
   • '''Oui, les deux empreintes sont strictement identiques à celle du fichier officiel d'Ubuntu 26.04.1.'''
   • '''Pourquoi vérifier ?: C'est une mesure de sécurité et de stabilité essentielle. Cela permet de s'assurer que le fichier
     n'a pas été corrompu (téléchargement coupé ou incomplet) et qu'il n'a pas été modifié par un pirate pour y injecter un virus
     ou un logiciel malveillant avant que vous ne l'installiez.'''

2. Ubuntu Desktop vs Ubuntu Server
  ### Ubuntu Desktop
  
**Interface**: '''Graphique (Bureau, fenêtres, souris)'''
**Logiciels inclus** : Navigateur web, bureautique, lecteur média
**Ressources (RAM/CPU)** : Consomme beaucoup (pour afficher le bureau)
###  Ubuntu Server

**Interface**:Ligne de commande uniquement (Écran noir et texte)
**Logiciels inclus** : Outils serveurs (SSH, bases de données, web)
**Ressources (RAM/CPU)** : '''Très léger et ultra-rapide'''
• Pourquoi choisit-on "Server" pour le Cloud ? Un serveur cloud n'a pas d'écran physique connecté. Installer une interface graphique 
serait un gaspillage inutile de mémoire RAM et de processeur. De plus, moins il y a de logiciels installés, moins il y a de failles 
de sécurité potentielles.

3. Signification de LTS 

• **Signification** : LTS signifie Long Term Support (Support à Long Terme). C'est une version ultra-stable d'Ubuntu, publiée tous les deux ans, destinée aux entreprises et aux serveurs de production.
• **Durée des mises à jour** : Une version LTS reçoit 5 ans de mises à jour de sécurité standard gratuites.

## Partie 3 - Création et configuration de la machine virtuelle

  ### Questions

1. Disque alloué dynamiquement vs Taille fixe & Fichier .vdi

• **La différence** :
	• Un disque alloué dynamiquement commence très petit et grossit sur votre vrai disque dur au fur et à mesure que vous ajoutez des fichiers dans la machine virtuelle (jusqu'à la limite maximale définie).
	• Un disque de taille fixe réserve immédiatement la totalité de l'espace sur votre vrai disque dur (si vous demandez 20 Go, il prend 20 Go dès la première seconde), ce qui prend plus de place mais offre de très légères meilleures performances.
  • Taille du fichier .vdi juste après sa création : Si vous avez choisi l'allocation dynamique, le fichier .vdi ne pèse quasiment rien, seulement quelques mégaoctets (Mo) pour stocker la structure vide du disque

2. Pourquoi désactiver l'installation sans surveillance (Unattended Installation) ?

On la désactive pour ce TP afin de garder le contrôle total sur l'installation de l'OS. 
L'installation sans surveillance laisse VirtualBox configurer Ubuntu automatiquement en arrière-plan avec des paramètres génériques
(langue, nom d'utilisateur, mot de passe par défaut). 
La désactiver force à faire l'installation manuellement, ce qui est le but pédagogique de votre TP pour apprendre à configurer un serveur
(choix de la langue, du clavier, création de votre propre utilisateur).

3. Mode NAT et Redirection de ports (2222 → 22)

• Pourquoi l'hôte ne peut pas joindre la VM en NAT ? :  '''En mode NAT, VirtualBox crée un réseau privé virtuel et isole votre machine virtuelle
derrière un routeur virtuel (comme une box internet). 
La VM peut sortir pour aller sur Internet, mais votre vrai PC (l'hôte) se trouve "à l'extérieur" de ce réseau et ne connaît pas l'adresse IP
privée de la VM. Il est bloqué par le pare-feu du routeur virtuel.'''
• À quoi sert la redirection 2222 → 22 ? : ''' Elle sert à ouvrir une passerelle à travers ce routeur. 
Le port 22 est le port standard du protocole SSH (qui permet de contrôler le serveur à distance en ligne de commande). 
Comme le port 22 de votre vrai PC est peut-être déjà utilisé, on dit à VirtualBox : "Si j'envoie une connexion SSH sur le port 2222 de mon vrai PC (localhost), redirige-la automatiquement vers le port 22 de ma machine virtuelle Ubuntu".
'''
## Parties 4 - Installation de Ubuntu Server
    ### Questions
      
1. Durée de l'installation : VirtualBox vs Instance Cloud

• **Durée sur VirtualBox** : Une installation classique d'Ubuntu Server sur VirtualBox prend généralement entre 10 et 20 minutes. Ce temps comprend le téléchargement/décompressage des paquets, la configuration manuelle (langue, clavier, partitionnement) et l'installation finale.
• **Comparaison avec une instance Cloud (image prête)** : La création d'une instance dans le cloud (sur AWS, GCP ou OpenStack) à partir d'une image prête prend seulement entre 30 secondes et 2 minutes.
• **Pourquoi cette différence ?** : L'image cloud est "pré-installée". Le fournisseur de cloud ne réinstalle pas le système de zéro ; il duplique simplement un disque virtuel déjà prêt (un template) et l'associe à votre instance.

2. Adresse IP sur enp0s3 et attribution

• **Adresse IP obtenue** :  l'interface enp0s3 a obtenu l'adresse IP 10.0.2.15/24.
• **Qui l'a attribuée ?** :  C'est le serveur DHCP virtuel intégré de VirtualBox qui a attribué cette adresse. Par défaut, VirtualBox configure le réseau de la machine en mode NAT. Dans ce mode, VirtualBox agit comme un routeur domestique et distribue toujours la première adresse disponible (souvent 10.0.2.15) aux machines virtuelles.

3. Utilité du serveur OpenSSH

• **Pourquoi l'installer dès le départ ?**  : On installe OpenSSH immédiatement car un serveur n'a pas vocation à posséder un écran, un clavier ou une interface graphique (comme c'est le cas ici avec votre Ubuntu Server textuel). SSH permet aux administrateurs de se connecter à distance et de manière sécurisée pour gérer le serveur depuis leur propre ordinateur via un terminal (comme PuTTY, PowerShell ou le terminal macOS).
• **Que se passerait-il sur un serveur cloud sans SSH ?** :  Si une instance cloud démarrait sans serveur SSH activé, le serveur deviendrait totalement inaccessible. N'ayant pas d'accès physique à la machine (puisqu'elle est dans un centre de données distant), vous ne pourriez plus du tout vous y connecter pour la configurer ou y installer des logiciels.

## Parties 5 - Premiers pas, réseau et accès SSH 

  1. Identification du système

     ### Commande 1 :
                     ''' Ubuntu
                          lsb_release -a
                          uname -r
                          hostnamectl
                          nproc
                          free -h
                          df -h /
                          lsblk
                       '''
     ### Captures sorties de la commande 1

     
         
       <img width="757" height="582" alt="etape1" src="https://github.com/user-attachments/assets/d43e5976-8017-4380-b198-4852c0a0a8da" />


        <img width="1081" height="567" alt="etape2" src="https://github.com/user-attachments/assets/35be2fa2-dedc-48e6-b7d7-f5db144bf0ba" />

        
    2.	Observer la configuration réseau et tester l'accès à Internet 

       ### Commande 2 :
                   ''' Ubuntu
                   ip -4 addr show
                    ip route
                    resolvectl status | grep "DNS Servers"
                    ping -c 3 ubuntu.com
                    sudo apt update && sudo apt upgrade -y
                    '''
        
        ### Capture de la sortie de la commande 2

            
  <img width="937" height="198" alt="etape_3" src="https://github.com/user-attachments/assets/281740d0-c884-4504-9d93-a9130d55e234" />


  <img width="1270" height="537" alt="etape_4" src="https://github.com/user-attachments/assets/510bbe61-149d-40be-8101-bd42dc334b08" />


  <img width="1269" height="155" alt="etape_5" src="https://github.com/user-attachments/assets/ab3e17bb-29dd-4ded-8e08-ede19bfce81c" />


    3.	Vérifier que le serveur SSH écoute 
          ### Commande 3 : 
                        ''' Ubuntu
                        systemctl status ssh --no-pager
                        sudo ss -tlnp | grep :22
                        '''
  4.	Depuis PowerShell sur l'hôte, se connecter à la VM par la redirection de port, puis copier un fichier
     
     ### Commande 4 :
    	            '''Powershell
                       ssh -p 2222 etudiant@127.0.0.1
                  # accepter l'empreinte du serveur (yes), saisir le mot de passe, puis : exit
                  "Bonjour depuis l'hôte" | Out-File -Encoding utf8 bonjour.txt
                  scp -P 2222 .\bonjour.txt etudiant@127.0.0.1:~
                  ssh -p 2222 etudiant@127.0.0.1 "cat ~/bonjour.txt"
    	            '''
    	### Captures de la sortie de la  commande 4 :

  <img width="1295" height="642" alt="etape7" src="https://github.com/user-attachments/assets/f46e0a7f-3ea9-484b-9169-363a23235ce6" />
      
          
  <img width="1211" height="240" alt="etape8" src="https://github.com/user-attachments/assets/35b54ba2-edc6-454a-9395-bf6df8df2307" />
      
      ### Questions

      
          1. Paramètres réseau de la VM et rôle de la passerelle
          
          • **Adresse IP & Masque** : 10.0.2.15/24 (Masque /24 ou 255.255.255.0).
          • **Passerelle (Gateway)** : 10.0.2.2
          • **Serveur DNS** : 10.0.2.3 (par défaut sous VirtualBox NAT) ou l'IP de votre box internet transmise par l'hôte.
          • **À quoi correspond la passerelle 10.0.2.2 ?** :  Elle correspond à la carte réseau virtuelle de VirtualBox qui fait office de routeur. C'est la porte de sortie qui permet à la machine virtuelle de traduire ses requêtes vers la vraie machine Windows (hôte) afin d'accéder au réseau extérieur et à Internet.

    	    
          2. Connexion à 127.0.0.1:2222 plutôt qu'à 10.0.2.15:22

            • L'adresse 10.0.2.15 est une IP privée interne au réseau NAT isolé de VirtualBox. Le système Windows hôte ne peut pas l'atteindre directement, car elle n'existe pas sur le réseau physique de l' ordinateur.
            • En se connectant à 127.0.0.1:2222 (votre propre PC Windows), On utilise la redirection de port (Port Forwarding) configurée dans VirtualBox. VirtualBox intercepte tout ce qui arrive sur le port 2222 de  Windows et le renvoie automatiquement vers le port 22 (SSH) de la VM.


           
          3. Contenu et rôle de l'empreinte SSH (Fingerprint)
          
          • Que contient ce message ? Il contient une clé publique d'identification unique (empreinte cryptographique) générée par le serveur SSH d'Ubuntu lors de sa toute première installation (généralement un condensé de type SHA256).
          • Quel risque permet-il d'éviter ? Il permet d'éviter l'attaque de l'Homme du milieu (Man-in-the-middle). L'empreinte garantit qu'on se connecte bien au serveur légitime. Si un pirate tentait de détourner la connexion pour intercepter les identifiants, l'empreinte changerait et SSH bloquerait immédiatement la connexion en affichant une alerte de sécurité critique.
    
          
          4. Avantages de l'authentification par clé vs Mot de passe
          
          • **Les avantages** :
          	• Sécurité absolue : Une clé cryptographique (comme ed25519) est impossible à deviner ou à pirater par force brute (contrairement à un mot de passe classique souvent trop court ou prévisible).
          	• Confort et automatisation : Elle permet de se connecter instantanément sans aucune action humaine, ce qui est indispensable pour les scripts automatisés de déploiement.
          • Pourquoi les fournisseurs cloud l'imposent par défaut ? Les serveurs Cloud ont une IP publique visible par le monde entier dès leur création. S'ils utilisaient des mots de passe par défaut (comme root/password), des robots malveillants scanneraient et pirateraient ces serveurs en quelques secondes. Imposer une clé SSH dès le premier démarrage garantit qu'un seul et unique administrateur légitime possède le sésame pour entrer.



  ## Parties 6 -Instantané , clone et arrêt

    1-prendre un instantané de VM propre 

      ###commande 1 : 
                  ''' Powershell
                      VBoxManage snapshot ubuntu-srv take apres-installation --description "Ubuntu installé, OpenSSH actif"
                      VBoxManage snapshot ubuntu-srv list
                  '''


        ### Capture de la sortie de la commande 1 
        
  <img width="924" height="126" alt="etape_9" src="https://github.com/user-attachments/assets/812264a9-77a3-477f-8b55-cbb601112c74" />
  2-Dans la VM (par SSH) , faire une modification visible , puis éteindre 
    ### commande 2 :

          '''Ubuntu
              sudo apt install -y htop
              touch ~/avant-restauration.txt
              sudo poweroff
          '''
    ### Capture de la sorties de commande 2

<img width="1214" height="310" alt="etape11" src="https://github.com/user-attachments/assets/d0a26980-74c4-4440-b081-03330d2bfddb" />

3.	Restaurer l'instantané, redémarrer sans fenêtre et vérifier que htop et le fichier ont disparu :
   
   ### Commande 3 :
               '''Powershell
                 VBoxManage snapshot ubuntu-srv restore apres-installation
                  VBoxManage startvm ubuntu-srv --type headless
                  ssh -p 2222 etudiant@127.0.0.1 "which htop; ls ~"
              '''

    ### Sortie de la commande 3 

  <img width="777" height="192" alt="etape12" src="https://github.com/user-attachments/assets/c8cbf33e-6086-4489-8af8-f5b24321c3f0" />

4.	Éteindre proprement, puis cloner la VM. Le clone doit utiliser un autre port hôte (2223)
   ### Commande 4 :
               '''Powershelle
   
                  VBoxManage controlvm ubuntu-srv acpipowerbutton
                  VBoxManage clonevm ubuntu-srv --name ubuntu-clone --register
                  VBoxManage modifyvm ubuntu-clone --natpf1 delete ssh
                  VBoxManage modifyvm ubuntu-clone --natpf1 "ssh,tcp,127.0.0.1,2223,,22"
                  VBoxManage startvm ubuntu-clone --type headless
                  ssh -p 2223 etudiant@127.0.0.1 "sudo hostnamectl set-hostname ubuntu-clone; hostname"
                  VBoxManage list runningvms
              '''
  ### Capture de la sortie de la commande 4 
  
  <img width="1278" height="289" alt="etape13" src="https://github.com/user-attachments/assets/41f668f9-5f3a-48bf-8b25-f3309a685816" />

5.	Comparer la place occupée sur l'hôte, puis supprimer le clone. Garder ubuntu-srv pour les TP suivants.
   ### commande 5 :
           '''Powershell
           
              Get-ChildItem "$HOME\VirtualBox VMs" -Directory | ForEach-Object { "{0} : {1:N2} Go" -f $_.Name, ((Get-ChildItem $_.FullName -Recurse -File | Measure-Object Length -Sum).Sum / 1GB) }
              VBoxManage controlvm ubuntu-clone acpipowerbutton
              VBoxManage unregistervm ubuntu-clone --delete
          '''
  ### Capture de la sortie de la commande 5 

  
  <img width="1172" height="161" alt="etape14" src="https://github.com/user-attachments/assets/3da3f1db-cb24-41b9-b5d2-573608e0f534" />


  
1. Instantané (Snapshot) vs Clone, et notion de sauvegarde

• Différence principale :
	• **Un instantané** (snapshot) capture l'état exact d'une machine virtuelle à un instant T (fichiers, configuration et parfois l'état de la mémoire RAM si elle est allumée). Il reste dépendant de la VM d'origine.
	• **Un clone** est une copie intégrale et indépendante de la machine virtuelle à partir d'un état donné. Il génère un nouveau fichier de disque virtuel (VDI) autonome.
• **Un instantané est-il une sauvegarde ?**: Non, absolument pas. Si le disque virtuel principal de votre VM est corrompu ou supprimé, l'instantané devient totalement inutilisable puisqu'il ne contient que les modifications apportées depuis sa création (principe du disque différentiel). Une vraie sauvegarde doit être indépendante de la source.

2. Équivalents dans le Cloud

• **Équivalent de l'instantané** : Le Snapshot de volume (ou de disque). Dans le cloud (comme AWS EBS Snapshot), il permet de figer l'état d'un stockage à un moment précis pour pouvoir y revenir ou restaurer le disque en cas de mauvaise manipulation.
• **Équivalent du clone** : L'Image personnalisée (Custom Image / AMI). On fige une instance configurée, on en extrait une image globale (système + applications) qui servira de modèle (template) pour déployer instantanément des dizaines de nouvelles instances identiques.

3. Différence entre acpipowerbutton et poweroff dans VBoxManage

• **acpipowerbutton** : Envoie un signal électrique virtuel équivalent à un appui court sur le bouton physique d'un ordinateur. Il demande poliment au système d'exploitation de s'éteindre proprement en fermant ses services et ses fichiers (comme un sudo poweroff ou shutdown).
• **poweroff** : Coupe brutalement le courant virtuel de la machine (équivalent à débrancher la prise électrique). C'est instantané, mais cela comporte un risque élevé de corruption des fichiers et du système de fichiers de votre Ubuntu.



4. Mode Headless (sans interface) et lien avec le Cloud

• **Pourquoi démarrer en mode headless ?** Ce mode lance la machine virtuelle en arrière-plan sans ouvrir la fenêtre d'affichage graphique de VirtualBox. Cela économise énormément de ressources graphiques (GPU) et de mémoire vive (RAM) sur votre machine Windows hôte.
• **Lien avec une instance Cloud** : C'est le fonctionnement natif et unique du Cloud. Dans un centre de données (AWS, Azure, etc.), les serveurs n'ont aucun écran ni interface physique. Ils tournent tous en mode headless et les administrateurs interagissent avec eux exclusivement à distance via le réseau (en SSH).

5. Pourquoi changer le port hôte et le nom d'hôte du clone ?

• **Changer le port hôte (redirection de port)** : Si le clone garde le port 2222, VirtualBox refusera de démarrer le réseau car deux machines ne peuvent pas écouter sur le même port de votre machine Windows (conflit d'adresse). Il faut attribuer un nouveau port au clone (par exemple 2223).
• **Changer le nom d'hôte (hostname)** : Le nom d'hôte identifie la machine sur le réseau. Si deux serveurs ont le même nom (ex: ubuntu-srv), cela crée des conflits d'identité dans les fichiers de logs, les scripts d'automatisation, et complique grandement la tâche de l'administrateur pour savoir sur quelle machine il est connecté.

      

    
          

        























