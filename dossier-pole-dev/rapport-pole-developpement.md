# Rapport de pôle — Développement
## Application Lidl Collect · Drive & Click and Collect
**Réponse à l'appel d'offre Lidl France · Pôle Développement · Avril 2026**

---

## Synthèse exécutive

Lidl France aborde un tournant stratégique majeur : après deux exercices déficitaires consécutifs et une perte documentée de 400 000 clients entre 2022 et 2025, l'enseigne investit 200 millions d'euros dans la reconquête de sa clientèle et accélère son repositionnement vers un segment discount-premium. Dans ce contexte, l'absence de Drive alimentaire représente une lacune compétitive réelle face à Leclerc Drive, Courses U et Intermarché Drive, opérateurs qui captent une clientèle organisée que Lidl ne peut pas encore fidéliser numériquement.

La solution proposée, **Lidl Collect**, est une WebApp de Drive et Click & Collect conçue pour combler ce manque tout en servant l'expérience de marque rénovée. Elle couvre l'intégralité du parcours : de la navigation catalogue à la confirmation de retrait, côté client comme côté magasin. Deux modes de retrait sont intégrés, piéton sur comptoir dédié et Drive voiture par scan QR, afin de répondre à la diversité des profils utilisateurs identifiés dans le cahier des charges.

L'équipe livre une application sécurisée, conforme RGPD, déployée sur une infrastructure conteneurisée avec pipeline d'intégration et de déploiement continus. Les livrables couvrent l'ensemble de la chaîne technique : maquettes, frontend, API, base de données, authentification, paiement, cybersécurité, infrastructure et réseau physique magasin.

---

## 1. Compréhension du contexte et de l'enjeu

### 1.1 Ce que le Drive représente réellement pour Lidl

La demande de Drive alimentaire chez Lidl ne naît pas d'une opportunité digitale abstraite. Elle répond à une pression concurrentielle concrète et à un comportement client en rupture avec le modèle historique de l'enseigne. Les profils identifiés: familles avec contrainte de temps, jeunes actifs avec liste préparée, retraités habitués au numérique depuis le Covid partagent un point commun : ils ont développé une pratique du Drive chez la concurrence qu'aucune action marketing ne remplacera tant qu'une solution équivalente n'existera pas chez Lidl.

Le magasin de Saint-Martin-d'Hères illustre bien cette tension. Avec 35 % de 15-29 ans dans sa zone de chalandise, 20 000 étudiants sur le campus UGA et un taux de pauvreté de 19 %, il dessert une clientèle structurellement sensible au prix mais exigeante sur la fluidité du service. Cette clientèle fréquente le Lidl pour ses prix ; elle fréquentera l'Intermarché Hyper voisin pour son Drive, à moins que Lidl Collect existe.

### 1.2 La tension discount-premium comme contrainte d'architecture

Le repositionnement de Lidl ne change ni les prix ni les produits. Il change l'expérience perçue. Pour le pôle Développement, cette contrainte a une traduction directe : l'application ne peut pas ressembler à un outil utilitaire ordinaire. L'interface doit incarner le territoire "Smart Shopper" défini par la direction artistique: fluide, moderne, accessible à toutes les générations, tout en étant assez robuste pour gérer les contraintes opérationnelles d'un magasin en charge réelle.

L'enjeu n'est pas seulement de livrer des features. C'est de livrer une expérience qui renforce la confiance dans la marque à chaque interaction : à la commande, au paiement, à la notification de préparation, et au retrait.

### 1.3 Un service critique dès le premier jour

Contrairement à une refonte graphique ou à une campagne sociale, une application de Drive est un service opérationnel. Une heure d'indisponibilité pendant un créneau de retrait ne se traduit pas seulement par un incident technique, elle génère des files, des frustrations en magasin et des retours sur les réseaux sociaux. La criticité du service impose des choix d'architecture qui ne peuvent pas être ajustés après le lancement : fiabilité du paiement, synchronisation du stock en temps réel, robustesse du QR code Drive en condition réseau dégradée. Ces contraintes structurent l'ensemble des décisions techniques présentées dans ce rapport.

---

## 2. Périmètre fonctionnel et architecture de la solution

### 2.1 Parcours client : de la liste à la confirmation de retrait

Le parcours standard commence par la navigation dans le catalogue, organisé par rayons, avec recherche textuelle et fiches produits enrichies (poids, allergènes, origines). L'ajout au panier déclenche une vérification de disponibilité en temps quasi-réel : le stock affiché reflète l'état du magasin sélectionné, pas un stock virtuel global.

Le tunnel de paiement est séquencé en trois étapes claires : vérification du panier, choix du créneau de retrait, paiement sécurisé. À l'issue de la commande, le client reçoit un QR code unique horodaté. Les notifications push ou SMS l'informent des changements de statut : commande reçue, en cours de préparation, prête au retrait.

La gestion des ruptures de stock fait partie du flux nominal, pas de l'exception. Lorsqu'un article n'est plus disponible après la commande, le système propose une substitution et demande une confirmation explicite du client : refus ou acceptation entraîne une mise à jour du montant et une notification immédiate. Ce mécanisme protège à la fois l'expérience client et l'organisation interne du préparateur.

### 2.2 Parcours opérationnel magasin

L'interface préparateur est conçue pour l'efficacité en conditions réelles : écran vertical, lecture rapide des listes de préparation, scan produit pour validation, signalement des ruptures en un geste. Les commandes arrivent par ordre de créneau de retrait et sont priorisées automatiquement. Le préparateur n'a pas à gérer la logique métier il exécute une liste structurée que le système a déjà ordonnancée.

L'interface manager donne une vision en temps réel : nombre de commandes en cours, taux de préparation, alertes sur les créneaux saturés, gestion du planning des préparateurs. Les KPIs sont accessibles sans manipulation complexe, le manager de magasin n'est pas un analyste data.

La distinction entre ces deux interfaces n'est pas cosmétique : elles répondent à des besoins métier différents, avec des droits d'accès distincts et des données exposées différemment. Cette séparation est intégrée dès la conception du modèle de données et de la matrice des rôles.

### 2.3 Les deux modes de retrait et leurs contraintes distinctes

**Mode piéton — Click & Collect comptoir.** Le client présente son QR code à une caisse dédiée. Le caissier scanne, valide, remet les courses. La contrainte principale est la fiabilité de la validation QR côté réseau interne magasin : le scan doit fonctionner même en cas de latence sur la connexion WAN.

**Mode Drive — retrait voiture.** Le client arrive à une borne extérieure, scanne son QR code. Le préparateur est notifié et apporte la commande au véhicule. Ce mode introduit plusieurs contraintes supplémentaires : la borne est physiquement à l'extérieur, potentiellement sous couverture réseau dégradée. Le QR code Drive est donc conçu avec une signature HMAC, une expiration courte et une validation à usage unique, une tentative de rejeu échoue systématiquement. L'infrastructure réseau couvrant la zone Drive fait l'objet d'une conception dédiée (section 3.7).

Le schéma d'architecture applicative global distingue trois couches : l'interface client (WebApp React), la couche API (backend REST), et les interfaces métier (préparateur, manager, administration). La base de données est le référentiel central des stocks, commandes et utilisateurs. Le module d'authentification est transverse à tous les accès.

```
┌─────────────────────────────────────────────────────────┐
│                   Client — WebApp React                  │
│         Catalogue · Panier · Paiement · Suivi            │
└──────────────────────┬──────────────────────────────────┘
                       │ HTTPS / API REST
┌──────────────────────▼──────────────────────────────────┐
│                   Backend API REST                        │
│   Auth (JWT) · Logique métier · Gestion commandes         │
└────┬─────────────┬──────────────┬───────────────────────┘
     │             │              │
┌────▼────┐  ┌─────▼─────┐  ┌────▼────────────────────┐
│   BDD   │  │  Module   │  │   Interfaces métier      │
│  (PG)   │  │ Paiement  │  │  Préparateur · Manager   │
└─────────┘  └───────────┘  └─────────────────────────┘
                                       │
                              ┌────────▼────────┐
                              │  Réseau magasin  │
                              │  Bornes Drive    │
                              └─────────────────┘
```

---

## 3. Choix techniques et justifications

### 3.1 Frontend — React

React est retenu pour le frontend de Lidl Collect. Face aux alternatives principales que sont Vue.js et Angular, le choix s'appuie sur trois contraintes spécifiques au projet.

La première est la complexité des états applicatifs. Le panier, le suivi de commande en temps réel, les notifications d'état et la gestion des substitutions génèrent des états concurrents qui doivent rester cohérents sur l'ensemble de l'interface. L'écosystème React notamment via des librairies de gestion d'état matures est éprouvé sur ce type de complexité. Vue.js aurait été viable sur une application plus simple ; sur Lidl Collect, la richesse de l'outillage React justifie le choix.
##########################!!!!!!!!!!#############################
La deuxième contrainte est le positionnement Desktop-first avec responsive progressif. La maquette principale cible un écran large ; la déclinaison mobile existe mais n'est pas le premier contexte d'usage. React et son écosystème de composants UI (notamment les librairies de catalogues et de grilles produits) s'adaptent naturellement à cette priorité. 
##########################!!!!!!!!!!#############################
La troisième est la maintenabilité dans un contexte multi-développeurs avec des interfaces distinctes (client, préparateur, manager). La séparation par composants de React permet à chaque interface d'être développée et testée indépendamment, avec des contrats d'interface API définis en amont.

### 3.2 Backend — API REST

L'architecture API REST est retenue face à GraphQL et au monolithe. Les raisons sont fonctionnelles avant d'être idéologiques.

GraphQL aurait présenté un avantage sur les requêtes flexibles côté catalogue, mais il ajoute une complexité de cache et de gestion des permissions difficile à maîtriser sur les endpoints critiques notamment le paiement et la validation QR code. REST permet de versionner les endpoints, de les documenter via Swagger, et de les exposer avec des politiques de cache distinctes selon leur criticité.

Le monolithe a été écarté pour une raison de maintenabilité : les interfaces préparateur et manager ont des cycles d'évolution différents de l'interface client. Une API REST découplée permet à chaque consommateur d'évoluer sans impacter les autres.

Les endpoints critiques en termes de charge liste catalogue, vérification de disponibilité, sont conçus avec une stratégie de cache explicite (TTL court, invalidation sur mise à jour de stock) pour absorber les pics de fréquentation sans solliciter la base de données à chaque requête.

### 3.3 Base de données — PostgreSQL

Le modèle relationnel est le seul adapté aux contraintes transactionnelles de Lidl Collect. La gestion d'une commande implique des opérations ACID sur plusieurs tables simultanément : décrémentation du stock, création de la commande, enregistrement du paiement. Une rupture de cohérence dans cette séquence commande enregistrée mais stock non décrémenté, ou paiement validé mais commande absente — est un incident opérationnel majeur.

PostgreSQL est retenu pour sa conformité ACID native, ses performances sur les requêtes analytiques nécessaires à l'interface manager, et sa robustesse documentée en production sur des charges comparables à celle d'un service de Drive. La modélisation (MCD, MLD, MPD) est documentée et optimisée pour les requêtes critiques : recherche catalogue, récupération d'une commande par QR code, agrégats KPI manager.

### 3.4 Authentification et sécurité des accès

**Gestion des sessions — JWT.** La gestion des sessions repose sur deux niveaux de tokens complémentaires. L'Access Token, à durée de vie courte (15 minutes pour les clients et opérateurs, 10 minutes pour les administrateurs), accompagne chaque requête et valide les droits en temps réel. Le Refresh Token assure la continuité de session selon un calendrier différencié par sensibilité du rôle : 7 jours pour un client, 8 heures pour un opérateur ou manager, 4 heures pour un administrateur. Cette différenciation reflète le niveau de risque associé à chaque profil d'accès et suit les recommandations conjointes de l'ANSSI et de l'OWASP.

Les tokens sont stockés dans des cookies `httpOnly` et `Secure`, inaccessibles aux scripts côté navigateur. En cas de compromission d'un compte, la révocation des tokens actifs est immédiate et entraîne la déconnexion sur tous les appareils.

**Authentification multi-facteurs (MFA).** La MFA est obligatoire pour les trois rôles à accès sensible : opérateur, manager, administrateur. Elle est déclenchée sur les actions critiques pour les comptes clients (modification de mot de passe, accès aux données personnelles). Elle repose sur un OTP à usage unique transmis par SMS ou application d'authentification. L'intérêt est direct : même en cas de vol d'identifiants, l'accès au système reste protégé.

**Hachage des mots de passe.** L'algorithme Bcrypt avec sel aléatoire par utilisateur est retenu, conformément aux recommandations CNIL. Aucun mot de passe n'est stocké en clair ni récupérable par le système.

**Paiement.** Le traitement de la carte bancaire sera délégué à un prestataire de paiement certifié PCI-DSS. Ce choix est à la fois technique et juridique : stocker des données de carte en propre impose des certifications que l'équipe ne peut pas obtenir dans ce contexte, et expose Lidl à des risques de responsabilité en cas de compromission. L'application ne manipulera que des tokens opaques transmis par le prestataire après autorisation — aucune donnée bancaire brute ne transitera par l'infrastructure Lidl Collect.

### 3.5 Cybersécurité

La surface d'attaque de Lidl Collect est significative : données personnelles de clients, transactions financières, interface de gestion de stocks. L'approche adoptée suit l'OWASP Top 10 comme socle minimum, avec une attention particulière aux vecteurs suivants :

- **Injection SQL** : requêtes paramétrées systématiquement, ORM avec typage strict, aucune concaténation de chaîne dans les requêtes dynamiques.
- **XSS et CSRF** : en-têtes de sécurité HTTP (CSP, X-Frame-Options, HSTS), tokens CSRF sur les formulaires d'action, sanitisation des entrées utilisateur côté serveur.
- **Force brute sur l'authentification** : rate limiting sur les endpoints d'auth, verrouillage temporaire après tentatives répétées, alerting sur les patterns anormaux.
- **Non-rejeu du QR code Drive** : token à usage unique signé HMAC, expiration à 15 minutes, invalidation en base à la première validation. Une tentative de présentation d'un QR déjà utilisé ou expiré échoue sans message d'erreur exploitable.
- **Élévation de privilèges** : matrice RBAC complète, vérification des droits à chaque endpoint côté serveur (pas uniquement côté client), journalisation des accès aux ressources sensibles.

Le référent cybersécurité intervient en transverse sur tous les modules à risque : validation des schémas de données, revue du module d'authentification, audit du module de paiement, modélisation des menaces STRIDE avant la mise en production.

### 3.6 CI/CD — Docker et GitHub Actions

La conteneurisation via Docker répond à une contrainte concrète : garantir que l'environnement d'exécution en production est identique à celui de développement et de staging. Les dérives de configuration entre environnements sont une source documentée d'incidents en production un comportement différent sur la gestion des sessions JWT selon l'environnement, par exemple, peut ouvrir une faille invisible en développement.

Le pipeline GitHub Actions est structuré en quatre étapes séquentielles : build de l'image Docker, exécution des tests unitaires et d'intégration, déploiement automatique sur l'environnement de staging, déploiement en production sur validation manuelle. Cette dernière étape de validation reste humaine : un déploiement automatique en production sans revue sur un service de paiement actif est un risque opérationnel disproportionné.

La stratégie de rollback est définie avant le premier déploiement : chaque image Docker est taguée avec le hash du commit, permettant un retour en arrière en moins de cinq minutes sur n'importe quelle version stable.

### 3.7 Infrastructure réseau magasin

L'infrastructure réseau physique est la composante la moins visible du projet mais l'une des plus critiques pour l'expérience Drive. Un QR code que la borne extérieure ne parvient pas à valider parce que la connexion WiFi de la zone Drive est intermittente anéantit l'expérience utilisateur indépendamment de la qualité de l'application.

L'architecture réseau du magasin de référence (SMH Grenoble) sépare les flux : un VLAN dédié aux terminaux de caisse et bornes Drive, isolé du réseau d'usage général du magasin. Les bornes extérieures sont couvertes par des points d'accès WiFi industriels (WiFi 6, résistance aux intempéries) positionnés pour garantir une couverture nominale dans la zone Drive, avec redondance sur deux accès points distincts.

En cas de perte de la connexion WAN (liaison internet principale), le système de validation QR code bascule en mode dégradé local : les tokens récemment émis sont mis en cache chiffré sur la borne, permettant la validation pendant une fenêtre de 30 minutes sans connexion active. Au-delà, la borne affiche un message opérateur et la caisse standard prend le relais pour le retrait manuel.

---

## 4. Organisation de l'équipe et gouvernance

### 4.1 Matrice de responsabilités orientée risques

L'organisation de l'équipe n'est pas une répartition de tâches arbitraire elle reflète la carte des risques du projet. Les composantes à plus forte criticité (paiement, authentification, réseau Drive) sont portées par des responsables identifiés avec un périmètre de décision autonome.

| Domaine | Responsable | Criticité | Dépendances principales |
|---------|-------------|-----------|------------------------|
| UX/UI | Gwen | Élevée — bloque le Frontend | Direction artistique Lidl 2.0 |
| Frontend React | Alex | Élevée | Maquettes (Gwen) + contrats API (Dorian) |
| Backend API REST | Dorian | Critique | BDD (Sabry), Auth (Willy), Sécu (Leo) |
| Base de données | Sabry | Critique | Backend (Dorian), Sécu (Leo) |
| Cybersécurité | Leo | Transverse | Tous les modules |
| Auth & Paiement | Willy | Critique | Backend (Dorian), Sécu (Leo) |
| Infra CI/CD | David | Élevée | Backend (Dorian), BDD (Sabry) |
| Réseau magasin | Patrice | Élevée | Indépendant applicatif |

La cybersécurité et l'authentification ont un statut transverse : Leo et Willy interviennent en revue sur tous les modules qui manipulent des données sensibles, et non uniquement sur leur périmètre nominal. Cette organisation évite le cloisonnement qui laisse des failles entre les couches.

### 4.2 Méthodologie et synchronisation

La méthode retenue est Scrum avec des sprints de deux semaines. Ce choix répond à une contrainte de parallélisation : les modules frontend, backend et infrastructure peuvent avancer simultanément à condition que les contrats d'interface soient définis et figés au sprint 1. Un contrat d'API modifié en cours de développement sans coordination génère des régressions coûteuses.

La convention adoptée est la suivante : les endpoints API sont définis et documentés (Swagger) avant le début du sprint de développement frontend correspondant. Toute modification de contrat passe par une revue d'impact formelle. Les maquettes de Gwen sont validées avant le début du sprint d'intégration d'Alex.

Les rituels de synchronisation daily de 15 minutes, rétrospective en fin de sprint ont pour seul objectif de détecter les dépendances bloquantes au plus tôt. Ils ne remplacent pas la communication bilatérale entre modules couplés.

---

## 5. Planning et jalons de livraison

| Jalon | Période | Livrables | Critères d'acceptation |
|-------|---------|-----------|----------------------|
| **J1 — Fondations** | S1–S2 | Maquettes UI validées · Contrats API Swagger · Modèle de données validé · Environnements Docker opérationnels | Maquettes approuvées par direction artistique · API contracts signés par Frontend et Backend |
| **J2 — MVP fonctionnel** | S3–S6 | Catalogue navigable · Panier · Authentification client · Tunnel paiement (sandbox) | Tests d'intégration Frontend/Backend · Paiement validé en environnement de test |
| **J3 — Interfaces métier** | S7–S9 | Interface préparateur complète · Interface manager · Notifications temps réel · Paiement production | Tests de charge sur endpoints critiques · Validation RBAC complète |
| **J4 — Drive et conformité** | S10–S12 | Module QR code Drive · Infrastructure réseau magasin · Audit cybersécurité · Registre RGPD | QR code validé en condition réseau dégradée · Audit OWASP sans finding critique |
| **J5 — Recette et livraison** | S13–S14 | Tests de charge complets · Documentation technique · Déploiement production · Livraison | Couverture de tests ≥ 80 % · Temps de réponse API < 200ms au 95e percentile |

---

## 6. Gestion des risques et conformité RGPD

### 6.1 Matrice des risques techniques et opérationnels

| Risque | Probabilité | Impact | Mesure de mitigation |
|--------|-------------|--------|---------------------|
| Panne API de paiement en pic de charge | Faible | Critique | Circuit breaker + fallback message d'erreur explicite + alerting PagerDuty-compatible |
| Désynchronisation stock en temps réel | Moyenne | Élevé | Cache avec TTL de 2 minutes + réconciliation toutes les 5 minutes + affichage "stock estimé" côté client |
| QR code Drive intercepté ou rejoué | Faible | Élevé | Token HMAC à usage unique, expiration 15 min, invalidation en base à la première validation |
| Indisponibilité réseau zone Drive | Moyenne | Élevé | Mode dégradé local avec cache chiffré, fenêtre de 30 min, bascule manuelle caisse |
| Surcharge catalogue en heure de pointe | Moyenne | Moyen | Mise en cache des pages catalogue, pagination côté serveur, rate limiting par IP |
| Compromission d'un compte préparateur | Faible | Élevé | 2FA obligatoire sur comptes métier, révocation immédiate, journalisation des accès |

### 6.2 Conformité RGPD

Lidl Collect collecte et traite des données personnelles de clients français au sens du RGPD : adresse email, adresse de livraison, historique de commandes, tokens de paiement. Les engagements de conformité sont les suivants.

**Durées de conservation définies par catégorie :**
- Données de compte actif : conservation pendant la durée du compte + 1 an après désinscription
- Historique de commandes : 3 ans (obligation comptable), puis suppression automatique
- Logs d'accès et de sécurité : 6 mois, puis purge automatique
- Tokens de paiement : supprimés immédiatement après expiration ou révocation par le prestataire

**Droit à l'effacement implémenté techniquement.** La suppression d'un compte entraîne une cascade d'effacement dans toutes les tables liées (commandes, adresses, tokens de fidélité), à l'exception des données soumises à obligation légale de conservation. Ce flux est testé et documenté, pas seulement mentionné.

**Chiffrement au repos et en transit.** Les données en base sont chiffrées au repos. Toutes les communications client-serveur passent par HTTPS avec TLS 1.3 minimum. Aucune donnée personnelle ne transite en clair dans les logs applicatifs.

**Données bancaires.** Aucune donnée de carte bancaire n'est stockée ou transitée par l'infrastructure Lidl Collect. Le prestataire PCI-DSS certifié gère l'intégralité du traitement. L'application ne manipule que des tokens opaques fournis par le prestataire après autorisation.

Un registre des traitements est tenu à jour pour chaque catégorie de donnée, avec base légale, finalité, durée de conservation et destinataires identifiés. Ce registre est un livrable formel du projet.

---

## 7. Engagements

L'équipe s'engage sur les livrables suivants, dans les conditions décrites dans ce rapport :

- Une WebApp fonctionnelle couvrant l'intégralité du périmètre fonctionnel défini en section 2, déployée sur infrastructure conteneurisée avec pipeline CI/CD opérationnel.
- Un système d'authentification et de paiement sécurisé, délégué à un prestataire PCI-DSS certifié, avec gestion JWT et 2FA sur les comptes métier.
- Une conformité RGPD documentée : registre des traitements, politique d'effacement implémentée, durées de conservation définies et appliquées.
- Une documentation technique complète : Swagger API, MCD/MLD/MPD, Dockerfiles, schéma réseau, matrice RBAC.

Les choix techniques présentés dans ce rapport sont argumentés mais discutables. Plusieurs arbitrages — notamment le choix du prestataire de paiement, la politique de cache catalogue ou la stratégie de déploiement — peuvent être révisés en fonction des contraintes opérationnelles ou contractuelles de Lidl. L'équipe est disponible pour défendre et, si nécessaire, ajuster ses décisions face aux retours du jury.
