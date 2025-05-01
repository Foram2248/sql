# Assignment 2: Design a Logical Model and Advanced SQL

🚨 **Please review our [Assignment Submission Guide](https://github.com/UofT-DSI/onboarding/blob/main/onboarding_documents/submissions.md)** 🚨 for detailed instructions on how to format, branch, and submit your work. Following these guidelines is crucial for your submissions to be evaluated correctly.

#### Submission Parameters:

- Submission Due Date: `April 27, 2025`
- Weight: 70% of total grade
- The branch name for your repo should be: `assignment-two`
- What to submit for this assignment:
  - This markdown (Assignment2.md) with written responses in Section 1
  - Two Entity-Relationship Diagrams (preferably in a pdf, jpeg, png format).
  - One .sql file
- What the pull request link should look like for this assignment: `https://github.com/<your_github_username>/sql/pulls/<pr_id>`
  - Open a private window in your browser. Copy and paste the link to your pull request into the address bar. Make sure you can see your pull request properly. This helps the technical facilitator and learning support staff review your submission easily.

Checklist:

- [ ] Create a branch called `assignment-two`.
- [ ] Ensure that the repository is public.
- [ ] Review [the PR description guidelines](https://github.com/UofT-DSI/onboarding/blob/main/onboarding_documents/submissions.md#guidelines-for-pull-request-descriptions) and adhere to them.
- [ ] Verify that the link is accessible in a private browser window.

If you encounter any difficulties or have questions, please don't hesitate to reach out to our team via our Slack at `#cohort-6-help`. Our Technical Facilitators and Learning Support staff are here to help you navigate any challenges.

---

## Section 1:

You can start this section following _session 1_, but you may want to wait until you feel comfortable wtih basic SQL query writing.

Steps to complete this part of the assignment:

- Design a logical data model
- Duplicate the logical data model and add another table to it following the instructions
- Write, within this markdown file, an answer to Prompt 3

### Design a Logical Model

#### Prompt 1

Design a logical model for a small bookstore. 📚

At the minimum it should have employee, order, sales, customer, and book entities (tables). Determine sensible column and table design based on what you know about these concepts. Keep it simple, but work out sensible relationships to keep tables reasonably sized.

Additionally, include a date table.

There are several tools online you can use, I'd recommend [Draw.io](https://www.drawio.com/) or [LucidChart](https://www.lucidchart.com/pages/).

**HINT:** You do not need to create any data for this prompt. This is a conceptual model only.

#### Prompt 2

We want to create employee shifts, splitting up the day into morning and evening. Add this to the ERD.

#### Prompt 3

The store wants to keep customer addresses. Propose two architectures for the CUSTOMER_ADDRESS table, one that will retain changes, and another that will overwrite. Which is type 1, which is type 2?

**HINT:** search type 1 vs type 2 slowly changing dimensions.

```
**Architecture 1: Overwriting Changes**
In this architecture, the customer's address is stored directly within the `CUSTOMER` table. When a customer updates their address, the existing `customer_address` field for that customer is simply updated with the new information. No historical record of previous addresses is kept.
**Architecture 2: Retaining Changes**
This architecture involves creating a separate `CUSTOMER_ADDRESS` table to store address history. Each time a customer's address changes, a new record is added to this table. Each record would include the address, the date it became valid, and potentially the date it ceased to be valid. A flag could also indicate the current address.
For example, the `CUSTOMER_ADDRESS` table might have columns like: `customer_id` (foreign key), `address`, `valid_from`, `valid_to`, and `is_current`. When an address changes, the `valid_to` date of the previous current address is updated, and a new record with the new address and a new `valid_from` date is inserted as the current address.
**Type Identification:**

- **Architecture 1 (Overwriting Changes)** corresponds to a **Type 1 Slowly Changing Dimension**. In Type 1, when an attribute changes, the old value is overwritten with the new value, and historical data is lost.
- **Architecture 2 (Retaining Changes)** corresponds to a **Type 2 Slowly Changing Dimension**. Type 2 SCDs preserve historical data by creating a new record whenever a tracked attribute changes.

```

---

## Section 2:

You can start this section following _session 4_.

Steps to complete this part of the assignment:

- Open the assignment2.sql file in DB Browser for SQLite:
  - from [Github](./02_activities/assignments/assignment2.sql)
  - or, from your local forked repository
- Complete each question

### Write SQL

#### COALESCE

1. Our favourite manager wants a detailed long list of products, but is afraid of tables! We tell them, no problem! We can produce a list with all of the appropriate details.

Using the following syntax you create our super cool and not at all needy manager a list:

```

SELECT \*
FROM product
WHERE product_name IS NULL OR product_size IS NULL OR product_qty_type IS NULL;
SELECT
product_name || ', ' ||
COALESCE(product_size, '') || ' (' ||
COALESCE(product_qty_type, 'unit') || ')'
FROM product;

```

But wait! The product table has some bad data (a few NULL values).
Find the NULLs and then using COALESCE, replace the NULL with a blank for the first problem, and 'unit' for the second problem.

**HINT**: keep the syntax the same, but edited the correct components with the string. The `||` values concatenate the columns into strings. Edit the appropriate columns -- you're making two edits -- and the NULL rows will be fixed. All the other rows will remain the same.

<div align="center">-</div>

#### Windowed Functions

1. Write a query that selects from the customer_purchases table and numbers each customer’s visits to the farmer’s market (labeling each market date with a different number). Each customer’s first visit is labeled 1, second visit is labeled 2, etc.

You can either display all rows in the customer_purchases table, with the counter changing on each new market date for each customer, or select only the unique market dates per customer (without purchase details) and number those visits.

**HINT**: One of these approaches uses ROW_NUMBER() and one uses DENSE_RANK().

```

SELECT
customer_id,
market_date,
ROW_NUMBER() OVER( PARTITION BY customer_id ORDER BY market_date) AS customer_visit_number
FROM customer_purchases;
--A customer could have made multiple purachases on the same market visit and to make it count as one we use dense_rank
SELECT DISTINCT
customer_id,
market_date,
DENSE_RANK() OVER (PARTITION BY customer_id ORDER BY market_date) AS customer_visit_numer
FROM customer_purchases;

```

2. Reverse the numbering of the query from a part so each customer’s most recent visit is labeled 1, then write another query that uses this one as a subquery (or temp table) and filters the results to only the customer’s most recent visit.

```

WITH RankedVisits AS (
SELECT DISTINCT
customer_id,
market_date,
DENSE_RANK() OVER (PARTITION BY customer_id ORDER BY market_date DESC) AS customer_visit_reversed_rank
FROM customer_purchases
)
SELECT
customer_id,
market_date
FROM RankedVisits
WHERE customer_visit_reversed_rank = 1;

```

3. Using a COUNT() window function, include a value along with each row of the customer_purchases table that indicates how many different times that customer has purchased that product_id.

```

--Solution
-- I could see two answers for this or two ways of interpreting with DISTINCT and without DISTINCT
--With DISTINCT it just reduces the rows, without it shows all rows for each purchase row.
--since it closely resembles the question I have left out DISTINCT
SELECT
customer_id,
product_id,
COUNT(\*) OVER( PARTITION BY customer_id, product_id ) AS purchase_count
FROM customer_purchases;

```

<div align="center">-</div>

#### String manipulations

1. Some product names in the product table have descriptions like "Jar" or "Organic". These are separated from the product name with a hyphen. Create a column using SUBSTR (and a couple of other commands) that captures these, but is otherwise NULL. Remove any trailing or leading whitespaces. Don't just use a case statement for each product!

| product_name               | description |
| -------------------------- | ----------- |
| Habanero Peppers - Organic | Organic     |

**HINT**: you might need to use INSTR(product_name,'-') to find the hyphens. INSTR will help split the column.

```

SELECT
product_name,
CASE
WHEN INSTR(product_name, '-') > 0
THEN TRIM(SUBSTR(product_name, INSTR(product_name, '-') + 1))
ELSE NULL
END AS description
FROM product;

```

/_ 2. Filter the query to show any product_size value that contain a number with REGEXP. _/

```

SELECT
product_name,
product_size
FROM product
WHERE product_size REGEXP '[0-9]';

```

<div align="center">-</div>

#### UNION

1. Using a UNION, write a query that displays the market dates with the highest and lowest total sales.

**HINT**: There are a possibly a few ways to do this query, but if you're struggling, try the following: 1) Create a CTE/Temp Table to find sales values grouped dates; 2) Create another CTE/Temp table with a rank windowed function on the previous query to create "best day" and "worst day"; 3) Query the second temp table twice, once for the best day, once for the worst day, with a UNION binding them.

```

WITH DailyRevenue AS (
SELECT
market_date,
SUM(quantity \* cost_to_customer_per_qty) AS total_revenue
FROM customer_purchases
GROUP BY market_date
),
DailyRevenueRanked AS (
SELECT
market_date,
total_revenue,
RANK() OVER (ORDER BY total_revenue DESC) AS revenue_rank_desc,
RANK() OVER (ORDER BY total_revenue ASC)AS revenue_rank_asc
FROM DailyRevenue
)
SELECT
market_date,
total_revenue,
'Highest Revenue' AS revenue_category
FROM DailyRevenueRanked
WHERE revenue_rank_desc = 1
UNION
SELECT
market_date,
total_revenue,
'Lowest Revenue' AS revenue_category
FROM DailyRevenueRanked
WHERE revenue_rank_asc = 1;

```

---

## Section 3:

You can start this section following _session 5_.

Steps to complete this part of the assignment:

- Open the assignment2.sql file in DB Browser for SQLite:
  - from [Github](./02_activities/assignments/assignment2.sql)
  - or, from your local forked repository
- Complete each question

### Write SQL

#### Cross Join

1. Suppose every vendor in the `vendor_inventory` table had 5 of each of their products to sell to **every** customer on record. How much money would each vendor make per product? Show this by vendor_name and product name, rather than using the IDs.

**HINT**: Be sure you select only relevant columns and rows. Remember, CROSS JOIN will explode your table rows, so CROSS JOIN should likely be a subquery. Think a bit about the row counts: how many distinct vendors, product names are there (x)? How many customers are there (y). Before your final group by you should have the product of those two queries (x\*y).

```

SELECT
v.vendor*name,
p.product_name,
COUNT(c.customer_id) * 5 \_ vi.original_price AS total_revenue
FROM
vendor_inventory vi
JOIN
vendor v ON vi.vendor_id = v.vendor_id
JOIN
product p ON vi.product_id = p.product_id
CROSS JOIN
(SELECT customer_id FROM customer) c
GROUP BY
v.vendor_name, p.product_name, vi.original_price
ORDER BY
v.vendor_name, p.product_name;

```

<div align="center">-</div>

#### INSERT

1. Create a new table "product_units". This table will contain only products where the `product_qty_type = 'unit'`. It should use all of the columns from the product table, as well as a new column for the `CURRENT_TIMESTAMP`. Name the timestamp column `snapshot_timestamp`.

```

CREATE TABLE product_units AS
SELECT
\*,
CURRENT_TIMESTAMP AS snapshot_timestamp
FROM product
WHERE product_qty_type = 'unit';

```

2. Using `INSERT`, add a new row to the product_unit table (with an updated timestamp). This can be any product you desire (e.g. add another record for Apple Pie).

```

INSERT INTO product_units (
product_id,
product_name,
product_size,
product_category_id,
product_qty_type,
snapshot_timestamp
)
VALUES ( 99, 'Artisnal Chocolates', '5oz', 3, 'unit', CURRENT_TIMESTAMP);

```

2. Using `INSERT`, add a new row to the product_unit table (with an updated timestamp). This can be any product you desire (e.g. add another record for Apple Pie).

<div align="center">-</div>

#### DELETE

1. Delete the older record for the whatever product you added.

**HINT**: If you don't specify a WHERE clause, [you are going to have a bad time](https://imgflip.com/i/8iq872).

```

DELETE FROM product_units
WHERE product_id = 99;

```

<div align="center">-</div>

#### UPDATE

1. We want to add the current_quantity to the product_units table. First, add a new column, `current_quantity` to the table using the following syntax.

```

ALTER TABLE product_units
ADD current_quantity INT;

```

Then, using `UPDATE`, change the current_quantity equal to the **last** `quantity` value from the vendor_inventory details.

**HINT**: This one is pretty hard. First, determine how to get the "last" quantity per product. Second, coalesce null values to 0 (if you don't have null values, figure out how to rearrange your query so you do.) Third, `SET current_quantity = (...your select statement...)`, remembering that WHERE can only accommodate one column. Finally, make sure you have a WHERE statement to update the right row, you'll need to use `product_units.product_id` to refer to the correct row within the product_units table. When you have all of these components, you can run the update statement.

--Solution

```

UPDATE product_units
SET current_quantity = COALESCE((
SELECT ri.quantity
FROM (
SELECT
product_id,
quantity,
market_date,
ROW_NUMBER () OVER (PARTITION BY product_id ORDER BY market_date DESC) as rn
FROM vendor_inventory
) ri
WHERE ri.rn = 1
AND ri.product_id = product_units.product_id ), 0 );

```
