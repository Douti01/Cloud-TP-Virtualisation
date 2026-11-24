# Compte rendu TP3 cloud seance 4
## theme: Une application multi-services avec Docker Compose
## date : 23/11/2026
## Nom encacdreur : DR BAKAYOKO
## Nom etudiant : SANOGO DOUTI AMADOU 

## 1.OBJECTIFS
  
  * Décrire une application de trois services dans un fichier Compose
  
  * comprendre réseau et volumes
  
  * brancher l'API sur une base PostgreSQL et sur un stockage objet compatible S3
  
  * sortir les secrets du fichier


  ### Partie 0 — Préparer la VM (5 min)

  L'objectif ici est d'augmenter la mémoire de la  VM (car 3 conteneurs vont tourner en parallèle) et
  d'ouvrir le port 9001 pour accéder à l'interface web de MinIO depuis Windows.

  **1. Sur votre PC Windows (PowerShell)**

  ''terminal 
  
      $env:Path += ";C:\Program Files\Oracle\VirtualBox"
      VBoxManage controlvm ubuntu-srv acpipowerbutton     # Éteint la VM si elle tourne (attendez 20 sec)
      VBoxManage modifyvm ubuntu-srv --memory 3072        # Passe la RAM à 3 Go
      VBoxManage modifyvm ubuntu-srv --natpf1 "minio,tcp,127.0.0.1,9001,,9001"
      VBoxManage modifyvm ubuntu-srv --natpf1 "api,tcp,127.0.0.1,5000,,5000"   # Ignorez l'erreur si la règle existe déjà
      VBoxManage startvm ubuntu-srv --type headless       # Démarre la VM en arrière-plan
      ssh -p 2222 etudiant@127.0.0.1                      # Connexion à la VM

  ''

''' sortie 

<img width="957" height="476" alt="image" src="https://github.com/user-attachments/assets/c5026684-72c4-4dfa-abdd-3d9ee82bf1bb" />

'''

  **2. Dans la VM ubuntu-srv (en SSH)**

  Nettoyer les anciens conteneurs du TP2, créez le dossier tp3 en copiant  l'API du TP2, 
  et télécharger les images Docker nécessaires 
  '' terminal
  
    docker rm -f $(docker ps -aq) 2>/dev/null
    mkdir -p ~/tp3 && cp -r ~/api ~/tp3/api && cd ~/tp3
    docker pull postgres:17
    docker pull cgr.dev/chainguard/minio:latest

''

'''sortie 

<img width="948" height="473" alt="image" src="https://github.com/user-attachments/assets/7edb9187-f825-4e62-a4a1-790ff1b9f9cc" />

<img width="931" height="255" alt="image" src="https://github.com/user-attachments/assets/6ca40383-074b-4c00-abde-4cb2f997b84a" />

'''


### Partie 1 — Décrire et lancer l'application (25 min)

**1.1 Écrire le fichier Compose**

  Toujours dans le dossier ~/tp3 de la VM, créer le fichier docker-compose.yml.

''' terminal 

    cat > docker-compose.yml <<'EOF'
    services:
      api:
        build: ./api
        ports: ["5000:5000"]
        environment:
          BINOME: "douti"
          DATABASE_URL: postgresql://app:app_mdp@db:5432/app
          S3_ENDPOINT: http://minio:9000
          S3_ACCESS_KEY: tp3admin
          S3_SECRET_KEY: tp3-secret-123
          S3_BUCKET: fichiers
        depends_on:
          db:
            condition: service_healthy
          minio:
            condition: service_started
     
      db:
        image: postgres:17
        environment:
          POSTGRES_USER: app
          POSTGRES_PASSWORD: app_mdp
          POSTGRES_DB: app
        volumes:
          - db-data:/var/lib/postgresql/data
        healthcheck:
          test: ["CMD-SHELL", "pg_isready -U app -d app"]
          interval: 5s
          timeout: 3s
          retries: 10
     
      minio:
        image: cgr.dev/chainguard/minio:latest
        user: "0:0"
        command: ["server", "/data", "--console-address", ":9001"]
        environment:
          MINIO_ROOT_USER: tp3admin
          MINIO_ROOT_PASSWORD: tp3-secret-123
        ports: ["9001:9001"]
        volumes:
          - minio-data:/data
     
    volumes:
      db-data:
      minio-data:
    EOF

'''


**1.2 Lancer et observer**

Vérifier la syntaxe, lancer les conteneurs et observer leur état 

''' terminal

    docker compose config -q          # Ne doit rien afficher si le YAML est valide
    docker compose up -d --build      # Construit l'API et démarre les 3 conteneurs
    docker compose ps                 # Vérifie que db est (healthy) et les autres Up
    docker compose logs --tail 5 minio
    curl -s http://localhost:5000

'''

Ensuite, exécuter ces commandes d'exploration réseau

'''sortie
  
<img width="947" height="323" alt="image" src="https://github.com/user-attachments/assets/fc187d29-dac0-4c4c-92ca-86a380afca70" />

'''


''' terminal 

    docker network ls | grep tp3
    docker network inspect tp3_default --format '{{range .Containers}}{{.Name}} {{.IPv4Address}}{{"\n"}}{{end}}'
    docker compose exec api python -c "import socket; print(socket.gethostbyname('db'), socket.gethostbyname('minio'))"
    nc -zv localhost 5432
    docker compose exec db psql -U app -d app -c "SELECT version();"

'''


''' sortie

<img width="959" height="360" alt="image" src="https://github.com/user-attachments/assets/6a9a5af8-3306-45ab-b343-6e0b98b88189" />


<img width="1344" height="642" alt="image" src="https://github.com/user-attachments/assets/676d32fb-e1e4-4263-893d-63d8d9b72209" />

'''
  

**📝 Réponses aux questions (Partie 1)**


• Q1 : Les noms des trois conteneurs créés sont tp3-api-1, tp3-db-1 et tp3-minio-1.
      Le préfixe tp3 provient du nom du répertoire parent (~/tp3) dans lequel se trouve le fichier docker-compose.yml.
  
• Q2 : Les adresses IP attribuées aux trois conteneurs sur le réseau tp3_default (172.18.0.0/16) sont :

• tp3-minio-1 : 172.18.0.2
  
• tp3-db-1 : 172.18.0.3
  
• tp3-api-1 : 172.18.0.4
  
Depuis le conteneur api, le nom DNS db se résout vers 172.18.0.3 (qui correspond exactement à l'adresse IP interne de tp3-db-1) 
                                                                  
et minio vers 172.18.0.2. Cela fonctionne grâce au serveur DNS interne intégré de Docker qui enregistre automatiquement chaque nom 
                                                                  
de service Compose sur le réseau bridge du projet.

• Q3 : Non, le port 5432 n'est pas joignable depuis la VM (Connection refused). 
      C'est parce qu'aucune directive ports: n'a été définie pour le service db : le port n'est exposé qu'à l'intérieur du réseau privé Docker (tp3_default). 
      C'est une excellente pratique de sécurité (principe de moindre exposition) : dans le cloud, cela correspond à placer la base de données dans un sous-réseau privé (Private Subnet) inaccessible directement depuis Internet,
      où seule l'application située dans le sous-réseau public/applicatif a le droit de l'interroger.
        
• Q4 : Le healthcheck vérifie que PostgreSQL est réellement prêt à accepter des connexions SQL (via pg_isready), 
       et pas seulement que le conteneur est démarré. La condition service_healthy oblige Docker à attendre ce signal avant de lancer le conteneur api.
       Sans eux, l'API démarrerait trop vite, tenterait de se connecter à une base encore en cours d'initialisation, et planterait immédiatement.


### Partie 2 — Vérifier la persistance (15 min)

Exécutez les commandes dans cet ordre dans la VM pour tester le comportement des volumes 


''' terminal 


    # 1. Créer une table et insérer une donnée
    docker compose exec db psql -U app -d app -c "CREATE TABLE test (msg TEXT); INSERT INTO test VALUES ('bonjour TP3');"
    docker compose exec db psql -U app -d app -c "SELECT * FROM test;"
    
    # 2. Supprimer conteneurs et réseau SANS toucher aux volumes, puis relancer
    docker compose down
    docker compose ps -a
    docker volume ls | grep tp3
    docker compose up -d
    docker compose exec db psql -U app -d app -c "SELECT * FROM test;"   # La ligne est toujours là !
    
    # 3. Inspecter l'emplacement physique du volume sur la VM
    docker volume inspect tp3_db-data --format '{{.Mountpoint}}'
    sudo du -sh $(docker volume inspect tp3_db-data --format '{{.Mountpoint}}')
    docker system df -v | grep tp3
    
    # 4. Supprimer conteneurs, réseau ET volumes (-v), puis relancer
    docker compose down -v
    docker compose up -d
    docker compose exec db psql -U app -d app -c "SELECT * FROM test;"   # Erreur : relation "test" does not exist
    
  '''

''' sortie 

<img width="955" height="472" alt="image" src="https://github.com/user-attachments/assets/e4256b93-bc2f-41c6-801d-4ca65782d010" />

<img width="956" height="477" alt="image" src="https://github.com/user-attachments/assets/781cd840-1ca9-4230-9477-f68544a1fa4a" />

<img width="952" height="317" alt="image" src="https://github.com/user-attachments/assets/89be5241-bbc9-4dc7-89b5-27007dd62704" />

'''
      
📝 Réponses aux questions (Partie 2)

• Q1 (Tableau récapitulatif) :

Action	                    Conteneurs	        Réseau (tp3_default)	    Volumes (db-data, minio-data)	   Ligne bonjour TP3
  
docker compose down	         Supprimés	       Supprimé	                  Conservés(47M sur disque)	        Conservée (Bonjour TP3 est toukours là)
  
docker compose down -v	     Supprimés	        Supprimé	                 Supprimés	                       Perdue définitivement (relation "test" does not existe)

• Q2 : Les données se trouvent physiquement sur le disque de la VM hôte, 
       dans le répertoire géré par Docker : /var/lib/docker/volumes/tp3_db-data/_data. 
       Dans le cloud, l'équivalent d'un volume persistant attaché à une instance est le stockage bloc (Block Storage), 
       comme Amazon EBS (Elastic Block Store) sur AWS ou Azure Managed Disks.
         
• Q3 : L'option -v détruit irréversiblement tous les volumes nommés associés au projet. 
  En production, cela effacerait instantanément l'intégralité de la base de données et des fichiers des utilisateurs sans possibilité de récupération.


### Partie 3 — Brancher l'API sur la base et le stockage (25 min)


**1. Mettre à jour le code de l'API (v2.0)**

Dans la VM (~/tp3), mettez à jour api/requirements.txt et api/app.py 
                                                                  
''' terminal 

      cat > api/requirements.txt <<'EOF'
      psycopg[binary]>=3.2,<4
      boto3>=1.34,<2
      EOF
      
      cat > api/Dockerfile <<'EOF'
      FROM api:1.1
      WORKDIR /app
      ENV PIP_DEFAULT_TIMEOUT=300
      COPY requirements.txt .
      RUN pip install --no-cache-dir --retries 10 -r requirements.txt
      COPY . .
      EXPOSE 5000
      CMD ["python", "app.py"]
      EOF
      
      cat > api/app.py <<'EOF'
      import os
      import time
      import uuid
       
      import boto3
      import psycopg
      from botocore.exceptions import ClientError
      from flask import Flask, Response, jsonify, request
       
      DATABASE_URL = os.environ["DATABASE_URL"]
      BUCKET = os.environ.get("S3_BUCKET", "fichiers")
       
      s3 = boto3.client(
          "s3",
          endpoint_url=os.environ["S3_ENDPOINT"],
          aws_access_key_id=os.environ["S3_ACCESS_KEY"],
          aws_secret_access_key=os.environ["S3_SECRET_KEY"],
          region_name="us-east-1",
      )
      app = Flask(__name__)
       
       
      def connexion():
          return psycopg.connect(DATABASE_URL, autocommit=True)
       
       
      def initialiser():
          """Crée la table et le bucket, en attendant que la base et le stockage répondent."""
          for _ in range(30):
              try:
                  with connexion() as c:
                      c.execute(
                          """CREATE TABLE IF NOT EXISTS fichiers (
                                 id SERIAL PRIMARY KEY,
                                 nom TEXT NOT NULL,
                                 cle TEXT NOT NULL,
                                 taille INTEGER NOT NULL,
                                 depose_le TIMESTAMP DEFAULT now())"""
                      )
                  try:
                      s3.head_bucket(Bucket=BUCKET)
                  except ClientError:
                      s3.create_bucket(Bucket=BUCKET)
                  print("Base et stockage prêts", flush=True)
                  return
              except Exception as erreur:
                  print(f"En attente des services : {erreur}", flush=True)
                  time.sleep(2)
          raise RuntimeError("Base ou stockage injoignable")
       
       
      @app.get("/")
      def accueil():
          return jsonify(message="API fil rouge", version="2.0",
                         binome=os.environ.get("BINOME", "inconnu"))
       
       
      @app.get("/sante")
      def sante():
          with connexion() as c:
              c.execute("SELECT 1")
          s3.head_bucket(Bucket=BUCKET)
          return jsonify(base="ok", stockage="ok")
       
       
      @app.post("/fichiers")
      def deposer():
          fichier = request.files.get("fichier")
          if fichier is None or fichier.filename == "":
              return jsonify(erreur="champ 'fichier' manquant"), 400
          donnees = fichier.read()
          cle = f"{uuid.uuid4().hex}-{fichier.filename}"
          s3.put_object(Bucket=BUCKET, Key=cle, Body=donnees,
                        ContentType=fichier.mimetype or "application/octet-stream")
          with connexion() as c:
              ligne = c.execute(
                  "INSERT INTO fichiers (nom, cle, taille) VALUES (%s, %s, %s) RETURNING id",
                  (fichier.filename, cle, len(donnees)),
              ).fetchone()
          return jsonify(id=ligne[0], nom=fichier.filename, cle=cle, taille=len(donnees)), 201
       
       
      @app.get("/fichiers")
      def lister():
          with connexion() as c:
              lignes = c.execute(
                  "SELECT id, nom, cle, taille, depose_le FROM fichiers ORDER BY id"
              ).fetchall()
          return jsonify([
              {"id": l[0], "nom": l[1], "cle": l[2], "taille": l[3],
               "depose_le": l[4].isoformat()}
              for l in lignes
          ])
       
       
      @app.get("/fichiers/<int:ident>")
      def telecharger(ident):
          with connexion() as c:
              ligne = c.execute("SELECT cle FROM fichiers WHERE id = %s", (ident,)).fetchone()
          if ligne is None:
              return jsonify(erreur="fichier introuvable"), 404
          objet = s3.get_object(Bucket=BUCKET, Key=ligne[0])
          return Response(objet["Body"].read(), mimetype=objet["ContentType"])
       
       
      initialiser()
       
      if __name__ == "__main__":
          app.run(host="0.0.0.0", port=5000)
      EOF

'''                                                                  
 

**2. Reconstruire et tester l'API**
           
Dans la VM (~/tp3), lancez ces commandes 
      
 ''' terminal 
                
    docker compose up -d --build api
    docker compose logs api | head -n 5
    curl -s http://localhost:5000
    curl -s http://localhost:5000/sante
    
    # Tests d'envoi et de lecture de fichiers
    echo "Relevé de notes fictif" > releve.txt
    curl -s -F "fichier=@releve.txt" http://localhost:5000/fichiers
    curl -s -F "fichier=@/etc/hostname" http://localhost:5000/fichiers
    curl -s http://localhost:5000/fichiers
    curl -s http://localhost:5000/fichiers/1
    curl -s -F "autre=@releve.txt" http://localhost:5000/fichiers
    
    # Vérification dans PostgreSQL
    docker compose exec db psql -U app -d app -c "SELECT id, nom, cle, taille FROM fichiers;"
                
'''












      
