```
[apim.sync_runtime_artifacts.publisher]
# Par défaut, DBSaver enregistre dans la DB ; si vous mettez true,
# le Publisher poussera directement les artefacts sur les Gateways
# sans passer par Traffic Manager.
publish_directly_to_gateway = true

[apim.sync_runtime_artifacts.gateway]
# Passez en "sync" pour forcer la Gateway à bloquer son démarrage
# tant qu’elle n’a pas récupéré tous les artefacts
data_retrieval_mode   = "sync"
# Durée maximum (ms) d’attente / retry en cas d’échec
deployment_retry_duration = 10000
max_retry_count           = 3
event_waiting_time        = 1000
```


## Installation Manuelle avec la base H2 Intégré

Il faut faire ces configurations  sur chaque instances Wso2am

- Télécharger le java developpment Kit version 11.0

```
sudo yum -y install wget curl unzip vim
wget https://download.java.net/java/ga/jdk11/openjdk-11_linux-x64_bin.tar.gz
```

- Configurer JDK11.0

```
tar zxvf openjdk-11_linux-x64_bin.tar.gz
sudo mv jdk-11/ /opt/jdk-11/
```

- Configurer JDK11 en éditant le fichier ~/.bashrc

```
export JAVA_HOME=/opt/jdk-11
export PATH=${JAVA_HOME}/bin:${PATH}
```

- Prise en compte de la configuration

```
source ~/.bashrc
```

- Vérification des configurations

```
echo $JAVA_HOME
java --version
```

- Ouverture des ports

```
firewall-cmd --add-port={4000,5672,8243,8280,9443,9763,9611,9711}/tcp --zone=public --permanent
firewall-cmd --reload
```

- Téléchargement de WSO2 API MANAGER

```
cd /var/lib
sudo curl -LO https://archive.smile.ci/wso2/wso2am-4.1.0.zip
```

```
sudo unzip wso2am-4.1.0.zip
cd /srv
ln -s /var/lib/wso2am
```

- Création de service

```
mkdir -p /var/lib/wso2am/.java
mkdir -p /var/lib/wso2am/.java/.userPrefs
```

Rajoutez dans le fichier /etc/default/wso2am   les variables d'environnement ci-dessous

vim /etc/default/wso2am

```
JAVA_HOME=/opt/jdk-11
    JVM_MEM_OPTS=-Xms4771M -Xmx4771M
JAVA_OPTS=-Dcom.sun.jndi.ldap.object.disableEndpointIdentification=true -Djava.util.prefs.systemRoot=/var/lib/wso2am/.java -Djava.util.prefs.userRoot=/var/lib/wso2am/.java/.userPrefs -server -Dorg.opensaml.httpclient.https.disableHostnameVerification=true  -Dorg.wso2.ignoreHostnameVerification=true -Dhttpclient.hostnameVerifier=AllowAll -XX:MaxMetaspaceSize=1g -XX:MaxPermSize=1024m -DentityExpansionLimit=10000 -XX:NewRatio=1 -XX:SurvivorRatio=6
```

Rajoutez la  configuration ci-dessous  /etc/systemd/system/wso2am.service

vim /etc/systemd/system/wso2am.service

```
[Install]
Description=Manage service wso2am with podman
WantedBy=multi-user.target
Wants=network-online.target
After=network-online.target remote-fs.target nss-lookup.target

[Service]
Type=simple
User=wso2am
Group=wso2am
EnvironmentFile=/etc/default/wso2am
LimitNOFILE=655360
WorkingDirectory=/srv/wso2am
Restart=on-failure
TimeoutSec=5min
IgnoreSIGPIPE=no
KillMode=control-group
GuessMainPID=no
ExecStart=/srv/wso2am/bin/api-manager.sh
ExecStop=/srv/wso2am/bin/api-manager.sh --stop
```

- Ajouter un utilisateur wso2am sur les instances

```
useradd wso2am
```

- Changez le proprietaire des dossier wso2

```
chown -R wso2am:wso2am /srv/
chown -R wso2am:wso2am /var/lib/wso2am
```
- Modifier l'address de connexion

```
Remplacer localhost par l'ip de votre vm dans:

/var/wso2am-4.4.0/repository/conf/deployment.toml

hostname = "IP_VM"
```

- configurer les règles selinux pour ce fichier  /srv/wso2am/bin/api-manager.sh

```
chcon system_u:object_r:bin_t:s0 /srv/wso2am/bin/api-manager.sh
```

- Reload le daemond et demarrer le service

```
systemctl daemon-reload
systemctl start  wso2am.service
```
# Key Manager configuration
[apim.key_manager]
service_url = "https://[control-plane-LB-host]/services/"
username = "$ref{super_admin.username}"
password = "$ref{super_admin.password}"

# Event Hub configurations
[apim.event_hub]
enable = true
username = "$ref{super_admin.username}"
password = "$ref{super_admin.password}"
service_url = "https://[control-plane-LB-host]/services/"
event_listening_endpoints = ["tcp://control-plane-1-host:5672", "tcp://control-plane-2-host:5672"]

