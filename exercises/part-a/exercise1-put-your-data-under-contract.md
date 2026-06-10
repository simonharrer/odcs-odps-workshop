# Exercise 1: Put Your Data Under Contract

You are the owner of the **orders** data in your company's e-commerce platform.
The data lives in a PostgreSQL database and consists of two tables: `orders` (containing order details like timestamps, totals, and customer information) and `line_items` (containing the individual items in each order, linked by `order_id`).
Your goal is to define a data contract so that consumers of your data know exactly what to expect.


## Install the CLI & Start the Database

1. Run `scripts/install.sh` script to get the Data Contract CLI.

   ```bash
   scripts/install.sh
   ```

   This might take a few minutes.

2. Start PostgreSQL with the preloaded data:

   ```
   docker compose up -d
   ```

3. Explore the data:

   ```
   docker compose exec postgres psql -U workshop -d workshop
   ```

   ```sql
   \dt orders_v1.*
   SELECT * FROM orders_v1.orders LIMIT 5;
   SELECT * FROM orders_v1.line_items LIMIT 5;
   ```

   You should see something like this:
   
   ```text
                  order_id               |    order_timestamp     | order_total |     customer_id      | customer_email_address 
   --------------------------------------+------------------------+-------------+----------------------+------------------------
   a8c38fec-2acd-4b55-883b-4b48572d4a26 | 2020-01-01 00:00:00+00 |       29747 | 6GSHKOZIEN           | test394@example.org
   9e44da97-4f72-4bcf-821a-9d9500d06651 | 2020-01-01 10:37:00+00 |       55156 | ZN661MOMVMQXRJ       | test4757@example.org
   8fc4621c-66ae-4031-91f1-5313beb9f541 | 2020-01-01 20:14:00+00 |       85365 | NF0PRHKQP9W9Q0MTC87P | test1991@example.org
   98d48daf-3532-4a59-b7c2-3777164bdc65 | 2020-01-02 06:51:00+00 |         265 | QG9ZQ32YAQKY         | test3491@example.org
   2fd9df43-77e8-4d00-b380-ab270e8b73f8 | 2020-01-02 16:28:00+00 |       18130 | LNJ3SQAMEY3ZY        | test6705@example.org
   (5 rows)
   ```
   
   ```text
               lines_item_id             |               order_id               |      sku      
   --------------------------------------+--------------------------------------+---------------
   94aa82c8-50ba-47fb-994a-9b041b4127af | a8c38fec-2acd-4b55-883b-4b48572d4a26 | D3KT74L5EV46T
   d67c963f-42a4-4aa8-afff-d7869008e3a9 | 9e44da97-4f72-4bcf-821a-9d9500d06651 | E202K62FT
   270ad2c1-f651-438e-a81a-d77713c1d3a3 | 8fc4621c-66ae-4031-91f1-5313beb9f541 | 1O7RID9Y5QJ
   cc763a72-cc07-4bc4-8ddf-c88d09db5daa | 98d48daf-3532-4a59-b7c2-3777164bdc65 | 7KJ8466FI39LW
   d7ea7f72-a266-469c-a9ef-60063d5ac243 | 2fd9df43-77e8-4d00-b380-ab270e8b73f8 | 7HXBABF0AOT5
   (5 rows)
   ```

   If you get no output at all, check for typos — `psql` returns nothing for a misspelled schema or table name.

   The raw data is also available as JSON files in [`data/orders_v1/`](/data/orders_v1/).


## Create the Contract

4. Go to [Data Contract Editor] and create a new data contract.
   Use the **Form** editor to get started.
   
   - **Name**: `Orders`
   - **ID**: `orders_v1`
   - **Version**: `1.0.0`
   - **Status**: `draft` (leave as is)
   
5. Go to **Servers** in the left navigation (or use the direct link [editor.datacontract.com/#/servers](https://editor.datacontract.com/#/servers)) and add a new server:
   
   - **Server**: `Orders`
   - **Type**: `postgres`
   - **Host**: `localhost`
   - **Port**: `5433`
   - **Database**: `workshop`
   - **Schema**: `orders_v1`
  
6. Go to Schemas and add two schemas with their properties:

   - Name: `orders`
   - Properties:
     
     | Property Name            | Logical Type | Physical Type |
     |--------------------------|--------------|---------------|
     | `order_id`               | `string`     | `TEXT`        |
     | `order_timestamp`        | `date`       | `TIMESTAMPTZ` | 
     | `order_total`            | `integer`    | `BIGINT`      |
     | `customer_id`            | `string`     | `TEXT`        |
     | `customer_email_address` | `string`     | `TEXT`        |

   - Name: `line_items`
   - Properties:

     | Property Name            | Logical Type | Physical Type |
     |--------------------------|--------------|---------------|
     | `lines_item_id`          | `string`     | `TEXT`        |
     | `order_id`               | `string`     | `TEXT`        |
     | `sku`                    | `string`     | `TEXT`        |

7. Save the contract locally as `orders_v1.odcs.yaml` by clicking the **Save** button on the top right.
   Place it into the workshop repository folder (move it there if your browser saved it to Downloads).
   
   > **Tip:** You might want to keep the tab open for later.


## Test the Contract

Testing checks your contract against the *real* database: it confirms the tables, columns, and types you described actually exist as specified — catching any drift between the contract and reality.

8. Your Data Contract CLI should be installed by now.
   This should work:

   ```bash
   datacontract --version
   ```

9. Create an `.env` file to provide the database credentials to the Data Contract CLI:
   
   ```bash
   cp .env.example .env
   ```

   The CLI picks up the `.env` file automatically (since version `1.0.1`) when run from this folder.

10. Run the CLI on your data contract:

   ```bash
   datacontract test orders_v1.odcs.yaml
   ```
   
   This will show a list of checks that should all pass.
   If not, fix what doesn't.
   If all checks fail, make sure that the database container is running.

11. Verify tests can also fail:
    In your [orders_v1.odcs.yaml](../../orders_v1.odcs.yaml) change `physicalType` of `customer_email_address` to `integer`, then run the tests again.
    Try other mistakes and see how the output of the CLI changes.
    Revert afterward.
 

## Enrich the Contract

Make sure all tests pass before continuing. From here you can enrich the contract in two ways — **pick whichever suits you**:

- **In the [Data Contract Editor]** — great for discovering the available fields and options. Reopen your contract (menu, top right) if you closed the tab, and **Save** to re-download the file after each change.
- **Locally in your IDE** — faster once you know the fields. Edit `orders_v1.odcs.yaml` directly (see [Open Data Contract Standard] for reference), or let an AI agent help.

> [!NOTE]
> The editor and the file on disk don't sync automatically. If you switch sides, re-import the file into the editor (or re-download from it) first — otherwise you'll overwrite your changes.

> [!TIP]
> You can get the best of both worlds with the **edit** command:
>
> ```bash
> datacontract edit orders_v1.odcs.yaml
> ```
>
> It starts the Data Contract Editor locally for that file — saving in the editor writes directly back to the file on disk, so there's nothing to re-import or re-download.

The steps below name the editor's form fields, but each one is just a key in the YAML, so do whichever is faster for you.

Let's add some more detail to the contract...

12. Go to **Terms of Use** and add
   
   - a **Description**,
   - a **Purpose**, and
   - **Limitations**.
  
   Feel free to use the ✨AI buttons to generate this.

13. In the **Fundamentals**, add **Tags** like `orders`, or `ecommerce`.

14. In your **Schemas** edit `orders.customer_email_address`:
    - add **Examples** based on the data, e.g., `test394@example.org`,
    - set **Classification & Security** → **Classification**: `confidential` (it's personally identifiable information!), and
    - set **Constraints** → **Required** to `true`

15. Save the contract file and find the added metadata in it.
    
16. Optionally, look at the HTML export of the contract and see how your changes reflect in it:

    ```bash
    datacontract export html orders_v1.odcs.yaml --output orders_v1.odcs.html && open orders_v1.odcs.html
    ```

17. Run the test again.

    ```bash
    datacontract test orders_v1.odcs.yaml
    ```
    
    Can you spot the new check?


## Define Relationships

18. Add a `relationship` from `line_items.order_id` to `orders.order_id` to express the foreign key.
    This tells consumers the two tables can be joined safely and documents the referential integrity between them.

    Add it on the `order_id` **property** inside the `line_items` schema (next to its `logicalType`/`physicalType`):

    ```yaml
    schema:
      # ...
      - name: line_items
        properties:
          # ...
          - name: order_id
            logicalType: string
            physicalType: TEXT
            relationships:
              - type: foreignKey
                to: orders.order_id
          # ...
    ```

## Add Quality Checks

19. Add a SQL quality check to ensure that `customer_email_address` contains an `@` sign (find invalid rows).

    Add it on the `customer_email_address` **property** inside the `orders` schema:

    ```yaml
    schema:
      # ...
      - name: orders
        properties:
          # ...
          - name: customer_email_address
            # ... (type, examples, classification, ...)
            quality:
              - type: sql
                description: Ensure email addresses are valid
                query: SELECT COUNT(*) FROM orders_v1.orders WHERE customer_email_address NOT LIKE '%@%';
                mustBe: 0
          # ...
    ```
    
20. Test the contract again and spot the new check added by this.

21. Add more constraints (if time allows), e.g.:
    
    - Every `order_id` in `line_items` must exist in `orders`
    - `order_total` must be greater than or equal to 0
    - Think of your own additional constraints...

    Reference for quality checks:

    ```yaml
    properties:
      - name: field
        quality:
          - type: text
            description: Ensure that ...
          - type: sql
            description: Ensure that ...
            query: SELECT COUNT(*) FROM ... WHERE ...;
            mustBe: 0
            # mustBeGreaterThan: 0
    ```

    Documentation: https://bitol-io.github.io/open-data-contract-standard/latest/data-quality/#sql


## Add Ownership

22. Add a `team` and `support` channel, so consumers know how to contact the owners and get help.
    These are **top-level** keys in the contract (not nested under a schema):

    ```yaml
    # ... (other top-level keys, e.g. schema, tags, description) ...
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


## Add SLA Properties

23. Define service-level agreements (SLAs), so consumers know how long data is retained and how fresh to expect it.
    Like `team`, `slaProperties` is a **top-level** key:

    ```yaml
    # ... (other top-level keys) ...
    slaProperties:
      - property: retention
        value: 1
        unit: y
        description: Order data retained for 1 year
      - property: frequency
        value: 1
        unit: d
        description: Data updated daily
    ```


## Set to Active

24. Your contract is complete — set `status` to `active`!


## Bonus

- Use the **export** command to create an HTML documentation of the data contract:
  
  ```bash
  datacontract export html orders_v1.odcs.yaml --output orders_v1.odcs.html
  ```
  
- Export to SQL DDL ([exports](https://cli.datacontract.com/#export)):
  
  ```bash
  datacontract export sql orders_v1.odcs.yaml
  ```
 
- Use the **catalog** command to create a data contract catalog:

  ```bash
  datacontract catalog
  ```


[Data Contract Editor]: <https://editor.datacontract.com>
[Open Data Contract Standard]: <https://bitol-io.github.io/open-data-contract-standard/latest/>