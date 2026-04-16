<link rel="stylesheet" href="./_cyberleo-style.css">

# Évaluation de la nécessité d'une DPIA Lidl Collect (Art. 35 RGPD)

> Référence : Art. 35 RGPD : Analyse d'impact relative à la protection des données (AIPD / DPIA)
> Grille CNIL : 9 critères : DPIA obligatoire si ≥ 2 critères remplis
> Dernière mise à jour : 2026-04-15

---

## Contexte

L'Art. 35 RGPD impose une DPIA (Data Protection Impact Assessment) lorsqu'un traitement est "susceptible d'engendrer un risque élevé pour les droits et libertés des personnes physiques". La CNIL a publié une liste de 9 critères permettant d'évaluer ce risque. Si au moins 2 critères sont remplis, la DPIA est obligatoire.

Le présent document évalue chacun des 9 critères pour le projet Lidl Collect.

---

## Note méthodologique : Scoring binaire

La grille CNIL est binaire : chaque critère est soit rempli (compte pour 1), soit non rempli (compte pour 0). L'usage d'un score intermédiaire (0,5) constitue une erreur de méthode, car il produirait un seuil de déclenchement artificiel qui ne correspond pas aux lignes directrices EDPB. La grille ci-dessous applique donc un scoring strict.

---

## Grille d'évaluation — 9 critères CNIL

### Critère 1 — Évaluation ou scoring (y compris profilage)

> Traitement qui évalue ou prédit des aspects de la personnalité, comportements, situations économiques, santé, préférences, localisation ou mouvements.

| Élément | Évaluation |
|---|---|
| **Programme de fidélité** | Comptage de points simple — pas de profilage comportemental. Le score de fidélité ne prédit ni ne classe les clients. ❌ Non rempli |
| **Performance des Préparateurs** (`Performance.global_score`) | Score synthétique calculé automatiquement (commandes préparées, temps moyen, taux d'erreur, signalements de rupture). C'est une **évaluation quantifiée et automatisée d'un employé**. Les lignes directrices EDPB (WP248) citent explicitement l'évaluation de la performance des employés comme exemple de traitement remplissant ce critère. ✅ Rempli |
| **Géolocalisation** | Utilisée à la volée uniquement, non stockée. Pas d'historique de mouvements. ❌ Non rempli |

**Verdict : ✅ Critère rempli.** Le `global_score` constitue un scoring individuel automatisé d'employés au sens des lignes directrices WP248. La portée est limitée (contexte professionnel, accès restreint Manager et Admin), mais le critère est structurellement satisfait.

---

### Critère 2 — Décision automatisée avec effet juridique ou similaire

> Traitement aboutissant à une décision automatisée produisant des effets juridiques ou affectant de façon significative la personne (ex : refus automatique de crédit, licenciement automatique).

| Élément | Évaluation |
|---|---|
| **Performance Préparateurs** | Le `global_score` est calculé automatiquement et présenté au Manager. **Aucune décision automatique** ne s'en suit : pas de licenciement automatique, pas de sanction sans intervention humaine. Le score est un outil d'aide à la décision, pas une décision en soi. ❌ Non rempli |
| **Commandes** | Les statuts de commande évoluent automatiquement mais n'ont aucun effet juridique sur le client. ❌ Non rempli |
| **Suspension de compte** | La désactivation d'un compte inactif (2 ans) est précédée d'une notification et d'un délai de réactivation — intervention humaine possible. ❌ Non rempli |

**Verdict : ❌ Critère non rempli**

---

### Critère 3 — Surveillance systématique

> Traitement utilisé pour observer, surveiller ou contrôler les personnes, y compris la collecte de données via des réseaux.

| Élément | Évaluation |
|---|---|
| **AuditLog** | Toutes les actions sensibles de tous les utilisateurs sont journalisées (connexions, modifications de statut, exercice de droits RGPD, tentatives d'accès). C'est un monitoring systématique de l'activité de chaque utilisateur sur la plateforme. ✅ Rempli |
| **Performance des Préparateurs** | Suivi continu et automatisé des indicateurs d'activité (commandes traitées, temps moyen, taux d'erreur). C'est un monitoring systématique de l'activité professionnelle individuelle. ✅ Rempli |
| **Géolocalisation** | Non stockée — pas de surveillance des mouvements hors-plateforme. ❌ Non rempli |

**Verdict : ✅ Critère rempli.** La surveillance est bornée au contexte professionnel et aux actions sur la plateforme (pas de géolocalisation, pas de surveillance hors-service), mais elle est systématique : toutes les actions de tous les utilisateurs sont tracées. Les lignes directrices EDPB (WP248) retiennent ce critère dès lors que la surveillance couvre de manière non ponctuelle l'activité de personnes dans le cadre d'un service numérique.

---

### Critère 4 — Données sensibles (Art. 9) ou données hautement personnelles

> Traitement portant sur des catégories particulières de données (santé, origine ethnique, opinions politiques, religion, orientation sexuelle, données génétiques, biométriques) ou des données pénales.

| Élément | Évaluation |
|---|---|
| **Données collectées** | Nom, prénom, email, téléphone, adresse, historique de commandes. Aucune donnée de santé, biométrique, pénale, de religion, d'origine ethnique ou d'orientation sexuelle. ❌ Non rempli |
| **Données de performance employés** | Métriques opérationnelles (temps, erreurs, volume). Pas de données sensibles au sens Art. 9. ❌ Non rempli |
| **Données financières** | Paiement : `IN_STORE` ou `SIMULATED_ONLINE` uniquement en MVP — aucune donnée de carte bancaire réelle. ❌ Non rempli |

**Verdict : ❌ Critère non rempli**

---

### Critère 5 — Données à grande échelle

> Le traitement porte sur un nombre important de personnes, ou sur une grande quantité de données, ou couvre une zone géographique étendue.

| Élément | Évaluation |
|---|---|
| **Volume estimé** | ~500 utilisateurs actifs, ~2 000 commandes/mois (estimation projet scolaire / prototype). Volume très limité. ❌ Non rempli |
| **Périmètre géographique** | 1 magasin (Saint-Martin-d'Hères) — contexte prototype |
| **Note** | En contexte de déploiement réel Lidl (1 600 magasins, 46 000 salariés, 15M utilisateurs Lidl Plus) : ce critère serait immédiatement rempli. |

**Verdict : ❌ Critère non rempli** (dans le cadre du prototype scolaire)

---

### Critère 6 — Croisement ou combinaison de jeux de données

> Traitement qui combine, recoupe ou agrège des données issues de plusieurs sources ou collectées à des fins différentes, au-delà des attentes raisonnables de la personne.

| Élément | Évaluation |
|---|---|
| **Performance + identité Préparateur** | La table `Performance` relie `preparer_id` (identifiant personnel) à des métriques comportementales professionnelles collectées à une finalité opérationnelle distincte de l'identification. Ce croisement reste dans les attentes raisonnables d'un contrat de travail. ❌ Non rempli — le critère 6 vise les croisements au-delà des attentes raisonnables ; la gestion RH opérationnelle est dans le périmètre prévisible. |
| **AuditLog + identité utilisateur** | L'`actor_id` relie chaque action à un utilisateur identifié, mais les utilisateurs qui se connectent à un service numérique s'attendent à ce que leurs actions sur ce service soient tracées à des fins de sécurité. ❌ Non rempli au sens du critère 6. |
| **Client + commandes + fidélité** | Combinaison dans les attentes raisonnables du service. ❌ Non rempli |

**Verdict : ❌ Critère non rempli.** Les croisements opérés restent dans les attentes raisonnables de chaque relation concernée (contrat de service, contrat de travail). Le critère 6 cible les croisements inter-finalités qui surprendraient la personne — ce n'est pas le cas ici.

---

### Critère 7 — Personnes vulnérables

> Traitement portant sur des personnes dont le consentement ne peut être donné librement, notamment les mineurs, les personnes sous tutelle, les personnes âgées, les patients.

| Élément | Évaluation |
|---|---|
| **Mineurs** | Service réservé aux majeurs (18+) — déclaration sur l'honneur à l'inscription. Aucun mécanisme d'accès pour les mineurs. Pour les produits réglementés (alcool), vérification visuelle au retrait par l'employé Lidl. Tabac : vente en ligne interdite en France (Art. L.3512-3 CSP) — exclu du catalogue. ❌ Non rempli |
| **Préparateurs** | Bien que des salariés puissent se trouver dans un rapport de subordination, le traitement de leurs données de performance reste dans le cadre légal du contrat de travail. ❌ Non rempli au sens Art. 35 |
| **Personnes âgées** | La clientèle inclut des seniors (persona Jean et Monique, 65 ans), mais aucun traitement spécifique à leur vulnérabilité n'est réalisé. ❌ Non rempli |

**Verdict : ❌ Critère non rempli**

---

### Critère 8 — Usage innovant ou nouvelles technologies

> Traitement qui fait appel à des technologies ou usages nouveaux (ex : IA, reconnaissance faciale, IoT, machine learning).

| Élément | Évaluation |
|---|---|
| **Stack technologique** | React, NestJS, PostgreSQL, Docker, Nginx, JWT — stack standard, éprouvée, sans innovation technologique particulière. ❌ Non rempli |
| **IA / ML** | Aucun algorithme de machine learning ou d'intelligence artificielle. ❌ Non rempli |
| **Biométrie** | Aucune. ❌ Non rempli |

**Verdict : ❌ Critère non rempli**

---

### Critère 9 — Transfert hors UE avec niveau de risque élevé

> Traitement impliquant des transferts de données vers un pays tiers dont le niveau de protection est jugé insuffisant, sans garanties appropriées.

| Élément | Évaluation |
|---|---|
| **Supabase** | Frankfurt (UE), aucun transfert hors UE. ❌ Non rempli |
| **Brevo Email + SMS** | AWS Irlande (EEE) + infra France, aucun transfert hors UE. ❌ Non rempli |
| **Autres composants** | Infrastructure locale Docker, aucun transfert hors UE. ❌ Non rempli |

**Verdict : ❌ Critère non rempli**

---

## Tableau récapitulatif

| # | Critère | Statut | Score |
|---|---|---|---|
| 1 | Évaluation / scoring | ✅ Rempli (global_score préparateurs — évaluation automatisée individuelle) | 1 |
| 2 | Décision automatisée à effet juridique | ❌ Non rempli | 0 |
| 3 | Surveillance systématique | ✅ Rempli (AuditLog exhaustif + monitoring performance continu) | 1 |
| 4 | Données sensibles (Art. 9) | ❌ Non rempli | 0 |
| 5 | Grande échelle | ❌ Non rempli (prototype scolaire) | 0 |
| 6 | Croisement de données | ❌ Non rempli (croisements dans les attentes raisonnables) | 0 |
| 7 | Personnes vulnérables | ❌ Non rempli | 0 |
| 8 | Technologies innovantes | ❌ Non rempli | 0 |
| 9 | Transfert hors UE à risque élevé | ❌ Non rempli | 0 |
| | **Total** | | **2 / 9** |

---

## Conclusion

> **La DPIA est obligatoire** : le seuil de 2 critères remplis est atteint (critères 1 et 3). La présente analyse constitue la DPIA conduite en réponse à cette obligation.

Deux critères sur neuf sont pleinement remplis, déclenchant l'obligation légale prévue par l'article 35 du RGPD. Les critères 1 (évaluation automatisée d'employés) et 3 (surveillance systématique des actions sur la plateforme) s'appliquent au traitement T12 (suivi des performances) et T08 (AuditLog).

### Analyse des risques — Résultats de la DPIA

| Risque identifié | Source | Mesure de maîtrise | Niveau résiduel |
|---|---|---|---|
| Utilisation du `global_score` à des fins disciplinaires sans procédure contradictoire | T12 | Aucune décision automatisée — le score est un outil d'aide, toute décision RH reste humaine. Documenté dans registre-traitement.md | **Faible** |
| Accès non autorisé aux scores individuels (fuite inter-rôles) | T12 | Accès strictement limité à Manager et Admin — matrice RBAC implémentée et testée | **Faible** |
| Conservation excessive des données de performance | T12 | Durée bornée à 5 ans après fin du contrat — conforme Référentiel CNIL RH 2026 | **Faible** |
| Surveillance de l'AuditLog utilisée à des fins étrangères à la sécurité | T08 | AuditLog en append-only — aucune route DELETE — accès Admin uniquement | **Faible** |
| Ré-identification via l'AuditLog après suppression de compte | T08 | Pseudonymisation HMAC-SHA256 de l'actor_id à la suppression — sel stocké hors base | **Résiduel documenté** |
| Absence d'information préalable des préparateurs | T12 | ⚠️ Obligation Art. L1222-4 Code du travail — notice RH à formaliser avant production | **À traiter** |

### Conclusion de la DPIA : risques résiduels acceptables

Les risques identifiés sont soit maîtrisés par les mesures techniques et organisationnelles documentées, soit documentés comme risques résiduels tolérés avec mesure compensatoire. **La DPIA ne révèle pas de risque élevé résiduel non traité.** Le traitement peut être mis en œuvre sous réserve que la notice d'information aux préparateurs (Art. L1222-4) soit formalisée avant tout déploiement.

En contexte de production réelle (Lidl SAS — 46 000 employés), une consultation préalable du CSE serait également requise (Art. L2312-38 Code du travail).

### Mesures de protection en place

| Mesure | Périmètre | Statut |
|---|---|---|
| Accès `Performance` limité aux rôles Manager et Admin uniquement | T12 | ✅ Implémenté dans matrice RBAC |
| Aucune décision automatisée sur base du `global_score` | T12 | ✅ Documenté dans registre-traitement.md |
| AuditLog pseudonymisé à la suppression de compte | T08 | ✅ Documenté dans politique-effacement.md |
| Durée de conservation Performance limitée à 5 ans après départ | T12 | ✅ Documenté dans registre-traitement.md |
| Notice d'information des préparateurs (Art. L1222-4) | T12 | ⚠️ À formaliser avant production |
| Consultation CSE (contexte production réelle uniquement) | T12 | ⚠️ Hors périmètre prototype — obligation documentée |

---

## Réévaluation obligatoire si

La DPIA devra être conduite si l'un des événements suivants se produit :

| Événement | Critère déclenché |
|---|---|
| Déploiement à plus de 1 000 utilisateurs actifs | Critère 5 (grande échelle) |
| Ajout d'un algorithme de recommandation produits basé sur l'historique | Critère 1 (profilage), Critère 8 (technologie nouvelle) |
| Utilisation du `global_score` pour déclencher automatiquement des sanctions ou alertes RH | Critère 2 (décision automatisée à effet juridique) |
| Service email hébergé hors UE sans DPA ni CCT | Critère 9 (transfert hors UE à risque) |
| Ajout de données de santé ou biométriques | Critère 4 (données sensibles Art. 9) |
| Déploiement sur plusieurs magasins avec croisement des données entre sites | Critères 5 et 6 |

---

← [registre-traitement.md](registre-traitement.md) | ← [analyse-sous-traitants.md](analyse-sous-traitants.md) | ← [politique-effacement.md](politique-effacement.md)
