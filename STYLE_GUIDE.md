# SQL Style Guide
This guide outlines best practices and conventions for writing clean, maintainable, and efficient SQL code. By adhering to these guidelines, teams can improve code readability, simplify collaboration, and ensure consistency across projects.

## 1. General Formatting
### 1.1. Line Breaks
- Place each clause (`select`, `from`, `where`, etc.) on a new line.
```sql
-- Good
select
    ORDER_ID,
    CUSTOMER_ID,
    ORDER_DATE
from
    ORDERS
where
    STATUS = 'Completed';

-- Bad
SELECT ORDER_ID, CUSTOMER_ID, ORDER_DATE FROM ORDERS WHERE STATUS='Completed';
```

### 1.2. Indentation
- Place each variable on its own line, indented under the keyword it belongs to.
- Keep SQL keywords (`select`, `from`, `where`) aligned to the left.
- Add a new line for each `AND` or `OR` in `WHERE` conditions, aligning them under the `WHERE` keyword.
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
    and STATE = 'IL'
    and ZIP_CODE = '60601';

-- Bad
SELECT CUSTOMER_ID, CUSTOMER_NAME, CUSTOMER_EMAIL
FROM CUSTOMERS
WHERE CITY='Chicago' AND STATE='IL' AND ZIP_CODE='60601';
```

### 1.3. Capitalization
- Use **lowercase** for SQL keywords (e.g., `select`, `from`, `where`).
- Use **UPPERCASE** for table and column names to make them stand out.
```sql
-- Good
select
    CUSTOMER_ID,
    CUSTOMER_NAME
from
    CUSTOMERS
where
    CUSTOMER_NAME like 'A%';

-- Bad
SELECT
    CUSTOMER_ID,
    CUSTOMER_NAME
FROM
    CUSTOMERS
WHERE
    CUSTOMER_NAME LIKE 'A%';
 ``` 

### 1.4. White Space Around Symbols
- Always include white space around operators like `=`, `*`, `-`, and `+` to improve readability.
```sql
-- Good
select
    CUSTOMER_ID,
    CUSTOMER_NAME,
    (ORDER_TOTAL - DISCOUNT) * (1 + TAX_RATE / 100) as FINAL_TOTAL,
    (YEAR(GETDATE()) - YEAR(REGISTRATION_DATE)) as YEARS_AS_CUSTOMER
from
    ORDERS
where
    (ORDER_TOTAL - DISCOUNT) * (1 + TAX_RATE / 100) > 500
    and DISCOUNT > 0;

-- Bad
select
    CUSTOMER_ID,
    CUSTOMER_NAME,
    (ORDER_TOTAL-DISCOUNT)*(1+TAX_RATE/100) as FINAL_TOTAL,
    (YEAR(GETDATE())-YEAR(REGISTRATION_DATE)) as YEARS_AS_CUSTOMER
from
    ORDERS
where
    (ORDER_TOTAL-DISCOUNT)*(1+TAX_RATE/100)>500
    and DISCOUNT>0;
```

## 1.5. Comments 
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

> [!TIP]
> ## Guidelines for Effective Comments:
> #### 1. Focus on the "why," not the "how":
> - Use comments to explain the reasoning behind your approach, such as the logic or business rules being applied, rather than describing the code's functionality line by line.
> - Avoid redundant comments; if the code is self-explanatory, additional comments about what the code does are unnecessary.
> ```sql
> -- Allocating sales to regions based on customer ZIP code
> select
>     REGION,
>     sum(SALES) as TOTAL_SALES
> from
>     SALES_DATA
> group by
>     REGION;
> ```
> #### 2. Document Major Steps:
> - Use comments to outline key steps in a multi-step process.
> ```sql
> -- Step 1: Retrieve orders placed in 2024
> with recent_orders as (
>     select
>         ORDER_ID,
>         CUSTOMER_ID
>     from
>         ORDERS
>     where
>         ORDER_DATE >= '2024-01-01'
> )
> -- Step 2: Calculate total sales by customer
> select
>     CUSTOMER_ID,
>     count(ORDER_ID) as TOTAL_ORDERS
> from
>     recent_orders
> group by
>     CUSTOMER_ID;
> ```

## 2. Joins
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
    on p.PRODUCT_ID = s.PRODUCT_ID
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

### 2.1. Using Different Join Types
#### 2.1.1. Inner Join (Most Common)
- **Defintion**: Returns only the rows where there is a match in both tables.
- **Use Case**: Used to retrieve data that exists in both tables.
- **Example**: Only includes customers that are present in both tables.
```sql
select
    o.ORDER_ID,
    c.CUSTOMER_NAME
from
    ORDERS as o
join
    CUSTOMERS as c
    on o.CUSTOMER_ID = c.CUSTOMER_ID
where
    o.ORDER_DATE >= '2024-01-01';
```


#### 2.1.2. Left Join (To Include Non-Matching Rows)
- **Defintion**: Returns all rows from the left table, and the matching rows from the right table. If there’s no match, `NULL` values are returned for the right table’s columns.
- **Use Case**: Use when you want to keep all rows from the left table, even if there are no matches in the right table.
- **Example**: Includes all customers, even if they have no orders.
```sql
select
    c.CUSTOMER_NAME,
    o.ORDER_ID
from
    CUSTOMERS as c
left join
    ORDERS as o
    on c.CUSTOMER_ID = o.CUSTOMER_ID
where
    c.REGION = 'Northwest';
```

#### 2.1.3. Full Outer Join (Rarely Used)
- **Defintion**: Returns all rows from both tables. For rows without a match in the other table, `NULL` values are returned for the unmatched columns.
- **Use Case**: Use when you need all rows from both tables, regardless of whether they match.
- **Example**: Includes rows that match in either table, with `NULL` values for missing data.
```sql
select
    t1.ID,
    t1.VALUE as VALUE1,
    t2.VALUE as VALUE2
from
    TABLE1 as t1
full outer join
    TABLE2 as t2
    on t1.ID = t2.ID;
```

### 2.2. Best Practices for Multi-Table Joins
 - **Use Aliases Consistently**: Always use short, meaningful aliases to distinguish tables.
 - **Use Qualified Column Names**: In queries with multiple tables, qualify column names with table aliases to avoid ambiguity.
 - **Order Joins Logically**: Place the primary or largest table (`from`) first, followed by smaller tables, to make the logic easier to follow.
 ```sql
 select
     o.ORDER_ID,
     c.CUSTOMER_NAME,
     p.PRODUCT_NAME
 from
     ORDERS as o
 join
     CUSTOMERS as c
     on o.CUSTOMER_ID = c.CUSTOMER_ID
 join
     PRODUCTS as p
     on o.PRODUCT_ID = p.PRODUCT_ID;
 ```
  
## 3. Common Table Expressions (CTEs):  
- **Defintion**:
    -  A **Common Table Expression (CTE)** is a temporary result set that you can reference within a `SELECT`, `INSERT`, `UPDATE`, or `DELETE` statement.
    -  It is defined at the start of a query using the `WITH` keyword and exists only for the duration of the query in which it is defined.
-  **How CTEs Work**:
    -  CTEs improve **query readability** by breaking down complex logic into manageable parts.
    -  They allow you to **reuse intermediate results** within the same query.
    -  Unlike temporary tables, CTEs are **memory-resident** and are not materialized on disk, making them lightweight and easy to use.
-  **Use Case**:
    - To **simplify complex queries** by modularizing the logic.
    - To **reuse intermediate results** within a single query.
    - To replace **nested subqueries**, making the query easier to read and debug.
- **Syntax**:
 ```sql
with CTE_Name as (
    select
        COLUMN1,
        COLUMN2
    from
        TABLE_NAME
    where
        CONDITION
)
select
    *
from
    CTE_Name;
```
- **Example**:
```sql
-- Good
with recent_order_totals as (
    select
        CUSTOMER_ID,
        count(ORDER_ID) as TOTAL_ORDERS
    from
        ORDERS
    where
        ORDER_DATE >= '2024-01-01'
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
    on c.CUSTOMER_ID = ct.CUSTOMER_ID;

-- Bad
select
    c.CUSTOMER_NAME,
    ct.TOTAL_ORDERS
from
    CUSTOMERS as c
join (
    select
        o.CUSTOMER_ID,
        count(o.ORDER_ID) as TOTAL_ORDERS
    from
        ORDERS as o
    where
        o.ORDER_DATE >= '2024-01-01'
    group by
        o.CUSTOMER_ID
    ) as ct
    on c.CUSTOMER_ID = ct.CUSTOMER_ID;
```
### 3.1. Tips for Writing Effective CTEs
Here are some best practices and tips to write clean, maintainable, and efficient CTEs:
#### 3.1.1. Keep CTEs Modular and Reusable
- Each CTE should have a clear, singular purpose.
- Limit CTEs to filtering, formatting, or basic calculations.
```sql
-- Good: CTE handles filtering
with filtered_orders as (
    select
        ORDER_ID,
        CUSTOMER_ID,
        ORDER_TOTAL
    from
        ORDERS
    where
        ORDER_DATE >= '2024-01-01'
)
select
    CUSTOMER_ID,
    sum(ORDER_TOTAL) as TOTAL_ORDER_VALUE
from
    filtered_orders
group by
    CUSTOMER_ID;
```

#### 3.1.2. Avoid Embedding Joins in CTEs
- Keep joins in the main query for better readability and flexibility.
- Joins are typically major processing steps and should not be buried within CTEs.
```sql
-- Good: Join in the main query
with filtered_orders as (
    select
        ORDER_ID,
        CUSTOMER_ID,
        ORDER_TOTAL
    from
        ORDERS
    where
        ORDER_DATE >= '2024-01-01'
),
filtered_customers as (
    select
        CUSTOMER_ID,
        CUSTOMER_NAME
    from
        CUSTOMERS
)
select
    fc.CUSTOMER_NAME,
    sum(fo.ORDER_TOTAL) as TOTAL_ORDER_VALUE
from
    filtered_customers as fc
join
    filtered_orders as fo
    on fc.CUSTOMER_ID = fo.CUSTOMER_ID
group by
    fc.CUSTOMER_NAME;

-- Bad: Embedding the join in the CTE
with customer_orders as (
    select
        c.CUSTOMER_ID,
        c.CUSTOMER_NAME,
        o.ORDER_TOTAL
    from
        CUSTOMERS as c
    join
        ORDERS as o
        on c.CUSTOMER_ID = o.CUSTOMER_ID
)
select
    CUSTOMER_NAME,
    sum(ORDER_TOTAL) as TOTAL_ORDER_VALUE
from
    customer_orders
group by
    CUSTOMER_NAME;
```

#### 3.1.3. Ensure CTEs Are Independently Executable
- Each CTE should be able to run on its own for easier debugging and validation.
- Avoid interdependent CTEs where one CTE cannot be tested or debugged in isolation.
```sql
-- Bad
with filtered_orders as (
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
        sum(ORDER_TOTAL) as TOTAL_ORDER_VALUE
    from
        filtered_orders
    group by
        CUSTOMER_ID
)
select
    ct.CUSTOMER_ID,
    ct.TOTAL_ORDER_VALUE
from
    customer_totals as ct
where
    ct.TOTAL_ORDER_VALUE > 1000;
```
> **Why This is Problematic:**
> - **Dependency**: `customer_totals` depends on `filtered_orders`, so you cannot test it independently without executing `filtered_orders` first.
> - **Debugging**: If something goes wrong, it’s harder to isolate issues in 1customer_totals1.

#### 3.1.4. Name CTEs Meaningfully
- Use descriptive, concise names to indicate the purpose of the CTE.
- Avoid generic names like `cte1` or `temp`.
```sql
-- Good: Meaningful names
with recent_orders as (
    ...
),
customer_aggregates as (
    ...
)

-- Bad
with cte1 as (
    ...
),
cte2 as (
    ...
)
```

#### 3.1.5. Avoid Using CTEs for Simple Queries
- If a single query can achieve the same result without compromising readability, skip the CTE.
```sql
-- Unnecessary CTE
with simple_query as (
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
    simple_query;

-- Better
select
    ORDER_ID,
    CUSTOMER_ID
from
    ORDERS
where
    STATUS = 'Completed';
```

#### 3.1.6. Limit the number of CTEs in a single query
- Use temporary tables for complex multi-step transformations instead of chaining too many dependent CTEs.
- Often combinations of temporary tables and CTE's are needed to help to break down scripts into manageable, testable parts.

> [!WARNING] 
> **Use CTEs for Readability and Test Performance**
> - CTEs can occasionally improve performance by helping the query optimizer break down complex logic, but **are primarily designed for improving query readability and modularity**
>     - Whether they help or hinder performance depends on:
>         - The size and complexity of the data.
>         - How the SQL engine optimizes the query.
>         - Whether the CTE is materialized (calculated once) or re-evaluated each time it's referenced.
> - It is essential to test the query execution plan. Compare the performance of using CTEs with alternatives like:
>     - **Temporary Tables**: Useful for large datasets or repeated use across multiple queries.
>     - **Indexed Views**: Helpful for frequently accessed pre-aggregated or joined data.

### 4. Dynamic Query Design
- Avoid hardcoding variables like dates, table names, or filter conditions can reduce maintainability and flexibility.
- Instead, use dynamic approaches that adapt to changing data or requirements. Here’s how you can avoid hardcoding and improve your query design:
#### 4.1. Use Functions to Dynamically Generate Dates
- Instead of hardcoding specific dates, use SQL functions like `GETDATE()`, `DATEADD()`, or `YEAR()` to dynamically calculate date ranges.
```sql
-- Good: Dynamically calculate the current year
select
    CUSTOMER_ID,
    ORDER_DATE,
    ORDER_TOTAL
from
    ORDERS
where
    year(ORDER_DATE) = year(getdate());

-- Bad: Hardcoded year
select
    CUSTOMER_ID,
    ORDER_DATE,
    ORDER_TOTAL
from
    ORDERS
where
    year(ORDER_DATE) = 2024;
```

#### 4.2. Use Functions to Dynamically Generate Dates
- Instead of hardcoding conditions, query the table dynamically to get the required value.
```sql
-- Good: Dynamically fetch the latest year from the table
select
    CUSTOMER_ID,
    ORDER_DATE,
    ORDER_TOTAL
from
    ORDERS
where
    year(ORDER_DATE) = (select max(year(ORDER_DATE)) from ORDERS);

-- Bad: Hardcoded year
select
    CUSTOMER_ID,
    ORDER_DATE,
    ORDER_TOTAL
from
    ORDERS
where
    year(ORDER_DATE) = 2023;
```

#### 4.3. Use Variables for Flexibility
- Declare and use variables for frequently changing values like dates or thresholds.

```sql
-- Good: Use a variable for the year
declare @currentYear int = year(getdate());

select
    CUSTOMER_ID,
    ORDER_DATE,
    ORDER_TOTAL
from
    ORDERS
where
    year(ORDER_DATE) = @currentYear;
```

#### 4.4. Avoid Hardcoding Table Names: Use Dynamic SQL
- When working with partitioned tables or tables with dynamic names (e.g., based on years), construct queries dynamically.
```sql
-- Good: Construct the table name dynamically
declare @currentYear nvarchar(4) = cast(year(getdate()) as nvarchar(4));
declare @sql nvarchar(max);

set @sql = '
select
    CUSTOMER_ID,
    ORDER_TOTAL
from
    ORDERS_' + @currentYear + '
where
    ORDER_TOTAL > 1000';

exec sp_executesql @sql;

-- Bad: Hardcoded table name
select
    CUSTOMER_ID,
    ORDER_TOTAL
from
    ORDERS_2024
where
    ORDER_TOTAL > 1000;
```

#### 4.5. Leverage Parameterized Queries
- Use parameters for dynamic filtering conditions to make your queries reusable and adaptable.
```sql
-- Good: Parameterized query
declare @state nvarchar(2) = 'IL';

select
    CUSTOMER_ID,
    CUSTOMER_NAME
from
    CUSTOMERS
where
    STATE = @state;

-- Bad: Hardcoded state
select
    CUSTOMER_ID,
    CUSTOMER_NAME
from
    CUSTOMERS
where
    STATE = 'IL';
```

## 6. Keys and Indexing
### 6.1. Primary Keys
- **Definition**:
  - A **primary key** is a column (or combination of columns) that uniquely identifies each row in a table. It:
    - Ensures uniqueness: No two rows can have the same value for the primary key column.
    - Prevents null values: The column(s) in a primary key must always have a value.
    - Automatically creates an index: A primary key typically creates a **clustered index** on the specified column(s).
- **Use Case**:
  - Use primary keys to uniquely identify each row in a table, such as `ORDER_ID` or `CUSTOMER_ID`.
- **Example**:  
```sql
create table ORDERS (
    ORDER_ID int not null primary key, -- Unique identifier for each order
    CUSTOMER_ID int not null,
    ORDER_DATE date not null,
    ORDER_TOTAL decimal(10, 2)
);
```
> [!TIP] 
> - **Best Practices**:
>   - Choose a column with immutable and unique values (e.g., an auto-incrementing integer or a UUID).
>   - Avoid using large columns (e.g., `VARCHAR(MAX)`) as primary keys, as they increase storage and query costs.
>   - Use surrogate keys (e.g., generated IDs) if no natural unique identifier exists.

### 6.2. Foreign Keys
- **Definition**:
  - A **foreign key** creates a relationship between two tables by linking a column in one table (child table) to the primary key in another table (parent table).
  - It enforces **referential integrity**, ensuring that data in the child table corresponds to data in the parent table.
- **Use Case**:
  - Use foreign keys to maintain relationships between tables, such as linking `ORDERS` to `CUSTOMERS`.
- **Example**:
```sql
create table CUSTOMERS (
    CUSTOMER_ID int primary key,
    CUSTOMER_NAME varchar(100) not null
);

create table ORDERS (
    ORDER_ID int primary key,
    CUSTOMER_ID int not null,
    ORDER_TOTAL decimal(10, 2),
    foreign key (CUSTOMER_ID) references CUSTOMERS(CUSTOMER_ID)
);
```
> [!TIP] 
> **Best Practices**:
>  -  Always index foreign key columns for better performance.
>  -  Use cascading actions (`ON DELETE CASCADE`, `ON UPDATE CASCADE`) judiciously.

### 6.3. Clustered and Non-clustered Indexes
- **Definition**:
  - Indexes improve the speed of data retrieval operations.
  - Proper use of indexes balances performance and storage requirements.
- **Clustered Index:**:
  - Determines the physical order of rows in a table.
  - A table can have only one **clustered index**, usually created on the primary key.
- **Non-Clustered Index:**
  - A separate structure from the table that points to the data rows.
  - Useful for columns frequently used in `WHERE`, `JOIN`, or `GROUP BY` clauses.
- **Example**:
```sql
-- Clustered index on primary key (default behavior)
create table ORDERS (
    ORDER_ID int not null primary key clustered,
    ORDER_DATE date not null
);

-- Non-clustered index on ORDER_DATE
create nonclustered index idx_order_date on ORDERS (ORDER_DATE);
```

### 6.4. Covering Indexes
- **Definition**:
  - A **covering index** includes all columns referenced in a query, allowing the database to retrieve results directly from the index without accessing the base table.
- **Use Case:**:
  - Covering indexes are ideal for queries that repeatedly access specific column subsets.
- **Example**:
```sql
-- Create a covering index for a query that retrieves ORDER_TOTAL and ORDER_DATE
create index idx_covering on ORDERS (CUSTOMER_ID, ORDER_TOTAL, ORDER_DATE);

-- Query optimized by the covering index
select
    CUSTOMER_ID,
    ORDER_TOTAL,
    ORDER_DATE
from
    ORDERS
where
    CUSTOMER_ID = 123;
```
### 6.5. Indexing Best Practices:
- Index columns frequently used in `where`, `join`, or `group` by clauses to improve query performance.
> [!WARNING]
> - Over-indexing can slow down insert/update operations, so use indexes judiciously.
> - Regularly analyze query performance and remove unused indexes.
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

> [!TIP]
> #### Naming Conventions
> - Follow consistent and descriptive naming conventions for keys and indexes:
> - **Primary Keys**: `pk_<table_name>`
> - **Foreign Keys**: `fk_<child_table>_<parent_table>`
> - **Indexes**: `idx_<table_name>_<column_name>`
>#### **Example**:
>```sql
>create table ORDERS (
>    ORDER_ID int not null primary key constraint pk_orders_order_id,
>    CUSTOMER_ID int not null,
>    constraint fk_orders_customers foreign key (CUSTOMER_ID) references CUSTOMERS(CUSTOMER_ID)
>);
>```
    
## 7. Performance Best Practices
### 7.1. Avoid `select *`
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

### 7.2. Use Appropriate Data Types:
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
    
### 7.3. Creating tables vs. `SELECT INTO`:
- Manually defining a table’s schema before inserting data gives you full control over its structure, indexes, and constraints.
  -   Allows for schema validation, ensuring column types and constraints match your requirements.
  -   Enables adding indexes or constraints before inserting data for better query performance.
  -   Easier to maintain and extend in the future.
- `SELECT INTO` should be reserved for temporary operations of ad-hoc queries
```sql
-- Good: Manually defining the table schema with a primary key
create table TEMP_ORDERS (
    ORDER_ID int not null primary key, -- Adding a primary key to ensure uniqueness
    CUSTOMER_ID int not null,
    ORDER_DATE date not null,
    ORDER_TOTAL decimal(10, 2)
);

insert into TEMP_ORDERS
select
    ORDER_ID,
    CUSTOMER_ID,
    ORDER_DATE,
    ORDER_TOTAL
from
    ORDERS
where
    ORDER_DATE >= '2024-01-01';

-- Bad: Using SELECT INTO without pre-defining the schema
select
    ORDER_ID,
    CUSTOMER_ID,
    ORDER_DATE,
    ORDER_TOTAL
into
    TEMP_ORDERS
from
    ORDERS
where
    ORDER_DATE >= '2024-01-01';

-- Adding a primary key constraint after the table is created
alter table TEMP_ORDERS
add constraint pk_temp_orders primary key (ORDER_ID);
```

### 7.4. Filter Early:
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

### 7.5. Modular Query Design
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
  
### 7.6. Group Aggregates Clearly:
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

### 7.7. Use `COALESCE` for Handling NULL Values
- Impute missing values with `COALESCE` rather than a `CASE` statement for better readability and performance.
```sql
-- Good: Using COALESCE
select
    CUSTOMER_ID,
    coalesce(TOTAL_ORDERS, 0) as TOTAL_ORDERS
from
    CUSTOMER_ORDERS;

-- Bad: Using a CASE Statement
select
    CUSTOMER_ID,
    case
        when TOTAL_ORDERS is null then 0
        else TOTAL_ORDERS
    end as TOTAL_ORDERS
from
    CUSTOMER_ORDERS;
```

  
## 8. Advanced Query Techniques
### 8.1. Pivoting Data
#### Use Case:
- Convert rows into columns for easier reporting or analysis.
#### Syntax  
```sql
select
    pivoted_columns
from
    (select ...) as source_table
pivot (
    aggregate_function(column_to_aggregate)
    for column_to_pivot in (value1, value2, ...)
) as pivoted_table;
```
#### Example
- Convert sales data into a pivoted format, showing total sales for each region by year.
``` sql
select
    REGION,
    [2022] as SALES_2022,
    [2023] as SALES_2023
from
    (select
         REGION,
         YEAR(SALE_DATE) as SALE_YEAR,
         SALE_AMOUNT
     from
         SALES) as source_data
pivot (
    sum(SALE_AMOUNT)
    for SALE_YEAR in ([2022], [2023])
) as pivoted_data;
```

### 8.2. Unpivoting Data
#### Use Case:
- Convert columns into rows, often to normalize a dataset.
#### Syntax  
```sql
select
    column1,
    column2,
    unpivoted_column,
    value_column
from
    table_name
unpivot (
    value_column for unpivoted_column in (column_to_unpivot1, column_to_unpivot2, ...)
) as unpivoted_table;
```
#### Example
- Unpivot sales data to analyze it row-wise by year.
```sql
select
    REGION,
    SALE_YEAR,
    SALE_AMOUNT
from
    (select
         REGION,
         SALES_2022,
         SALES_2023
     from
         SALES_BY_YEAR) as source_data
unpivot (
    SALE_AMOUNT for SALE_YEAR in ([SALES_2022], [SALES_2023])
) as unpivoted_data;
```
> [!TIP]
> #### Use Dynamic SQL for Unpivoting when Many Columns are Present
> - When you have many columns to unpivot, manually listing them can be tedious and error-prone. Dynamic SQL can automate this process by dynamically identifying all the columns in a table.
> ```sql
>-- Step 1: Dynamically generate the column list for UNPIVOT
>declare @columns nvarchar(max);
>declare @sql nvarchar(max);
>
>select
>    @columns = string_agg(quotename(COLUMN_NAME), ',')
>from
>    INFORMATION_SCHEMA.COLUMNS
>where
>    TABLE_NAME = 'SALES_BY_YEAR'
>    and COLUMN_NAME like 'SALES_%';
>
>-- Step 2: Use the dynamically generated column list to construct the UNPIVOT query.
>set @sql = '
>select
>    REGION,
>    SALE_YEAR,
>    SALE_AMOUNT
>from
>    (select REGION, ' + @columns + ' from SALES_BY_YEAR) as source_data
>unpivot (
>    SALE_AMOUNT for SALE_YEAR in (' + @columns + ')
>) as unpivoted_data;
>';
>
>-- Step 3: Execute the dynamic query
>exec sp_executesql @sql;
>```

### 8.3. Ranking with `RANK()` and `PARTITION BY`
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

    
### 8.4. Imputing Missing Values with an Average (Using `COALESCE` and `PARTITION BY`)
#### Use Case:
- Fill `NULL` values in a column with the average value of that column, calculated within groups (e.g., by `REGION`).
#### Syntax  
```sql
select
    COLUMN_1,
    COLUMN_2,
    coalesce(COLUMN_WITH_NULL, avg(COLUMN_WITH_NULL) over (partition by GROUPING_COLUMN)) as IMPUTED_VALUE
from
    TABLE_NAME;
```
#### Example
- Suppose you have a sales table with `NULL` values in the `SALE_AMOUNT` column. You want to replace these `NULL` values with the average sales for the same `REGION`.
```sql
select
    REGION,
    SALE_ID,
    coalesce(SALE_AMOUNT, avg(SALE_AMOUNT) over (partition by REGION)) as IMPUTED_SALE_AMOUNT
from
    SALES;
```
- `avg(SALE_AMOUNT) over (partition by REGION)` calculates the average sales amount for each region.
- `coalesce` replaces `NULL` values in `SALE_AMOUNT` with the computed average for that region.

### 8.5. Rolling Averages with `PARTITION BY`
#### Use Case:
- Calculate a rolling average of sales within groups (e.g., by `REGION`).
#### Syntax  
```sql
select
    COLUMN_1,
    COLUMN_2,
    COLUMN_WITH_VALUES,
    avg(COLUMN_WITH_VALUES) over (
        partition by GROUPING_COLUMN
        order by SORTING_COLUMN
        rows between N preceding and current row
    ) as ROLLING_AVERAGE
from
    TABLE_NAME;
```
#### Example
```sql
select
    REGION,
    SALE_DATE,
    SALE_AMOUNT,
    avg(SALE_AMOUNT) over (
        partition by REGION
        order by SALE_DATE
        rows between 2 preceding and current row
    ) as ROLLING_AVG
from
    SALES;
```
- The `rows between 2 preceding and current row` clause specifies a 3-row rolling window (current row + 2 previous rows).
- `partition by REGION` ensures rolling averages are calculated separately for each region.

### 8.6. Finding Percentiles with `NTILE`
#### Use Case:
- Distribute rows into quartiles or other equal-sized groups.
#### Syntax  
```sql
select
    COLUMN_1,
    COLUMN_2,
    ntile(NUMBER_OF_BUCKETS) over (
        order by SORTING_COLUMN
    ) as PERCENTILE_BUCKET
from
    TABLE_NAME;
```
#### Example
```sql
select
    SALE_ID,
    SALE_AMOUNT,
    ntile(4) over (
        order by SALE_AMOUNT
    ) as QUARTILE
from
    SALES;
```
- `ntile(4)` divides the dataset into four equal parts (quartiles) based on `SALE_AMOUNT`.

### 8.7. Case-Specific Aggregations 
#### Use Case:
- Calculate different metrics for different conditions within the same query.
#### Syntax  
```sql
select
    GROUPING_COLUMN,
    sum(case when CONDITION then COLUMN else 0 end) as METRIC_1,
    sum(case when OTHER_CONDITION then COLUMN else 0 end) as METRIC_2
from
    TABLE_NAME
group by
    GROUPING_COLUMN;
```
#### Example
```sql
select
    REGION,
    sum(case when SALE_AMOUNT > 100 then SALE_AMOUNT else 0 end) as HIGH_VALUE_SALES,
    sum(case when SALE_AMOUNT <= 100 then SALE_AMOUNT else 0 end) as LOW_VALUE_SALES
from
    SALES
group by
    REGION;
```
- Conditional aggregation with `CASE` allows you to compute metrics for specific conditions.

### 8.8. Case-Specific Aggregations 
#### Use Case:
- Compare a row’s value with the previous or next row within a group.
- Helpful in calculating YoY percent change in long format.
#### Syntax  
```sql
select
    GROUPING_COLUMN,
    SORTING_COLUMN,
    COLUMN_WITH_VALUES,
    lag(COLUMN_WITH_VALUES) over (
        partition by GROUPING_COLUMN
        order by SORTING_COLUMN
    ) as PREVIOUS_VALUE,
    lead(COLUMN_WITH_VALUES) over (
        partition by GROUPING_COLUMN
        order by SORTING_COLUMN
    ) as NEXT_VALUE
from
    TABLE_NAME;
```
#### Example
```sql
select
    REGION,
    SALE_DATE,
    SALE_AMOUNT,
    lag(SALE_AMOUNT) over (
        partition by REGION
        order by SALE_DATE
    ) as PREVIOUS_SALE,
    lead(SALE_AMOUNT) over (
        partition by REGION
        order by SALE_DATE
    ) as NEXT_SALE
from
    SALES;
```
- `lag` retrieves the value from the previous row.
- `lead` retrieves the value from the next row.


## 9. Error Handling
### 9.1. Debugging and Testing Queries
#### 9.1.1. Test Incrementally:
- Build queries step by step, testing each component individually before combining them.
- Validate intermediate results using `SELECT` statements or temporary tables.
#### 9.1.2. Use `TOP` or `LIMIT`:
- For large datasets, test with a small subset to quickly validate the logic.
```sql
select
    top 10 *
from
    ORDERS;
```
#### 9.1.3. Cross-Check Results:
- Validate query results against expected outputs or smaller subsets of data.
#### 9.1.4. Use Aggregates for Sanity Checks:
- Verify totals, counts, or averages to ensure the query is processing data as expected.
```sql
select
    count(*) as TOTAL_ORDERS,
    sum(ORDER_TOTAL) as TOTAL_SALES
from
    ORDERS;
```

### 9.2. Prevent Common Errors
#### 9.2.1. Handle NULL Values:
- Use `COALESCE` or `ISNULL` to manage `NULL` values and avoid unexpected results in calculations.
```sql 
select
    CUSTOMER_ID,
    isnull(ORDER_TOTAL, 0) as ORDER_TOTAL
from
    ORDERS;
```
#### 9.2.2. Check for Duplicate Data:
- Use `DISTINCT` or `GROUP BY` to ensure unique results when required.
```sql
select distinct
    CUSTOMER_ID
from
    ORDERS;
```
#### 9.2.3. Validate Filters:
- Double-check `WHERE` clauses for logical errors, such as unintentional exclusions or inclusions.

### 9.3. Query Optimization for Error Handling
#### 9.3.1. Use EXPLAIN or EXPLAIN ANALYZE:
-Analyze the query execution plan to identify bottlenecks or inefficient operations.
```sql
explain
select
    *
from
    ORDERS
where
    ORDER_DATE >= '2024-01-01';
```
#### 9.4.1. Log Errors:
- Use database error-handling mechanisms (like `TRY`...`CATCH` in SQL Server) to log and capture runtime errors.
```sql
begin try
    with valid_orders as (
        select
            ORDER_ID,
            CUSTOMER_ID,
            isnull(ORDER_TOTAL, 0) as ORDER_TOTAL
        from
            ORDERS
        where
            ORDER_DATE >= '2024-01-01'
    )
    select
        CUSTOMER_ID,
        sum(ORDER_TOTAL) as TOTAL_SALES
    from
        valid_orders
    group by
        CUSTOMER_ID;
end try
begin catch
    select
        ERROR_MESSAGE() as ERROR_LOG,
        ERROR_NUMBER() as ERROR_CODE,
        ERROR_LINE() as LINE_NUMBER;
end catch;
```
