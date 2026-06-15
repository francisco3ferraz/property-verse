# PropertyVerse

PropertyVerse is a full-stack property rental marketplace built with **Ruby on Rails 7**. It allows users to browse, favorite, and book rental properties, leave reviews after their stays, and optionally become hosts to list their own properties. Payments are processed securely via **Stripe**.

---

## Table of Contents

- [Features](#features)
- [Tech Stack](#tech-stack)
- [Architecture Overview](#architecture-overview)
- [Data Model](#data-model)
- [Prerequisites](#prerequisites)
- [Local Development Setup](#local-development-setup)
- [Environment Variables](#environment-variables)
- [Database Setup](#database-setup)
- [Running the App](#running-the-app)
- [Running Tests](#running-tests)
- [Docker](#docker)
- [API Endpoints](#api-endpoints)
- [Authorization](#authorization)

---

## Features

| Feature | Description |
|---|---|
| **Authentication** | Sign up, log in, and password reset via Devise |
| **User Roles** | Users can upgrade to **Host** status to list properties |
| **Property Listings** | Hosts create listings with images, descriptions, pricing, and address |
| **Geocoding** | Properties and profiles are automatically geocoded from their address |
| **Search & Browse** | Browse all listings from the home page; filter by city or country |
| **Favorites** | Logged-in users can favourite/unfavourite properties (AJAX, JSON API) |
| **Reservations** | Guests book a property for a date range; availability is enforced |
| **Payments** | Stripe charges are created at booking time; subtotal, cleaning fee (€50), and 8% service fee are calculated |
| **Reviews** | Guests who have stayed at a property can leave a rated review (1–5 ★); average rating is updated automatically |
| **Host Dashboard** | Hosts see their own listings and incoming payments |
| **Profiles** | Every user has a profile with name, address, and avatar |
| **Image Uploads** | Property images and profile pictures stored via Active Storage |

---

## Tech Stack

| Layer | Technology |
|---|---|
| **Language** | Ruby 3.1.3 |
| **Framework** | Rails 7.0 |
| **Database** | PostgreSQL 14 |
| **CSS** | Tailwind CSS (via `tailwindcss-rails`) |
| **JavaScript** | Hotwire (Turbo + Stimulus) + Importmap |
| **Payments** | Stripe (`stripe-rails`) |
| **Authentication** | Devise |
| **Authorization** | Pundit |
| **Geocoding** | Geocoder |
| **Money** | money-rails |
| **Image Processing** | Active Storage + libvips |
| **Serialization** | JSONAPI::Serializer |
| **Testing** | RSpec, FactoryBot, Shoulda Matchers |
| **Containerization** | Docker + Docker Compose |

---

## Architecture Overview

```
app/
├── controllers/
│   ├── api/                  # JSON API endpoints (favorites, user lookups)
│   ├── host/                 # Host-scoped controllers (property CRUD, payments)
│   ├── home_controller.rb    # Landing page – lists all properties
│   ├── properties_controller.rb
│   ├── reservation_payments_controller.rb
│   ├── reviews_controller.rb
│   ├── favorites_controller.rb
│   ├── hostify_controller.rb # Upgrades a user to host role
│   ├── profiles_controller.rb
│   ├── accounts_controller.rb
│   └── passwords_controller.rb
├── models/
│   ├── user.rb          # Devise auth + host/customer roles
│   ├── profile.rb       # One-to-one with User; geocoded
│   ├── property.rb      # Core listing model; geocoded + monetized
│   ├── reservation.rb   # Links guest ↔ property for a date range
│   ├── payment.rb       # Monetized breakdown: subtotal, cleaning, service, total
│   ├── review.rb        # Polymorphic; auto-updates average_rating
│   └── favorite.rb      # Join table: user ↔ property
├── policies/            # Pundit authorization policies
└── serializers/         # JSONAPI serializers for API responses
```

---

## Data Model

```
User ──< Property          (host owns listings)
User ──< Reservation       (guest books a stay)
User ──< Review
User ──< Favorite
User ──  Profile

Property ──< Reservation
Property ──< Review        (polymorphic)
Property ──< Favorite
Property ──< Payment       (through Reservations)

Reservation ── Payment     (one-to-one)
```

**Fee structure** (defined on `Property`):

| Fee | Value |
|---|---|
| Cleaning fee | €50 flat |
| Service fee | 8% of subtotal |

---

## Prerequisites

- Ruby **3.1.3** (use `rbenv` or `rvm`)
- Bundler
- PostgreSQL **14+**
- Node.js + Yarn (for asset compilation)
- A [Stripe](https://stripe.com) account (test keys are fine)

---

## Local Development Setup

```bash
# 1. Clone the repository
git clone <repo-url>
cd property-verse

# 2. Install Ruby gems
bundle install

# 3. Configure environment variables
cp .env.example .env
# Edit .env with your credentials (see Environment Variables below)

# 4. Set up the database
bin/rails db:create db:migrate db:seed

# 5. Start the development server
bin/dev
```

`bin/dev` runs two processes concurrently (defined in `Procfile.dev`):

| Process | Command |
|---|---|
| `web` | `bin/rails server -p 3000` |
| `css` | `bin/rails tailwindcss:watch` |

The app will be available at **http://localhost:3000**.

---

## Environment Variables

Copy `.env.example` to `.env` and fill in the values:

| Variable | Description |
|---|---|
| `DATABASE_URL` | PostgreSQL connection string |
| `POSTGRES_DB` | Database name |
| `POSTGRES_USER` | Database username |
| `POSTGRES_PASSWORD` | Database password |
| `STRIPE_PUBLISHABLE_KEY` | Stripe publishable API key |
| `STRIPE_SECRET_KEY` | Stripe secret API key |
| `RAILS_MASTER_KEY` | Rails credentials master key (found in `config/master.key`) |

---

## Database Setup

```bash
# Create databases
bin/rails db:create

# Run all migrations
bin/rails db:migrate

# Seed sample data (creates 1 admin user + 5 fake users + 10 properties with reviews)
bin/rails db:seed
```

The seed script creates:

- **Admin user** — `admin@propertyverse.com` / `admin123`
- **5 guest users** with randomised names and avatars
- **10 properties** in Portugal with sample images and reviews

---

## Running Tests

The test suite uses **RSpec** with FactoryBot and Shoulda Matchers.

```bash
# Run the full test suite
bundle exec rspec

# Run a specific file
bundle exec rspec spec/models/property_spec.rb
```

---

## Docker

A `docker-compose.yml` is provided to run the app alongside a PostgreSQL container.

```bash
# Build and start all services
docker compose up --build

# Run in detached mode
docker compose up -d

# Tear down
docker compose down
```

The `propertyverse` service runs on port **3000** and depends on the `db` service (PostgreSQL 14).

---

## API Endpoints

The `/api` namespace exposes JSON endpoints consumed by the Stimulus controllers on the front-end.

| Method | Path | Description |
|---|---|---|
| `POST` | `/api/favorites` | Favourite a property |
| `DELETE` | `/api/favorites/:id` | Unfavourite a property |
| `GET` | `/api/users_by_emails` | Look up a user by email address |

Responses follow the **JSON:API** spec via `jsonapi-serializer`.

---

## Authorization

Authorization is handled by **Pundit** policies:

| Policy | Guards |
|---|---|
| `HostPolicy` | Only users with role `"host"` may access host-scoped actions |
| `ProfilePolicy` | Users may only edit their own profile |
| `AccountPolicy` | Users may only edit their own account |
| `PasswordPolicy` | Users may only change their own password |

A regular user can become a host by visiting their profile and clicking **"Become a Host"**, which calls `HostifyController#update` and sets `role = "host"`.