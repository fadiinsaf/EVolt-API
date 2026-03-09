# EVolt API 🚗⚡

API REST développée avec **Laravel** pour la gestion des bornes de recharge pour véhicules électriques.

Elle permet aux utilisateurs de :

- s'authentifier de manière sécurisée,
- rechercher des bornes disponibles,
- réserver un créneau de recharge,
- modifier ou annuler une réservation,
- consulter leurs sessions de recharge.

Elle permet aussi aux administrateurs de :

- gérer les bornes de recharge,
- suivre les statistiques d'utilisation,
- superviser les réservations et les utilisateurs.

---

## 📌 Contexte du projet

L'objectif principal de **EVolt API** est de fournir une solution backend moderne pour la gestion des bornes de recharge électrique.

Chaque borne est caractérisée par :

- un **type de connecteur**,
- une **puissance** exprimée en **kW**,
- un **état de disponibilité**.

Le système doit permettre une gestion précise des réservations et du suivi des sessions de recharge, tout en offrant une API claire et documentée pour une future intégration avec des applications web ou mobiles.

---

## 🎯 Objectifs

- Développer une API REST avec **Laravel**
- Utiliser **Laravel Sanctum** pour l'authentification
- Gérer les bornes, les réservations et les sessions de recharge
- Mettre à jour automatiquement la disponibilité des bornes
- Fournir une documentation API claire
- Tester l'ensemble des fonctionnalités avec **Unit Tests** et **Postman**

---

## 🧩 Fonctionnalités principales

### 👤 Utilisateur

- Authentification avec Laravel Sanctum
- Recherche de bornes disponibles par zone géographique
- Consultation des détails d'une borne :
  - disponibilité
  - type de connecteur
  - puissance
- Réservation d'une borne pour une période donnée
- Modification d'une réservation
- Annulation d'une réservation
- Consultation des sessions de recharge passées et actuelles

### 🔐 Administrateur

- Ajouter une borne
- Modifier une borne
- Supprimer une borne
- Définir et mettre à jour :
  - le type de connecteur
  - la puissance disponible
- Consulter les statistiques :
  - taux d'occupation
  - énergie délivrée
  - activité des bornes

### 🧪 Développeur

- Tests unitaires pour chaque fonctionnalité
- Tests Postman pour différents scénarios
- Documentation détaillée des endpoints
- Gestion automatique de l'expiration des réservations via **queues/jobs**

---

## ⭐ Bonus

- Utilisation de **Laravel Sail** pour la conteneurisation
- Utilisation de **slugs** lisibles avec **Spatie Laravel Sluggable**
- Suppression en cascade d'un utilisateur et de ses réservations
- Notifications par email ou SMS avant le début et la fin d'une session

---

## 🛠️ Stack technique

| Technologie | Rôle |
|---|---|
| **PHP / Laravel** | Framework principal |
| **Laravel Sanctum** | Authentification par token |
| **MySQL** | Base de données |
| **Laravel Queues / Jobs** | Tâches asynchrones |
| **Postman** | Tests API |
| **Swagger / Scribe** | Documentation |
| **Laravel Sail** *(bonus)* | Conteneurisation Docker |

---

## 🗂️ Structure fonctionnelle

Le projet est organisé autour des modules suivants :

- **Auth**
- **Users**
- **Stations**
- **Reservations**
- **Charging Sessions**
- **Admin Dashboard / Statistics**
- **Notifications**

---

## 🧱 Modèle métier

### Entités principales

| Entité | Description |
|---|---|
| `User` | Utilisateur authentifié |
| `Station` | Borne de recharge |
| `Reservation` | Créneau réservé par un utilisateur |
| `ChargingSession` | Session de recharge active ou passée |
| `Notification` | Alerte envoyée à l'utilisateur |

### Relations

- Un `User` peut avoir plusieurs `Reservations`
- Une `Station` peut avoir plusieurs `Reservations`
- Une `Reservation` peut générer une `ChargingSession`
- Un `User` peut recevoir plusieurs `Notifications`

---

## 🔑 Authentification

L'authentification se fait via **Laravel Sanctum**. Le token doit être envoyé dans chaque requête protégée :
```http
Authorization: Bearer YOUR_TOKEN
Accept: application/json
```

---

## 📡 Endpoints principaux

> Les routes exactes peuvent varier selon votre implémentation.

### Auth

| Méthode | Route | Description |
|---|---|---|
| `POST` | `/api/register` | Inscription |
| `POST` | `/api/login` | Connexion |
| `POST` | `/api/logout` | Déconnexion |
| `GET` | `/api/user` | Utilisateur authentifié |

### Stations

| Méthode | Route | Description |
|---|---|---|
| `GET` | `/api/stations` | Liste des bornes |
| `GET` | `/api/stations/{id}` | Détail d'une borne |
| `POST` | `/api/stations` | Créer une borne *(admin)* |
| `PUT` | `/api/stations/{id}` | Modifier une borne *(admin)* |
| `DELETE` | `/api/stations/{id}` | Supprimer une borne *(admin)* |

### Reservations

| Méthode | Route | Description |
|---|---|---|
| `GET` | `/api/reservations` | Liste des réservations |
| `POST` | `/api/reservations` | Créer une réservation |
| `GET` | `/api/reservations/{id}` | Détail d'une réservation |
| `PUT` | `/api/reservations/{id}` | Modifier une réservation |
| `DELETE` | `/api/reservations/{id}` | Annuler une réservation |

### Charging Sessions

| Méthode | Route | Description |
|---|---|---|
| `GET` | `/api/sessions` | Toutes les sessions |
| `GET` | `/api/sessions/current` | Session en cours |
| `GET` | `/api/sessions/history` | Historique des sessions |

### Admin / Statistics

| Méthode | Route | Description |
|---|---|---|
| `GET` | `/api/admin/statistics` | Statistiques globales |

---

## 📘 Exemples de payload

### Créer une réservation — Requête
```json
{
  "station_id": 1,
  "start_time": "2026-03-10 14:00:00",
  "duration": 90
}
```

### Créer une réservation — Réponse `201 Created`
```json
{
  "message": "Réservation créée avec succès",
  "data": {
    "id": 12,
    "station_id": 1,
    "start_time": "2026-03-10 14:00:00",
    "end_time": "2026-03-10 15:30:00",
    "status": "reserved"
  }
}
```

---

## ✅ Règles métier

- Une borne ne peut pas être réservée si elle est déjà occupée sur la même plage horaire
- Seul le **propriétaire** d'une réservation peut la modifier ou l'annuler
- Seul l'**administrateur** peut gérer les bornes
- À l'expiration d'une réservation, la disponibilité de la borne est mise à jour automatiquement
- Les sessions passées et actuelles sont consultables par l'utilisateur

---

## ⚠️ Codes de réponse HTTP

| Code | Statut | Signification |
|---|---|---|
| `200` | OK | Succès |
| `201` | Created | Ressource créée |
| `204` | No Content | Suppression réussie |
| `401` | Unauthorized | Utilisateur non authentifié |
| `403` | Forbidden | Accès refusé |
| `404` | Not Found | Ressource introuvable |
| `409` | Conflict | Conflit métier (ex. borne déjà réservée) |
| `422` | Unprocessable Entity | Erreur de validation |

---

## 🧪 Tests

### Tests unitaires

Des tests doivent couvrir :

- Authentification
- Gestion des bornes
- Réservations
- Sessions
- Statistiques
```bash
php artisan test
```

### Tests Postman

La collection Postman doit valider :

- Les cas nominaux
- Les cas d'erreur
- Les permissions utilisateur / admin
- Les règles de validation

---

## 📄 Documentation API

Générée avec **Postman**, **Swagger / OpenAPI** ou **Scribe**, elle doit inclure :

- La description de chaque endpoint
- Les paramètres attendus
- Les exemples de requêtes et réponses
- Les erreurs possibles

---

## 🚀 Installation

### 1. Cloner le projet
```bash
git clone https://github.com/fadiinsaf/EVolt-API.git
cd evolt-api
```

### 2. Installer les dépendances
```bash
composer install
```

### 3. Environnement
```bash
cp .env.example .env
php artisan key:generate
```

### 4. Configurer `.env`
```env
APP_NAME=EVoltAPI
APP_ENV=local
APP_KEY=
APP_DEBUG=true
APP_URL=http://localhost

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=evolt_api
DB_USERNAME=root
DB_PASSWORD=

QUEUE_CONNECTION=database
```

### 5. Base de données
```bash
php artisan migrate
php artisan db:seed
```

### 6. Démarrer
```bash
php artisan serve
```

---

## 🕒 Gestion automatique de la disponibilité
```bash
php artisan queue:table
php artisan migrate
php artisan queue:work
```

---

## 🐳 Laravel Sail *(bonus)*
```bash
./vendor/bin/sail up -d
./vendor/bin/sail artisan migrate
```

---

## 📬 Notifications *(bonus)*

Envoi automatique :

- ✉️ **Email** ou 📱 **SMS**
- Avant le **début** de la session
- Avant la **fin** de la session

---

## 👨‍💻 Auteur

**Fadi Insaf** – [GitHub](https://github.com/fadiinsaf) | [Email](mailto:fadiinafff@gmail.com)

---

## 📜 Licence

Ce projet est destiné à un usage pédagogique et académique.