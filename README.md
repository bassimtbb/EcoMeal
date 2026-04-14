# EcoMeal — Workflow de Conception (Étapes 1–3)
> **Cours :** Conception d'Application & Architecture N-Tier — Prof. Addi  
> **Projet :** EcoMeal — Réduire le gaspillage alimentaire en connectant les commerces locaux avec les utilisateurs pour vendre les invendus à prix réduit.

---

## Table des matières

1. [Étape 1 — Liste brute des fonctionnalités](#étape-1--liste-brute-des-fonctionnalités)
2. [Étape 2 — Regroupement par domaine métier (DDD)](#étape-2--regroupement-par-domaine-métier-ddd)
3. [Étape 3 — Entités métier par module](#étape-3--entités-métier-par-module)

---

## Étape 1 — Liste brute des fonctionnalités

> **Méthode :** Liste exhaustive et non triée. Nous sommes en mode « fonctionnel pur » — *ce que* l'application fait, pas *comment* elle est construite. Trois perspectives : Consommateur, Partenaire Commercial et Administrateur Système.

### 👤 Perspective Consommateur

- Créer un compte utilisateur (email + mot de passe)
- Se connecter / Se déconnecter
- Réinitialiser le mot de passe oublié par email
- Consulter et modifier son profil (nom, adresse, préférences alimentaires)
- Parcourir les offres de repas disponibles sur une carte ou en liste
- Filtrer les offres par catégorie (boulangerie, restaurant, sushi…), distance, prix ou tag diététique (végan, sans gluten…)
- Rechercher des offres par nom d’établissement ou type de nourriture
- Voir une offre en détail (photos, description, quantité, horaire de retrait, prix original, prix réduit)
- Ajouter une offre au panier
- Retirer un article du panier
- Appliquer un code de réduction ou promo
- Passer une commande et payer en ligne
- Recevoir une confirmation de commande (in-app + email)
- Voir les commandes en cours et passées
- Annuler une commande (dans un délai imparti)
- Générer et afficher un code QR pour le retrait en magasin
- Noter et commenter un établissement après retrait
- Enregistrer des établissements favoris
- Recevoir des notifications (push/email) pour les nouvelles offres à proximité
- Gérer les préférences de notifications (fréquence, type)
- Signaler un problème avec une commande ou un établissement
- Demander un remboursement ou ouvrir un litige

### 🏪 Perspective Partenaire Commercial

- Créer un compte établissement (nom, adresse, SIRET, type)
- Télécharger le logo et des photos de l’établissement
- Soumettre le compte pour vérification/approbation par l’admin
- Se connecter au tableau de bord établissement
- Créer une nouvelle offre de repas (titre, description, photos, catégorie, quantité, créneau de retrait, prix original, prix réduit, date d’expiration)
- Modifier une offre existante
- Désactiver ou supprimer une offre
- Marquer une offre comme « épuisée »
- Voir le stock en temps réel pour les offres actives
- Voir les commandes reçues pour la journée
- Scanner le code QR d’un consommateur pour valider le retrait
- Marquer une commande comme retirée / terminée
- Voir les analyses de l’établissement (repas sauvés, chiffre d’affaires, note moyenne, statistiques de réduction du gaspillage)
- Recevoir les paiements pour les commandes terminées
- Voir l’historique des paiements et les factures
- Répondre aux avis des consommateurs
- Gérer les horaires d’ouverture
- Définir un rayon géographique ou un lieu de retrait
- Recevoir des notifications quand une offre est en stock bas
- Recevoir une notification pour une nouvelle commande

### 🛡️ Perspective Administrateur Système

- Voir et gérer tous les comptes utilisateurs (consommateurs + établissements)
- Approuver ou rejeter les demandes d’inscription d’établissements
- Suspendre ou bannir un compte (consommateur ou établissement)
- Modérer les signalements (avis, offres)
- Gérer les codes promo et promotions à l’échelle de la plateforme
- Voir le tableau de bord analytique de la plateforme (total repas sauvés, commandes, utilisateurs actifs, chiffre d’affaires)
- Gérer les catégories et les tags alimentaires
- Configurer les taux de commission par type d’établissement
- Traiter les demandes de remboursement et les litiges
- Envoyer des annonces ou notifications à tout le système
- Surveiller la santé du système (disponibilité API, taux d’erreur)
- Gérer le contenu (FAQ, conditions d’utilisation, politique de confidentialité)

---

## Étape 2 — Regroupement par domaine métier (DDD)

> **Principe appliqué :** Forte cohésion dans chaque module (toutes les fonctionnalités partagent la même raison de changer métier), Faible couplage entre modules (chaque module expose des interfaces propres, ne dépend pas des internes d’un autre).

Après analyse de la liste des fonctionnalités, six **Contextes Délimités (Bounded Contexts)** émergent naturellement :

| # | Module / Contexte délimité | Responsabilité principale | Raison de séparation |
|---|---|---|---|
| 1 | **UserIdentity** | Qui êtes-vous ? Authentification & profil | L’authentification/sécurité change indépendamment du reste |
| 2 | **Catalog** | Qu’est-ce qui est disponible ? Cycle de vie des offres | Les établissements publient/modifient des offres ; change souvent |
| 3 | **Order** | Qu’a-t-on acheté ? Cycle de vie des commandes & validation QR | Le paiement, la livraison et les litiges sont complexes et distincts |
| 4 | **Payment** | Comment l’argent est-il géré ? Paiements, reversements, remboursements | La logique financière est régulée et complètement isolée |
| 5 | **Notification** | Comment les utilisateurs sont-ils informés ? Alertes, emails, push | La logique des canaux de communication change indépendamment |
| 6 | **Administration** | Qui gère la plateforme ? Modération & gouvernance | Les workflows d’administration sont internes et jamais exposés aux utilisateurs finaux |

---

### Module 1 — `UserIdentity`

**Justification de cohésion :** Toutes les fonctionnalités ici ont une seule raison de changer — comment les utilisateurs sont authentifiés et comment leur profil personnel est géré. Envoyer un email à un utilisateur n’est *pas* le travail de ce module (c’est le rôle de `Notification`).

| Fonctionnalité | Perspective |
|---|---|
| Créer un compte consommateur | Consommateur |
| Se connecter / déconnecter | Consommateur |
| Réinitialiser le mot de passe | Consommateur |
| Consulter et modifier le profil | Consommateur |
| Gérer les préférences de notification | Consommateur |
| Créer un compte établissement | Commercial |
| Soumettre un établissement pour approbation admin | Commercial |
| Accéder au tableau de bord établissement | Commercial |
| Gérer les comptes utilisateurs/établissements | Admin |
| Approuver / rejeter / suspendre des comptes | Admin |

---

### Module 2 — `Catalog`

**Justification de cohésion :** Tout ce qui concerne *ce qui est disponible* vit ici. L’offre est l’artefact central : sa création, sa visibilité, son stock, sa découverte. La recherche géographique est également ici car elle interroge directement les offres. Les règles de prix (remises, commissions) sont déléguées à `Payment` et `Administration`.

| Fonctionnalité | Perspective |
|---|---|
| Créer une nouvelle offre de repas | Commercial |
| Modifier une offre existante | Commercial |
| Désactiver / supprimer une offre | Commercial |
| Marquer une offre comme épuisée | Commercial |
| Voir le stock en temps réel | Commercial |
| Gérer les catégories et tags alimentaires | Admin |
| Parcourir les offres (carte / liste) | Consommateur |
| Filtrer les offres (catégorie, distance, prix, tag diététique) | Consommateur |
| Rechercher des offres par nom ou type | Consommateur |
| Voir le détail d’une offre | Consommateur |
| Enregistrer des favoris | Consommateur |
| Télécharger logo et photos | Commercial |
| Gérer les horaires d’ouverture | Commercial |
| Définir le lieu de retrait / rayon | Commercial |

---

### Module 3 — `Order`

**Justification de cohésion :** Toutes les fonctionnalités partagent le même cycle de vie métier — du panier à la confirmation de retrait. La validation QR appartient ici car c’est une étape du flux de traitement de la commande, pas un événement de communication. Les litiges sont initiés ici mais *résolus* dans `Payment`.

| Fonctionnalité | Perspective |
|---|---|
| Ajouter / retirer du panier | Consommateur |
| Appliquer un code promo | Consommateur |
| Passer une commande | Consommateur |
| Recevoir la confirmation de commande | Consommateur |
| Voir les commandes en cours et passées | Consommateur |
| Annuler une commande | Consommateur |
| Générer un QR pour le retrait | Consommateur |
| Scanner le QR pour valider le retrait | Commercial |
| Marquer une commande comme retirée / terminée | Commercial |
| Voir les commandes du jour | Commercial |
| Signaler un problème avec une commande | Consommateur |
| Voir les commandes et litiges (admin) | Admin |

---

### Module 4 — `Payment`

**Justification de cohésion :** Tous les mouvements d’argent sont isolés ici. Ce module s’intègre avec des prestataires de paiement externes (Stripe, etc.) et est soumis à la réglementation financière — il doit pouvoir évoluer (ou être remplacé) sans toucher à `Order` ou `Catalog`.

| Fonctionnalité | Perspective |
|---|---|
| Payer en ligne lors de la commande | Consommateur |
| Demander un remboursement | Consommateur |
| Ouvrir et résoudre un litige | Consommateur |
| Recevoir les reversements pour commandes terminées | Commercial |
| Voir l’historique des paiements et factures | Commercial |
| Configurer les taux de commission | Admin |
| Traiter les remboursements et litiges | Admin |
| Gérer les codes promo plateforme | Admin |

---

### Module 5 — `Notification`

**Justification de cohésion :** Tous les événements de communication sont centralisés ici, quel que soit le canal (email, push, in-app). Les autres modules émettent des *Événements de Domaine* (ex. `OrderPlaced`, `OfferLowStock`) ; ce module les *écoute* et décide comment communiquer. C’est un pattern Observateur classique — l’émetteur ne connaît pas le canal de notification.

| Fonctionnalité | Perspective |
|---|---|
| Envoyer la confirmation de commande (in-app + email) | Système |
| Envoyer des alertes push/email pour nouvelles offres à proximité | Consommateur |
| Notifier l’établissement d’une nouvelle commande | Commercial |
| Notifier l’établissement en cas de stock bas | Commercial |
| Envoyer des annonces système | Admin |
| Gérer les préférences de notification (consommateur) | Consommateur |

---

### Module 6 — `Administration`

**Justification de cohésion :** La gouvernance, la modération et la configuration plateforme sont des opérations internes qu’aucun consommateur ou établissement ne peut accéder. Ces fonctionnalités changent au rythme des règles métier — indépendamment des fonctionnalités produit.

| Fonctionnalité | Perspective |
|---|---|
| Modérer les signalements (avis, offres) | Admin |
| Voir le tableau de bord analytique | Admin |
| Surveiller la santé système | Admin |
| Gérer le contenu (FAQ, CGU, politique confidentialité) | Admin |
| Voir le tableau de bord analytique établissement | Commercial (lecture seule) |
| Noter et commenter un établissement | Consommateur |
| Répondre aux avis consommateurs | Commercial |

---

### Module Dependency Map

```
                  ┌─────────────────┐
                  │  UserIdentity   │
                  └────────┬────────┘
                           │ (auth token / user identity)
          ┌────────────────┼──────────────────┐
          ▼                ▼                  ▼
   ┌─────────────┐  ┌─────────────┐  ┌───────────────┐
   │   Catalog   │  │    Order    │  │ Administration│
   └──────┬──────┘  └──────┬──────┘  └───────────────┘
          │                │
          │  (offer ref)   │  (order events)
          └────────┬───────┘
                   ▼
           ┌──────────────┐
           │   Payment    │
           └──────┬───────┘
                  │
                  │ (domain events: OrderPlaced,
                  │  PaymentConfirmed, OfferLowStock…)
                  ▼
          ┌───────────────┐
          │  Notification │
          └───────────────┘
```

> **Faible couplage appliqué :** `Notification` n’importe jamais directement depuis `Order` ou `Catalog`. Il réagit seulement aux Événements de Domaine publiés. `Catalog` ne sait jamais que `Payment` existe — il stocke simplement un prix. `Order` appelle `Payment` via une interface (Principe d’Inversion des Dépendances).
---


## Étape 3 — Entités métier par module

> **Vocabulaire :**
> - **Entité** — Possède une identité unique, est mutable dans le temps (ex. `Order` change de statut)
> - **Value Object** — Pas d’identité, immuable, défini uniquement par ses valeurs (ex. `Money(8.50, EUR)`)
> - **Agrégat (Aggregate Root)** — Point d’entrée unique vers un cluster d’entités ; les modules externes ne gardent qu’une référence vers l’ID de la racine

---

### Module 1 — `UserIdentity`

#### Entité : `User`
| Attribut | Type | Provenance |
|---|---|---|
| `id` | UUID | Généré par le système |
| `email` | String | Inscription |
| `passwordHash` | String | Inscription |
| `role` | Enum : `CONSUMER, BUSINESS, ADMIN` | Type d’inscription |
| `status` | Enum : `PENDING, ACTIVE, SUSPENDED` | Action admin |
| `createdAt` | DateTime | Système |

#### Entité : `ConsumerProfile` *(propriété de User)*
| Attribut | Type | Provenance |
|---|---|---|
| `userId` | UUID | FK vers User |
| `firstName` | String | Édition profil |
| `lastName` | String | Édition profil |
| `dietaryPreferences` | List\<Tag\> | Édition profil |
| `notificationPreferences` | NotificationPreferences | Paramètres |

#### Entité : `BusinessProfile` *(propriété de User)*
| Attribut | Type | Provenance |
|---|---|---|
| `userId` | UUID | FK vers User |
| `businessName` | String | Inscription |
| `registrationNumber` | String | Inscription (SIRET) |
| `businessType` | Enum : `RESTAURANT, BAKERY, SUSHI, ...` | Inscription |
| `logoUrl` | String | Téléchargement |
| `approvalStatus` | Enum : `PENDING, APPROVED, REJECTED` | Action admin |

#### Value Object : `NotificationPreferences`
| Attribut | Type |
|---|---|
| `emailEnabled` | Boolean |
| `pushEnabled` | Boolean |
| `nearbyOffersAlerts` | Boolean |
| `orderUpdatesEnabled` | Boolean |

---

### Module 2 — `Catalog`

#### Agrégat (Racine) : `Offer`
> Une `Offer` est l’artefact central de toute la plateforme. Son cycle de vie (DRAFT → ACTIVE → SOLD_OUT → EXPIRED) entraîne de nombreux événements aval.

| Attribut | Type | Provenance |
|---|---|---|
| `id` | UUID | Généré par le système |
| `businessId` | UUID | FK vers BusinessProfile |
| `title` | String | Créer offre |
| `description` | String | Créer offre |
| `photoUrls` | List\<String\> | Télécharger photos |
| `category` | Enum : `BAKERY, RESTAURANT, SUSHI, ...` | Filtrage par catégorie |
| `dietaryTags` | List\<Tag\> | Filtrage diététique |
| `originalPrice` | Money | Prix original |
| `discountedPrice` | Money | Prix de vente réduit |
| `totalQuantity` | Integer | Stock initial |
| `remainingQuantity` | Integer | Stock temps réel |
| `pickupWindow` | TimeWindow | Créneau retrait |
| `expiresAt` | DateTime | Expiration offre |
| `status` | Enum : `DRAFT, ACTIVE, SOLD_OUT, EXPIRED, DELETED` | Cycle de vie |
| `pickupLocation` | Address | Lieu de retrait |

#### Entité : `Business` *(projection lecture seule depuis UserIdentity)*
> `Catalog` ne conserve qu’une référence `businessId`. Les données complètes de l’établissement vivent dans `UserIdentity`. Cela applique le faible couplage.

#### Value Object : `Money`
| Attribut | Type |
|---|---|
| `amount` | BigDecimal |
| `currency` | Enum : `EUR, USD, ...` |

#### Value Object : `TimeWindow`
| Attribut | Type |
|---|---|
| `startTime` | LocalTime |
| `endTime` | LocalTime |
| `date` | LocalDate |

#### Value Object : `Address`
| Attribut | Type |
|---|---|
| `street` | String |
| `city` | String |
| `postalCode` | String |
| `latitude` | Double |
| `longitude` | Double |

#### Entité : `Category`
| Attribut | Type |
|---|---|
| `id` | UUID |
| `name` | String |
| `iconUrl` | String |

#### Entité : `Tag` *(diététique / label alimentaire)*
| Attribut | Type |
|---|---|
| `id` | UUID |
| `label` | String (ex. "vegan", "sans gluten") |

---

### Module 3 — `Order`

#### Agrégat (Racine) : `Order`
> L’agrégat `Order` encapsule le cycle de vie complet de l’achat. Le `Cart` est un état pré-commande transitoire — il devient une `Order` après confirmation du paiement. Aucun module externe ne modifie une `Order` directement ; tous les changements passent par la racine d’agrégat.

| Attribut | Type | Provenance |
|---|---|---|
| `id` | UUID | Généré par le système |
| `consumerId` | UUID | FK vers UserIdentity |
| `businessId` | UUID | FK vers UserIdentity |
| `items` | List\<OrderItem\> | Ajout au panier |
| `subtotal` | Money | Calculé |
| `discountApplied` | Money | Code promo |
| `totalAmount` | Money | Calculé |
| `promoCode` | String (nullable) | Appliquer promo |
| `status` | Enum : `PENDING, CONFIRMED, READY, PICKED_UP, CANCELLED, DISPUTED` | Cycle de vie |
| `pickupCode` | String (payload QR) | Générer QR |
| `placedAt` | DateTime | Système |
| `confirmedAt` | DateTime | Événement paiement |
| `pickedUpAt` | DateTime | Scan établissement |

#### Entité : `OrderItem` *(partie de l’agrégat Order)*
| Attribut | Type | Provenance |
|---|---|---|
| `offerId` | UUID | FK vers Catalog.Offer |
| `offerTitle` | String | Instantané à la commande |
| `unitPrice` | Money | Instantané à la commande |
| `quantity` | Integer | Ajout au panier |

> **Note de conception :** `offerTitle` et `unitPrice` sont *instantanés* (snapshot) à la création de la commande. C’est critique — si un établissement modifie ou supprime une offre, les commandes historiques doivent rester intactes. C’est le patron **Instantané Immuable (Immutable Snapshot)**.

#### Entité : `Dispute`
| Attribut | Type | Provenance |
|---|---|---|
| `id` | UUID | Généré par le système |
| `orderId` | UUID | FK vers Order |
| `consumerId` | UUID | FK vers UserIdentity |
| `reason` | String | Signaler problème |
| `status` | Enum : `OPEN, UNDER_REVIEW, RESOLVED, REJECTED` | Action admin |
| `resolution` | String (nullable) | Action admin |
| `createdAt` | DateTime | Système |

---

### Module 4 — `Payment`

#### Agrégat (Racine) : `Payment`
> Tous les enregistrements financiers sont immuables une fois créés (append-only). Un `Refund` est un nouvel enregistrement, jamais une mutation du `Payment` original. Cela respecte les exigences d’audit financier et est une base pour l’Event Sourcing si nécessaire.

| Attribut | Type | Provenance |
|---|---|---|
| `id` | UUID | Généré par le système |
| `orderId` | UUID | FK vers Order |
| `consumerId` | UUID | FK vers UserIdentity |
| `amount` | Money | Total commande |
| `status` | Enum : `PENDING, SUCCEEDED, FAILED, REFUNDED` | Prestataire paiement |
| `externalTransactionId` | String | Stripe / prestataire |
| `createdAt` | DateTime | Système |

#### Entité : `Payout`
| Attribut | Type | Provenance |
|---|---|---|
| `id` | UUID | Généré par le système |
| `businessId` | UUID | FK vers UserIdentity |
| `amount` | Money | Commandes terminées sur la période |
| `commissionDeducted` | Money | Configuration commission |
| `status` | Enum : `SCHEDULED, PROCESSED, FAILED` | Tâche batch |
| `periodStart` | LocalDate | Calendrier reversement |
| `periodEnd` | LocalDate | Calendrier reversement |

#### Entité : `Refund`
| Attribut | Type | Provenance |
|---|---|---|
| `id` | UUID | Généré par le système |
| `paymentId` | UUID | FK vers Payment |
| `disputeId` | UUID (nullable) | FK vers Dispute |
| `amount` | Money | Montant remboursé |
| `reason` | String | Admin / consommateur |
| `status` | Enum : `PENDING, PROCESSED, FAILED` | Prestataire |

#### Value Object : `CommissionRate`
| Attribut | Type |
|---|---|
| `businessType` | Enum |
| `rate` | BigDecimal (ex. 0.15 = 15%) |

---

### Module 5 — `Notification`

> Ce module ne contient **aucune entité métier** au sens DDD — il est purement réactif. Il s’abonne aux Événements de Domaine émis par les autres modules et les transforme en communications.

#### Entité : `NotificationLog` *(piste d’audit)*
| Attribut | Type | Provenance |
|---|---|---|
| `id` | UUID | Généré par le système |
| `recipientId` | UUID | Utilisateur cible |
| `channel` | Enum : `EMAIL, PUSH, IN_APP` | Préférences |
| `type` | Enum : `ORDER_CONFIRMED, NEW_OFFER, LOW_STOCK, ...` | Événement Domaine |
| `payload` | JSON | Données de l’événement |
| `sentAt` | DateTime | Système |
| `status` | Enum : `SENT, FAILED, SKIPPED` | Résultat livraison |

#### Événements de Domaine consommés (depuis autres modules) :
| Événement | Émis par | Action |
|---|---|---|
| `OrderPlaced` | Order | Notifier consommateur (confirmation) + établissement (nouvelle commande) |
| `OrderPickedUp` | Order | Notifier consommateur (reçu) |
| `OrderCancelled` | Order | Notifier établissement |
| `OfferLowStock` | Catalog | Notifier établissement |
| `BusinessApproved` | UserIdentity | Notifier établissement |
| `DisputeResolved` | Payment | Notifier consommateur |

---

### Module 6 — `Administration`

#### Entité : `Review`
| Attribut | Type | Provenance |
|---|---|---|
| `id` | UUID | Généré par le système |
| `consumerId` | UUID | FK vers UserIdentity |
| `businessId` | UUID | FK vers UserIdentity |
| `orderId` | UUID | FK vers Order |
| `rating` | Integer (1–5) | Noter établissement |
| `comment` | String | Écrire avis |
| `businessReply` | String (nullable) | Réponse établissement |
| `status` | Enum : `VISIBLE, FLAGGED, REMOVED` | Modération |
| `createdAt` | DateTime | Système |

#### Entité : `PromoCode`
| Attribut | Type | Provenance |
|---|---|---|
| `id` | UUID | Généré par le système |
| `code` | String | Création admin |
| `discountType` | Enum : `PERCENTAGE, FIXED` | Règle |
| `discountValue` | BigDecimal | Règle |
| `validFrom` | DateTime | Règle |
| `validUntil` | DateTime | Règle |
| `maxUses` | Integer | Règle |
| `currentUses` | Integer | Calculé |
| `status` | Enum : `ACTIVE, EXPIRED, DISABLED` | Cycle de vie |

#### Entité : `ModerationReport`
| Attribut | Type | Provenance |
|---|---|---|
| `id` | UUID | Généré par le système |
| `reportedBy` | UUID | FK vers UserIdentity |
| `targetType` | Enum : `REVIEW, OFFER, USER` | Signalement |
| `targetId` | UUID | Entité ciblée |
| `reason` | String | Signalement |
| `status` | Enum : `PENDING, REVIEWED, ACTIONED` | Admin |

---

## Synthèse de conception — Checklist des 4 questions

Comme enseigné par le Prof. Addi, chaque fonctionnalité doit répondre à ces quatre questions :

| Module | Quel module ? | Quelles données ? | Quelle logique ? | Quelle interaction ? |
|---|---|---|---|---|
| UserIdentity | Cycle de vie utilisateur | User, ConsumerProfile, BusinessProfile | Hachage mot de passe, attribution rôles, flux d’approbation | REST (POST /register, POST /login), dashboard admin |
| Catalog | Disponibilité des offres | Offer, Category, Tag | Décrément stock, vérification expiration, recherche géographique | REST (GET /offers, POST /offers), Cron (expiration offres) |
| Order | Flux d’achat | Order, OrderItem, Dispute | Validation promo, génération QR, instantané des prix | REST (POST /orders, PATCH /orders/{id}/cancel), événement scan QR |
| Payment | Mouvements d’argent | Payment, Payout, Refund | Calcul commission, éligibilité remboursement | Webhook Stripe, REST (GET /payouts), Cron (reversement hebdomadaire) |
| Notification | Communication | NotificationLog | Routage par canal, filtrage par préférences | Écoute d’événements (Kafka / Observer), fournisseurs Email/Push |
| Administration | Gouvernance plateforme | Review, PromoCode, ModerationReport | Règles de modération, validité promo | API REST Admin, dashboard interne |

---

*Document réalisé dans le cadre du TP EcoMeal — Workflow de conception Étapes 1–3.*  
*Prochaine étape : Étape 4 — Déduction des composants techniques et rédaction des ADRs.*
