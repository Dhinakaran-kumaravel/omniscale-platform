# OmniScale Database Notes (Step 1)

This file explains the 5 main tables in our PostgreSQL database, what each one stores, and how they connect to one another.

---

## 1. `users`
- **What it stores:** Individual people who sign up for the platform (their unique ID, email address, hashed password, account active status, and join date).
- **Why it is separate:** A person's login identity should not be tied to just one company.

## 2. `organizations`
- **What it stores:** Companies, teams, or workspaces (their ID, name, URL-friendly `slug`, `owner_id`, `plan_tier`, and creation date).
- **How it connects:** `owner_id` points to a user in `users` who created/owns the organization.

## 3. `memberships`
- **What it stores:** The link between a user and an organization, along with that user's `role` (`ADMIN`, `MEMBER`, or `VIEWER`).
- **How it connects:**
  - `user_id` links to `users.id`
  - `org_id` links to `organizations.id`
- **Why we need a separate table:** One user can belong to multiple organizations with different roles in each (for example, `ADMIN` in one workspace and `VIEWER` in another). A single `role` column on `users` could not represent that cleanly.

## 4. `api_keys`
- **What it stores:** Secret credentials used by external apps or scripts to send events to our FastAPI service (`id`, `org_id`, `key_prefix`, `hashed_key`, `is_active`, `last_used_at`).
- **How it connects:** `org_id` links to `organizations.id`, so every API key belongs to a specific organization.
- **Security note:** We store `key_prefix` (for display/identification) and `hashed_key` (a cryptographic hash), never the raw secret key in plain text.

## 5. `event_logs`
- **What it stores:** Incoming events sent by organizations (`id`, `org_id`, `event_name`, `payload` as JSON data, client `ip_address`, and `timestamp`).
- **How it connects:** `org_id` associates each logged event with the organization that sent it so analytics and queries stay isolated per tenant.
- **Performance note:** Because `event_logs` can grow to hundreds of thousands or millions of rows, indexing `(org_id, timestamp)` helps keep time-series and per-organization queries fast.

---

## How the Relationships Flow at a Glance

- `users` <--(many-to-many via `memberships`)--> `organizations`
- `users` --(owns)--> `organizations` (`owner_id`)
- `organizations` --(has many)--> `api_keys`
- `organizations` --(generates many)--> `event_logs`
