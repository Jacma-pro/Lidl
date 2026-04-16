# Mes notes 

>Pour commencer j'ai fait une vielle sur comment conteneuriser une application et j'ai vu que c'était le même principe que le dernier TP fait avec Thomas mais avec des nouveautées comme l'ajout de la base de donnée ou encore la notion de volume 

```
Concernant les stacks utilisées j'ai noté
- Frontend : React(vite)
- Backend : NestJS
- Base de donnée : PostgresSQL(Supabase)
```

## Cache : Redis (Non Retenue parceque Nginx peut faire le travail de cache et en plus le travaille de sécurité et de load balancer)

J'ai décidé de mettre entre parenthèse Redis. Pourquoi ?
- Redis est un système de gestion de base de données en mémoire, souvent utilisé pour le caching et la gestion de sessions. 
il peut être utilisé dans notre cas pour améliorer les perfomances de l'application en stockant temporairement des données fréquemment consultées, comme les sessions utilisateur ou les résultats de requêtes coûteuses. Mais comme on est sur un POC je ne pense pas que l'objectif premier soit d'optimiser les perfomances mais 
plutôt de montrer ce qui est fesable mais quand même tout en disant qu'il pourrait être très important dans une version plus avancée.   
>Et ici je pense que l'utilisation de Nginx qui a un accès sera plus approprié car avec lui on pourra gérer différentes tâches (voir juste en dessous à *Nginx*)


## Nginx utilisé comme reverse proxy (Je vais en parler à L'oral mais je ne pense pas le faire dans le projet)

-Il est utilisé pour centraliser toutes les requêtes entrantes et les rediriger vers les services appropriés 
-Load balancer : il peut répartir le trafic vers plusieurs serveurs
- Il peut également gérer le SSL/TLS pour sécuriser les communications
>Nginx va nous permettre de cacher l'accès l'accès à nos services comme le backend (base de données, API)
>de gérer l'aspect sécurité en chiffrant les communications et en filtrant les requêtes malveillantes
>de gérer la montée en charge en répartissant le trafic de nos services
>de gérer l'aspect performance 
> Si ca peut être utilise C'est la doc du site de Nginx https://docs.nginx.com/nginx/admin-guide/monitoring/logging/

# Logs et Cache

## Logs
```
En France la CNIL impose aux opérateurs et hébergeurs (ce qui inclut les entreprises gérant un site web) de conserver les données de connexion pendant 1 an. Pourquoi Fraude à la carte bancaire : Si un client conteste une commande 3 mois plus tard, tu dois pouvoir prouver via les logs l'adresse IP et le parcours de l'utilisateur.
Source :Legifrance : Article L34-1 (Obligation de conservation d'un an) et CNIL
```
---
## Cache

# On utilise deux types de cache :

- Cache Statique côté client (Activé) : Le navigateur va stocké des données par exemple des images des assets JS des images(logo etc) des URL dates de connexion pour déja améliorer l'expérience utilisateur en pour accélérer les temps de chargement. 
#Concernant la durée de vie de vie des données en cache va dépendre du types de navigateur et la configuration des serveurs.
- Par exemple Google chrome eux conserve les pages Web dans son cache pendant environ 90 jours, ou jusqu’à ce que la page soit exploréeà nouveau. https://adcod.com/fr/combien-de-temps-dure-le-cache-du-navigateur/
*Mais pour la configuration du cache on peut la faire aussi nous même dans les navigateurs.*

- Cache côté serveur (Désactivé): Toutes les données qui transitent entre le frontend et le backend sont par nature personnelles et dynamiques : un solde bancaire, une liste de commandes, un statut de livraison. Si Nginx mettait ces réponses en cache, même seulement 5 minutes, un utilisateur qui effectue une action sur son compte verrait encore l'ancien état de ses données pendant toute la durée du cache. Il lirait donc une information incorrecte sans même le savoir. C'est pourquoi le cache est strictement désactivé sur les routes API.

# Important 
Les URLs à usages uniques ne doivent jamais données la même réponse à deux requêtes différentes. Par exemple, une URL de réinitialisation de mot de passe doit être unique et ne doit pas être mise en cache, sinon une autre personne pourrait réinitialiser le mot de passe d'un autre en utilisant la même URL.
#Pour un attaquant : le cache côté serveur peut être une cible pour voler des données sensibles avec le :
*Un replay attack* c'est quand un attaquant :
1. Intercepte ou récupère une URL API valide
2. La rejoue plus tard
3. Et obtient la même réponse grâce au cache.
>Sans cache, cette attaque est impossible parce que chaque requête repart de zéro jusqu'à la base de données, qui vérifie l'état réel à cet instant précis.


# Github actions 
- C'est un outil d'intégration continue (CI) et de déploiement continu (CD) proposé par GitHub. Il permet d'automatiser les processus de développement, de test et de déploiement de notre code.
Je suis le best