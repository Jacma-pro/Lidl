<link rel="stylesheet" href="../_style.css">

# Notes oral – Présentation Backend Lidl Collect
**Durée totale : 5 min · ~1 min 15 par slide**

---

## Slide 1 — Le backend : le moteur invisible

> *Pointer le schéma de gauche à droite pendant la présentation.*

- Commencer par situer son rôle dans le groupe : *"Ma partie, c'est le backend — ce que l'utilisateur ne voit pas, mais sans quoi rien ne fonctionne."*
- Expliquer le schéma simplement : *"Quand un client clique sur 'Commander', le site envoie une demande à l'API. L'API vérifie, applique les règles, interroge la base de données, et renvoie une réponse. Tout ça en quelques millisecondes."*
- Insister sur le rôle central de l'API : *"L'API, c'est le chef d'orchestre — elle reçoit toutes les demandes, décide quoi faire, et garantit que les bonnes données arrivent aux bonnes personnes."*

---

## Slide 2 — 3 profils, 3 accès distincts

> *Pointer chaque colonne en la nommant.*

- Introduire la logique des rôles sans jargon : *"L'application s'adresse à trois types d'utilisateurs — et c'est le backend qui s'assure que chacun ne voit que ce qui le concerne."*
- Dérouler rapidement les trois profils :
  - *"Le client : il navigue, commande, suit son retrait."*
  - *"Le préparateur : il reçoit les commandes à préparer en magasin, signale si un produit manque."*
  - *"Le gérant : il a une vue d'ensemble — les préparateurs, les commandes du jour, les statistiques."*
- Conclure sur la sécurité : *"Un client ne peut pas accéder à l'interface du gérant. Un préparateur ne voit pas les données d'un autre magasin. C'est le backend qui applique ces règles à chaque requête."*

---

## Slide 3 — Cycle de vie d'une commande

> *Suivre les états de gauche à droite avec le doigt ou un pointeur.*

- Entrer dans le concret : *"Voilà ce qui se passe concrètement quand une commande est passée."*
- Dérouler les états : *"Elle démarre en attente, un préparateur la prend en charge — elle passe en préparation — une fois prête, le client est notifié — et à la récupération, elle est marquée comme retirée."*
- Mentionner l'annulation : *"À tout moment, la commande peut être annulée — par le client ou par le système si un problème survient."*
- Ajouter la substitution : *"Et si un produit est en rupture pendant la préparation, le système propose automatiquement un article de remplacement que le client peut accepter ou refuser."*

---

## Slide 4 — Des garanties concrètes

> *Lire les bullets en les reformulant, ne pas les lire mot pour mot.*

- Introduire : *"Concrètement, qu'est-ce que ça apporte pour Lidl ?"*
- Sur les stocks : *"Les stocks sont vérifiés en temps réel — pas de commande acceptée si le produit n'est plus disponible."*
- Sur les créneaux : *"Les créneaux de retrait ont une capacité limitée — impossible de saturer le comptoir ou le drive."*
- Sur la sécurité : *"Les mots de passe ne sont jamais stockés en clair — ils sont chiffrés avant d'être enregistrés."*
- Sur le retrait drive : *"Le retrait voiture se fait par scan d'un QR code unique généré à la commande — rapide, sans contact, sans erreur."*
- Conclure : *"Ce sont des garanties invisibles pour l'utilisateur, mais qui font toute la différence dans la fiabilité du service au quotidien."*

---

## Transition finale (optionnelle si il reste du temps)

- *"Mon collègue va vous présenter la base de données qui stocke toutes ces informations, et vous verrez comment les deux parties s'articulent."*

