<link rel="stylesheet" href="./_cyberleo-style.css">

# Analyse des sous-traitants — Lidl Collect (Art. 28 RGPD)

> Référence : Art. 28 RGPD (Sous-traitant)
> Art. 46 RGPD (Transferts vers pays tiers)
> Dernière mise à jour : 2026-04-15

---

## Définitions préalables

**Sous-traitant (Art. 28 RGPD) :** Toute personne physique ou morale traitant des données personnelles pour le compte du responsable de traitement, sur instruction de celui-ci.

**Responsable conjoint (Art. 26 RGPD) :** Deux entités déterminant conjointement les finalités et les moyens du traitement.

**Accord de traitement des données (DPA) :** Contrat requis par l'article 28, paragraphe 3 du RGPD entre le responsable de traitement et chaque sous-traitant. Il doit couvrir les éléments suivants : l'objet, la durée, la nature et la finalité du traitement, le type de données, les mesures de sécurité ainsi que les droits des personnes concernées.

---

## Cartographie des sous-traitants

### Sous-traitant #1 : Supabase (Hébergement BDD)

| Champ | Valeur |
|---|---|
| **Rôle** | Hébergeur de la base de données PostgreSQL |
| **Qualification RGPD** | **Sous-traitant** au sens de l'Art. 28. Supabase héberge et stocke les données personnelles pour le compte du projet, sans en déterminer les finalités. |
| **Données traitées** | Toutes les données personnelles stockées en base (profils clients, commandes, données employés, consentements, journaux d'audit) |
| **Localisation des serveurs** | **Frankfurt, Allemagne (UE)**, confirmé par Sabry/David |
| **Transfert hors UE** | **Aucun.** Données hébergées en UE, aucune réplication hors Union européenne. |
| **DPA disponible** | **Oui.** DPA standard disponible sur [supabase.com/privacy](https://supabase.com/privacy), compatible RGPD. |
| **Certifications** | SOC 2 Type 2 (en cours), ISO 27001 (via infrastructure AWS Frankfurt) |
| **Chiffrement** | AES-256 au repos, TLS en transit |
| **Sauvegardes** | Chiffrées AES-256, même région Frankfurt. Stratégie prototype : exports manuels hebdomadaires via CLI Supabase (plan Free, aucune sauvegarde automatique). Stratégie recommandée en production : PITR 7 jours, WAL continu vers AWS S3 Frankfurt et sauvegardes physiques périodiques (fréquence interne Supabase, non configurable). Suppression manuelle d'un enregistrement dans les sauvegardes : **impossible**. Risque résiduel documenté et accepté (voir `politique-effacement.md`, section 6). |
| **Risque résiduel** | Une donnée supprimée de la base principale subsiste dans les fichiers WAL et les sauvegardes physiques pendant toute la durée de rétention. En cas de restauration depuis un point antérieur à une demande d'effacement Art. 17, les suppressions et pseudonymisations concernées doivent être ré-appliquées manuellement avant remise en service. L'utilisateur est informé du délai résiduel de sept jours dans l'email de confirmation. |
| **Statut conformité** | ✅ **Conforme.** DPA disponible, données en UE, chiffrement AES-256. |
| **Lacune L05** | ✅ **Résolue.** Fréquence des sauvegardes confirmée : WAL continu (à chaque transaction) et sauvegardes physiques périodiques internes Supabase. Source : `BDD/Save_Retention_PITR`. |

---

### Sous-traitant #2 : Brevo Transactional Email API v3

| Champ | Valeur |
|---|---|
| **Rôle** | Envoi des notifications transactionnelles par email (confirmation de commande, commande prête, confirmation d'effacement, double opt-in, réinitialisation du mot de passe) |
| **Qualification RGPD** | **Sous-traitant** au sens de l'Art. 28. Brevo traite les adresses email et le contenu des notifications pour le compte du projet. |
| **Données traitées** | Adresse email du destinataire, prénom (personnalisation), contenu de la notification (type de commande, statut) |
| **Provider retenu** | **Brevo** (ex-Sendinblue, Sendinblue SAS, société française, Paris), retenu par Patrice |
| **Localisation des serveurs** | UE, datacenters AWS eu-west-1 (Irlande, EEE) et infrastructure propre France |
| **Transfert hors UE** | **Aucun.** Données hébergées dans l'EEE. |
| **DPA disponible** | **Oui.** DPA signable depuis l'interface Brevo. **Action requise : signer le DPA avant mise en production.** |
| **Bloquant L01** | ✅ **Résolu.** Provider email identifié et retenu (Brevo, EEE). |
| **Analyse** | La nationalité française de Brevo ne constitue pas en soi une garantie de conformité. Ce qui détermine la conformité : la localisation des serveurs (AWS Irlande, EEE) et l'existence d'un DPA signable. L'option auto-hébergée (Postal) a été écartée en raison de la complexité de configuration (réputation IP à construire, SPF/DKIM/DMARC à déployer sans base préexistante, absence de SLA), injustifiée pour un prototype. |
| **Point de sortie** | Brevo retenu ✅. DPA à signer depuis l'interface Brevo. Vérifier qu'aucune extension analytique transmettant des données hors UE n'est activée dans la configuration du compte. |
| **Statut conformité** | ✅ **Conforme** sous réserve de signature du DPA |

---

### Sous-traitant #3 : Brevo Transactional SMS API v3

| Champ | Valeur |
|---|---|
| **Rôle** | Envoi de SMS automatiques liés au service (confirmation courte, notification commande prête, rappel de retrait, alerte changement de statut) |
| **Qualification RGPD** | **Sous-traitant** au sens de l'Art. 28. Brevo traite les numéros de téléphone et le contenu des SMS pour le compte du projet. |
| **Données traitées** | Numéro de téléphone du destinataire, contenu du SMS (statut commande, code de retrait) |
| **Provider retenu** | **Brevo Transactional SMS**, même écosystème que le service email, retenu par Patrice |
| **Localisation des serveurs** | UE, même infrastructure que le service email Brevo |
| **Transfert hors UE** | **Aucun** côté Brevo. Note : le SMS transite ensuite par les opérateurs mobiles (Orange, SFR, etc.), hors périmètre RGPD du traitement applicatif. |
| **DPA disponible** | **Oui.** Couvert par le même DPA Brevo que le service email. |
| **Analyse** | Jasmin (passerelle SMS open source) a été écarté : pertinent pour un agrégateur à grande échelle, superflu pour un prototype. LINK Mobility directement via API REST constituerait une alternative viable, mais Brevo offre le même écosystème que l'email (un seul DPA, une seule intégration). Jasmin et LINK Mobility écartés au profit de Brevo. |
| **Point de sortie** | Brevo SMS retenu ✅. DPA commun avec le service email : un seul contrat à signer. Les communications transactionnelles (notifications commande) doivent être strictement séparées des communications promotionnelles dans la configuration Brevo. |
| **Statut conformité** | ✅ **Conforme** sous réserve de signature du DPA |

---

### Composant interne : Nginx (reverse proxy et cache)

| Champ | Valeur |
|---|---|
| **Rôle** | Reverse proxy et cache côté serveur. Redis ayant été abandonné, Nginx assure la mise en cache des assets statiques. |
| **Qualification RGPD** | **Composant interne.** Nginx est opéré par l'équipe projet, pas un sous-traitant tiers. Aucun DPA requis. |
| **Données potentiellement traitées** | Adresses IP dans les journaux (donnée personnelle). Risque identifié : réponses API mises en cache si des routes authentifiées sont couvertes par la même politique de cache que les assets statiques. |
| **Localisation** | Infrastructure locale / Docker, même périmètre que l'application |
| **Transfert hors UE** | Aucun |
| **Constat initial** | Notes David (`infradavid/mesnotes.md`) : un seul cache Nginx couvrant assets statiques et routes API authentifiées, TTL 5 à 10 minutes côté serveur. |
| **Analyse** | Les assets statiques (JS, CSS, images) sont identiques pour tous les utilisateurs : la mise en cache ne présente pas de risque. Les routes API authentifiées retournent des données personnelles propres à chaque utilisateur : le cache Nginx peut les confondre entre sessions si la clé de cache est l'URI et non le token JWT. Deux classes d'objets aux besoins opposés, traitées avec une politique unique : non-conformité identifiée. |
| **Mesure corrective documentée** | Séparation en deux blocs Nginx distincts : bloc `/` avec cache activé pour les assets statiques, bloc `/api/` avec `proxy_no_cache 1`. Dorian ajoute un intercepteur NestJS global (`Cache-Control: private, no-store`) sur toutes les routes protégées par le Guard JWT, filet de sécurité indépendant de la configuration Nginx. Voir [conseils-equipe.md](../../Lidl-Perso/conseils-equipe.md), section "Cache Nginx". |
| **Statut conformité** | ⚠️ **Correction documentée, implémentation requise avant mise en production** |

---

### Composant interne : Docker / Infrastructure

| Champ | Valeur |
|---|---|
| **Rôle** | Conteneurisation de l'application (NestJS backend, React frontend, Nginx) |
| **Qualification RGPD** | **Composant interne.** Pas de sous-traitant tiers si hébergé sur infrastructure propre. |
| **Transfert hors UE** | Aucun si hébergement local ou UE |
| **Statut conformité** | ✅ **Non concerné** : pas de traitement de données personnelles en propre. |

---

## Tableau récapitulatif

| Sous-traitant | Qualification | Localisation | DPA | Transfert hors UE | Statut |
|---|---|---|---|---|---|
| **Supabase** | Sous-traitant Art. 28 | Frankfurt, UE ✅ | Disponible ✅ | Non ✅ | ✅ Conforme |
| **Brevo Email** | Sous-traitant Art. 28 | AWS Irlande + FR, UE ✅ | À signer ⚠️ | Non ✅ | ✅ Conforme (DPA à signer) |
| **Brevo SMS** | Sous-traitant Art. 28 | UE ✅ | À signer ⚠️ | Non ✅ | ✅ Conforme (DPA à signer) |
| **Nginx** | Composant interne | Infrastructure locale | N/A | Non ✅ | ⚠️ Correction documentée |
| **Docker** | Composant interne | Infrastructure locale | N/A | Non ✅ | ✅ Non concerné |

---

## Transferts hors Union européenne — Cadre juridique de référence

Cette section est conservée à titre de référence. Elle ne s'applique pas au projet en l'état, tous les prestataires retenus étant hébergés dans l'EEE.

### Option 1 : Clauses contractuelles types (CCT, décision UE 2021/914)

La Commission européenne a adopté de nouvelles clauses contractuelles types le 4 juin 2021. Ces clauses permettent de transférer des données hors de l'Union européenne vers un prestataire non soumis à une décision d'adéquation, à condition de :

1. Signer les clauses contractuelles types avec le prestataire (module 2 : responsable vers sous-traitant)
2. Réaliser un Transfer Impact Assessment (TIA) pour évaluer les risques liés au droit local du pays destinataire

### Option 2 : Décision d'adéquation

Les États-Unis bénéficient du **Data Privacy Framework (DPF)** depuis juillet 2023, décision d'adéquation de la Commission européenne. Si le prestataire est certifié DPF, aucune clause contractuelle type n'est nécessaire. La certification est vérifiable sur [dataprivacyframework.gov](https://www.dataprivacyframework.gov).

### Option 3 : Prestataire établi dans l'Union européenne

La solution la plus directe consiste à recourir à un prestataire dont les serveurs sont établis au sein de l'Union européenne. Brevo (anciennement Sendinblue, France) ou Postmark (points de terminaison européens) constituent des alternatives recevables.

---

## Suivi des actions

| Priorité | Action | Responsable | Statut |
|---|---|---|---|
| 🔴 | Identifier le service email transactionnel et vérifier la localisation des serveurs | Patrice | ✅ Résolu — Brevo retenu (EEE) |
| 🔴 | Si email hors UE : signer les CCT ou choisir un prestataire UE | Équipe | ✅ Non applicable — Brevo hébergé dans l'EEE |
| 🟠 | Signer le DPA Brevo (email + SMS) avant mise en production | Patrice | ⚠️ Action requise |
| 🟠 | Implémenter la séparation des blocs Nginx (assets / routes API) | David / Dorian | ⚠️ Correction documentée, implémentation requise |
| 🟠 | Confirmer la fréquence des sauvegardes Supabase | Sabry | ✅ Résolu — WAL continu, lacune L05 clôturée |
| 🟡 | Décider du canal SMS et identifier le provider | Équipe | ✅ Résolu — Brevo SMS retenu |

---

← [registre-traitement.md](registre-traitement.md) | → [dpia-analyse.md](dpia-analyse.md)
