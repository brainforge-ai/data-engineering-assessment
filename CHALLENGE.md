# Data Engineering Challenge — Interview Assignment

This challenge evaluates ingestion, transformation, orchestration, and database design using **Shopify-style data** (customers, orders, products) in the `DATA/` folder. Use the metadata docs in `DATA/` to understand the JSONL structure.

**Deliverables:** Working implementation plus documentation. You will write code, SQL, configs, and docs as needed — use any tools and languages you prefer.

---

## 1. Data and Metadata (Reference)

- **Raw data:** Three JSONL files in `DATA/`:
  - `portable_shopify.customers.sample.jsonl`
  - `portable_shopify.orders.sample.jsonl`
  - `portable_shopify.products.sample.jsonl`
- **Metadata:** See `DATA/METADATA_customers.md`, `DATA/METADATA_orders.md`, `DATA/METADATA_products.md` for field-level descriptions and join keys.

---

## 2. Ingestion: Airbyte + Postgres

**Tasks:**

1. **Run Airbyte (open source)**  
   - Run a local (or containerized) Airbyte OSS instance.  
   - Document how you started it and how a reviewer can run it (e.g. `docker-compose`, ports, env).

2. **Ingest JSONL → Postgres**  
   - Use Airbyte to load the three JSONL files into a Postgres database.  
   - You may use a “File” or “Local JSON” style source (or equivalent) and Postgres as destination.  
   - Decide how to represent nested structures (e.g. orders with `LINE_ITEMS`, `CUSTOMER`; products with `VARIANTS`; customers with `ADDRESSES`) — e.g. raw JSON columns, separate tables, or flattened tables — and document the choice.

3. **Schema and naming**  
   - Use a clear schema/namespace for raw tables (e.g. `raw` or `ingestion`) so they are distinct from transformed layers.  
   - Document the final table names and a short description of what each contains.

---

## 3. Postgres: Roles and RBAC

**Tasks:**

Define and implement **four distinct roles** in Postgres and document the exact privileges (e.g. in a `docs/` or `scripts/` file):

1. **Developer**  
   - Can read/write in development/staging schemas (e.g. `staging`, `dev`).  
   - No direct DDL/DML on production analytics schemas if you separate them; or document how you restrict prod.

2. **Ingestion tool (Airbyte)**  
   - Only writes (and reads if needed for upserts) into the **raw/ingestion** schema.  
   - No access to transformation or analytics schemas.

3. **Transformation tool (dbt)**  
   - Reads from raw (and optionally staging) schemas.  
   - Writes only to defined transformation/analytics schemas (e.g. `staging`, `marts`, or `analytics`).  
   - No ability to modify the raw schema.

4. **BI tool**  
   - Read-only access to the **analytics/marts** layer (e.g. `marts` or `analytics`).  
   - No read access to raw data; no write access anywhere.

Provide:

- Role names and a 1–2 sentence purpose for each.  
- Which schema(s) each role can SELECT/INSERT/UPDATE/DELETE (or CREATE if applicable).  
- Optional: one SQL script or list of commands that create the roles and grant these privileges so a reviewer can apply them.

---

## 4. Transformation: dbt Core

**Tasks:**

1. **dbt project layout**  
   - Set up a dbt Core project with a clear structure (e.g. `models/` with subfolders such as `staging`, `intermediate`, `marts`).  
   - Use **separate targets/environments** for:  
     - **Staging** (e.g. `staging` schema or database).  
     - **Production** (e.g. `analytics` or `marts` schema).  
   - Document how to run against staging vs production (e.g. `profiles.yml` and `dbt run --target prod`).

2. **Staging models**  
   - Build staging models that read from the **raw** tables populated by Airbyte.  
   - Apply light cleaning and naming (e.g. lowercase, snake_case) and document any assumptions (e.g. handling of nested JSON).

3. **Order summary table (mart)**  
   - Build a **single order summary** table (or view) that:  
     - Is at **one row per order** (or per order + line, depending on what you define as “order summary”).  
     - Joins orders with customers and product/variant information as needed.  
     - Includes at least: order identifier, customer identifier, order date, financial/fulfillment status, and a small set of useful metrics (e.g. total amount, number of line items, discount amount).  
   - Document the grain and column definitions.

4. **Tests and docs**  
   - Add at least one dbt test (e.g. uniqueness, not null, or relationship) relevant to the order summary.  
   - Add a short description for the order summary model and its grain in `schema.yml` (or equivalent).

---

## 5. Orchestration: GitHub Actions

**Tasks:**

Implement **two** GitHub Actions jobs that run dbt:

1. **PR / merge to main**  
   - Trigger: push or pull request targeting `main` (or default branch).  
   - Run dbt (e.g. `dbt deps`, `dbt compile`, and `dbt run` — and optionally `dbt test`) against the **staging** target.  
   - Use secrets or env for Postgres credentials; do not commit credentials.  
   - Document how to configure the repo secrets (e.g. `DBT_STAGING_HOST`, `DBT_STAGING_USER`, etc.).

2. **Scheduled production run**  
   - Trigger: schedule (e.g. daily or every 6 hours — state the chosen schedule).  
   - Run dbt against the **production** target (e.g. `dbt run --target prod` and optionally `dbt test`).  
   - Document the schedule and any assumptions (e.g. main branch only, or a specific branch).

Provide:

- Path to the workflow file(s).  
- A short README or `docs/` section describing both jobs, triggers, and required secrets.

---

## 6. Deliverables Checklist

- [ ] Airbyte running and documented; three JSONL streams ingested into Postgres raw layer. Showcase in final presentation. 
- [ ] Postgres roles and RBAC: developer, ingestion tool, transformation tool, BI tool — documented and scripted.  
- [ ] dbt project with staging vs production targets and staging models.  
- [ ] One order summary mart (grain and columns documented).  
- [ ] At least one dbt test and model description.  
- [ ] Two GitHub Actions jobs: PR → staging dbt run; schedule → production dbt run.  
- [ ] README or docs covering: how to run Airbyte, run dbt (staging/prod), configure GitHub secrets, and apply Postgres roles.

---

## 7. Out of Scope

- No requirement to deploy Airbyte or Postgres to a cloud provider; local or Docker is sufficient.  
- No specific BI tool setup; the “BI tool” role is for Postgres permissions only.

Good luck.
