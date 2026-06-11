# Exercise 3: Describe Your Data Product

The data contracts from [Exercise 1](exercise1-put-your-data-under-contract.md) and [Exercise 2](exercise2-data-contract-evolution.md) describe the *interface* of your data.
But consumers also want to know about the **data product** behind it:
who owns it, what it is for, and which contracts it offers.
That is what the [Open Data Product Standard] (ODPS) is for.

Note that there is *one* data product — even though it currently offers *two* contract versions.
The product is the stable unit of ownership; its ports evolve.

_Where we are — two contracts, still just files on disk:_

```mermaid
flowchart TB
    classDef tbl fill:#dcfce7,stroke:#16a34a,color:#14532d;
    classDef con fill:#dbeafe,stroke:#2563eb,color:#1e3a5f;
    classDef ret fill:#f1f5f9,stroke:#94a3b8,color:#64748b;
    subgraph DC1["📄 orders_v1 · retired"]
        o1["orders"]:::ret
        l1["line_items"]:::ret
    end
    subgraph DC2["📄 orders_v2 · active"]
        o2["orders"]:::con
        l2["line_items · +quantity"]:::con
    end
    subgraph PG["🐘 PostgreSQL"]
        subgraph S1["schema: orders_v1"]
            to1[("orders")]:::tbl
            tl1[("line_items")]:::tbl
        end
        subgraph S2["schema: orders_v2"]
            to2[("orders")]:::tbl
            tl2[("line_items")]:::tbl
        end
    end
    o1 -. describes .-> to1
    l1 -. describes .-> tl1
    o2 -. describes .-> to2
    l2 -. describes .-> tl2
    style DC1 fill:#f8fafc,stroke:#94a3b8
    style DC2 fill:#eff6ff,stroke:#2563eb
    style PG fill:#ffffff,stroke:#64748b
    style S1 fill:#f8fafc,stroke:#94a3b8
    style S2 fill:#f0fdf4,stroke:#16a34a
```


## Create the Data Product

1. Create a new file `orders.odps.yaml` with this skeleton:

   ```yaml
   apiVersion: v1.0.0
   kind: DataProduct
   id: orders # snake_case of the name
   name: Orders
   version: 1.0.0 # the version of the data product, independent of the contract versions
   status: active
   domain: ecommerce
   description:
     purpose: # what is this data product for?
     limitations: # what should consumers know before using it?
   ```

   Replace the `purpose` and `limitations` comments with real text.

2. Add an **output port** per data contract.
   The `contractId` must match the `id` of the respective data contract.
   Entropy Data reads a display name and the server connection from `customProperties`:

   ```yaml
   outputPorts:
     - name: orders_v1
       description: Orders and line items tables in PostgreSQL (v1, superseded by v2)
       version: 1.0.0
       contractId: orders_v1
       customProperties:
         - property: displayName
           value: Orders v1
         - property: server
           value:
             type: postgres
             host: localhost
             port: 5433
             database: workshop
             schema: orders_v1
     - name: orders_v2
       description: # ...
       version: 2.0.0
       contractId: orders_v2
       customProperties:
         # ... displayName and server for v2
   ```

3. Add `team` and `support` — you can reuse what you defined in the contracts:

   ```yaml
   team:
     name: order_data_team
     members:
       - username: owner@example.com
         role: Owner

   support:
     - channel: "#order-data-help"
       url: https://example.slack.com/archives/order-data-help
       tool: slack
   ```

## Validate

4. Validate your data product description against the official JSON schema:

   ```bash
   uvx check-jsonschema --schemafile schemas/odps-json-schema-v1.0.0.json orders.odps.yaml
   ```

_After this exercise — one ODPS data product bundles both contracts, exposing each as an output port:_

```mermaid
flowchart TB
    classDef tbl fill:#dcfce7,stroke:#16a34a,color:#14532d;
    classDef con fill:#dbeafe,stroke:#2563eb,color:#1e3a5f;
    classDef ret fill:#f1f5f9,stroke:#94a3b8,color:#64748b;
    classDef port fill:#ede9fe,stroke:#7c3aed,color:#4c1d95;
    subgraph DP["📦 Data Product · Orders (ODPS)"]
        p1(["output port · orders_v1"]):::port
        p2(["output port · orders_v2"]):::port
        subgraph DC1["📄 orders_v1 · retired"]
            o1["orders"]:::ret
            l1["line_items"]:::ret
        end
        subgraph DC2["📄 orders_v2 · active"]
            o2["orders"]:::con
            l2["line_items"]:::con
        end
    end
    subgraph PG["🐘 PostgreSQL"]
        subgraph S1["schema: orders_v1"]
            to1[("orders")]:::tbl
            tl1[("line_items")]:::tbl
        end
        subgraph S2["schema: orders_v2"]
            to2[("orders")]:::tbl
            tl2[("line_items")]:::tbl
        end
    end
    p1 --> DC1
    p2 --> DC2
    DC1 -. describes .-> S1
    DC2 -. describes .-> S2
    style DP fill:#faf5ff,stroke:#7c3aed
    style DC1 fill:#f8fafc,stroke:#94a3b8
    style DC2 fill:#eff6ff,stroke:#2563eb
    style PG fill:#ffffff,stroke:#64748b
    style S1 fill:#f8fafc,stroke:#94a3b8
    style S2 fill:#f0fdf4,stroke:#16a34a
```

[Open Data Product Standard]: <https://bitol-io.github.io/open-data-product-standard/latest/>
