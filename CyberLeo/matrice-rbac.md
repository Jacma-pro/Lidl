# Matrice RBAC — Webapp Drive Lidl

> Ce document trace les droits de chaque rôle pour le rapport RGPD (Art. 30) et la documentation cybersécurité.
> Les droits sont implémentés dans le backend — ce fichier en est la référence documentaire.
> 4 niveaux hiérarchiques : CLIENT < OPERATOR < MANAGER < ADMIN.

---

## Rôles et périmètre

| Rôle | Acteur | Périmètre de données |
|---|---|---|
| **CLIENT** | Utilisateur final | Ses propres données uniquement |
| **OPERATOR** | Préparateur | Commandes et stock de son magasin (storeId) |
| **MANAGER** | Manager de magasin | Opérateurs, planning et stock de son magasin (storeId) |
| **ADMIN** | Administrateur système | Tous les magasins, toutes les données |

---

## Matrice des droits par ressource

| Action | CLIENT | OPERATOR | MANAGER | ADMIN |
|---|:---:|:---:|:---:|:---:|
| **Profil** |||||
| Voir son propre profil | ✅ | ✅ | ✅ | ✅ |
| Modifier son propre profil (adresse, téléphone, email) | ✅ | ✅ | ✅ | ✅ |
| Voir le profil d'un autre utilisateur | ❌ | ❌ | ❌ | ✅ |
| Désactiver / supprimer un compte | ❌ | ❌ | ❌ | ✅ |
| **Commandes** |||||
| Passer une commande | ✅ | ❌ | ❌ | ✅ |
| Voir ses propres commandes | ✅ | ❌ | ❌ | ✅ |
| Annuler sa propre commande | ✅ | ❌ | ❌ | ✅ |
| Voir les commandes du magasin | ❌ | ✅ | ✅ | ✅ |
| Modifier le statut d'une commande | ❌ | ✅ | ✅ | ✅ |
| Signaler une rupture de stock | ❌ | ✅ | ✅ | ✅ |
| **Catalogue et produits** |||||
| Voir le catalogue | ✅ | ✅ | ✅ | ✅ |
| Modifier les produits / catégories / restrictions allergènes | ❌ | ❌ | ❌ | ✅ |
| **Stock** |||||
| Voir le stock de son magasin | ❌ | ✅ | ✅ | ✅ |
| Modifier le stock de son magasin | ❌ | ❌ | ✅ | ✅ |
| **Opérateurs et Planning** |||||
| Voir son propre planning | ❌ | ✅ | ❌ | ✅ |
| Voir les plannings des préparateurs de son magasin | ❌ | ❌ | ✅ | ✅ |
| Gérer le planning des opérateurs | ❌ | ❌ | ✅ | ✅ |
| Consulter la performance des opérateurs | ❌ | ❌ | ✅ | ✅ |
| **Administration** |||||
| Accéder à l'interface admin | ❌ | ❌ | ❌ | ✅ |
| Gérer les magasins | ❌ | ❌ | ❌ | ✅ |
| Gérer les permissions | ❌ | ❌ | ❌ | ✅ |
| Voir les logs d'audit | ❌ | ❌ | ❌ | ✅ |
| **Droits RGPD** |||||
| Demander l'effacement de son compte (Art. 17) | ✅ | ✅ | ✅ | ✅ |
| Exporter ses données (Art. 20) | ✅ | ✅ | ✅ | ✅ |
| Rectifier ses données (Art. 16) | ✅ | ✅ | ✅ | ✅ |

---

## Contrainte storeId

OPERATOR et MANAGER ne peuvent accéder qu'aux données de **leur propre magasin**.
La règle appliquée côté backend :

```
role == OPERATOR  →  user.storeId === resource.storeId
role == MANAGER   →  user.storeId === resource.storeId
```

Un OPERATOR ou MANAGER qui tente d'accéder à un autre magasin doit recevoir une erreur `403 Forbidden`.

---

## Données accessibles par rôle (vue RGPD)

| Donnée personnelle | CLIENT | OPERATOR | MANAGER | ADMIN |
|---|:---:|:---:|:---:|:---:|
| Nom / prénom client | Les siens | ❌ | ❌ | ✅ |
| Email client | Le sien | ❌ | ❌ | ✅ |
| Téléphone client | Le sien | ❌ | ❌ | ✅ |
| Adresse client | La sienne | ❌ | ❌ | ✅ |
| Géolocalisation | Non stockée — à la volée uniquement | — | — | — |
| Historique commandes | Les siennes | Son magasin | Son magasin | ✅ |
| Points fidélité | Les siens | ❌ | ❌ | ✅ |
| Données pro Préparateur | ❌ | Les siennes | Son magasin | ✅ |
| Performance Préparateur | ❌ | Les siennes | Son magasin | ✅ |
| Planning | ❌ | Le sien | Son magasin | ✅ |
| Logs d'audit | ❌ | ❌ | ❌ | ✅ |
