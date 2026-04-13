# Différentes tables utilisées dans le projet

## Client

- id (PK)
- id_permission (FK) (Léo)
- nom
- prénom
- email
- mot de passe (hashé)
- teléphone
- adresse
- magasin préféré (FK)
- Géolocalisation (latitude, longitude)

## Client_Account

- id (PK)
- client_id (FK)
- is_verified (boolean)
- active/inactive (boolean)
- date de création
- date de dernière connexion
- update_at

## Client_conso
- id (PK)
- client_id (FK)
- id_fidélité (FK)
- historique_transactions (FK)
- status_fidélité (boolean)
- nb_points_fidélité

## Préparateur
- id (PK)
- id_performance (FK) (léo)
- id_planning (FK) (léo)
- id_permission (FK) (léo)
- id_magasin (FK)
- prénom
- premieres_lettres_nom
- email_pro
- mot de passe (hashé)
- teléphone_pro
- performance (ex: nombre de commandes préparées, temps moyen de préparation, etc.)

## Manager

- id (PK)
- id_permission (FK) (léo)
- id_planning (FK) (léo)
- id_magasin (FK)
- prénom
- premieres_lettres_nom
- email_pro
- mot de passe (hashé)
- teléphone_pro

---

## Permission

> Stocke uniquement le rôle de l'acteur.
> Les droits associés à chaque rôle sont gérés directement dans le backend.
> 4 niveaux hiérarchiques : CLIENT < OPERATOR < MANAGER < ADMIN.

- id (PK)
- role (enum : CLIENT / OPERATOR / MANAGER / ADMIN)
- created_at
- updated_at

---

## Performance

> Suivi des indicateurs de performance des Préparateurs uniquement.
> Les Managers supervisent mais ne sont pas eux-mêmes évalués sur ces critères.

- id (PK)
- id_preparateur (FK) — référence vers Préparateur
- nb_commandes_preparees (integer) — nombre total de commandes traitées sur la période
- temps_moyen_preparation (float) — temps moyen en minutes entre la prise en charge et le passage en statut PRETE
- taux_erreur (float) — pourcentage de commandes ayant généré une anomalie (mauvais produit, quantité incorrecte, commande non remise…)
- nb_ruptures_signalees (integer) — nombre de fois où le préparateur a signalé qu'un produit commandé était absent en rayon au moment de la préparation
- note_globale (float) — score synthétique calculé par le backend à partir des 4 indicateurs ci-dessus, consultable par le Manager pour avoir une vue rapide sans lire chaque chiffre
- date_debut_periode (date)
- date_fin_periode (date)
- created_at

---

## Planning

> Gestion des créneaux de travail des Préparateurs et Managers par magasin.

- id (PK)
- id_employe (FK) — référence vers Préparateur ou Manager
- type_employe (enum : PREPARATEUR / MANAGER)
- id_magasin (FK)
- date (date)
- heure_debut (time)
- heure_fin (time)
- statut (enum : PLANIFIE / CONFIRME / ABSENT / ANNULE)
- commentaire (varchar) — optionnel, ex : "remplacement congé"
- created_at
- updated_at

