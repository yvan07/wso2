# Installation et configuration

## Installer l'environnement d'exécution OpenJDK ( Prerequis )

Java JRE et JDK sont tous deux disponibles dans le référentiel de Rocky Linux et il en va de même pour d'autres distributions de serveurs Linux similaires. Ici, nous allons installer OpenJDK 11, vous pouvez également opter pour la version 8 si vous le souhaitez

```
sudo dnf install java-11-openjdk.x86_64
```

pour open-jdk 8

```
sudo dnf install java-1.8.0-openjdk.x86_64
```

### Vérifier la version Java

Une fois l'installation de l'une des versions ci-dessus terminée, vérifiez-la à l'aide de la commande ci-dessous pour confirmer l'installation.

```
java --version
```

la sortie devrait ressembler à celà:

```
openjdk 11.0.11 2021-04-20 LTS
OpenJDK Runtime Environment 18.9 (build 11.0.11+9-LTS)
OpenJDK 64-Bit Server VM 18.9 (build 11.0.11+9-LTS, mixed mode, sharing)
```

## Installation manuelle (package tomcat non recommandé)

installez JDK et lancez la commande suivante:

```
yum -y install java-1.8.0-openjdk.x86_64 java-1.8.0-openjdk-devel.x86_64
```

Lorsque Java est installé, vérifiez la version java avec la commande
suivante:

```
java -version
```

Exécutez la commande suivante pour installer le package Tomcat:

```
sudo yum install tomcat tomcat-webapps tomcat-admin-webapps
```

Exécutez la commande suivante pour autoriser le port d'écoute via l'extérieur :

```
sudo firewall-cmd --zone=public --permanent --add-port=8080/tcp
sudo firewall-cmd --reload
```

## Installation par Ansible

L'installation de Tomcat se fait à travers le rôle Ansible Tomcat:

```
ansible-playbook /etc/ansible/playbooks/tomcat/tomcat.yml
```

## Installation sur RHEL8

### Mise à jour

Exécutez la commande de mise à jour du système pour obtenir le dernier état stable de tous les packages installés sur votre système RHEL8.

```
sudo dnf update
```

## Télécharger apache tomcat ( Recommandé)

## Créer un utilisateur non root pour Tomcat

Créons un groupe et un utilisateur qui n'auront accès qu'à Tomcat et ne pourront pas être utilisés à d'autres fins, telles que la connexion au système pour installer ou supprimer quoi que ce soit.

## Ajouter un groupe Tomcat

```
sudo groupadd tomcat
```

Ajoutez un utilisateur et définissez le répertoire créé ci-dessus comme dossier d'accueil et désactivez également ses droits de connexion à l'aide de la commande ci-dessous

```
sudo useradd -d /opt/tomcat -r -s /bin/false -g tomcat tomcat

```

Plusieurs versions de Tomcat sont disponibles, telles que Tomcat 8, 9 et 10. Toutes ces trois versions prennent en charge Java 8 et les versions ultérieures. Ainsi, vous pouvez télécharger celui selon votre choix. Ici nous telechargerons tomcat ( [https://downloads.apache.org/tomcat/tomcat-9/](https://downloads.apache.org/tomcat/tomcat-9/) )

```
yum -y install wget
export VER="9.0.64"
wget https://archive.apache.org/dist/tomcat/tomcat-9/v${VER}/bin/apache-tomcat-${VER}.tar.gz
```

## Extraire et déplacer des fichiers

Une fois le téléchargement terminé, extrayez le fichier Tar et copiez-le dans le répertoire /opt/tomcat que nous avons créé précédemment.

```
sudo tar xvzf apache-tomcat-9.0.64.tar.gz -C /opt/
```

## Définir les autorisations

Comme nous avons déjà créé un utilisateur dédié pour Tomcat, nous lui permettons donc de lire les fichiers qui y sont disponible

```
ln -s /opt/apache-tomcat-$VER/ /opt/tomcat

sudo chown -R tomcat: /opt/tomcat
sudo chown -R tomcat: /opt/apache-tomcat-$VER/

```

## Autorisez également l'exécution du script disponible dans le dossier

```
sudo sh -c 'chmod +x /opt/tomcat/bin/*.sh'
```

## Créer un fichier de service Apache Tomcat

Par défaut, nous n'aurons pas de fichier Systemd pour Tomcat comme le serveur Apache pour arrêter, démarrer et activer ses services. Ainsi, nous en créons un, afin de pouvoir le gérer facilement.

```
sudo vi /etc/systemd/system/tomcat.service
```

Collez le code suivant à l'interieur

```
[Unit]
Description=Tomcat webs servlet container
After=network.target
[Service]
Type=forking
User=tomcat
Group=tomcat
Environment="JAVA_HOME=/usr/lib/jvm/jre"
Environment="JAVA_OPTS=-Djava.awt.headless=true -Djava.security.egd=file:/dev/./urandom"
Environment="CATALINA_BASE=/opt/tomcat"
Environment="CATALINA_HOME=/opt/tomcat"
Environment="CATALINA_PID=/opt/tomcat/temp/tomcat.pid"
Environment="CATALINA_OPTS= -server -XX:+UseParallelGC"
ExecStart=/opt/tomcat/bin/startup.sh
ExecStop=/opt/tomcat/bin/shutdown.sh
[Install]
WantedBy=multi-user.target
```

## Démarrer, activer et vérifier l'état du service

Après avoir créé avec succès le fichier systemd pour tomcat, démarrez son service à l'aide des commandes ci-dessous.

## Demarrer

```
sudo systemctl enable --now tomcat
sudo systemctl status tomcat
```

## Ouvrir le port 8080 du parefeu Rocky Linux

Pour accéder à l'interface Web Apache Tomcat en dehors de l'hôte local, vous devez ouvrir le port 8080 dans le pare-feu de RHEL8 que vous utilisez.

```
sudo firewall-cmd --zone=public --permanent --add-port=8080/tcp
sudo firewall-cmd --reload
```

## Accéder à l'interface Web

Ouvrir le navigateur et taper:

```
http://localhost:8080 ou http://@ip_serveur:8080
```

Une erreur 403 apparaîtra lors de son utilisation sur n'importe quel autre PC pour accéder à l'interface de gestion.

## 403 Accès refusé "erreur Tomcat"

Lorsque nous cliquons sur " Server Status ", " Manager App " et " Host Manager ", on peut voir une erreur 403 Access Denied.

# Pour résoudre cette erreur, suivre les étapes suivantes…

## 1\. Ajoutez le nom d'utilisateur et le mot de passe au fichier XML de l'utilisateur Tomcat\

```
sudo vi /opt/tomcat/conf/tomcat-users.xml
```


A la fin juste avant la balise copiez et collez les lignes suivantes.

```
<role rolename="admin"/>
<role rolename="admin-gui"/>
<role rolename="manager"/>
<role rolename="manager-gui"/>
<user username="h2s" password="pwd" roles="admin,admin-gui,manager,manager-gui"/>
```

(Modifiez le nom d' utilisateur et le mot de passe , avec ce que vous souhaitez définir pour votre Tomcat.)

## 2\. Modifier le fichier XML Manager Context

sudo vi /opt/tomcat/webapps/manager/META-INF/context.xml

Dans le fichier, faites défiler et allez à la fin et commentez le bloc de texte suivant:

```
<Valve className="org.apache.catalina.valves.RemoteAddrValve"
allow="127\.\d+\.\d+\.\d+|::1|0:0:0:0:0:0:0 : 1" />
```

## 3\. Modifier le fichier Host\-Manager Context\.XML
    
```
sudo vi /opt/tomcat/webapps/host-manager/META-INF/context.xml
```

Tout comme ci-dessus, commenter le texte ci-desous

```
<Valve className="org.apache.catalina.valves.RemoteAddrValve"
allow="127\.\d+\.\d+\.\d+|::1|0:0:0:0:0 : 0:0:1" />
```

# Redemarrer les service tomcat

```
sudo systemctl restart tomcat
```

on pourra maintenant acceder à " Server Status ", " Manager App " et " Host Manager ".
