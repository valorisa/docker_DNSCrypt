### **Description du projet :**
**docker_DNSCrypt** est un projet qui utilise Docker pour déployer un serveur DNS sécurisé basé sur **DNSCrypt**. 
Il permet de chiffrer et d'authentifier les requêtes DNS, garantissant ainsi la confidentialité et la sécurité des communications DNS. 
Ce projet peut également inclure une configuration pour **AdGuard Home** afin de bloquer les publicités et les trackers.

---

### **Fonctionnalités principales :**
- Déploiement rapide d'un serveur DNS sécurisé avec **DNSCrypt**.
- Option d'intégration avec **AdGuard Home** pour le filtrage des publicités et des trackers.
- Configuration personnalisable via des fichiers comme dnscrypt-proxy.toml et docker-compose.yml.
- Structure de fichiers claire pour une gestion facile.

---

### **Structure du projet :**
- **`docker-compose.yml`** : Fichier de configuration Docker Compose pour orchestrer les conteneurs.
- **`dnscrypt-proxy.toml`** : Fichier de configuration pour DNSCrypt Proxy.
- **adguard_conf** : Répertoire contenant les configurations pour AdGuard Home.
- **adguard_work** : Répertoire pour les données de travail d'AdGuard Home.
- **`README.md`** : Documentation du projet.
- **`files_structure.txt`** : Description de la structure des fichiers.

---

### **Objectifs :**
- Fournir une solution simple et sécurisée pour protéger les requêtes DNS.
- Offrir une alternative auto-hébergée pour bloquer les publicités et les trackers.
- Simplifier le déploiement grâce à Docker.

---
