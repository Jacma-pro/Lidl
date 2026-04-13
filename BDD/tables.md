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
- id_performance (FK) (léo)
- id_planning (FK) (léo)
- id_magasin (FK)
- prénom
- premieres_lettres_nom
- email_pro
- mot de passe (hashé)
- teléphone_pro

## Magasin
- id (PK)
- id_stock (FK)
- nom
- mail
- tel
- adresse
- code_postal
- ville
- pays
- géolocalisation (latitude, longitude)
- horaires d'ouverture
- créneaux de retrait disponibles (ex: 9h-10h, 10h-11h, etc.)
- maximum de commandes par créneau horaire
- moyenne de temps de préparation d'une commande
- drive disponible (boolean)
- click&collect disponible (boolean)

