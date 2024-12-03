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
- Align and/or in conditions under the where clause.
### White Space Around Symbols
- Always include white space around operators like =, *, -, and + to improve readability.
  
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

## Joins
- Use explicit join syntax (join + on) for clarity.
- Always use aliases to make your queries easier to read.
- Use the `as` keyword for explicit aliasing.

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

## Use CTEs for Clarity:  
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

## Performance Best Practices
### 1. Avoid `select *`
- Always specify the columns you need for better performance and readability.
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
- Ensure columns use the most efficient data type (e.g., `INT` for numeric IDs, `VARCHAR` for text).
### 3. Indexing:
- Index columns frequently used in `where`, `join`, or `group` by clauses.
### 4. Filter Early:
- Use the where clause to filter rows before applying group by or having.
### 5.
- Avoid overly complex queries; break them into smaller, modular parts.
### 5. Group Aggregates Clearly:
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
