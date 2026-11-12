
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

d) mapred-site.xml — MapReduce sur YARN 
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
 
e) yarn-site.xml — gestionnaire de ressources 
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


