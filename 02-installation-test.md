# Installation et configuration de WSO2 Micro Integrator et WSO2 Integration Control Plane

## 1. Introduction

Cette documentation décrit le processus d'installation et de configuration de WSO2 Micro Integrator (WSO2MI) et WSO2 Integration Control Plane (WSO2ICP) sur un serveur Rocky Linux .

- Prerequis:
Installer Java avec :

```bash
dnf install java-11-openjdk -y
```

Obtenir le home de java avec :

```bash
readlink -f $(which java)
```

## 2. Installation de WSO2 Micro Integrator

### 2.1 Téléchargement et extraction

Téléchargez l'archive de WSO2 Micro Integrator et extrayez-la dans un répertoire approprié. [https://wso2.com/integrator/micro-integrator/]

```bash
unzip wso2mi-4.4.0.zip -d /root/
```

Ajuster la conf du fichier  <MI-HOME>/conf/deployment.toml comme suite :

```ini
[server]
hostname = "@ip-srv" 

[dashboard_config]
dashboard_url = "https://@ip-srv:9743/dashboard/api/"

```

### 2.2 Création du service systemd pour WSO2MI

Créez le fichier `/etc/systemd/system/wso2mi.service` et ajoutez le contenu suivant :

```ini
[Unit]
Description=WSO2 Micro Integrator
After=network.target

[Service]
Environment="JAVA_HOME=/usr/lib/jvm/java-11-openjdk-11.0.25.0.9-3.el9.x86_64"
Environment="PATH=/usr/lib/jvm/java-11-openjdk-11.0.25.0.9-3.el9.x86_64/bin:/usr/bin:/bin"
Type=simple
User=root
Group=root
WorkingDirectory=/root/wso2mi-4.4.0
ExecStart=sh /root/wso2mi-4.4.0/bin/micro-integrator.sh
ExecStop=sh /root/wso2mi-4.4.0/bin/micro-integrator.sh stop
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```

### 2.3 Activation et démarrage du service

Rechargez systemd, activez et démarrez le service :

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now wso2mi
sudo systemctl status wso2mi
```

## 3. Installation de WSO2 Integration Control Plane

### 3.1 Téléchargement et extraction

Téléchargez l'archive de WSO2 Integration Control Plane [https://wso2.com/integrator/integration-control-plane/] et extrayez-la :

```bash
unzip wso2-integration-control-plane-1.0.0.zip -d /root/
```

### 3.2 Création du service systemd pour WSO2ICP

Créez le fichier `/etc/systemd/system/wso2icp.service` et ajoutez le contenu suivant :

```ini
[Unit]
Description=WSO2 Integration Control Plane
Requires=wso2mi.service
After=wso2mi.service

[Service]
Environment="JAVA_HOME=/usr/lib/jvm/java-11-openjdk-11.0.25.0.9-3.el9.x86_64"
Environment="PATH=${JAVA_HOME}:/usr/bin:/bin"
Type=simple
User=root
Group=root
WorkingDirectory=/root/wso2-integration-control-plane-1.0.0/
ExecStart=/bin/bash /root/wso2-integration-control-plane-1.0.0/bin/dashboard.sh
ExecStop=/bin/bash /root/wso2-integration-control-plane-1.0.0/bin/dashboard.sh stop
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```

### 3.3 Activation et démarrage du service

Rechargez systemd, activez et démarrez le service :

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now wso2icp
sudo systemctl status wso2icp
```

## 4. Configuration de la connexion entre WSO2MI et WSO2ICP

### 4.1 Modification du fichier de configuration

Éditez la configuration du Micro Integrator pour pointer vers l'IP du serveur dans le fichier <MI-ICP-HOME>/conf/deployment.toml
:

```ini
[server]
hostname = "@ip"

[dashboard_config]
dashboard_url = "https://@ip:9743/dashboard/api/"
heartbeat_interval = 5
group_id = "mi_dev"
node_id = "node-01"
```

Ensuite redemarrez le service

```bash
sudo systemctl restart wso2icp
```

## 5. Vérification du bon fonctionnement

Vérifiez que les services sont bien démarrés et fonctionnent correctement :

```bash
sudo systemctl status wso2mi
sudo systemctl status wso2icp
```

Ouvrez les ports

```bash
firewall-cmd --add-port=9743/{udp,tcp} --permanent 
firewall-cmd --reload
```

Testez l'accès à l'interface web de WSO2ICP via :

```
https://ip:9743
```

![Dashboard WSO2 ICP](pictures/wso2-icp-dashboard.png)