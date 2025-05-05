Plan de Présentation Détaillé : Formation Prometheus et Grafana (3 Jours)Introduction GénéraleCe document présente un plan détaillé pour une formation intensive de trois jours sur Prometheus et Grafana. L'objectif est de fournir aux participants les connaissances théoriques et les compétences pratiques nécessaires pour déployer, configurer, gérer et utiliser efficacement une solution de supervision moderne basée sur ces outils open-source. La formation est conçue pour être interactive, intégrant un Travail Pratique (TP) fil rouge qui permettra aux participants de construire progressivement une infrastructure de monitoring complète. Le public visé comprend les ingénieurs DevOps, SRE, administrateurs systèmes et développeurs impliqués dans la supervision d'applications et d'infrastructures. Une familiarité de base avec la ligne de commande Linux, Docker et les concepts réseau est présupposée.Jour 1 : Fondations et Configuration de Prometheus(Objectif du Jour 1 : Maîtriser les concepts fondamentaux de Prometheus, son installation, sa configuration de base, le modèle de données, les types de métriques, l'introduction à PromQL et le relabelling.)(a) Matinée : Concepts, Installation et Configuration (Approx. 15-20 Slides)

1. Introduction à la Supervision Moderne et Prometheus

Contexte et Importance : La supervision est devenue essentielle pour garantir la fiabilité et la performance des systèmes informatiques modernes, en particulier dans les architectures microservices et cloud-natives. Comprendre l'état interne d'un système est crucial pour diagnostiquer les problèmes et optimiser les performances. L'observabilité repose sur trois piliers principaux : les métriques (quantitatives, agrégables), les logs (événements discrets) et les traces (suivi des requêtes à travers les services). Prometheus se positionne comme un outil fondamental pour la collecte et l'analyse des métriques.
Présentation de Prometheus : Initialement développé chez SoundCloud en 2012 et désormais projet gradué de la Cloud Native Computing Foundation (CNCF), Prometheus est un toolkit open-source de monitoring et d'alerting. Sa philosophie repose sur la fiabilité et l'autonomie ; chaque serveur Prometheus est conçu pour être indépendant et fonctionner même en cas de défaillance d'autres parties de l'infrastructure.
Cas d'Usage : Prometheus est largement utilisé pour superviser des serveurs (métriques système), des applications (métriques métiers, performance), des bases de données, des équipements réseau, et est particulièrement adapté aux environnements dynamiques comme Kubernetes.
Points Forts : Son modèle de données multi-dimensionnel basé sur des métriques et des labels (clé-valeur), son puissant langage de requête PromQL, son vaste écosystème d'exporteurs pour intégrer des systèmes tiers, et son système d'alerting intégré via Alertmanager sont ses principaux atouts.
Limites : Par défaut, Prometheus n'est pas conçu pour le stockage de données à très long terme (rétention limitée) et nécessite des solutions complémentaires pour cela. Il se concentre sur les métriques numériques et ne gère pas nativement les logs ou les traces. De plus, bien que très fiable, il ne garantit pas une exactitude à 100% des données collectées (en raison du scraping par intervalles), ce qui le rend inadapté pour des cas d'usage comme la facturation à la requête.
Conception et Implications : L'accent mis par Prometheus sur la fiabilité et l'autonomie a des implications directes sur son architecture. Le choix d'un modèle "pull" 1 et d'un stockage local TSDB garantit que le serveur peut continuer à fonctionner et à collecter des données même si des cibles ou des systèmes de stockage réseau sont défaillants.5 Ce design robuste implique cependant que la scalabilité pour le stockage à très long terme et la haute disponibilité native nécessitent des composants externes (comme VictoriaMetrics 6 ou Thanos 2), contrairement à certains systèmes nativement distribués ou basés sur un modèle "push". Comprendre ce compromis est essentiel pour déterminer quand Prometheus seul suffit et quand des extensions sont nécessaires.



2. Architecture de Prometheus

Vue d'Ensemble : L'écosystème Prometheus comprend plusieurs composants.5 Le Serveur Prometheus est le cœur, responsable du scraping des métriques, de leur stockage dans sa base de données temporelles (TSDB), de l'évaluation des règles (enregistrement et alerte) et de la fourniture d'une API pour les requêtes PromQL. Les Cibles (Targets) sont les endpoints (applications, serveurs via exporteurs) que Prometheus scrape pour obtenir les métriques. Les Exporteurs agissent comme des traducteurs, exposant les métriques de systèmes tiers (ex: bases de données, matériel) dans un format compréhensible par Prometheus. Le Pushgateway est une passerelle intermédiaire pour les jobs éphémères qui ne peuvent pas être scrapés directement. La Découverte de Services (Service Discovery) permet à Prometheus de trouver dynamiquement les cibles dans des environnements changeants (ex: Kubernetes, Consul). Alertmanager gère les alertes déclenchées par Prometheus, en assurant la déduplication, le groupage, le routage vers différents canaux (email, Slack, PagerDuty...), ainsi que la gestion des silences et des inhibitions. Enfin, des outils comme Grafana ou des clients API interrogent Prometheus pour visualiser les données.
Modèle Pull : Prometheus utilise principalement un modèle "pull".5 Le serveur interroge activement les endpoints HTTP (/metrics par défaut) des cibles configurées à intervalles réguliers (définis par scrape_interval).1 Ce modèle offre plusieurs avantages : un contrôle centralisé de la collecte, la capacité de détecter si une cible est indisponible (scrape échoué), une simplicité pour les cibles (il suffit d'exposer un endpoint HTTP), et une gestion souvent plus simple des règles de pare-feu (flux sortant depuis Prometheus vers les cibles).1 Les inconvénients potentiels incluent des difficultés de scalabilité si le nombre de cibles devient extrêmement élevé et des complexités dans certains scénarios réseau (NAT, pare-feu très restrictifs interdisant les connexions sortantes).1
Pushgateway : Pour les cas où le modèle pull n'est pas adapté, comme les scripts batch ou les fonctions serverless qui s'exécutent brièvement et se terminent avant d'avoir pu être scrapés, le Pushgateway sert d'intermédiaire.1 Ces jobs éphémères peuvent "pousser" leurs métriques vers le Pushgateway, qui les expose ensuite pour que Prometheus puisse les scraper comme une cible classique.1 Son utilisation doit rester limitée à ces cas spécifiques, car il peut devenir un point unique de défaillance et masque la source réelle de la métrique.1
Modularité Architecturale : L'architecture de Prometheus illustre une séparation claire des responsabilités. Le serveur central se concentre sur les fonctions essentielles de collecte, stockage et interrogation. Des composants spécialisés gèrent des tâches spécifiques : les exporteurs adaptent les données, le Pushgateway gère un mode d'ingestion alternatif, Alertmanager offre une gestion avancée des alertes, et les mécanismes de découverte de services s'adaptent aux environnements dynamiques. Cette modularité offre une grande flexibilité, permettant aux utilisateurs de ne déployer que les composants nécessaires et facilitant l'extension de l'écosystème, contrairement à des systèmes de monitoring monolithiques. Par exemple, la séparation d'Alertmanager permet de gérer les alertes (inhibition, silences 11) indépendamment de l'évaluation des règles par Prometheus 14, et de centraliser les alertes de plusieurs serveurs Prometheus. De même, le modèle d'exporteur 17 favorise un large écosystème communautaire.



3. Modèle de Données Prometheus

Concept Fondamental : Au cœur de Prometheus se trouve la série temporelle (Time Series) : un flux de valeurs horodatées appartenant à la même métrique et au même ensemble de dimensions labellisées. Chaque série est identifiée de manière unique par la combinaison de son nom de métrique et de ses labels (paires clé-valeur).
Nom de Métrique : Il décrit la mesure générale (ex: http_requests_total). Les noms peuvent contenir des caractères UTF-8, mais il est fortement recommandé de respecter la regex [a-zA-Z_:][a-zA-Z0-9_:]* pour une compatibilité maximale.18 Les deux-points (:) sont réservés aux règles d'enregistrement définies par l'utilisateur.18
Labels : Les labels ajoutent des dimensions aux métriques, permettant de différencier et de filtrer les instances d'une même mesure (ex: method="POST", status="500" pour http_requests_total). C'est le modèle de données dimensionnel de Prometheus.18 Modifier la valeur d'un label, en ajouter ou en supprimer un, crée une nouvelle série temporelle.18 Les noms de labels suivent les mêmes recommandations de caractères que les noms de métriques (regex [a-zA-Z_][a-zA-Z0-9_]* recommandée), et ceux commençant par __ sont réservés à un usage interne.18 Les valeurs de labels peuvent contenir n'importe quel caractère Unicode.18 Un label avec une valeur vide est considéré comme inexistant 18
Samples (Échantillons) : Chaque point de données dans une série temporelle est un échantillon, composé d'une valeur (float64 ou, expérimentalement, un histogramme natif) et d'un timestamp (avec une précision à la milliseconde).
Notation : La notation standard pour identifier une série temporelle est <nom_métrique>{<nom_label>="<valeur_label>",...}. Exemple : api_http_requests_total{method="POST", handler="/messages"}.18
Cardinalité : La cardinalité désigne le nombre total de séries temporelles uniques stockées par Prometheus. Elle est directement liée au nombre de combinaisons uniques de noms de métriques et de labels.19 Une cardinalité élevée peut impacter significativement les performances (RAM, CPU, disque) et la vitesse des requêtes.19 Il est crucial d'éviter les labels ayant un très grand nombre de valeurs possibles (ex: ID utilisateur, ID de requête, timestamps précis).19
Gestion de la Cardinalité : Le modèle dimensionnel est extrêmement puissant pour le filtrage et l'agrégation, mais une mauvaise gestion des labels peut entraîner une "explosion de cardinalité". Chaque combinaison unique de nom de métrique et de paires clé-valeur de labels génère une série temporelle distincte que Prometheus doit stocker et indexer.18 L'ajout de labels avec des valeurs très variables (ID utilisateur, ID de session, adresses IP spécifiques, etc.) peut multiplier de manière exponentielle le nombre de séries.19 Cela augmente l'utilisation de la mémoire pour l'indexation, l'espace disque pour le stockage, et le temps CPU pour les requêtes et les agrégations. Une bonne pratique fondamentale est donc de concevoir soigneusement les labels : utiliser des dimensions significatives pour le filtrage et l'agrégation, mais éviter celles qui génèrent une infinité de séries uniques. Des techniques comme le relabelling 21, abordées plus tard, sont essentielles pour gérer la cardinalité en supprimant ou modifiant des labels avant leur ingestion ou leur envoi vers un stockage distant.



4. Stockage TSDB (Time Series Database)

Vue d'Ensemble : Prometheus intègre sa propre base de données temporelles (TSDB), optimisée pour le stockage et l'interrogation efficaces des métriques. Elle est conçue pour un stockage local sur disque par défaut.
Structure sur Disque : Les données sont organisées en blocs immuables couvrant généralement une période de 2 heures.30 Chaque répertoire de bloc contient un sous-répertoire chunks (stockant les échantillons compressés par série temporelle), un fichier index (permettant de retrouver rapidement les séries basées sur les labels) et un fichier meta.json (métadonnées du bloc).30 Les suppressions de séries sont gérées via des fichiers tombstones distincts.30
Bloc en Mémoire et WAL : Les données les plus récentes (généralement les dernières ~2 heures) résident dans un bloc en mémoire appelé "Head Block".30 Pour garantir la durabilité en cas de crash, Prometheus utilise un Write-Ahead Log (WAL). Avant d'écrire en mémoire, chaque opération (création de série, ajout d'échantillons, suppression via tombstones) est enregistrée séquentiellement dans le WAL.30 En cas de redémarrage, Prometheus relit le WAL pour reconstruire l'état en mémoire.30 Le WAL est stocké sur disque sous forme de segments (fichiers de 128 Mo par défaut).30 Un processus de checkpointing crée périodiquement une version filtrée et compactée des anciens segments WAL avant leur suppression, pour accélérer la récupération.32 La compression du WAL peut être activée pour économiser de l'espace disque (--storage.tsdb.wal-compression).30
Compaction : En arrière-plan, les blocs initiaux de 2 heures sont progressivement fusionnés (compactés) en blocs plus grands, couvrant des périodes plus longues (jusqu'à 10% du temps de rétention configuré, ou 31 jours, la valeur la plus petite étant retenue).30 Cela optimise les requêtes sur des périodes plus longues en réduisant le nombre de blocs à consulter.
Configuration de la Rétention : La durée de conservation des données est contrôlée par deux flags principaux :

--storage.tsdb.retention.time : Définit la durée maximale de conservation des données (ex: 15d, 90d). Le défaut est 15d.30
--storage.tsdb.retention.size : Définit la taille maximale (en octets) que les blocs de données persistants peuvent occuper (ex: 50GB, 1TB). Le défaut est 0 (désactivé).30
Si les deux sont définis, la politique qui est atteinte en premier (durée ou taille) déclenche la suppression des blocs les plus anciens.30 La suppression effective peut prendre jusqu'à deux heures.30 Il est crucial de prévoir suffisamment d'espace disque, en tenant compte non seulement des blocs compactés mais aussi du pic d'utilisation du WAL et du Head Block.30 Une estimation grossière de l'espace nécessaire peut être calculée : espace_disque ≈ temps_rétention_secondes * samples_ingérés_par_seconde * octets_par_sample (environ 1-2 octets par sample après compression).30


Implications Opérationnelles (Sauvegarde/Restauration) : La structure en blocs de 2 heures et le mécanisme du WAL ont un impact direct sur les stratégies de sauvegarde et de restauration. Le "Head Block" contenant les données les plus récentes est principalement en mémoire, protégé par le WAL.30 Une simple copie de fichiers risque de capturer les fichiers WAL dans un état incohérent ou de manquer des données en mémoire tampon. Le WAL est rejoué au démarrage pour restaurer cet état.30 Le checkpointing assure un état récupérable des anciens segments WAL avant leur suppression.32 Par conséquent, la seule méthode fiable pour sauvegarder une instance Prometheus en cours d'exécution sans l'arrêter est d'utiliser l'API d'administration pour créer un snapshot (/api/v1/admin/tsdb/snapshot).30 Cette API garantit une vue cohérente des données, incluant les segments WAL nécessaires et les informations de checkpoint. Les sauvegardes effectuées sans snapshot risquent de perdre jusqu'aux ~2 dernières heures de données.30 Ce détail opérationnel est critique pour les environnements de production.



5. Installation et Configuration Initiale


Méthodes d'Installation : Prometheus peut être installé de plusieurs manières : en téléchargeant les binaires précompilés (disponibles pour Linux, Windows, macOS), en compilant depuis le code source (Go requis), en utilisant des images Docker officielles, ou via des systèmes de gestion de configuration (Ansible, Chef, Puppet, SaltStack). Pour Windows, le binaire est prometheus.exe.


Focus sur Docker Compose : Pour le TP, Docker Compose est privilégié car il simplifie la gestion d'une stack de monitoring complète (Prometheus, Grafana, Exporters, Alertmanager, etc.) dans des conteneurs isolés. Il permet de définir et de lancer l'ensemble de l'environnement avec un seul fichier docker-compose.yml.


Fichier prometheus.yml : La configuration de Prometheus se fait via un fichier YAML. Sa structure principale comprend :

global: Définit les paramètres globaux par défaut, comme scrape_interval (fréquence de collecte des métriques, défaut 1m) et evaluation_interval (fréquence d'évaluation des règles, défaut 1m).
rule_files: Spécifie les chemins (avec wildcards possibles) vers les fichiers contenant les règles d'enregistrement (recording rules) et d'alerte (alerting rules).
scrape_configs: Une liste de "jobs" définissant les cibles à scraper. Chaque job a un nom (job_name) et spécifie comment trouver ses cibles (via static_configs pour une liste fixe ou via des mécanismes de découverte de service comme consul_sd_configs, kubernetes_sd_configs, etc.). On peut y surcharger les paramètres globaux comme scrape_interval.
alerting: Configure comment Prometheus communique avec une ou plusieurs instances d'Alertmanager.37
remote_write / remote_read: Configure les endpoints pour envoyer (write) ou lire (read) des données depuis/vers un système de stockage distant.



Validation de la Configuration : L'outil promtool (inclus avec Prometheus) permet de vérifier la syntaxe et la validité d'un fichier de configuration avant de démarrer ou recharger Prometheus : promtool check config /etc/prometheus/prometheus.yml.40 Il peut aussi vérifier les fichiers de règles (promtool check rules...).14


Démarrage et UI : Prometheus se lance via la commande prometheus --config.file=prometheus.yml (ou via Docker/Docker Compose). Son interface web est accessible par défaut sur le port 9090. L'UI permet d'explorer les cibles (Status -> Targets), la configuration chargée (Status -> Configuration), les règles (Status -> Rules), et d'exécuter des requêtes PromQL (Graph).


TP Fil Rouge - Étape 1 : Environnement Docker Compose Initial

Créer la structure : Créer un répertoire prometheus-grafana-training pour le projet.
docker-compose.yml : Créer un fichier docker-compose.yml avec le contenu suivant :
YAMLversion: '3.8'

volumes:
  prometheus-data: {}
  portainer-data: {}

services:
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    restart: unless-stopped
    ports:
      - "9090:9090"
    volumes:
      -./prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus-data:/prometheus # Volume pour les données TSDB
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--web.console.libraries=/usr/share/prometheus/console_libraries'
      - '--web.console.templates=/usr/share/prometheus/consoles'
      - '--web.enable-lifecycle' # Permet le rechargement via API

  portainer:
    image: portainer/portainer-ce:latest
    container_name: portainer
    restart: unless-stopped
    ports:
      - "9443:9443"
      - "9000:9000" # Optionnel, pour HTTP
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock # Accès au socket Docker
      - portainer-data:/data # Volume pour les données Portainer

Justification : Utilisation d'images officielles, mapping des ports standards, montage du fichier de configuration Prometheus, montage de volumes nommés pour la persistance des données TSDB et Portainer, montage du socket Docker pour Portainer 44, et activation du rechargement de configuration Prometheus via API (--web.enable-lifecycle).46
prometheus.yml : Créer un fichier prometheus.yml de base :
YAMLglobal:
  scrape_interval: 15s # Intervalle de scrape par défaut

scrape_configs:
  - job_name: 'prometheus'
    # Scrape Prometheus lui-même
    static_configs:
      - targets: ['localhost:9090']

Justification : Configuration minimale pour que Prometheus se scrape lui-même.
Lancer les services : Ouvrir un terminal dans le répertoire du projet et exécuter : docker-compose up -d.38
Vérification :

Accéder à l'UI Prometheus : http://localhost:9090. Aller dans "Status" -> "Targets". La cible "prometheus" doit être "UP".
Accéder à l'UI Portainer : https://localhost:9443 (ignorer l'avertissement de sécurité). Créer un compte administrateur. Explorer l'interface pour voir les conteneurs prometheus et portainer.






(b) Après-midi : Métriques, PromQL et Relabelling (Approx. 20-25 Slides)
1. Types de Métriques Prometheus

Introduction : Prometheus définit quatre types de métriques fondamentaux, différenciés principalement au niveau des bibliothèques client pour guider leur utilisation correcte. Le serveur Prometheus traite actuellement toutes les données comme des séries temporelles non typées, bien que cela puisse évoluer.47
Counter (Compteur) :

Définition : Représente une valeur numérique cumulative qui ne peut qu'augmenter ou être remise à zéro (typiquement lors d'un redémarrage de l'application ou de l'exporteur). Il ne faut jamais utiliser un compteur pour une valeur qui peut diminuer (comme le nombre de processus en cours).
Cas d'usage : Idéal pour compter des événements comme le nombre total de requêtes HTTP servies, le nombre de tâches terminées, le nombre d'erreurs survenues, ou le volume total de données transmises.
Interprétation : La valeur brute d'un compteur est souvent moins intéressante que son taux de changement. On utilise les fonctions PromQL rate() (taux de croissance par seconde sur une période donnée) ou increase() (augmentation totale sur une période donnée) pour analyser son évolution. Exemple : rate(http_requests_total[5m]) donne le taux moyen de requêtes par seconde sur les 5 dernières minutes.


Gauge (Jauge) :

Définition : Représente une valeur numérique instantanée qui peut librement augmenter ou diminuer au fil du temps.
Cas d'usage : Parfait pour mesurer des états instantanés comme la température actuelle, l'utilisation de la mémoire ou du CPU, le nombre de connexions actives, la taille d'une file d'attente, le niveau de batterie.
Interprétation : On s'intéresse généralement à la valeur actuelle de la jauge et à son évolution (tendances, pics, creux). Exemple : node_memory_MemAvailable_bytes.23


Histogram (Histogramme) :

Définition : Échantillonne des observations (typiquement des durées de requête ou des tailles de réponse) et les répartit dans des "buckets" (seuils) préconfigurés. Il expose également la somme de toutes les valeurs observées (_sum) et le nombre total d'observations (_count).
Structure : Un histogramme avec un nom de base <basename> expose plusieurs séries temporelles :

<basename>_bucket{le="<borne_sup>"}: Compteurs cumulatifs pour chaque bucket. le signifie "less than or equal to". Chaque bucket inclut les comptes des buckets inférieurs. Le bucket le="+Inf" contient le nombre total d'observations.
<basename>_sum: La somme de toutes les valeurs observées.
<basename>_count: Le nombre total d'observations (identique à la valeur du bucket le="+Inf").


Cas d'usage : Mesurer la distribution des latences de requêtes, des tailles de réponses, ou calculer des scores Apdex.
Interprétation : Permet le calcul de quantiles (percentiles) approximatifs côté serveur en utilisant la fonction PromQL histogram_quantile(). Les histogrammes peuvent être agrégés entre différentes instances avant le calcul des quantiles (ex: sum(rate(mon_histogramme_bucket[5m])) by (le)). Exemple : histogram_quantile(0.95, sum by (le) (rate(http_request_duration_seconds_bucket[5m]))) calcule le 95ème percentile de la latence sur 5 minutes pour l'ensemble des instances agrégées.


Summary (Sommaire) :

Définition : Similaire à un histogramme, il échantillonne des observations. Cependant, il calcule des quantiles configurables (φ-quantiles, 0 ≤ φ ≤ 1) directement côté client sur une fenêtre temporelle glissante et les expose comme des métriques distinctes. Il fournit également _sum et _count.
Structure : Un sommaire avec un nom de base <basename> expose :

<basename>{quantile="<φ>"}: La valeur estimée pour chaque φ-quantile configuré.
<basename>_sum: La somme de toutes les valeurs observées.
<basename>_count: Le nombre total d'observations.


Cas d'usage : Utile lorsque des quantiles précis sont nécessaires pour une instance spécifique, ou pour le suivi direct de SLO basés sur des percentiles, et lorsque l'agrégation inter-instances n'est pas requise.
Interprétation : L'agrégation des quantiles pré-calculés entre différentes instances est mathématiquement incorrecte dans la plupart des cas. Le calcul des quantiles côté client peut être plus coûteux en ressources pour l'application instrumentée. Moins flexible que les histogrammes pour l'analyse ad-hoc car les quantiles et la fenêtre temporelle doivent être définis à l'avance. Exemple : http_request_duration_seconds{quantile="0.99"} donne directement le 99ème percentile pré-calculé.


Comparaison Histogram vs Summary : Le choix entre Histogram et Summary dépend du cas d'usage et des compromis acceptables.


CaractéristiqueHistogramSummaryCalcul des QuantilesCôté serveur (via histogram_quantile())Côté client (pré-calculé, exposé directement)Agrégation Inter-InstancesOui, supportée via PromQL (ex: sum by (le) (rate(...)))Non, l'agrégation des quantiles est généralement incorrecteFlexibilité d'AnalyseHaute (calcul ad-hoc de n'importe quel quantile, ajustement de la période)Basse (quantiles et fenêtre temporelle préconfigurés)Performance ClientLéger (incrémentation de compteurs)Plus coûteux (algorithme de streaming pour les quantiles)Performance ServeurPlus coûteux (calcul des quantiles lors des requêtes)Léger (quantiles déjà calculés)Précision/ErreurDépend de la configuration des buckets (erreur sur l'axe des valeurs)Dépend de la configuration de l'erreur (±ε sur l'axe des quantiles φ)Quand utiliser?Agrégation nécessaire, analyse ad-hoc, SLOs globauxPrécision requise sur une instance, pas d'agrégation, coût serveur faible    *Justification du tableau :* Ce tableau synthétise les différences fondamentales et les compromis entre les deux types de métriques servant à mesurer des distributions, comme souligné dans plusieurs sources. Il aide à prendre une décision éclairée lors de l'instrumentation : privilégier la flexibilité et l'agrégation (Histogram) ou la précision locale et la performance serveur (Summary).


2. Node Exporter


Rôle et Installation : Le Node Exporter est un composant essentiel de l'écosystème Prometheus, conçu pour collecter une large variété de métriques liées au matériel et au noyau des systèmes d'exploitation de type *nix (Linux, macOS, BSD). Il s'agit généralement d'un binaire statique unique, facile à déployer. Il peut être installé via les gestionnaires de paquets (comme apt sur Debian/Ubuntu 50), en téléchargeant le binaire depuis la page de releases Prometheus, ou via Docker.49 Pour Windows, un exporteur analogue existe (windows_exporter).


Configuration : Le Node Exporter écoute par défaut sur le port 9100. Son comportement est contrôlé par des flags en ligne de commande. On peut activer ou désactiver des collecteurs spécifiques (modules qui collectent un certain type de métriques) avec --collector.<nom> et --no-collector.<nom>. Par exemple, --collector.disable-defaults désactive tous les collecteurs par défaut, nécessitant d'activer explicitement ceux souhaités. Des filtres peuvent être appliqués à certains collecteurs, comme --collector.filesystem.mount-points-exclude pour ignorer certains points de montage.


Métriques Clés (Collecteurs par défaut) : Le Node Exporter expose de nombreuses métriques. Parmi les plus importantes collectées par défaut, on trouve :

CPU : node_cpu_seconds_total (Counter) : Temps CPU total passé dans différents modes (idle, user, system, iowait, steal, etc.), par cœur CPU.49
Mémoire (RAM) : node_memory_MemTotal_bytes (Gauge), node_memory_MemFree_bytes (Gauge), node_memory_MemAvailable_bytes (Gauge, mémoire réellement disponible), node_memory_Buffers_bytes (Gauge), node_memory_Cached_bytes (Gauge).23
Disque (I/O) : node_disk_read_bytes_total (Counter), node_disk_written_bytes_total (Counter), node_disk_io_time_seconds_total (Counter, temps passé en I/O), node_disk_reads_completed_total (Counter), node_disk_writes_completed_total (Counter).54
Disque (Système de fichiers) : node_filesystem_size_bytes (Gauge, taille totale), node_filesystem_avail_bytes (Gauge, espace dispo pour non-root), node_filesystem_free_bytes (Gauge, espace libre total), par point de montage (mountpoint) et type de système de fichiers (fstype).49
Réseau : node_network_receive_bytes_total (Counter), node_network_transmit_bytes_total (Counter), node_network_receive_packets_total (Counter), node_network_transmit_packets_total (Counter), node_network_receive_errs_total (Counter), node_network_transmit_errs_total (Counter), par interface (device).54
Système : node_load1, node_load5, node_load15 (Gauges, charge système moyenne sur 1, 5, 15 min), node_time_seconds (Gauge, timestamp Unix actuel du nœud), node_boot_time_seconds (Gauge, timestamp Unix du démarrage du nœud), node_uname_info (Gauge avec valeur 1 et labels contenant les infos uname).54



Exploration : On peut consulter toutes les métriques exposées en accédant à l'URL http://<ip_node_exporter>:9100/metrics via curl ou un navigateur.


TP Fil Rouge - Étape 2 : Installation et Scraping du Node Exporter

Prérequis : S'assurer qu'une machine virtuelle Debian est disponible (créée via VirtualBox, par exemple) et accessible depuis la machine hôte exécutant Docker Compose. Noter l'adresse IP de cette VM.
Installation sur la VM Debian :

Se connecter à la VM Debian en SSH.
Mettre à jour les paquets : sudo apt update && sudo apt upgrade -y.
Installer Node Exporter via apt (méthode simple pour Debian) : sudo apt install prometheus-node-exporter.50 (Alternative : suivre les étapes manuelles de téléchargement/création d'utilisateur/service systemd).
Vérifier que le service est démarré et actif : sudo systemctl status prometheus-node-exporter. Il devrait écouter sur 0.0.0.0:9100.51
Activer le démarrage automatique : sudo systemctl enable prometheus-node-exporter.
Vérifier l'accès aux métriques depuis la VM : curl http://localhost:9100/metrics.
Si un pare-feu est actif sur la VM (ex: ufw), autoriser le port 9100 : sudo ufw allow 9100/tcp.


Configuration de Prometheus (sur la machine hôte) :

Ouvrir le fichier prometheus.yml dans le répertoire du TP.
Ajouter un nouveau job sous scrape_configs :
YAMLscrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'node_exporter'
    # Cible statique pointant vers la VM Debian
    static_configs:
      - targets: # Remplacer <IP_VM_DEBIAN> par l'IP réelle

Justification : Ajout d'un job dédié 'node_exporter' avec une configuration statique pointant vers l'IP et le port de l'exporteur sur la VM.


Redémarrer Prometheus : Dans le terminal sur la machine hôte, dans le répertoire du TP : docker-compose restart prometheus.
Vérification :

Accéder à l'UI Prometheus (http://localhost:9090).
Aller dans "Status" -> "Targets".
Vérifier que la cible pour le job node_exporter avec l'adresse <IP_VM_DEBIAN>:9100 apparaît et a l'état "UP".37 (Cela peut prendre quelques secondes après le redémarrage).







3. Introduction à PromQL


Rôle et Concepts : PromQL (Prometheus Query Language) est le langage fonctionnel intégré à Prometheus, conçu spécifiquement pour interroger et agréger les données de séries temporelles stockées. Les requêtes peuvent être instantanées (évaluées à un seul point dans le temps) ou de plage (évaluées à intervalles réguliers sur une durée).82 PromQL opère sur quatre types de données principaux :

Instant Vector : Un ensemble de séries temporelles où chaque série a une seule valeur (sample) à un instant donné. C'est le résultat le plus courant d'une sélection simple.22
Range Vector : Un ensemble de séries temporelles où chaque série a une plage de valeurs sur une durée spécifiée. Utilisé principalement comme entrée pour des fonctions (ex: rate()).22
Scalar : Une simple valeur numérique flottante.22
String : Une chaîne de caractères littérale (actuellement peu utilisé dans les résultats de requêtes).22



Sélecteurs de Séries Temporelles (pour Instant Vectors) :

Par nom de métrique : Sélectionne toutes les séries ayant ce nom. Ex: http_requests_total.
Avec des labels : Filtre les séries basées sur des correspondances de labels, entre accolades {}. Ex: http_requests_total{job="prometheus", group="canary"}.
Opérateurs de correspondance :

=: Égalité exacte.
!=: Différent de.
=~: Correspondance avec une expression régulière (RE2).
!~: Ne correspond pas à une expression régulière.
Ex: http_requests_total{status!~"4.."} sélectionne les requêtes dont le statut n'est pas 4xx.22


Filtrage par nom via __name__ : Le nom de la métrique est stocké internement comme un label __name__. On peut donc utiliser les opérateurs de correspondance dessus. Ex: {__name__=~"node_.*", job="node_exporter"} sélectionne toutes les métriques commençant par node_ pour le job node_exporter.



Sélecteurs de Plage Temporelle (pour Range Vectors) :

Syntaxe : Ajoute une durée entre crochets `` à la fin d'un sélecteur d'instant vector. Ex: http_requests_total{job="prometheus"}[5m].
Unités de durée : ms (millisecondes), s (secondes), m (minutes), h (heures), d (jours), w (semaines), y (années).22
Usage : Principalement pour fournir une fenêtre de temps aux fonctions comme rate(), increase(), delta(), *_over_time() qui calculent des agrégations ou des taux sur cette période.22



Modificateur offset :

Permet de décaler la fenêtre temporelle de la requête dans le passé. Ex: http_requests_total offset 5m renvoie la valeur de la métrique telle qu'elle était il y a 5 minutes.22
S'applique immédiatement après le sélecteur (avant les fonctions d'agrégation). Ex: sum(rate(http_requests_total[5m] offset 1w)) calcule la somme du taux de requêtes d'il y a une semaine.22



Opérateurs :

Arithmétiques : +, -, *, / (division), % (modulo), ^ (puissance). Peuvent opérer entre scalaires, entre instant vectors (opération élément par élément sur les séries ayant les mêmes labels), ou entre un scalaire et un instant vector.22
Comparaison : ==, !=, >, <, >=, <=. Utilisés pour filtrer des vecteurs (ex: ma_metrique > 10) ou renvoyer 0 ou 1 avec le modificateur bool (ex: ma_metrique > bool 10).22
Logiques/Ensemblistes : Opèrent sur des instant vectors, basés sur les ensembles de labels.

and: Intersection (garde les séries de gauche présentes des deux côtés avec les mêmes labels).
or: Union (garde les séries de gauche + celles de droite n'ayant pas de correspondance à gauche).
unless: Différence (garde les séries de gauche absentes à droite avec les mêmes labels).
.86





Interface Utilisateur Prometheus : L'onglet "Graph" de l'UI web de Prometheus (:9090/graph) contient l'"Expression Browser", un champ permettant de taper et d'exécuter des requêtes PromQL. Les résultats peuvent être affichés sous forme de table ("Table" view) montrant les valeurs instantanées, ou de graphique ("Graph" view) montrant l'évolution temporelle. C'est l'outil idéal pour tester et explorer les requêtes.


TP Fil Rouge - Étape 3 : Premières requêtes PromQL

Accéder à l'UI : Ouvrir http://localhost:9090/graph dans un navigateur.
Explorer les métriques Node Exporter : Taper les requêtes suivantes dans l'Expression Browser et observer les résultats en vues "Table" et "Graph".

Afficher toutes les séries pour node_cpu_seconds_total :
Extrait de codenode_cpu_seconds_total


Filtrer pour le mode idle uniquement :
Extrait de codenode_cpu_seconds_total{mode="idle"}

56
Filtrer pour l'instance spécifique de la VM Debian (remplacer <IP_VM_DEBIAN>):
Extrait de codenode_cpu_seconds_total{instance="<IP_VM_DEBIAN>:9100"}


Combiner les filtres :
Extrait de codenode_cpu_seconds_total{mode="idle", instance="<IP_VM_DEBIAN>:9100"}


Utiliser != pour exclure le mode idle :
Extrait de codenode_cpu_seconds_total{mode!="idle", instance="<IP_VM_DEBIAN>:9100"}

56
Calculer le taux d'utilisation CPU (non-idle) par seconde sur les 5 dernières minutes :
Extrait de coderate(node_cpu_seconds_total{mode!="idle", instance="<IP_VM_DEBIAN>:9100"}[5m])

56 (Observer le résultat en mode Graph).
Calculer le pourcentage d'utilisation CPU moyen par instance (formule courante) :
Extrait de code100 - (avg by (instance) (rate(node_cpu_seconds_total{mode="idle", instance="<IP_VM_DEBIAN>:9100"}[5m])) * 100)

57 (Observer en mode Table et Graph).
Afficher la mémoire disponible :
Extrait de codenode_memory_MemAvailable_bytes{instance="<IP_VM_DEBIAN>:9100"}

23
Calculer le pourcentage de mémoire utilisée :
Extrait de code100 * (1 - node_memory_MemAvailable_bytes{instance="<IP_VM_DEBIAN>:9100"} / node_memory_MemTotal_bytes{instance="<IP_VM_DEBIAN>:9100"})

23
Afficher l'espace disque disponible pour le point de montage / :
Extrait de codenode_filesystem_avail_bytes{instance="<IP_VM_DEBIAN>:9100", mountpoint="/"}

55
Calculer le pourcentage d'espace disque utilisé pour / :
Extrait de code100 * (1 - node_filesystem_avail_bytes{instance="<IP_VM_DEBIAN>:9100", mountpoint="/"} / node_filesystem_size_bytes{instance="<IP_VM_DEBIAN>:9100", mountpoint="/"})

75


Utiliser offset : Comparer le taux d'utilisation CPU actuel avec celui d'il y a 1 heure :
Extrait de code# Taux actuel
rate(node_cpu_seconds_total{mode!="idle", instance="<IP_VM_DEBIAN>:9100"}[5m])
# Taux il y a 1 heure
rate(node_cpu_seconds_total{mode!="idle", instance="<IP_VM_DEBIAN>:9100"}[5m] offset 1h)

(Comparer les graphiques ou les tables).





4. Importance des Labels et Relabelling


Rôle des Labels : Les labels sont fondamentaux dans Prometheus car ils permettent de créer un modèle de données riche et multi-dimensionnel. Ils sont essentiels pour :

Filtrer les données lors des requêtes PromQL (ex: ne voir que les requêtes POST ou les erreurs 5xx).
Agréger les données selon des dimensions spécifiques (ex: calculer le taux de requêtes moyen par job ou par instance).
Créer des vues spécifiques dans les dashboards (ex: un panneau par service).
Corréler différentes métriques partageant les mêmes labels.
.18



Introduction au Relabelling : Le relabelling est un mécanisme puissant de Prometheus qui permet de modifier l'ensemble des labels associés à une cible avant que Prometheus ne la scrape.21 Il est configuré dans la section relabel_configs d'un scrape_config.


Cas d'Usage de relabel_configs :

Standardisation : Renommer ou reformater des labels pour assurer la cohérence entre différentes sources.19
Enrichissement : Ajouter des labels statiques (ex: environment="production") ou dérivés d'autres labels (ex: extraire le nom du datacenter depuis le nom d'hôte instance).21
Filtrage de Cibles : Décider quelles cibles découvertes par la Service Discovery doivent être effectivement scrapées ou ignorées, basé sur leurs labels initiaux (méta-labels).21
Utilisation des Méta-Labels : Exploiter les labels spéciaux fournis par les mécanismes de Service Discovery (commençant par __meta_), comme __meta_consul_service, __meta_kubernetes_pod_name, pour piloter le relabelling.21 Par exemple, utiliser __meta_kubernetes_pod_label_<nom_label> pour transformer un label de pod Kubernetes en label Prometheus.



Syntaxe de relabel_configs : C'est une liste de règles appliquées séquentiellement. Chaque règle contient :

source_labels: [Optionnel] Liste de noms de labels existants dont les valeurs seront concaténées (avec separator) pour former la chaîne source.
separator: [Optionnel] Caractère utilisé pour joindre les valeurs des source_labels (défaut: ;).
regex: [Optionnel] Expression régulière (format RE2) à appliquer sur la chaîne source (défaut: (.*) qui capture tout).
modulus: [Optionnel] Pour l'action hashmod, le modulo à appliquer.
target_label: [Optionnel] Le nom du label sur lequel l'action va écrire.
replacement: [Optionnel] La valeur à écrire dans target_label. Peut utiliser les groupes capturés par regex (ex: $1, $2). Défaut: $1.
action: [Optionnel] L'action à effectuer (défaut: replace).
.21



Actions Courantes (relabel_configs) :

replace (défaut) : Si regex correspond à la source, écrit replacement dans target_label (après substitution des groupes capturés). Si target_label n'est pas défini, la valeur est écrite mais pas attachée. Si regex ne correspond pas, aucune action.21
keep : Ne garde (scrape) que les cibles pour lesquelles la regex correspond à la source.21
drop : Ignore (ne scrape pas) les cibles pour lesquelles la regex correspond à la source.21
hashmod : Calcule le hash de la source, applique le modulus, et écrit le résultat dans target_label. Utile pour le sharding.21
labelmap : Applique la regex à tous les noms de labels de la cible. Pour chaque nom qui correspond, copie la valeur de ce label vers un nouveau label dont le nom est donné par replacement (avec substitution des groupes capturés depuis le nom du label source). Très utile pour préserver les méta-labels (ex: regex: __meta_kubernetes_pod_label_(.+), replacement: k8s_pod_$1).21
labeldrop : Applique la regex à tous les noms de labels. Supprime les labels dont le nom correspond.21
labelkeep : Applique la regex à tous les noms de labels. Ne conserve que les labels dont le nom correspond.21
Autres (moins courants) : keepequal, dropequal, lowercase, uppercase.21



Distinction Importante : Il existe aussi metric_relabel_configs qui a la même syntaxe mais s'applique après le scrape et avant l'ingestion dans le TSDB ou l'envoi via remote_write. metric_relabel_configs permet de filtrer/modifier les métriques et leurs labels, tandis que relabel_configs agit sur les cibles et leurs labels initiaux.21


TP Fil Rouge - Étape 4 : Relabelling Simple

Objectif : Ajouter un label statique environment=training à toutes les métriques provenant de la cible node_exporter.
Modifier prometheus.yml : Localiser le job node_exporter.
Ajouter relabel_configs :
YAMLscrape_configs:
  #... (job prometheus)...
  - job_name: 'node_exporter'
    static_configs:
      - targets:
    relabel_configs:
      - target_label: environment # Label à ajouter/modifier
        replacement: training    # Valeur à assigner
        # action: replace (implicite car c'est le défaut)

Justification : Cette règle simple utilise l'action replace par défaut. Comme il n'y a pas de source_labels ni de regex, la condition est toujours vraie, et la valeur training est assignée au target_label environment pour chaque cible découverte dans ce job.24
Recharger la configuration Prometheus :

Soit redémarrer le conteneur : docker-compose restart prometheus.
Soit utiliser l'API de rechargement (car --web.enable-lifecycle est activé) : curl -X POST http://localhost:9090/-/reload. Vérifier les logs Prometheus (docker-compose logs prometheus) pour confirmer le rechargement réussi.


Vérification :

Dans l'UI Prometheus ("Status" -> "Targets"), vérifier que la cible du job node_exporter possède maintenant un label environment avec la valeur training.
Dans l'onglet "Graph", exécuter une requête comme node_cpu_seconds_total. Observer que toutes les séries retournées pour l'instance de la VM incluent maintenant le label environment="training".
Tester le filtrage : node_cpu_seconds_total{environment="training"} doit retourner les métriques, tandis que node_cpu_seconds_total{environment="production"} ne doit rien retourner.






Jour 2 : Visualisation, Sécurisation et Exporteurs(Objectif du Jour 2 : Maîtriser Grafana pour la visualisation des métriques Prometheus, sécuriser l'accès via Nginx, et intégrer des exporteurs spécifiques comme ceux pour PostgreSQL, Nginx et Blackbox.)(a) Matinée : Grafana et Sécurisation Nginx (Approx. 15-20 Slides)

1. Introduction à Grafana

Présentation : Grafana est la plateforme open-source de référence pour la visualisation et l'analyse de données temporelles (métriques, logs, traces). Il permet de créer des dashboards interactifs et esthétiques à partir de diverses sources de données.
Synergie avec Prometheus : Bien que Prometheus dispose d'une interface web basique pour l'exploration, Grafana offre des capacités de visualisation bien plus avancées et flexibles. Il s'intègre nativement avec Prometheus comme source de données, supporte PromQL, et permet de construire des tableaux de bord complexes et personnalisés. Cette complémentarité en fait le choix privilégié pour visualiser les métriques Prometheus.89 De nombreuses solutions packagées comme kube-prometheus-stack les déploient ensemble par défaut.90
Architecture (Simplifiée) : Grafana est une application web composée d'un backend écrit en Go et d'un frontend en React/TypeScript.95 Il utilise une base de données (SQLite par défaut, supporte aussi PostgreSQL, MySQL) pour stocker ses propres configurations (dashboards, utilisateurs, sources de données, etc.).96 Son architecture est extensible via un système de plugins pour ajouter de nouvelles sources de données, de nouveaux types de panneaux de visualisation, ou des applications complètes.95
Concepts Clés :

Data Sources : Connexions configurées vers des bases de données ou des API (Prometheus, Loki, InfluxDB, SQL, etc.).95
Dashboards : Collection de visualisations (panneaux) organisées.99
Panels : Unités de visualisation individuelles (graphiques, jauges, tableaux, etc.) affichant les données d'une ou plusieurs requêtes.61
Rows : Permettent de regrouper logiquement les panneaux dans un dashboard.59
Variables : Permettent de créer des dashboards dynamiques et interactifs en offrant des listes déroulantes pour filtrer ou modifier les requêtes (ex: choisir un serveur, un environnement).79
Templates : Utilisation des variables pour répéter des panneaux ou des lignes pour chaque valeur sélectionnée.98
Plugins : Extensions pour ajouter des fonctionnalités (Data Sources, Panels, Apps).95
Utilisateurs, Équipes, Organisations, Permissions : Gestion fine des accès.77





2. Installation et Configuration de Grafana


Méthodes : Grafana peut être installé via des paquets système, des binaires, Docker, ou via Helm pour Kubernetes.77


Focus Docker Compose : Pour le TP, nous utiliserons Docker Compose pour une gestion simplifiée aux côtés de Prometheus.


Configuration Essentielle : Il faut principalement mapper le port d'écoute (défaut 3000) et assurer la persistance des données de Grafana (dashboards, configuration des sources, utilisateurs) en montant un volume sur /var/lib/grafana dans le conteneur.96


TP Fil Rouge - Étape 5 : Ajout de Grafana à Docker Compose

Modifier docker-compose.yml : Ajouter le service suivant :
YAMLservices:
  #... (service prometheus)...
  #... (service portainer)...

  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    restart: unless-stopped
    ports:
      - "3000:3000"
    volumes:
      - grafana-data:/var/lib/grafana # Volume pour les données Grafana
    # depends_on: # Optionnel, assure que Prometheus démarre avant
    #   - prometheus

volumes:
  prometheus-data: {}
  portainer-data: {}
  grafana-data: {} # Déclarer le volume nommé

Justification : Utilisation de l'image officielle 38, mapping du port standard 3000 38, et création/montage d'un volume nommé grafana-data pour la persistance.38
Redémarrer les services : docker-compose up -d.
Premier accès : Ouvrir http://localhost:3000 dans un navigateur. Se connecter avec les identifiants par défaut admin / admin. Grafana demandera de changer le mot de passe lors de la première connexion.38 Choisir un nouveau mot de passe sécurisé.





3. Connexion de Grafana à Prometheus


Processus : L'ajout d'une source de données ("Data Source") se fait via l'interface web de Grafana, généralement dans la section "Configuration" ou "Connections".68


Configuration pour Prometheus :

Type : Sélectionner "Prometheus" dans la liste des sources de données disponibles.89
Nom : Donner un nom descriptif (ex: "Prometheus-Local") qui apparaîtra dans les sélecteurs de panneaux.115
URL : Indiquer l'URL HTTP(S) où le serveur Prometheus est accessible. Dans le contexte de Docker Compose, utiliser le nom du service et le port interne (ex: http://prometheus:9090).81
Access Mode : Choisir entre "Server" (anciennement Proxy) et "Browser" (anciennement Direct). Le mode "Server" est généralement recommandé : les requêtes sont envoyées depuis le backend Grafana vers la source de données, ce qui évite les problèmes de CORS et masque l'URL de la source aux navigateurs clients.79 Le mode "Browser" fait que le navigateur de l'utilisateur interroge directement la source de données.
Authentication : Configurer si Prometheus nécessite une authentification (Basic Auth, TLS Client Auth, etc.). Non nécessaire pour notre TP initial.
Scrape Interval : Optionnel, peut aider Grafana à aligner les graphiques (variable $__interval). Indiquer l'intervalle de scrape le plus fréquent configuré dans Prometheus.116



Test : Un bouton "Save & Test" permet de vérifier que Grafana peut se connecter et interroger la source de données Prometheus configurée.81


TP Fil Rouge - Étape 6 : Ajouter Prometheus comme Data Source

Navigation : Dans l'UI Grafana (http://localhost:3000), aller dans le menu de gauche -> "Connections" -> "Data sources" (ou icône engrenage -> "Data Sources").
Ajouter : Cliquer sur "Add data source".81
Sélectionner : Rechercher "Prometheus" et cliquer dessus.89
Configurer :

Name : Prometheus-Local.
URL : http://prometheus:9090 (Grafana peut résoudre le nom de service prometheus car ils sont sur le même réseau Docker Compose).
Access : Laisser sur "Server".
Laisser les autres champs par défaut.


Tester et Sauvegarder : Cliquer sur "Save & Test". Un message vert "Data source is working" devrait apparaître.81





4. Création de Dashboards Grafana

Processus de Base : La création d'un dashboard commence par cliquer sur "Dashboards" dans le menu, puis "New" -> "New dashboard".99 Un dashboard vide est créé, prêt à recevoir des panneaux.
Ajout de Panneaux : Cliquer sur "Add visualization" sur le dashboard vide ou sur l'icône "Add panel" en haut à droite d'un dashboard existant.61 Sélectionner la source de données (ex: "Prometheus-Local").99
Configuration d'un Panneau : L'éditeur de panneau s'ouvre :

Onglet Query : C'est ici que l'on écrit la ou les requêtes PromQL pour récupérer les données.61 L'éditeur offre souvent une auto-complétion pour les noms de métriques et les labels.
Choix de la Visualisation : Sur la droite, choisir le type de panneau le plus adapté aux données :

Time series (Graph) : Le choix par défaut pour visualiser l'évolution de métriques dans le temps (CPU, mémoire, requêtes/sec).57
Stat : Affiche une valeur unique, souvent agrégée (dernière valeur, moyenne, min, max, total). Idéal pour les indicateurs clés (KPIs), l'état actuel, l'uptime.61
Gauge : Similaire à Stat, mais représente la valeur sur une échelle avec des seuils colorés. Parfait pour les pourcentages (utilisation disque, CPU) ou les niveaux avec des limites définies.61
Table : Affiche les données brutes ou transformées sous forme de tableau.61
Autres : Bar chart, Pie chart, Heatmap, Node Graph, etc.


Options du Panneau : Configurer le titre, la description, l'unité de mesure (essentiel pour une bonne interprétation : secondes, octets, pourcentage...), les seuils pour changer les couleurs (dans Stat et Gauge), le formatage de la légende, etc..60


Organisation : Les panneaux peuvent être regroupés dans des "Rows" (lignes) pour structurer le dashboard (ex: une ligne pour le CPU, une pour la mémoire).59 Les lignes peuvent être repliées.
Sauvegarde : Cliquer sur l'icône "Save dashboard" en haut à droite, donner un nom au dashboard, éventuellement une description et choisir un dossier de sauvegarde.61
Développement Itératif : La création de dashboards efficaces est un processus itératif.81 Il est préférable de commencer par visualiser les métriques clés (indicateurs de performance, utilisation des ressources critiques comme CPU/Mémoire/Disque/Réseau - souvent basés sur les méthodes USE/RED) et de créer des dashboards ciblés plutôt qu'un unique dashboard surchargé. L'objectif est de fournir des informations exploitables rapidement, en particulier lors d'incidents. En utilisant les dashboards et en analysant les incidents passés, l'équipe identifiera les manques ou les améliorations possibles, menant à un affinement progressif des requêtes, des visualisations et de l'organisation du dashboard. La flexibilité de Grafana supporte bien cette approche.95



5. Concepts Avancés de Grafana


Variables : Les variables sont une fonctionnalité clé pour créer des dashboards dynamiques et réutilisables.79 Elles apparaissent généralement sous forme de listes déroulantes en haut du dashboard et permettent aux utilisateurs de modifier les données affichées sans changer les requêtes manuellement.

Types Courants :

Query : La liste des valeurs est générée par une requête sur une source de données. Très utilisé pour lister des environnements, des jobs, des instances, des serveurs, etc. Exemple pour Prometheus : label_values(up{job="node_exporter"}, instance) pour lister toutes les instances du job node_exporter qui sont UP.98
Custom : L'utilisateur définit manuellement une liste de valeurs (séparées par des virgules) ou de paires clé/valeur (ex: Nom Affiche : valeur_interne).98
Interval : Permet de sélectionner des intervalles de temps (ex: 1m, 5m, 1h) utilisables dans les agrégations temporelles.98
DataSource : Permet de changer dynamiquement la source de données utilisée par les panneaux.79
Autres : Constant, Text box, Ad hoc filters.


Utilisation : On référence une variable dans les requêtes PromQL (ou autres langages), les titres de panneaux, les liens, etc., en utilisant la syntaxe $nom_variable ou ${nom_variable}.98 Si une variable peut avoir plusieurs valeurs (option "Multi-value" cochée), Grafana adapte la syntaxe pour la source de données (ex: instance=~"$instance" pour Prometheus, où $instance sera remplacé par val1|val2|...).104 L'option "Include All option" ajoute une valeur spéciale "All".105



Templating (Panneaux/Lignes Répétables) : En combinant les variables (surtout celles à valeurs multiples) avec la fonctionnalité "Repeat" d'un panneau ou d'une ligne, on peut générer dynamiquement des visualisations pour chaque valeur sélectionnée de la variable.98 Par exemple, répéter un panneau d'utilisation CPU pour chaque instance sélectionnée dans une variable instance.


Annotations : Les annotations permettent de superposer des événements sur les graphiques temporels.16 Elles peuvent marquer des déploiements, des changements de configuration, des alertes déclenchées, ou des incidents. Les annotations peuvent être ajoutées manuellement via l'UI de Grafana (clic sur le graphique), via l'API HTTP de Grafana, ou automatiquement depuis une source de données (comme Prometheus lui-même, en requêtant les alertes ALERTS ou ALERTS_FOR_STATE, ou une autre base de données contenant des événements).


Dashboards Communautaires : Grafana.com héberge une vaste bibliothèque de dashboards partagés par la communauté et les éditeurs.100 On peut les rechercher par source de données (Prometheus), par application (Node Exporter, Kubernetes, etc.) et les importer directement dans son instance Grafana via leur ID ou en uploadant le fichier JSON.68 C'est un excellent moyen de démarrer rapidement, mais il faut souvent les adapter à sa propre configuration (noms de jobs, labels spécifiques, nom de la source de données).89


TP Fil Rouge - Étape 7 : Création d'un Dashboard Grafana pour Node Exporter

Créer un Dashboard : Aller dans Dashboards -> New -> New Dashboard.
Ajouter une Variable instance :

Cliquer sur l'icône engrenage (Dashboard settings) en haut à droite -> Variables -> Add variable.
Name : instance.
Type : Query.
Label : Instance.
Data source : Prometheus-Local.
Query : label_values(node_exporter_build_info, instance) (utilise une métrique qui existe pour chaque instance node_exporter).
Refresh : On Dashboard Load.
Sort : Alphabetical (asc).
Activer "Multi-value" et "Include All option".
Cliquer sur "Add" puis "Save dashboard" (donner un nom, ex: "Node Exporter TP").
Vérifier que la liste déroulante "Instance" apparaît en haut du dashboard avec l'IP de la VM.


Ajouter une Ligne "CPU" : Cliquer sur "Add panel" -> "Add new row". Nommer la ligne "CPU".
Ajouter Panneau Utilisation CPU (%) :

Cliquer sur "Add panel" DANS la ligne CPU.
Data source : Prometheus-Local.
Query (mode Code) :
Extrait de code100 - (avg by (instance) (rate(node_cpu_seconds_total{mode="idle", instance=~"$instance"}[5m])) * 100)

57
Justification : Utilise rate() sur le temps idle, calcule la moyenne par instance (nécessaire si plusieurs CPU), soustrait de 100 pour obtenir l'utilisation non-idle, et multiplie par 100 pour le pourcentage. Le filtre instance=~"$instance" utilise la variable créée.
Visualization : Choisir "Time series" (Graph).
Panel options (droite) :

Title : CPU Usage %.
Standard options -> Unit : Percent (0-100).60
Legend -> Values : Cocher "Last *" (pour voir la dernière valeur).


Cliquer "Apply". Redimensionner/placer le panneau.


Ajouter Ligne "Memory" : "Add panel" -> "Add new row". Nommer "Memory".
Ajouter Panneau Utilisation Mémoire (%) :

Ajouter un panneau dans la ligne Memory.
Query (mode Code) :
Extrait de code100 * (1 - (node_memory_MemAvailable_bytes{instance=~"$instance"} / node_memory_MemTotal_bytes{instance=~"$instance"}))

23
Justification : Calcule la fraction de mémoire non disponible (MemAvailable est plus pertinent que MemFree) et la convertit en pourcentage utilisé.
Visualization : Choisir "Gauge".
Panel options :

Title : Memory Usage %.
Standard options -> Unit : Percent (0-100).
Standard options -> Min : 0, Max : 100.
Thresholds : Ajouter des seuils (ex: 80 pour orange, 90 pour rouge).
Value options -> Calculation : Last * (non-null).


Cliquer "Apply".


Ajouter Ligne "Disk" : "Add panel" -> "Add new row". Nommer "Disk".
Ajouter Variable mountpoint :

Dashboard settings -> Variables -> Add variable.
Name : mountpoint.
Type : Query.
Label : Mount Point.
Data source : Prometheus-Local.
Query : label_values(node_filesystem_size_bytes{instance=~"$instance", fstype!~"tmpfs|squashfs"}, mountpoint) (filtre les FS temporaires).
Refresh : On Dashboard Load.
Sort : Alphabetical (asc).
Activer "Multi-value" et "Include All option".
Cliquer "Add". Réorganiser pour que instance soit avant mountpoint. Sauvegarder.


Ajouter Panneau Utilisation Disque (%) - Répétable :

Ajouter un panneau dans la ligne Disk.
Query (mode Code) :
Extrait de code100 * (node_filesystem_size_bytes{instance=~"$instance", mountpoint=~"$mountpoint", fstype!~"tmpfs|squashfs"} - node_filesystem_avail_bytes{instance=~"$instance", mountpoint=~"$mountpoint", fstype!~"tmpfs|squashfs"}) / node_filesystem_size_bytes{instance=~"$instance", mountpoint=~"$mountpoint", fstype!~"tmpfs|squashfs"}

75 (Formule légèrement différente mais équivalente pour l'utilisation).
Justification : Calcule (Taille - Dispo) / Taille * 100 pour chaque point de montage sélectionné via la variable $mountpoint.
Visualization : Choisir "Gauge".
Panel options :

Title : Disk Usage $mountpoint %. (Utilise la variable dans le titre).
Standard options -> Unit : Percent (0-100).
Standard options -> Min : 0, Max : 100.
Thresholds : Ajouter des seuils (ex: 85, 95).
Value options -> Calculation : Last * (non-null).


Repeat options (en bas de l'éditeur) :

Repeat by variable : Sélectionner mountpoint.
Repeat direction : Horizontal.


Cliquer "Apply". Le panneau devrait se répéter pour chaque point de montage.


Sauvegarder : Cliquer sur l'icône "Save dashboard".
Tester : Utiliser les listes déroulantes "Instance" et "Mount Point" pour filtrer les données.
(Optionnel) Importer Dashboard Communautaire :

Aller dans Dashboards -> New -> Import.
Entrer l'ID 1860 (ou un autre ID trouvé sur Grafana.com pour Node Exporter Full) dans le champ "Import via grafana.com".100 Cliquer Load.
Choisir un nom (ex: "Node Exporter Full (Community)"). Sélectionner la source de données "Prometheus-Local". Cliquer Import.
Comparer ce dashboard pré-fait avec celui créé manuellement.







6. Sécurisation avec Nginx Reverse Proxy


Rôle : Un reverse proxy comme Nginx se place devant les applications web (ici, Prometheus et Grafana) pour intercepter les requêtes des clients. Il offre plusieurs avantages :

Sécurité : Il peut masquer la topologie interne du réseau, gérer le chiffrement SSL/TLS, appliquer des règles d'accès (authentification, filtrage IP).
Load Balancing : Répartir la charge entre plusieurs instances d'une application (non pertinent pour notre TP simple).
Terminaison SSL : Décharger les applications de la gestion du chiffrement/déchiffrement SSL.
Caching, Compression : Améliorer les performances.



Installation (Docker Compose) : Intégrer Nginx comme un autre service dans le fichier docker-compose.yml.


Configuration Nginx (nginx.conf) : La configuration se fait via des directives dans des blocs. Les blocs clés sont http, server (définit un serveur virtuel écoutant sur un port/domaine) et location (définit comment traiter les requêtes pour une URI spécifique).


Proxy pour Prometheus/Grafana : On utilise la directive proxy_pass dans un bloc location pour rediriger la requête vers le service backend (Prometheus ou Grafana). Il est crucial de gérer correctement les chemins d'URL si les services sont exposés sous un sous-chemin (ex: /prometheus/, /grafana/) via la directive rewrite ou la configuration de l'application elle-même (pour Grafana, via root_url dans grafana.ini).131 Il faut aussi transmettre les bons headers HTTP au backend (Host, X-Real-IP, X-Forwarded-For, X-Forwarded-Proto) pour que l'application sache d'où vient la requête originale.131


Authentification : Nginx peut ajouter une couche d'authentification basique (via auth_basic et auth_basic_user_file) devant des services qui n'en ont pas nativement, comme Prometheus.133


SSL/TLS : Pour sécuriser les connexions avec HTTPS :

Certificats : Pour le TP, on génère des certificats auto-signés avec l'outil openssl. En production, on utiliserait des certificats émis par une autorité de certification (ex: Let's Encrypt).134
Configuration Nginx : Les directives ssl_certificate et ssl_certificate_key pointent vers les fichiers de certificat et de clé privée. D'autres directives (ssl_protocols, ssl_ciphers, etc.) configurent les paramètres de sécurité TLS.134
Redirection HTTP->HTTPS : Il est recommandé de rediriger automatiquement les requêtes HTTP (port 80) vers HTTPS (port 443) pour forcer l'utilisation de connexions sécurisées.136



TP Fil Rouge - Étape 8 : Configuration de Nginx en Reverse Proxy

Ajouter le service Nginx à docker-compose.yml :
YAMLservices:
  #... (prometheus, grafana, portainer, node_exporter...)

  nginx:
    image: nginx:latest
    container_name: nginx
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    volumes:
      -./nginx/nginx.conf:/etc/nginx/nginx.conf:ro # Config Nginx
      -./nginx/certs:/etc/nginx/certs:ro       # Certificats SSL
    depends_on:
      - prometheus
      - grafana

Justification : Utilisation de l'image Nginx standard, mapping des ports HTTP/HTTPS, montage en lecture seule (:ro) de la configuration et des certificats locaux, et dépendance pour s'assurer que les backends sont démarrés avant Nginx.
Créer les répertoires : mkdir -p nginx/certs.
Générer le certificat auto-signé :
Bashopenssl req -x509 -nodes -newkey rsa:2048 -days 365 \
  -keyout nginx/certs/nginx-selfsigned.key \
  -out nginx/certs/nginx-selfsigned.crt \
  -subj "/C=FR/ST=IDF/L=Paris/O=Training/CN=localhost"

135 (Le -subj évite les questions interactives).
Créer nginx/nginx.conf :
Nginxworker_processes auto;
events { worker_connections 1024; }

http {
    # Définir les serveurs upstream (Prometheus et Grafana)
    upstream prometheus_backend {
        server prometheus:9090;
    }
    upstream grafana_backend {
        server grafana:3000;
    }

    # Redirection HTTP vers HTTPS
    server {
        listen 80;
        server_name localhost; # Ou votre domaine si configuré
        return 301 https://$host$request_uri; # Redirection permanente
    }

    # Serveur HTTPS principal
    server {
        listen 443 ssl http2;
        server_name localhost; # Ou votre domaine

        # Configuration SSL
        ssl_certificate /etc/nginx/certs/nginx-selfsigned.crt;
        ssl_certificate_key /etc/nginx/certs/nginx-selfsigned.key;
        ssl_protocols TLSv1.2 TLSv1.3;
        ssl_prefer_server_ciphers off;
        # Ajouter d'autres paramètres SSL recommandés si nécessaire

        # Proxy vers Prometheus
        location /prometheus/ {
            # Supprimer le préfixe /prometheus/ avant de passer au backend
            rewrite ^/prometheus/(.*)$ /$1 break;
            proxy_pass http://prometheus_backend;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            # Optionnel: Ajouter Basic Auth pour Prometheus
            # auth_basic "Prometheus Restricted";
            # auth_basic_user_file /etc/nginx/htpasswd_prometheus;
        }

         # Proxy vers Grafana
        location /grafana/ {
            proxy_pass http://grafana_backend/; # Le / final est important ici
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            # Nécessaire pour les WebSockets de Grafana Live
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";
        }

        # Location racine (optionnel, pour une page par défaut)
        location / {
            return 404; # Ou servir une page statique
        }
    }
}

Justification : Définit les upstreams par nom de service Docker 132, configure la redirection 301 HTTP->HTTPS 136, active SSL avec les certificats montés 135, et configure les location pour Prometheus et Grafana avec proxy_pass, rewrite (pour Prometheus) et les headers nécessaires.132 Notez le / final dans proxy_pass pour Grafana, qui aide à gérer le chemin. Les headers Upgrade et Connection sont ajoutés pour Grafana Live (WebSockets).132
Redémarrer/Créer les services : docker-compose up -d --force-recreate nginx.
Tester :

Ouvrir http://localhost. Devrait rediriger vers https://localhost.
Ouvrir https://localhost/prometheus/. Accepter l'avertissement de sécurité. L'UI Prometheus devrait s'afficher.
Ouvrir https://localhost/grafana/. Accepter l'avertissement. L'UI Grafana devrait s'afficher.
(Important) Vérifier que Grafana fonctionne correctement sous ce sous-chemin. Si des ressources (CSS, JS) ne chargent pas, il faudra peut-être configurer l'option root_url dans la configuration de Grafana (via variables d'environnement ou fichier grafana.ini monté) pour lui indiquer qu'il est servi sous /grafana/. Pour ce TP, nous supposerons que cela fonctionne sans configuration supplémentaire de Grafana pour simplifier.






(b) Après-midi : Exploration d'Exporteurs (Approx. 20-25 Slides)

1. Écosystème des Exporteurs Prometheus

Rappel du Rôle : Les exporteurs sont des programmes intermédiaires qui s'exécutent à côté de l'application ou du système cible. Ils collectent les métriques natives de cette cible et les traduisent dans le format texte simple attendu par Prometheus, les exposant ensuite sur un endpoint HTTP (généralement /metrics) que Prometheus peut scraper.
Diversité : Il existe un très grand nombre d'exporteurs, couvrant presque toutes les technologies courantes. On trouve des exporteurs pour :

Bases de données : PostgreSQL, MySQL, MongoDB, Redis, Oracle, Cassandra, Elasticsearch, etc..17
Systèmes d'exploitation/Matériel : Node Exporter (Linux/Unix), Windows Exporter, SNMP Exporter (équipements réseau), IPMI exporter..17
Serveurs HTTP/Proxies : Nginx, Apache, HAProxy, Traefik.17
Systèmes de Messagerie : Kafka, RabbitMQ, NATS, Solace.17
Stockage : Ceph, GlusterFS.
Cloud Providers : AWS (CloudWatch Exporter), Azure, GCP.17
Orchestration : Kubernetes (kube-state-metrics, cAdvisor - souvent intégrés), Consul.17
Autres systèmes de monitoring : StatsD, Graphite, InfluxDB, Nagios.17
Spécifiques : Blackbox Exporter (sondage d'endpoints), JMX Exporter (applications Java).17


Trouver des Exporteurs : Les ressources principales sont la page "Exporters and Integrations" de la documentation Prometheus, le dépôt GitHub de l'organisation Prometheus, l'organisation Prometheus Community 139, et des sites comme ExporterHub.io.139 Grafana.com liste aussi des dashboards associés à des exporteurs spécifiques.
Choix et Bonnes Pratiques : Lors du choix d'un exporteur, vérifier s'il est activement maintenu (commits récents, issues traitées), sa popularité (stars/forks GitHub), la richesse et la pertinence des métriques exposées pour votre besoin, et la clarté de sa documentation de configuration.17 Privilégier les exporteurs "officiels" (maintenus par Prometheus ou Prometheus Community) ou ceux fournis par le vendeur du logiciel/matériel surveillé (ex: Nginx Inc. pour Nginx Exporter) lorsque disponibles.
Force de l'Écosystème : La force de Prometheus réside en grande partie dans cet écosystème d'exporteurs vaste et dynamique. Le format d'exposition simple et le modèle pull facilitent l'intégration. Cela a encouragé la communauté à développer et maintenir des exporteurs pour une multitude de technologies.17 Ainsi, il est rare de ne pas trouver d'exporteur pour un système donné. Si aucun n'existe, il est relativement aisé d'en développer un en utilisant les bibliothèques client Prometheus. Cette adaptabilité est une clé majeure de l'adoption de Prometheus.



2. Postgres Exporter


Présentation : L'exporteur le plus commun est prometheus-community/postgres_exporter.139 Il se connecte à une ou plusieurs instances PostgreSQL pour en extraire des métriques via des requêtes SQL.


Installation : Peut être déployé comme binaire autonome ou, plus pratiquement pour notre TP, comme conteneur Docker.140


Configuration :

Connexion DB : La méthode la plus simple est de définir la variable d'environnement DATA_SOURCE_NAME avec l'URL de connexion PostgreSQL au format postgresql://[user[:password]@][host][:port][/database][?parameter_list].139 Il est fortement recommandé de ne pas utiliser l'utilisateur postgres mais de créer un utilisateur dédié.
Utilisateur DB Dédié : Créer un utilisateur spécifique (ex: monitor) dans PostgreSQL. Pour PostgreSQL 10 et plus, lui accorder le rôle prédéfini pg_monitor est la méthode recommandée et la plus simple : CREATE ROLE monitor WITH LOGIN PASSWORD 'secret'; GRANT pg_monitor TO monitor;.138 Pour les versions antérieures, des GRANT plus spécifiques sur les vues pg_stat_database, pg_stat_activity, etc., sont nécessaires. Le rôle pg_monitor donne accès à un ensemble de vues et fonctions utiles pour la supervision sans accorder de droits superutilisateur.145
Requêtes Personnalisées : L'exporteur utilise un ensemble de requêtes par défaut, mais on peut étendre ou remplacer ces requêtes via des fichiers YAML spécifiés avec --extend.query-path pour collecter des métriques spécifiques à l'application.146



Métriques Clés :

Disponibilité : pg_up (Gauge) : 1 si l'exporteur a pu se connecter à l'instance, 0 sinon.139
Connexions : pg_stat_database_numbackends (Gauge) : Nombre actuel de connexions par base de données. pg_settings_max_connections (Gauge) : Limite configurée.139
Transactions : pg_stat_database_xact_commit (Counter), pg_stat_database_xact_rollback (Counter) : Nombre total de transactions validées/annulées par base.140 Utile avec rate() pour voir le débit de transactions.
Activité Base : pg_stat_database_blks_read (Counter, blocs lus depuis disque), pg_stat_database_blks_hit (Counter, blocs lus depuis le cache), pg_stat_database_tup_returned (Counter, lignes retournées par les requêtes), pg_stat_database_tup_fetched (Counter, lignes lues), pg_stat_database_tup_inserted, _updated, _deleted (Counters).138 Permettent de calculer le taux de hit cache, l'activité I/O, etc.
Verrous (Locks) : pg_locks_count (Gauge) : Nombre de verrous par type et par base.143
Réplication : pg_stat_replication_..., pg_replication_slots_... : Métriques sur l'état de la réplication (lag, slots).143
Taille : pg_database_size_bytes (Gauge) : Taille de chaque base.139
Autres : Métriques sur le cache, le checkpoint, le background writer, etc.



TP Fil Rouge - Étape 9 : Monitoring PostgreSQL

Prérequis : S'assurer que PostgreSQL est installé et fonctionnel sur la VM Debian. Si ce n'est pas le cas : sudo apt update && sudo apt install postgresql postgresql-contrib -y. Vérifier le statut : sudo systemctl status postgresql.
Créer l'utilisateur Monitor : Se connecter à psql en tant qu'utilisateur postgres : sudo -u postgres psql. Puis exécuter :
SQLCREATE USER monitor WITH LOGIN PASSWORD 'YourSecurePassword'; -- Remplacer par un mot de passe sûr
GRANT pg_monitor TO monitor;
\q

138
Ajouter postgres-exporter à docker-compose.yml :
YAMLservices:
  #... (autres services)...

  postgres-exporter:
    image: quay.io/prometheuscommunity/postgres-exporter:latest
    container_name: postgres-exporter
    restart: unless-stopped
    environment:
      # Adapter l'URL avec l'IP de la VM, le bon user/password, et la base (ex: postgres)
      - DATA_SOURCE_NAME=postgresql://monitor:YourSecurePassword@<IP_VM_DEBIAN>:5432/postgres?sslmode=disable
    # Pas besoin de ports mappés, Prometheus le scrape via le réseau interne Docker

140
Configurer Prometheus pour scraper l'exporteur : Ajouter un job dans prometheus.yml :
YAMLscrape_configs:
  #... (autres jobs)...

  - job_name: 'postgres_exporter'
    static_configs:
      - targets: ['postgres-exporter:9187'] # Port par défaut de l'exporteur


Redémarrer les services concernés : docker-compose up -d --force-recreate postgres-exporter prometheus.
Vérification :

UI Prometheus -> Status -> Targets : Vérifier que postgres_exporter est UP.
UI Prometheus -> Graph : Explorer les métriques commençant par pg_. Essayer pg_up (devrait être 1), pg_stat_database_numbackends.


Ajouter un panneau à Grafana :

Ouvrir le dashboard "Node Exporter TP".
Ajouter une nouvelle ligne "PostgreSQL".
Ajouter un panneau "Gauge" pour les connexions actives :

Requête : sum(pg_stat_database_numbackends{datname="postgres"}) (adapter datname si besoin).
Titre : Active Connections.
Unit : None.


Ajouter un panneau "Time series" pour le taux de commit/rollback :

Requête A : rate(pg_stat_database_xact_commit{datname="postgres"}[5m]) -> Légende: Commits/sec
Requête B : rate(pg_stat_database_xact_rollback{datname="postgres"}[5m]) -> Légende: Rollbacks/sec
Titre : Transaction Rate.


Sauvegarder le dashboard.







3. Nginx Exporter

Présentation : L'exporteur officiel maintenu par Nginx Inc. est nginxinc/nginx-prometheus-exporter.147 Il peut scraper soit le module stub_status (pour Nginx OSS et Plus), soit l'API de Nginx Plus pour des métriques beaucoup plus détaillées.
Prérequis Nginx (stub_status) : Le module ngx_http_stub_status_module doit être activé dans la configuration Nginx. Il expose un endpoint (par convention /stub_status) qui affiche quelques métriques basiques. Il est crucial de restreindre l'accès à cet endpoint (par exemple, autoriser uniquement l'IP de l'exporteur ou localhost).147

Exemple de configuration Nginx :
Nginxserver {
    listen 8080; # Ou un autre port interne
    server_name _;
    location /stub_status {
        stub_status on;
        access_log off;
        allow 127.0.0.1; # Autoriser localhost
        allow 172.16.0.0/12; # Exemple: Autoriser le réseau Docker
        deny all; # Interdire tout le reste
    }
}




Installation Exporteur : Binaire ou conteneur Docker.147
Configuration Exporteur : L'argument principal est --nginx.scrape-uri qui doit pointer vers l'URL où stub_status est exposé (ex: http://nginx_service_name:8080/stub_status). Pour Nginx Plus, utiliser --nginx.plus et pointer vers l'URL de l'API (ex: /api).147
**Métriques Clés (via stub_status) :


