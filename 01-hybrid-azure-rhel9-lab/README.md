# 🏢 Infrastructure Hybride Azure + Linux RHEL9
### Lab Professionnel Entreprise — Portfolio GitHub

<div align="center">

![Azure](https://img.shields.io/badge/Azure-AZ--104%20Certified-0078D4?style=for-the-badge&logo=microsoftazure&logoColor=white)
![RHEL9](https://img.shields.io/badge/Linux-RHEL9-DC2626?style=for-the-badge&logo=redhat&logoColor=white)
![Ansible](https://img.shields.io/badge/Ansible-Automatisation-1A1918?style=for-the-badge&logo=ansible&logoColor=white)
![SSH](https://img.shields.io/badge/SSH-Sécurisé-16A34A?style=for-the-badge&logo=openssh&logoColor=white)
![SELinux](https://img.shields.io/badge/SELinux-Enforcing-7C3AED?style=for-the-badge&logo=linux&logoColor=white)

**Auteur : Serge TOGNON — AZ-104 Certified | Préparation RHCSA**

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Serge%20TOGNON-0077B5?style=flat&logo=linkedin)](https://linkedin.com)
![Status](https://img.shields.io/badge/Status-En%20cours-orange?style=flat)
![Level](https://img.shields.io/badge/Niveau-Junior%20Cloud%2FDevOps-blue?style=flat)

</div>

---

## 📋 Table des Matières

- [Présentation](#-présentation-du-projet)
- [Contexte Entreprise](#-contexte-entreprise)
- [Architecture](#-architecture--topologie)
- [Prérequis](#-prérequis)
- [Configuration Réseau](#-configuration-réseau-linux)
- [Configuration SSH](#-configuration-ssh-sécurisée)
- [Sécurité Linux](#-sécurité-linux--firewalld--selinux)
- [Déploiement Nginx](#-déploiement-nginx)
- [Automatisation Ansible](#-automatisation-ansible)
- [Infrastructure Azure](#-infrastructure-azure)
- [Monitoring](#-monitoring--supervision)
- [Troubleshooting](#-troubleshooting)
- [Checklists](#-checklists-certification)
- [Captures d'écran](#-captures-décran-requises)
- [Compétences développées](#-compétences-développées)

---

## 🎯 Présentation du Projet

> Ce lab simule une **infrastructure hybride d'entreprise** combinant des VM Linux RHEL9 locales avec des ressources Azure Cloud. Il couvre l'administration système, la sécurité, l'automatisation Ansible et la supervision — orienté certifications **AZ-104** et **RHCSA**.

### Objectifs Pédagogiques

| Domaine | Compétences Acquises |
|---|---|
| ☁️ **Azure Cloud** | Resource Group, VNet, NSG, VM, Monitor, Public IP |
| 🐧 **Linux RHEL9** | nmcli, SSH, firewalld, SELinux, systemctl, journalctl |
| ⚙️ **Services** | Nginx, Apache, chronyd, rsyslog |
| 🤖 **Automatisation** | Ansible inventory, playbooks, variables, handlers |
| 🔒 **Sécurité** | Clés SSH, désactivation root, permissions Linux |
| 📊 **Monitoring** | CPU, RAM, disque, logs système, alertes Azure |
| 🌐 **Réseau** | Topologie hybride, connectivité Azure, diagnostics |

---

## 🏢 Contexte Entreprise

**TechCorp SA** souhaite déployer une infrastructure hybride sécurisée pour héberger son application web interne. L'équipe IT doit :

- ✅ Sécuriser l'accès SSH aux serveurs Linux
- ✅ Automatiser le déploiement via Ansible
- ✅ Superviser les services et la performance
- ✅ Appliquer les bonnes pratiques de sécurité Linux (SELinux, firewalld)
- ✅ Préparer une migration progressive vers Azure


## 🏗️ Architecture & Topologie

### Diagramme d'Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                  INFRASTRUCTURE HYBRIDE                          │
│              Azure Cloud + Linux RHEL9                           │
└─────────────────────────────────────────────────────────────────┘

  ┌──────────────────────┐        ┌───────────────────────┐
  │   VM CLIENT (RHEL9)  │        │   VM SERVER (RHEL9)   │
  │  client.lab.local    │        │  server.lab.local     │
  │  192.168.10.1/24     │──SSH──▶│  192.168.10.2/24      │
  │                      │        │                       │
  │  • Ansible Ctrl Node │─Ansible▶  • Nginx Web Server   │
  │  • SSH Client        │        │  • Managed Node       │
  │  • Admin Workstation │        │  • SELinux Enforcing  │
  └──────────┬───────────┘        └──────────┬────────────┘
             │                               │
             └───────────┬───────────────────┘
                         │  Réseau Local 192.168.10.0/24
                         │
              ┌──────────▼────────────────────────────┐
              │           AZURE CLOUD                  │
              │  Resource Group: rg-lab-hybride        │
              │                                        │
              │  ┌─────────────────────────────────┐  │
              │  │  VNet: vnet-lab (10.0.0.0/16)   │  │
              │  │                                 │  │
              │  │  ┌──────────────────────────┐   │  │
              │  │  │  Subnet: 10.0.1.0/24     │   │  │
              │  │  │                          │   │  │
              │  │  │  ┌────────────────────┐  │   │  │
              │  │  │  │  Azure VM Linux    │  │   │  │
              │  │  │  │  Ubuntu 22.04      │  │   │  │
              │  │  │  │  Public IP + NSG   │  │   │  │
              │  │  │  └────────────────────┘  │   │  │
              │  │  └──────────────────────────┘   │  │
              │  └─────────────────────────────────┘  │
              │                                        │
              │  • Azure Monitor / Log Analytics       │
              │  • NSG Rules (Port 22, 80, 443)        │
              └────────────────────────────────────────┘
```

### Environnement Local

| VM | Configuration |
|---|---|
| **client.lab.local** | IP: `192.168.10.1/24` · RHEL9 · Ansible Control Node · SSH Client |
| **server.lab.local** | IP: `192.168.10.2/24` · RHEL9 · Nginx/Apache · Ansible Managed Node |
| **Réseau local** | `192.168.10.0/24` · Passerelle: `192.168.10.254` |

### Infrastructure Azure

| Ressource Azure | Configuration |
|---|---|
| **Resource Group** | `rg-lab-hybride` · East US |
| **Virtual Network** | `vnet-lab` · `10.0.0.0/16` |
| **Subnet** | `subnet-web` · `10.0.1.0/24` |
| **NSG** | `nsg-lab` · Règles : SSH (22), HTTP (80), HTTPS (443) |
| **VM Linux** | `vm-azure-lab` · Ubuntu 22.04 · Standard_B1s |
| **Public IP** | `pip-lab-vm` · Allocation Dynamique |
| **Azure Monitor** | Log Analytics Workspace · Alertes CPU/RAM |

---

## ⚙️ Prérequis

### Outils Requis

| Outil | Version / Notes |
|---|---|
| **RHEL9** | Red Hat Enterprise Linux 9.x (ou AlmaLinux 9 / Rocky Linux 9) |
| **Azure CLI** | `az --version` ≥ 2.50 |
| **Ansible** | `ansible --version` ≥ 2.14 |
| **SSH Client** | OpenSSH inclus dans RHEL9 |
| **Git** | `git --version` ≥ 2.39 |
| **Compte Azure** | Abonnement actif ou Azure Free Tier |

### Installation Azure CLI (RHEL9)

```bash
# Importer la clé Microsoft GPG
sudo rpm --import https://packages.microsoft.com/keys/microsoft.asc

# Ajouter le repo Azure CLI
sudo dnf install -y https://packages.microsoft.com/config/rhel/9.0/packages-microsoft-prod.rpm

# Installer Azure CLI
sudo dnf install -y azure-cli

# Vérification
az --version
```

> 📝 **Explication** : `rpm --import` importe la clé GPG pour vérifier l'authenticité des paquets Microsoft. `dnf install` installe azure-cli et toutes ses dépendances Python.

### Installation Ansible (RHEL9)

```bash
# Activer EPEL (Extra Packages for Enterprise Linux)
sudo dnf install -y epel-release

# Installer Ansible
sudo dnf install -y ansible

# Vérification complète
ansible --version
ansible-galaxy collection list

# Installer la collection Azure pour Ansible
ansible-galaxy collection install azure.azcollection
```

> ⚠️ **Attention** : EPEL doit être activé avant l'installation d'Ansible sur RHEL9. Sans EPEL, le paquet `ansible` n'est pas disponible dans les repos standard Red Hat.

---

## 🌐 Configuration Réseau Linux

### Vérification de l'état réseau

```bash
# Lister toutes les connexions réseau
nmcli connection show

# Afficher les interfaces et leurs adresses IP
ip addr show

# Afficher la table de routage
ip route show

# Vérifier le nom d'hôte
hostnamectl
```

> 📝 **Explication** : `nmcli` = NetworkManager Command Line Interface, outil principal de gestion réseau sous RHEL9. `ip addr show` affiche les adresses IP de toutes les interfaces.

### Configuration IP statique (VM Client)

```bash
# Identifier le nom de l'interface (ex: enp0s3, enp0s8)
nmcli device status

# Modifier la connexion existante pour une IP statique
sudo nmcli connection modify enp0s8 \
  ipv4.method manual \
  ipv4.addresses 192.168.10.1/24 \
  ipv4.gateway 192.168.10.254 \
  ipv4.dns "8.8.8.8 8.8.4.4" \
  connection.autoconnect yes

# Appliquer les changements
sudo nmcli connection up enp0s8

# Vérification
ip addr show enp0s8
ping -c 4 192.168.10.2
```

| Option | Description |
|---|---|
| `ipv4.method manual` | Désactive DHCP — configure une IP fixe |
| `ipv4.addresses` | Définit l'adresse IP avec masque en notation CIDR |
| `connection.autoconnect yes` | Active la connexion au démarrage |
| `nmcli connection up` | Applique immédiatement la configuration |

### Configuration hostname

```bash
# Sur VM Client
sudo hostnamectl set-hostname client.lab.local

# Sur VM Server
sudo hostnamectl set-hostname server.lab.local

# Ajouter les entrées dans /etc/hosts (sur les deux VMs)
sudo tee -a /etc/hosts << 'EOF'
192.168.10.1  client.lab.local client
192.168.10.2  server.lab.local server
EOF

# Vérification
hostnamectl
cat /etc/hosts
ping -c 2 server.lab.local
```

> 📸 **Capture** : `capture-01-ip-client.png` — Exécuter `ip addr show` et `hostnamectl` sur client.lab.local — l'IP `192.168.10.1/24` et le hostname doivent apparaître clairement.

### Commandes de Diagnostic Réseau

| Commande | Description | Usage |
|---|---|---|
| `ping -c 4 <host>` | Test de connectivité ICMP | Vérifier la joignabilité |
| `traceroute <host>` | Tracer le chemin réseau | Diagnostiquer les sauts |
| `ss -tuln` | Ports ouverts en écoute | Vérifier les services actifs |
| `nmap -sV <host>` | Scanner les ports | Audit de sécurité réseau |
| `curl -I http://<host>` | Tester HTTP | Vérifier un service web |
| `dig <hostname>` | Résolution DNS | Diagnostiquer le DNS |
| `ip route get <IP>` | Route utilisée vers une IP | Comprendre le routage |
| `tcpdump -i ens33` | Capturer le trafic réseau | Analyse réseau avancée |

---

## 🔐 Configuration SSH Sécurisée

### Génération des Clés SSH

```bash
# Sur VM Client — Générer une paire de clés ED25519 (recommandé)
ssh-keygen -t ed25519 -C "serge.tognon@techcorp.sa" -f ~/.ssh/id_ed25519_lab

# Options :
#   -t ed25519  : algorithme moderne et sécurisé (préféré à RSA)
#   -C          : commentaire pour identifier la clé
#   -f          : fichier de destination de la clé

# Vérification des clés générées
ls -la ~/.ssh/
cat ~/.ssh/id_ed25519_lab.pub

# Définir les permissions correctes
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_ed25519_lab
chmod 644 ~/.ssh/id_ed25519_lab.pub
```

> 🔒 **Sécurité** : ED25519 est l'algorithme de clé SSH le plus sécurisé et le plus rapide en 2024. Préféré à RSA-4096 pour les nouvelles infrastructures. La passphrase est fortement recommandée en production.

### Déploiement de la Clé Publique

```bash
# Méthode 1 : ssh-copy-id (recommandée)
ssh-copy-id -i ~/.ssh/id_ed25519_lab.pub adminuser@192.168.10.2

# Méthode 2 : Manuelle
cat ~/.ssh/id_ed25519_lab.pub | ssh adminuser@192.168.10.2 \
  'mkdir -p ~/.ssh && chmod 700 ~/.ssh && \
   cat >> ~/.ssh/authorized_keys && \
   chmod 600 ~/.ssh/authorized_keys'

# Tester la connexion par clé
ssh -i ~/.ssh/id_ed25519_lab adminuser@server.lab.local
```

### Durcissement SSH — sshd_config

> ⚠️ **Critique** : La désactivation du login root et l'authentification par clé uniquement sont des exigences RHCSA et CIS Benchmark niveau 1.

```bash
# Sauvegarder la configuration originale
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak

# Créer le fichier de configuration sécurisée
sudo tee /etc/ssh/sshd_config.d/99-security.conf << 'EOF'
# ═══════════════════════════════════════════
# SSHD Security Configuration — TechCorp SA
# Auteur: Serge TOGNON · Date: 2025
# ═══════════════════════════════════════════

# Désactiver login root
PermitRootLogin no

# Authentification par clé uniquement
PasswordAuthentication no
PubkeyAuthentication yes

# Limiter les tentatives de connexion
MaxAuthTries 3
MaxSessions 5

# Délai de connexion
LoginGraceTime 30

# Interdire les comptes vides
PermitEmptyPasswords no

# X11 Forwarding désactivé
X11Forwarding no

# Banner de connexion
Banner /etc/ssh/banner.txt
EOF

# Créer le banner de connexion
sudo tee /etc/ssh/banner.txt << 'EOF'
╔══════════════════════════════════════════════════════╗
║     SYSTÈME INFORMATIQUE TECHCORP SA                ║
║     ACCÈS RÉSERVÉ AU PERSONNEL AUTORISÉ             ║
║     Toutes connexions sont enregistrées et          ║
║     peuvent être auditées à tout moment.            ║
╚══════════════════════════════════════════════════════╝
EOF

# Vérifier la syntaxe
sudo sshd -t

# Redémarrer SSH
sudo systemctl restart sshd
sudo systemctl status sshd
```

### Configuration SSH Client (~/.ssh/config)

```bash
tee ~/.ssh/config << 'EOF'
# Configuration SSH — TechCorp SA Lab

Host server
    HostName 192.168.10.2
    User adminuser
    IdentityFile ~/.ssh/id_ed25519_lab
    Port 22
    StrictHostKeyChecking yes
    ServerAliveInterval 60
    ServerAliveCountMax 3

Host azure-vm
    HostName <PUBLIC_IP_AZURE>
    User azureuser
    IdentityFile ~/.ssh/id_ed25519_lab
    Port 22
EOF

chmod 600 ~/.ssh/config

# Connexion simplifiée
ssh server
```

> 📸 **Capture** : `capture-02-ssh-connection.png` — Connexion SSH réussie avec affichage du banner + connexion établie par clé sans demande de mot de passe.

---

## 🔒 Sécurité Linux — Firewalld & SELinux

### Configuration Firewalld

```bash
# Vérifier l'état de firewalld
sudo systemctl status firewalld
sudo firewall-cmd --state

# Afficher la zone active et les règles
sudo firewall-cmd --get-active-zones
sudo firewall-cmd --list-all

# ─── Sur VM Server ────────────────────────────────────────

# Autoriser SSH, HTTP et HTTPS
sudo firewall-cmd --permanent --add-service=ssh
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https

# Appliquer les règles
sudo firewall-cmd --reload

# Vérification finale
sudo firewall-cmd --list-all
sudo firewall-cmd --list-services

# Vérifier les ports ouverts
ss -tuln | grep -E '22|80|443'
```

| Option | Description |
|---|---|
| `--permanent` | Rend la règle persistante après redémarrage |
| `--reload` | Recharge les règles sans couper les connexions actives |
| `--list-all` | Affiche toutes les règles de la zone active |
| `--add-service` | Autorise un service par nom (utilise /etc/firewalld/services/) |

### Configuration SELinux

```bash
# Vérifier l'état SELinux
getenforce
sestatus

# Résultat attendu : Enforcing

# Activer SELinux en mode Enforcing (permanent)
sudo setenforce 1
sudo sed -i 's/^SELINUX=.*/SELINUX=enforcing/' /etc/selinux/config

# Vérifier le contexte SELinux des fichiers web
ls -laZ /var/www/html/

# Corriger le contexte SELinux pour Nginx/Apache (méthode recommandée)
sudo restorecon -Rv /var/www/html/

# Autoriser Nginx à se connecter au réseau si nécessaire
sudo setsebool -P httpd_can_network_connect 1

# Lister les booleans SELinux pour httpd
getsebool -a | grep httpd

# Audit des messages SELinux (blocages récents)
sudo ausearch -m avc --start recent
sudo audit2allow -a
```

> 📸 **Capture** : `capture-05-selinux-enforcing.png` — `sestatus` affichant `SELinux status: enabled` et `Current mode: enforcing`.

### Gestion des Utilisateurs et Permissions

```bash
# Créer l'utilisateur d'administration
sudo useradd -m -s /bin/bash -c 'Serge TOGNON Admin' adminuser
sudo passwd adminuser

# Ajouter au groupe wheel (sudo) et webadmin
sudo usermod -aG wheel adminuser
sudo groupadd webadmin
sudo usermod -aG webadmin adminuser

# Configurer les permissions des répertoires web
sudo chown -R root:webadmin /var/www/html
sudo chmod -R 2775 /var/www/html
# 2 = setgid (nouveaux fichiers héritent du groupe)
# 7 = rwx propriétaire | 7 = rwx groupe | 5 = r-x autres

# Vérification
ls -la /var/www/html
id adminuser
groups adminuser
```

---

## 🌍 Déploiement Nginx

### Installation et Configuration

```bash
# Sur VM Server — Installer Nginx
sudo dnf install -y nginx

# Démarrer et activer Nginx
sudo systemctl start nginx
sudo systemctl enable nginx
sudo systemctl status nginx

# Configuration personnalisée
sudo tee /etc/nginx/conf.d/lab-app.conf << 'EOF'
server {
    listen 80;
    server_name server.lab.local;

    root /var/www/html;
    index index.html index.htm;

    access_log /var/log/nginx/lab-access.log;
    error_log  /var/log/nginx/lab-error.log;

    # Headers de sécurité
    add_header X-Frame-Options DENY;
    add_header X-Content-Type-Options nosniff;
    add_header X-XSS-Protection "1; mode=block";

    location / {
        try_files $uri $uri/ =404;
    }
}
EOF

# Page web de démonstration
sudo tee /var/www/html/index.html << 'EOF'
<!DOCTYPE html>
<html>
<head><title>TechCorp SA — Lab Hybride</title></head>
<body>
  <h1>Infrastructure Hybride Azure + RHEL9</h1>
  <p>Serveur: server.lab.local | IP: 192.168.10.2</p>
  <p>Auteur: Serge TOGNON | AZ-104 + RHCSA Lab</p>
</body>
</html>
EOF

# Tester la configuration
sudo nginx -t

# Recharger Nginx
sudo systemctl reload nginx

# Test depuis VM Client
curl -I http://192.168.10.2
curl http://server.lab.local
```

> 📸 **Capture** : `capture-04-nginx-running.png` — `systemctl status nginx` sur server.lab.local **ET** `curl http://192.168.10.2` depuis client.lab.local affichant HTTP 200.

---

## 🤖 Automatisation Ansible

### Configuration de l'Inventaire

```bash
# Sur VM Client — Créer la structure Ansible
mkdir -p ~/ansible/{inventory,playbooks,roles,vars,group_vars}
cd ~/ansible

# Créer l'inventaire
tee inventory/hosts.ini << 'EOF'
[local_servers]
server.lab.local ansible_user=adminuser ansible_ssh_private_key_file=~/.ssh/id_ed25519_lab

[azure_servers]
# azure-vm ansible_host=<PUBLIC_IP> ansible_user=azureuser

[all:vars]
ansible_python_interpreter=/usr/bin/python3
EOF

# Tester la connectivité Ansible
ansible -i inventory/hosts.ini all -m ping

# Récupérer des infos système
ansible -i inventory/hosts.ini local_servers -m command -a 'uname -a'
ansible -i inventory/hosts.ini local_servers -m command -a 'df -h'
```

### Playbook 1 — Setup Serveur

```yaml
# playbooks/01-setup-server.yml
---
- name: Setup initial du serveur RHEL9
  hosts: local_servers
  become: yes
  vars:
    admin_user: adminuser
    web_port: 80

  tasks:

    - name: Mettre à jour tous les paquets
      dnf:
        name: '*'
        state: latest
        update_cache: yes

    - name: Installer les paquets essentiels
      dnf:
        name:
          - nginx
          - firewalld
          - policycoreutils-python-utils
          - rsyslog
          - chrony
          - git
          - curl
          - vim
        state: present

    - name: Démarrer et activer firewalld
      systemd:
        name: firewalld
        state: started
        enabled: yes

    - name: Ouvrir les ports HTTP et SSH
      firewalld:
        service: "{{ item }}"
        permanent: yes
        state: enabled
        immediate: yes
      loop:
        - ssh
        - http
        - https

    - name: Activer SELinux en mode enforcing
      selinux:
        policy: targeted
        state: enforcing

    - name: Démarrer Nginx
      systemd:
        name: nginx
        state: started
        enabled: yes
```

```bash
# Exécuter le playbook
ansible-playbook -i inventory/hosts.ini playbooks/01-setup-server.yml -v
```

### Playbook 2 — Déploiement Web

```yaml
# playbooks/02-deploy-nginx.yml
---
- name: Déploiement application web Nginx
  hosts: local_servers
  become: yes
  vars:
    web_root: /var/www/html
    app_name: lab-app
    server_name: server.lab.local

  handlers:
    - name: reload nginx
      systemd:
        name: nginx
        state: reloaded

    - name: restart nginx
      systemd:
        name: nginx
        state: restarted

  tasks:

    - name: Créer le répertoire web
      file:
        path: "{{ web_root }}"
        state: directory
        owner: root
        group: webadmin
        mode: '2775'

    - name: Déployer la page index.html
      template:
        src: templates/index.html.j2
        dest: "{{ web_root }}/index.html"
        owner: root
        group: webadmin
        mode: '0644'
      notify: reload nginx

    - name: Configurer Nginx
      template:
        src: templates/nginx.conf.j2
        dest: /etc/nginx/conf.d/{{ app_name }}.conf
      notify: restart nginx

    - name: Corriger le contexte SELinux
      command: restorecon -Rv {{ web_root }}
      changed_when: false

    - name: Vérifier que Nginx répond
      uri:
        url: http://{{ server_name }}
        status_code: 200
      register: result

    - name: Afficher le résultat du test
      debug:
        msg: "Nginx répond HTTP {{ result.status }} - OK!"
```

> 📸 **Capture** : `capture-03-ansible-success.png` — Toutes les tâches affichant `ok` ou `changed` en vert, sans erreur rouge.

### Playbook 3 — Security Hardening

```yaml
# playbooks/03-security-hardening.yml
---
- name: Durcissement sécurité RHEL9
  hosts: local_servers
  become: yes

  tasks:

    - name: Configurer sshd - désactiver root login
      lineinfile:
        path: /etc/ssh/sshd_config
        regexp: '^#?PermitRootLogin'
        line: 'PermitRootLogin no'
        backup: yes
      notify: restart sshd

    - name: Configurer sshd - auth par clé uniquement
      lineinfile:
        path: /etc/ssh/sshd_config
        regexp: '^#?PasswordAuthentication'
        line: 'PasswordAuthentication no'
      notify: restart sshd

    - name: Désactiver les services inutiles
      systemd:
        name: "{{ item }}"
        state: stopped
        enabled: no
      loop:
        - avahi-daemon
        - cups
      ignore_errors: yes

    - name: Configurer les limites système
      pam_limits:
        domain: '*'
        limit_type: hard
        limit_item: nofile
        value: '65536'

  handlers:
    - name: restart sshd
      systemd:
        name: sshd
        state: restarted
```

---

## ☁️ Infrastructure Azure

### Connexion et Variables

```bash
# Connexion à Azure
az login

# Lister les abonnements
az account list --output table

# Sélectionner l'abonnement
az account set --subscription <SUBSCRIPTION_ID>

# Définir les variables
RESOURCE_GROUP='rg-lab-hybride'
LOCATION='eastus'
VNET_NAME='vnet-lab'
SUBNET_NAME='subnet-web'
NSG_NAME='nsg-lab'
VM_NAME='vm-azure-lab'
```

### Création du Resource Group

```bash
az group create \
  --name $RESOURCE_GROUP \
  --location $LOCATION \
  --tags Environment=Lab Owner=SergeTognon Project=HybridInfra

# Vérification
az group show --name $RESOURCE_GROUP --output table
```

> 📝 `--tags` : bonne pratique d'organisation Azure — facilitent la facturation et la gouvernance.

### Création du Réseau Virtuel

```bash
az network vnet create \
  --resource-group $RESOURCE_GROUP \
  --name $VNET_NAME \
  --address-prefix 10.0.0.0/16 \
  --subnet-name $SUBNET_NAME \
  --subnet-prefix 10.0.1.0/24

# Vérification
az network vnet show \
  --resource-group $RESOURCE_GROUP \
  --name $VNET_NAME \
  --output table
```

### Configuration du NSG

```bash
# Créer le NSG
az network nsg create \
  --resource-group $RESOURCE_GROUP \
  --name $NSG_NAME

# Règle SSH (port 22)
az network nsg rule create \
  --resource-group $RESOURCE_GROUP \
  --nsg-name $NSG_NAME \
  --name Allow-SSH \
  --protocol tcp \
  --priority 1000 \
  --destination-port-range 22 \
  --access Allow \
  --direction Inbound

# Règle HTTP (port 80)
az network nsg rule create \
  --resource-group $RESOURCE_GROUP \
  --nsg-name $NSG_NAME \
  --name Allow-HTTP \
  --protocol tcp \
  --priority 1010 \
  --destination-port-range 80 \
  --access Allow \
  --direction Inbound

# Règle HTTPS (port 443)
az network nsg rule create \
  --resource-group $RESOURCE_GROUP \
  --nsg-name $NSG_NAME \
  --name Allow-HTTPS \
  --protocol tcp \
  --priority 1020 \
  --destination-port-range 443 \
  --access Allow \
  --direction Inbound

# Lister les règles NSG
az network nsg rule list \
  --resource-group $RESOURCE_GROUP \
  --nsg-name $NSG_NAME \
  --output table
```

| Option | Description |
|---|---|
| `--priority` | Ordre d'évaluation (100–4096). Plus bas = plus prioritaire |
| `--access Allow/Deny` | Autorise ou refuse le trafic |
| `--direction Inbound` | Sens du trafic (entrant/sortant) |

### Déploiement de la VM Azure

```bash
# Créer la VM Linux
az vm create \
  --resource-group $RESOURCE_GROUP \
  --name $VM_NAME \
  --image Ubuntu2204 \
  --size Standard_B1s \
  --admin-username azureuser \
  --ssh-key-values ~/.ssh/id_ed25519_lab.pub \
  --vnet-name $VNET_NAME \
  --subnet $SUBNET_NAME \
  --nsg $NSG_NAME \
  --public-ip-sku Standard \
  --output table

# Récupérer l'IP publique
PUBLIC_IP=$(az vm show \
  --resource-group $RESOURCE_GROUP \
  --name $VM_NAME \
  --show-details \
  --query publicIps -o tsv)

echo "IP Publique Azure VM: $PUBLIC_IP"

# Connexion SSH à la VM Azure
ssh -i ~/.ssh/id_ed25519_lab azureuser@$PUBLIC_IP
```

> 📸 **Capture** : `capture-06-azure-portal.png` — Portail Azure montrant le Resource Group `rg-lab-hybride` avec toutes les ressources créées.

---

## 📊 Monitoring & Supervision

### Surveillance Système Linux

```bash
# ═══ CPU ═══════════════════════════════════════════════════
# Utilisation CPU en temps réel
top -bn1 | grep 'Cpu(s)'
ps aux --sort=-%cpu | head -10
uptime

# ═══ RAM ═══════════════════════════════════════════════════
free -h
cat /proc/meminfo | grep -E 'MemTotal|MemFree|MemAvailable|Cached'

# ═══ DISQUE ════════════════════════════════════════════════
df -hT
df -i
du -sh /* 2>/dev/null | sort -rh | head -10

# ═══ SERVICES ══════════════════════════════════════════════
for svc in nginx sshd firewalld chronyd rsyslog; do
  echo -n "$svc: "
  systemctl is-active $svc
done
```

### Script de Monitoring Automatisé

```bash
sudo tee /usr/local/bin/monitoring-report.sh << 'SCRIPT'
#!/bin/bash
# ═══════════════════════════════════════════════════
# Script de Monitoring — TechCorp SA
# Auteur: Serge TOGNON
# ═══════════════════════════════════════════════════

REPORT_DIR="/var/log/monitoring"
DATE=$(date '+%Y-%m-%d_%H-%M')
REPORT_FILE="$REPORT_DIR/report-$DATE.log"
ALERT_THRESHOLD_CPU=80
ALERT_THRESHOLD_DISK=85

mkdir -p $REPORT_DIR

echo "==== RAPPORT SYSTÈME ==== $DATE" > $REPORT_FILE
echo "Hostname: $(hostname)" >> $REPORT_FILE

# CPU
CPU_USAGE=$(top -bn1 | grep 'Cpu(s)' | awk '{print $2}' | cut -d. -f1)
echo "CPU Usage: ${CPU_USAGE}%" >> $REPORT_FILE
[ $CPU_USAGE -gt $ALERT_THRESHOLD_CPU ] && \
  echo "⚠️  ALERTE CPU > ${ALERT_THRESHOLD_CPU}%" >> $REPORT_FILE

# RAM
RAM_FREE=$(free -m | awk '/Mem:/{print $4}')
RAM_TOTAL=$(free -m | awk '/Mem:/{print $2}')
echo "RAM: ${RAM_FREE}MB libre / ${RAM_TOTAL}MB total" >> $REPORT_FILE

# DISQUE
DISK_USAGE=$(df / | tail -1 | awk '{print $5}' | tr -d '%')
echo "Disk /: ${DISK_USAGE}%" >> $REPORT_FILE
[ $DISK_USAGE -gt $ALERT_THRESHOLD_DISK ] && \
  echo "⚠️  ALERTE DISQUE > ${ALERT_THRESHOLD_DISK}%" >> $REPORT_FILE

# SERVICES
echo '' >> $REPORT_FILE
echo '=== STATUT SERVICES ===' >> $REPORT_FILE
for svc in nginx sshd firewalld; do
  STATUS=$(systemctl is-active $svc)
  echo "$svc: $STATUS" >> $REPORT_FILE
done

cat $REPORT_FILE
SCRIPT

sudo chmod +x /usr/local/bin/monitoring-report.sh

# Automatiser avec cron (toutes les heures)
echo '0 * * * * root /usr/local/bin/monitoring-report.sh' | sudo tee /etc/cron.d/monitoring

# Test manuel
sudo /usr/local/bin/monitoring-report.sh
```

### Analyse des Logs Système

```bash
# Logs du boot actuel
journalctl -b

# Logs SSH (connexions)
journalctl -u sshd --since '1 hour ago'

# Erreurs critiques du système
journalctl -p err --since '24 hours ago'

# Logs en temps réel
journalctl -f

# Tentatives de connexion SSH échouées
grep 'Failed password\|Failed publickey\|Invalid user' /var/log/secure | tail -20

# Connexions SSH réussies
grep 'Accepted' /var/log/secure | tail -20

# Logs SELinux
sudo ausearch -m avc --start today
sudo grep 'denied' /var/log/audit/audit.log | tail -10
```

### Azure Monitor

```bash
# Créer un Log Analytics Workspace
az monitor log-analytics workspace create \
  --resource-group $RESOURCE_GROUP \
  --workspace-name law-lab-hybride \
  --location $LOCATION

# Créer une alerte CPU > 80%
az monitor metrics alert create \
  --resource-group $RESOURCE_GROUP \
  --name 'alert-cpu-high' \
  --resource $VM_NAME \
  --resource-type 'Microsoft.Compute/virtualMachines' \
  --metric 'Percentage CPU' \
  --operator GreaterThan \
  --threshold 80 \
  --window-size 5m \
  --evaluation-frequency 1m
```

---

## 🔧 Troubleshooting

### Erreurs Fréquentes et Solutions

| Erreur | Solution |
|---|---|
| `ssh: connection refused` | `systemctl status sshd` + `firewall-cmd --list-services` |
| `Permission denied (publickey)` | `chmod 700 ~/.ssh` + `chmod 600 authorized_keys` |
| SELinux bloque Nginx | `restorecon -Rv /var/www/html` |
| `nginx: bind() to 0.0.0.0:80 failed` | Port 80 déjà utilisé : `ss -tuln \| grep :80` |
| `ansible: host unreachable` | Tester : `ssh adminuser@server.lab.local` manuellement |
| `az login: device flow` | Ouvrir l'URL affichée dans un navigateur |
| `dnf: No package ansible` | Activer EPEL : `dnf install -y epel-release` |

### Commandes de Diagnostic Complètes

```bash
# ─── Connectivité réseau ──────────────────────────────────
ping -c 4 192.168.10.2           # Test ICMP
traceroute 192.168.10.2          # Tracer la route
curl -v http://192.168.10.2      # Test HTTP détaillé
nmap -p 22,80,443 192.168.10.2   # Scanner les ports

# ─── SSH ──────────────────────────────────────────────────
ssh -vvv adminuser@192.168.10.2  # Debug SSH niveau 3
ssh-add -l                        # Clés chargées dans l'agent
ssh-add ~/.ssh/id_ed25519_lab    # Ajouter la clé à l'agent

# ─── Services ─────────────────────────────────────────────
systemctl status nginx           # Statut Nginx
journalctl -xe                   # Dernières erreurs système
nginx -t                         # Tester la config Nginx

# ─── Firewall ─────────────────────────────────────────────
firewall-cmd --list-all          # Toutes les règles
firewall-cmd --get-zones         # Zones disponibles

# ─── SELinux ──────────────────────────────────────────────
getenforce                       # Mode actuel
ausearch -m avc --start recent   # Blocages récents
audit2why < /var/log/audit/audit.log  # Explication des blocages

# ─── Azure CLI ────────────────────────────────────────────
az vm list-ip-addresses --output table
az network nsg rule list -g $RESOURCE_GROUP --nsg-name $NSG_NAME --output table
```

---

## ✅ Checklists Certification

### Checklist RHCSA (EX200)

| Domaine | Tâche | Statut |
|---|---|:---:|
| Gestion réseau | `nmcli` — IP statique, hostname | ☐ |
| Gestion réseau | Configuration `/etc/hosts` | ☐ |
| Utilisateurs | `useradd`, `usermod`, `passwd`, `groupadd` | ☐ |
| Permissions | `chmod`, `chown`, SUID/SGID/Sticky bit | ☐ |
| SSH | Génération clés, `authorized_keys`, `sshd_config` | ☐ |
| SELinux | `getenforce`, `setenforce`, `chcon`, `restorecon` | ☐ |
| Firewall | `firewalld` zones, services, ports | ☐ |
| Services | `systemctl` start/stop/enable/status | ☐ |
| Logs | `journalctl`, `/var/log/secure`, `audit.log` | ☐ |
| Stockage | `lsblk`, `fdisk`, `mkfs`, `mount`, `/etc/fstab` | ☐ |
| Cron | `crontab -e`, `/etc/cron.d/` | ☐ |
| Paquets | `dnf` install/remove/update, `rpm` | ☐ |
| NFS/NTP | Mount NFS, `chronyd`, `timedatectl` | ☐ |

### Checklist AZ-104

| Domaine | Tâche | Statut |
|---|---|:---:|
| Identité | Azure AD, RBAC, rôles personnalisés | ☐ |
| Gouvernance | Resource Groups, Tags, Policies | ☐ |
| Stockage | Blob, Files, disques managés | ☐ |
| Réseau | VNet, Subnet, NSG, peering | ☐ |
| Réseau | Load Balancer, Application Gateway | ☐ |
| VMs | Déploiement, tailles, extensions | ☐ |
| VMs | Availability Sets, Scale Sets | ☐ |
| PaaS | App Service, Container Instances | ☐ |
| Sauvegarde | Recovery Services Vault, backup policies | ☐ |
| Monitoring | Azure Monitor, Log Analytics, Alertes | ☐ |
| Sécurité | Key Vault, Microsoft Defender | ☐ |
| Automatisation | ARM Templates, Bicep, Azure CLI | ☐ |

---

## 📸 Captures d'Écran Requises

| Fichier | Quoi Capturer | Moment |
|---|---|---|
| `capture-01-ip-client.png` | `ip addr show` + `hostnamectl` sur client.lab.local | Après config réseau |
| `capture-02-ssh-connection.png` | Connexion SSH réussie + banner affiché | Après durcissement SSH |
| `capture-03-ansible-success.png` | `ansible-playbook` terminé en vert (ok/changed) | Après playbook 02 |
| `capture-04-nginx-running.png` | `systemctl status nginx` + `curl` HTTP 200 | Après déploiement web |
| `capture-05-selinux-enforcing.png` | `sestatus` : enforcing + contexte `/var/www/html` | Après config SELinux |
| `capture-06-azure-portal.png` | Portail Azure : Resource Group avec toutes ressources | Après déploiement Azure |
| `capture-07-monitoring.png` | Script `monitoring-report.sh` en exécution | Après config monitoring |
| `capture-08-firewall-rules.png` | `firewall-cmd --list-all` sur server.lab.local | Après config firewalld |

---

## 💼 Bonnes Pratiques Entreprise

- Toujours sauvegarder les fichiers de configuration avant modification (`cp file file.bak`)
- Utiliser des clés SSH **ED25519** plutôt que RSA pour les nouvelles infrastructures
- Appliquer le **principe du moindre privilège** (`sudo` uniquement quand nécessaire)
- Taguer toutes les ressources Azure (`Environment`, `Owner`, `Project`, `CostCenter`)
- Versionner tous les playbooks Ansible et scripts dans **Git**
- Tester les playbooks Ansible avec `--check --diff` avant exécution en production
- Activer **SELinux en mode Enforcing** — ne jamais le désactiver en production
- Documenter chaque étape de configuration dans le README.md

### Conseils Entretien Technique

> 💡 Lors d'un entretien, soyez capable d'expliquer **POURQUOI** vous avez fait chaque choix technique, pas seulement **COMMENT**.

- Expliquez la différence entre `firewalld` (niveau applicatif) et `iptables` (niveau kernel)
- Sachez expliquer SELinux : Mandatory Access Control vs Discretionary Access Control
- Connaissez les différences entre Azure NSG, Azure Firewall et Application Gateway
- Maîtrisez le cycle de vie d'un playbook Ansible : `lint → check → diff → run`
- Sachez débugger une connexion SSH étape par étape : réseau → firewall → sshd → clés
- Comprenez la différence entre un VNET Azure et un réseau local on-premise

---

## 🏆 Compétences Développées

| Compétence | Niveau Atteint |
|---|---|
| Configuration réseau Linux (nmcli) | Intermédiaire — Avancé |
| Administration SSH sécurisée | Avancé |
| SELinux — gestion des contextes | Intermédiaire |
| Firewalld — règles et zones | Intermédiaire — Avancé |
| Déploiement Nginx / Apache | Intermédiaire |
| Automatisation Ansible | Intermédiaire — Avancé |
| Azure CLI — infrastructure as code | Intermédiaire |
| Azure Networking (VNet, NSG) | Intermédiaire |
| Monitoring Linux (journalctl, scripts) | Intermédiaire |
| Documentation technique | Avancé |

---

## 🎯 Conclusion

Ce lab professionnel démontre la capacité à concevoir, déployer et sécuriser une **infrastructure hybride** combinant Linux RHEL9 et Azure Cloud. Il couvre l'ensemble des compétences requises pour les certifications **RHCSA** et **AZ-104**, tout en reflétant les pratiques réelles d'une équipe IT en entreprise.

Les technologies maîtrisées — Ansible, SSH, SELinux, firewalld, Azure CLI, Azure Monitor — constituent le socle des rôles **Junior Cloud Engineer**, **Linux Administrator** et **DevOps Engineer**.

---

<div align="center">

**Serge TOGNON** · AZ-104 Certified · Préparation RHCSA · 2025

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Serge%20TOGNON-0077B5?style=flat&logo=linkedin)](https://linkedin.com)

*Infrastructure Hybride Azure + RHEL9 — Lab Professionnel Portfolio GitHub*

</div>
