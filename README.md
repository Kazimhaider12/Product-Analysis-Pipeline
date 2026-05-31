# Product Analytics Pipeline

An end-to-end data analytics project that extracts product data from a public API, stores raw JSON data in PostgreSQL, transforms it using dbt, and visualizes business insights in Power BI.

This project demonstrates a complete modern analytics workflow from data extraction to reporting.

---

## Project Workflow

```text
Python API Extraction
        |
        v
PostgreSQL Raw Layer
        |
        v
dbt Data Transformation
        |
        v
PostgreSQL Analytics Layer
        |
        v
Power BI Dashboard
```

---


---

## Project Objective

The objective of this project is to build a practical analytics pipeline that converts raw product API data into clean, structured, and dashboard-ready business insights.

The pipeline is designed to answer questions such as:

- How many products are available?
- Which categories contain the most products?
- What are the pricing trends across products and categories?
- Which products have low stock?
- What is the rating performance by category?
- Which inventory items require business attention?

---

## Tech Stack

- Python
- PostgreSQL
- Docker
- dbt
- Power BI
- DummyJSON Products API

---

## Data Source

This project uses the free public DummyJSON Products API:

```text
https://dummyjson.com/products
```

The API provides product-level data such as:

- Product ID
- Product title
- Category
- Brand
- Price
- Discount percentage
- Rating
- Stock
- Availability status
- Images and thumbnails
- Product description

---

## Project Architecture

```text
API Source
   |
   |  Python requests
   v
Raw JSON Snapshot
   |
   |  psycopg2 load
   v
PostgreSQL: raw.raw_products
   |
   |  dbt source + staging model
   v
PostgreSQL: staging.stg_products
   |
   |  dbt marts
   v
PostgreSQL Analytics Tables
   |
   |  Power BI connection
   v
Interactive Dashboard
```

---

## Project Structure

```text
.
├── scripts/
│   ├── fetch_products.py
│   ├── load_products_to_postgres.py
│   ├── run_pipeline.py
│   └── config.py
│
├── dbt/
│   ├── dbt_project.yml
│   ├── profiles.example.yml
│   ├── profiles.yml
│   ├── macros/
│   │   └── generate_schema_name.sql
│   └── models/
│       ├── staging/
│       │   ├── sources.yml
│       │   └── stg_products.sql
│       └── marts/
│           ├── dim_products.sql
│           ├── category_summary.sql
│           └── inventory_alerts.sql
│
├── docs/
│   ├── product_analytics_pipeline_full_workflow.gif
│   └── looker_studio_dashboard_plan.md
│
├── sql/
│   └── 001_create_schemas.sql
│
├── docker-compose.yml
├── requirements.txt
├── .env.example
└── README.md
```

---

## Pipeline Steps

### 1. Python API Extraction

The Python extraction script fetches product data from the API using `requests`.

```python
response = requests.get(
    PRODUCTS_API_URL,
    params={"limit": 100, "skip": skip},
    timeout=30,
)

payload = response.json()
products.extend(payload["products"])
```

The extracted data is saved as a raw JSON snapshot.

```text
data/products_raw.json
```

---

### 2. PostgreSQL Raw Layer

Raw API data is loaded into PostgreSQL as JSONB.

```sql
create table raw.raw_products (
    product_id integer primary key,
    source text not null,
    fetched_at timestamptz not null,
    raw_payload jsonb not null,
    loaded_at timestamptz not null default now()
);
```

This raw layer keeps the original API payload for traceability and auditing.

---

### 3. dbt Transformation

dbt is used to transform raw JSON fields into clean structured columns.

Example from the staging model:

```sql
select
    product_id,
    raw_payload ->> 'title' as product_name,
    raw_payload ->> 'category' as category,
    raw_payload ->> 'brand' as brand,
    nullif(raw_payload ->> 'price', '')::numeric(12, 2) as price,
    nullif(raw_payload ->> 'discountPercentage', '')::numeric(8, 2) as discount_percentage,
    nullif(raw_payload ->> 'rating', '')::numeric(4, 2) as rating,
    nullif(raw_payload ->> 'stock', '')::integer as stock,
    raw_payload ->> 'availabilityStatus' as availability_status
from {{ source('raw', 'raw_products') }}
```

---

### 4. PostgreSQL Analytics Layer

dbt creates analytics-ready tables inside PostgreSQL.

Final analytics tables:

```text
analytics.dim_products
analytics.category_summary
analytics.inventory_alerts
```

These tables are used directly in Power BI.

---

### 5. Power BI Dashboard

Power BI connects to PostgreSQL and visualizes the final analytics tables.

The report includes:

- Product performance overview
- Category analysis
- Price analysis
- Rating analysis
- Stock and inventory monitoring
- Low stock and out-of-stock alerts

---

## Final Output

The completed pipeline produced:

| Metric | Result |
|---|---:|
| Total Products Processed | 194 |
| Product Categories Analyzed | 24 |
| Inventory Alert Products | 28 |
| Main Analytics Tables | 3 |

---

## Analytics Tables

### `analytics.dim_products`

Product-level analytics table.

Main columns:

- product_id
- product_name
- category
- brand
- price
- discount_percentage
- discounted_price
- rating
- stock
- availability_status
- thumbnail_url

---

### `analytics.category_summary`

Category-level summary table.

Main columns:

- category
- product_count
- avg_price
- min_price
- max_price
- avg_rating
- total_stock

---

### `analytics.inventory_alerts`

Inventory monitoring table.

This table identifies products with low stock or unavailable inventory status.

Main columns:

- product_id
- product_name
- category
- brand
- stock
- availability_status
- price
- rating

---

## Setup Instructions

### 1. Clone the Repository

```bash
git clone <your-repository-url>
cd product-analytics-pipeline
```

---

### 2. Create Environment File

Copy the example environment file:

```bash
cp .env.example .env
```

For Windows PowerShell:

```powershell
Copy-Item .env.example .env
```

---

### 3. Install Python Dependencies

```bash
pip install -r requirements.txt
```

On this machine, the verified Python command was:

```powershell
& 'C:\Users\THISLAPTOP\AppData\Local\Programs\Python\Python312\python.exe' -m pip install -r requirements.txt
```

---

### 4. Start PostgreSQL with Docker

```bash
docker compose up -d
```

If Docker is not available in PATH on Windows, use:

```powershell
& 'C:\Program Files\Docker\Docker\resources\bin\docker.exe' compose up -d
```

---

### 5. Run Python Extraction and Load Pipeline

```bash
python -m scripts.run_pipeline
```

Verified Windows command:

```powershell
& 'C:\Users\THISLAPTOP\AppData\Local\Programs\Python\Python312\python.exe' -m scripts.run_pipeline
```

This command:

- Fetches product data from the API
- Saves a raw JSON snapshot
- Creates the PostgreSQL raw table
- Loads the data into `raw.raw_products`

---

### 6. Run dbt Models

Go to the dbt folder:

```bash
cd dbt
```

Run dbt:

```bash
dbt run --profiles-dir .
dbt test --profiles-dir .
```

Verified Windows command:

```powershell
& 'C:\Users\THISLAPTOP\AppData\Local\Programs\Python\Python312\Scripts\dbt.exe' run --profiles-dir .
& 'C:\Users\THISLAPTOP\AppData\Local\Programs\Python\Python312\Scripts\dbt.exe' test --profiles-dir .
```

---

## PostgreSQL Commands

### View Analytics Tables

```sql
select table_schema, table_name
from information_schema.tables
where table_schema = 'analytics'
order by table_name;
```

Expected tables:

```text
analytics.category_summary
analytics.dim_products
analytics.inventory_alerts
```

---

### Check Row Counts

```sql
select 'dim_products' as table_name, count(*) from analytics.dim_products
union all
select 'category_summary', count(*) from analytics.category_summary
union all
select 'inventory_alerts', count(*) from analytics.inventory_alerts;
```

---

### Preview Product Data

```sql
select
    product_id,
    product_name,
    category,
    brand,
    price,
    rating,
    stock
from analytics.dim_products
limit 10;
```

---

## Power BI Connection

Use these connection details in Power BI Desktop:

```text
Server: localhost:5432
Database: ecommerce_analytics
Username: postgres
Password: postgres
```

Then select the following tables:

```text
analytics.dim_products
analytics.category_summary
analytics.inventory_alerts
```

---

## Why This Project Is Important

This project is important because it shows how raw API data can be converted into useful business insights through a complete analytics workflow.

In real-world businesses, data is often collected from different systems in raw formats. Before it can be used for decision-making, it needs to be extracted, stored, cleaned, modeled, validated, and visualized.

This project demonstrates those key steps using industry-relevant tools:

- Python for data extraction
- PostgreSQL for data storage
- dbt for transformation and testing
- Power BI for business reporting

It represents a practical data engineering and business intelligence workflow that can be applied to real analytics projects.

---

## Skills Demonstrated

- API data extraction
- Python scripting
- PostgreSQL database design
- JSONB handling in PostgreSQL
- dbt modeling
- Data transformation
- Data validation with dbt tests
- Analytics table design
- Power BI dashboard development
- End-to-end data pipeline development

---

## Project Outcome

The final Power BI dashboard provides clear business insights into product performance, pricing, categories, ratings, and inventory status.

The project successfully converts API data into analytics-ready reporting tables and demonstrates a complete path from raw data to business intelligence.

---

## Author

**Kazim Haider Syed**
