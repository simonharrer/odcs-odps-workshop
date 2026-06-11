# Exercise 5: Implement Your Data Product

In [Exercise 4](exercise4-design-your-data-product.md) you designed the contract — now fulfill it: a SQL view `analytics.sku_sales_per_year` on top of the `orders_v2` schema that makes your contract tests pass.

This is the perfect task for an **AI coding agent**: your data contract is machine-readable metadata that specifies exactly what to build, and `datacontract test` gives the agent a feedback loop to verify its work.

_Where we are — the contract is designed, but the view does not exist yet (dashed):_

```mermaid
flowchart TB
    classDef tbl fill:#dcfce7,stroke:#16a34a,color:#14532d;
    classDef con fill:#dbeafe,stroke:#2563eb,color:#1e3a5f;
    classDef ret fill:#f1f5f9,stroke:#94a3b8,color:#64748b;
    classDef port fill:#ede9fe,stroke:#7c3aed,color:#4c1d95;
    classDef plan fill:#fefce8,stroke:#a8a29e,color:#78716c,stroke-dasharray:4 3;
    subgraph DPO["📦 Data Product · Orders"]
        rv1["orders_v1 · retired"]:::ret
        p2(["output port · orders_v2"]):::port
        subgraph DC2["📄 orders_v2"]
            o2["orders"]:::con
            l2["line_items"]:::con
        end
    end
    subgraph DPS["📦 Data Product · SKU Sales · draft"]
        ip(["input port · orders_v2"]):::port
        op(["output port · sku_sales_per_year"]):::port
        subgraph SC["📄 sku_sales_per_year · draft"]
            ss["sku_sales_per_year"]:::con
        end
    end
    subgraph PG["🐘 PostgreSQL"]
        subgraph S2["schema: orders_v2"]
            to2[("orders")]:::tbl
            tl2[("line_items")]:::tbl
        end
        subgraph SAN["schema: analytics"]
            vss[/"sku_sales_per_year · not built yet"/]:::plan
        end
    end
    p2 --> DC2
    DC2 -. describes .-> S2
    ip -. consumes .-> p2
    op --> SC
    SC -. specifies .-> vss
    style DPO fill:#faf5ff,stroke:#7c3aed
    style DPS fill:#faf5ff,stroke:#7c3aed
    style DC2 fill:#eff6ff,stroke:#2563eb
    style SC fill:#eff6ff,stroke:#2563eb
    style PG fill:#ffffff,stroke:#64748b
    style S2 fill:#f0fdf4,stroke:#16a34a
    style SAN fill:#fffbeb,stroke:#a8a29e
```


## Implement with an AI Coding Agent

1. Start your AI coding agent of choice (Claude Code, Codex, Copilot, ...) in the repository and prompt it, for example:

   > Implement a PostgreSQL view that fulfills the data contract in `sku_sales_per_year.odcs.yaml`. The source data is described by the contract `orders_v2.odcs.yaml`. Apply it to the database with `docker compose exec postgres psql -U workshop -d workshop`. Then verify with `datacontract test sku_sales_per_year.odcs.yaml` (username and password are `workshop`) and iterate until all tests pass.

2. Watch what the agent does:

   - Does it read both contracts — yours for the target, `orders_v2` for the source?
   - Does it run the tests and react to failures?
   - The repository tells agents not to peek into `solutions/` — it has to work from the contract, just like a real engineer would.


## Or Implement Manually

You can also do it yourself:

```sql
CREATE SCHEMA IF NOT EXISTS analytics;

CREATE OR REPLACE VIEW analytics.sku_sales_per_year AS
SELECT
    -- your transformation here
FROM orders_v2.line_items li
JOIN orders_v2.orders o ON li.order_id = o.order_id
GROUP BY ...;
```

> [!TIP]
> `EXTRACT(YEAR FROM order_timestamp)` returns a `numeric` in PostgreSQL — cast it with `::int`. The same goes for `SUM(quantity)`, which returns `numeric` — cast it with `::bigint`. Types matter: they are part of your contract!


## Go Live

3. Make sure the tests are green:

   ```bash
   datacontract test sku_sales_per_year.odcs.yaml
   ```

4. Your data product is live — set the `status` to `active` in both the [sku_sales_per_year.odcs.yaml](../../sku_sales_per_year.odcs.yaml) and the [sku_sales_per_year.odps.yaml](../../sku_sales_per_year.odps.yaml)!

_After this exercise — the analytics view is live and reads the `orders_v2` tables directly (thick edges = the broad dependency Exercise 6 will narrow):_

```mermaid
flowchart TB
    classDef tbl fill:#dcfce7,stroke:#16a34a,color:#14532d;
    classDef view fill:#fef9c3,stroke:#ca8a04,color:#713f12;
    classDef con fill:#dbeafe,stroke:#2563eb,color:#1e3a5f;
    classDef ret fill:#f1f5f9,stroke:#94a3b8,color:#64748b;
    classDef port fill:#ede9fe,stroke:#7c3aed,color:#4c1d95;
    subgraph DPO["📦 Data Product · Orders"]
        rv1["orders_v1 · retired"]:::ret
        p2(["output port · orders_v2"]):::port
        subgraph DC2["📄 orders_v2"]
            o2["orders"]:::con
            l2["line_items"]:::con
        end
    end
    subgraph DPS["📦 Data Product · SKU Sales · active"]
        ip(["input port · orders_v2"]):::port
        op(["output port · sku_sales_per_year"]):::port
        subgraph SC["📄 sku_sales_per_year · active"]
            ss["sku_sales_per_year"]:::con
        end
    end
    subgraph PG["🐘 PostgreSQL"]
        subgraph S2["schema: orders_v2"]
            to2[("orders")]:::tbl
            tl2[("line_items")]:::tbl
        end
        subgraph SAN["schema: analytics"]
            vss[/"sku_sales_per_year"/]:::view
        end
    end
    p2 --> DC2
    DC2 -. describes .-> S2
    ip -. consumes .-> p2
    op --> SC
    SC -. describes .-> vss
    vss == reads ==> to2
    vss == reads ==> tl2
    style DPO fill:#faf5ff,stroke:#7c3aed
    style DPS fill:#faf5ff,stroke:#7c3aed
    style DC2 fill:#eff6ff,stroke:#2563eb
    style SC fill:#eff6ff,stroke:#2563eb
    style PG fill:#ffffff,stroke:#64748b
    style S2 fill:#f0fdf4,stroke:#16a34a
    style SAN fill:#fffbeb,stroke:#ca8a04
```
