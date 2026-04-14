# Tables de la base de données — Lidl Drive

> Document de référence pour la modélisation de la base de données du projet.
> Toutes les tables sont listées par domaine fonctionnel.
> Les corrections et ajouts par rapport à la version précédente sont intégrés.

---

## Sommaire

- [1. Domaine Utilisateurs](#1-domaine-utilisateurs)
  - [Permission](#permission)
  - [Client](#client)
  - [Client_Account](#client_account)
  - [Client_Conso](#client_conso)
  - [Préparateur](#préparateur)
  - [Manager](#manager)
- [2. Domaine Magasin & Catalogue](#2-domaine-magasin--catalogue)
  - [Magasin](#magasin)
  - [Category](#category)
  - [Produit](#produit)
  - [Stock](#stock)
- [3. Domaine Commande & Panier](#3-domaine-commande--panier)
  - [PickupSlot](#pickupslot)
  - [Cart](#cart)
  - [CartItem](#cartitem)
  - [Order](#order)
  - [OrderItem](#orderitem)
  - [SubstitutionProposal](#substitutionproposal)
  - [Payment](#payment)
- [4. Domaine RH & Opérationnel](#4-domaine-rh--opérationnel)
  - [Planning](#planning)
  - [Performance](#performance)
- [5. Domaine Système](#5-domaine-système)
  - [Notification](#notification)
  - [AuditLog](#auditlog)

---

## 1. Domaine Utilisateurs

---

### Permission

> Stocke uniquement le rôle d'un acteur du système.
> Les droits associés à chaque rôle sont gérés directement dans le backend, pas en base.
> 4 niveaux hiérarchiques : CLIENT < OPERATOR < MANAGER < ADMIN.

- id (PK)
- role (enum : CLIENT / OPERATOR / MANAGER / ADMIN)
- created_at — date et heure à laquelle le rôle a été attribué à l'acteur
- updated_at — date et heure de la dernière modification du rôle (ex : passage OPERATOR → MANAGER)

---

### Client

> Informations personnelles et de contact du client. Représente l'identité civile.
> Séparée de Client_Account pour distinguer les données personnelles des données de connexion.
> La géolocalisation n'est PAS stockée — utilisée à la volée côté frontend pour détecter
> le magasin le plus proche, puis jetée. Décision RGPD : pas de consentement géoloc requis.

- id (PK)
- permission_id (FK → Permission)
- nom
- prénom
- email
- mot_de_passe (hashé)
- telephone
- adresse
- preferred_store_id (FK → Magasin, nullable) — magasin préféré du client (défini après sélection manuelle)
- created_at
- updated_at

---

### Client_Account

> Données de gestion du compte client : statut, vérification, activité.
> Séparée de Client pour pouvoir désactiver un compte sans supprimer les données personnelles.

- id (PK)
- client_id (FK → Client)
- is_verified (boolean) — email vérifié ou non
- is_active (boolean) — compte actif ou suspendu
- created_at
- last_login_at
- updated_at

---

### Client_Conso

> Suivi de la fidélité et des points du client.
> Table du MVP avancé — non bloquante pour le MVP minimum.

- id (PK)
- client_id (FK → Client)
- nb_points_fidelite (integer)
- status_fidelite (boolean) — programme fidélité actif ou non
- created_at
- updated_at

---

### Préparateur

> Employé de magasin chargé de préparer les commandes clients.
> Rattaché à un seul magasin. Ses performances sont suivies dans la table Performance.

- id (PK)
- permission_id (FK → Permission)
- store_id (FK → Magasin)
- prénom
- premieres_lettres_nom — anonymisation partielle du nom de famille
- email_pro
- mot_de_passe (hashé)
- telephone_pro
- created_at
- updated_at

---

### Manager

> Responsable de magasin. Supervise les préparateurs, gère les plannings et consulte les performances.
> N'est pas lui-même évalué sur des indicateurs de performance.

- id (PK)
- permission_id (FK → Permission)
- store_id (FK → Magasin)
- prénom
- premieres_lettres_nom
- email_pro
- mot_de_passe (hashé)
- telephone_pro
- created_at
- updated_at

---

## 2. Domaine Magasin & Catalogue

---

### Magasin

> Représente un point de vente physique Lidl.
> Contient les informations de contact, de localisation et la configuration du service drive.
> Le stock et les créneaux de retrait sont gérés dans des tables dédiées (Stock, PickupSlot).

- id (PK)
- nom
- email
- telephone
- adresse
- code_postal
- ville
- pays
- latitude (decimal)
- longitude (decimal)
- horaires_ouverture (json ou varchar) — ex : "Lun-Sam 8h-21h"
- slot_duration_minutes (integer) — durée d'un créneau de retrait en minutes
- max_orders_per_slot (integer) — capacité maximale par créneau
- avg_preparation_time_minutes (integer) — temps moyen de préparation d'une commande
- drive_available (boolean)
- click_collect_available (boolean)
- created_at
- updated_at

---

### Category

> Catégories de produits (ex : Fruits & Légumes, Produits laitiers, Surgelés).
> Permet d'organiser le catalogue et de proposer des filtres côté client.

- id (PK)
- nom
- restrictions (json, nullable) — ex : {"gluten": true, "arachides": false}
- description (nullable)

---

### Produit

> Référence produit du catalogue global.
> Un produit peut exister au catalogue sans être disponible dans tous les magasins — c'est la table Stock qui gère la disponibilité locale.

- id (PK)
- category_id (FK → Category)
- nom
- description (nullable)
- prix (decimal)
- poids (decimal, nullable)
- longueur (decimal, nullable)
- largeur (decimal, nullable)
- hauteur (decimal, nullable)
- image_url (nullable)
- barcode (varchar, nullable) — code-barres EAN
- nutriscore (varchar, nullable) — A, B, C, D ou E
- is_active (boolean) — produit visible ou archivé
- created_at
- updated_at

---

### Stock

> Quantité disponible d'un produit dans un magasin précis.
> C'est la relation entre le catalogue global et l'inventaire local de chaque magasin.
> Un produit peut être en stock dans un magasin et en rupture dans un autre.

- id (PK)
- store_id (FK → Magasin)
- product_id (FK → Produit)
- quantite_disponible (integer)
- updated_at

---

## 3. Domaine Commande & Panier

---

### PickupSlot

> Créneau de retrait proposé dans un magasin à une date et heure données.
> Permet de limiter le nombre de commandes simultanées et d'organiser la préparation.
> Un créneau est marqué indisponible quand sa capacité max est atteinte.

- id (PK)
- store_id (FK → Magasin)
- date (date)
- heure_debut (time)
- heure_fin (time)
- max_orders (integer) — capacité maximale du créneau
- current_orders (integer) — nombre de commandes déjà réservées
- is_available (boolean)

---

### Cart

> Panier en cours d'un client, rattaché à un magasin précis.
> Persistant : le panier est conservé même si le client ferme son navigateur.
> Quand la commande est validée, le panier passe en statut CONVERTI.

- id (PK)
- client_id (FK → Client)
- store_id (FK → Magasin)
- status (enum : ACTIF / ABANDONNE / CONVERTI)
- created_at
- updated_at

---

### CartItem

> Ligne de produit dans un panier.
> Le prix unitaire est stocké au moment de l'ajout pour éviter qu'une modification
> de prix en cours de route n'affecte silencieusement le panier du client.

- id (PK)
- cart_id (FK → Cart)
- product_id (FK → Produit)
- quantite (integer)
- prix_unitaire (decimal) — prix au moment de l'ajout au panier

---

### Order

> Commande validée par le client.
> Créée à partir d'un Cart confirmé. Porte le statut de préparation et le code de retrait.
> C'est l'entité centrale que le préparateur consulte dans l'interface admin.

- id (PK)
- client_id (FK → Client)
- store_id (FK → Magasin)
- pickup_slot_id (FK → PickupSlot)
- preparateur_id (FK → Préparateur, nullable) — assigné lors de la prise en charge
- status (enum : EN_ATTENTE / EN_PREPARATION / PRETE / RETIREE / ANNULEE)
- prix_total (decimal)
- pickup_code (varchar) — code court ou QR code remis au client pour le retrait
- created_at
- updated_at

---

### OrderItem

> Ligne de produit dans une commande validée.
> Distincte de CartItem : une fois la commande créée, les quantités et prix sont figés
> et ne doivent plus évoluer même si le catalogue change.

- id (PK)
- order_id (FK → Order)
- product_id (FK → Produit)
- quantite (integer)
- prix_unitaire (decimal) — prix au moment de la validation de commande
- substitution_id (FK → SubstitutionProposal, nullable)

---

### SubstitutionProposal

> Proposition de remplacement d'un produit en rupture lors de la préparation.
> Le préparateur signale qu'un produit est absent et propose une alternative.
> Le client peut accepter ou refuser via une notification.

- id (PK)
- order_item_id (FK → OrderItem)
- original_product_id (FK → Produit) — produit commandé en rupture
- proposed_product_id (FK → Produit) — produit de remplacement proposé
- status (enum : EN_ATTENTE / ACCEPTEE / REFUSEE)
- created_at

---

### Payment

> IMPORTANT ! : Ne pas intégrer dans la webapp du MVP, mais juste avoir la table prête pour la partie théorique du projet.
> Donc pour l'ajout du système de paiement, on peut se contenter d'une simulation ou d'une simple validation au retrait.

> Suivi du règlement d'une commande.
> Dans le MVP, le paiement peut être simulé ou prévu au retrait.
> La table reste utile pour tracer le statut et permettre un remboursement même basique.

- id (PK)
- order_id (FK → Order)
- montant (decimal)
- methode (enum : EN_MAGASIN / SIMULE_EN_LIGNE)
- status (enum : EN_ATTENTE / VALIDE / REMBOURSE)
- transaction_ref (varchar, nullable) — référence externe si paiement en ligne simulé
- created_at

---

## 4. Domaine RH & Opérationnel

---

### Planning

> Suivi de la présence des Préparateurs sur site, par magasin et par créneau horaire.
> Permet au Manager de savoir quels préparateurs sont disponibles à un moment donné,
> afin d'estimer la capacité de préparation et d'organiser les créneaux de retrait (PickupSlot).
> Le Manager consulte en lecture seule — il ne saisit pas son propre planning dans cette table.

- id (PK)
- preparateur_id (FK → Préparateur)
- store_id (FK → Magasin)
- date (date)
- heure_debut (time)
- heure_fin (time)
- statut (enum : PRESENT / ABSENT / CONGE)
- commentaire (varchar, nullable) — ex : "remplacement congé"
- created_at
- updated_at

---

### Performance

> Suivi des indicateurs de performance des Préparateurs uniquement.
> Les Managers supervisent mais ne sont pas eux-mêmes évalués sur ces critères.
> Les entrées sont générées par période (semaine, mois) par le backend.
> La note_globale est un score synthétique calculé automatiquement à partir des 4 indicateurs —
> permet au Manager d'avoir une vue rapide sans lire chaque chiffre individuellement.

- id (PK)
- preparateur_id (FK → Préparateur)
- nb_commandes_preparees (integer) — nombre total de commandes traitées sur la période
- temps_moyen_preparation (float) — temps moyen en minutes entre prise en charge et passage au statut PRETE
- taux_erreur (float) — part des commandes ayant généré un problème : mauvais produit, quantité incorrecte, commande non remise au bon client, etc. Exprimé en pourcentage.
- nb_ruptures_signalees (integer) — nombre de fois où le préparateur a constaté qu'un produit commandé était absent en rayon au moment de la préparation
- note_globale (float) — score synthétique calculé par le backend à partir des 4 indicateurs ci-dessus (nb_commandes, temps_moyen, taux_erreur, nb_ruptures). Consultable directement par le Manager.
- date_debut_periode (date)
- date_fin_periode (date)
- created_at

---

## 5. Domaine Système

---

### Notification

> Historique de tous les messages envoyés aux clients.
> Permet d'éviter les doublons, de tracer les échecs d'envoi et d'avoir une preuve
> en cas de litige ("je n'ai pas reçu de confirmation").

- id (PK)
- client_id (FK → Client)
- order_id (FK → Order, nullable)
- type (enum : CONFIRMATION / PRETE / ANNULATION / SUBSTITUTION)
- channel (enum : EMAIL / SMS / PUSH)
- status (enum : EN_ATTENTE / ENVOYEE / ECHOUEE)
- sent_at (datetime, nullable)
- created_at

---

### AuditLog

> Journal des actions sensibles réalisées sur le système.
> Indispensable pour la traçabilité et la détection d'anomalies (accès non autorisé,
> modification frauduleuse d'un statut de commande, etc.).
> Table principalement exploitée par l'équipe cybersécurité.

- id (PK)
- actor_id (integer) — id de l'utilisateur ayant effectué l'action
- actor_role (enum : CLIENT / OPERATOR / MANAGER / ADMIN)
- action (varchar) — ex : ORDER_CREATED, STATUS_UPDATED, LOGIN_FAILED
- target_table (varchar) — table concernée par l'action
- target_id (integer, nullable) — id de l'enregistrement concerné
- ip_address (varchar)
- created_at

---

## Relations clés

| Table | Liée à | Type |
|---|---|---|
| Client | Permission | N-1 |
| Client | Client_Account | 1-1 |
| Client | Client_Conso | 1-1 |
| Préparateur | Magasin | N-1 |
| Manager | Magasin | N-1 |
| Produit | Category | N-1 |
| Stock | Magasin + Produit | N-N (table de jonction) |
| PickupSlot | Magasin | N-1 |
| Cart | Client + Magasin | N-1 |
| CartItem | Cart + Produit | N-1 |
| Order | Client + Magasin + PickupSlot | N-1 |
| OrderItem | Order + Produit | N-1 |
| SubstitutionProposal | OrderItem + Produit (x2) | N-1 |
| Payment | Order | 1-1 |
| Notification | Client + Order | N-1 |
| AuditLog | — (log libre) | — |
| Planning | Préparateur ou Manager + Magasin | N-1 |
| Performance | Préparateur | N-1 |