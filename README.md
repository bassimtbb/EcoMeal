# EcoMeal — Workflow de Conception (Étapes 1–5)

> **Projet :** EcoMeal — Réduire le gaspillage alimentaire en connectant des commerces locaux avec des utilisateurs pour vendre les invendus à prix réduit.

---

## Table des matières

1. [Étape 1 — Liste brute des fonctionnalités](#étape-1--liste-brute-des-fonctionnalités)
2. [Étape 2 — Regroupement par domaine métier (DDD)](#étape-2--regroupement-par-domaine-métier-ddd)
3. [Étape 3 — Entités métier par module](#étape-3--entités-métier-par-module)
4. [Étape 4 — Dérivation des composants techniques (N-Tier)](#étape-4--dérivation-des-composants-techniques-n-tier)
5. [Étape 5 — Patterns et Modèle de données](#étape-5--patterns-et-modèle-de-données)

---

## Étape 1 — Liste brute des fonctionnalités

> **Méthode :** Liste exhaustive, non triée. Mode « fonctionnel pur » — *ce que* fait l'application, pas *comment* elle est construite. Trois perspectives : Consommateur, Partenaire commercial, et Administrateur système.

### 👤 Perspective Consommateur

- Créer un nouveau compte utilisateur (email + mot de passe)
- Se connecter / Se déconnecter
- Réinitialiser un mot de passe oublié par email
- Consulter et modifier son profil personnel (nom, adresse, préférences alimentaires)
- Parcourir les offres disponibles sur une carte ou en vue liste
- Filtrer les offres par catégorie (boulangerie, restaurant, sushi…), distance, prix ou tag alimentaire (vegan, sans gluten…)
- Rechercher des offres par nom de commerce ou type d'aliment
- Consulter le détail d'une offre (photos, description, quantité, créneau de retrait, prix original, prix réduit)
- Ajouter une offre au panier
- Retirer un article du panier
- Appliquer un code de réduction ou un code promo
- Passer une commande et payer en ligne
- Recevoir une confirmation de commande (in-app + email)
- Consulter les commandes en cours et passées
- Annuler une commande (dans un délai limité)
- Générer et afficher un QR code pour le retrait en magasin
- Noter et laisser un avis sur un commerce après retrait
- Sauvegarder ses commerces favoris
- Recevoir des notifications push/email pour les nouvelles offres à proximité
- Gérer les préférences de notification (fréquence, type)
- Signaler un problème avec une commande ou un commerce
- Demander un remboursement ou ouvrir un litige

### 🏪 Perspective Partenaire Commercial

- Créer un compte professionnel (nom, adresse, numéro SIRET, type)
- Télécharger le logo et les photos du commerce
- Soumettre le compte pour vérification/approbation par l'administrateur
- Se connecter au tableau de bord professionnel
- Créer une nouvelle offre de repas (titre, description, photos, catégorie, quantité, créneau de retrait, prix original, prix réduit, heure d'expiration)
- Modifier une offre existante
- Désactiver ou supprimer une offre
- Marquer une offre comme « épuisée »
- Consulter le stock en temps réel pour les offres actives
- Consulter les commandes du jour
- Scanner le QR code d'un consommateur pour valider le retrait
- Marquer une commande comme retirée / terminée
- Consulter les statistiques du commerce (nombre de repas sauvés, chiffre d'affaires, note moyenne, statistiques de réduction des déchets)
- Recevoir des paiements pour les commandes terminées
- Consulter l'historique des paiements et les factures
- Répondre aux avis des consommateurs
- Gérer les horaires d'ouverture du commerce
- Définir le rayon de service géographique ou le lieu de retrait
- Recevoir des notifications lorsqu'une offre est presque épuisée
- Recevoir une notification pour chaque nouvelle commande

### 🛡️ Perspective Administrateur Système

- Consulter et gérer tous les comptes utilisateurs (consommateurs + commerces)
- Approuver ou rejeter les demandes d'inscription des commerces
- Suspendre ou bannir un compte (consommateur ou commerce)
- Modérer les contenus signalés (avis, offres)
- Gérer les codes de réduction et promotions à l'échelle de la plateforme
- Consulter le tableau de bord analytique de la plateforme (total de repas sauvés, total des commandes, utilisateurs actifs, chiffre d'affaires)
- Gérer les catégories et les tags alimentaires
- Configurer les taux de commission par type de commerce
- Traiter les demandes de remboursement et les litiges
- Envoyer des annonces ou notifications à l'ensemble de la plateforme
- Surveiller l'état du système (disponibilité de l'API, taux d'erreurs)
- Gérer le contenu (FAQ, conditions d'utilisation, politique de confidentialité)

---

## Étape 2 — Regroupement par domaine métier (DDD)

> **Principe appliqué :** Forte cohésion au sein de chaque module (toutes les fonctionnalités partagent la même raison métier d'évoluer), faible couplage entre les modules (chaque module expose des interfaces propres et ne dépend pas des détails internes d'un autre).

Six **Contextes Bornés** (Bounded Contexts) émergent naturellement :

| # | Module / Contexte Borné | Responsabilité principale | Raison clé de la séparation |
|---|---|---|---|
| 1 | **UserIdentity** | Qui êtes-vous ? Authentification et profil | La sécurité/auth évolue indépendamment de tout le reste |
| 2 | **Catalog** | Qu'est-ce qui est disponible ? Gestion du cycle de vie des offres | Les commerces publient/modifient des offres ; change fréquemment |
| 3 | **Order** | Qu'a-t-on acheté ? Cycle de vie des commandes et validation QR | Les règles de paiement, de fulfillment et de litige sont complexes et distinctes |
| 4 | **Payment** | Comment l'argent est géré ? Paiements, virements, remboursements | La logique financière est réglementée et complètement isolée |
| 5 | **Notification** | Comment les utilisateurs sont informés ? Alertes, emails, push | La logique des canaux de communication évolue indépendamment |
| 6 | **Administration** | Qui gère la plateforme ? Modération et gouvernance | Les flux de travail admin sont internes et jamais exposés aux utilisateurs finaux |

### Carte des dépendances entre modules

```
                  ┌─────────────────┐
                  │  UserIdentity   │
                  └────────┬────────┘
                           │ (token d'auth / identité utilisateur)
          ┌────────────────┼──────────────────┐
          ▼                ▼                  ▼
   ┌─────────────┐  ┌─────────────┐  ┌──────────────┐
   │   Catalog   │  │    Order    │  │Administration│
   └──────┬──────┘  └──────┬──────┘  └──────────────┘
          │                │
          │  (réf. offre)  │  (événements commande)
          └────────┬───────┘
                   ▼
           ┌──────────────┐
           │   Payment    │
           └──────┬───────┘
                  │
                  │ (événements domaine : OrderPlaced,
                  │  PaymentConfirmed, OfferLowStock…)
                  ▼
          ┌───────────────┐
          │  Notification │
          └───────────────┘
```

> **Faible couplage garanti :** `Notification` ne connaît jamais directement `Order` ou `Catalog` — il réagit uniquement aux Événements Domaine. `Catalog` ne sait pas que `Payment` existe. `Order` appelle `Payment` via une interface (Principe d'Inversion de Dépendance).

---

## Étape 3 — Entités métier par module

> **Vocabulaire :**
> - **Entité** — Possède une identité unique, est mutable dans le temps
> - **Objet Valeur (Value Object)** — Pas d'identité, immuable, défini entièrement par ses valeurs
> - **Racine d'Agrégat** — Point d'entrée unique d'un groupe d'entités liées

### Module 1 — `UserIdentity`

#### Entité : `User`
| Attribut | Type | Description |
|---|---|---|
| `id` | UUID | Généré par le système |
| `email` | String | Adresse email unique |
| `passwordHash` | String | Mot de passe haché (bcrypt) |
| `role` | Enum : `CONSUMER, BUSINESS, ADMIN` | Type d'inscription |
| `status` | Enum : `PENDING, ACTIVE, SUSPENDED` | Approbation admin |
| `createdAt` | DateTime | Timestamp de création |

#### Entité : `ConsumerProfile` *(appartenant à User)*
| Attribut | Type | Description |
|---|---|---|
| `userId` | UUID | FK vers User |
| `firstName` | String | Prénom |
| `lastName` | String | Nom de famille |
| `dietaryPreferences` | List\<Tag\> | Préférences alimentaires |
| `notificationPreferences` | NotificationPreferences | Paramètres de notification |

#### Entité : `BusinessProfile` *(appartenant à User)*
| Attribut | Type | Description |
|---|---|---|
| `userId` | UUID | FK vers User |
| `businessName` | String | Nom du commerce |
| `registrationNumber` | String | Numéro SIRET |
| `businessType` | Enum : `RESTAURANT, BAKERY, SUSHI, ...` | Type de commerce |
| `logoUrl` | String | URL du logo |
| `approvalStatus` | Enum : `PENDING, APPROVED, REJECTED` | Statut d'approbation admin |

#### Objet Valeur : `NotificationPreferences`
| Attribut | Type |
|---|---|
| `emailEnabled` | Boolean |
| `pushEnabled` | Boolean |
| `nearbyOffersAlerts` | Boolean |
| `orderUpdatesEnabled` | Boolean |

---

### Module 2 — `Catalog`

#### Racine d'Agrégat : `Offer`
> Cycle de vie : `DRAFT → ACTIVE → SOLD_OUT → EXPIRED`

| Attribut | Type | Description |
|---|---|---|
| `id` | UUID | Généré par le système |
| `businessId` | UUID | FK vers BusinessProfile |
| `title` | String | Titre de l'offre |
| `description` | String | Description détaillée |
| `photoUrls` | List\<String\> | URLs des photos |
| `category` | Enum : `BAKERY, RESTAURANT, SUSHI, ...` | Catégorie |
| `dietaryTags` | List\<Tag\> | Tags alimentaires |
| `originalPrice` | Money | Prix original |
| `discountedPrice` | Money | Prix réduit |
| `totalQuantity` | Integer | Stock initial |
| `remainingQuantity` | Integer | Stock en temps réel |
| `pickupWindow` | TimeWindow | Créneau de retrait |
| `expiresAt` | DateTime | Date/heure d'expiration |
| `status` | Enum : `DRAFT, ACTIVE, SOLD_OUT, EXPIRED, DELETED` | Statut du cycle de vie |
| `pickupLocation` | Address | Adresse de retrait |

#### Objets Valeur du module Catalog

**`Money`**
| Attribut | Type |
|---|---|
| `amount` | BigDecimal |
| `currency` | Enum : `EUR, USD, ...` |

**`TimeWindow`**
| Attribut | Type |
|---|---|
| `startTime` | LocalTime |
| `endTime` | LocalTime |
| `date` | LocalDate |

**`Address`**
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

#### Entité : `Tag`
| Attribut | Type |
|---|---|
| `id` | UUID |
| `label` | String (ex. : « vegan », « sans gluten ») |

---

### Module 3 — `Order`

#### Racine d'Agrégat : `Order`

| Attribut | Type | Description |
|---|---|---|
| `id` | UUID | Généré par le système |
| `consumerId` | UUID | FK vers UserIdentity |
| `businessId` | UUID | FK vers UserIdentity |
| `items` | List\<OrderItem\> | Articles commandés |
| `subtotal` | Money | Sous-total calculé |
| `discountApplied` | Money | Remise appliquée |
| `totalAmount` | Money | Total final |
| `promoCode` | String (nullable) | Code promo utilisé |
| `status` | Enum : `PENDING, CONFIRMED, READY, PICKED_UP, CANCELLED, DISPUTED` | Statut |
| `pickupCode` | String | Contenu du QR code |
| `placedAt` | DateTime | Timestamp de la commande |
| `confirmedAt` | DateTime | Timestamp de confirmation paiement |
| `pickedUpAt` | DateTime | Timestamp de retrait |

#### Entité : `OrderItem` *(partie de l'agrégat Order)*
| Attribut | Type | Description |
|---|---|---|
| `offerId` | UUID | FK vers Catalog.Offer |
| `offerTitle` | String | **Snapshot** du titre au moment de la commande |
| `unitPrice` | Money | **Snapshot** du prix au moment de la commande |
| `quantity` | Integer | Quantité commandée |

> **Pattern Snapshot Immuable :** `offerTitle` et `unitPrice` sont capturés au moment de la commande. Si un commerce modifie ou supprime une offre ultérieurement, les commandes historiques restent intactes.

#### Entité : `Dispute`
| Attribut | Type | Description |
|---|---|---|
| `id` | UUID | Généré par le système |
| `orderId` | UUID | FK vers Order |
| `consumerId` | UUID | FK vers UserIdentity |
| `reason` | String | Motif du litige |
| `status` | Enum : `OPEN, UNDER_REVIEW, RESOLVED, REJECTED` | Statut |
| `resolution` | String (nullable) | Décision de l'admin |
| `createdAt` | DateTime | Timestamp |

---

### Module 4 — `Payment`

#### Racine d'Agrégat : `Payment`
> Tous les enregistrements financiers sont **immuables** (append-only). Un `Refund` est un nouvel enregistrement, jamais une mutation du `Payment` original.

| Attribut | Type | Description |
|---|---|---|
| `id` | UUID | Généré par le système |
| `orderId` | UUID | FK vers Order |
| `consumerId` | UUID | FK vers UserIdentity |
| `amount` | Money | Montant total |
| `status` | Enum : `PENDING, SUCCEEDED, FAILED, REFUNDED` | Statut |
| `externalTransactionId` | String | ID Stripe / prestataire |
| `createdAt` | DateTime | Timestamp |

#### Entité : `Payout`
| Attribut | Type | Description |
|---|---|---|
| `id` | UUID | Généré par le système |
| `businessId` | UUID | FK vers UserIdentity |
| `amount` | Money | Montant à virer |
| `commissionDeducted` | Money | Commission déduite |
| `status` | Enum : `SCHEDULED, PROCESSED, FAILED` | Statut |
| `periodStart` | LocalDate | Début de période |
| `periodEnd` | LocalDate | Fin de période |

#### Entité : `Refund`
| Attribut | Type | Description |
|---|---|---|
| `id` | UUID | Généré par le système |
| `paymentId` | UUID | FK vers Payment |
| `disputeId` | UUID (nullable) | FK vers Dispute |
| `amount` | Money | Montant remboursé |
| `reason` | String | Motif |
| `status` | Enum : `PENDING, PROCESSED, FAILED` | Statut |

#### Objet Valeur : `CommissionRate`
| Attribut | Type |
|---|---|
| `businessType` | Enum |
| `rate` | BigDecimal (ex. : 0.15 = 15%) |

---

### Module 5 — `Notification`

> Module purement réactif. Il s'abonne aux Événements Domaine et les convertit en communications.

#### Entité : `NotificationLog` *(piste d'audit)*
| Attribut | Type | Description |
|---|---|---|
| `id` | UUID | Généré par le système |
| `recipientId` | UUID | Utilisateur cible |
| `channel` | Enum : `EMAIL, PUSH, IN_APP` | Canal de communication |
| `type` | Enum : `ORDER_CONFIRMED, NEW_OFFER, LOW_STOCK, ...` | Type d'événement |
| `payload` | JSON | Données de l'événement |
| `sentAt` | DateTime | Timestamp d'envoi |
| `status` | Enum : `SENT, FAILED, SKIPPED` | Résultat |

#### Événements Domaine consommés :
| Événement | Émis par | Action déclenchée |
|---|---|---|
| `OrderPlaced` | Order | Confirmation consommateur + alerte commerce |
| `OrderPickedUp` | Order | Reçu consommateur |
| `OrderCancelled` | Order | Notification commerce |
| `OfferLowStock` | Catalog | Alerte stock faible au commerce |
| `BusinessApproved` | UserIdentity | Notification d'approbation au commerce |
| `DisputeResolved` | Payment | Notification de résolution au consommateur |

---

### Module 6 — `Administration`

#### Entité : `Review`
| Attribut | Type | Description |
|---|---|---|
| `id` | UUID | Généré par le système |
| `consumerId` | UUID | FK vers UserIdentity |
| `businessId` | UUID | FK vers UserIdentity |
| `orderId` | UUID | FK vers Order |
| `rating` | Integer (1–5) | Note |
| `comment` | String | Commentaire |
| `businessReply` | String (nullable) | Réponse du commerce |
| `status` | Enum : `VISIBLE, FLAGGED, REMOVED` | Statut de modération |
| `createdAt` | DateTime | Timestamp |

#### Entité : `PromoCode`
| Attribut | Type | Description |
|---|---|---|
| `id` | UUID | Généré par le système |
| `code` | String | Code alphanumérique |
| `discountType` | Enum : `PERCENTAGE, FIXED` | Type de réduction |
| `discountValue` | BigDecimal | Valeur de la réduction |
| `validFrom` | DateTime | Début de validité |
| `validUntil` | DateTime | Fin de validité |
| `maxUses` | Integer | Limite d'utilisation |
| `currentUses` | Integer | Utilisations actuelles |
| `status` | Enum : `ACTIVE, EXPIRED, DISABLED` | Statut |

#### Entité : `ModerationReport`
| Attribut | Type | Description |
|---|---|---|
| `id` | UUID | Généré par le système |
| `reportedBy` | UUID | FK vers UserIdentity |
| `targetType` | Enum : `REVIEW, OFFER, USER` | Type de cible |
| `targetId` | UUID | Entité signalée |
| `reason` | String | Motif du signalement |
| `status` | Enum : `PENDING, REVIEWED, ACTIONED` | Statut admin |

---

## Étape 4 — Dérivation des composants techniques (N-Tier)

> Pour chaque fonctionnalité clé, on détaille la chaîne verticale complète :
> **Controller REST → Service (Logique métier) → Repository (Persistance/DB)**

---

### Fonctionnalité 1 — Création d'une offre (`POST /api/catalog/offers`)

```
┌─────────────────────────────────────────────────────────────────────┐
│  COUCHE PRÉSENTATION — Controller REST                              │
│  POST /api/catalog/offers                                           │
│  @RestController → OfferController                                  │
│                                                                     │
│  - Valide le JWT (role = BUSINESS)                                  │
│  - Désérialise le payload JSON → CreateOfferRequest DTO             │
│  - Délègue à OfferService.createOffer(businessId, dto)              │
│  - Retourne HTTP 201 Created + OfferResponseDTO                     │
└──────────────────────────┬──────────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────────────┐
│  COUCHE MÉTIER — Service                                            │
│  OfferService                                                       │
│                                                                     │
│  - Vérifie que le businessId est APPROVED (appel UserIdentity)      │
│  - Valide les règles métier :                                       │
│    • discountedPrice < originalPrice                                │
│    • pickupWindow.endTime > maintenant                              │
│    • totalQuantity > 0                                              │
│  - Construit l'entité Offer avec status = DRAFT                     │
│  - Appelle OfferRepository.save(offer)                              │
│  - Publie l'événement domaine OfferCreated (→ Notification)         │
│  - Planifie un Job cron pour l'expiration automatique               │
└──────────────────────────┬──────────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────────────┐
│  COUCHE PERSISTANCE — Repository                                    │
│  OfferRepository (interface JPA/Hibernate)                          │
│                                                                     │
│  Méthodes :                                                         │
│  - save(Offer offer) → Offer                                        │
│  - findByBusinessId(UUID businessId) → List<Offer>                  │
│  - findActiveOffersNear(Double lat, Double lon, Double radius)       │
│    (requête spatiale PostGIS / POINT géographique)                  │
│  - updateStatus(UUID offerId, OfferStatus status)                   │
│                                                                     │
│  DB : TABLE offers (PostgreSQL)                                     │
│  Index : idx_offers_status, idx_offers_location (GIST)              │
└─────────────────────────────────────────────────────────────────────┘
```

---

### Fonctionnalité 2 — Paiement d'une commande (`POST /api/orders/{id}/pay`)

```
┌─────────────────────────────────────────────────────────────────────┐
│  COUCHE PRÉSENTATION — Controller REST                              │
│  POST /api/orders/{orderId}/pay                                     │
│  @RestController → OrderController                                  │
│                                                                     │
│  - Valide le JWT (role = CONSUMER, consumerId == order.consumerId)  │
│  - Extrait orderId depuis le path param                             │
│  - Délègue à PaymentOrchestrationService.initiatePayment(orderId)   │
│  - Retourne HTTP 202 Accepted + PaymentIntentDTO (clientSecret)     │
└──────────────────────────┬──────────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────────────┐
│  COUCHE MÉTIER — Service (Transaction @Transactional)               │
│  PaymentOrchestrationService                                        │
│                                                                     │
│  1. Charge l'Order via OrderRepository (vérifie status = PENDING)   │
│  2. Applique PromoCodeService.validate(promoCode) si présent        │
│  3. Calcule commissionRate via CommissionRateRepository             │
│  4. Appelle StripeGateway.createPaymentIntent(amount, currency)     │
│     (interface PaymentGateway — pattern Strategy / DIP)             │
│  5. Crée l'entité Payment (status = PENDING)                        │
│  6. Sauvegarde via PaymentRepository.save(payment)                  │
│  7. Met à jour Order.status = CONFIRMED via OrderRepository         │
│  8. Décrémente remainingQuantity dans Catalog via OfferRepository   │
│  9. Publie PaymentConfirmed → Notification émet confirmation        │
└──────────────────────────┬──────────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────────────┐
│  COUCHE PERSISTANCE — Repositories                                  │
│                                                                     │
│  OrderRepository                                                    │
│  - findById(UUID) → Order                                           │
│  - updateStatus(UUID, OrderStatus)                                  │
│                                                                     │
│  PaymentRepository                                                  │
│  - save(Payment) → Payment                                          │
│  - findByOrderId(UUID) → Optional<Payment>                          │
│                                                                     │
│  OfferRepository                                                    │
│  - decrementRemainingQuantity(UUID offerId, int qty)                │
│  (requête atomique avec SELECT FOR UPDATE pour éviter race condition)│
│                                                                     │
│  DB : TABLEs orders, payments, offers (PostgreSQL, transactions ACID)│
└─────────────────────────────────────────────────────────────────────┘
```

---

### Fonctionnalité 3 — Collecte QR (`POST /api/orders/{id}/validate-pickup`)

```
┌─────────────────────────────────────────────────────────────────────┐
│  COUCHE PRÉSENTATION — Controller REST                              │
│  POST /api/orders/{orderId}/validate-pickup                         │
│  Body : { "pickupCode": "ABC123XY" }                                │
│  @RestController → OrderController                                  │
│                                                                     │
│  - Valide le JWT (role = BUSINESS)                                  │
│  - Extrait orderId et pickupCode                                    │
│  - Délègue à OrderFulfillmentService.validatePickup(...)            │
│  - Retourne HTTP 200 OK + PickupConfirmationDTO                     │
└──────────────────────────┬──────────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────────────┐
│  COUCHE MÉTIER — Service                                            │
│  OrderFulfillmentService                                            │
│                                                                     │
│  1. Charge l'Order via OrderRepository                              │
│  2. Vérifie Order.businessId == JWT.businessId (autorisation)       │
│  3. Vérifie Order.status == CONFIRMED (sinon lève une exception)     │
│  4. Compare pickupCode == Order.pickupCode (comparaison sécurisée)  │
│  5. Met à jour Order.status = PICKED_UP, Order.pickedUpAt = now()   │
│  6. Sauvegarde via OrderRepository.save(order)                      │
│  7. Déclenche PayoutScheduler pour planifier le virement commerce   │
│  8. Publie OrderPickedUp → Notification (reçu consommateur)         │
└──────────────────────────┬──────────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────────────┐
│  COUCHE PERSISTANCE — Repository                                    │
│  OrderRepository                                                    │
│                                                                     │
│  - findByIdAndBusinessId(UUID orderId, UUID businessId) → Order     │
│  - save(Order order) → Order                                        │
│                                                                     │
│  PayoutRepository                                                   │
│  - scheduleOrUpdate(UUID businessId, Money amount, LocalDate period)│
│                                                                     │
│  DB : TABLE orders, payouts (PostgreSQL)                            │
│  Audit : Chaque changement de statut est loggé dans order_events    │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Étape 5 — Patterns et Modèle de données

### 5.1 — Patterns architecturaux induits

| Pattern | Module concerné | Application dans EcoMeal |
|---|---|---|
| **Observer / Event-Driven** | `Notification` | Le module s'abonne aux événements domaine (`OrderPlaced`, `OfferLowStock`, etc.) sans couplage direct. Implémenté via Kafka ou un bus d'événements interne. |
| **Strategy** | `Payment` | L'interface `PaymentGateway` peut être implémentée par `StripeGateway`, `PayPalGateway`, etc. — le Service ne dépend que de l'interface. |
| **Snapshot Immuable** | `Order` | `OrderItem.offerTitle` et `OrderItem.unitPrice` capturés au moment de la commande ; isolation du catalogue ultérieur. |
| **Aggregate Root** | `Order`, `Offer`, `Payment` | Les entités enfants (OrderItem, Dispute) ne sont accessibles qu'à travers leur racine d'agrégat. |
| **Repository** | Tous les modules | Abstraction de la persistance — le Service ne connaît pas le SQL ; facilite les tests unitaires avec des mocks. |
| **Append-Only / Event Log** | `Payment` | Les enregistrements financiers ne sont jamais mutés : un remboursement crée un `Refund`, pas une modification du `Payment`. Fondement pour l'Event Sourcing. |
| **Cron Job / Scheduler** | `Catalog`, `Payment` | Expiration automatique des offres ; virements hebdomadaires aux commerces. |
| **Anti-Corruption Layer** | `Payment` → Stripe | Le `StripeGateway` traduit le modèle Stripe vers le modèle domaine interne, isolant le domaine des API tierces. |

---

### 5.2 — Schéma SQL — Relations complexes et snapshots

#### Table de jonction : offres et tags alimentaires

```sql
-- Tags alimentaires (dictionnaire)
CREATE TABLE dietary_tags (
    id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    label       VARCHAR(100) NOT NULL UNIQUE  -- ex: 'vegan', 'sans-gluten'
);

-- Offres
CREATE TABLE offers (
    id                  UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    business_id         UUID NOT NULL,
    title               VARCHAR(255) NOT NULL,
    description         TEXT,
    category            VARCHAR(50) NOT NULL,
    original_price      NUMERIC(10,2) NOT NULL,
    discounted_price    NUMERIC(10,2) NOT NULL,
    currency            CHAR(3) NOT NULL DEFAULT 'EUR',
    total_quantity      INT NOT NULL CHECK (total_quantity > 0),
    remaining_quantity  INT NOT NULL,
    pickup_date         DATE NOT NULL,
    pickup_start        TIME NOT NULL,
    pickup_end          TIME NOT NULL,
    expires_at          TIMESTAMP WITH TIME ZONE NOT NULL,
    status              VARCHAR(20) NOT NULL DEFAULT 'DRAFT'
                            CHECK (status IN ('DRAFT','ACTIVE','SOLD_OUT','EXPIRED','DELETED')),
    pickup_street       VARCHAR(255),
    pickup_city         VARCHAR(100),
    pickup_postal_code  VARCHAR(20),
    pickup_latitude     DOUBLE PRECISION,
    pickup_longitude    DOUBLE PRECISION,
    created_at          TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    CONSTRAINT chk_price CHECK (discounted_price < original_price),
    CONSTRAINT chk_stock  CHECK (remaining_quantity >= 0 AND remaining_quantity <= total_quantity)
);

-- Index spatial pour la recherche géographique
CREATE INDEX idx_offers_location ON offers
    USING GIST (point(pickup_longitude, pickup_latitude));

CREATE INDEX idx_offers_status ON offers (status);
CREATE INDEX idx_offers_business ON offers (business_id);

-- Table de jointure many-to-many : Offres ↔ Tags alimentaires
CREATE TABLE offer_dietary_tags (
    offer_id    UUID NOT NULL REFERENCES offers(id) ON DELETE CASCADE,
    tag_id      UUID NOT NULL REFERENCES dietary_tags(id) ON DELETE RESTRICT,
    PRIMARY KEY (offer_id, tag_id)
);
```

#### Table commandes avec snapshot de prix (Pattern Snapshot Immuable)

```sql
-- Commandes
CREATE TABLE orders (
    id                  UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    consumer_id         UUID NOT NULL,
    business_id         UUID NOT NULL,
    subtotal_amount     NUMERIC(10,2) NOT NULL,
    subtotal_currency   CHAR(3) NOT NULL DEFAULT 'EUR',
    discount_amount     NUMERIC(10,2) NOT NULL DEFAULT 0,
    total_amount        NUMERIC(10,2) NOT NULL,
    promo_code          VARCHAR(50),
    status              VARCHAR(20) NOT NULL DEFAULT 'PENDING'
                            CHECK (status IN ('PENDING','CONFIRMED','READY',
                                              'PICKED_UP','CANCELLED','DISPUTED')),
    pickup_code         VARCHAR(20) NOT NULL UNIQUE,  -- contenu du QR code
    placed_at           TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    confirmed_at        TIMESTAMP WITH TIME ZONE,
    picked_up_at        TIMESTAMP WITH TIME ZONE
);

-- Articles de commande — SNAPSHOT IMMUABLE des prix au moment de l'achat
-- Raison : si l'offre est modifiée ou supprimée, la commande historique reste cohérente
CREATE TABLE order_items (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    order_id        UUID NOT NULL REFERENCES orders(id) ON DELETE CASCADE,
    offer_id        UUID NOT NULL,                  -- référence logique (pas de FK stricte)
    offer_title     VARCHAR(255) NOT NULL,          -- SNAPSHOT : titre au moment de la commande
    unit_price      NUMERIC(10,2) NOT NULL,         -- SNAPSHOT : prix au moment de la commande
    currency        CHAR(3) NOT NULL DEFAULT 'EUR',
    quantity        INT NOT NULL CHECK (quantity > 0)
);

CREATE INDEX idx_order_items_order ON order_items (order_id);

-- NOTE : offer_id dans order_items n'est PAS une FK avec ON DELETE CASCADE.
-- Si l'offre est supprimée du catalogue, les lignes de commande historiques
-- conservent leur snapshot et restent lisibles.
```

#### Table paiements (Append-Only)

```sql
-- Paiements — jamais mutés, toujours append-only
CREATE TABLE payments (
    id                      UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    order_id                UUID NOT NULL REFERENCES orders(id),
    consumer_id             UUID NOT NULL,
    amount                  NUMERIC(10,2) NOT NULL,
    currency                CHAR(3) NOT NULL DEFAULT 'EUR',
    status                  VARCHAR(20) NOT NULL DEFAULT 'PENDING'
                                CHECK (status IN ('PENDING','SUCCEEDED','FAILED','REFUNDED')),
    external_transaction_id VARCHAR(255),           -- ID Stripe
    created_at              TIMESTAMP WITH TIME ZONE DEFAULT NOW()
    -- Pas de updated_at : les paiements sont immuables
);

-- Remboursements — nouvel enregistrement, jamais une mutation du payment original
CREATE TABLE refunds (
    id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    payment_id  UUID NOT NULL REFERENCES payments(id),
    dispute_id  UUID,                               -- nullable : remboursement préventif possible
    amount      NUMERIC(10,2) NOT NULL,
    currency    CHAR(3) NOT NULL DEFAULT 'EUR',
    reason      TEXT NOT NULL,
    status      VARCHAR(20) NOT NULL DEFAULT 'PENDING'
                    CHECK (status IN ('PENDING','PROCESSED','FAILED')),
    created_at  TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Virements aux commerces
CREATE TABLE payouts (
    id                  UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    business_id         UUID NOT NULL,
    amount              NUMERIC(10,2) NOT NULL,
    currency            CHAR(3) NOT NULL DEFAULT 'EUR',
    commission_deducted NUMERIC(10,2) NOT NULL DEFAULT 0,
    status              VARCHAR(20) NOT NULL DEFAULT 'SCHEDULED'
                            CHECK (status IN ('SCHEDULED','PROCESSED','FAILED')),
    period_start        DATE NOT NULL,
    period_end          DATE NOT NULL,
    processed_at        TIMESTAMP WITH TIME ZONE
);
```

#### Table avis avec modération

```sql
-- Avis consommateurs sur les commerces
CREATE TABLE reviews (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    consumer_id     UUID NOT NULL,
    business_id     UUID NOT NULL,
    order_id        UUID NOT NULL REFERENCES orders(id),
    rating          SMALLINT NOT NULL CHECK (rating BETWEEN 1 AND 5),
    comment         TEXT,
    business_reply  TEXT,
    status          VARCHAR(20) NOT NULL DEFAULT 'VISIBLE'
                        CHECK (status IN ('VISIBLE','FLAGGED','REMOVED')),
    created_at      TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    UNIQUE (consumer_id, order_id)  -- Un seul avis par commande
);

CREATE INDEX idx_reviews_business ON reviews (business_id, status);
```

#### Table codes promo avec gestion de la concurrence

```sql
-- Codes promo plateforme
CREATE TABLE promo_codes (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    code            VARCHAR(50) NOT NULL UNIQUE,
    discount_type   VARCHAR(20) NOT NULL CHECK (discount_type IN ('PERCENTAGE','FIXED')),
    discount_value  NUMERIC(10,2) NOT NULL CHECK (discount_value > 0),
    valid_from      TIMESTAMP WITH TIME ZONE NOT NULL,
    valid_until     TIMESTAMP WITH TIME ZONE NOT NULL,
    max_uses        INT NOT NULL,
    current_uses    INT NOT NULL DEFAULT 0 CHECK (current_uses >= 0),
    status          VARCHAR(20) NOT NULL DEFAULT 'ACTIVE'
                        CHECK (status IN ('ACTIVE','EXPIRED','DISABLED')),
    CHECK (current_uses <= max_uses),
    CHECK (valid_until > valid_from)
);

-- Incrémentation atomique pour éviter les race conditions sur les codes promo
-- Utiliser : UPDATE promo_codes SET current_uses = current_uses + 1
--            WHERE id = ? AND current_uses < max_uses AND status = 'ACTIVE'
-- Vérifier rowsAffected == 1 pour confirmer le succès.
```

---

*Document produit dans le cadre du projet EcoMeal — Workflow de Conception Étapes 1–5.*  
*Prochaine étape : Étape 6 — Architecture Decision Records (ADR) & Préparation à la Soutenance.*
