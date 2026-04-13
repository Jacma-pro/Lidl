Pour commencer j'ai fait une vielle sur comment conteneuriser une application et j'ai vu que c'était le même principe que le dernier TP fait avec Thomas mais avec des nouveautées. 
``` 
comme l'ajout de la base de donnée 
ou encore la notion de volume 

---
Concernant les stacks utilisées j'ai noté
- Frontend : React(vite)
- Backend : NestJS
- Base de donnée : PostgresSQL(Supabase)
--- Cache : Redis (optionnel)
J'ai décidé de mettre entre parenthèse Redis. Pourquoi ?
- Redis est un système de gestion de base de données en mémoire, souvent utilisé pour le caching et la gestion de sessions. 
il peut être utilisé dabs notre cas pour améliorer les perfomances de l'application en stockant temporairement des données fréquemment consultées, comme les sessions utilisateur ou les résultats de requêtes coûteuses. Mais comme on est sur un POC je ne pense pas que l'objectif soit d'optimiser les perfomances. 

--- Reverse proxy : Nginx (optionnel)
- Il est utilisé pour centraliser toutes les requêtes entrantes et les rediriger vers les services appropriés (frontend, backend, etc.)./moi j'ai noté ici qu'il centrailse les requêtes vers les ports 80 et 443.
- Il peut également gérer le SSL/TLS pour sécuriser les communications, ce qui est crucial pour protéger les données des utilisateurs et assurer la confidentialité.

--- Justification de l'utilisation de Docker :

