# SQL Style Guide
This guide outlines best practices and conventions for writing clean, maintainable, and efficient SQL code. By adhering to these guidelines, teams can improve code readability, simplify collaboration, and ensure consistency across projects.

## General Formatting
### Capitalization
- Use **lowercase** for SQL keywords (e.g., `select`, `from`, `where`).
- Use **UPPERCASE** for table and column names to make them stand out.
### Indentation
- Place each variable on its own line, indented under the keyword it belongs to.
- Keep SQL keywords (`select`, `from`, `where`) aligned to the left.
### Line Breaks
- Place each clause (`select`, `from`, `where`, etc.) on a new line.
- Align `and/or` in conditions under the where clause.
  
```sql
-- Good
select
    CUSTOMER_ID,
    CUSTOMER_NAME,
    CUSTOMER_EMAIL
from
    CUSTOMERS
where
    CITY = 'Chicago'
    and STATE = 'IL';

-- Bad
SELECT CUSTOMER_ID, CUSTOMER_NAME, CUSTOMER_EMAIL FROM CUSTOMERS WHERE CITY='Chicago' AND STATE='IL';
```

### White Space Around Symbols
- Always include white space around operators like `=`, `*`, `-`, and `+` to improve readability.
```sql
-- Good
select
    CUSTOMER_ID,
    CUSTOMER_NAME,
    ORDER_TOTAL * 1.05 as TOTAL_WITH_TAX
from
    ORDERS
where
    ORDER_TOTAL > 100
    and DISCOUNT = 0;

-- Bad
select
    CUSTOMER_ID,
    CUSTOMER_NAME,
    ORDER_TOTAL*1.05 as TOTAL_WITH_TAX
from
    ORDERS
where
    ORDER_TOTAL>100
    and DISCOUNT=0;
```

## Comments 
- Use -- for inline comments and place them on their own line.
- Comments should be used **minimally**, aligning with best coding practices. 
  - If the code is clear, concise, and explains the logic straightforwardly, additional comments are unnecessary.
- Use comments to explain **methodology** or **major process steps**, not to restate what the code is doing.

```sql
-- Good
-- Calculate total sales for each customer in 2024
select
    CUSTOMER_ID,
    sum(ORDER_TOTAL) as TOTAL_SALES
from
    ORDERS
where
    ORDER_DATE >= '2024-01-01'
group by
    CUSTOMER_ID;

-- Bad
-- Selecting customer ID and calculating the sum of order totals
select
    CUSTOMER_ID,
    sum(ORDER_TOTAL) as TOTAL_SALES
from
    ORDERS
where
    ORDER_DATE >= '2024-01-01'
group by
    CUSTOMER_ID;
```

### Guidelines for Effective Comments:
#### 1. Explain Methodology:
- Describe the reasoning behind complex calculations, transformations, or processes.
  
```sql
-- Allocating sales to regions based on customer ZIP code
select
    REGION,
    sum(SALES) as TOTAL_SALES
from
    SALES_DATA
group by
    REGION;
```
#### 2. Document Major Steps:
- Use comments to outline key steps in a multi-step process.
  
```sql
-- Step 1: Retrieve orders placed in 2024
with recent_orders as (
    select
        ORDER_ID,
        CUSTOMER_ID
    from
        ORDERS
    where
        ORDER_DATE >= '2024-01-01'
)
-- Step 2: Calculate total sales by customer
select
    CUSTOMER_ID,
    count(ORDER_ID) as TOTAL_ORDERS
from
    recent_orders
group by
    CUSTOMER_ID;
```

## Joins
- Use explicit join syntax (join + on) for clarity and to avoid Cartesian joins.
- Always use aliases to make your queries easier to read.
- Use the `as` keyword for explicit aliasing.
- Qualify column names with table aliases to avoid ambiguity in multi-table joins.
  
```sql
-- Good
select
    p.PRODUCT_ID,
    p.PRODUCT_NAME,
    s.STOCK_LEVEL
from
    PRODUCTS as p
join
    STOCK as s
on
  p.PRODUCT_ID = s.PRODUCT_ID
where
    s.STOCK_LEVEL < 10;

-- Bad
select
    PRODUCT_ID,
    PRODUCT_NAME,
    STOCK_LEVEL
from
    PRODUCTS, STOCK
where
    PRODUCTS.PRODUCT_ID = STOCK.PRODUCT_ID
    and STOCK_LEVEL < 10;
```

### Using Different Join Types
#### Inner Join (Most Common)
```sql
select
    o.ORDER_ID,
    c.CUSTOMER_NAME
from
    ORDERS as o
join
    CUSTOMERS as c
on
    o.CUSTOMER_ID = c.CUSTOMER_ID
where
    o.ORDER_DATE >= '2024-01-01';
```
  - Includes all customers, even if they have no orders.

#### Left Join (To Include Non-Matching Rows)

```sql
select
    c.CUSTOMER_NAME,
    o.ORDER_ID
from
    CUSTOMERS as c
left join
    ORDERS as o
on
    c.CUSTOMER_ID = o.CUSTOMER_ID
where
    c.REGION = 'Northwest';
```
  - Includes all customers, even if they have no orders.

#### Full Outer Join (Rarely Used)

```sql
select
    t1.ID,
    t1.VALUE as VALUE1,
    t2.VALUE as VALUE2
from
    TABLE1 as t1
full outer join
    TABLE2 as t2
on
    t1.ID = t2.ID;
```
  - Includes rows that match in either table, with `NULL` values for missing data.

### Best Practices for Multi-Table Joins
#### 1. Use Aliases Consistently:
- Always use short, meaningful aliases to distinguish tables.
#### 2. Use Qualified Column Names:
- In queries with multiple tables, qualify column names with table aliases to avoid ambiguity.
#### 3. Order Joins Logically:
- Place the primary or largest table (`from`) first, followed by smaller tables, to make the logic easier to follow.
  
```sql
select
    o.ORDER_ID,
    c.CUSTOMER_NAME,
    p.PRODUCT_NAME
from
    ORDERS as o
join
    CUSTOMERS as c
on
    o.CUSTOMER_ID = c.CUSTOMER_ID
join
    PRODUCTS as p
on
    o.PRODUCT_ID = p.PRODUCT_ID;
```
  
## Common Table Expressions (CTEs):  
- Use with Common Table Expressions (CTEs) for reusable logic and improved readability.
- Avoid overusing nested subqueries, as they can make the code harder to understand and maintain.
  
```sql
-- Good
with recent_orders as (
    select
        ORDER_ID,
        CUSTOMER_ID,
        ORDER_DATE
    from
        ORDERS
    where
        ORDER_DATE >= '2024-01-01'
),
customer_totals as (
    select
        CUSTOMER_ID,
        count(ORDER_ID) as TOTAL_ORDERS
    from
        recent_orders
    group by
        CUSTOMER_ID
)
select
    c.CUSTOMER_NAME,
    ct.TOTAL_ORDERS
from
    CUSTOMERS as c
join
    customer_totals as ct
on
  c.CUSTOMER_ID = ct.CUSTOMER_ID;

-- Bad
select
    c.CUSTOMER_NAME,
    (
        select
            count(o.ORDER_ID)
        from
            ORDERS as o
        where
            o.CUSTOMER_ID = c.CUSTOMER_ID
            and o.ORDER_DATE >= '2024-01-01'
    ) as TOTAL_ORDERS
from
    CUSTOMERS as c;
```
> [!TIP]
> #### Why is the "Bad" Example Problematic?
> - The nested subquery inside the `SELECT` clause makes the query harder to read and debug.
> - Repeating logic (e.g., filtering orders by date) increases the risk of errors if the logic needs to change.
> - Using a CTE instead separates the filtering logic, making the query easier to maintain and understand.

### When Not to Use CTEs:
- If the logic is simple and doesn't repeat, a straightforward query might suffice.
```sql
-- Unnecessary CTE
with simple_cte as (
    select
        ORDER_ID,
        CUSTOMER_ID
    from
        ORDERS
    where
        STATUS = 'Completed'
)
select
    *
from
    simple_cte;

-- Better
select
    ORDER_ID,
    CUSTOMER_ID
from
    ORDERS
where
    STATUS = 'Completed';
```

## Handling NULL Values
- Impute missing values with `COALESCE` rather than a `CASE` statement for better readability and performance.
  
```sql
-- Good: Using COALESCE
with customer_orders as (
    select
        CUSTOMER_ID,
        count(*) as TOTAL_ORDERS
    from
        ORDERS
    group by
        CUSTOMER_ID
)
select
    c.CUSTOMER_ID,
    c.CUSTOMER_NAME,
    coalesce(o.TOTAL_ORDERS, 0) as TOTAL_ORDERS
from
    CUSTOMERS as c
left join
    customer_orders as o
on
    c.CUSTOMER_ID = o.CUSTOMER_ID;

-- Bad: Using a CASE Statement
with customer_orders as (
    select
        CUSTOMER_ID,
        count(*) as TOTAL_ORDERS
    from
        ORDERS
    group by
        CUSTOMER_ID
)
select
    c.CUSTOMER_ID,
    c.CUSTOMER_NAME,
    case
        when o.TOTAL_ORDERS is null then 0
        else o.TOTAL_ORDERS
    end as TOTAL_ORDERS
from
    CUSTOMERS as c
left join
    customer_orders as o
on
    c.CUSTOMER_ID = o.CUSTOMER_ID;
```

## Advanced Query Techniques
### 1. Ranking with `RANK()` and `PARTITION BY`
#### Use Case:
- When you need to rank rows within groups (e.g., finding the top orders by customer).
#### Syntax
```sql
select
    column1,
    column2,
    rank() over (partition by column_to_partition order by column_to_rank desc) as rank_column
from
    table_name;
```
#### Example
- Find the top-ranked orders for each customer based on ORDER_TOTAL.
```sql
select
    CUSTOMER_ID,
    ORDER_ID,
    ORDER_TOTAL,
    rank() over (
        partition by CUSTOMER_ID
        order by ORDER_TOTAL desc
    ) as ORDER_RANK
from
    ORDERS;
```
  - The `PARTITION BY` clause groups rows by `CUSTOMER_ID`.
  - The `ORDER BY` clause ranks the rows within each group based on `ORDER_TOTAL` in descending order.

  
  
## Performance Best Practices
### 1. Avoid `select *`
- Always specify the columns you need for better performance and readability.
- Selecting all columns retrieves unnecessary data, increasing query time and resource usage.
  
```sql
-- Good
select
    CUSTOMER_ID,
    CUSTOMER_NAME
from
    CUSTOMERS;

-- Bad
select * from CUSTOMERS;
```

### 2. Use Appropriate Data Types:
- Ensure columns use the most efficient data type to save storage and improve query performance. For example:
  - Use INT for numeric IDs (instead of `VARCHAR` or `TEXT`).
  - Use `DECIMAL` for precise financial data.
  - For large datatypes, use `VARCHAR` or `NVARCHAR` with a defined length whenever possible for better performance.

```sql
-- Example of Efficient Data Types
create table ORDERS (
    ORDER_ID INT,               -- Efficient for IDs
    CUSTOMER_ID INT,            -- Matches the referenced table
    ORDER_TOTAL DECIMAL(10, 2), -- Precise for financial data
    ORDER_DATE DATE             -- Specific for date storage
);
```
    
### 3. Indexing:
- Index columns frequently used in `where`, `join`, or `group` by clauses to improve query performance.
- **Note**: Over-indexing can slow down insert/update operations, so use indexes judiciously.
  
```sql
-- Create an index on CUSTOMER_ID for faster lookups
create index idx_customer_id on ORDERS (CUSTOMER_ID);

-- Query leveraging the index
select
    CUSTOMER_ID,
    sum(ORDER_TOTAL) as TOTAL_ORDERS
from
    ORDERS
where
    CUSTOMER_ID = 123
group by
    CUSTOMER_ID;
```
  
### 4. Filter Early:
- Use the where clause to filter rows as early as possible to reduce the amount of data processed in subsequent steps like `group by` or `having`.

```sql
-- Good
select
    CUSTOMER_ID,
    sum(ORDER_TOTAL) as TOTAL_ORDERS
from
    ORDERS
where
    STATUS = 'Shipped'          -- Filter early
group by
    CUSTOMER_ID;

-- Bad
select
    CUSTOMER_ID,
    sum(ORDER_TOTAL) as TOTAL_ORDERS
from
    ORDERS
group by
    CUSTOMER_ID
having
    STATUS = 'Shipped';         -- Late filtering
```

### 5.
- Avoid overly complex queries; break complex queries into smaller, modular parts using Common Table Expressions (CTEs) or temporary tables.

```sql
-- Good: Break into smaller parts using CTEs
with recent_orders as (
    select
        ORDER_ID,
        CUSTOMER_ID,
        ORDER_TOTAL
    from
        ORDERS
    where
        ORDER_DATE >= '2024-01-01'
),
customer_totals as (
    select
        CUSTOMER_ID,
        sum(ORDER_TOTAL) as TOTAL_ORDERS
    from
        recent_orders
    group by
        CUSTOMER_ID
)
select
    CUSTOMER_ID,
    TOTAL_ORDERS
from
    customer_totals
where
    TOTAL_ORDERS > 1000;

-- Bad: Overly complex query
select
    CUSTOMER_ID,
    sum(ORDER_TOTAL) as TOTAL_ORDERS
from
    (
        select
            ORDER_ID,
            CUSTOMER_ID,
            ORDER_TOTAL
        from
            ORDERS
        where
            ORDER_DATE >= '2024-01-01'
    ) as subquery
group by
    CUSTOMER_ID
having
    sum(ORDER_TOTAL) > 1000;
```
  
### 6. Group Aggregates Clearly:
- Use descriptive column names for aggregates.
  
```sql
select
    DEPARTMENT,
    count(EMPLOYEE_ID) as EMPLOYEE_COUNT
from
    EMPLOYEES
group by
    DEPARTMENT
having
    EMPLOYEE_COUNT > 5;
```

## Naming Conventions
### 1. Tables and Columns:
- Use **UPPERCASE** for table and column names.
- Use **snake_case** for naming: `CUSTOMER_ID`, `ORDER_DATE`.
### 2. Primary Keys:
- Name primary keys as `id` or `TABLE_NAME_ID`.
### 3. Foreign Keys:
- Use `REFERENCED_TABLE_NAME_ID` for foreign keys.
