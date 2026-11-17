
# Partie 1 — Installation d'Hadoop en mode pseudo-distribué 

## 2.1 Mise à jour du système et installation de Java 
**commande**

'''terminal

    sudo apt update && sudo apt upgrade -y 
    sudo apt install -y openjdk-11-jdk ssh pdsh wget python3 
    java -version  # doit afficher openjdk version "11...
'''

**sortie**

'''ubuntu

  <img width="1256" height="652" alt="Capture d&#39;écran 2026-11-11 224415" src="https://github.com/user-attachments/assets/42173fa3-d4cb-4f04-a046-971853c1dc06" />

  <img width="1269" height="650" alt="Capture d&#39;écran 2026-11-11 225325" src="https://github.com/user-attachments/assets/77049ae7-8a49-4205-837d-821c8996d0e4" />
  
  <img width="907" height="292" alt="image" src="https://github.com/user-attachments/assets/8dede4ca-c3e3-4205-91e7-e9b303e1556f" />

## 2.2 Création d'un utilisateur dédié 

**commmande**

''' terminal

  sudo adduser hadoop            # choisir un mot de passe 
  sudo usermod -aG sudo hadoop   # droits administrateur (facultatif) 
  su - hadoop                    # se connecter en tant que hadoop 

'''
**sortie**

'''ubuntu 

<img width="1267" height="647" alt="image" src="https://github.com/user-attachments/assets/5910d976-b0e0-4fd8-a94d-d144c9c7e2d9" />

## 2.3 Configuration de SSH sans mot de passe 

**commande**

'''Terminal (utilisateur hadoop) 

  ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa 
  cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys 
  chmod 0600 ~/.ssh/authorized_keys 
  ssh localhost          # répondre "yes" la première fois, puis taper exit 
'''

**sortie**

'''ubuntu

<img width="1118" height="316" alt="Capture d&#39;écran 2026-11-11 231352" src="https://github.com/user-attachments/assets/70fad9f6-1690-43ee-b80e-da0141f0c1ed" />

<img width="1076" height="654" alt="Capture d&#39;écran 2026-11-11 232110" src="https://github.com/user-attachments/assets/79d4606f-d398-431b-9696-fe3acfc56ceb" />


## 2.4 Téléchargement et installation d'Hadoop 

**commande**

''' Terminal 

    cd ~ 
    wget https://archive.apache.org/dist/hadoop/common/hadoop-3.3.6/hadoop-3.3.6.tar.gz 
    tar -xzf hadoop-3.3.6.tar.gz 
    mv hadoop-3.3.6 hadoop 
'''

**sortie**

'''ubuntu 

<img width="1077" height="458" alt="image" src="https://github.com/user-attachments/assets/48ecbf8d-ced9-416c-bf43-bba6f26cf9d8" />


## 2.5 Variables d'environnement 

**commande**

'''  ~/.bashrc 

      # ---- Hadoop ---- 
      export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64 
      export HADOOP_HOME=$HOME/hadoop 
      export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop 
      export HADOOP_MAPRED_HOME=$HADOOP_HOME 
      export HADOOP_COMMON_HOME=$HADOOP_HOME 
      export HADOOP_HDFS_HOME=$HADOOP_HOME 
      export HADOOP_YARN_HOME=$HADOOP_HOME 
      export PDSH_RCMD_TYPE=ssh 
      export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
      
  ''
''' Terminal 

      source ~/.bashrc 
      hadoop version         # doit afficher Hadoop 3.3.6
      
'''

**sortie**

<img width="1020" height="236" alt="image" src="https://github.com/user-attachments/assets/9bcf26af-c135-474d-8082-65d7af377182" />


## 2.6 Fichiers de configuration 

**commande**
a)  hadoop-env.sh 

''' Terminal 

     cd $HADOOP_HOME/etc/hadoop
     nano adoop-env.sh 
     export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64 
''
b) core-site.xml 

''' Terminal

    core-site.xml 
    <configuration> 
      <property> 
        <name>fs.defaultFS</name> 
        <value>hdfs://localhost:9000</value> 
      </property> 
      <property> 
        <name>hadoop.tmp.dir</name> 
        <value>/home/hadoop/hadoopdata/tmp</value> 
      </property> 
    </configuration>

'''
c) hdfs-site.xml — paramètres HDFS 
'''Terminal

    hdfs-site.xml 
    <configuration> 
      <property> 
        <name>dfs.replication</name> 
        <value>1</value>   <!-- 1 seule machine : 1 seule copie --> 
      </property> 
      <property> 
        <name>dfs.namenode.name.dir</name> 
        <value>file:///home/hadoop/hadoopdata/namenode</value> 
      </property> 
      <property> 
        <name>dfs.datanode.data.dir</name> 
        <value>file:///home/hadoop/hadoopdata/datanode</value> 
      </property> 
    </configuration>

'''
d) mapred-site.xml — MapReduce sur YARN 

''' Terminal

    mapred-site.xml 
    <configuration> 
      <property> 
        <name>mapreduce.framework.name</name> 
        <value>yarn</value> 
      </property> 
      <property> 
        <name>mapreduce.application.classpath</name> 
        
    <value>$HADOOP_MAPRED_HOME/share/hadoop/mapreduce/*:$HADOOP_MAPRED_HOME/share/hadoop/mapreduce/lib/*
    </value> 
      </property> 
      <property> 
        <name>yarn.app.mapreduce.am.env</name> 
        <value>HADOOP_MAPRED_HOME=/home/hadoop/hadoop</value> 
      </property> 
      <property> 
        <name>mapreduce.map.env</name> 
        <value>HADOOP_MAPRED_HOME=/home/hadoop/hadoop</value> 
      </property> 
      <property> 
        <name>mapreduce.reduce.env</name> 
        <value>HADOOP_MAPRED_HOME=/home/hadoop/hadoop</value> 
      </property> 
    </configuration> 

'''
 
e) yarn-site.xml — gestionnaire de ressources 

''' Terminal

    yarn-site.xml 
    <configuration> 
      <property> 
        <name>yarn.nodemanager.aux-services</name> 
        <value>mapreduce_shuffle</value> 
      </property> 
      <property> 
        <name>yarn.nodemanager.env-whitelist</name> 
        
    <value>JAVA_HOME,HADOOP_COMMON_HOME,HADOOP_HDFS_HOME,HADOOP_CONF_DIR,CLASSPATH_PREPEND_DISTCACHE,HAD
    OOP_YARN_HOME,HADOOP_HOME,PATH,LANG,TZ,HADOOP_MAPRED_HOME</value> 
      </property> 
      <property> 
        <name>yarn.nodemanager.resource.memory-mb</name> 
        <value>4096</value> 
      </property> 
    </configuration>

'''

## 2.7 Formatage de HDFS et démarrage du cluster 

**commande**

''' Terminal

    mkdir -p ~/hadoopdata/{namenode,datanode,tmp} 
    hdfs namenode -format          # UNE SEULE FOIS, à la première installation 
    start-dfs.sh                   # démarre NameNode, DataNode, SecondaryNameNode 
    start-yarn.sh                  # démarre ResourceManager et NodeManager 
    jps                            # liste les processus Java actifs

'''

**sortie**

'''ubuntu 

<img width="828" height="261" alt="image" src="https://github.com/user-attachments/assets/af272421-82d4-4851-9fc3-d7762c20d5b6" />


## 2.8 Interfaces web 

**sortie**

<img width="1335" height="613" alt="image" src="https://github.com/user-attachments/assets/1d0abcac-6e35-4b0b-8859-f0a973a71dc0" />

<img width="1350" height="492" alt="image" src="https://github.com/user-attachments/assets/94b18fc6-1950-40b0-8e04-858201fb5be7" />

<img width="1309" height="615" alt="image" src="https://github.com/user-attachments/assets/d51037be-59ee-4116-a614-cee3d39a750d" />

**commande**

'''Terminal 

      stop-yarn.sh
      stop-dfs.sh
'''

**sortie**

<img width="374" height="156" alt="image" src="https://github.com/user-attachments/assets/570a1129-abd6-4f7c-8983-c0dd753e16a9" />

# Partie 3 — Manipulation de HDFS 

## Exercice 1 — Premiers pas avec HDFS 

**commande**

'''Terminal 

      # 1. Créer le répertoire de travail 
      hdfs dfs -mkdir -p /user/hadoop/tp/input 
        
      # 2. Créer un fichier texte local 
      mkdir -p ~/tp && cd ~/tp 
      cat > poeme.txt << 'EOF' 
      le chat dort sur le tapis 
      le chien court dans le jardin 
      le chat court apres le chien 
      EOF 
        
      # 3. Le copier dans HDFS et vérifier 
      hdfs dfs -put poeme.txt /user/hadoop/tp/input/ 
      hdfs dfs -ls /user/hadoop/tp/input 
      hdfs dfs -cat /user/hadoop/tp/input/poeme.txt 
        
      # 4. Observer les blocs 
      hdfs fsck /user/hadoop/tp/input/poeme.txt -files -blocks -locations 
'''

**sortie**

<img width="611" height="331" alt="image" src="https://github.com/user-attachments/assets/487f424f-9388-4082-a03d-a8f49584c2a1" />


## Question 
**Réponses aux questions pour votre compte rendu**


1) Combien de blocs le fichier poeme.txt occupe-t-il ? Pourquoi ?

• Nombre de blocs : Le fichier occupe 1 seul bloc (comme le confirme la ligne Total blocks (validated): 1 affichée par la commande hdfs fsck).
• Explication : Dans Hadoop 3, la taille maximale d'un bloc HDFS par défaut est de 128 Mo. Comme le fichier poeme.txt ne pèse que 87 octets, il est largement inférieur à 128 Mo et tient entièrement dans un unique bloc (qui n'occupe réellement que 87 octets sur le disque physique).

2) Retrouver le fichier dans l'interface web du NameNode

•<img width="1330" height="635" alt="image" src="https://github.com/user-attachments/assets/7790d73d-2dc2-44df-933e-a2f7975de3a8" />

3) Calculs pour un fichier de 1 Go avec une réplication de 3

• Nombre de blocs (avec 1 Go = 1024 Mo) :
(1024MO/128MO) =8bloc
• Espace disque utilisé : Votre résultat (1G0*3 = 3 GO  et 24 blocs physiques au total) 
 
# Partie 5 — Pratique : MapReduce en Python 

## 5.2 Exercice 2 — WordCount en Python 

**Etape1:creation de fichier mapper.py**

''' terminal

    #!/usr/bin/env python3
    """mapper.py : émet (mot, 1) pour chaque mot lu sur l'entrée standard."""
    import re
    import sys
    
    # Lecture ligne par ligne depuis l'entrée standard (stdin)
    for ligne in sys.stdin:
      # Suppression des espaces en début/fin et conversion en minuscules
      ligne = ligne.strip().lower()
      # Extraction des mots composés uniquement de lettres (accents inclus)
      mots = re.findall(r"[a-zàâäéèêëîïôöùûüç]+", ligne)
      # Émission de chaque paire (clé=mot, valeur=1) séparée par une tabulation
      for mot in mots:
        print(f"{mot}\t1")

'''

**Etape2:Creation de fichier**

''' Terminal

     #!/usr/bin/env python3
    """reducer.py : additionne les compteurs de chaque mot.
    
    Les lignes arrivent triées par clé : les mêmes mots sont consécutifs.
    """
    import sys
    
    mot_courant = None
    total = 0
    
    # Lecture des paires triées envoyées par la phase Shuffle & Sort
    for ligne in sys.stdin:
      # Découpage de la ligne en clé (mot) et valeur (compteur)
      mot, valeur = ligne.rstrip("\n").split("\t", 1)
      try:
        valeur = int(valeur)
      except ValueError:
        continue  # Ignore les lignes mal formées
    
      # Si le mot est identique au précédent, on cumule son compteur
      if mot == mot_courant:
        total += valeur
      else:
        # Changement de mot : on affiche le total du mot précédent
        if mot_courant is not None:
          print(f"{mot_courant}\t{total}")
        mot_courant = mot
        total = valeur
    
    # Affichage du dernier mot traité en fin de flux
    if mot_courant is not None:
      print(f"{mot_courant}\t{total}")


'''

**Etape3:tester en local (sans hadoop)**

''' Terminal

    chmod +x mapper.py reducer.py
    cat poeme.txt | ./mapper.py | sort | ./reducer.py
    
'''

''' sortie

<img width="871" height="363" alt="image" src="https://github.com/user-attachments/assets/93472e7a-24db-467b-b163-d9451b176416" />

'''

**Etape 4 : Exécution sur Hadoop**

''' Terminal 

    hadoop jar $HADOOP_HOME/share/hadoop/tools/lib/hadoop-streaming-3.3.6.jar \
      -files mapper.py,reducer.py \
      -mapper "python3 mapper.py" \
      -reducer "python3 reducer.py" \
      -input /user/hadoop/tp/input \
      -output /user/hadoop/tp/output_wordcount
    
    # Lire le résultat
    hdfs dfs -cat /user/hadoop/tp/output_wordcount/part-00000

'''
''' sortie 

<img width="948" height="457" alt="image" src="https://github.com/user-attachments/assets/fd924d5d-f087-4e60-ba58-373976f50007" />
    
<img width="952" height="465" alt="image" src="https://github.com/user-attachments/assets/e0c3fcfd-5798-4708-97fe-9912f78d01ae" />

<img width="952" height="469" alt="image" src="https://github.com/user-attachments/assets/e2879156-d320-46e1-94a7-6a3eb9a2f45b" />

<img width="920" height="238" alt="image" src="https://github.com/user-attachments/assets/dd7edd3f-f98d-4557-b98c-85c652cb029e" />

'''

**Etape 5 : Test sur un vrai texte**


''' Terminal 

        wget -O ~/tp/miserables.txt https://gutenberg.org
        hdfs dfs -mkdir -p /user/hadoop/tp/livre
        hdfs dfs -put ~/tp/miserables.txt /user/hadoop/tp/livre/
        
        hadoop jar $HADOOP_HOME/share/hadoop/tools/lib/hadoop-streaming-3.3.6.jar \
          -files mapper.py,reducer.py \
          -mapper "python3 mapper.py" \
          -combiner "python3 reducer.py" \
          -reducer "python3 reducer.py" \
          -input /user/hadoop/tp/livre \
          -output /user/hadoop/tp/output_livre
        
        # Afficher les 20 mots les plus fréquents
        hdfs dfs -cat /user/hadoop/tp/output_livre/part-* | sort -k2 -nr | head -20

'''

''' sortie 

'''


## Exercice 3 — Chiffre d'affaires par ville

**Creation du fichier CSV**

''' Terminal 

    cat > ~/tp/ventes.csv << 'EOF'
    date,ville,produit,quantite,prix_unitaire
    2026-01-03,Abidjan,Ordinateur,2,350000
    2026-01-03,Bouake,Imprimante,1,95000
    2026-01-04,Abidjan,Souris,10,5000
    2026-01-05,Yamoussoukro,Ordinateur,1,350000
    2026-01-05,San-Pedro,Ecran,3,80000
    2026-01-06,Bouake,Ordinateur,1,350000
    2026-01-06,Abidjan,Ecran,2,80000
    2026-01-07,Korhogo,Clavier,5,8000
    2026-01-07,San-Pedro,Souris,4,5000
    2026-01-08,Yamoussoukro,Imprimante,2,95000
    EOF


''''
**Creation de mapper_ventes.py**

''' Terminal

    #!/usr/bin/env python3
    """mapper_ventes.py : émet (ville, montant) pour chaque vente."""
    import sys
    
    for ligne in sys.stdin:
      ligne = ligne.strip()
      # Ignore les lignes vides et la ligne d'en-tête du fichier CSV
      if not ligne or ligne.startswith("date"):
        continue
    
      # Découpage des 5 colonnes : date, ville, produit, quantite, prix_unitaire
      champs = ligne.split(",")
      if len(champs) != 5:
        continue
    
      _, ville, _, quantite, prix = champs
      try:
        # Calcul du montant total de la transaction
        montant = int(quantite) * float(prix)
      except ValueError:
        continue  # Ignore la ligne en cas d'erreur de conversion numérique
    
      # Émission de la paire (clé=ville, valeur=montant)
      print(f"{ville}\t{montant}")

'''
**Creation de reducer_ventes.py**

''' Terminal

        #!/usr/bin/env python3
    """reducer_ventes.py : calcule le CA total et le nombre de ventes par ville."""
    import sys
    
    ville_courante, total, nb = None, 0.0, 0
    
    
    def emettre(ville, total, nb):
      """Affiche la ville, son chiffre d'affaires arrondi et son nombre de ventes."""
      print(f"{ville}\t{total:.0f}\t{nb}")
    
    
    for ligne in sys.stdin:
      # Extraction de la ville et du montant de la vente
      ville, montant = ligne.rstrip("\n").split("\t")
      montant = float(montant)
    
      # Cumul du chiffre d'affaires et du nombre de ventes pour une même ville
      if ville == ville_courante:
        total += montant
        nb += 1
      else:
        # Nouvelle ville : on émet les résultats cumulés de la ville précédente
        if ville_courante is not None:
          emettre(ville_courante, total, nb)
        ville_courante, total, nb = ville, montant, 1
    
    # Émission des résultats de la dernière ville
    if ville_courante is not None:
      emettre(ville_courante, total, nb)

'''

**Rendre exécutables et envoyer sur HDFS**

''' Terminal

    chmod +x ~/tp/mapper_ventes.py ~/tp/reducer_ventes.py
    hdfs dfs -mkdir -p /user/hadoop/tp/ventes
    hdfs dfs -put ~/tp/ventes.csv /user/hadoop/tp/ventes/

'''

**Exécuter le job Hadoop et lire le résultat **

''' Terminal

    hadoop jar $HADOOP_HOME/share/hadoop/tools/lib/hadoop-streaming-3.3.6.jar \
      -files mapper_ventes.py,reducer_ventes.py \
      -mapper "python3 mapper_ventes.py" \
      -reducer "python3 reducer_ventes.py" \
      -input /user/hadoop/tp/ventes \
      -output /user/hadoop/tp/output_ventes
        
'''

''' sortie 

<img width="1133" height="687" alt="image" src="https://github.com/user-attachments/assets/a05e9d52-e174-431b-b861-e0931ca915f3" />

<img width="1127" height="685" alt="image" src="https://github.com/user-attachments/assets/8b8383db-3d77-40d7-99db-f5b97e0dd926" />

<img width="1062" height="192" alt="image" src="https://github.com/user-attachments/assets/239b339c-6b3a-485b-b102-b5ce0323b0e0" />

'''

''' Terminal

    hdfs dfs -cat /user/hadoop/tp/output_ventes/part-00000
    
'''

''' sortie

<img width="879" height="214" alt="image" src="https://github.com/user-attachments/assets/f7cb984c-6f3a-4d6b-8983-4ef1e193d19e" />

'''


## Exercice 4 — Température maximale par année 


**Etape 1 : Creation du fichier de données ~/tp/meteo.txt**

''' Terminal 

    cat > ~/tp/meteo.txt << 'EOF'
    2024,ABJ,31.2
    2024,BKE,33.5
    2025,ABJ,32.8
    2025,KHG,36.1
    2024,KHG,35.0
    2025,BKE,30.9
    EOF
    
'''

**Etape 2 : Creation du Mapper (~/tp/mapper_meteo.py)**

Il extrait l'année et la température sur chaque ligne et les affiche séparées par une tabulation (annee<TAB>temperature) :
''' Terminal

    #!/usr/bin/env python3
    """mapper_meteo.py : émet (annee, temperature) pour chaque relevé météo."""
    import sys
    
    for ligne in sys.stdin:
      ligne = ligne.strip()
      if not ligne:
        continue
    
      # Découpage des 3 colonnes : annee, station, temperature
      champs = ligne.split(",")
      if len(champs) != 3:
        continue
    
      annee, _, temp = champs
      try:
        # Conversion de la température en nombre décimal et émission
        temp_val = float(temp)
        print(f"{annee}\t{temp_val}")
      except ValueError:
        continue


'''
**Etape 3 : Créer le Reducer (~/tp/reducer_meteo.py)**

Comme Hadoop trie les paires par année, le reducer compare les températures d'une même année et ne garde que le maximum :
''' Terminal

    #!/usr/bin/env python3
    """reducer_meteo.py : détermine la température maximale enregistrée par année."""
    import sys
    
    annee_courante = None
    temp_max = None
    
    for ligne in sys.stdin:
      # Récupération de l'année et de la température associée
      annee, temp = ligne.rstrip("\n").split("\t")
      temp = float(temp)
    
      # Comparaison pour conserver uniquement la température la plus élevée de l'année
      if annee == annee_courante:
        if temp > temp_max:
          temp_max = temp
      else:
        # Changement d'année : affichage du maximum de l'année précédente
        if annee_courante is not None:
          print(f"{annee_courante}\t{temp_max}")
        annee_courante = annee
        temp_max = temp
    
    # Affichage du maximum de la dernière année traitée
    if annee_courante is not None:
      print(f"{annee_courante}\t{temp_max}")

    
'''
**Etape 4 : Tester en local (sans Hadoop)**

''' Terminal

    chmod +x ~/tp/mapper_meteo.py ~/tp/reducer_meteo.py
    cat ~/tp/meteo.txt | ~/tp/mapper_meteo.py | sort | ~/tp/reducer_meteo.py
    
'''

''' sortie 

<img width="685" height="104" alt="image" src="https://github.com/user-attachments/assets/58c6acab-4d3b-4253-bd51-7cf7d12021ad" />

'''
**Etape 5 : Exécuter sur Hadoop et lire le résultat**
1-Envoyer meteo.txt dans HDFS 

''' Terminal

    hdfs dfs -mkdir -p /user/hadoop/tp/meteo
    hdfs dfs -put ~/tp/meteo.txt /user/hadoop/tp/meteo/
'''

'''sortie

<img width="820" height="83" alt="image" src="https://github.com/user-attachments/assets/67ac61dd-3d1f-4eef-b107-de334e293ab9" />

'''

2.Lancer le job MapReduce 

''' Terminal

    hadoop jar $HADOOP_HOME/share/hadoop/tools/lib/hadoop-streaming-3.3.6.jar \
      -files mapper_meteo.py,reducer_meteo.py \
      -mapper "python3 mapper_meteo.py" \
      -reducer "python3 reducer_meteo.py" \
      -input /user/hadoop/tp/meteo \
      -output /user/hadoop/tp/output_meteo
    
'''


'' sortie

<img width="1109" height="680" alt="image" src="https://github.com/user-attachments/assets/a93df513-d915-40b8-a514-4546bf7663f8" />

<img width="1107" height="684" alt="image" src="https://github.com/user-attachments/assets/99f523e4-3261-49cd-ae36-d8345d277678" />

<img width="1043" height="198" alt="image" src="https://github.com/user-attachments/assets/3cc87d37-35f9-4140-9b97-3a7f0cb4b723" />

'''

''' Terminal

    hdfs dfs -cat /user/hadoop/tp/output_meteo/part-00000

'''

''' sortie

<img width="988" height="192" alt="image" src="https://github.com/user-attachments/assets/1dcf54bc-afd9-4bd5-ab78-87ef6732c1a1" />

'''

## Exercice 5 — Moyenne des notes par matière 



**Etape 1 : Réponse à l'encadré « Question »**

• Peut-on utiliser le reducer de moyenne comme combiner ? Non.
• Justification : La moyenne arithmétique n'est pas une opération associative ni distributive. La moyenne des moyennes calculées sur des sous-groupes de tailles différentes n'est pas égale à la moyenne globale (par exemple, la moyenne de [10, 20] est 15 et celle de [0] est 0 ; faire la moyenne de 15 et 0 donne 7,5 au lieu de la vraie moyenne (10 + 20 + 0) / 3 = 10).
• Solution : Le mapper émet la note accompagnée d'un compteur 1 (matiere<TAB>note<TAB>1). Le combiner (facultatif) et le reducer somment séparément les notes et les compteurs, puis le reducer effectue la division finale somme / total_notes.

**Etape 2 : Créer le fichier ~/tp/notes.csv**


'''terminal

    cat > ~/tp/notes.csv << 'EOF'
    etudiant,matiere,note
    Kouassi,BigData,15.5
    Aya,Python,14.0
    Jean,BigData,12.5
    Awa,Python,16.0
    Marc,BigData,17.0
    EOF
'''

**Etape 3 : Créer le Mapper (~/tp/mapper_notes.py)**

''' Terminal

     #!/usr/bin/env python3
    """mapper_notes.py : émet (matiere, note, 1) pour chaque note d'étudiant."""
    import sys
    
    for ligne in sys.stdin:
      ligne = ligne.strip()
      # Ignore les lignes vides et l'en-tête du CSV
      if not ligne or ligne.startswith("etudiant"):
        continue
    
      # Découpage des 3 colonnes : etudiant, matiere, note
      champs = ligne.split(",")
      if len(champs) != 3:
        continue
    
      _, matiere, note = champs
      try:
        note_val = float(note)
        # Émet la matière, la note et un compteur unitaire (1) pour préparer la moyenne
        print(f"{matiere}\t{note_val}\t1")
      except ValueError:
        continue

'''

**Etape 4 : Créer le Reducer (~/tp/reducer_notes.py)**

''' Terminal

        #!/usr/bin/env python3
        """reducer_notes.py : calcule la moyenne globale des notes par matière."""
        import sys
        
        matiere_courante = None
        somme = 0.0
        nb = 0
        
        for ligne in sys.stdin:
          # Récupération de la matière, de la note et du compteur
          matiere, note, count = ligne.rstrip("\n").split("\t")
          note, count = float(note), int(count)
        
          # Cumul séparé de la somme des notes et du nombre d'étudiants
          if matiere == matiere_courante:
            somme += note
            nb += count
          else:
            # Changement de matière : calcul et affichage de la moyenne (somme / effectif)
            if matiere_courante is not None:
              print(f"{matiere_courante}\t{somme / nb:.2f}")
            matiere_courante, somme, nb = matiere, note, count
        
        # Calcul et affichage de la moyenne pour la dernière matière
        if matiere_courante is not None:
          print(f"{matiere_courante}\t{somme / nb:.2f}")

'''

**Etape 5 : Tester en local puis exécuter sur Hadoop**

1. Rendre exécutables et tester en local :

''' Terminal 

    chmod +x ~/tp/mapper_notes.py ~/tp/reducer_notes.py
    cat ~/tp/notes.csv | ~/tp/mapper_notes.py | sort | ~/tp/reducer_notes.py
'''
'''sortie 

<img width="768" height="124" alt="image" src="https://github.com/user-attachments/assets/6fd2f0a9-2760-43db-b5fe-b14211735a50" />

'''

3. Envoyer notes.csv sur HDFS :

''' Terminal 

    hdfs dfs -mkdir -p /user/hadoop/tp/notes
    hdfs dfs -put ~/tp/notes.csv /user/hadoop/tp/notes/

'''

5. Lancer le job MapReduce et lire le resultat :

''' Terminal
        
        hadoop jar $HADOOP_HOME/share/hadoop/tools/lib/hadoop-streaming-3.3.6.jar \
          -files mapper_notes.py,reducer_notes.py \
          -mapper "python3 mapper_notes.py" \
          -reducer "python3 reducer_notes.py" \
          -input /user/hadoop/tp/notes \
          -output /user/hadoop/tp/output_notes
        hdfs dfs -cat /user/hadoop/tp/output_notes/part-00000

'''

''' sortie 

<img width="885" height="656" alt="image" src="https://github.com/user-attachments/assets/130f4c64-6ec2-441e-ba85-864403878e68" />

<img width="890" height="682" alt="image" src="https://github.com/user-attachments/assets/330b151a-cf4d-4df1-926f-fcddfac4c3c3" />

<img width="862" height="216" alt="image" src="https://github.com/user-attachments/assets/2f8f2442-8e3f-4c0c-a3ec-1624873e4737" />

'''

## Exercice 6 — Index inversé 

**Création de  plusieurs fichiers texte d'essai**

''' TErminal

        mkdir -p ~/tp/docs_index
        echo "hadoop et spark sont des outils big data" > ~/tp/docs_index/doc1.txt
        echo "python est utilise avec hadoop streaming" > ~/tp/docs_index/doc2.txt
        echo "le big data utilise hdfs pour le stockage" > ~/tp/docs_index/doc3.txt

'''

**Création de  le Mapper (~/tp/mapper_index.py)**

''' Terminal

        #!/usr/bin/env python3
    """mapper_index.py : associe chaque mot au nom du fichier d'où il provient."""
    import os
    import re
    import sys
    
    # Récupération du chemin complet du fichier lu sur HDFS via la variable Hadoop
    chemin = os.environ.get("mapreduce_map_input_file", "inconnu")
    # Extraction uniquement du nom du fichier (ex. : doc1.txt)
    nom_fichier = os.path.basename(chemin)
    
    for ligne in sys.stdin:
      ligne = ligne.strip().lower()
      # Extraction des mots de la ligne
      mots = re.findall(r"[a-zàâäéèêëîïôöùûüç]+", ligne)
      # Émission de chaque paire (clé=mot, valeur=nom_fichier)
      for mot in mots:
        print(f"{mot}\t{nom_fichier}")

'''

**Création du Reducer (~/tp/reducer_index.py)**

''' Terminal 
    
            #!/usr/bin/env python3
    """reducer_index.py : regroupe sans doublon les fichiers contenant chaque mot."""
    import sys
    
    mot_courant = None
    # Utilisation d'un ensemble (set) pour éliminer automatiquement les doublons de fichiers
    fichiers = set()
    
    for ligne in sys.stdin:
      mot, fichier = ligne.rstrip("\n").split("\t", 1)
    
      # Ajout du nom de fichier à l'ensemble associé au mot courant
      if mot == mot_courant:
        fichiers.add(fichier)
      else:
        # Changement de mot : affichage du mot suivi de la liste triée des fichiers
        if mot_courant is not None:
          print(f"{mot_courant}\t{', '.join(sorted(fichiers))}")
        mot_courant = mot
        fichiers = {fichier}
    
    # Affichage du dernier mot et de ses fichiers associés
    if mot_courant is not None:
      print(f"{mot_courant}\t{', '.join(sorted(fichiers))}")

'''

**Exécuter sur Hadoop et afficher l'index inversé**

''' Terminal

    chmod +x ~/tp/mapper_index.py ~/tp/reducer_index.py
    hdfs dfs -mkdir -p /user/hadoop/tp/index_in
    hdfs dfs -put ~/tp/docs_index/* /user/hadoop/tp/index_in/
    
    hadoop jar $HADOOP_HOME/share/hadoop/tools/lib/hadoop-streaming-3.3.6.jar \
      -files mapper_index.py,reducer_index.py \
      -mapper "python3 mapper_index.py" \
      -reducer "python3 reducer_index.py" \
      -input /user/hadoop/tp/index_in \
      -output /user/hadoop/tp/output_index
'''

'''sortie 

<img width="998" height="633" alt="image" src="https://github.com/user-attachments/assets/682140dd-8901-4c44-8a56-bac6ca5ac13a" />


<img width="1002" height="637" alt="image" src="https://github.com/user-attachments/assets/78547197-ec49-4282-965e-c8b84dc8b750" />


<img width="863" height="287" alt="image" src="https://github.com/user-attachments/assets/98e9b6ae-12ce-4ff4-807d-30618f180b8a" />


'''
''' Terminal 

    hdfs dfs -cat /user/hadoop/tp/output_index/part-00000

'''
'''sortie

<img width="717" height="342" alt="image" src="https://github.com/user-attachments/assets/83897c3b-807c-4576-8d05-6a605bbabcc4" />


'''

## 5.5 Plusieurs reducers 


''' Terminal
    
    hadoop jar $HADOOP_HOME/share/hadoop/tools/lib/hadoop-streaming-3.3.6.jar \
      -D mapreduce.job.reduces=3 \
      -files mapper.py,reducer.py \
      -mapper "python3 mapper.py" \
      -reducer "python3 reducer.py" \
      -input /user/hadoop/tp/input \
      -output /user/hadoop/tp/output_3red

'''
''' sortie 

<img width="1072" height="675" alt="image" src="https://github.com/user-attachments/assets/d86667a7-52bb-4a90-b360-c32f5b9fece8" />

<img width="1071" height="683" alt="image" src="https://github.com/user-attachments/assets/ea153f95-53cd-4d2e-9825-f407f51201df" />

<img width="1075" height="684" alt="image" src="https://github.com/user-attachments/assets/6d244f1d-67f7-4d95-8c44-e072c12bef34" />

<img width="1073" height="474" alt="image" src="https://github.com/user-attachments/assets/f2ac471a-745a-4218-9aea-19b4ca210432" />

'''
'''TErminal

    hdfs dfs -ls /user/hadoop/tp/output_3red

'''

''' sortie

<img width="901" height="199" alt="image" src="https://github.com/user-attachments/assets/f8058901-426a-41ae-be92-89e631bbf696" />


'''















