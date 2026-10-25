
# TP3 : Premiers conteneurs avec Docker

## Partie 0 : Preparation de la VM et Installer DOcker 

### 1-Demarage de ubuntu-srv

**commande**

''' Powershell

            $env:Path += ";C:\Program Files\Oracle\VirtualBox"
            VBoxManage startvm ubuntu-srv --type headless
            VBoxManage controlvm ubuntu-srv natpf1 "web,tcp,127.0.0.1,8080,,8080"
            VBoxManage controlvm ubuntu-srv natpf1 "api,tcp,127.0.0.1,5000,,5000"
            ssh -p 2222 etudiant@127.0.0.1

'''

**sortie**
<img width="1340" height="683" alt="image" src="https://github.com/user-attachments/assets/a5598720-3bee-4d94-8474-752ce0a71664" />

### 2-Installation de docker engine 


**commande**

''' Powershell

        sudo apt update
        sudo apt install -y docker.io docker-buildx docker-compose-v2
        sudo usermod -aG docker $USER
        exit
'''

**sortie**

<img width="1340" height="688" alt="image" src="https://github.com/user-attachments/assets/e042dd2e-c585-4295-94a7-f71c30ea5aab" />



### 3- Reconnexion pour le groupe Docker 

**commande**

''' Powershell

  docker version
  docker info | grep -E "Server Version|Storage Driver|Cgroup|Kernel Version"

'''

**sortie**

<img width="1057" height="663" alt="image" src="https://github.com/user-attachments/assets/3c5b411d-6de2-42b6-9386-d3dc6decd1f3" />


###Questions



**Q1. Identification de la version et du noyau**

• Version de Docker :29.1.3
• Version du noyau (Kernel) : 7.0.0-30generic

**Q2. Les risques de sécurité liés au groupe docker**

• Pourquoi cela revient à être root (administrateur) :
Le démon Docker (dockerd) s'exécute avec les privilèges de l'administrateur suprême (root). Lorsqu'un utilisateur est ajouté au groupe docker, il acquiert le droit de lancer des conteneurs sans taper sudo.
Un utilisateur malveillant ou imprudent peut alors lancer un conteneur en lui donnant un accès total au disque dur réel du serveur (en montant la racine / de l'hôte dans le conteneur via un volume, ex: docker run -v /:/host ...). Une fois dans le conteneur, il peut modifier n'importe quel fichier du serveur hôte, créer un nouvel utilisateur administrateur, ou lire les mots de passe de la machine. Le contrôle est alors total.
• Précaution à imposer sur un serveur partagé :
Sur un serveur utilisé par plusieurs personnes (étudiants, développeurs, collègues) :
1. Restreindre l'accès : On ne doit ajouter dans le groupe docker uniquement les personnes de confiance absolue (les administrateurs du système).
2. Utiliser le mode "Rootless" : Si des utilisateurs ordinaires ont absolument besoin de Docker, il faut configurer Docker en mode Rootless (sans privilèges root), ce qui isole complètement leurs conteneurs et les empêche de nuire au serveur principal.
3. Surveillance (Audit) : Activer des journaux de logs stricts pour surveiller l'historique des commandes Docker exécutées sur la machine.


## Partie 1 : Lançage et inspection des conteneurs 

**1.1 Premier conteneur**

**commande**

''' console 

      docker run hello-world
      docker images
      docker ps -a

'''

**sortie**

<img width="1150" height="309" alt="image" src="https://github.com/user-attachments/assets/67f60433-f103-41c3-b089-256e1c67d053" />


**1.2 Un serveur web une demande**

**commande**

''' console 

      docker run -d --name web -p 8080:80 nginx
      docker ps
      curl -s http://localhost:8080 | head -n 5
'''

**sortie**


<img width="1229" height="378" alt="image" src="https://github.com/user-attachments/assets/f7cf0594-a4d7-4783-a4da-bf2082344914" />


**1.3 Regarder à l'interieur**

**commande**

''' console 

      docker exec web cat /etc/os-release | head -n 2
      cat /etc/os-release | head -n 2
      docker exec web uname -r
      uname -r

'''

**sortie**

<img width="981" height="302" alt="image" src="https://github.com/user-attachments/assets/bf0b1427-c15b-4142-9e4d-71e4daa2c531" />

**commande**

''' console 

    docker logs --tail 5 web
    docker top web
    ps aux | grep "nginx: master" | grep -v grep
    docker stats --no-stream web
    

'''

**sortie**

<img width="1342" height="357" alt="image" src="https://github.com/user-attachments/assets/b6c0384b-d6ee-4cb5-8b04-ce99663db235" />


**1.4 un conteneur est ephemère**

**commande**

''' console 


      docker exec web sh -c 'echo "<h1>Binôme NOM1 - NOM2</h1>" > /usr/share/nginx/html/index.html'
      curl -s http://localhost:8080
      docker rm -f web
      docker run -d --name web -p 8080:80 nginx
      curl -s http://localhost:8080 | grep -o "<title>.*</title>"

'''

**sortie**

<img width="1280" height="387" alt="image" src="https://github.com/user-attachments/assets/172aaeec-a8af-4806-a362-ad1372e64321" />


**commande**

''' console 


      mkdir -p ~/site
      echo "<h1>Binôme NOM1 - NOM2 : servi depuis un volume</h1>" > ~/site/index.html
      docker rm -f web
      docker run -d --name web -p 8080:80 -v ~/site:/usr/share/nginx/html:ro nginx
      curl -s http://localhost:8080


'''


**sortie**

<img width="1123" height="261" alt="image" src="https://github.com/user-attachments/assets/cb8e56b1-bd5d-461b-9561-7f9dbfb92dfe" />


### Questions


**Q1. Les quatre étapes réalisées par Docker (Message de hello-world)**

D'après le message officiel du conteneur hello-world, le client Docker effectue les étapes suivantes lorsqu'on lance une commande docker run :
1. Le client Docker contacte le démon Docker (le service d'arrière-plan).
2. Le démon cherche l'image localement. Ne la trouvant pas, il la télécharge (pull) depuis le registre public Docker Hub.
3. Le démon crée un nouveau conteneur à partir de cette image et l'exécute, ce qui lance le script ou le programme interne.
4. Le démon redirige la sortie (le texte généré par le programme) vers le client Docker, qui l'affiche directement sur votre terminal.

**Q2. Distribution et noyau (Conteneur vs VM)**

• La distribution : Elle est différente. Votre VM tourne sous Ubuntu, alors que le conteneur Nginx officiel est généralement basé sur une distribution plus légère comme Debian ou Alpine (visible avec la commande cat /etc/os-release). Un conteneur embarque uniquement les fichiers et outils spécifiques de sa propre distribution.
• Le noyau : Il est identique. Les deux commandes uname -r renvoient exactement la même version.
• Explication : Contrairement à une machine virtuelle (VirtualBox) qui émule un matériel et lance un système d'exploitation complet avec son propre noyau, un conteneur partage le noyau Linux de la machine hôte (votre VM). Il isole simplement son environnement applicatif.

**Q3. Visibilité des processus et nature d'un conteneur**

• Observation : Oui, les processus de Nginx exécutés à l'intérieur du conteneur apparaissent bien dans la commande ps aux lancée depuis la VM.
• Ce que cela montre : Cela prouve qu'un conteneur n'est pas une machine virtuelle, mais un simple groupe de processus isolés s'exécutant directement sur le système de la VM. Docker utilise les fonctionnalités du noyau Linux (namespaces pour l'isolation et cgroups pour la limitation des ressources) pour faire croire au conteneur qu'il est seul au monde, alors qu'il s'agit de processus standards vus de l'extérieur.

**Q4. Chemin d'une requête réseau (Navigateur hôte ➔ Nginx)**

Pour atteindre la page web, votre requête traverse deux niveaux de redirections de ports :
1. Niveau 1 (VirtualBox) : Le navigateur de votre PC Windows (Hôte) tape sur http://127.0.0.1:8080. Grâce à la redirection de port NAT configurée dans VirtualBox au cours de l'étape précédente, la requête quitte Windows et entre dans votre VM ubuntu-srv sur son port 8080.
2. Niveau 2 (Docker) : Sur la VM, Docker écoute sur ce port 8080 (grâce à l'option -p 8080:80). Il attrape la requête et la redirige vers le port interne 80 du conteneur où le serveur Nginx est en attente.

**Q5. Disparition des données et rôle de l'option -v (Volume)**

• Pourquoi la page a disparu : Par nature, un conteneur est éphémère. Les modifications faites avec docker exec sont écrites dans une couche temporaire (la Writable Layer). Lorsque vous exécutez docker rm -f web, le conteneur et cette couche temporaire sont définitivement détruits. Recréer un conteneur relance l'image Nginx d'origine, à son état initial.
• Ce que change l'option -v (Volume) : L'option -v ~/site:/usr/share/nginx/html:ro crée un pont persistant (un montage). Elle demande à Docker de lier le dossier réel de votre VM (~/site) à l'intérieur du conteneur. Les fichiers web ne sont plus stockés dans le conteneur mais sur le disque dur de votre VM. Si le conteneur est supprimé ou recréé, les données restent intactes sur la VM et le nouveau conteneur les réutilisera immédiatement. L'option :ro (read-only) garantit par sécurité que le conteneur peut lire les fichiers mais n'a pas le droit de les modifier.


## Partie 2 : comparer conteneur et VM 

**1 Mesurer la taille**

**commande**

''' console 

      docker pull alpine
      docker images
      time docker run --rm alpine echo "Bonjour depuis un conteneur"
      time docker run --rm alpine echo "Bonjour depuis un conteneur"
      
'''

**sortie**

<img width="1156" height="548" alt="image" src="https://github.com/user-attachments/assets/ab6725f6-ea24-4eb6-bbf5-295fdff57139" />


**2 Mesurer le demarage**

''' console 

      systemd-analyze
      free -h
'''
**sortie**

<img width="898" height="154" alt="image" src="https://github.com/user-attachments/assets/7c499ee7-512c-4ea0-9787-4497c01e3632" />





Mesure	VM ubuntu-srv	Conteneur nginx	Conteneur alpine
Taille sur disque	~2 à 4 Go (Dossier VirtualBox)	~140 à 190 Mo	~5 à 8 Mo (Ultra léger)
Temps de démarrage	~15 à 30 secondes	~0,5 seconde (Instantané)	~0,1 à 0,3 seconde
Mémoire utilisée	~150 à 300 Mo (Système complet)	~5 à 10 Mo	~1 à 2 Mo (Uniquement le echo)
Système	Ubuntu complet + noyau	Debian (ou Alpine) partiel	Alpine Linux minimal


### Questions



**Q1. Rapport de légèreté, de rapidité et validation du cours**

• Rapport de taille sur disque : Le conteneur Alpine (~7 Mo) est environ 500 fois plus léger que la VM Ubuntu (~3,5 Go). Même un conteneur d'application complet comme Nginx (~150 Mo) reste environ 20 fois plus léger.
• Rapport de temps de démarrage : La VM met environ 20 secondes à démarrer là où le conteneur met une fraction de seconde (ex: 0.2s). Le conteneur est donc environ 100 fois plus rapide à démarrer.
• Validation : Oui , elles prouvent que le conteneur élimine la lourdeur de l'émulation matérielle et le chargement d'un système d'exploitation complet (pas de phase de boot, pas de noyau à charger), ce qui le rend drastiquement plus agile.


**Q2. Pourquoi le premier time docker run est plus long ?**

Le premier lancement est plus long car Docker doit effectuer des opérations d'initialisation en arrière-plan :
1. Création et allocation : Le démon Docker doit instancier le conteneur, configurer l'espace de noms (namespace), monter le système de fichiers en lecture-écriture et allouer une interface réseau virtuelle pour la première fois.
2. Mise en cache : Lors du deuxième lancement, le sous-système de Docker et le noyau Linux ont déjà mis en cache mémoire (RAM) les fichiers de l'image Alpine et les structures réseau nécessaires. L'exécution est alors purement instantanée.


**Q3. Cas où l'on choisit une VM plutôt qu'un conteneur**

On privilégiera une Machine Virtuelle (VirtualBox, VMware, AWS EC2) plutôt qu'un conteneur Docker dans les scénarios suivants :
1. Besoin d'un noyau ou d'un OS différent : Si votre machine hôte tourne sous Linux mais que vous devez impérativement exécuter une application qui requiert un noyau Windows Server ou macOS. Un conteneur Linux ne peut pas tourner directement sur un noyau Windows (sans couche de virtualisation intermédiaire).
2. Isolation de sécurité stricte (Multi-tenant) : Dans le Cloud, si vous devez exécuter du code inconnu ou potentiellement dangereux provenant de clients différents. Comme les conteneurs partagent le même noyau Linux, une faille de sécurité critique du noyau (Kernel exploit) permettrait de s'échapper du conteneur et d'attaquer l'hôte. Une VM offre une barrière étanche grâce à l'isolation matérielle complète de l'hyperviseur.
3. Persistance et services lourds : Pour gérer des infrastructures complexes nécessitant un contrôle total du démarrage du système (via systemd), des pilotes de périphériques spécifiques ou des configurations de stockage matériel très avancées.


## Partie 3 : Conteneuriser l'API fil rouge 

**1 crztion du dossier du projet**

**commande**

''' console 

        mkdir -p ~/api && cd ~/api
        cat > app.py <<'EOF'
        import os
        import socket
        from flask import Flask, jsonify
         
        app = Flask(__name__)
         
        @app.get("/")
        def accueil():
            return jsonify(
                message="API fil rouge",
                binome=os.environ.get("BINOME", "inconnu"),
                conteneur=socket.gethostname(),
            )
         
        @app.get("/sante")
        def sante():
            return jsonify(statut="ok")
         
        if __name__ == "__main__":
            app.run(host="0.0.0.0", port=5000)
        EOF
        echo "flask>=3,<4" > requirements.txt
'''

**2 Ecriture de dockerfile**

**commande**

'''  console 

    cat > Dockerfile <<'EOF'
    FROM python:3.12-slim
    WORKDIR /app
    COPY requirements.txt .
    RUN pip install --no-cache-dir -r requirements.txt
    COPY . .
    EXPOSE 5000
    CMD ["python", "app.py"]
    EOF
    printf '__pycache__/\n*.pyc\n.git\n' > .dockerignore

'''
**3 construction de l'image**

**commande**

'''  console 

      docker build -t api:1.0 .
      docker images api
      docker history api:1.0
'''

**sortie**

<img width="1344" height="508" alt="image" src="https://github.com/user-attachments/assets/dd5b76b2-7bc6-4789-b5da-9cf158a77e89" />

<img width="1321" height="372" alt="image" src="https://github.com/user-attachments/assets/3e54c3e4-cc37-420c-b122-c3c1cc7f02d5" />


**4 Lancer l'API**


**commande**


''' console 

      docker run -d --name api -p 5000:5000 -e BINOME="NOM1-NOM2" api:1.0
      curl -s http://localhost:5000
      curl -s http://localhost:5000/sante
      docker logs api


'''

**sortie**


<img width="1038" height="255" alt="image" src="https://github.com/user-attachments/assets/91828d9f-b6ad-465f-b8a4-000dbba22bd7" />


**5 lancer trois copies de la même image**

**commande**

''' console 

      for p in 5001 5002 5003; do docker run -d --name api-$p -p $p:5000 -e BINOME="NOM1-NOM2" api:1.0; done
      for p in 5001 5002 5003; do curl -s http://localhost:$p; echo; done
      docker rm -f api-5001 api-5002 api-5003

'''

**sortie**

<img width="1182" height="308" alt="image" src="https://github.com/user-attachments/assets/48e8711e-1d0e-44ff-b718-09472141b499" />


**6 Ajouter une route**

**commande**

''' console 

      docker build -t api:1.1 .
      docker rm -f api && docker run -d --name api -p 5000:5000 api:1.1
      curl -s http://localhost:5000/version

'''

**sortie**

<img width="1346" height="141" alt="image" src="https://github.com/user-attachments/assets/0e5c225b-b0ba-4e10-a31d-5dce43fcd351" />


### Questions



**Q1. Analyse de la taille et des couches de l'image (api:1.0)**

• Taille de l'image : La commande docker images api affiche généralement une taille d'environ 140 Mo à 160 Mo. C'est une taille très optimisée grâce à l'utilisation de la version de base python:3.12-slim (qui n'embarque que le strict minimum pour exécuter du Python).
• Nombre de couches : La commande docker history api:1.0 liste l'intégralité des couches. Le nombre exact dépend de la version de l'image de base, mais les 5 dernières couches du haut proviennent directement de votre Dockerfile :
	1. La couche du WORKDIR /app (création du dossier).
	2. La couche du COPY requirements.txt ..
	3. La couche du RUN pip install... (contenant la bibliothèque Flask).
	4. La couche du COPY . . (contenant vos fichiers de code app.py).
	5. La couche finale de configuration CMD ["python", "app.py"].
(Toutes les couches situées en dessous proviennent de l'image parente python:3.12-slim).

**Q2. Intérêt de l'ordre des instructions et mécanisme de cache (CACHED)**

• Pourquoi cet ordre ? On sépare la copie des dépendances (requirements.txt) de celle du code source pour optimiser le temps de reconstruction en exploitant le système de cache de Docker.
• Explication avec l'étape 6 : Docker construit ses images couche par couche. Si une étape ne change pas, Docker réutilise la couche déjà prête sans réexécuter la commande (ce sont les lignes marqués CACHED).
	• Si vous modifiez uniquement une ligne de code dans app.py, le fichier requirements.txt reste identique.
	• Docker s'en aperçoit et réutilise le cache pour l'étape RUN pip install.... Vous vous évitez ainsi de retélécharger et réinstaller Flask à chaque fois que vous corrigez une ligne de code. L'étape de build ne prend alors qu'une fraction de seconde.

**Q3. Analyse des trois copies (Étape 5) et concept du Cloud**

• Retour du champ conteneur : Pour les trois copies (api-5001, api-5002 et api-5003), la valeur du champ conteneur renvoie trois identifiants (hostnames) totalement différents.
• Caractéristique du Cloud illustrée : Cette expérience illustre le concept d'Élasticité (ou de Passage à l'échelle horizontal / Horizontal Scaling). Dans le Cloud, si une application subit une forte hausse de trafic, on ne modifie pas le code et on ne reconstruit pas l'image : on déploie instantanément plusieurs copies (instances) de la même image de base de manière identique et isolée pour répartir la charge réseau.

**Q4. Rôle des variables d'environnement et gestion des secrets**

• Pourquoi utiliser -e plutôt que de figer la valeur dans le code ? Cela permet de dissocier complètement la configuration (les données variables) du code source (la logique applicative). En faisant cela, votre image Docker devient universelle et réutilisable. La même image api:1.0 peut être déployée pour le Binôme A, le Binôme B, ou en production sans jamais avoir à modifier un seul fichier Python.
• Lien avec les secrets (mots de passe, clés d'API) : C'est une règle de sécurité fondamentale. On ne doit jamais écrire un mot de passe, un identifiant ou une clé de base de données directement dans le code source (ce qu'on appelle le hardcoding). Si le code est partagé sur un dépôt public comme GitHub, n'importe qui peut voler vos accès. En utilisant des variables d'environnement, l'administrateur système ou la plateforme Cloud injecte les clés secrètes de manière sécurisée uniquement au moment où le conteneur démarre.



## Parti4 : 

**commande**

''' console 

    docker login ghcr.io -u <compte>                # mot de passe = jeton GitHub avec le droit write:packages
    docker tag api:1.1 ghcr.io/<compte>/api-fil-rouge:1.1
    docker push ghcr.io/<compte>/api-fil-rouge:1.1
    
'''


**sortie**

<img width="1346" height="187" alt="image" src="https://github.com/user-attachments/assets/21024279-8107-49be-9cbf-abc6eb439bd9" />


**1 Verification de l'image**

'''  console 

      docker rm -f api
      docker rmi <compte>/api-fil-rouge:1.1 api:1.1
      docker run -d --name api -p 5000:5000 <compte>/api-fil-rouge:1.1      # préfixer par ghcr.io/ pour l'option B
      curl -s http://localhost:5000/version

'''

**sortie**

<img width="1003" height="211" alt="image" src="https://github.com/user-attachments/assets/5b8d6229-c779-47d3-95e4-caf84b0bbee6" />

**2 Nettoyage**

**commande**


''' console 

      docker rm -f $(docker ps -aq)
      docker system df
      docker image prune -a -f
      docker system df

''''

**sortie**


<img width="976" height="350" alt="image" src="https://github.com/user-attachments/assets/f0a3ad84-6569-4fa6-86d9-d931b98092ce" />



### Questions 



Q1. Adresse complète de l'image et visibilité

• Adresse complète : L'adresse dépend de l'option choisie lors du TP :
 Option B (GitHub Packages) : ghcr.io/<votre_compte_github>/api-fil-rouge:1.1.
• Visibilité  :Publique
Q2. Pourquoi étiqueter les versions plutôt que d'utiliser toujours latest ?

Utiliser un numéro de version strict (comme 1.1) à la place du tag automatique latest est une bonne pratique essentielle pour les raisons suivantes :
1. Traçabilité et Reproductibilité : Le tag latest n'est qu'un pointeur mouvant vers la toute dernière image poussée sur le registre. Si vous déployez une application avec latest, vous ne pouvez jamais garantir quelle version exacte du code s'exécute. Spécifier 1.1 garantit que l'environnement reste strictement identique à chaque déploiement.
2. Gestion des retours en arrière (Rollback) : Si votre mise à jour comporte un bug critique en production, avoir des images étiquetées explicitement (1.0, 1.1) permet de revenir instantanément à la version précédente stable en changeant simplement le tag dans votre commande de déploiement. Avec latest, l'ancienne image stable aurait été écrasée sur le registre.
3. Éviter les erreurs de cache : Par défaut, si l'hôte possède déjà une image locale marquée latest, Docker peut décider de ne pas aller chercher une nouvelle version sur le serveur distant lors d'un docker run.

Q3. Nettoyage de l'espace disque et gestion des registres Cloud

• Espace disque libéré : La commande docker image prune -a -f supprime toutes les images locales inutilisées (qui ne sont rattachées à aucun conteneur actif ou arrêté). L'espace exact libéré correspond à la différence affichée par la commande docker system df entre le premier et le deuxième appel. Dans votre cas, cela a supprimé les versions intermédiaires ou les images parentes comme python:3.12-slim et nginx si leurs conteneurs de test associés avaient bien été supprimés auparavant.
• Pourquoi nettoyer les registres dans le Cloud ?
Dans un environnement de Cloud public réel (comme AWS ECR, Azure ACR ou Google Artifact Registry), le stockage des images sur un registre distant n'est pas gratuit : le fournisseur facturera l'espace de stockage consommé au Go/mois.
Étant donné que les pipelines d'intégration continue (CI/CD) créent et poussent automatiquement une nouvelle image Docker à chaque fois qu'un développeur modifie une ligne de code, le registre peut rapidement accumuler des centaines de gigaoctets d'anciennes images obsolètes. Mettre en place des politiques de nettoyage automatique sur le registre évite une explosion inutile de la facture Cloud.














