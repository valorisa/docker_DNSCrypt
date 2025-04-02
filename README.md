# DNSCrypt-Proxy avec AdGuard Home via Docker Compose

Ce projet configure [DNSCrypt-Proxy](https://github.com/DNSCrypt/dnscrypt-proxy) et [AdGuard Home](https://github.com/AdguardTeam/AdGuardHome) pour fonctionner ensemble en utilisant Docker Compose. AdGuard Home agit comme serveur DNS principal sur votre réseau, filtrant les publicités et les traqueurs, et utilise DNSCrypt-Proxy comme unique serveur amont (upstream) pour chiffrer et anonymiser les requêtes DNS sortantes.

Cette configuration est conçue pour fonctionner sur macOS, Linux ou Windows via Docker.

## Architecture

Vos Appareils --> AdGuard Home (Docker) --> DNSCrypt-Proxy (Docker) --> Résolveurs DNS Chiffrés
| | |
(DNS pointant (Écoute sur port 53 (Écoute sur 0.0.0.0:5353
vers l'IP de de l'hôte Docker) sur le réseau interne Docker)
la machine hôte
Docker, ou 127.0.0.1
si sur la même machine)
|
(Filtrage + Cache AdGuard)


## Prérequis

*   [Docker Engine](https://docs.docker.com/engine/install/)
*   [Docker Compose](https://docs.docker.com/compose/install/) (Généralement inclus avec Docker Desktop)
*   **Pour Windows/macOS :** [Docker Desktop](https://www.docker.com/products/docker-desktop/) est le moyen le plus simple. Assurez-vous qu'il utilise le backend WSL2 sous Windows pour de meilleures performances réseau.
*   **Pour Linux :** Installez `docker-ce`, `docker-ce-cli`, `containerd.io`, et `docker-compose-plugin` (ou `docker-compose` autonome).

## Mise en Place

1.  **Cloner le dépôt ou créer les fichiers :**
    Clonez ce dépôt ou créez manuellement les fichiers suivants dans un nouveau dossier (par ex. `docker-dns`):
    *   `docker-compose.yml`
    *   `dnscrypt-proxy.toml`
    *   `.gitignore` (Recommandé)

2.  **Créer les répertoires pour les volumes AdGuard :**
    Dans le même dossier, créez les répertoires qui stockeront les données persistantes d'AdGuard Home. Cela permet à votre configuration et à vos statistiques de survivre aux redémarrages des conteneurs.

    ```bash
    mkdir adguard_work adguard_conf
    ```
    *(Sous Windows, utilisez l'Explorateur de fichiers ou `mkdir adguard_work adguard_conf` dans PowerShell ou CMD)*

3.  **Contenu du fichier `docker-compose.yml` :**
    Copiez le contenu du fichier `docker-compose.yml` fourni dans ce dépôt.

4.  **Contenu du fichier `dnscrypt-proxy.toml` :**
    Copiez le contenu du fichier `dnscrypt-proxy.toml` fourni dans ce dépôt.

5.  **Contenu du fichier `.gitignore` :**
    Copiez le contenu du fichier `.gitignore` fourni dans ce dépôt pour éviter de versionner les données AdGuard.

## Configuration

### 1. Fichier `dnscrypt-proxy.toml` (Vérification Cruciale)

Ce fichier configure DNSCrypt-Proxy. Le paramètre le plus important est `listen_addresses`.

*   **Assurez-vous** que ce fichier est bien dans le même dossier que `docker-compose.yml`.
*   **Vérifiez** que `listen_addresses` est défini sur `['0.0.0.0:5353']`.
    *   **Pourquoi `0.0.0.0` et pas `127.0.0.1` ?** `0.0.0.0` permet au service DNSCrypt-Proxy *à l'intérieur* de son conteneur d'écouter les connexions provenant d'autres conteneurs (comme AdGuard Home) sur le réseau Docker interne (`dns-network`). Si vous utilisiez `127.0.0.1`, seul le conteneur DNSCrypt-Proxy lui-même pourrait s'y connecter, rendant impossible pour AdGuard Home de l'utiliser comme serveur amont. Le port `5353` n'est **pas** exposé à l'extérieur de Docker.
*   **Adaptez** `server_names` pour choisir vos résolveurs DNS chiffrés préférés (ex: `cloudflare`, `google`, `quad9`, etc.). Consultez la documentation DNSCrypt-Proxy pour une liste complète.
*   **Logging :** La ligne `log_file` a été supprimée pour que les logs soient envoyés vers la sortie standard (stdout/stderr), ce qui est la pratique recommandée avec Docker et vous permet d'utiliser `docker-compose logs`.
*   Les options `require_dnssec`, `require_no_log`, `require_nofilter` sont des choix raisonnables (le filtrage étant délégué à AdGuard). Ajustez si nécessaire.

### 2. Configuration Initiale d'AdGuard Home (Après Lancement)

La configuration principale d'AdGuard Home se fait via son interface web après le premier démarrage.

## Démarrage des Services

1.  Ouvrez un terminal ou une invite de commande dans le dossier contenant vos fichiers `docker-compose.yml`, `dnscrypt-proxy.toml`, et les répertoires `adguard_*`.
2.  Lancez les conteneurs en arrière-plan :

    ```bash
    docker-compose up -d
    ```
    *Note :* Sous Windows, si le pare-feu vous demande une autorisation pour Docker ou ses composants réseau, accordez-la.

## Configuration Post-Lancement d'AdGuard Home

1.  **Accéder à l'interface web :** Ouvrez votre navigateur et allez sur `http://127.0.0.1:3000` (ou `http://<ip-de-votre-machine-docker>:3000` si vous y accédez depuis une autre machine sur votre réseau).
2.  **Assistant de configuration AdGuard :**
    *   **Important :** Lors de la première configuration, AdGuard vous demandera sur quelles interfaces écouter pour le port DNS (53) et l'interface web (3000). Laissez les valeurs par défaut (`Toutes les interfaces`).
    *   Suivez les étapes pour créer un nom d'utilisateur et un mot de passe administrateur.
3.  **Configurer le Serveur DNS Amont (Upstream) - ÉTAPE CRUCIALE :**
    *   Une fois connecté à l'interface AdGuard, allez dans **Paramètres** -> **Paramètres DNS**.
    *   Dans la section **Serveurs DNS en amont**, **supprimez toutes les entrées par défaut** (comme `https://dns.adguard.com/dns-query` ou autres).
    *   **Ajoutez** l'adresse suivante (c'est le *nom du service* Docker et le *port interne* défini dans `dnscrypt-proxy.toml`) :

        ```text
        dnscrypt-proxy:5353
        ```

        *(Docker Compose permet au conteneur `adguardhome` de résoudre le nom de service `dnscrypt-proxy` en son adresse IP interne sur le réseau `dns-network`)*
    *   **Serveurs DNS de démarrage (Bootstrap) :** Vous pouvez laisser ce champ vide ou ajouter un DNS public standard (ex: `1.1.1.1` ou `8.8.8.8`). Ces serveurs sont utilisés *uniquement* par AdGuard lui-même pour résoudre l'adresse IP des serveurs DoH/DoT/DoQ si vous en configurez *directement* dans AdGuard (ce qui n'est pas notre cas ici, car nous passons par `dnscrypt-proxy`) et pour des vérifications initiales. Ils **ne sont pas utilisés** pour les requêtes DNS normales de vos appareils clients.
    *   Cliquez sur **Tester les serveurs en amont**. Vous devriez voir une réponse positive indiquant qu'AdGuard peut joindre `dnscrypt-proxy:5353`.
    *   Cliquez sur **Appliquer**.

## Configuration des Paramètres DNS sur vos Appareils

Pour utiliser votre nouvelle configuration DNS :

*   **Méthode 1 : Configurer chaque appareil individuellement**
    *   Allez dans les paramètres réseau de votre appareil (Windows, macOS, Linux, iOS, Android).
    *   Trouvez les paramètres DNS pour votre connexion Wi-Fi ou Ethernet.
    *   **Supprimez** tous les serveurs DNS existants (FAI, Google, etc.).
    *   Ajoutez **uniquement** l'adresse IP de la machine qui héberge Docker comme serveur DNS. Si vous configurez l'appareil sur lequel Docker tourne, utilisez `127.0.0.1`.
    *   *Note :* Supprimer les autres DNS force tout le trafic DNS via AdGuard/DNSCrypt. Cependant, cela peut empêcher la résolution de noms locaux fournis par votre routeur (ex: `imprimante.local`). Si cela pose problème, vous pouvez configurer AdGuard pour utiliser votre routeur comme résolveur inverse privé (voir Paramètres -> Paramètres DNS -> section "Serveurs DNS privés inversés" dans AdGuard).

*   **Méthode 2 : Configurer votre routeur (Recommandé pour couvrir tout le réseau)**
    *   Connectez-vous à l'interface d'administration de votre routeur.
    *   Trouvez les paramètres DNS (souvent dans la section LAN ou DHCP).
    *   Remplacez les serveurs DNS existants par l'adresse IP de la machine hébergeant Docker.
    *   Sauvegardez les paramètres et redémarrez éventuellement les appareils pour qu'ils obtiennent les nouveaux paramètres DNS via DHCP.

## Vérification

1.  **Vérifier que les conteneurs sont en cours d'exécution** :

    ```bash
    docker ps
    ```
    *(Vous devriez voir les conteneurs `adguard` et `dnscrypt` avec le statut "Up")*

2.  **Tester la Résolution DNS** :
    Depuis un appareil configuré pour utiliser le nouveau DNS, utilisez `nslookup` (Windows/Linux/macOS) ou `dig` (Linux/macOS - installez via `brew install bind` sur Mac ou `sudo apt install dnsutils`/`sudo yum install bind-utils` sur Linux) :

    ```bash
    nslookup example.com <IP_MACHINE_DOCKER_OU_127.0.0.1>
    # Ou
    dig example.com @<IP_MACHINE_DOCKER_OU_127.0.0.1>
    ```
    *(La requête doit réussir et passer par AdGuard -> DNSCrypt)*

3.  **Consulter les Logs Docker** :
    Pour voir les logs agrégés des deux services :

    ```bash
    docker-compose logs -f
    ```
    Pour voir les logs d'un service spécifique (arrêtez avec Ctrl+C) :

    ```bash
    docker-compose logs dnscrypt-proxy
    docker-compose logs adguardhome
    ```

4.  **Consulter le Journal des Requêtes AdGuard :**
    L'interface web d'AdGuard (`http://<ip>:3000`) a un "Journal des requêtes" très utile pour voir quelles requêtes sont bloquées ou autorisées, et par quel serveur amont elles sont passées (devrait être `dnscrypt-proxy:5353`).

## Mise à Jour

Pour mettre à jour les images AdGuard Home et DNSCrypt-Proxy vers les dernières versions spécifiées dans `docker-compose.yml` (ici `:latest`) :

1.  Télécharger les nouvelles images :

    ```bash
    docker-compose pull
    ```

2.  Recréer et redémarrer les conteneurs avec les nouvelles images (votre configuration et vos données dans les volumes `adguard_*` seront préservées) :

    ```bash
    docker-compose up -d
    ```

## Arrêt des Services

Pour arrêter et supprimer les conteneurs (sans supprimer les volumes de données `adguard_*`) :

```bash
docker-compose down
```

Pour arrêter sans supprimer les conteneurs :

```bash
docker-compose stop
```

## Conclusion
Vous disposez maintenant d'une configuration DNS locale robuste et sécurisée utilisant AdGuard Home pour le filtrage et DNSCrypt-Proxy pour le chiffrement et l'anonymisation, le tout géré via Docker Compose. Explorez l'interface web d'AdGuard Home pour affiner vos listes de blocage, ajouter des règles personnalisées, configurer le contrôle parental, etc.
