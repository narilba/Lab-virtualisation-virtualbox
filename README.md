# 🖥️ Lab de Virtualisation avec VirtualBox

![VirtualBox](https://img.shields.io/badge/VirtualBox-7.x-183A61?style=for-the-badge&logo=virtualbox&logoColor=white)
![pfSense](https://img.shields.io/badge/pfSense-CE_2.8.1-212121?style=for-the-badge&logo=pfsense&logoColor=white)
![Ubuntu](https://img.shields.io/badge/Ubuntu-26.04_LTS-E95420?style=for-the-badge&logo=ubuntu&logoColor=white)
![Apache](https://img.shields.io/badge/Apache-2.4-D22128?style=for-the-badge&logo=apache&logoColor=white)
![PHP](https://img.shields.io/badge/PHP-8.5-777BB4?style=for-the-badge&logo=php&logoColor=white)
![GitHub](https://img.shields.io/badge/GitHub-Déploiement-181717?style=for-the-badge&logo=github&logoColor=white)

> Infrastructure réseau virtualisée complète déployée sur Windows avec des outils **100% gratuits**.
> Comprend un routeur/firewall pfSense, un serveur web Ubuntu hébergeant le site Restaurant NEMDO, et un poste client Ubuntu Desktop.

---

## 📋 Table des matières

- [Architecture](#-architecture)
- [Technologies utilisées](#-technologies-utilisées)
- [Prérequis](#-prérequis)
- [Configuration réseau](#-configuration-réseau)
- [Étapes du projet](#-étapes-du-projet)
- [Captures d'écran](#-captures-décran)
- [Problèmes rencontrés](#-problèmes-rencontrés)
- [Compétences démontrées](#-compétences-démontrées)

---

## 🏗️ Architecture

```
                    ┌─────────────────────────────────────────┐
                    │           PC Windows (Hôte)             │
                    │              VirtualBox                  │
                    │                                          │
                    │  ┌─────────────┐                         │
                    │  │   pfSense   │ WAN: 10.0.2.15 (NAT)   │
  INTERNET ────────────│  Routeur /  │ LAN: 192.168.1.1/24    │
                    │  │  Firewall   │                         │
                    │  └──────┬──────┘                         │
                    │         │  Réseau interne "labnet"        │
                    │         │  192.168.1.0/24                │
                    │    ─────┴──────────────────              │
                    │    │                        │            │
                    │  ┌─┴────────────┐  ┌────────┴───────┐   │
                    │  │Ubuntu Server │  │Ubuntu Desktop  │   │
                    │  │ Apache + PHP │  │   (Client)     │   │
                    │  │192.168.1.10  │  │ 192.168.1.20   │   │
                    │  └──────────────┘  └────────────────┘   │
                    └─────────────────────────────────────────┘
```

---

## 🛠️ Technologies utilisées

| Outil | Version | Rôle |
|---|---|---|
| Oracle VirtualBox | 7.x | Hyperviseur |
| pfSense CE | 2.8.1 | Routeur / Firewall |
| Ubuntu Server | 26.04 LTS | Serveur web |
| Ubuntu Desktop | 26.04 LTS | Poste client |
| Apache | 2.4.66 | Serveur HTTP |
| PHP | 8.5 | Moteur PHP |
| Git + GitHub | — | Versioning et déploiement |

---

## ✅ Prérequis

- PC Windows 10/11 avec au moins **8 Go de RAM** et **60 Go d'espace disque libre**
- [Oracle VirtualBox](https://virtualbox.org) installé
- [Git pour Windows](https://git-scm.com/download/win) installé
- Compte [GitHub](https://github.com) créé
- ISOs téléchargées :
  - [Ubuntu Server 26.04 LTS](https://ubuntu.com/download/server)
  - [pfSense CE 2.8.1](https://www.pfsense.org/download/) (AMD64 ISO)

---

## 🌐 Configuration réseau

| VM | Interface | IP | Rôle |
|---|---|---|---|
| pfSense | em0 (NAT) | 10.0.2.15 | WAN — accès internet |
| pfSense | em1 (labnet) | 192.168.1.1 | LAN — passerelle du lab |
| Ubuntu Server | enp0s3 (NAT) | 10.0.2.15 | Accès internet |
| Ubuntu Server | enp0s8 (labnet) | **192.168.1.10** | Serveur web Apache |
| Ubuntu Desktop | enp0s3 (labnet) | **192.168.1.20** | Poste client |

### Configuration IP permanente — Ubuntu Server

Fichier : `/etc/netplan/50-cloud-init.yaml` (voir dossier [`configs/`](configs/))

```yaml
network:
  version: 2
  ethernets:
    enp0s3:
      dhcp4: true
    enp0s8:
      dhcp4: no
      addresses:
        - 192.168.1.10/24
      routes:
        - to: 192.168.1.0/24
          via: 192.168.1.1
```

---

## 📌 Étapes du projet

### Phase 1 — Création des VMs

- Création de 3 VMs dans VirtualBox avec la configuration réseau adaptée
- pfSense : Adapter 1 (NAT) + Adapter 2 (Réseau interne `labnet`)
- Ubuntu Server : Adapter 1 (NAT) + Adapter 2 (Réseau interne `labnet`)
- Ubuntu Desktop : Adapter 1 (Réseau interne `labnet`) uniquement

### Phase 2 — Installation des systèmes

- Installation de **pfSense CE** avec assignation des interfaces WAN/LAN
- Installation d'**Ubuntu Server 26.04 LTS** avec OpenSSH activé
- Installation d'**Ubuntu Desktop**

### Phase 3 — Configuration réseau permanente

- Configuration IP statique permanente sur Ubuntu Server via `netplan`
- Configuration IP statique permanente sur Ubuntu Desktop via l'interface graphique
- Tests de connectivité entre les 3 VMs (ping, curl HTTPS/TLS1.3)

### Phase 4 — Déploiement du site web

```bash
# Sur Ubuntu Server
apt install apache2 php libapache2-mod-php -y
systemctl enable apache2

cd /var/www/html
rm index.html
git clone https://github.com/narilba/restaurant-nemdo-feat-ai.git .
```

### Phase 5 — Résultat final

Le site **Restaurant NEMDO** est accessible depuis Ubuntu Desktop via Firefox à l'adresse `http://192.168.1.10`.

---

## 📸 Captures d'écran

### Infrastructure VirtualBox
![VMs en fonction](screenshots/02-virtualbox-all-vms-running.png)
*Les 3 VMs démarrées simultanément — pfSense, Ubuntu Server, Ubuntu Desktop*

### pfSense — Routeur/Firewall
![pfSense WAN/LAN](screenshots/03-pfsense-wan-lan-menu.png)
*Menu principal pfSense — WAN: 10.0.2.15 / LAN: 192.168.1.1*

### Tests de connectivité
![Ping entre VMs](screenshots/11-ubuntu-server-ping-apres-ip-permanente.png)
*Ping réussi depuis Ubuntu Server vers pfSense (192.168.1.1) et Ubuntu Desktop (192.168.1.20) — 0% perte*

### Configuration IP permanente
![Netplan apply](screenshots/10-ubuntu-server-netplan-apply.png)
*Configuration IP statique permanente via netplan — inet 192.168.1.10/24*

### Connexion HTTPS vers pfSense
![curl TLS1.3](screenshots/09-ubuntu-server-curl-pfsense-https.png)
*Connexion HTTPS TLS1.3 vers pfSense confirmée via curl*

### Déploiement Apache
![Apache running](screenshots/17-apache-active-running.png)
*Apache2 active (running) sur Ubuntu Server*

### Résultat final ⭐
![Site NEMDO](screenshots/20-ubuntu-desktop-nemdo-site.png)
*Site Restaurant NEMDO affiché dans Firefox sur Ubuntu Desktop — servi depuis Ubuntu Server via le réseau labnet*

---

## 🔧 Problèmes rencontrés

| Problème | Solution |
|---|---|
| pfSense ne contactait pas les serveurs Netgate | Basculé temporairement en mode Pont pour l'installation |
| IP de enp0s8 disparaissait au redémarrage | Configuration permanente via `/etc/netplan/50-cloud-init.yaml` |
| Conflit d'IP : Ubuntu Server avait pris 192.168.1.1 | `ip addr del 192.168.1.1/24 dev enp0s8` |
| `git push` échouait : branche `main` introuvable | La branche s'appelle `master` → `git push -u origin master` |
| lynx incompatible avec TLS1.3 de pfSense | Utilisation de `curl -k -v https://192.168.1.1` |
| VMs ne communiquaient plus après redémarrage | pfSense doit être démarré en premier — c'est la passerelle |

---

## 🎯 Compétences démontrées

- **Virtualisation** — Création et configuration de VMs, réseaux NAT et internes
- **Administration réseau** — TCP/IP statique, routage, passerelle, DNS, segmentation
- **Firewall** — Déploiement pfSense CE, interfaces WAN/LAN, TLS1.3
- **Administration Linux** — Ubuntu Server/Desktop, netplan, systemctl, commandes réseau
- **Serveur web** — Apache2 + PHP, déploiement application PHP
- **Git & GitHub** — Push depuis Windows, clone depuis Linux, gestion des remotes
- **Dépannage réseau** — ip addr, ip route, ping, curl, résolution de conflits

---

## 📁 Structure du dépôt

```
lab-virtualisation-virtualbox/
├── README.md
├── screenshots/          # Captures d'écran du projet
├── configs/
│   └── netplan-50-cloud-init.yaml
└── docs/
    └── documentation_lab_virtualisation.docx
```

---

## 🔗 Projets liés

- [Restaurant NEMDO](https://github.com/narilba/restaurant-nemdo-feat-ai) — Site PHP hébergé sur ce lab

---

*Projet réalisé dans le cadre d'un portfolio — Network & Systems Technology, 2ème année*
