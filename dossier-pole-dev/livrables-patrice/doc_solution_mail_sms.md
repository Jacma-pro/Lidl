<link rel="stylesheet" href="../_style.css">

# Mini-doc technique — solutions retenues / alternatives

## 1. Hébergement des comptes utilisateurs : Mon Compte Lidl

### Rôle
Service central de gestion des comptes clients Lidl.
- création de compte
- authentification
- vérification e-mail / téléphone
- rattachement du compte aux autres services Lidl

### Pourquoi c’est cohérent
Lidl publie que Mon "Compte Lidl" fonctionne avec une logique de vérification par e-mail ou SMS selon le service utilisé. Lidl publie aussi que lidl.fr est hébergé sur Microsoft Azure, Google Cloud Platform et Schwarz IT KG, ce qui va dans le sens d’un service de compte centralisé à l’échelle nationale.

### Avantages
- centralisation des comptes
- cohérence entre les services Lidl
- meilleure gestion de la sécurité
- compatible avec une architecture cloud nationale

### Inconvénients
- dépendance à une infrastructure centrale
- complexité plus forte qu’une gestion locale par magasin




## 2. Mail transactionnel : Brevo Transactional Email API v3

### Rôle
Envoyer automatiquement les e-mails liés au compte et au service.
- activation de compte
- confirmation de commande
- reset de mot de passe
- confirmation de réservation
- e-mails de suivi

### Pourquoi c’est cohérent
Brevo documente son API transactionnelle comme une solution faite pour les messages automatiques non promotionnels, par exemple la création de compte, les confirmations de commande et les password resets.²

### Avantages
- intégration simple en API REST
- rapide à déployer
- bon suivi des envois
- gestion native des templates
- webhooks disponibles pour suivre les statuts

### Inconvénients
- dépendance à un prestataire tiers
- données qui sortent du SI interne
- coût récurrent SaaS
- souveraineté plus limitée qu’une solution auto-hébergée




## 3. SMS notificationnel : Brevo Transactional SMS API v3

### Rôle
Envoyer des SMS automatiques liés au service.
- confirmation courte
- notification de commande
- rappel de retrait
- alerte de changement de statut
- éventuellement SMS promotionnels via d’autres usages Brevo

### Pourquoi c’est cohérent
Brevo documente son Transactional SMS comme un service destiné aux notifications et confirmations non promotionnelles. La plateforme Brevo gère aussi d’autres cas marketing, mais il faut bien séparer transactionnel et promotionnel dans la configuration métier.

### Avantages
- même écosystème que l’e-mail
- intégration API simple
- pratique pour les notifications courtes
- déploiement rapide

### Inconvénients
- dépendance à un prestataire externe
- coût par SMS
- moins de contrôle qu’une passerelle interne




## 4. Alternative mail transactionnel : Postal

### Rôle
Plateforme open source d’envoi d’e-mails transactionnels, auto-hébergée.
- envoi par API HTTP ou SMTP
- gestion DKIM
- suppression list
- webhooks
- alternative self-hosted à un SaaS type Brevo

### Pourquoi c’est intéressant
Postal est pensé comme une plateforme mail sortante open source comparable à SendGrid ou Mailgun, mais hébergée par l’entreprise elle-même.

### Avantages
- meilleur contrôle des données
- hébergement maîtrisé par Lidl
- plus grande souveraineté
- logique compatible avec un SI interne

### Inconvénients
- plus lourd à exploiter
- gestion DNS / délivrabilité à la charge de Lidl
- réputation IP à maintenir
- moins naturel sur Azure si l’objectif est de faire du delivery SMTP direct, car Azure limite le SMTP sortant sur le port 25 et recommande plutôt un relay authentifié sur 587. :contentReference[oaicite:5]{index=5}




## 5. Alternative SMS : Jasmin

### Rôle
Passerelle SMS open source.
- exposition HTTP ou SMPP
- routage des SMS
- connexion à un fournisseur SMS amont
- gestion des retours de livraison

### Pourquoi c’est intéressant
Jasmin est une passerelle SMS open source en Python/Twisted, conçue pour envoyer des SMS via HTTP ou SMPP, avec dépendances RabbitMQ et Redis.

### Fonctionnement
L’application n’envoie pas directement le SMS au téléphone.
Le flux est :
- application
- Jasmin
- upstream SMS
- opérateur mobile
- téléphone du client

### Exemple d’upstream
Un fournisseur SMS amont de type LINK Mobility ou équivalent, raccordé en SMPP. C’est ce prestataire qui donne réellement accès au réseau télécom.

### Avantages
- open source
- très bon contrôle du routage
- compatible avec une architecture centralisée
- plus souverain qu’un SaaS pur

### Inconvénients
- plus complexe à exploiter
- il faut quand même un fournisseur SMS amont
- demande plus de compétences techniques qu’une API SaaS simple