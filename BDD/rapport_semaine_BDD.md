# Rapport hebdomadaire — Base de données | Click & Collect Drive (Lidl)

> **Projet :** Application web Click & Collect Drive — Appel d'offre Lidl  
> **Rôle :** Responsable base de données  
> **Période :** Semaine du 13 au 17 avril 2026  
> **Auteur :** Sabry Cheribet  
> **Stack BDD :** PostgreSQL · Supabase · Prisma

---

## Contexte du projet

Dans le cadre d'un appel d'offre lancé par **Lidl**, notre équipe développe une application web de **Click & Collect Drive** permettant aux clients de commander en ligne et de récupérer leurs achats en magasin sans contact.

L'équipe est composée de trois membres avec des périmètres distincts :

| Membre | Rôle |
|---|---|
| **Sabry Cheribet** | Base de données |
| **Léo** | (équipe) |
| **Dorian** | Back-end (Next.js) |

Ma mission couvre l'intégralité du domaine BDD : conception du schéma relationnel, mise en place sur Supabase, connexion avec le back-end, intégrité des données, stratégie de sauvegarde et conformité RGPD.

---

## Journal de bord

### Lundi 13 avril — Analyse & prise en main des outils

**Tâches réalisées :**
- Prise en main de **Supabase** (interface, SQL Editor, Table Editor, auth, storage) et de **Prisma** (ORM, schéma déclaratif, migrations)
- Analyse des besoins fonctionnels d'un supermarché/hypermarché type : flux de commandes, gestion des stocks, créneaux de retrait, personnel
- Collecte et analyse de données publiques disponibles sur le secteur de la grande distribution
- Agrégation des données en fichiers texte structurés pour construire la base du modèle
- Mise en accord avec Léo et Dorian pour valider les entités retenues et aligner les périmètres techniques

**Difficulté rencontrée :**  
L'analyse a été plus longue que prévu. Je n'ai pas pu finaliser la structure complète de la base de données, repoussant la liaison avec le back-end au lendemain. Journée frustrante sur ce point.

---

### Mardi 14 avril — Finalisation de la base de données

**Tâches réalisées :**
- Finalisation et consolidation de la structure complète du schéma relationnel
- Définition des tables, colonnes, types de données et premières contraintes
- Préparation à la connexion avec le back-end Next.js de Dorian
- Relecture collaborative avec Dorian pour corriger les fautes de frappe dans le schéma (noms de colonnes, types)

**Ressenti :**  
Journée posée et méthodique, un peu trop calme, mais nécessaire pour poser des bases solides.

---

### Mercredi 15 avril — Connexion BDD / Back-end & stratégie de sauvegarde

**Tâches réalisées :**
- **Connexion effective** de la base de données avec le back-end via les variables d'environnement Supabase (détail section [Synchronisation back-end](#synchronisation-avec-le-back-end))
- Corrections et mise en commun du schéma suite aux retours de l'intégration
- Ajustements itératifs des tables pour correspondre aux besoins du back-end (noms de champs, types, relations)
- Recherches approfondies sur les stratégies de sauvegarde adaptées au projet : **PITR** *(Point-In-Time Recovery)* via Supabase (détail section [Stratégie de sauvegarde](#stratégie-de-sauvegarde--pitr))

**Ressenti :**  
Journée très chargée. J'ai jonglé entre la finalisation de la connexion, le soutien au back-end et mes recherches sur les sauvegardes. Bordélique dans l'organisation, mais productive — une des journées les plus denses de la semaine.

---

### Jeudi 16 avril — Corrections finales & enrichissement des données

**Tâches réalisées :**
- Dernières corrections et ajustements du schéma de base de données
- Mise en place de l'**auto-incrémentation** (`GENERATED ALWAYS AS IDENTITY`) sur l'ensemble des identifiants de toutes les tables
- Ajout de produits réels dans la base de données
- Intégration des images produits via les URL Supabase Storage

---

### Vendredi 17 avril — Oral de présentation

*À compléter après la soutenance.*

---

## Architecture de la base de données

### Visualisation du schéma (ERD)

![Schéma ERD — Base de données Click & Collect Drive](./Order_Management_Ecosystem-2026-04-16-075932.png)

Le schéma complet comporte **18 tables actives** organisées autour de cinq domaines fonctionnels.

---

### Domaines fonctionnels

| Domaine | Tables |
|---|---|
| **Utilisateurs & Accès** | `client`, `client_account`, `client_history`, `permission` |
| **Personnel** | `preparer`, `manager`, `schedule`, `performance` |
| **Catalogue & Stock** | `product`, `category`, `stock` |
| **Commandes & Panier** | `order`, `order_item`, `cart`, `cart_item`, `substitution_proposal`, `payment` |
| **Logistique & Magasin** | `store`, `pickup_slot`, `notification`, `audit_log` |

---

### Choix de conception : séparation `client` / `client_account`

La table `client` et la table `client_account` sont volontairement séparées, ce qui peut sembler redondant au premier regard. Ce choix répond à plusieurs contraintes techniques et réglementaires.

**`client`** — contient l'**identité** du client : nom, prénom, email, mot de passe haché, téléphone, adresse, magasin préféré. Ce sont des données stables qui définissent *qui est* l'utilisateur.

**`client_account`** — contient l'**état du compte** : `is_verified`, `is_active`, `last_login_at`, dates de création et de mise à jour. Ce sont des données dynamiques qui définissent *l'état de la relation* entre l'utilisateur et le service.

Cette séparation apporte plusieurs avantages concrets :

- **Désactivation sans suppression** : on peut passer `is_active = false` pour suspendre un compte signalé sans toucher aux données d'identité ni casser les relations existantes.
- **Vérification progressive** : le flag `is_verified` permet de gérer les comptes créés mais non encore confirmés (validation par email) indépendamment du profil client.
- **Conformité RGPD** : en cas de demande d'effacement (Article 17 RGPD), on peut anonymiser les données PII de `client` tout en conservant `client_account` pour des besoins d'audit, sans rupture d'intégrité référentielle.
- **Principe de responsabilité unique** : chaque table a un périmètre clairement délimité, ce qui facilite la maintenance et l'évolution indépendante du schéma.

---

### Contraintes d'intégrité référentielle

Toutes les clés primaires utilisent `bigint GENERATED ALWAYS AS IDENTITY`, garantissant l'auto-incrémentation native PostgreSQL sans dépendance à une séquence externe.

#### Clés étrangères (Foreign Keys)

| Table | Colonne | Référence |
|---|---|---|
| `client` | `permission_id` | `permission(id)` |
| `client_account` | `client_id` | `client(id)` |
| `client_history` | `client_id` | `client(id)` |
| `manager` | `permission_id` | `permission(id)` |
| `manager` | `store_id` | `store(id)` |
| `preparer` | `permission_id` | `permission(id)` |
| `preparer` | `store_id` | `store(id)` |
| `schedule` | `preparer_id` | `preparer(id)` |
| `schedule` | `store_id` | `store(id)` |
| `performance` | `preparer_id` | `preparer(id)` |
| `cart` | `client_id` | `client(id)` |
| `cart` | `store_id` | `store(id)` |
| `cart_item` | `cart_id` | `cart(id)` |
| `cart_item` | `product_id` | `product(id)` |
| `order` | `client_id` | `client(id)` |
| `order` | `store_id` | `store(id)` |
| `order` | `pickup_slot_id` | `pickup_slot(id)` |
| `order` | `preparer_id` | `preparer(id)` |
| `order_item` | `order_id` | `order(id)` |
| `order_item` | `product_id` | `product(id)` |
| `payment` | `order_id` | `order(id)` |
| `notification` | `client_id` | `client(id)` |
| `notification` | `order_id` | `order(id)` |
| `pickup_slot` | `store_id` | `store(id)` |
| `product` | `category_id` | `category(id)` |
| `stock` | `store_id` | `store(id)` |
| `stock` | `product_id` | `product(id)` |
| `substitution_proposal` | `order_item_id` | `order_item(id)` |
| `substitution_proposal` | `original_product_id` | `product(id)` |
| `substitution_proposal` | `proposed_product_id` | `product(id)` |

---

### Gestion des rôles et des permissions

La table `permission` centralise les rôles applicatifs. Chaque acteur (`client`, `preparer`, `manager`) y est rattaché via une clé étrangère `permission_id`.

| Rôle | Périmètre d'accès |
|---|---|
| **Client** | Interface Click & Collect / Drive complète : navigation catalogue, panier, commandes, créneaux de retrait, historique de fidélité |
| **Préparateur** | Lecture des commandes assignées, de ses horaires et des stocks en cours. Peut uniquement mettre à jour le statut d'une commande (prête, livrée, annulée) |
| **Manager** | Vision globale sur les préparateurs et les clients. Peut modifier les stocks, gérer les créneaux et consulter les données de performance |
| **Admin** | Contrôle total sur le système et la base de données : droits en lecture/écriture sur l'ensemble des tables, gestion des utilisateurs et des configurations |

---

## Synchronisation avec le back-end

La connexion entre la base de données Supabase et le back-end Next.js de Dorian a été établie via les variables d'environnement suivantes :

| Variable | Usage |
|---|---|
| `NEXT_PUBLIC_SUPABASE_URL` | URL publique de l'instance Supabase |
| `NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY` | Clé publique pour les appels côté front-end |
| `DATABASE_URL` | Chaîne de connexion directe PostgreSQL pour Prisma |
| `SUPABASE_ANON_PUBLIC` | Clé anonyme pour les requêtes non authentifiées |
| `SUPABASE_SERVICE_ROLE_SECRET` | Clé service avec droits admin (back-end uniquement, non exposée côté client) |

La coordination a impliqué plusieurs allers-retours : les noms de colonnes, les types de données et certaines relations ont été ajustés à plusieurs reprises pour correspondre aux attentes du back-end et faciliter la génération des types Prisma. Ces itérations sont inhérentes à tout travail d'intégration BDD / back-end et ont été gérées en communication directe avec Dorian.

---

## Stratégie de sauvegarde — PITR

### Qu'est-ce que le PITR ?

Le **Point-In-Time Recovery (PITR)** est une fonctionnalité de sauvegarde avancée permettant de restaurer une base de données dans l'état exact où elle se trouvait à **n'importe quel instant précis**, à la seconde près.

Exemple concret appliqué au projet : un bug en production efface par erreur toutes les commandes en cours à 14h37. Avec le PITR, on restaure la base telle qu'elle était à 14h36. Sans PITR, la restauration la plus récente est celle du backup de la nuit précédente — une journée entière de commandes est perdue.

### Fonctionnement technique

Le PITR repose sur deux mécanismes PostgreSQL combinés :

1. **Sauvegardes physiques (base backup)** — Supabase prend régulièrement un instantané complet de la base au niveau des fichiers disque (copie binaire du répertoire PostgreSQL, non au niveau SQL).
2. **Fichiers WAL (Write-Ahead Log)** — PostgreSQL enregistre chaque transaction dans un journal chronologique *avant* de l'appliquer à la base.

**Processus de restauration :**
1. Chargement de la dernière sauvegarde physique antérieure au point cible
2. Téléchargement des fichiers WAL générés entre cette sauvegarde et le point cible
3. Rejeu (*replay*) chronologique des transactions WAL
4. Reconstruction de la base dans l'état exact souhaité, à la seconde près

### Pertinence pour le projet Lidl Drive

Notre base gère des données métier critiques dont la perte aurait un impact direct sur l'activité : commandes clients, statuts de préparation, créneaux réservés, stocks, comptes clients.

| Scénario | Sans PITR | Avec PITR |
|---|---|---|
| `DELETE` sans `WHERE` en production | Perte d'une journée de commandes | Restauration à T-30 secondes |
| Bug de migration SQL (colonnes supprimées) | Rollback impossible précisément | Rollback au point précédant la migration |
| Corruption silencieuse des statuts commandes | Irréversible si non détectée avant le backup | Restauration au moment exact du bug |
| Intrusion avec modification des données | Perte selon l'heure du dernier backup | Restauration au moment identifié via `audit_log` |
| Désynchronisation stock / commande | État incohérent difficile à corriger | Retour à un état cohérent identifié |

### Disponibilité Supabase et décision projet

| Critère | Backup quotidien (sans PITR) | PITR 7 jours |
|---|---|---|
| Granularité | 1 point/jour | 1 point/seconde |
| Perte maximale de données | Jusqu'à 24h | Quelques secondes |
| Coût mensuel (plan Pro) | $25 | ~$130 |
| Recommandé pour la production | Non | Oui |

**Décision pour notre MVP :**  
Le PITR n'est pas activé en raison des contraintes budgétaires du projet académique. La stratégie de sauvegarde actuelle repose sur des **dumps CLI hebdomadaires** stockés sur Google Drive, complétés par des **tables `backup_*`** (snapshots manuels à des instants clés du projet).

En contexte de production réelle pour Lidl, le PITR 7 jours (~$130/mois) constituerait le minimum acceptable pour garantir la continuité de service. L'hébergement sur **AWS eu-central-1 (Frankfurt)** est recommandé : proximité géographique avec Grenoble (~900 km), hébergement intégralement dans l'Union Européenne, conformité RGPD simplifiée, aucun transfert de données hors de l'Espace Économique Européen.

Certifications de l'infrastructure AWS sous-jacente : ISO/IEC 27001, SOC 2 Type II, RGPD.

---

## Conformité RGPD

### Politique de rétention des données métier

| Catégorie | Tables concernées | Durée de conservation | Base légale |
|---|---|---|---|
| Données clients (identité, adresse) | `client`, `client_account` | 3 ans après la dernière activité | Recommandation CNIL |
| Historique commandes et facturation | `order`, `order_item`, `payment` | 10 ans | Art. L123-22 Code de commerce |
| Données bancaires | `payment` | Aucune — délégation à un prestataire tiers | PCI DSS |
| Logs de sécurité | `audit_log` | 6 mois à 1 an | Recommandation CNIL |
| Points de fidélité | `client_history` | Durée de la relation commerciale | Intérêt légitime |

### RGPD et sauvegardes — point d'attention

Il existe un conflit technique entre le **droit à l'effacement** (Article 17 RGPD) et le PITR. La suppression d'un client en base principale ne supprime pas ses données des fichiers WAL et des sauvegardes physiques, qui les conservent pendant toute la fenêtre de rétention.

Ce comportement est toléré à trois conditions :
1. La durée de rétention des backups est documentée et raisonnable (7 jours recommandés)
2. Ces sauvegardes sont sécurisées et ne servent qu'à la reprise sur incident
3. En cas de restauration PITR depuis un point dans le passé, toutes les demandes d'effacement reçues entre ce point et aujourd'hui doivent être ré-appliquées manuellement

**Recommandation retenue :** gérer le droit à l'effacement par **anonymisation des champs PII** (`client.email`, `client.last_name`, `client.phone`, `client.address`) plutôt que par suppression physique, afin de préserver l'intégrité référentielle tout en respectant le RGPD.

### Traçabilité via `audit_log`

La table `audit_log` (colonnes : `actor_id`, `actor_role`, `action`, `target_table`, `target_id`, `ip_address`, `created_at`) permet d'identifier l'heure exacte de tout incident : connexion suspecte, modification frauduleuse de statut, suppression non autorisée. Cette heure constitue le point de référence précis pour une restauration PITR ciblée.

---

## Bilan de la semaine

| Jour | Réalisations principales |
|---|---|
| Lundi 13 | Prise en main Supabase/Prisma, analyse des besoins, validation équipe |
| Mardi 14 | Finalisation schéma BDD, préparation connexion back-end |
| Mercredi 15 | Connexion BDD ↔ back-end, ajustements itératifs, recherches PITR |
| Jeudi 16 | Corrections finales, auto-incrémentation, ajout produits et images |
| Vendredi 17 | Oral de présentation *(à compléter)* |

---

## Sources & Références

- [Supabase — Database Backups](https://supabase.com/docs/guides/platform/backups)
- [Supabase — Point-In-Time Recovery](https://supabase.com/docs/guides/platform/manage-your-usage/point-in-time-recovery)
- [Supabase — Régions disponibles](https://supabase.com/docs/guides/platform/regions)
- [AWS — Région eu-central-1 Frankfurt](https://aws.amazon.com/about-aws/global-infrastructure/regions_az/)
- [RGPD — Article 17 (droit à l'effacement)](https://gdpr-info.eu/art-17-gdpr/)
- [Code de commerce — Article L123-22](https://www.legifrance.gouv.fr/codes/article_lc/LEGIARTI000006219312)
