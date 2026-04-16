<link rel="stylesheet" href="./_cyberleo-style.css">

# Politique d'effacement & Droits des personnes — Lidl Collect

> Référence : RGPD Art. 17 (droit à l'effacement), Art. 20 (droit à la portabilité), Art. 12 (délais de réponse)
> Base de données : PostgreSQL via Supabase — région Frankfurt (Union européenne) — aucun transfert hors de l'Union européenne
> Redis : abandonné — aucune purge de session à prévoir

---

## 1. Cartographie des données personnelles par emplacement

### Tables supprimées physiquement à la suppression de compte

| Table | Données personnelles | Base légale de suppression |
|---|---|---|
| `Client` | nom, prénom, email, mot de passe hashé, téléphone, adresse | Art. 17 RGPD — aucune obligation de conservation |
| `Client_Account` | is_verified, is_active, last_login_at | Art. 17 RGPD — aucune obligation de conservation |
| `Client_History` | loyalty_points, loyalty_status | Art. 17 RGPD — aucune obligation de conservation |
| `Cart` | client_id | Art. 17 RGPD — aucune obligation de conservation |
| `CartItem` | via cart_id (cascade) | Art. 17 RGPD — aucune obligation de conservation |
| `Notification` | client_id | Art. 17 RGPD — aucune obligation de conservation |

### Tables pseudonymisées (conservation légale obligatoire)

> **Note juridique importante** — La pseudonymisation est une mesure de réduction du risque (Art. 32 RGPD), **pas un effacement** au sens de l'Art. 17. Les données ci-dessous ne sont pas effacées parce qu'une exception à l'Art. 17.3 s'applique — la pseudonymisation vient en complément pour minimiser le risque résiduel. L'exception Art. 17.3 applicable est identifiée colonne par colonne.

| Table | Champ pseudonymisé | Exception Art. 17.3 applicable | Base légale de conservation | Durée |
|---|---|---|---|---|
| `Order` | client_id → token anonyme | Art. 17.3.b — obligation légale (Art. L123-22 Code de commerce) | Art. 6.1.c — Obligation légale comptable | 10 ans |
| `OrderItem` | via order_id | Art. 17.3.b — même base | Art. 6.1.c | 10 ans |
| `Payment` | via order_id | Art. 17.3.b — même base + directive TVA 2006/112/CE | Art. 6.1.c | 10 ans |
| `SubstitutionProposal` | via order_item_id | Art. 17.3.b — même base | Art. 6.1.c | 10 ans |
| `AuditLog` | actor_id → token anonyme | Art. 17.3.e — constatation, exercice ou défense de droits en justice | Art. 6.1.f — intérêt légitime (sécurité Art. 32) | 1 an |
| `Consent` | client_id → token anonyme | Art. 17.3.e — preuve de conformité réglementaire (accountability Art. 5.2) | Art. 6.1.f — intérêt légitime | Durée indéterminée |
| `Schedule` | preparer_id → token anonyme | Art. 17.3.b — obligation légale (Art. L3243-4 Code du travail) | Art. 6.1.c | 5 ans après départ |
| `Performance` | preparer_id → token anonyme | Art. 17.3.b — même base | Art. 6.1.c | 5 ans après départ |

### Emplacements hors base de données

| Emplacement | Données personnelles | Durée légale | Procédure à la demande individuelle |
|---|---|---|---|
| Logs Nginx | ip_address, user-agent | 1 an — Art. L34-1 | Non modifiable individuellement — durée légale fixe |
| Logs applicatifs NestJS | À vérifier avec Dorian — aucun email en clair autorisé | 1 an — Art. L34-1 | Non modifiable individuellement |
| Sauvegardes Supabase | Copie complète de la base | 7 jours (plan actuel Supabase) | Risque résiduel documenté — voir section 6 |

---

## 2. Règle de pseudonymisation

### Algorithme retenu : HMAC-SHA256

```
token_anonyme = HMAC-SHA256(userId, sel_secret)
```

- Le sel est stocké en variable d'environnement (jamais en base)
- HMAC-SHA256 est préféré à SHA-256 simple : résistant aux attaques par force brute sur des userId prévisibles (séquences PostgreSQL)
- La même fonction appliquée au même userId produit toujours le même token → cohérence entre tables

### Qualification juridique importante

> Cette opération est une **pseudonymisation** au sens du considérant 26 du RGPD — **pas une anonymisation**.
> Tant que le sel existe, la ré-identification reste théoriquement possible par quiconque le détient.
> Les données pseudonymisées restent des données personnelles soumises au RGPD.
> La pseudonymisation est une mesure de sécurité qui réduit le risque sans l'éliminer.

### Gestion de la rotation du sel

- En cas de rotation du sel, les anciens tokens deviennent orphelins dans les tables pseudonymisées
- Procédure : remappage complet des tokens avant rotation (script de migration)
- La rotation ne doit pas être effectuée sans coordination entre l'équipe BDD et cyber

---

## 3. Procédure de traitement d'une demande d'effacement (Art. 17)

### Délai légal : 30 jours (Art. 12 RGPD)

```
Étape 1 — Réception et vérification d'identité
  → Via endpoint DELETE /api/user/me : JWT httpOnly cookie valide = identité vérifiée
  → Via email / formulaire hors-session :
      - Demander un code de confirmation envoyé sur l'email du compte
      - Ne jamais traiter une demande sans vérification — risque de suppression forcée d'un tiers
  → Enregistrement dans AuditLog (action : DELETION_REQUESTED)

Étape 2 — Vérification des obligations légales
  Arbre de décision :
  - Le compte a-t-il des commandes validées ?
      → Oui : pseudonymisation de Order, OrderItem, Payment, SubstitutionProposal
      → Non : suppression physique uniquement
  - Le compte a-t-il des entrées dans AuditLog ?
      → Oui : pseudonymisation de actor_id
  - Le compte a-t-il des enregistrements Consent ?
      → Oui : pseudonymisation de client_id (jamais suppression)

Étape 3 — Exécution (transaction atomique — voir section 4)
  → Pseudonymisation des FK dans les tables sous obligation légale
  → Suppression physique des tables sans obligation légale
  → Entrée AuditLog (DELETION_COMPLETED) avec actor_id déjà pseudonymisé

Étape 4 — Confirmation à l'utilisateur
  → Email de confirmation sous 30 jours
  → Contenu : liste de ce qui est supprimé, liste de ce qui est conservé + base légale
```

---

## 4. Spécification des endpoints

### DELETE /api/user/me

**Authentification requise :** JWT httpOnly cookie valide (rôle CLIENT)
**Exécution :** transaction atomique (BEGIN / COMMIT — ROLLBACK en cas d'échec partiel)

> Une suppression partielle est pire qu'une absence de suppression : l'utilisateur croit ses données effacées alors qu'elles ne le sont qu'en partie. Toutes les actions ci-dessous doivent être dans la même transaction PostgreSQL.

```
BEGIN;
  1. HMAC-SHA256(client_id) → token_anonyme
  2. UPDATE Order SET client_id = token_anonyme WHERE client_id = :id
  3. UPDATE AuditLog SET actor_id = token_anonyme WHERE actor_id = :id
  4. UPDATE Consent SET client_id = token_anonyme WHERE client_id = :id
  5. DELETE FROM CartItem WHERE cart_id IN (SELECT id FROM Cart WHERE client_id = :id)
  6. DELETE FROM Cart WHERE client_id = :id
  7. DELETE FROM Notification WHERE client_id = :id
  8. DELETE FROM Client_History WHERE client_id = :id
  9. DELETE FROM Client_Account WHERE client_id = :id
  10. DELETE FROM Client WHERE id = :id
  11. INSERT INTO AuditLog (actor_id, action) VALUES (token_anonyme, 'DELETION_COMPLETED')
COMMIT;
-- En cas d'erreur sur l'une des étapes :
ROLLBACK;
INSERT INTO AuditLog (actor_id, action, context) VALUES (token_anonyme, 'DELETION_FAILED', 'étape X — détail technique sans donnée personnelle');
```

**Réponse succès :** `200 OK` + email de confirmation envoyé
**Réponse échec :** `500 Internal Server Error` + retry manuel déclenché

---

### GET /api/user/export

**Authentification requise :** JWT httpOnly cookie valide (rôle CLIENT)

#### Distinction Art. 15 vs Art. 20

| | Art. 15 — Droit d'accès | Art. 20 — Droit à la portabilité |
|---|---|---|
| Périmètre | Toutes les données détenues, y compris déduites | Données fournies activement par l'utilisateur |
| Format | Tout format lisible | Format structuré, lisible par machine |
| Cet endpoint | Couverture partielle | Couverture complète |

> Cet endpoint couvre principalement l'**Art. 20**. Pour une réponse complète à l'Art. 15 (incluant logs, métadonnées, données déduites), une procédure manuelle complémentaire est nécessaire — à documenter dans la politique de confidentialité.

**Format de réponse :** JSON

```json
{
  "exported_at": "2025-04-14T10:00:00Z",
  "scope": "Art. 20 RGPD — données fournies activement",
  "account": {
    "email": "client@example.com",
    "first_name": "Prénom",
    "last_name": "Nom",
    "phone": "0600000000",
    "address": "1 rue Example, 38000 Grenoble",
    "created_at": "2024-01-15T08:00:00Z",
    "preferred_store": "Lidl Grenoble Centre"
  },
  "loyalty": {
    "points": 120,
    "status": true
  },
  "orders": [
    {
      "id": "ORD-001",
      "date": "2024-03-10T14:30:00Z",
      "store": "Lidl Grenoble Centre",
      "status": "PICKED_UP",
      "total_price": 34.50,
      "pickup_slot": "2024-03-11 10h00-10h30",
      "items": [
        { "product": "Lait entier 1L", "quantity": 2, "unit_price": 1.29 }
      ]
    }
  ],
  "consents": {
    "cookie_analytics": false,
    "cookie_loyalty": true,
    "consented_at": "2024-01-15T08:05:00Z"
  }
}
```

---

## 5. Template de réponse à l'utilisateur

```
Objet : Confirmation de suppression de votre compte Lidl Collect

Bonjour,

Nous avons bien reçu et traité votre demande de suppression de compte.

Ce qui a été supprimé :
- Vos informations personnelles (nom, prénom, email, téléphone, adresse)
- Votre historique de connexion et de fidélité
- Votre panier et vos préférences

Ce qui est conservé (obligation légale) :
- Vos historiques de commandes sous forme pseudonymisée — 10 ans
  (Art. L123-22 du Code de commerce)
- Les traces de sécurité pseudonymisées — 1 an
  (Art. 6.1.f RGPD — démonstration de conformité)
- Les enregistrements de consentement pseudonymisés — durée indéterminée
  (preuve légale de consentement)

Note sur les sauvegardes :
Vos données peuvent subsister dans nos sauvegardes chiffrées
pendant une durée maximale de sept jours après suppression.
Ces sauvegardes sont chiffrées et hébergées en Europe (Frankfurt).

Votre identité directe n'est plus associée à ces données conservées ; elles sont pseudonymisées et ne permettent pas votre identification sans recours à une clé de correspondance détenue par le responsable du traitement.

Pour toute question, vous pouvez contacter la CNIL :
https://www.cnil.fr/fr/plaintes

Cordialement,
L'équipe Lidl Collect
```

---

## 6. Politique de traitement des sauvegardes Supabase

- Sauvegardes chiffrées AES-256, hébergées région Frankfurt (Union européenne)
- **Stratégie de sauvegarde (prototype académique) :** exports manuels hebdomadaires via CLI Supabase, stockés sur le Google Drive du groupe. Aucune sauvegarde automatique activée (plan Free).
- **Stratégie recommandée en production réelle :** PITR 7 jours (~130 €/mois). Restauration à la seconde près via les fichiers WAL (Write-Ahead Log) enregistrés en continu vers AWS S3, même région Frankfurt.
- Supabase ne permet pas la suppression manuelle d'un enregistrement dans les sauvegardes.
- **Risque résiduel documenté et accepté :** une donnée supprimée de la base principale peut subsister dans les fichiers WAL et les sauvegardes physiques pendant toute la durée de rétention souscrite.

### Conditions de tolérance du risque résiduel (RGPD)

Le risque résiduel lié aux sauvegardes est légalement toléré sous trois conditions cumulatives :

1. La durée de rétention des sauvegardes est documentée et raisonnable (sept jours).
2. Ces sauvegardes sont sécurisées et réservées à la seule reprise sur incident. Aucun accès opérationnel aux données pseudonymisées n'est autorisé depuis les sauvegardes.
3. En cas de restauration depuis un point antérieur à une ou plusieurs demandes d'effacement au titre de l'article 17, toutes les suppressions et pseudonymisations réalisées entre ce point et la date de restauration doivent être ré-appliquées manuellement par l'équipe avant toute remise en service.

> **Point d'attention opérationnel :** une restauration d'urgence peut techniquement réintroduire les données d'un compte supprimé. La procédure de restauration doit systématiquement inclure une consultation de l'AuditLog (action `DELETION_COMPLETED`) afin d'identifier les comptes supprimés dans la fenêtre temporelle concernée.

- **Mesure compensatoire :** l'utilisateur est informé du risque résiduel de sept jours dans l'email de confirmation (voir section 5).

---

## 7. AuditLog — Actions tracées et durée de conservation

| Action | Déclencheur | Données loggées |
|---|---|---|
| `DELETION_REQUESTED` | Réception de la demande | actor_id (valide), timestamp |
| `DELETION_COMPLETED` | Fin de transaction réussie | actor_id (pseudonymisé), timestamp |
| `DELETION_FAILED` | ROLLBACK sur une étape | actor_id (pseudonymisé), étape échouée, timestamp — aucune donnée personnelle dans le contexte d'erreur |
| `EXPORT_REQUESTED` | Appel GET /api/user/export | actor_id (valide), timestamp |

**Durée de conservation des AuditLogs : 1 an**
Base légale : Art. 6.1.f RGPD — intérêt légitime (sécurité du traitement, Art. 32 RGPD) + Art. L34-1 (obligation sectorielle de conservation des données de connexion).

> Note sur la ligne `DELETION_REQUESTED` : cette entrée est insérée avec `actor_id` valide *avant* pseudonymisation. Elle doit être pseudonymisée immédiatement dans la même transaction, à l'étape 1 (calcul du token HMAC), et non après les suppressions. Sans cette précaution, l'`actor_id` réel subsiste dans les logs le temps de l'exécution de la transaction.

---

## 8. Droits des personnes — Tableau complet

| Droit | Article | Endpoint / Procédure | Statut |
|---|---|---|---|
| Droit d'accès | Art. 15 | `GET /api/user/me` (données directes) + procédure manuelle (logs) | À implémenter |
| Droit de rectification | Art. 16 | `PATCH /api/user/me` | À implémenter |
| Droit à l'effacement | Art. 17 | `DELETE /api/user/me` | Spécifié — section 4 |
| Droit à la limitation | Art. 18 | Flag `processing_restricted` en base — gel des traitements pendant contestation | À implémenter |
| Droit à la portabilité | Art. 20 | `GET /api/user/export` | Spécifié — section 4 |
| Droit d'opposition | Art. 21 | Applicable aux traitements fondés sur Art. 6.1.f : T05 (fidélité), T08 (AuditLog), T12 (performance préparateurs). Procédure : réception de l'opposition → vérification que le motif est légitime → suspension du traitement concerné → réponse sous 1 mois (Art. 12). Pour T12 : opposition d'un préparateur à la surveillance de ses performances → gel de la table Performance pour ce `preparer_id`, notification au Manager. Aucune opposition ne peut être opposée aux traitements fondés sur Art. 6.1.b ou 6.1.c. | ⚠️ À implémenter |
| Droit de réclamation CNIL | Art. 77 | Information à inclure dans la politique de confidentialité : https://www.cnil.fr/fr/plaintes | À mentionner |
