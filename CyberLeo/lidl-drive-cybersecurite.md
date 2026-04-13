# Cybersécurité — Tâches Trello | Webapp Drive Lidl

---

## Threat Model (STRIDE — Spoofing, Tampering, Repudiation, Information Disclosure, Denial of Service, Elevation of Privilege)

- Lister les 3 flux principaux à analyser : authentification / passage de commande / accès interface admin
- Produire un DFD (Data Flow Diagram) niveau 1 (acteurs, flux de données, périmètres de confiance) avant d'appliquer STRIDE
- Cartographier la surface d'attaque : lister les endpoints exposés par route (auth, orders, products, slots, admin)
- Appliquer STRIDE sur le flux authentification
- Appliquer STRIDE sur le flux passage de commande
- Appliquer STRIDE sur le flux accès interface admin
- Pour chaque menace identifiée, lister la contre-mesure retenue ou justifier l'acceptation du risque
- Évaluer chaque menace via une matrice 3×3 (probabilité × impact) et justifier le niveau retenu
- Rédiger le document `threat-model.md`

---

## Registre de traitement RGPD (Règlement Général sur la Protection des Données — Art. 30)

- Lister toutes les données personnelles collectées par l'application (email, mot de passe, historique commandes, créneau de retrait, logs, notifications)
- Associer une base légale à chaque traitement (contrat, intérêt légitime, consentement)
- Justifier le choix de la base légale pour chaque traitement
- Définir la durée de conservation pour chaque donnée
- Identifier qui a accès à chaque donnée (client, opérateur, admin, système)
- Identifier les transferts de données hors UE (Union Européenne) réels : service d'envoi d'email, CDN (Content Delivery Network) si les assets sont servis depuis des serveurs hors UE
- Évaluer si une DPIA (Data Protection Impact Assessment) est requise (remplir la grille de critères Art. 35)
- Conclure et justifier la décision sur la DPIA
- Rédiger le document `registre-traitement.md`

---

## Matrice RBAC (Role-Based Access Control)

- Lister les 3 rôles du système : CLIENT, OPERATOR, ADMIN
- Lister les actions CRUD (Create, Read, Update, Delete) par ressource : User, Order, Product, Slot, Store
- Remplir le tableau rôle × action pour chaque combinaison
- Ajouter la contrainte attributaire `storeId` pour le rôle OPERATOR
- Décrire la règle d'accès OPERATOR : `role == OPERATOR && user.storeId == resource.storeId`
- Identifier les vecteurs d'escalade de privilèges (IDOR — Insecure Direct Object Reference, payload JWT modifiable, endpoint admin non protégé)
- Vérifier la configuration JWT (JSON Web Token) : algorithme imposé (HS256 ou RS256), rejet de `alg: none`, durée d'expiration
- Documenter les mesures associées à chaque vecteur identifié
- Tester au moins 3 scénarios d'escalade (ex : CLIENT accédant à un endpoint ADMIN) et documenter le résultat attendu vs observé
- Rédiger le document `matrice-rbac.md`

---

## Droit à l'effacement (Art. 17 RGPD — Règlement Général sur la Protection des Données)

- Cartographier tous les endroits où une donnée personnelle existe : table `User`, table `Order`, table `AuditLog`, sessions Redis, logs Nginx, logs applicatifs, sauvegardes PostgreSQL
- Distinguer ce qui doit être effacé (email, mot de passe) de ce qui doit être pseudonymisé (historique commandes)
- Justifier la conservation pseudonymisée des commandes avec la base légale (Art. L123-22 Code de commerce — 10 ans)
- Définir la procédure de pseudonymisation : remplacer les champs identifiants par un token anonyme
- Définir la procédure de purge des sessions Redis au moment de la suppression
- Définir la politique de traitement des sauvegardes PostgreSQL : délai de rotation, impossibilité de restaurer une donnée supprimée après X jours — ou justifier l'acceptation du risque résiduel
- Rédiger la procédure interne d'effacement étape par étape
- Rédiger le template de réponse à l'utilisateur sous 30 jours (Art. 12 RGPD)
- Définir l'entrée AuditLog pour tracer chaque demande d'effacement traitée
- Spécifier l'endpoint `DELETE /user/me` (données supprimées, données pseudonymisées, sessions révoquées)
- Spécifier l'endpoint `GET /user/export` pour le droit à la portabilité (Art. 20) — format JSON (JavaScript Object Notation)
- Définir les champs inclus dans l'export (données compte, historique commandes)
- Rédiger le document `politique-effacement.md`


# Droit de visualisation des données par rôles
Manager -> Donnée preparateur + Clients
Préparateur -> Clients
Clients -> Préparateurs
cyber centré