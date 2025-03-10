# CONFIGURATION WSO2 APK

## 1. Configuration de la base de données MySQL et des utilisateurs

Pour remplacer la base de données H2 par défaut par MySQL ou MariaDB, suivez ces étapes :

- Se connecter à MySQL
  
  ```bash
  mysql -h <IP_HOTE_MYSQL> -u <NOM_UTILISATEUR> -p
  ```

  Entrez ensuite le mot de passe lorsque vous y êtes invité.

- Créer la base de données

  ```sql
  CREATE DATABASE <NOM_BD_APIM>;
  ```

  **Note** : Il est recommandé d'utiliser une collation de base de données sensible à la casse pour éviter les problèmes lors de la gestion des API dans MySQL.

- Creer les utilisateurs

  ```sql
  CREATE USER 'apimadmin'@'%' IDENTIFIED BY 'apimadmin';
  CREATE USER 'sharedadmin'@'%' IDENTIFIED BY 'sharedadmin';
  ```

- Attribuer des permissions

  ```sql
  GRANT ALL ON apim_db.* TO 'apimadmin'@'%' IDENTIFIED BY 'apimadmin';
  GRANT ALL ON shared_db.* TO 'sharedadmin'@'%' IDENTIFIED BY 'sharedadmin';
  FLUSH PRIVILEGES;
  ```

## 2. Exécution des scripts SQL

Sur le serveur wso2

- Telecharger le fichier compressé wso2 am 4.3.0

```bash
   curl -LO https://archive.smile.ci/wso2/wso2am-4.3.0.zip
```

- Decompresser dans le repertoire /tmp

```bash
   unzip wso2am-4.3.0.zip -d /tmp/
```

Exécuter les scripts SQL pour créer les tables

- Pour créer les tables dans la base de données **shared_db** (WSO2_SHARED_DB), utilisez la commande suivante :

```bash
    mysql -u sharedadmin -p -h @ip -Dshared_db < '/tmp/wso2am-4.3.0/dbscripts/mysql.sql';
```

- Pour la base de données **apim_db** (WSO2AM_DB), exécutez :

```bash
  mysql -u apimadmin -p -h @ip -Dapim_db < '/tmp/wso2am-4.3.0/dbscripts/apimgt/mysql.sql';
```

## 3. Modification de la definition de WSO2 API Manager

- Modifier la configuration des bases de données dans le fichier *apim-values.yaml* téléchargé au paravant avec les éléments suivants

```
database:
  type: "mysql"
  jdbc:
  driver: "com.mysql.cj.jdbc.Driver"

  apim_db:
    url: "jdbc:mysql://ip-database:3306/apim_db?useSSL=false&autoReconnect=true"
    username: "apimadmin"
    password: "apimadmin"
    poolParameters:
    defaultAutoCommit: false
    testOnBorrow: true
    testWhileIdle: true
    validationInterval: 30000
    maxActive: 100
    maxWait: 60000
    
    minIdle: 5

  shared_db:
    url: "jdbc:mysql://ip-database:3306/shared_db?useSSL=false&autoReconnect=true"
    username: "sharedadmin"
    password: "sharedadmin"
    poolParameters:
    defaultAutoCommit: false
    testOnBorrow: true
    testWhileIdle: true
    validationInterval: 30000
    maxActive: 100
    maxWait: 60000
    minIdle: 5
```

- Ensuite, faites helm upgrade --install apim wso2apim/wso2am-cp --version 4.3.0 -f apim-values.yaml -n apk
