C'est parfait, j'ai repris l'intégralité de ton texte initial sans aucune suppression, en insérant le schéma et son analyse détaillée exactement là où ils sont les plus pertinents (après la gestion des sessions).

Voici ton document final complet :

***

# **Politique d’authentification - Lidl Collect**


## **Introduction : une sécurité pensée pour tous**

Chez Lidl, la confiance de nos utilisateurs est essentielle. Avec l’application **Lidl Collect**, nous avons conçu un système d’authentification qui protège vos données tout en restant simple à utiliser au quotidien.

Que vous soyez client, préparateur de commandes ou administrateur, chaque accès à la plateforme est sécurisé selon son niveau de sensibilité. Notre objectif est clair : garantir que chacun accède uniquement à ce qui lui est autorisé, tout en assurant une expérience fluide.

Pour construire cette architecture, nous nous appuyons sur les recommandations d’organismes reconnus comme ANSSI, CNIL et OWASP.


## **Comprendre comment vous vous connectez**

Lorsque vous vous connectez à Lidl Collect, plusieurs éléments permettent de vérifier votre identité.

Le premier est votre mot de passe, que vous seul connaissez. C’est ce qu’on appelle un facteur de connaissance. Dans certains cas, nous demandons également un second élément, comme un code généré sur votre téléphone. Cela correspond à un facteur de possession.

Le fait de combiner ces éléments s’appelle l’authentification forte, ou MFA. Elle est recommandée par l’ANSSI pour protéger les accès sensibles, notamment pour les comptes internes.


## **Pourquoi nous utilisons les tokens (JWT)**

Pour vous permettre de naviguer facilement dans l’application sans avoir à vous reconnecter en permanence, Lidl Collect utilise une technologie appelée **JWT (JSON Web Token)**.

Concrètement, après votre connexion, nous vous attribuons un jeton numérique sécurisé. Ce jeton fonctionne comme un badge temporaire qui prouve votre identité à chaque action.

Ce choix repose sur plusieurs raisons.

D’abord, il permet une navigation rapide et fluide, notamment sur mobile. Ensuite, il évite de stocker des sessions côté serveur, ce qui améliore les performances et la stabilité de la plateforme. Enfin, il permet d’intégrer directement votre rôle dans le système, ce qui facilite la gestion des droits.

Cette approche est aujourd’hui largement utilisée et recommandée dans les architectures modernes, notamment par les bonnes pratiques de l’OWASP.


## **Une sécurité renforcée grâce à la gestion des sessions**

Le système Lidl Collect repose sur deux types de jetons complémentaires qui permettent de sécuriser votre session tout en garantissant une navigation fluide.

Lors de votre connexion, un premier jeton appelé **Access Token** vous est attribué. Ce jeton est utilisé pour toutes vos actions dans l’application. Afin de limiter les risques en cas d’interception, sa durée de validité est volontairement courte. Chez Lidl Collect, elle est fixée à **15 minutes pour les clients et les opérateurs**, et **10 minutes pour les comptes administrateurs**, qui nécessitent un niveau de sécurité encore plus élevé.

Une fois ce délai dépassé, le jeton n’est plus utilisable. Cela permet de réduire fortement la fenêtre d’exploitation en cas de tentative d’accès frauduleux.

Pour éviter de vous demander de vous reconnecter en permanence, un second jeton appelé **Refresh Token** est utilisé. Celui-ci permet de générer automatiquement un nouveau Access Token lorsque le précédent expire.

La durée de ce second jeton est plus longue, mais elle est adaptée selon le niveau de sensibilité du compte. Pour un client, la session peut être maintenue jusqu’à **7 jours**, afin de garantir une expérience utilisateur confortable. En revanche, pour les utilisateurs internes, cette durée est réduite à **8 heures pour les opérateurs et managers**, et à **4 heures pour les administrateurs**, conformément aux recommandations de sécurité renforcée pour les accès sensibles.

Ce mécanisme permet de trouver un équilibre entre simplicité d’utilisation et niveau de protection élevé, en accord avec les bonnes pratiques recommandées notamment par l’OWASP et l’ANSSI.


***
![Cycle Sécurisé des Sessions JWT](./jwt-cycle.png)
***

### **Analyse visuelle du cycle de vie des sessions**

Le schéma ci-dessus illustre la synergie entre l'interface utilisateur développée sous React et l'infrastructure serveur de Lidl Collect. Ce cycle garantit que l'accès aux services reste fluide pour l'utilisateur légitime tout en maintenant une barrière de sécurité constante. Le processus débute par une connexion via un tunnel sécurisé HTTPS, protégeant les identifiants contre toute interception. Une fois la vérification effectuée par le serveur à l'aide du hachage Bcrypt, deux jetons sont émis pour segmenter les responsabilités de sécurité et de confort.

Le premier jeton, le Jeton d’Accès (Access Token), est le garant de la sécurité immédiate avec une validité courte de 15 minutes. Il accompagne chaque requête de l'utilisateur pour valider ses droits en temps réel. Le second, le Jeton de Renouvellement (Refresh Token), assure la continuité de l'expérience sur 7 jours. Comme le montre la boucle de "Renouvellement Automatique", l'application utilise ce second jeton pour obtenir un nouvel accès de manière transparente dès que le premier expire. Cette architecture permet de déconnecter instantanément un utilisateur en cas d'anomalie tout en offrant une navigation sans interruption lors d'un usage normal.


## **Comment vos mots de passe sont protégés**

La sécurité de votre compte commence dès la création de votre mot de passe.

Chez Lidl Collect, aucun mot de passe n’est stocké tel quel. Nous utilisons une technique appelée **hachage**, qui transforme votre mot de passe en une version illisible. Cette transformation est irréversible, ce qui signifie que même nous ne pouvons pas retrouver votre mot de passe d’origine.

Nous utilisons l’algorithme **Bcrypt**, recommandé par la CNIL pour le stockage des mots de passe. Son avantage principal est qu’il ralentit volontairement les calculs, ce qui rend les attaques informatiques extrêmement difficiles.

Il existe également un algorithme plus récent appelé Argon2, recommandé par l’ANSSI. Celui-ci est encore plus avancé techniquement, notamment face aux attaques utilisant des cartes graphiques. Cependant, Bcrypt reste aujourd’hui une solution fiable, éprouvée et largement utilisée dans les systèmes en production, ce qui justifie notre choix dans le cadre de Lidl Collect.

En complément, nous ajoutons une donnée aléatoire unique appelée “sel” à chaque mot de passe. Cela empêche deux mots de passe identiques d’avoir le même résultat, renforçant encore la sécurité.


## **Des accès adaptés à chaque utilisateur**

Tous les utilisateurs n’ont pas les mêmes besoins, ni les mêmes niveaux de risque. C’est pourquoi Lidl Collect applique une gestion des accès par rôle.

Un client accède uniquement à ses propres commandes et informations. Un opérateur peut consulter les commandes pour les préparer, mais n’a pas accès aux données sensibles des clients. Un manager supervise les opérations, tandis qu’un administrateur dispose d’un accès complet.

Plus le niveau de responsabilité est élevé, plus les mesures de sécurité sont renforcées. Par exemple, les accès internes nécessitent une authentification forte, et certaines connexions sont limitées à des environnements sécurisés.

Cette approche suit le principe du moindre privilège recommandé par l’ANSSI et les bonnes pratiques de l’OWASP.


## **Une protection contre les principales menaces**

Le système intègre plusieurs mécanismes pour se protéger contre les attaques courantes.

En cas de tentatives répétées de connexion, le compte est temporairement bloqué afin d’empêcher les attaques automatisées. Cette mesure est recommandée par l’OWASP.

Les tentatives de fraude, comme le phishing, sont limitées grâce à l’utilisation de la double authentification. Même si un mot de passe est compromis, l’accès reste protégé.

Enfin, les sessions sont sécurisées grâce à des durées limitées et des mécanismes de renouvellement. Cela réduit fortement les risques liés au vol de session.


## **Votre rôle dans la sécurité**

La sécurité du système Lidl Collect ne repose pas uniquement sur la technologie, mais aussi sur la vigilance de ses utilisateurs. Pour garantir une protection optimale, chacun doit respecter des règles de prudence fondamentales.

**1. Pour nos clients : les bonnes pratiques d'hygiène numérique**
* **Unicité du mot de passe :** Nous recommandons l'utilisation d'un mot de passe unique, non utilisé sur d'autres sites. Cela évite l'effet "domino" : si l'un de vos comptes est piraté ailleurs, votre compte Lidl Collect reste protégé.
* **Vigilance face aux codes de validation :** Si vous recevez un code de connexion ou une notification de double authentification (MFA) par SMS ou e-mail sans avoir initié de demande, ne le validez sous aucun prétexte. C'est le signe d'une tentative de fraude.
* **Détection du Phishing :** Lidl ne vous demandera jamais votre mot de passe par e-mail ou par téléphone. Vérifiez toujours que vous êtes sur l'application officielle ou sur le domaine lidl.fr avant de saisir vos identifiants.

**2. Pour les utilisateurs internes : la charte de comportement professionnel**
En tant qu'acteur du réseau Lidl Collect, votre accès est une porte d'entrée vers nos systèmes. Des règles strictes s'appliquent :
* **Gestion des accès personnels :** Vos identifiants sont strictement personnels. Le partage de compte entre préparateurs ou managers, même pour "gagner du temps", est une faille majeure et est formellement interdit.
* **Sécurisation immédiate du poste :** Le verrouillage de session (raccourci Windows + L) est obligatoire dès que vous vous éloignez de votre terminal de préparation ou de votre ordinateur, afin d'empêcher toute manipulation par un tiers.
* **Utilisation des outils homologués :** L'accès aux interfaces d'administration et de gestion des commandes doit se faire exclusivement via les appareils fournis et sécurisés par Lidl (PDT, terminaux mobiles, PC fixes de l'entrepôt).
* **Remontée d'incidents :** Tout comportement anormal du système (lenteur inhabituelle, fenêtres surgissantes, demandes de droits administrateur) doit être immédiatement signalé au responsable d'infrastructure ou au pôle sécurité.

**3. Engagement et Confidentialité**
Tout manquement à ces règles peut compromettre l'intégrité des données de milliers de clients. Le respect de ces consignes est un gage de professionnalisme et une condition sine qua non pour l'utilisation des services Lidl Collect.


## **Conclusion : une sécurité simple et efficace**

Avec Lidl Collect, nous avons conçu un système qui allie sécurité et simplicité.

Grâce à l’utilisation de technologies reconnues comme les JWT, le hachage Bcrypt et l’authentification forte, nous protégeons efficacement vos données tout en vous offrant une expérience fluide.

Notre approche repose sur un principe simple : rendre la sécurité invisible pour l’utilisateur légitime, tout en restant robuste face aux menaces.


## **Sources**

Recommandations de ANSSI

Recommandations de CNIL

Bonnes pratiques de OWASP