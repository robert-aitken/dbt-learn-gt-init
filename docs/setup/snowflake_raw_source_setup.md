# Snowflake Raw Source Setup

## Purpose

This note documents the Snowflake setup used to create the raw source tables for the dbt learning project.

It creates:

- a Snowflake warehouse
- raw and analytics databases
- raw source schemas
- raw Jaffle Shop and Stripe source tables
- sample source data loaded from public S3 CSV files

This setup supports the dbt Fundamentals and Incremental Models practice work.

## Caution

This is a learning/demo setup script.

Review object names before running this in a shared Snowflake account, especially:

- warehouse name
- database names
- schema names
- table names

## Setup SQL

Run this in Snowflake.

```sql
CREATE WAREHOUSE transforming;

CREATE DATABASE raw;

CREATE DATABASE analytics;

CREATE SCHEMA raw.jaffle_shop;

CREATE SCHEMA raw.stripe;

CREATE TABLE raw.jaffle_shop.customers (
    id INTEGER,
    first_name VARCHAR,
    last_name VARCHAR
);

COPY INTO raw.jaffle_shop.customers (
    id,
    first_name,
    last_name
)
FROM 's3://dbt-tutorial-public/jaffle_shop_customers.csv'
FILE_FORMAT = (
    TYPE = 'CSV'
    FIELD_DELIMITER = ','
    SKIP_HEADER = 1
);

CREATE TABLE raw.jaffle_shop.orders (
    id INTEGER,
    user_id INTEGER,
    order_date DATE,
    status VARCHAR,
    _etl_loaded_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

COPY INTO raw.jaffle_shop.orders (
    id,
    user_id,
    order_date,
    status
)
FROM 's3://dbt-tutorial-public/jaffle_shop_orders.csv'
FILE_FORMAT = (
    TYPE = 'CSV'
    FIELD_DELIMITER = ','
    SKIP_HEADER = 1
);

CREATE TABLE raw.stripe.payment (
    id INTEGER,
    orderid INTEGER,
    paymentmethod VARCHAR,
    status VARCHAR,
    amount INTEGER,
    created DATE,
    _batched_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

COPY INTO raw.stripe.payment (
    id,
    orderid,
    paymentmethod,
    status,
    amount,
    created
)
FROM 's3://dbt-tutorial-public/stripe_payments.csv'
FILE_FORMAT = (
    TYPE = 'CSV'
    FIELD_DELIMITER = ','
    SKIP_HEADER = 1
);
```

## Check the loaded source data

Run these checks in Snowflake.

```sql
SELECT *
FROM raw.jaffle_shop.customers;

SELECT *
FROM raw.jaffle_shop.orders;

SELECT *
FROM raw.stripe.payment;
```

## Expected purpose of each source table

| Source table | Purpose |
|---|---|
| `raw.jaffle_shop.customers` | Raw customer records |
| `raw.jaffle_shop.orders` | Raw order records |
| `raw.stripe.payment` | Raw payment records |

## Notes

The raw tables are used by dbt source definitions and staging models.

The `analytics` database is used as the target database for transformed dbt models.

The `transforming` warehouse is used to run dbt transformations.