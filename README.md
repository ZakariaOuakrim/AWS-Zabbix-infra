# Infrastructure Cloud de Supervision Centralisée sous AWS avec Zabbix



---

## Introduction

### Objectif du Projet

Mettre en place une infrastructure de supervision centralisée sur AWS, basée sur Zabbix conteneurisé avec Docker, afin de surveiller un environnement hybride composé de serveurs **Linux** et **Windows**.


## Phase 1 : Configuration AWS

### Étape 1.1 : Connexion à AWS Academy

1. Connectez-vous à votre portail AWS Academy
2. Lancez le **Learner Lab**
3. Cliquez sur **AWS** (bouton vert) pour ouvrir la console


---

### Étape 1.2 : Création du VPC

1. Recherchez **VPC** dans la barre de recherche AWS
2. Cliquez sur **Your VPCs** → **Create VPC**
3. Configurez :

| Paramètre           | Valeur                |
| ------------------- | --------------------- |
| Resources to create | VPC only              |
| Name tag            | `VotreNom-Zabbix-VPC` |
| IPv4 CIDR block     | `10.0.0.0/16`         |
| IPv6 CIDR block     | No IPv6 CIDR block    |
| Tenancy             | Default               |

4. Cliquez sur **Create VPC**


---

### Étape 1.3 : Création du Subnet Public

1. Dans le menu VPC, cliquez sur **Subnets** → **Create subnet**
2. Configurez :

| Paramètre         | Valeur                             |
| ----------------- | ---------------------------------- |
| VPC ID            | Sélectionnez `VotreNom-Zabbix-VPC` |
| Subnet name       | `VotreNom-Public-Subnet`           |
| Availability Zone | `us-east-1a`                       |
| IPv4 CIDR block   | `10.0.1.0/24`                      |

3. Cliquez sur **Create subnet**

#### Activer l'auto-attribution d'IP publique :

1. Sélectionnez le subnet créé
2. Actions → **Edit subnet settings**
3. Cochez ✅ **Enable auto-assign public IPv4 address**
4. Sauvegardez

> 📸 **Figure 2** : Configuration du Subnet

---

### Étape 1.4 : Création de l'Internet Gateway

1. Dans le menu VPC → **Internet Gateways** → **Create internet gateway**
2. Name tag : `VotreNom-IGW`
3. Cliquez sur **Create internet gateway**
4. Sélectionnez l'IGW → Actions → **Attach to VPC**
5. Sélectionnez votre VPC et attachez

> 📸 **Figure 3** : Internet Gateway attaché au VPC

---

### Étape 1.5 : Configuration de la Table de Routage

1. Menu VPC → **Route Tables**
2. Sélectionnez la route table associée à votre VPC
3. Onglet **Routes** → **Edit routes** → **Add route**

| Destination | Target                              |
| ----------- | ----------------------------------- |
| `0.0.0.0/0` | Sélectionnez votre Internet Gateway |

4. Sauvegardez
5. Onglet **Subnet associations** → **Edit subnet associations**
6. Cochez votre subnet public et sauvegardez

> 📸 **Figure 4** : Table de routage configurée

---

### Étape 1.6 : Création des Security Groups

#### Security Group 1 : Serveur Zabbix

1. Menu EC2 → **Security Groups** → **Create security group**
2. Configurez :

| Paramètre           | Valeur                           |
| ------------------- | -------------------------------- |
| Security group name | `VotreNom-Zabbix-Server-SG`      |
| Description         | Security group for Zabbix Server |
| VPC                 | Sélectionnez votre VPC           |

3. **Inbound rules** → Add rule :

| Type       | Port Range | Source      | Description       |
| ---------- | ---------- | ----------- | ----------------- |
| SSH        | 22         | My IP       | SSH Access        |
| HTTP       | 80         | 0.0.0.0/0   | Zabbix Web        |
| HTTPS      | 443        | 0.0.0.0/0   | Zabbix Web Secure |
| Custom TCP | 10050      | 10.0.0.0/16 | Zabbix Agent      |
| Custom TCP | 10051      | 10.0.0.0/16 | Zabbix Trapper    |

4. **Create security group**

#### Security Group 2 : Clients Zabbix

1. Créez un nouveau security group : `VotreNom-Zabbix-Clients-SG`
2. **Inbound rules** :

| Type       | Port Range | Source                      | Description  |
| ---------- | ---------- | --------------------------- | ------------ |
| SSH        | 22         | My IP                       | SSH Linux    |
| RDP        | 3389       | My IP                       | RDP Windows  |
| Custom TCP | 10050      | `VotreNom-Zabbix-Server-SG` | Zabbix Agent |

> 📸 **Figure 5** : Security Groups configurés

---

## Phase 2 : Création des Instances EC2

### Étape 2.1 : Serveur Zabbix (Ubuntu)

1. Menu EC2 → **Instances** → **Launch instances**
2. Configurez :

| Paramètre               | Valeur                                    |
| ----------------------- | ----------------------------------------- |
| Name                    | `VotreNom-Zabbix-Server`                  |
| AMI                     | Ubuntu Server 22.04 LTS (64-bit x86)      |
| Instance type           | `t3.large`                                |
| Key pair                | Créez ou sélectionnez une paire existante |
| Network settings        | Edit                                      |
| → VPC                   | Votre VPC                                 |
| → Subnet                | Votre subnet public                       |
| → Auto-assign public IP | Enable                                    |
| → Security group        | `VotreNom-Zabbix-Server-SG`               |
| Storage                 | 30 GiB gp3                                |

3. **Launch instance**

> 📸 **Figure 6** : Instance Zabbix Server créée

---

### Étape 2.2 : Client Linux (Ubuntu)

| Paramètre      | Valeur                       |
| -------------- | ---------------------------- |
| Name           | `VotreNom-Client-Linux`      |
| AMI            | Ubuntu Server 22.04 LTS      |
| Instance type  | `t3.medium`                  |
| Key pair       | Même paire de clés           |
| Network        | Même VPC/Subnet              |
| Security group | `VotreNom-Zabbix-Clients-SG` |
| Storage        | 20 GiB gp3                   |

> 📸 **Figure 7** : Instance Client Linux créée

---

### Étape 2.3 : Client Windows

| Paramètre      | Valeur                             |
| -------------- | ---------------------------------- |
| Name           | `VotreNom-Client-Windows`          |
| AMI            | Microsoft Windows Server 2022 Base |
| Instance type  | `t3.large` (4 Go RAM minimum)      |
| Key pair       | Même paire de clés                 |
| Network        | Même VPC/Subnet                    |
| Security group | `VotreNom-Zabbix-Clients-SG`       |
| Storage        | 30 GiB gp3                         |

#### Récupérer le mot de passe Windows :

1. Attendez ~4 minutes après le lancement
2. Sélectionnez l'instance → Actions → **Security** → **Get Windows password**
3. Uploadez votre fichier .pem ou collez la clé privée
4. **Decrypt password**
5. 📝 Notez le mot de passe Administrator

> 📸 **Figure 8** : Instance Client Windows créée

---

### Étape 2.4 : Vue d'ensemble des Instances

Vérifiez que vos 3 instances sont en état **Running** :

| Instance                | Type      | État    | IP Publique | IP Privée |
| ----------------------- | --------- | ------- | ----------- | --------- |
| VotreNom-Zabbix-Server  | t3.large  | Running | x.x.x.x     | 10.0.1.x  |
| VotreNom-Client-Linux   | t3.medium | Running | x.x.x.x     | 10.0.1.x  |
| VotreNom-Client-Windows | t3.large  | Running | x.x.x.x     | 10.0.1.x  |

> 📸 **Figure 9** : Vue d'ensemble - 3 instances en état Running

---

## Phase 3 : Installation de Docker et Zabbix

### Étape 3.1 : Connexion au Serveur Zabbix

```bash
# Téléchargez votre fichier .pem et modifiez les permissions
chmod 400 votre-cle.pem

# Connexion SSH
ssh -i "votre-cle.pem" ubuntu@<IP-PUBLIQUE-ZABBIX-SERVER>
```

---

### Étape 3.2 : Installation de Docker

Exécutez ces commandes une par une :

```bash
# Mise à jour du système
sudo apt update && sudo apt upgrade -y

# Installation des dépendances
sudo apt install -y apt-transport-https ca-certificates curl software-properties-common gnupg lsb-release

# Ajout de la clé GPG Docker
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

# Ajout du dépôt Docker
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Installation de Docker
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Ajout de l'utilisateur au groupe docker (évite sudo)
sudo usermod -aG docker $USER

# Appliquer les changements de groupe (ou déconnectez/reconnectez)
newgrp docker

# Vérification de l'installation
docker --version
docker compose version
```

> 📸 **Figure 10** : Docker installé avec succès

---

### Étape 3.3 : Déploiement de Zabbix avec Docker Compose

#### Créer le répertoire et le fichier de configuration :

```bash
# Créer le répertoire
mkdir -p ~/zabbix
cd ~/zabbix

# Créer le fichier docker-compose.yml
nano docker-compose.yml
```

#### Contenu du fichier `docker-compose.yml` :

```yaml
version: "3.8"

services:
  # Base de données MySQL
  mysql-server:
    image: mysql:8.0
    container_name: zabbix-mysql
    restart: always
    environment:
      MYSQL_DATABASE: zabbix
      MYSQL_USER: zabbix
      MYSQL_PASSWORD: zabbix_pwd_secure
      MYSQL_ROOT_PASSWORD: root_pwd_secure
    command:
      - mysqld
      - --character-set-server=utf8mb4
      - --collation-server=utf8mb4_bin
      - --default-authentication-plugin=mysql_native_password
    volumes:
      - mysql_data:/var/lib/mysql
    networks:
      - zabbix-net
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 10s
      timeout: 5s
      retries: 5

  # Serveur Zabbix
  zabbix-server:
    image: zabbix/zabbix-server-mysql:alpine-7.0-latest
    container_name: zabbix-server
    restart: always
    environment:
      DB_SERVER_HOST: mysql-server
      MYSQL_DATABASE: zabbix
      MYSQL_USER: zabbix
      MYSQL_PASSWORD: zabbix_pwd_secure
      MYSQL_ROOT_PASSWORD: root_pwd_secure
      ZBX_CACHESIZE: 128M
    ports:
      - "10051:10051"
    depends_on:
      mysql-server:
        condition: service_healthy
    networks:
      - zabbix-net
    volumes:
      - zabbix_server_data:/var/lib/zabbix

  # Interface Web Zabbix
  zabbix-web:
    image: zabbix/zabbix-web-nginx-mysql:alpine-7.0-latest
    container_name: zabbix-web
    restart: always
    environment:
      ZBX_SERVER_HOST: zabbix-server
      DB_SERVER_HOST: mysql-server
      MYSQL_DATABASE: zabbix
      MYSQL_USER: zabbix
      MYSQL_PASSWORD: zabbix_pwd_secure
      PHP_TZ: Europe/Paris
    ports:
      - "80:8080"
      - "443:8443"
    depends_on:
      - zabbix-server
    networks:
      - zabbix-net

  # Agent Zabbix pour le serveur lui-même
  zabbix-agent:
    image: zabbix/zabbix-agent:alpine-7.0-latest
    container_name: zabbix-agent
    restart: always
    environment:
      ZBX_SERVER_HOST: zabbix-server
      ZBX_HOSTNAME: "Zabbix-Server"
    privileged: true
    depends_on:
      - zabbix-server
    networks:
      - zabbix-net

volumes:
  mysql_data:
  zabbix_server_data:

networks:
  zabbix-net:
    driver: bridge
```

**Pour sauvegarder dans nano** : `Ctrl+O`, `Enter`, `Ctrl+X`

---

### Étape 3.4 : Lancement des Conteneurs

```bash
# Lancer les conteneurs en arrière-plan
docker compose up -d

# Vérifier que tous les conteneurs sont en cours d'exécution
docker compose ps

# Voir les logs (optionnel, pour déboguer)
docker compose logs -f
```

Attendez **2-3 minutes** que MySQL initialise la base de données.

> 📸 **Figure 11** : Conteneurs Zabbix en cours d'exécution (docker compose ps)

---

### Étape 3.5 : Accès à l'Interface Web Zabbix

1. Ouvrez votre navigateur
2. Allez à : `http://<IP-PUBLIQUE-ZABBIX-SERVER>`
3. Connectez-vous avec :
   - **Username** : `Admin`
   - **Password** : `zabbix`

> ⚠️ **Important** : Changez le mot de passe par défaut après la première connexion !

> 📸 **Figure 12** : Interface de connexion Zabbix réussie

> 📸 **Figure 13** : Dashboard Zabbix après connexion

---

## Phase 4 : Configuration des Agents

### Étape 4.1 : Agent Zabbix sur Client Linux

#### Connexion au Client Linux :

```bash
# Depuis votre machine locale
ssh -i "votre-cle.pem" ubuntu@<IP-PUBLIQUE-CLIENT-LINUX>
```

#### Installation de l'Agent Zabbix :

```bash
# Télécharger et installer le dépôt Zabbix
wget https://repo.zabbix.com/zabbix/7.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_latest_7.0+ubuntu22.04_all.deb
sudo dpkg -i zabbix-release_latest_7.0+ubuntu22.04_all.deb
sudo apt update

# Installer l'agent Zabbix
sudo apt install -y zabbix-agent

# Sauvegarder la configuration originale
sudo cp /etc/zabbix/zabbix_agentd.conf /etc/zabbix/zabbix_agentd.conf.backup
```

#### Configuration de l'Agent :

```bash
# Éditer le fichier de configuration
sudo nano /etc/zabbix/zabbix_agentd.conf
```

**Modifiez ces lignes** (recherchez avec `Ctrl+W`) :

```ini
# Ligne ~117 - Adresse du serveur Zabbix (IP PRIVÉE)
Server=10.0.1.X  # Remplacez par l'IP PRIVÉE du serveur Zabbix

# Ligne ~168 - Adresse du serveur pour les checks actifs
ServerActive=10.0.1.X  # Même IP PRIVÉE

# Ligne ~179 - Nom de l'hôte (doit correspondre exactement dans Zabbix)
Hostname=Client-Linux-VotreNom
```

> ⚠️ **Utilisez l'IP PRIVÉE** du serveur Zabbix (ex: 10.0.1.45), pas l'IP publique !

#### Démarrage de l'Agent :

```bash
# Redémarrer l'agent
sudo systemctl restart zabbix-agent

# Activer au démarrage
sudo systemctl enable zabbix-agent

# Vérifier le statut
sudo systemctl status zabbix-agent
```

> 📸 **Figure 14** : Configuration du fichier zabbix_agentd.conf (Linux)

> 📸 **Figure 15** : Agent Zabbix actif sur Linux

---

### Étape 4.2 : Agent Zabbix sur Client Windows

#### Connexion RDP au Client Windows :

1. Utilisez **Remote Desktop Connection** (Windows) ou **Remmina** (Linux)
2. Adresse : `<IP-PUBLIQUE-CLIENT-WINDOWS>`
3. Utilisateur : `Administrator`
4. Mot de passe : Celui récupéré précédemment

#### Téléchargement de l'Agent :

1. Ouvrez Internet Explorer/Edge sur le serveur Windows
2. Allez à : https://www.zabbix.com/download_agents
3. Sélectionnez :
   - **Zabbix version** : 7.0 LTS
   - **OS Distribution** : Windows
   - **Architecture** : amd64
   - **Encryption** : OpenSSL
   - **Packaging** : MSI
4. Téléchargez le fichier `.msi`

#### Installation de l'Agent :

1. Double-cliquez sur le fichier `.msi` téléchargé
2. Suivez l'assistant d'installation :

| Étape                             | Configuration                     |
| --------------------------------- | --------------------------------- |
| Host name                         | `Client-Windows-VotreNom`         |
| Zabbix server IP/DNS              | `10.0.1.X` (IP PRIVÉE du serveur) |
| Agent listen port                 | `10050`                           |
| Server or Proxy for active checks | `10.0.1.X`                        |

3. Terminez l'installation

#### Vérification du Service :

1. Ouvrez **Services** (services.msc)
2. Vérifiez que **Zabbix Agent** est en état **Running**

> 📸 **Figure 16** : Installation de l'Agent Windows

> 📸 **Figure 17** : Service Zabbix Agent actif sur Windows

---

## Phase 5 : Configuration du Monitoring

### Étape 5.1 : Ajout des Hôtes dans Zabbix

#### Ajouter le Client Linux :

1. Connectez-vous à l'interface Zabbix
2. Allez dans **Data collection** → **Hosts** → **Create host**
3. Onglet **Host** :

| Paramètre    | Valeur                  |
| ------------ | ----------------------- |
| Host name    | `Client-Linux-VotreNom` |
| Visible name | Client Linux            |
| Host groups  | Linux servers           |

4. Section **Interfaces** → **Add** → **Agent** :

| Paramètre  | Valeur                                 |
| ---------- | -------------------------------------- |
| IP address | `10.0.1.X` (IP PRIVÉE du client Linux) |
| Port       | `10050`                                |

5. Onglet **Templates** :

   - Cliquez sur **Select**
   - Cherchez et ajoutez : `Linux by Zabbix agent`

6. Cliquez sur **Add**

#### Ajouter le Client Windows :

Répétez la procédure avec :

| Paramètre    | Valeur                                   |
| ------------ | ---------------------------------------- |
| Host name    | `Client-Windows-VotreNom`                |
| Visible name | Client Windows                           |
| Host groups  | Windows servers                          |
| IP address   | `10.0.1.X` (IP PRIVÉE du client Windows) |
| Template     | `Windows by Zabbix agent`                |

> 📸 **Figure 18** : Ajout des hôtes dans Zabbix

---

### Étape 5.2 : Vérification de la Connectivité

1. Allez dans **Data collection** → **Hosts**
2. Attendez 1-2 minutes
3. Vérifiez que l'icône **ZBX** est **vert** ✅ pour chaque hôte

| Statut       | Signification                 |
| ------------ | ----------------------------- |
| 🟢 ZBX vert  | Agent connecté et fonctionnel |
| 🔴 ZBX rouge | Problème de connexion         |
| ⚪ ZBX gris  | En attente de données         |

> 📸 **Figure 19** : Statut "Vert" (ZBX) des deux clients

---

### Étape 5.3 : Visualisation des Données

1. Allez dans **Monitoring** → **Latest data**
2. Filtrez par hôte (Client-Linux ou Client-Windows)
3. Observez les métriques collectées :
   - CPU utilization
   - Memory usage
   - Disk space
   - Network traffic

> 📸 **Figure 20** : Latest data avec métriques temps réel

---

### Étape 5.4 : Création de Graphiques

1. Allez dans **Monitoring** → **Hosts**
2. Cliquez sur le nom d'un hôte
3. Cliquez sur **Graphs**
4. Sélectionnez un graphique (ex: CPU utilization)

> 📸 **Figure 21** : Graphique CPU d'un client

---


#### Sur le Client Linux :

```bash
# Générer une charge CPU
stress --cpu 4 --timeout 60s

# OU si stress n'est pas installé
sudo apt install -y stress
stress --cpu 4 --timeout 60s
```

---


### Étape 6.2 : Contenu du README.md

```markdown
# 🖥️ Infrastructure de Supervision Centralisée - AWS & Zabbix

## Description

Ce projet déploie une infrastructure de monitoring centralisée sur AWS
utilisant Zabbix conteneurisé pour surveiller un parc hybride Linux/Windows.


## Déploiement Rapide

### Prérequis

- Compte AWS Academy/Learner Lab
- Région : us-east-1 (N. Virginia)

### Installation

1. Cloner ce dépôt
2. Suivre le guide dans [docs/installation-guide.md](docs/installation-guide.md)



## 👤 Auteur

- **Nom** : Zakaria Ouakrim
- **Encadrant** : Prof. Azeddine KHIAT
- **Année** : 2025/2026
- **Filière** : II-BDCC


---

## Dépannage

### Problème : L'agent Zabbix ne se connecte pas

**Symptôme** : ZBX rouge dans la liste des hôtes

**Solutions** :

1. Vérifiez que le Security Group autorise le port 10050
2. Vérifiez l'IP du serveur dans la config de l'agent
3. Vérifiez que l'agent est démarré : `sudo systemctl status zabbix-agent`
4. Vérifiez les logs : `sudo tail -f /var/log/zabbix/zabbix_agentd.log`

### Problème : Interface Web Zabbix inaccessible

**Symptôme** : Page ne se charge pas

**Solutions** :

1. Vérifiez que les conteneurs sont en cours : `docker compose ps`
2. Vérifiez le Security Group (port 80)
3. Vérifiez les logs : `docker compose logs zabbix-web`




---



---

