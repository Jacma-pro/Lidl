## Mes notes 

>Pour commencer j'ai fait une vielle sur comment conteneuriser une application et j'ai vu que c'était le même principe que le dernier TP fait avec Thomas mais avec des nouveautées comme l'ajout de la base de donnée ou encore la notion de volume 

```
Concernant les stacks utilisées j'ai noté
- Frontend : React(vite)
- Backend : NestJS
- Base de donnée : PostgresSQL(Supabase)
```

# Cache : Redis (optionnel)

J'ai décidé de mettre entre parenthèse Redis. Pourquoi ?
- Redis est un système de gestion de base de données en mémoire, souvent utilisé pour le caching et la gestion de sessions. 
il peut être utilisé dans notre cas pour améliorer les perfomances de l'application en stockant temporairement des données fréquemment consultées, comme les sessions utilisateur ou les résultats de requêtes coûteuses. Mais comme on est sur un POC je ne pense pas que l'objectif premier soit d'optimiser les perfomances mais 
plutôt de montrer ce qui est fesable mais quand même tout en disant qu'il pourrait être très important dans une version plus avancée.   
>Et ici je pense que l'utilisation de Nginx qui a un accès sera plus approprié car avec lui on pourra gérer différentes tâches (voir juste en dessous à *Nginx*)


# Nginx utilisé comme reverse proxy (à revoir)

- Il est utilisé pour centraliser toutes les requêtes entrantes et les rediriger vers les services appropriés 
-Load balancer : il peut répartir le trafic vers plusieurs serveurs
- Il peut également gérer le SSL/TLS pour sécuriser les communications
>Nginx va nous permettre de cacher l'accès l'accès à nos services comme le backend (base de données, API)
>de gérer l'aspect sécurité en chiffrant les communications et en filtrant les requêtes malveillantes
>de gérer la montée en charge en répartissant le trafic de nos services
>de gérer l'aspect performance 
> Si ca peut être utilise C'est la doc du site de Nginx https://docs.nginx.com/nginx/admin-guide/monitoring/logging/

# Logs et Cache

---
# Logs
```
En France la CNIL impose aux opérateurs et hébergeurs (ce qui inclut les entreprises gérant un site web) de conserver les données de connexion pendant 1 an. Pourquoi Fraude à la carte bancaire : Si un client conteste une commande 3 mois plus tard, tu dois pouvoir prouver via les logs l'adresse IP et le parcours de l'utilisateur.
Source :Legifrance : Article L34-1 (Obligation de conservation d'un an) et CNIL
```
---
# Cache
```
On utilise deux types de cache :
- Cache (Statique) côté client : stocke les données sur le navigateur de l'utilisateur pour accélérer les temps de chargement des pages ( Généralement utilisé pour les ressources statiques comme les images, les fichiers CSS et JavaScript etc...) durée de vie : 1 ans.
- Cache côté serveur : stocke les données sur le serveur pour réduire la charge et améliorer les performances de l'application. durée de vie : 5 à 10min.
```

---
# Frontend 
```
FROM         # image de base
WORKDIR                # dossier de travail dans le conteneur
COPY        # copier les fichiers
RUN            # installer les dépendances lors de construction de l'image
COPY . .                  # copier tout le code
EXPOSE                 # port exposé
CMD []  # commande de démarrage
```
---

# Backend
```
FROM         # image de base
WORKDIR                # dossier de travail dans le conteneur   
COPY        # copier les fichiers
RUN            # installer les dépendances lors de construction de l'image
COPY . .                  # copier tout le code
EXPOSE                 # port exposé
CMD []  # commande de démarrage
```
---
# Base de données 
```FROM         # image de base
ENV POSTGRES_USER=postgres      