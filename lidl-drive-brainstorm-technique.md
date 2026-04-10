# Brainstorming technique — Webapp Drive Lidl

---

## 1. Contexte du projet

Nous concevons une **webapp de type Drive / Click & Collect pour Lidl**.
L'idée principale est de permettre à un client de :
- choisir un magasin,
- consulter les produits disponibles,
- remplir un panier,
- réserver un créneau,
- valider une commande,
- venir récupérer ses courses.

Le sujet se rapproche d'un modèle **click & collect retail**, avec une logique de disponibilité locale, préparation de commande, retrait et suivi. D'après les ressources consultées, ce type de parcours inclut généralement consultation du catalogue, sélection d'un point de retrait, choix du créneau, confirmation de commande puis récupération sur place.[web:18][web:22]

En parallèle, l'écosystème Lidl existant comprend déjà des fonctions utiles comme le compte client, les informations magasin, la fidélité et les tickets numériques, ce qui peut inspirer certaines idées sans forcément tout intégrer au MVP.[web:17][web:21][web:25]

---

## 2. But du document

Ce fichier sert à :
- poser des **idées techniques**,
- répartir le travail entre les profils,
- cadrer un **MVP réaliste**,
- préparer la soutenance,
- garder une trace des décisions importantes.

Le choix du format Markdown est pertinent pour des documents techniques et d'architecture collaboratifs, car il reste léger, lisible dans Git et simple à faire évoluer dans le temps.[web:26][web:27][web:29]

---

## 3. Vision produit

Le projet ne doit pas être pensé seulement comme "un site e-commerce", mais comme un **système de drive**.
La différence est importante :
- le stock dépend du magasin,
- la commande doit être préparée,
- le retrait se fait sur un créneau,
- certaines ruptures peuvent apparaître avant la remise au client.

Cette logique est typique du click & collect en retail, où la gestion du point de retrait et du créneau est aussi importante que l'achat lui-même.[web:18][web:22]

---

## 4. Proposition de MVP

### MVP minimum

Fonctionnalités indispensables :
- authentification / compte client,
- choix du magasin,
- consultation du catalogue,
- recherche et filtres,
- panier,
- choix d'un créneau de retrait,
- validation de commande,
- suivi du statut de commande,
- interface d'administration / préparation.

### MVP intermédiaire

Fonctionnalités intéressantes si le temps le permet :
- gestion du stock par magasin,
- proposition de substitution produit,
- notification mail de confirmation,
- QR code ou code de retrait,
- tableau de bord opérateur magasin.

### MVP avancé

Fonctionnalités bonus :
- fidélité inspirée de Lidl Plus,
- coupons,
- analytics,
- recommandations,
- dashboard temps réel,
- paiement en ligne simulé.

Les fonctionnalités autour de la fidélité, du compte client et des tickets numériques s'inspirent de services déjà visibles dans l'univers Lidl Plus.[web:17][web:21][web:25]

---

## 5. Architecture globale proposée

## Vue simple

```text
[ Client web ]
      |
      v
[ Frontend Webapp ]
      |
      v
[ API Backend ]
  |       |        |
  |       |        +--> [ Service notifications ]
  |       +------------> [ Service auth ]
  |
  +--------------------> [ Base PostgreSQL ]
  +--------------------> [ Cache Redis ]

[ Admin magasin ] --> [ Interface admin ] --> [ API Backend ]

[ Reverse proxy / TLS ]
[ Logs / Monitoring / Healthchecks ]
```

## Vue plus réaliste

```text
Internet / Réseau utilisateur
        |
        v
[ Reverse Proxy / Nginx / Traefik ]
        |
  +-----+------------------+
  |                        |
  v                        v
[ Front client ]      [ Front admin ]
        \               /
         \             /
          v           v
            [ API Backend ]
            |     |      |
            |     |      +--> [ Notifications ]
            |     +---------> [ Auth / rôles ]
            |
            +-------------> [ PostgreSQL ]
            +-------------> [ Redis ]

Zone supervision :
[ Prometheus ] [ Grafana ] [ Logs ]
```

Une architecture de ce type correspond bien à une webapp modulaire : front, backend métier, base de données, authentification, notifications et supervision. Le schéma peut rester simple tout en étant suffisamment crédible pour un projet d'équipe.[web:28][cite:1]

---

## 6. Répartition par pôles

### Dev

L'équipe dev peut prendre en charge :
- frontend client,
- frontend admin,
- API backend,
- modélisation de la base,
- logique métier du panier, des commandes et des créneaux,
- tests front / back,
- intégration Docker et CI/CD.

### Cyber

L'équipe cyber peut travailler sur :
- analyse de menaces,
- gestion des rôles et permissions,
- sécurisation des API,
- validation des entrées,
- protection contre XSS / injection / brute force,
- journalisation des actions sensibles,
- politique de mots de passe,
- conformité RGPD simplifiée.

Le contexte Lidl/Schwarz met fortement en avant la cybersécurité et la gouvernance des données, ce qui en fait un axe cohérent à valoriser dans le projet.[web:16][web:20][web:24]

### Infra / réseau

L'équipe infra/réseau peut prendre en charge :
- docker-compose ou environnement de déploiement,
- reverse proxy,
- segmentation logique des services,
- variables d'environnement et secrets,
- supervision,
- sauvegarde de la base,
- stratégie de restauration,
- pipeline de déploiement.

---

## 7. Idées de stack

### Stack simple et réaliste

- **Frontend** : React + Vite ou Vue + Vite
- **Backend** : Node.js + Express ou NestJS
- **Base de données** : PostgreSQL
- **Cache** : Redis
- **Conteneurisation** : Docker + Docker Compose
- **Proxy** : Nginx ou Traefik
- **Auth** : JWT + refresh token
- **Tests** : Playwright + tests API
- **CI/CD** : GitHub Actions

### Stack plus “entreprise”

- React ou Vue
- NestJS
- PostgreSQL
- Redis
- Keycloak
- Traefik
- Prometheus / Grafana / Loki
- SAST / DAST dans la CI

Pour la doc produit et backlog, il est courant de structurer les besoins en user stories priorisées dans un backlog maintenu au départ en Markdown puis dans un outil de gestion de projet.[web:31][web:34][web:40]

---

## 8. Entités métier à prévoir

Entités principales :
- **User**
- **Role**
- **Store**
- **Product**
- **Category**
- **Inventory**
- **Cart**
- **CartItem**
- **Order**
- **OrderItem**
- **PickupSlot**
- **Payment**
- **Notification**
- **AuditLog**
- **SubstitutionProposal**

### Idée importante

Bien distinguer :
- le **catalogue global**,
- le **stock par magasin**,
- la **commande client**,
- la **préparation en magasin**.

Cette séparation permet de gérer des cas réalistes comme un produit visible au catalogue mais indisponible dans un magasin précis, ou un créneau disponible mais bientôt saturé.[web:18]

---

## 9. Flux utilisateur principal

```text
1. L'utilisateur se connecte
2. Il choisit un magasin
3. Il consulte les produits disponibles
4. Il ajoute des produits au panier
5. Il choisit un créneau de retrait
6. Le système vérifie stock + capacité
7. Il valide la commande
8. Le magasin prépare la commande
9. Le client reçoit une confirmation
10. Le client récupère sa commande
11. La commande passe en statut "retirée"
```

Ce parcours correspond au fonctionnement classique du click & collect : sélection, réservation, confirmation, retrait.[web:18][web:22]

---

## 10. Flux admin magasin

```text
1. L'opérateur se connecte à l'interface admin
2. Il voit les commandes à préparer
3. Il change les statuts (à préparer / en cours / prête)
4. Il signale une rupture éventuelle
5. Il propose un remplacement si nécessaire
6. Il valide la commande prête
7. Le client reçoit l'information de retrait
```

---

## 11. Cas techniques intéressants

### Gestion des créneaux

Questions à se poser :
- combien de commandes max par créneau ?
- combien de temps un panier réserve un créneau ?
- que se passe-t-il si deux clients réservent en même temps ?

### Gestion du stock

Questions à se poser :
- le stock est-il décrémenté au panier ou à la validation ?
- comment éviter de vendre la dernière unité deux fois ?
- comment gérer une rupture pendant la préparation ?

### Résilience métier

Cas à prévoir :
- produit indisponible,
- créneau plein,
- client absent,
- commande annulée,
- échec de notification,
- bug API,
- panne partielle du service.

---

## 12. Sécurité — pistes cyber

### Points à traiter

- authentification sécurisée,
- gestion des rôles,
- permissions admin / client,
- validation des données côté front et back,
- limitation de débit,
- logs de sécurité,
- journal d'audit,
- sécurité des conteneurs,
- protection des secrets,
- chiffrement en transit,
- hash des mots de passe,
- minimisation des données personnelles.

### Risques à présenter

- vol de compte,
- accès admin non autorisé,
- injection SQL,
- XSS,
- fuite de données client,
- déni de service sur l'API,
- falsification du statut d'une commande.

Les enjeux de cybersécurité et de conformité sont particulièrement crédibles ici, car Lidl et le groupe Schwarz communiquent déjà sur ces thèmes au niveau de leur environnement numérique.[web:16][web:20][web:24]

---

## 13. Infra / réseau — pistes techniques

### Ce que l'équipe infra peut montrer

- un schéma réseau simple,
- un reverse proxy,
- des services isolés par réseau Docker,
- une base non exposée publiquement,
- des healthchecks,
- du monitoring,
- des sauvegardes PostgreSQL,
- des variables d'environnement,
- une séparation dev / staging / prod si possible.

### Exemple de logique réseau

```text
[ Utilisateur ]
      |
      v
[ Reverse Proxy ]
      |
  +---+----------------+
  |                    |
  v                    v
[ Front ]         [ API Backend ]
                        |
                 +------+------+
                 |             |
                 v             v
           [ PostgreSQL ]   [ Redis ]

Réseau public : Reverse Proxy
Réseau applicatif : Front + API
Réseau privé : DB + cache
```

---

## 14. Observabilité / exploitation

Idées à intégrer :
- logs applicatifs structurés,
- métriques API,
- temps de réponse,
- taux d'erreur,
- nombre de commandes créées,
- health endpoint,
- dashboard de supervision,
- alertes simples.

Prévoir une partie supervision et exploitation renforce la crédibilité du projet technique, notamment pour la démonstration et la soutenance.[web:28][web:29]

---

## 15. Idées de backlog technique

### Front
- page d'accueil
- connexion / inscription
- choix du magasin
- liste produits
- fiche produit
- panier
- checkout / créneau
- suivi de commande
- espace profil

### Back
- auth API
- produits API
- stock API
- commandes API
- créneaux API
- notifications API
- audit API

### Admin
- connexion admin
- dashboard commandes
- changement de statut
- gestion des ruptures
- historique des actions

### Cyber
- matrice des rôles
- analyse de menaces
- checklist OWASP
- politique de logs
- revue des secrets

### Infra
- dockerisation
- reverse proxy
- monitoring
- backup DB
- pipeline CI/CD
- documentation d'architecture

---

## 16. Questions de cadrage à trancher

- Est-ce qu'on fait un **paiement en ligne** ou seulement un paiement simulé / au retrait ?
- Est-ce qu'on gère un **vrai stock par magasin** ou une simulation simplifiée ?
- Est-ce qu'on fait une **interface admin séparée** ou intégrée ?
- Est-ce qu'on veut une **auth complète** ou une version simple pour la démo ?
- Est-ce qu'on veut prioriser la **richesse fonctionnelle** ou la **qualité architecture / sécurité / infra** ?
- Est-ce qu'on veut un projet très propre techniquement ou très démonstratif visuellement ?

---

## 17. Proposition de répartition concrète

### Équipe dev
- front client,
- admin,
- API,
- BDD,
- tests.

### Équipe cyber
- auth,
- rôles,
- audit,
- analyse de menaces,
- recommandations sécurité,
- revue des endpoints.

### Équipe infra/réseau
- conteneurs,
- réseau,
- proxy,
- supervision,
- pipeline,
- sauvegardes,
- documentation d'exploitation.

---

## 18. Message fort pour la soutenance

Une bonne manière de présenter le projet :

> Nous n'avons pas seulement développé un site de courses. Nous avons conçu une plateforme de drive retail, avec des enjeux de disponibilité locale, de préparation magasin, de sécurité, d'observabilité et d'exploitation technique.

Cette formulation colle bien à la réalité du click & collect, où l'architecture, la logistique applicative et la sécurité sont aussi importantes que l'interface utilisateur.[web:18][web:22]

---

## 19. À compléter plus tard

Sections possibles à ajouter ensuite :
- schéma de base de données,
- backlog en user stories,
- diagrammes de séquence,
- matrice des rôles,
- ADR (Architecture Decision Records),
- checklist cybersécurité,
- plan de tests,
- plan de démo.

Les modèles de documents d'architecture et de design en Markdown recommandent justement d'inclure architecture globale, composants, flux, risques, déploiement et décisions importantes dans un format court mais évolutif.[web:26][web:28][web:30]

---

## 20. Notes libres

- Ajouter ici les idées du groupe
- Ajouter ici les décisions prises en réunion
- Ajouter ici les éléments à valider avec les profs
- Ajouter ici les contraintes de temps / techno

