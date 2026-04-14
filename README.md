# EcoMeal — Workflow de Conception
> **Cours :** Conception d'Application & Architecture N-Tier — Prof. Addi  
> **Projet :** EcoMeal — Réduire le gaspillage alimentaire en connectant des commerces locaux avec des utilisateurs pour vendre les invendus à prix réduit.

---

## Table des matières

1. [Étape 1 — Liste brute des fonctionnalités](#étape-1--liste-brute-des-fonctionnalités)
2. [Étape 2 — Regroupement par domaine métier (DDD)](#étape-2--regroupement-par-domaine-métier-ddd)
3. [Étape 3 — Entités métier par module](#étape-3--entités-métier-par-module)

---

## Étape 1 — Liste brute des fonctionnalités

> **Méthode :** Liste exhaustive, non triée. Nous sommes en mode « fonctionnel pur » — *ce que* fait l'application, pas *comment* elle est construite. Trois perspectives : Consommateur, Partenaire commercial, et Administrateur système.

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

Après analyse de la liste de fonctionnalités, six **Contextes Bornés** émergent naturellement :

| # | Module / Contexte Borné | Responsabilité principale | Raison clé de la séparation |
|---|---|---|---|
| 1 | **UserIdentity** | Qui êtes-vous ? Authentification et profil | La sécurité/auth évolue indépendamment de tout le reste |
| 2 | **Catalog** | Qu'est-ce qui est disponible ? Gestion du cycle de vie des offres | Les commerces publient/modifient des offres ; change fréquemment |
| 3 | **Order** | Qu'a-t-on acheté ? Cycle de vie des commandes et validation QR | Les règles de paiement, de fulfillment et de litige sont complexes et distinctes |
| 4 | **Payment** | Comment l'argent est géré ? Paiements, virements, remboursements | La logique financière est réglementée et complètement isolée |
| 5 | **Notification** | Comment les utilisateurs sont informés ? Alertes, emails, push | La logique des canaux de communication évolue indépendamment |
| 6 | **Administration** | Qui gère la plateforme ? Modération et gouvernance | Les flux de travail admin sont internes et jamais exposés aux utilisateurs finaux |

---

### Module 1 — `UserIdentity`

**Justification de la cohésion :** Toutes les fonctionnalités ici ont une seule raison d'évoluer — la façon dont les utilisateurs sont authentifiés et dont leur profil personnel est géré. Envoyer un email à un utilisateur n'est *pas* le rôle de ce module (cela appartient à `Notification`).

| Fonctionnalité | Origine |
|---|---|
| Créer un compte consommateur | Consommateur |
| Se connecter / Se déconnecter | Consommateur |
| Réinitialiser le mot de passe oublié | Consommateur |
| Consulter et modifier le profil personnel | Consommateur |
| Gérer les préférences de notification | Consommateur |
| Créer un compte professionnel | Commerce |
| Soumettre le commerce pour approbation admin | Commerce |
| Se connecter au tableau de bord professionnel | Commerce |
| Gérer les comptes utilisateurs / commerces (admin) | Admin |
| Approuver / rejeter / suspendre des comptes | Admin |

---

### Module 2 — `Catalog`

**Justification de la cohésion :** Tout ce qui concerne *les aliments disponibles* réside ici. L'offre est l'artefact central : sa création, sa visibilité, son stock et sa découverte. La recherche géographique est aussi ici car elle interroge directement les offres. Les règles de tarification (remises, commissions) sont déléguées à `Payment` et `Administration`.

| Fonctionnalité | Origine |
|---|---|
| Créer une nouvelle offre de repas | Commerce |
| Modifier une offre existante | Commerce |
| Désactiver / supprimer une offre | Commerce |
| Marquer une offre comme « épuisée » | Commerce |
| Consulter le stock en temps réel | Commerce |
| Gérer les catégories et tags alimentaires | Admin |
| Parcourir les offres (carte / liste) | Consommateur |
| Filtrer les offres (catégorie, distance, prix, tag alimentaire) | Consommateur |
| Rechercher des offres par nom ou type | Consommateur |
| Consulter le détail d'une offre | Consommateur |
| Sauvegarder ses commerces favoris | Consommateur |
| Télécharger le logo et les photos du commerce | Commerce |
| Gérer les horaires d'ouverture du commerce | Commerce |
| Définir le lieu de retrait / rayon de service | Commerce |

---

### Module 3 — `Order`

**Justification de la cohésion :** Toutes les fonctionnalités partagent le même cycle de vie métier — du panier à la confirmation du retrait. La validation QR appartient ici car c'est une étape du flux de fulfillment des commandes, et non un événement de communication. Les litiges sont initiés ici mais *résolus* dans `Payment`.

| Fonctionnalité | Origine |
|---|---|
| Ajouter / retirer un article du panier | Consommateur |
| Appliquer un code promo | Consommateur |
| Passer une commande | Consommateur |
| Recevoir la confirmation de commande | Consommateur |
| Consulter les commandes en cours et passées | Consommateur |
| Annuler une commande | Consommateur |
| Générer un QR code pour le retrait | Consommateur |
| Scanner le QR code pour valider le retrait | Commerce |
| Marquer une commande comme retirée / terminée | Commerce |
| Consulter les commandes du jour | Commerce |
| Signaler un problème avec une commande | Consommateur |
| Consulter les commandes et litiges (admin) | Admin |

---

### Module 4 — `Payment`

**Justification de la cohésion :** Tous les mouvements d'argent sont isolés ici. Ce module s'intègre avec des prestataires de paiement externes (Stripe, etc.) et est soumis à la réglementation financière — il doit pouvoir évoluer (ou être remplacé) sans toucher à `Order` ou `Catalog`.

| Fonctionnalité | Origine |
|---|---|
| Traiter le paiement en ligne à la commande | Consommateur |
| Traiter une demande de remboursement | Consommateur |
| Ouvrir et résoudre un litige | Consommateur |
| Recevoir des virements pour les commandes terminées | Commerce |
| Consulter l'historique des virements et les factures | Commerce |
| Configurer les taux de commission | Admin |
| Traiter les remboursements et résolutions de litiges | Admin |
| Gérer les codes promo à l'échelle de la plateforme | Admin |

---

### Module 5 — `Notification`

**Justification de la cohésion :** Tous les événements de communication sont centralisés ici, quel que soit le canal (email, push, in-app). Les autres modules émettent des *Événements Domaine* (ex. `OrderPlaced`, `OfferLowStock`) ; ce module *écoute* et décide comment communiquer. C'est un pattern Observer classique — l'émetteur ne connaît pas le canal de notification.

| Fonctionnalité | Origine |
|---|---|
| Envoyer la confirmation de commande (in-app + email) | Système |
| Envoyer des alertes push/email pour les nouvelles offres à proximité | Consommateur |
| Notifier le commerce d'une nouvelle commande | Commerce |
| Notifier le commerce lorsque le stock d'une offre est faible | Commerce |
| Envoyer des annonces à l'ensemble de la plateforme | Admin |
| Gérer les préférences de notification (consommateur) | Consommateur |

---

### Module 6 — `Administration`

**Justification de la cohésion :** La gouvernance, la modération et la configuration de la plateforme sont des opérations internes auxquelles aucun consommateur ou commerce n'a accès. Ces fonctionnalités évoluent au rythme de la politique métier — indépendamment des fonctionnalités produit.

| Fonctionnalité | Origine |
|---|---|
| Modérer les contenus signalés (avis, offres) | Admin |
| Consulter le tableau de bord analytique de la plateforme | Admin |
| Surveiller l'état du système | Admin |
| Gérer le contenu (FAQ, CGU, politique de confidentialité) | Admin |
| Consulter le tableau de bord analytique du commerce | Commerce (lecture seule) |
| Noter et laisser un avis sur un commerce | Consommateur |
| Répondre aux avis des consommateurs | Commerce |

---

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

> **Faible couplage garanti :** `Notification` n'importe jamais directement depuis `Order` ou `Catalog`. Il réagit uniquement aux Événements Domaine publiés. `Catalog` ne sait pas que `Payment` existe — il stocke simplement un prix. `Order` appelle `Payment` via une interface (Principe d'Inversion de Dépendance).

---

## Étape 3 — Entités métier par module

> **Vocabulaire :**
> - **Entité** — Possède une identité unique, est mutable dans le temps (ex. : `Order` change de statut)
> - **Objet Valeur** — Pas d'identité, immuable, défini entièrement par ses valeurs (ex. : `Money(8.50, EUR)`)
> - **Racine d'Agrégat** — Point d'entrée unique d'un groupe d'entités liées ; les modules externes ne conservent qu'une référence à l'ID de la racine

---

### Module 1 — `UserIdentity`

#### Entité : `User`
| Attribut | Type | Issu de |
|---|---|---|
| `id` | UUID | Généré par le système |
| `email` | String | Inscription |
| `passwordHash` | String | Inscription |
| `role` | Enum : `CONSUMER, BUSINESS, ADMIN` | Type d'inscription |
| `status` | Enum : `PENDING, ACTIVE, SUSPENDED` | Approbation admin |
| `createdAt` | DateTime | Système |

#### Entité : `ConsumerProfile` *(appartenant à User)*
| Attribut | Type | Issu de |
|---|---|---|
| `userId` | UUID | FK vers User |
| `firstName` | String | Modification du profil |
| `lastName` | String | Modification du profil |
| `dietaryPreferences` | List\<Tag\> | Modification du profil |
| `notificationPreferences` | NotificationPreferences | Paramètres |

#### Entité : `BusinessProfile` *(appartenant à User)*
| Attribut | Type | Issu de |
|---|---|---|
| `userId` | UUID | FK vers User |
| `businessName` | String | Inscription |
| `registrationNumber` | String | Inscription (SIRET) |
| `businessType` | Enum : `RESTAURANT, BAKERY, SUSHI, ...` | Inscription |
| `logoUrl` | String | Téléchargement |
| `approvalStatus` | Enum : `PENDING, APPROVED, REJECTED` | Action admin |

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
> Une `Offer` est l'artefact central de toute la plateforme. Son cycle de vie (DRAFT → ACTIVE → SOLD_OUT → EXPIRED) déclenche de nombreux événements en aval.

| Attribut | Type | Issu de |
|---|---|---|
| `id` | UUID | Généré par le système |
| `businessId` | UUID | FK vers BusinessProfile |
| `title` | String | Fonctionnalité : Créer une offre |
| `description` | String | Fonctionnalité : Créer une offre |
| `photoUrls` | List\<String\> | Fonctionnalité : Télécharger des photos |
| `category` | Enum : `BAKERY, RESTAURANT, SUSHI, ...` | Fonctionnalité : Filtrage par tag |
| `dietaryTags` | List\<Tag\> | Fonctionnalité : Filtre alimentaire |
| `originalPrice` | Money | Fonctionnalité : Afficher le prix original |
| `discountedPrice` | Money | Fonctionnalité : Prix de vente réduit |
| `totalQuantity` | Integer | Fonctionnalité : Définir le stock |
| `remainingQuantity` | Integer | Fonctionnalité : Stock en temps réel |
| `pickupWindow` | TimeWindow | Fonctionnalité : Créneau de retrait |
| `expiresAt` | DateTime | Fonctionnalité : Expiration de l'offre |
| `status` | Enum : `DRAFT, ACTIVE, SOLD_OUT, EXPIRED, DELETED` | Cycle de vie |
| `pickupLocation` | Address | Fonctionnalité : Définir le lieu |

#### Entité : `Business` *(projection en lecture seule depuis UserIdentity)*
> `Catalog` ne conserve qu'une référence `businessId`. Les données complètes du commerce résident dans `UserIdentity`. Cela renforce le faible couplage.

#### Objet Valeur : `Money`
| Attribut | Type |
|---|---|
| `amount` | BigDecimal |
| `currency` | Enum : `EUR, USD, ...` |

#### Objet Valeur : `TimeWindow`
| Attribut | Type |
|---|---|
| `startTime` | LocalTime |
| `endTime` | LocalTime |
| `date` | LocalDate |

#### Objet Valeur : `Address`
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

#### Entité : `Tag` *(label alimentaire / diététique)*
| Attribut | Type |
|---|---|
| `id` | UUID |
| `label` | String (ex. : « vegan », « sans gluten ») |

---

### Module 3 — `Order`

#### Racine d'Agrégat : `Order`
> L'agrégat `Order` encapsule le cycle de vie complet d'un achat. Le `Cart` est un état pré-commande transitoire — il devient une `Order` lors de la confirmation du paiement. Aucun module externe ne modifie directement une `Order` ; tous les changements passent par la racine d'agrégat.

| Attribut | Type | Issu de |
|---|---|---|
| `id` | UUID | Généré par le système |
| `consumerId` | UUID | FK vers UserIdentity |
| `businessId` | UUID | FK vers UserIdentity |
| `items` | List\<OrderItem\> | Fonctionnalité : Ajouter au panier |
| `subtotal` | Money | Calculé |
| `discountApplied` | Money | Fonctionnalité : Code promo |
| `totalAmount` | Money | Calculé |
| `promoCode` | String (nullable) | Fonctionnalité : Appliquer un promo |
| `status` | Enum : `PENDING, CONFIRMED, READY, PICKED_UP, CANCELLED, DISPUTED` | Cycle de vie |
| `pickupCode` | String (contenu QR) | Fonctionnalité : Générer QR |
| `placedAt` | DateTime | Système |
| `confirmedAt` | DateTime | Événement paiement |
| `pickedUpAt` | DateTime | Scan commerce |

#### Entité : `OrderItem` *(partie de l'agrégat Order)*
| Attribut | Type | Issu de |
|---|---|---|
| `offerId` | UUID | FK vers Catalog.Offer |
| `offerTitle` | String | Snapshot au moment de la commande |
| `unitPrice` | Money | Snapshot au moment de la commande |
| `quantity` | Integer | Fonctionnalité : Ajouter au panier |

> **Note de conception :** `offerTitle` et `unitPrice` sont *capturés en snapshot* à la création de la commande. C'est essentiel — si un commerce modifie ou supprime une offre, les commandes historiques doivent rester intactes. Il s'agit du pattern **Snapshot Immuable**.

#### Entité : `Dispute`
| Attribut | Type | Issu de |
|---|---|---|
| `id` | UUID | Généré par le système |
| `orderId` | UUID | FK vers Order |
| `consumerId` | UUID | FK vers UserIdentity |
| `reason` | String | Fonctionnalité : Signaler un problème |
| `status` | Enum : `OPEN, UNDER_REVIEW, RESOLVED, REJECTED` | Action admin |
| `resolution` | String (nullable) | Action admin |
| `createdAt` | DateTime | Système |

---

### Module 4 — `Payment`

#### Racine d'Agrégat : `Payment`
> Tous les enregistrements financiers sont immuables une fois créés (append-only). Un `Refund` est un nouvel enregistrement, jamais une mutation du `Payment` original. Cela est conforme aux exigences d'audit financier et constitue un fondement pour l'Event Sourcing si nécessaire.

| Attribut | Type | Issu de |
|---|---|---|
| `id` | UUID | Généré par le système |
| `orderId` | UUID | FK vers Order |
| `consumerId` | UUID | FK vers UserIdentity |
| `amount` | Money | Total de la commande |
| `status` | Enum : `PENDING, SUCCEEDED, FAILED, REFUNDED` | Prestataire de paiement |
| `externalTransactionId` | String | Stripe / prestataire |
| `createdAt` | DateTime | Système |

#### Entité : `Payout`
| Attribut | Type | Issu de |
|---|---|---|
| `id` | UUID | Généré par le système |
| `businessId` | UUID | FK vers UserIdentity |
| `amount` | Money | Commandes terminées sur la période |
| `commissionDeducted` | Money | Fonctionnalité : Config commission |
| `status` | Enum : `SCHEDULED, PROCESSED, FAILED` | Traitement par lot |
| `periodStart` | LocalDate | Calendrier des virements |
| `periodEnd` | LocalDate | Calendrier des virements |

#### Entité : `Refund`
| Attribut | Type | Issu de |
|---|---|---|
| `id` | UUID | Généré par le système |
| `paymentId` | UUID | FK vers Payment |
| `disputeId` | UUID (nullable) | FK vers Dispute |
| `amount` | Money | Montant du remboursement |
| `reason` | String | Admin / consommateur |
| `status` | Enum : `PENDING, PROCESSED, FAILED` | Prestataire |

#### Objet Valeur : `CommissionRate`
| Attribut | Type |
|---|---|
| `businessType` | Enum |
| `rate` | BigDecimal (ex. : 0.15 = 15%) |

---

### Module 5 — `Notification`

> Ce module ne contient **aucune entité métier** au sens DDD — il est purement réactif. Il s'abonne aux Événements Domaine émis par d'autres modules et les convertit en communications.

#### Entité : `NotificationLog` *(piste d'audit)*
| Attribut | Type | Issu de |
|---|---|---|
| `id` | UUID | Généré par le système |
| `recipientId` | UUID | Utilisateur cible |
| `channel` | Enum : `EMAIL, PUSH, IN_APP` | Préférences |
| `type` | Enum : `ORDER_CONFIRMED, NEW_OFFER, LOW_STOCK, ...` | Événement Domaine |
| `payload` | JSON | Données de l'événement |
| `sentAt` | DateTime | Système |
| `status` | Enum : `SENT, FAILED, SKIPPED` | Résultat de livraison |

#### Événements Domaine consommés (depuis d'autres modules) :
| Événement | Émis par | Action |
|---|---|---|
| `OrderPlaced` | Order | Notifier le consommateur (confirmation) + le commerce (nouvelle commande) |
| `OrderPickedUp` | Order | Notifier le consommateur (reçu) |
| `OrderCancelled` | Order | Notifier le commerce |
| `OfferLowStock` | Catalog | Notifier le commerce |
| `BusinessApproved` | UserIdentity | Notifier le commerce |
| `DisputeResolved` | Payment | Notifier le consommateur |

---

### Module 6 — `Administration`

#### Entité : `Review`
| Attribut | Type | Issu de |
|---|---|---|
| `id` | UUID | Généré par le système |
| `consumerId` | UUID | FK vers UserIdentity |
| `businessId` | UUID | FK vers UserIdentity |
| `orderId` | UUID | FK vers Order |
| `rating` | Integer (1–5) | Fonctionnalité : Noter le commerce |
| `comment` | String | Fonctionnalité : Rédiger un avis |
| `businessReply` | String (nullable) | Fonctionnalité : Réponse du commerce |
| `status` | Enum : `VISIBLE, FLAGGED, REMOVED` | Modération |
| `createdAt` | DateTime | Système |

#### Entité : `PromoCode`
| Attribut | Type | Issu de |
|---|---|---|
| `id` | UUID | Généré par le système |
| `code` | String | Créé par l'admin |
| `discountType` | Enum : `PERCENTAGE, FIXED` | Fonctionnalité |
| `discountValue` | BigDecimal | Fonctionnalité |
| `validFrom` | DateTime | Fonctionnalité |
| `validUntil` | DateTime | Fonctionnalité |
| `maxUses` | Integer | Fonctionnalité |
| `currentUses` | Integer | Calculé |
| `status` | Enum : `ACTIVE, EXPIRED, DISABLED` | Cycle de vie |

#### Entité : `ModerationReport`
| Attribut | Type | Issu de |
|---|---|---|
| `id` | UUID | Généré par le système |
| `reportedBy` | UUID | FK vers UserIdentity |
| `targetType` | Enum : `REVIEW, OFFER, USER` | Fonctionnalité : Signalement |
| `targetId` | UUID | Entité cible |
| `reason` | String | Fonctionnalité : Signalement |
| `status` | Enum : `PENDING, REVIEWED, ACTIONED` | Admin |

---

## Résumé de conception — Checklist en 4 questions

Comme enseigné par le Prof. Addi, chaque fonctionnalité doit répondre à ces quatre questions :

| Module | Quel module ? | Quelles données ? | Quelle logique ? | Quelle interaction ? |
|---|---|---|---|---|
| UserIdentity | Cycle de vie utilisateur | User, ConsumerProfile, BusinessProfile | Hachage du mot de passe, attribution de rôle, flux d'approbation | REST (POST /register, POST /login), tableau de bord Admin |
| Catalog | Disponibilité des offres | Offer, Category, Tag | Décrémentation du stock, vérification d'expiration, recherche géo | REST (GET /offers, POST /offers), Cron (expiration des offres) |
| Order | Flux d'achat | Order, OrderItem, Dispute | Validation promo, génération QR, snapshot des prix | REST (POST /orders, PATCH /orders/{id}/cancel), événement scan QR |
| Payment | Mouvement d'argent | Payment, Payout, Refund | Déduction de commission, éligibilité au remboursement | Webhook Stripe, REST (GET /payouts), Cron (virement hebdomadaire) |
| Notification | Communication | NotificationLog | Routage par canal, filtrage des préférences | Écouteur d'événements (Kafka / Observer), prestataire Email/Push |
| Administration | Gouvernance de la plateforme | Review, PromoCode, ModerationReport | Règles de modération, validité des promos | API REST Admin, tableau de bord interne |

---

*Document produit dans le cadre du TP EcoMeal — Workflow de Conception Étapes 1–3.*  
*Prochaine étape : Étape 4 — Dérivation des composants techniques et rédaction des ADR.*
