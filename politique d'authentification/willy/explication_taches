# Synthèse des Réalisations : Sécurité, Paiement et Développement

Ce document détaille la mise en œuvre technique et stratégique des trois piliers du projet : la politique d'authentification, la sécurisation du tunnel de paiement et le développement de l'interface utilisateur.

---

## 1. Conception et rédaction de la politique d’authentification

Le travail a débuté par la création d'un cadre de sécurité rigoureux pour l'application **Lidl Collect**, visant à protéger les données des clients, des préparateurs et des administrateurs. N'ayant pas de connaissances préalables sur la rédaction d'une telle politique, une phase d'investigation approfondie a été menée auprès de la **CNIL**, de l’**ANSSI** et de l’**OWASP**, tout en analysant la politique de confidentialité de Lidl de novembre 2025. 

Le système repose sur la technologie des **JWT (JSON Web Tokens)**, qui agissent comme des badges numériques temporaires prouvant l'identité à chaque action sans surcharger les serveurs. Pour sécuriser ces accès, une architecture à deux niveaux a été mise en place via la gestion des sessions :
* **Access Token :** Durée de validité très courte (15 min pour les clients, 10 min pour les administrateurs) pour limiter les risques en cas d'interception.
* **Refresh Token :** Permet de renouveler le badge automatiquement avec des durées adaptées au risque (7 jours pour un client, 4 heures pour un administrateur).

Pour protéger ces jetons du vol par des logiciels malveillants, ils sont stockés dans des **cookies sécurisés** (paramétrés en *HttpOnly* et *Secure*), les rendant invisibles pour les scripts et limitant leur circulation aux connexions cryptées. La protection des mots de passe repose sur l’algorithme **Bcrypt** (recommandé par la CNIL), qui transforme le mot de passe en un "hachage" irréversible augmenté d'un **"sel" (salt)** unique. Enfin, une **authentification forte (MFA)** est imposée pour les comptes sensibles conformément aux principes de l’ANSSI.

---

## 2. Stratégie théorique pour le paiement en ligne sécurisé

Sur le volet du paiement, l'étude a porté sur la réduction maximale des risques lors du stockage et du transport des données. Un arbitrage a été réalisé entre le chiffrement et la **tokenisation**. Le choix s'est porté sur la tokenisation (standard IBM et Stripe), où le numéro de carte est remplacé par un jeton sans valeur marchande. En cas de fuite de données, Lidl ne perd rien de sensible. 

Pour sécuriser le flux, plusieurs mécanismes ont été spécifiés :
* **Transport :** Utilisation obligatoire du protocole **TLS 1.2+ (HTTPS)** pour chiffrer le tunnel entre le client et le site.
* **Interface :** Utilisation d'**iFrames isolées** (type Stripe Elements) pour protéger contre le *formjacking* (le numéro de carte ne touche jamais notre code source).
* **Validation :** Intégration du **3D Secure 2** (obligation DSP2) utilisant la biométrie pour confirmer le consentement de l'acheteur.
* **Prévention :** Algorithmes de **Machine Learning** pour détecter les comportements de fraude en amont.

**Données conservées :** Seuls le jeton (token), le statut de vérification, le *fingerprint* de l'appareil et les 4 derniers chiffres de la carte (pour l'affichage) sont conservés. Le CVV et la date d'expiration complète ne sont jamais stockés.

---

## 3. Développement opérationnel : Page Fruits & Légumes

La phase de réalisation concrète a eu lieu dans un environnement **React** (développement local). Pour collaborer efficacement, la gestion de versions a été opérée via **GitHub** sur des branches isolées avant fusion sur la branche principale (*main*), évitant ainsi tout conflit de code.

Le travail pratique a consisté à transformer les maquettes des designers en une interface réelle pour la catégorie **"Fruits et Légumes"** :
* **Composants :** Création de blocs réutilisables pour les cartes produits (sans bordures, badges promotionnels, prix dynamiques).
* **Style :** Utilisation du langage **SCSS** et de la méthode **BEM** pour une structure de styles rigoureuse et maintenable.
* **Responsive Design :** Optimisation de l'affichage pour garantir une expérience fluide sur smartphone.
* **Logique :** Connexion des éléments au **CartContext** pour la gestion du panier en temps réel.

Ce parcours complet démontre une progression allant de la recherche théorique (normes ANSSI, CNIL, Checkout.com) à une mise en application technique sécurisée, ergonomique et fonctionnelle.