<link rel="stylesheet" href="../_style.css">

# Notes oral – Présentation Backend Lidl Collect
**Durée totale : 6 min 40 · intro pôle dev 1 min 40 + backend 5 min**

---

## Intro — Responsable Pôle Développement (1 min 40)

> *Prise de parole en premier, avant de passer la main aux autres membres du pôle.*

- Poser le contexte du pôle : *"Le pôle Développement avait une mission claire : rendre cette vision concrète et fiable."*
- Expliquer la philosophie : *"On a abordé la technologie comme un service qui doit s'effacer devant l'usage — l'utilisateur ne doit jamais ressentir la complexité de ce qui tourne derrière."*
  - Développer : insister sur le fait que la complexité logistique de Lidl (stocks, créneaux, multi-profils) est réelle, et que le défi c'est de la rendre invisible pour le client final.
- Présenter ce qu'on a construit : *"On a développé une application complète de Drive et Click & Collect, pensée pour absorber la logistique de Lidl sans jamais la faire peser sur le client."*
  - Développer : citer rapidement les trois grandes briques — le frontend pour l'expérience utilisateur, le backend pour la logique et la sécurité, la base de données pour la donnée.
- Conclure en passant la main : *"Architecture robuste, données sécurisées, parcours fluide — ce sont les fondations sur lesquelles repose la confiance de vos clients. Et c'est ce que chacun d'entre nous va vous détailler maintenant."*
  - Développer : montrer que le pôle a travaillé de manière cohérente et coordonnée — chaque partie s'emboîte avec les autres.

---

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

---

## Conclusion — Responsable Pôle Développement (1 min)

> *Reprendre la parole après les autres membres du pôle, pour clore.*

- Ouvrir sur le bilan global : *"Au-delà des outils qu'on vient de vous présenter, ce qu'on a construit, c'est un socle technologique durable."*
  - Développer : rappeler que chaque brique — front, back, base de données — a été pensée pour fonctionner ensemble, pas indépendamment.
- Insister sur la pérennité : *"Ma priorité a été de faire en sorte que notre expertise soit une force invisible — un moteur fiable, qui tient dans le temps."*
  - Développer : le système peut évoluer, on peut ajouter des magasins, de nouveaux profils, de nouvelles fonctionnalités sans repartir de zéro.
- Conclure sur l'ambition Lidl : *"Ce socle est capable de soutenir vos ambitions de croissance nationale et de s'adapter à l'évolution de vos services — sans jamais faillir face à la réalité du terrain."*
  - Développer : évoquer concrètement ce que "réalité du terrain" veut dire — pics de commandes, ruptures de stock, multi-magasins — autant de situations que le système gère.

