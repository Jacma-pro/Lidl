# Webapp Drive Lidl — Spécification Technique

> **Document de référence pour l'équipe dev**
> Version simplifiée et scoped pour le MVP réaliste.

---

## 1. Vue d'ensemble

Nous développons une **webapp Click & Collect pour Lidl**.

**Parcours client simplifié :**
1. Authentification
2. Choix du magasin
3. Consultation du catalogue + ajout au panier
4. Choix d'un créneau de retrait (limité à 20 commandes/créneau)
5. Validation de la commande
6. Retrait sur place

**Parcours préparateur/admin :**
1. Authentification (rôle = `preparateur` ou `admin`)
2. Vue des commandes à préparer
3. Changement de statut (validation → en préparation → prête)
4. Gestion du stock par magasin (interface admin simple)

---

## 2. Architecture globale

```text
Internet
    |
    v
[ Reverse Proxy Nginx ]
    |
    +---> [ Front Client (React/Vue) — port 3000 ]
    |
    +---> [ API Backend (Node.js/Express) — port 5000 ]
               |
               +---> [ PostgreSQL — port 5432 ]
```

**Chaque service en Docker séparé** → `docker-compose.yml` qui les orchestrate.

### Technologie choisie

| Composant      | Technologie        | Justification                          |
|----------------|--------------------|----------------------------------------|
| Frontend       | React + Vite       | Rapide, modernes, facile à tester      |
| Backend        | Node.js + Express  | JavaScript full-stack, JWT natif       |
| Base de données| PostgreSQL         | Robuste, JSON support, fiable          |
| Conteneurs     | Docker Compose     | Local dev + démo facile                |
| Proxy          | Nginx              | Reverse proxy simple, production-ready |
| Auth           | JWT + refresh token| Stateless, rôles intégrés              |

---

## 3. Entités métier

### Schéma simplifié

```
User
  ├─ id (PK)
  ├─ email (UNIQUE)
  ├─ password_hash
  ├─ role (client | preparateur | admin)
  └─ store_id (FK) [nullable — client peut changer]

Store
  ├─ id (PK)
  ├─ name
  └─ address

Product
  ├─ id (PK)
  ├─ name
  ├─ price
  ├─ category
  └─ available (boolean)

Inventory
  ├─ id (PK)
  ├─ store_id (FK)
  ├─ product_id (FK)
  └─ quantity (int)

Cart
  ├─ id (PK)
  ├─ user_id (FK)
  └─ store_id (FK)

CartItem
  ├─ id (PK)
  ├─ cart_id (FK)
  ├─ product_id (FK)
  └─ quantity

Order
  ├─ id (PK)
  ├─ user_id (FK)
  ├─ store_id (FK)
  ├─ status (validée | en_préparation | prête | retirée)
  ├─ time_slot (ex: "09:00-10:00")
  ├─ created_at
  └─ updated_at

OrderItem
  ├─ id (PK)
  ├─ order_id (FK)
  ├─ product_id (FK)
  └─ quantity

PickupSlot
  ├─ id (PK)
  ├─ store_id (FK)
  ├─ time_slot (ex: "09:00-10:00")
  ├─ capacity (int = 20)
  └─ date
```

**Clés métier :**
- Un **User** a un rôle : `client`, `preparateur`, ou `admin`.
- Un **Inventory** lie `store_id` + `product_id` pour gérer le stock localement.
- Un **Order** a un `time_slot` et un `status` pour suivre la préparation.
- Un **PickupSlot** définit les créneaux disponibles (9h-10h, 10h-11h, etc.) avec une capacité max de 20 commandes.

---

## 4. Flux métier — côté client

```
1. Login / Register
   └─> Crée un User avec role = 'client'

2. Choix du magasin
   └─> Sauvegarde store_id en session/JWT

3. Accès au catalogue
   └─> GET /api/products?store_id=X
       (retourne les produits disponibles pour ce magasin)

4. Ajout au panier
   └─> POST /api/cart/items
       Ajoute product_id + quantity à Cart

5. Validation du panier
   └─> GET /api/pickup-slots?store_id=X&date=2025-04-15
       (montre les créneaux disponibles avec nombre de places restantes)
   └─> Client choisit un créneau

6. Création de la commande
   └─> POST /api/orders
       - Crée un Order avec status = 'validée'
       - Décrémente Inventory(store_id, product_id)
       - Assigne le time_slot
       - Incrémente le PickupSlot.reserved_count

7. Confirmation
   └─> Client voit son numéro de commande
   └─> Peut suivre le statut via GET /api/orders/{id}

8. Suivi du statut
   └─> Client regarde périodiquement GET /api/orders/{id}
       Voit : validée → en_préparation → prête → retirée
```

---

## 5. Flux métier — côté préparateur/admin

### 5.1 Préparateur (role = `preparateur`)

```
1. Login avec role = 'preparateur'

2. Dashboard des commandes
   └─> GET /api/orders?status=validée&store_id=X
       Voit les commandes à préparer

3. Voir le détail d'une commande (picking list)
   └─> GET /api/orders/{id}
       Affiche la liste des produits à aller chercher
       Exemple : "Lait x2, Fromage x1, Œufs x3"

4. Marquer la commande "en préparation"
   └─> PATCH /api/orders/{id}
       { "status": "en_préparation" }

5. Une fois préparée, valider "prête"
   └─> PATCH /api/orders/{id}
       { "status": "prête" }
       → Client reçoit une notification (bonus : voir section 7)

6. Voir les commandes prêtes
   └─> GET /api/orders?status=prête&store_id=X
       Pour suivre le retrait
```

### 5.2 Admin (role = `admin`)

Même accès que le préparateur, PLUS :

```
7. Gérer le stock
   └─> GET /api/inventory?store_id=X
       Voit les stocks par produit

   └─> PATCH /api/inventory/{id}
       { "quantity_adjustment": +30 }
       (Admin rentre simplement le nombre à ajouter/retirer)

8. Voir tous les magasins (pas juste le sien)
   └─> Interface admin dédiée : /admin
```

---

## 6. API — Endpoints clés

### Auth
```
POST /api/auth/register
  Body: { email, password, store_id }
  Return: { user_id, role, token, refresh_token }

POST /api/auth/login
  Body: { email, password }
  Return: { user_id, role, store_id, token, refresh_token }

POST /api/auth/refresh
  Body: { refresh_token }
  Return: { token }
```

### Products & Inventory
```
GET /api/products?store_id=X
  Return: [ { id, name, price, category, available } ]

GET /api/inventory?store_id=X
  (Admin only)
  Return: [ { product_id, quantity } ]

PATCH /api/inventory/{id}
  (Admin only)
  Body: { quantity_adjustment: +30 }
```

### Cart
```
GET /api/cart
  Return: { cart_id, items: [ { product_id, quantity, price } ], total }

POST /api/cart/items
  Body: { product_id, quantity }
  Return: { cart updated }

DELETE /api/cart/items/{product_id}
  Return: { cart updated }
```

### Orders
```
GET /api/orders?status=validée&store_id=X
  (Préparateur/Admin)
  Return: [ { order_id, user_id, items, created_at, status } ]

GET /api/orders/{id}
  Return: { order_id, user_id, items, status, time_slot, store_id, created_at }

POST /api/orders
  Body: { cart_id, time_slot }
  Return: { order_id, status, confirmation_number }

PATCH /api/orders/{id}
  Body: { status: "en_préparation" | "prête" | "retirée" }
  (Préparateur/Admin)
```

### Pickup Slots
```
GET /api/pickup-slots?store_id=X&date=YYYY-MM-DD
  Return: [
    { time_slot: "09:00-10:00", capacity: 20, reserved: 5, available: true },
    { time_slot: "10:00-11:00", capacity: 20, reserved: 20, available: false },
    ...
  ]
```

---

## 7. Fonctionnalités par priorité

### MVP Minimum (MUST HAVE)
- [x] Authentification (JWT)
- [x] Rôles : client / preparateur / admin
- [x] Catalogue de produits
- [x] Panier
- [x] Gestion du stock par magasin
- [x] Création de commande avec créneau
- [x] Dashboard préparateur (lister, changer statut)
- [x] Dashboard admin (gérer le stock)

### MVP Intermédiaire (SHOULD HAVE)
- [ ] Interface admin séparée (/admin)
- [ ] Validation des créneaux (max 20 par créneau)
- [ ] Notifications email de confirmation (bonus)
- [ ] QR code de retrait (bonus)

### MVP Avancé (NICE TO HAVE)
- [ ] Points Lidl Plus au retrait
- [ ] Coupons
- [ ] Analytics
- [ ] Recommandations produits
