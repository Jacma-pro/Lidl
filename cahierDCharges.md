# Lidl Drive — README de démarrage

> À lire avant d'écrire du code.

---

## Le projet

Une **PWA** (Progressive Web App) Drive / Click & Collect. L'utilisateur choisit un magasin, remplit un panier, réserve un créneau de retrait et suit sa commande. Un opérateur en magasin prépare la commande avant le retrait.

La PWA signifie que l'application doit être installable sur mobile et fonctionner avec un Service Worker. Cela se configure côté React avec Vite (plugin `vite-plugin-pwa`). -> gérer par Alexandre

---

## Stack

| Couche           | Choix                   |
| ---------------- | ----------------------- |
| Frontend         | React + Vite (PWA) + TS |
| Backend          | NestJS                  |
| Base de données  | PostgreSQL via Supabase |
| Cache / sessions | Redis                   |
| Auth             | JWT + Refresh Token     |
| Conteneurisation | Docker + Docker Compose |
| Reverse Proxy    | Nginx ou Traefik        |
| CI/CD            | GitHub Actions          |

---

## Pole Cybersecurity et Conformité

### Leo : Conformité et Sécurité

Leo produit 3 livrables :

Le registre RGPD : recenser tous les traitements de données (clients et employés), identifier les bases légales, et définir techniquement comment implémenter le droit à l'effacement dans PostgreSQL et Supabase.

La matrice RBAC : définir précisément les droits de chaque rôle (`CLIENT`, `OPERATEUR`, `ADMIN`) sur chaque ressource de l'API.

La modélisation des menaces : identifier et prioriser les vecteurs d'attaque sur le périmètre du projet (authentification, gestion des commandes, interface admin). Ce livrable se limite à l'identification et la priorisation des risques (ex : brute force, vol de token, élévation de privilèges) — Leo ne définit pas les contre-mesures techniques. Les risques identifiés sont transmis à Willy (qui en déduira la politique JWT et le besoin éventuel de 2FA) et à Patrice (qui en tiendra compte pour la segmentation réseau).

### Willy : Authentification et Sécurité des Flux

Willy produit 2 livrables :

Willy définit la politique d'authentification JWT : durée de vie des access tokens et refresh tokens, mécanisme de révocation, règles de renouvellement. Il s'appuie sur les risques identifiés par Leo (modélisation des menaces) pour justifier ses choix de politique.

Il étudie également la mise en place d'une double authentification (2FA) pour les comptes `OPERATEUR` et `ADMIN`, et cherche comment il serait possible d'intégrer un moyen de paiment.

### Patrice : Infrastructure Réseau

Patrice produit deux schémas réseau. 
Un schéma directeur à l'échelle départementale montrant comment le projet s'intègrerait dans l'infrastructure Lidl existante. Une topologie détaillée pour le site pilote de Saint-Martin-d'Hères, avec la segmentation exacte entre zone publique (reverse proxy), zone applicative (fronts + API) et zone privée (PostgreSQL + Redis). Ces schémas doivent être cohérents avec les choix Docker de David.
- Schéma réseau à échelle départementale 
- Schéma réseau détaillé pour le site pilote de Saint-Martin-d'Hères

---

## Pôle Design et Expérience Utilisateur

### Gwendoline : UX/UI

Gwen livre les maquettes des deux interfaces : le parcours client complet (sélection magasin, panier, créneau...) et l'interface admin (dashboard commandes, changement de statut, gestion des ruptures). Elle rédige le dossier de conception (avec Dorian), qui justifie les choix retenus. Elle travaille en coordination avec Leo pour que les interfaces respectent les exigences RGPD.

---

## Pôle Infrastructure DevOps

### David : Déploiement et Automatisation

David s'occupe de la CI/CD et livre un `docker-compose.yml` fonctionnel avec six services isolés par réseau Docker : `reverse-proxy`, `frontend-client`, `frontend-admin`, `api`, `postgres`, `redis`. Il configure le reverse proxy (Nginx ou Traefik) avec terminaison TLS et headers de sécurité. Il met en place le pipeline CI/CD sur GitHub Actions (lint, build, construction des images). Il crée et gère le `.env` et s'assure qu'aucun secret n'est commité.

---

## Pôle Données et Modélisation

### Sabry : Architecture Base de Données

Sabry modélise et implémente l'intégralité de la base de données Supabase. Il produit les trois niveaux de schéma (MCD, MLV, MPD). Il crée les index nécessaires pour les recherches produits par magasin. Il communique ses choix avec Dorian et Leo et Alexandre.

---

## Pôle Développement

### Dorian : Backend

Dorian produit 2 livrables :

Dorian gère le backend de l'API NestJS

Il attend deux livrables avant de commencer : la matrice RBAC de Leo et la politique JWT de Willy. Les endpoints à implémenter dans l'ordre : authentification (`/auth/*`), catalogue et stocks (`/stores`, `/stores/:id/inventory`), panier (`/cart`), créneaux (`/slots`), commandes (`/orders`). Il coordonne avec Sabry pour que les requêtes correspondent au modèle de données réel. Il rédige le dossier technique de soutenance documentant les choix avec Gwendoline.

### Alexandre : Frontend et Intégration PWA

Alexandre intègre les deux interfaces (client et admin) React + Vite à partir des maquettes de Gwen. Il configure la PWA via `vite-plugin-pwa` (Service Worker, manifeste, installation mobile). Il implémente les tests du tunnel d'achat (Playwright) pour garantir que le parcours complet fonctionne sans erreur. Il sera aussi en charge de validé l'ergonomie et l'utilisabilité des deux types d'interfaces (user et admin).

---

## Règles communes

Tout passe par `.env` et est géré par le `gitignore` donc attention à ne rien mettre de privé en public

Toute modification du modèle de données doit être signalée aux trois équipes pour éviter de casser le code

Pull Requests obligatoires, pas de push direct sur `main`. -> ceux qui connaissent pas peuvent voir avec Dorian et Alexandre

Le projet se lance avec `npm run build && npm run preview`