# Incremental `fct_orders` Merge Workflow

## Purpose

This note documents the manual workflow used to test the `fct_orders` incremental model with a Snowflake-supported `merge` strategy.

The model is configured to use:

```sql
{{
    config(
        materialized='incremental',
        unique_key='order_id',
        incremental_strategy='merge',
        on_schema_change='fail'
    )
}}
```

The aim is to show how new source rows are inserted into the raw tables, then processed into the incremental `fct_orders` model.

## Workflow

### Step 1: Run the model before inserting new source data

Run this in the terminal:

```bash
dbt run --select fct_orders
```

### Step 2: Confirm the new orders are not yet in the target model

Run this in Snowflake:

```sql
SELECT *
FROM analytics.dbt_rait.fct_orders
WHERE order_id >= 100
ORDER BY order_id;
```

This should return no rows before the new source data has been inserted and processed.

### Step 3: Insert new source rows into Snowflake

Run this in Snowflake:

```sql
INSERT INTO raw.jaffle_shop.orders VALUES
    (100, 100, '2025-02-15', 'shipped', CURRENT_TIMESTAMP),
    (101, 84, '2025-02-15', 'shipped', CURRENT_TIMESTAMP),
    (102, 42, '2025-02-15', 'shipped', CURRENT_TIMESTAMP),
    (103, 101, '2025-02-15', 'shipped', CURRENT_TIMESTAMP),
    (104, 66, '2025-02-15', 'shipped', CURRENT_TIMESTAMP);

INSERT INTO raw.jaffle_shop.customers VALUES
    (101, 'Michelle', 'B.'),
    (102, 'Faith', 'L.');

INSERT INTO raw.stripe.payment VALUES
    (121, 100, 'bank_transfer', 'success', 1000, '2025-02-14', CURRENT_TIMESTAMP),
    (122, 101, 'credit_card', 'fail', 400, '2025-02-14', CURRENT_TIMESTAMP),
    (123, 102, 'credit_card', 'success', 1900, '2025-02-14', CURRENT_TIMESTAMP),
    (124, 103, 'credit_card', 'success', 1000, '2025-02-15', CURRENT_TIMESTAMP),
    (125, 104, 'coupon', 'success', 100, '2025-02-15', CURRENT_TIMESTAMP);
```

Only run these insert statements once unless the source tables have been reset.

### Step 4: Preview the incremental slice before running the model

Run this in the terminal:

```bash
dbt show --select fct_orders --limit 20
```

This previews the rows returned by the model SQL. Because the model uses `is_incremental()`, this should show the rows that currently pass the incremental filter.

### Step 5: Run the incremental model again

Run this in the terminal:

```bash
dbt run --select fct_orders
```

This applies the incremental `merge` into the target `fct_orders` table.

### Step 6: Confirm the new rows are now in the target model

Run this in Snowflake:

```sql
SELECT *
FROM analytics.dbt_rait.fct_orders
WHERE order_id >= 100
ORDER BY order_id;
```

Expected result:

```text
order_id  customer_id  order_date   amount
100       100          2025-02-15   10
101       84           2025-02-15   0
102       42           2025-02-15   19
103       101          2025-02-15   10
104       66           2025-02-15   1
```

## Notes

The `merge` strategy uses `order_id` as the `unique_key`.

This means dbt can match incoming records to existing records in the target table.

The merge can do both:

```text
If order_id does not already exist in analytics.dbt_rait.fct_orders:
    insert the row

If order_id already exists in analytics.dbt_rait.fct_orders:
    update the existing row
```

SQL equivalent:

```sql
MERGE INTO analytics.dbt_rait.fct_orders AS target
USING incremental_source_rows AS source
    ON target.order_id = source.order_id

WHEN MATCHED THEN UPDATE SET
    customer_id = source.customer_id,
    order_date = source.order_date,
    amount = source.amount

WHEN NOT MATCHED THEN INSERT (
    order_id,
    customer_id,
    order_date,
    amount
)
VALUES (
    source.order_id,
    source.customer_id,
    source.order_date,
    source.amount
);
```

In this practice workflow, orders `100` to `104` did not already exist in `fct_orders`, so the merge mainly inserts those new rows.

If one of those `order_id` values already existed, dbt would use the merge strategy to update the matching row instead of appending a duplicate.

Simple comparison:

```text
append = always add rows
merge = add new rows and update matching existing rows
```

This is safer than a simple append strategy when records may arrive again or be updated later.
