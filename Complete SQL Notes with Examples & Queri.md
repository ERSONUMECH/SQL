# Complete SQL Notes with Examples & Queries

## Contents

1. [Introduction to SQL](#1-introduction-to-sql)
2. [What is a Database Schema?](#2-what-is-a-database-schema)
3. [What is Metadata?](#3-what-is-metadata)
4. [Types of Databases - Relational vs Non-Relational](#4-types-of-databases---relational-vs-non-relational)
5. [Sample Schema](#5-sample-schema-used-throughout-these-notes)
6. [DDL - Data Definition Language](#6-ddl---data-definition-language)
7. [DML - Insert, Update, Delete](#7-dml---insert-update-delete)
8. [DCL - Data Control Language](#8-dcl---data-control-language)
9. [DQL - SELECT Queries](#9-dql---select-queries)
10. [Joins](#10-joins---the-most-interview-critical-topic)
11. [Subqueries](#11-subqueries)
12. [Common Table Expressions (CTEs)](#12-common-table-expressions-ctes)
13. [Window Functions](#13-window-functions-advanced-frequently-asked-in-interviews)
14. [CASE Statements & Conditional Functions](#14-case-statements--conditional-functions)
15. [Set Operations](#15-set-operations)
16. [String Functions](#16-string-functions)
17. [Date & Time Functions](#17-date--time-functions)
18. [Mathematical Functions](#18-mathematical-functions)
19. [Indexes & Performance](#19-indexes--performance)
20. [Transactions (TCL)](#20-transactions-tcl---critical-for-banking-systems)
21. [Normalization](#21-normalization-quick-reference)
22. [Common Interview Query Patterns](#22-common-interview-query-patterns)
23. [NULL Handling](#23-null-handling)
24. [Views](#24-views)
25. [Temporary Tables](#25-temporary-tables)
26. [Stored Procedures & Functions](#26-stored-procedures--functions)
27. [Triggers](#27-triggers)
28. [Keys & Constraints](#28-keys--constraints)
29. [Data Types](#29-data-types-common-ones)
30. [Auto-Increment / Sequences](#30-auto-increment--sequences)
31. [JSON Functions](#31-json-functions)
32. [Entity-Relationship (ER) Concepts](#32-entity-relationship-er-concepts)
33. [Quick-Reference Cheat Sheet](#33-quick-reference-cheat-sheet)

---
## 1. Introduction to SQL
Definition: SQL (Structured Query Language) is the standard language used to create, read, update, and delete data in a relational database (MySQL, PostgreSQL, Oracle, SQL Server).
Why we use it: Applications need a reliable, structured way to store and retrieve data — think of a banking app that must save customer records, loan details, and payments. SQL gives every relational database a common language for this, so the same core skills transfer across MySQL, PostgreSQL, Oracle, etc. It also enforces structure (via schemas and constraints) so data stays consistent and relationships between tables (like customer → loan) are reliably maintained.
SQL commands are grouped into categories:
Category	Full Form	Commands	Why this category exists
DDL	Data Definition Language	CREATE, ALTER, DROP, TRUNCATE	Defines/changes the shape of the database (tables, columns)
DML	Data Manipulation Language	INSERT, UPDATE, DELETE	Changes the actual data stored inside tables
DQL	Data Query Language	SELECT	Reads/retrieves data without changing anything
DCL	Data Control Language	GRANT, REVOKE	Controls who has permission to access or modify data
TCL	Transaction Control Language	COMMIT, ROLLBACK, SAVEPOINT	Controls grouping of operations so they succeed/fail together
Separating these categories matters in practice: a backend developer writes mostly DML/DQL day-to-day, while DDL changes (schema migrations) are usually more controlled/reviewed since they affect the whole application, and DCL is typically managed by DBAs for security.
---
## 2. What is a Database Schema?
Definition: A schema is the overall blueprint/structure of a database — it defines what tables exist, what columns each table has, the data types of those columns, and how tables relate to each other (via keys). Think of it as the architectural plan for how data is organized, separate from the actual data itself.
Why we use it: Without a defined schema, there’d be no consistent way to know what shape the data should take — every insert could look different, making the data unreliable and nearly impossible to query predictably. A schema acts as a contract: any application reading from or writing to the database knows exactly what to expect (e.g., amount is always a DECIMAL, customer_id always links to a valid customer).
Types of schema (good to know for interviews): | Type | What it describes | Why it matters | |—|—|—| | Logical schema | The conceptual design — entities, attributes, relationships (independent of any specific database product) | Used during design/planning, so the structure is thought through before writing actual SQL | | Physical schema | The actual implementation — real tables, columns, data types, indexes as created in a specific database engine | This is what you interact with day-to-day when writing CREATE TABLE statements | | View schema (external schema) | A tailored subset/perspective of the data exposed to a specific user or application | Lets different users/apps see only the relevant slice of data without touching the full physical schema |
In MySQL/PostgreSQL specifically, “schema” also refers to a named container that groups related tables together (similar to a folder) — useful for separating, say, lending tables from reporting tables within the same database, and for managing access permissions per group.
---
## 3. What is Metadata?
Definition: Metadata is “data about data” — information that describes the structure, properties, and context of the actual data stored in a database, rather than the data values themselves. If a loans table row (101, 5, 'Home Loan', 1850000, ...) is the data, then the fact that amount is a DECIMAL(12,2) column, that loan_id is the primary key, and that the table was created on a certain date — that’s metadata.
Why we use it: The database engine, your SQL client, and your application all need to know the shape and rules of the data before they can work with it correctly — metadata is what makes that possible. It’s also what powers everyday conveniences like autocomplete in a SQL editor, validation before a bad insert is allowed, and tools that auto-generate documentation or ER diagrams from an existing database.
Types of metadata in a database:
Type	What it describes	Example
Structural metadata	The schema itself — tables, columns, data types, keys, constraints	“loans.amount is DECIMAL(12,2), NOT NULL”
Descriptive metadata	Human-readable context about what the data means	A column comment: “amount — the disbursed principal in INR”
Statistical metadata	Data profiling info the query optimizer uses	Row counts, index cardinality, value distribution — used to decide the fastest way to run a query
Operational/administrative metadata	Information about how/when data was managed	Created/modified timestamps, who ran a migration, last backup time
Where metadata lives — the system catalog / data dictionary:
Every relational database maintains its own internal metadata store, queryable like a normal table:
-- MySQL / most SQL databases: INFORMATION_SCHEMA holds metadata about every object
SELECT table_name, column_name, data_type, is_nullable
FROM INFORMATION_SCHEMA.COLUMNS
WHERE table_name = 'loans';

-- See all tables in the current database
SELECT table_name FROM INFORMATION_SCHEMA.TABLES
WHERE table_schema = 'banking_db';

-- MySQL shortcut commands (query INFORMATION_SCHEMA under the hood)
DESCRIBE loans;
SHOW CREATE TABLE loans;
SHOW TABLES;
Why this matters practically: Tools like MySQL Workbench, DBeaver, or an ORM (like Hibernate/JPA in a Java backend) don’t “guess” your table structure — they read it directly from this metadata layer, which is exactly how autocomplete, schema validation, and auto-generated entity classes work.
---
## 4. Types of Databases — Relational vs Non-Relational
Definition: Databases broadly split into two families based on how they structure and store data: relational (SQL) databases, which organize data into fixed tables with rows/columns and enforce a strict schema, and non-relational (NoSQL) databases, which store data in more flexible formats and often relax strict schema rules in favor of scalability and flexibility.
Why this classification matters: The right choice depends on the shape of your data and what you’re optimizing for. Highly structured, relationship-heavy data (like banking transactions, where correctness and consistency are non-negotiable) fits relational databases well. Rapidly changing, loosely structured, or extremely high-volume data (like user activity logs, product catalogs, or social media feeds) often fits non-relational databases better.
### Relational Databases (SQL)
Definition: Data is stored in structured tables with predefined columns and data types, and relationships between tables are enforced via keys (as covered throughout these notes).
Why we use them: Strong consistency (ACID transactions), a mature standard query language (SQL), and enforced data integrity make relational databases the right fit whenever correctness and structured relationships matter more than raw scale — which is exactly why banking systems are built on them.
Examples	Best suited for
MySQL, PostgreSQL, Oracle, SQL Server	Banking/financial systems, ERP, inventory management, anything needing strict data integrity and complex joins/reporting
### Non-Relational Databases (NoSQL)
Definition: Data is stored in flexible, often schema-less formats optimized for specific access patterns rather than general-purpose querying and joins.
Why we use them: When data doesn’t naturally fit rigid tables, when the schema needs to evolve quickly, or when the priority is horizontal scalability (spreading data across many servers) over strict consistency, NoSQL databases are often a better fit than forcing the data into relational tables.
Classification of NoSQL databases:
Type	How it stores data	Examples	Best suited for
Document-based	Data stored as JSON/BSON-like documents, each document can have a different structure	MongoDB, CouchDB	Content management, product catalogs, and any data with varying/nested attributes per record
Key-Value stores	Data stored as simple key → value pairs, extremely fast lookups	Redis, DynamoDB, Riak	Caching, session storage, real-time leaderboards — anywhere lookup speed by a known key matters most
Column-family (wide-column) stores	Data stored in column families rather than rows, optimized for reading/writing large volumes of a few specific columns across many rows	Apache Cassandra, HBase	Time-series data, IoT sensor data, large-scale logging/analytics
Graph databases	Data stored as nodes and edges, optimized for traversing relationships	Neo4j, Amazon Neptune	Social networks, fraud-detection networks, recommendation engines — anywhere relationships themselves are the primary thing being queried
Quick comparison (relational vs non-relational):
Aspect	Relational (SQL)	Non-Relational (NoSQL)
Schema	Fixed, defined upfront	Flexible / schema-less
Scaling	Primarily vertical (bigger server)	Primarily horizontal (more servers)
Consistency	Strong (ACID) by default	Often “eventual consistency” (BASE) for scale, though many now offer tunable consistency
Relationships	Enforced via foreign keys and joins	Typically handled at the application level (except graph DBs)
Best for	Structured, relationship-heavy, transaction-critical data	High-volume, rapidly evolving, or loosely structured data
Why this is relevant to a banking/backend background: Real-world systems are increasingly polyglot — a banking application might use a relational database (MySQL/PostgreSQL) for core ledger/transaction data where correctness is critical, while using Redis for session caching and MongoDB for storing flexible customer-support ticket data, all within the same overall system.
---
## 5. Sample Schema (used throughout these notes)
CREATE TABLE customers (
    customer_id   INT PRIMARY KEY,
    name          VARCHAR(100),
    city          VARCHAR(50),
    signup_date   DATE
);

CREATE TABLE loans (
    loan_id       INT PRIMARY KEY,
    customer_id   INT,
    loan_type     VARCHAR(30),   -- Home Loan, Personal Loan
    amount        DECIMAL(12,2),
    interest_rate DECIMAL(4,2),
    status        VARCHAR(20),   -- Active, Closed, Defaulted
    disbursed_on  DATE,
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
);

CREATE TABLE payments (
    payment_id    INT PRIMARY KEY,
    loan_id       INT,
    amount_paid   DECIMAL(12,2),
    payment_date  DATE,
    FOREIGN KEY (loan_id) REFERENCES loans(loan_id)
);
Why this schema: Three related tables (customers → loans → payments) mirror a real banking/lending system, so every example below reflects a genuine business scenario instead of abstract table1/table2 data — this makes the patterns directly reusable in real interviews and real work.
---
## 6. DDL — Data Definition Language
Definition: DDL commands define and modify the structure of database objects (tables, schemas) — not the data inside them. Most DDL statements auto-commit (cannot be rolled back).
Why we use it: Before any data can be stored, the database needs to know the shape of that data — what columns exist, their types, and how tables relate. DDL is how you set that shape up, and how you evolve it later (e.g., adding a phone column when the business starts collecting phone numbers).
•	CREATE — creates a new database object (table, view, index, etc.). Why: this is the starting point for storing any new kind of data — e.g., a new branches table when the bank starts tracking branch info.
•	ALTER — modifies the structure of an existing object (add/drop/modify columns). Why: requirements change over time; ALTER lets the schema evolve without rebuilding the table from scratch.
•	DROP — permanently deletes an object and its structure. Why: used to remove obsolete tables/objects entirely, e.g., a deprecated staging table no longer needed.
•	TRUNCATE — removes all rows from a table but keeps the structure intact (faster than DELETE, resets auto-increment). Why: useful for quickly wiping a table (e.g., a temporary/staging table) between batch loads, without the overhead of deleting rows one by one.
-- Create table
CREATE TABLE branches (
    branch_id INT PRIMARY KEY,
    branch_name VARCHAR(100),
    city VARCHAR(50)
);

-- Add a column
ALTER TABLE customers ADD COLUMN phone VARCHAR(15);

-- Modify a column
ALTER TABLE customers MODIFY COLUMN phone VARCHAR(20);

-- Drop a column
ALTER TABLE customers DROP COLUMN phone;

-- Delete all rows but keep structure (faster than DELETE, cannot rollback in some DBs)
TRUNCATE TABLE payments;

-- Delete entire table
DROP TABLE branches;
---
## 7. DML — Insert, Update, Delete
Definition: DML commands manipulate the data stored inside tables (as opposed to DDL, which manipulates structure). DML changes can be rolled back within a transaction.
Why we use it: Once the structure exists (via DDL), the application needs to actually create, change, and remove records as real-world events happen — a customer signs up, a loan gets closed, a bad payment record needs correcting. DML is how the running application interacts with data day-to-day.
•	INSERT — adds new row(s) to a table. Why: used whenever something new happens in the business — a new customer registers, a new loan is disbursed.
•	UPDATE — modifies existing row(s) matching a condition. Why: used when the state of existing data changes — a loan gets closed, an interest rate is revised.
•	DELETE — removes row(s) matching a condition (row-by-row, can be rolled back, slower than TRUNCATE). Why: used to remove specific incorrect or obsolete records (e.g., a duplicate payment entry) without affecting the rest of the table.
-- Insert single row
INSERT INTO customers (customer_id, name, city, signup_date)
VALUES (1, 'Piyush Kumar', 'Bangalore', '2023-01-15');

-- Insert multiple rows
INSERT INTO customers (customer_id, name, city, signup_date) VALUES
(2, 'Anita Rao', 'Chennai', '2023-02-10'),
(3, 'Rahul Singh', 'Delhi', '2023-03-05');

-- Update
UPDATE loans
SET status = 'Closed'
WHERE loan_id = 101;

-- Update with condition on joined logic
UPDATE loans
SET interest_rate = interest_rate - 0.5
WHERE loan_type = 'Home Loan' AND status = 'Active';

-- Delete
DELETE FROM payments WHERE payment_id = 55;
---
## 8. DCL — Data Control Language
Definition: DCL commands manage permissions and access control — who is allowed to do what within the database. The two commands are GRANT (give a privilege) and REVOKE (remove a privilege).
Why we use it: Not everyone who touches a database should have the same level of access — e.g., a reporting analyst may only need to SELECT from tables, while a backend application’s database user needs INSERT/UPDATE/DELETE too, and only a DBA should be able to DROP tables. DCL enforces this principle of least privilege directly at the database level, which matters enormously in regulated environments like banking, where unauthorized data access or accidental schema changes can have serious compliance consequences.
-- Grant a user read-only access to the loans table
GRANT SELECT ON loans TO 'report_user'@'localhost';

-- Grant full data-manipulation access (but not structural changes)
GRANT SELECT, INSERT, UPDATE, DELETE ON loans TO 'app_user'@'localhost';

-- Grant all privileges on the entire database
GRANT ALL PRIVILEGES ON banking_db.* TO 'admin_user'@'localhost';

-- Revoke a previously granted privilege
REVOKE INSERT, UPDATE, DELETE ON loans FROM 'report_user'@'localhost';

-- Apply changes immediately
FLUSH PRIVILEGES;
Common privilege types: SELECT, INSERT, UPDATE, DELETE, CREATE, DROP, ALTER, EXECUTE (for procedures/functions), ALL PRIVILEGES.
---
## 9. DQL — SELECT Queries
Definition: DQL is used purely to retrieve/read data from one or more tables without changing it. SELECT is the only DQL command, but it’s the most powerful and frequently used statement in SQL.
Why we use it: Storing data is only half the job — the business constantly needs to ask questions of that data: which loans are active, what’s the average interest rate, which customers are overdue. SELECT (with its many clauses) is how every one of those questions gets answered, which is why it’s the command developers write most often.
### 9.1. Basic SELECT
Definition: Retrieves specific columns (or all columns with *) from a table. DISTINCT removes duplicate rows from the result set.
Why we use it: You rarely need every column in a table for a given task — selecting only the needed columns (rather than *) reduces the amount of data transferred and makes queries faster and easier to read. DISTINCT is used when you only care about unique values — e.g., listing which cities have customers, without repeating a city for every customer in it.
SELECT name, city FROM customers;

SELECT * FROM loans WHERE status = 'Active';

SELECT DISTINCT city FROM customers;
### 9.2. Filtering — WHERE, AND/OR, IN, BETWEEN, LIKE
Definitions & why we use them: - WHERE — filters rows based on a condition, evaluated before grouping. Why: almost no real query wants every row in a table — WHERE narrows results to exactly what’s relevant (e.g., only active loans). - AND / OR — combine multiple conditions (AND = both true, OR = either true). Why: real business rules are rarely single-condition — e.g., “active AND high-value” loans needs both to be true. - IN — checks if a value matches any value in a given list. Why: cleaner and often faster than writing multiple OR conditions for the same column (e.g., loan_type = ‘Home Loan’ OR loan_type = ‘Personal Loan’). - BETWEEN — checks if a value falls within an inclusive range. Why: common for range-based business questions like “loans between ₹5L and ₹20L” or a date range. - LIKE — pattern matching using wildcards (% = any number of characters, _ = exactly one character). Why: used for partial text matches — e.g., searching customers by the start of their name, without knowing the exact full value.
-- Loans between 5L and 20L
SELECT * FROM loans
WHERE amount BETWEEN 500000 AND 2000000;

-- Multiple loan types
SELECT * FROM loans
WHERE loan_type IN ('Home Loan', 'Personal Loan');

-- Pattern match (customers whose name starts with 'A')
SELECT * FROM customers WHERE name LIKE 'A%';

-- Combined condition
SELECT * FROM loans
WHERE status = 'Active' AND amount > 1000000;
### 9.3. Sorting & Limiting
Definitions & why we use them: - ORDER BY — sorts the result set by one or more columns (ASC default, or DESC). Why: raw table order is not meaningful (often insertion order) — sorting is needed to answer questions like “highest loan first” or “most recent payments first.” - LIMIT — restricts the number of rows returned (used with OFFSET to skip rows, e.g. for pagination). Why: prevents pulling huge result sets when only a few rows matter (e.g., “top 5 loans”), and powers pagination in UIs so a page shows 20 records at a time instead of the whole table.
SELECT * FROM loans ORDER BY amount DESC;

SELECT * FROM loans ORDER BY amount DESC LIMIT 5;   -- Top 5 highest loans
### 9.4. Aggregate Functions
Definition: Aggregate functions perform a calculation across a set of rows and return a single summarized value.
Why we use them: Businesses rarely care about one row at a time — they care about totals, averages, and extremes for reporting and decision-making (e.g., “how much have we disbursed in total,” “what’s our average interest rate”). Aggregates let the database do this math directly instead of pulling all rows and calculating in application code.
•	COUNT() — number of rows (or non-NULL values in a column). Why: answers “how many” questions — e.g., how many loans are active.
•	SUM() — total of a numeric column. Why: answers “how much in total” — e.g., total amount disbursed.
•	AVG() — average of a numeric column. Why: answers typical/central-tendency questions — e.g., average interest rate across all loans.
•	MAX() / MIN() — highest / lowest value in a column. Why: answers extremes — e.g., the largest loan ever disbursed.
SELECT COUNT(*) AS total_loans FROM loans;

SELECT SUM(amount) AS total_disbursed FROM loans WHERE status = 'Active';

SELECT AVG(interest_rate) AS avg_rate FROM loans;

SELECT MAX(amount) AS highest_loan, MIN(amount) AS lowest_loan FROM loans;
### 9.5. GROUP BY & HAVING
Definitions & why we use them: - GROUP BY — groups rows that share the same value(s) in specified column(s), typically used with aggregate functions to summarize per group. Why: used whenever you need a summary per category rather than one grand total — e.g., total loan amount per loan type, not just overall. - HAVING — filters groups after aggregation (unlike WHERE, which filters individual rows before grouping). Why: you can’t filter on an aggregate result (like SUM(amount) > X) using WHERE, because WHERE runs before the aggregation happens — HAVING is specifically built to filter the summarized groups.
-- Total loan amount per loan type
SELECT loan_type, SUM(amount) AS total_amount
FROM loans
GROUP BY loan_type;

-- Loan types where total disbursed amount exceeds 1 crore
SELECT loan_type, SUM(amount) AS total_amount
FROM loans
GROUP BY loan_type
HAVING SUM(amount) > 10000000;

-- Number of loans per customer, only customers with more than 1 loan
SELECT customer_id, COUNT(*) AS loan_count
FROM loans
GROUP BY customer_id
HAVING COUNT(*) > 1;
 
Colorful bar-chart style screenshot of a GROUP BY query result, showing total loan amount per loan type
### 9.6. Advanced Aggregation (COUNT DISTINCT, GROUP_CONCAT, ROLLUP)
Why this matters: Basic GROUP BY gives one summary row per group, but real reporting often needs a bit more — counting only unique values, combining grouped values into a readable list, or getting subtotal + grand-total rows in a single query instead of running multiple queries.
•	COUNT(DISTINCT column) — counts only unique non-NULL values in a column. Why: used when duplicates would otherwise inflate a count — e.g., counting how many distinct cities customers come from, not just how many customer rows exist.
SELECT COUNT(DISTINCT city) AS unique_cities FROM customers;
•	GROUP_CONCAT() (MySQL) / STRING_AGG() (PostgreSQL/SQL Server) — combines values from multiple rows in a group into a single delimited string. Why: useful for producing a compact, human-readable summary per group — e.g., listing all loan types a customer holds as one comma-separated string, instead of one row per loan type.
SELECT customer_id, GROUP_CONCAT(loan_type SEPARATOR ', ') AS loan_types
FROM loans
GROUP BY customer_id;
•	WITH ROLLUP — adds extra summary rows to a GROUP BY result: subtotals for each group plus a grand total row. Why: saves writing a separate query for the grand total — a single query can produce a full report structure (per-group totals + overall total) directly, which is common in financial/management reports.
SELECT loan_type, SUM(amount) AS total_amount
FROM loans
GROUP BY loan_type WITH ROLLUP;
-- Returns one row per loan_type, plus a final row with loan_type = NULL representing the grand total
### 9.7. Query Execution Order (why some things don’t work the way you’d expect)
Definition: SQL is written in one order (SELECT ... FROM ... WHERE ... GROUP BY ... HAVING ... ORDER BY ... LIMIT), but the database engine actually processes it in a different logical order:
1. FROM        -- identify the source table(s)
2. JOIN        -- combine with other tables
3. WHERE       -- filter individual rows
4. GROUP BY    -- group the filtered rows
5. HAVING      -- filter the grouped results
6. SELECT      -- compute the output columns/expressions
7. DISTINCT    -- remove duplicate rows from the output
8. ORDER BY    -- sort the final result
9. LIMIT       -- restrict the number of rows returned
Why this matters: This explains several “gotchas” that confuse people early on: - You can’t use a column alias defined in SELECT inside a WHERE clause, because WHERE runs before SELECT computes that alias. - You can use a SELECT alias inside ORDER BY, because ORDER BY runs after SELECT. - WHERE can’t filter on an aggregate function result (like SUM(amount) > 1000000) — that’s exactly why HAVING exists, since it runs after grouping/aggregation.
-- This FAILS in most databases — alias not yet available at WHERE-time
-- SELECT amount * 1.1 AS updated_amount FROM loans WHERE updated_amount > 100000;

-- This WORKS — alias is available at ORDER BY time
SELECT amount * 1.1 AS updated_amount FROM loans ORDER BY updated_amount DESC;
---
## 10. Joins — the most interview-critical topic
Definition: A JOIN combines rows from two or more tables based on a related column between them (usually a primary key / foreign key relationship), letting you query data spread across multiple tables as a single result set.
Why we use it: In a well-designed (normalized) database, related data lives in separate tables to avoid duplication — customer details live in customers, loan details in loans. But most real questions need both together (e.g., “show me each customer’s loan amount”) — JOINs are how you reconnect that split data at query time, without duplicating customer info inside the loans table.
 
Colorful screenshot of a JOIN query result table with color-coded loan status badges
-- INNER JOIN: only matching rows in both tables
SELECT c.name, l.loan_type, l.amount
FROM customers c
INNER JOIN loans l ON c.customer_id = l.customer_id;

-- LEFT JOIN: all customers, even those with no loans
SELECT c.name, l.loan_type, l.amount
FROM customers c
LEFT JOIN loans l ON c.customer_id = l.customer_id;

-- RIGHT JOIN: all loans, even if customer record is missing (rare in practice)
SELECT c.name, l.loan_type
FROM customers c
RIGHT JOIN loans l ON c.customer_id = l.customer_id;

-- FULL OUTER JOIN (MySQL doesn't support directly — emulate with UNION)
SELECT c.name, l.loan_type
FROM customers c LEFT JOIN loans l ON c.customer_id = l.customer_id
UNION
SELECT c.name, l.loan_type
FROM customers c RIGHT JOIN loans l ON c.customer_id = l.customer_id;

-- SELF JOIN example: customers in the same city
SELECT a.name AS customer1, b.name AS customer2, a.city
FROM customers a
JOIN customers b ON a.city = b.city AND a.customer_id <> b.customer_id;

-- Multi-table join: customer -> loan -> payments
SELECT c.name, l.loan_type, p.amount_paid, p.payment_date
FROM customers c
JOIN loans l ON c.customer_id = l.customer_id
JOIN payments p ON l.loan_id = p.loan_id;
Quick reference: | Join Type | Returns | Why you’d use it | |—|—|—| | INNER JOIN | Only matching rows in both tables | You only care about records that definitely have a relationship on both sides — e.g., customers who actually have a loan | | LEFT JOIN | All rows from left + matches from right (NULL if none) | You want everything from the “main” table even if related data doesn’t exist — e.g., all customers, showing NULL for those with no loan | | RIGHT JOIN | All rows from right + matches from left | Same idea as LEFT JOIN, but the table you must keep in full is on the right | | FULL OUTER JOIN | All rows from both, matched where possible | You need a complete picture from both sides, including unmatched rows on either side — e.g., auditing for orphaned records | | SELF JOIN | Table joined with itself | You need to compare rows within the same table — e.g., customers who share a city | | CROSS JOIN | Cartesian product (every row × every row) | You need every possible combination — e.g., generating all (branch × product) combinations for a report template |
---
## 11. Subqueries
Definition: A subquery (or inner query) is a query nested inside another query. It runs first (or, if correlated, once per outer row) and its result is used by the outer query.
Why we use them: Some questions naturally depend on the answer to a smaller question first — e.g., “which loans are above average” requires first computing the average. Subqueries let you express this dependency directly in one query instead of running two separate queries and combining results manually in application code.
•	Scalar subquery — returns a single value, used in SELECT or comparisons. Why: useful when a single computed number (like an average or a per-row total) needs to be compared against or displayed alongside other columns.
•	Row/column subquery — returns a list of values, typically used with IN / NOT IN. Why: useful when checking membership against a dynamically computed list, e.g., customers who appear in a filtered subset of loans.
•	Correlated subquery — references a column from the outer query, so it re-executes for every row of the outer query (slower, but powerful for row-by-row comparisons like defaulter detection below). Why: needed when the “smaller question” is different for every row — e.g., “has this specific loan been fully paid,” which depends on that loan’s own payments.
-- Customers who have taken a Home Loan (subquery in WHERE)
SELECT name FROM customers
WHERE customer_id IN (
    SELECT customer_id FROM loans WHERE loan_type = 'Home Loan'
);

-- Loans with amount above the average loan amount
SELECT * FROM loans
WHERE amount > (SELECT AVG(amount) FROM loans);

-- Correlated subquery: customers whose total payments are below their loan amount (defaulters)
SELECT c.name, l.amount
FROM customers c
JOIN loans l ON c.customer_id = l.customer_id
WHERE l.amount > (
    SELECT COALESCE(SUM(p.amount_paid), 0)
    FROM payments p
    WHERE p.loan_id = l.loan_id
);

-- Subquery in SELECT (scalar subquery)
SELECT l.loan_id, l.amount,
       (SELECT SUM(p.amount_paid) FROM payments p WHERE p.loan_id = l.loan_id) AS total_paid
FROM loans l;
---
## 12. Common Table Expressions (CTEs)
Definition: A CTE (defined with the WITH clause) is a named, temporary result set that exists only for the duration of a single query. It behaves like a subquery, but is defined upfront and can be referenced by name — including, in a recursive CTE, referencing itself.
Why we use them: Deeply nested subqueries become hard to read and debug. A CTE lets you break a complex query into named, logical steps (like naming variables in code), making it much easier to follow and maintain. Recursive CTEs solve a category of problems plain SQL can’t handle at all — traversing hierarchical/tree-structured data (org charts, category trees, bill-of-materials) where the depth isn’t known in advance.
### Basic (non-recursive) CTE
WITH high_value_loans AS (
    SELECT loan_id, customer_id, amount
    FROM loans
    WHERE amount > 1000000
)
SELECT c.name, h.amount
FROM high_value_loans h
JOIN customers c ON c.customer_id = h.customer_id;
Why this is better than a subquery here: high_value_loans is named and readable, and could be reused multiple times later in the same query without repeating the subquery logic.
### Multiple CTEs in one query
WITH active_loans AS (
    SELECT * FROM loans WHERE status = 'Active'
),
loan_totals AS (
    SELECT customer_id, SUM(amount) AS total_active_amount
    FROM active_loans
    GROUP BY customer_id
)
SELECT c.name, lt.total_active_amount
FROM loan_totals lt
JOIN customers c ON c.customer_id = lt.customer_id;
Recursive CTE — traversing hierarchical data
Why: Useful for org charts, category trees, or (in banking) a loan approval hierarchy where each approver reports to another — something a normal JOIN can’t handle because the number of levels isn’t fixed.
-- Example: an employee hierarchy (manager_id references employee_id)
WITH RECURSIVE employee_hierarchy AS (
    -- Anchor member: top-level employees (no manager)
    SELECT employee_id, name, manager_id, 1 AS level
    FROM employees
    WHERE manager_id IS NULL

    UNION ALL

    -- Recursive member: employees reporting to someone already in the hierarchy
    SELECT e.employee_id, e.name, e.manager_id, eh.level + 1
    FROM employees e
    JOIN employee_hierarchy eh ON e.manager_id = eh.employee_id
)
SELECT * FROM employee_hierarchy
ORDER BY level;
---
## 13. Window Functions (advanced, frequently asked in interviews)
Definition: A window function performs a calculation across a set of rows related to the current row — without collapsing them into a single output row (unlike GROUP BY). Defined using the OVER() clause.
Why we use them: GROUP BY collapses many rows into one summary row per group, which loses the individual row detail. Window functions solve cases where you need both — the detail rows and a calculation across a related set of rows, like ranking each loan within its type while still showing every loan’s own row, or a running balance over time.
•	PARTITION BY — divides rows into groups (“windows”) the function operates on independently. Why: lets you reset the calculation per group — e.g., rank restarts for each loan_type instead of ranking across the whole table.
•	RANK() — assigns a rank within a partition, with gaps after ties (1, 2, 2, 4). Why: used for leaderboard-style ranking where tied values should still “use up” a rank position.
•	DENSE_RANK() — like RANK but without gaps after ties (1, 2, 2, 3). Why: used when you want consecutive rank numbers even with ties, e.g., for a fixed number of “tiers.”
•	ROW_NUMBER() — assigns a unique sequential number to each row within a partition, no ties. Why: commonly used to pick exactly “the first/last row per group” — e.g., the most recent payment per loan.
•	LAG() / LEAD() — access a value from a previous / next row within the partition. Why: used for row-to-row comparisons over time, e.g., comparing each payment to the one before it to spot missed or reduced payments.
-- Rank loans by amount within each loan_type
SELECT loan_id, loan_type, amount,
       RANK() OVER (PARTITION BY loan_type ORDER BY amount DESC) AS rank_in_type,
       DENSE_RANK() OVER (PARTITION BY loan_type ORDER BY amount DESC) AS dense_rank_in_type
FROM loans;
 
Colorful screenshot of a window function query result, showing RANK restarting at 1 for each loan_type partition
-- Running total of payments per loan
SELECT loan_id, payment_date, amount_paid,
       SUM(amount_paid) OVER (PARTITION BY loan_id ORDER BY payment_date) AS running_total
FROM payments;

-- ROW_NUMBER to find the latest payment per loan
SELECT * FROM (
    SELECT p.*, ROW_NUMBER() OVER (PARTITION BY loan_id ORDER BY payment_date DESC) AS rn
    FROM payments p
) t
WHERE rn = 1;

-- LEAD/LAG: compare each payment with the previous one
SELECT loan_id, payment_date, amount_paid,
       LAG(amount_paid) OVER (PARTITION BY loan_id ORDER BY payment_date) AS previous_payment
FROM payments;
---
## 14. CASE Statements & Conditional Functions
### 14.1. CASE Statement
Definition: CASE provides if/else-style conditional logic inside a SQL query. It evaluates conditions in order and returns the value from the first WHEN that matches; if none match, it returns the ELSE value (or NULL if omitted).
Why we use it: Sometimes raw data needs to be turned into a business-friendly category directly in the query — e.g., turning a raw loan amount into “High/Medium/Low Value” for a report — rather than pulling raw numbers and classifying them afterward in application code.
SELECT loan_id, amount,
    CASE
        WHEN amount >= 2000000 THEN 'High Value'
        WHEN amount >= 500000 THEN 'Medium Value'
        ELSE 'Low Value'
    END AS loan_category
FROM loans;

-- "Simple CASE" form — compares one expression against a list of values
SELECT loan_id, loan_type,
    CASE loan_type
        WHEN 'Home Loan' THEN 'HL'
        WHEN 'Personal Loan' THEN 'PL'
        ELSE 'OTHER'
    END AS loan_code
FROM loans;
### 14.2. IF() — inline conditional (MySQL)
Definition: IF(condition, value_if_true, value_if_false) is a compact two-branch conditional — essentially a shorthand CASE for simple true/false logic.
Why we use it: Quicker to write than a full CASE when there are only two outcomes — e.g., flagging a loan as overdue or not, without the extra CASE/WHEN/END syntax.
SELECT loan_id, status,
       IF(status = 'Defaulted', 'Risk', 'Safe') AS risk_flag
FROM loans;
### 14.3. NULLIF() — turning a specific value into NULL
Definition: NULLIF(expr1, expr2) returns NULL if the two expressions are equal, otherwise returns expr1.
Why we use it: Useful for avoiding errors or misleading results caused by a specific “placeholder” value — e.g., preventing a divide-by-zero error, or treating a 0% interest rate as “not set” rather than a real value in a calculation.
-- Avoid divide-by-zero: if amount is 0, treat the denominator as NULL instead of erroring
SELECT loan_id, amount, interest_rate,
       amount / NULLIF(interest_rate, 0) AS amount_per_rate_point
FROM loans;
### 14.4. COALESCE() and IFNULL() (conditional fallback — recap)
Definition: Already covered under NULL Handling — COALESCE() returns the first non-NULL value from a list, and IFNULL() is MySQL’s two-value shorthand. They’re conditional in nature (a value “if NULL, then…”) so they’re commonly grouped with conditional functions in interviews.
Why we use it: Ensures a query never surfaces a raw, confusing NULL to a report or application — instead substituting a sensible business default (e.g., “Unknown” or 0).
### 14.5. GREATEST() and LEAST()
Definition: GREATEST(a, b, c, ...) returns the largest value among its arguments; LEAST(a, b, c, ...) returns the smallest.
Why we use it: Useful for row-wise (not column-wise) comparisons — e.g., picking the higher of two possible interest rates for a customer based on two different rate calculation methods, without writing a CASE statement.
SELECT loan_id,
       GREATEST(interest_rate, 8.5) AS effective_min_rate
FROM loans;
---
## 15. Set Operations
Definition: Set operations combine the results of two or more SELECT queries (which must have the same number of columns and compatible data types) into a single result set.
Why we use them: Sometimes the data you need doesn’t come from one join-able relationship, but from comparing two independent result sets as whole “sets” — e.g., which customers have Home Loans vs Personal Loans, and how those two groups overlap.
•	UNION — combines results from two queries and removes duplicate rows. Why: used to merge two similar result sets into one list without repeating entries that appear in both.
•	UNION ALL — combines results and keeps duplicates (faster, since no duplicate-check is done). Why: used when duplicates are meaningful (or don’t exist) and you want the performance benefit of skipping the de-duplication step.
•	INTERSECT — returns only rows present in both result sets. Why: used to find overlap between two groups — e.g., customers who hold both loan types.
•	EXCEPT / MINUS — returns rows from the first query that do not appear in the second. Why: used to find what’s unique to one group — e.g., customers with a Home Loan but no Personal Loan, useful for cross-sell targeting.
-- UNION removes duplicates, UNION ALL keeps them
SELECT customer_id FROM loans WHERE loan_type = 'Home Loan'
UNION
SELECT customer_id FROM loans WHERE loan_type = 'Personal Loan';

-- INTERSECT: customers who have BOTH loan types (MySQL 8.0.31+ / PostgreSQL)
SELECT customer_id FROM loans WHERE loan_type = 'Home Loan'
INTERSECT
SELECT customer_id FROM loans WHERE loan_type = 'Personal Loan';

-- EXCEPT / MINUS: customers with Home Loan but NOT Personal Loan
SELECT customer_id FROM loans WHERE loan_type = 'Home Loan'
EXCEPT
SELECT customer_id FROM loans WHERE loan_type = 'Personal Loan';
---
## 16. String Functions
Definition: String functions manipulate and transform text data stored in VARCHAR/CHAR/TEXT columns.
Why we use them: Raw stored text often needs cleaning, combining, or reformatting before it’s useful — e.g., combining first and last name for display, standardizing case for comparison, or trimming accidental whitespace from user input before saving it.
Function	What it does	Example use case
CONCAT(a, b, ...)	Joins strings together	Combine first_name + last_name into a full display name
SUBSTRING(str, start, len)	Extracts part of a string	Pull out the first 4 digits of an account number
TRIM(str)	Removes leading/trailing whitespace	Clean up user-entered data before storing/comparing it
REPLACE(str, from, to)	Replaces occurrences of a substring	Mask part of a sensitive value, e.g., replacing digits with *
UPPER(str) / LOWER(str)	Converts case	Normalize text for case-insensitive comparisons
LENGTH(str)	Returns the character/byte length	Validate that an input meets a minimum/maximum length rule
CONCAT_WS(sep, a, b, ...)	Joins strings with a separator, skipping NULLs	Build a clean “City, State” style string even if one part is missing
SELECT
    CONCAT(name, ' (', city, ')') AS display_name,
    UPPER(city) AS city_upper,
    LENGTH(name) AS name_length,
    SUBSTRING(name, 1, 3) AS name_prefix,
    TRIM('  Piyush  ') AS trimmed_name
FROM customers;
---
## 17. Date & Time Functions
Definition: Date/time functions perform calculations and extractions on DATE, DATETIME, and TIMESTAMP columns.
Why we use them: Business logic constantly depends on time — loan tenure, days overdue, monthly reports, age of an account. Doing this math in SQL (rather than pulling raw dates and calculating in application code) keeps it fast and consistent, especially across large datasets.
Function	What it does	Example use case
NOW() / CURDATE()	Current date+time / current date	Stamp when a record was created, or compare against “today”
DATEDIFF(date1, date2)	Number of days between two dates	Calculate how many days a loan has been overdue
DATE_ADD(date, INTERVAL n unit)	Adds a time interval to a date	Calculate a loan’s due date, e.g., disbursed_on + 1 year
DATE_SUB(date, INTERVAL n unit)	Subtracts a time interval from a date	Find loans disbursed in the last 90 days
EXTRACT(unit FROM date)	Pulls out a specific part (year, month, day)	Group loans by disbursement year for a report
TIMESTAMPDIFF(unit, date1, date2)	Difference between two datetimes in a chosen unit	Calculate a customer’s tenure in months since signup
DATE_FORMAT(date, format)	Formats a date as a string	Display dates as ‘DD-MON-YYYY’ for a report
SELECT loan_id,
       disbursed_on,
       DATEDIFF(CURDATE(), disbursed_on) AS days_since_disbursed,
       DATE_ADD(disbursed_on, INTERVAL 1 YEAR) AS first_review_date,
       EXTRACT(YEAR FROM disbursed_on) AS disbursed_year
FROM loans
WHERE disbursed_on >= DATE_SUB(CURDATE(), INTERVAL 90 DAY);
---
## 18. Mathematical Functions
Definition: Mathematical functions perform numeric calculations directly within a query.
Why we use them: Financial calculations (rounding interest, ensuring a value is never negative, computing remainders) need to be precise and consistent — doing this in SQL avoids inconsistent rounding logic scattered across different parts of an application.
Function	What it does	Example use case
ROUND(n, d)	Rounds a number to d decimal places	Round a computed interest amount to 2 decimal places for display
CEIL(n) / CEILING(n)	Rounds up to the nearest integer	Calculate the minimum number of EMIs needed to cover a loan
FLOOR(n)	Rounds down to the nearest integer	Determine completed full years for a tenure-based rule
MOD(n, d) (or %)	Remainder after division	Distribute records evenly across batches, e.g., batch = id MOD 10
ABS(n)	Absolute (non-negative) value	Get the magnitude of a variance between expected and actual payment
POWER(n, p)	Raises a number to a power	Compound interest calculations
SELECT loan_id,
       amount,
       ROUND(amount * interest_rate / 100, 2) AS annual_interest,
       CEIL(amount / 12) AS min_monthly_installment,
       ABS(amount - 1000000) AS variance_from_10L
FROM loans;
---
## 19. Indexes & Performance
Definition: An index is a separate data structure (typically a B-Tree) that stores a sorted reference to table data, allowing the database to find rows without scanning the entire table — similar to an index at the back of a book.
Why we use them: Without an index, the database has to scan every row (a “full table scan”) to find matches — fine for a few hundred rows, but very slow on millions of rows (e.g., looking up a customer’s loans in a banking table with millions of records). An index lets the database jump almost directly to the matching rows, which is critical for production performance.
•	Primary index — automatically created on the PRIMARY KEY. Why: the primary key is used to identify/look up specific rows constantly (e.g., fetching one loan by loan_id), so it’s indexed by default.
•	Composite index — an index on multiple columns together; column order matters (leftmost-prefix rule). Why: used when queries commonly filter on the same combination of columns together, e.g., loan_type + status, so both conditions benefit from one index.
•	EXPLAIN — shows the query execution plan, revealing whether indexes are being used. Why: used to diagnose slow queries — if EXPLAIN shows a full table scan where an index was expected, that’s a sign the index is missing, unused, or the query needs rewriting.
-- Create an index to speed up frequent lookups
CREATE INDEX idx_customer_id ON loans(customer_id);

-- Composite index (order matters — leftmost column used first)
CREATE INDEX idx_type_status ON loans(loan_type, status);

-- Check query plan (MySQL)
EXPLAIN SELECT * FROM loans WHERE customer_id = 10;

-- Drop an index
DROP INDEX idx_customer_id ON loans;
Key points: - Index the columns used in WHERE, JOIN, and ORDER BY clauses. - Too many indexes slow down INSERT/UPDATE/DELETE (each index must be maintained). - A PRIMARY KEY and often FOREIGN KEY columns are automatically indexed.
---
## 20. Transactions (TCL) — critical for banking systems
Definition: A transaction is a sequence of one or more SQL operations executed as a single logical unit of work — either all operations succeed, or none do. This is essential in banking systems (e.g., debiting one account and crediting another must both succeed or both fail).
Why we use them: Without transactions, a failure partway through a multi-step operation (e.g., the app crashes after marking a loan “Closed” but before recording the final payment) would leave the data in an inconsistent, half-updated state. Transactions guarantee that related changes happen together or not at all — directly protecting data integrity in financial systems.
•	START TRANSACTION / BEGIN — marks the start of a transaction. Why: tells the database to treat everything that follows as one unit, not independent statements.
•	COMMIT — permanently saves all changes made in the transaction. Why: used once every step has succeeded, to make the changes permanent and visible to other users/sessions.
•	ROLLBACK — undoes all changes made since the transaction began. Why: used when something fails partway through, to cleanly undo partial changes rather than leaving inconsistent data.
•	SAVEPOINT — marks an intermediate point within a transaction that you can roll back to, without undoing the entire transaction. Why: useful in longer transactions where only part of the work needs to be undone on error, without discarding everything already done.
START TRANSACTION;

UPDATE loans SET status = 'Closed' WHERE loan_id = 101;
INSERT INTO payments (payment_id, loan_id, amount_paid, payment_date)
VALUES (500, 101, 50000, CURDATE());

-- If everything succeeded:
COMMIT;

-- If something failed:
ROLLBACK;
-- SAVEPOINT example
START TRANSACTION;
UPDATE loans SET status = 'Active' WHERE loan_id = 102;
SAVEPOINT sp1;
UPDATE loans SET amount = amount + 10000 WHERE loan_id = 102;
ROLLBACK TO sp1;  -- undoes only the second update
COMMIT;
ACID Properties (why transactions are trustworthy): | Property | Meaning | Why it matters | |—|—|—| | Atomicity | All operations in a transaction succeed or none do | Prevents “half-done” updates — e.g., money leaving one account without arriving in another | | Consistency | Database moves from one valid state to another | Ensures rules (constraints, relationships) are never violated, even mid-transaction | | Isolation | Concurrent transactions don’t interfere | Prevents two simultaneous transactions (e.g., two payments processing at once) from corrupting each other’s results | | Durability | Committed data survives crashes | Guarantees that once a transaction is confirmed (e.g., a payment is confirmed to the customer), it won’t be silently lost if the server crashes right after |
### 20.1. Transaction Isolation Levels
Definition: The Isolation property in ACID isn’t all-or-nothing — it’s tunable. An isolation level controls exactly how much one transaction can “see” of another transaction’s uncommitted or concurrently-changing data.
Why we use them: Full isolation (the strictest level) is the safest, but also the slowest, since it forces transactions to wait on each other more. Different parts of a system have different needs — a real-time balance check might need strict correctness, while a background analytics report can tolerate slightly stale data for better performance. Isolation levels let you choose the right trade-off between consistency and speed for a given use case.
Level	Prevents	Allows	Typical use
READ UNCOMMITTED	Nothing	Dirty reads (reading another transaction’s uncommitted changes)	Rarely used — only where absolute performance matters more than correctness
READ COMMITTED	Dirty reads	Non-repeatable reads (same row read twice in one transaction gives different results)	Common default (e.g., PostgreSQL, Oracle) — good balance for most applications
REPEATABLE READ	Dirty reads, non-repeatable reads	Phantom reads (a repeated query returns new rows that appeared after the transaction started)	MySQL’s default — needed when a transaction must see a consistent snapshot of existing rows throughout
SERIALIZABLE	All of the above	Nothing — transactions behave as if run one at a time	Used for the most sensitive financial operations, e.g., processing money transfers where correctness matters more than throughput
-- Set isolation level for the current session
SET SESSION TRANSACTION ISOLATION LEVEL READ COMMITTED;

START TRANSACTION;
SELECT amount FROM loans WHERE loan_id = 101;  -- read within this isolation level
-- ... business logic ...
COMMIT;
Key terms: - Dirty read — reading data another transaction has changed but not yet committed (risky — that change might get rolled back). - Non-repeatable read — re-reading the same row within a transaction and getting a different value because another transaction updated and committed it in between. - Phantom read — re-running the same query within a transaction and getting additional/different rows because another transaction inserted/deleted matching rows in between.
---
## 21. Normalization (quick reference)
Definition: Normalization is the process of organizing tables and columns to minimize data redundancy and avoid update/insert/delete anomalies, by splitting data into related tables according to a set of formal rules (“normal forms”).
Why we use it: Without normalization, the same data gets repeated across many rows (e.g., a customer’s city stored on every single loan row), which wastes space and, worse, creates a risk that the same fact gets updated in one place but not another — causing inconsistent data. Normalization fixes this by storing each fact in exactly one place and linking tables via keys instead.
Normal Form	Rule	Why it matters
1NF	Atomic values, no repeating groups	Avoids storing multiple values crammed into a single column, which makes filtering/searching that data unreliable
2NF	1NF + no partial dependency on part of a composite key	Ensures a column truly depends on the whole key, not just part of it — avoiding redundant partial data
3NF	2NF + no transitive dependency (non-key column depends on another non-key column)	Prevents indirect redundancy, e.g., storing a branch’s city on the loan table just because it depends on the branch, not the loan
BCNF	Every determinant is a candidate key	A stricter version of 3NF that closes edge cases 3NF misses, for very high data integrity needs
Example (violates 1NF → fixed):
BAD:  loan_id | products         →  (101, 'Home Loan, Insurance')
GOOD: loan_id | product          →  (101, 'Home Loan'), (101, 'Insurance')
---
## 22. Common Interview Query Patterns
Why this section exists: These specific patterns (Nth highest value, duplicates, anti-joins, running trends) come up repeatedly across interviews and real reporting tasks because they represent recurring real-world questions — “who’s missing,” “what’s the outlier,” “what’s the trend over time” — so they’re worth memorizing as reusable templates rather than solving from scratch each time.
-- 1. Second highest loan amount
SELECT MAX(amount) AS second_highest
FROM loans
WHERE amount < (SELECT MAX(amount) FROM loans);

-- Alternative using LIMIT/OFFSET
SELECT DISTINCT amount FROM loans
ORDER BY amount DESC
LIMIT 1 OFFSET 1;

-- 2. Nth highest salary/amount (generic pattern) — 3rd highest
SELECT DISTINCT amount FROM loans
ORDER BY amount DESC
LIMIT 1 OFFSET 2;

-- 3. Find duplicate rows
SELECT customer_id, COUNT(*)
FROM loans
GROUP BY customer_id, loan_type
HAVING COUNT(*) > 1;

-- 4. Delete duplicate rows, keep the lowest id
DELETE l1 FROM loans l1
JOIN loans l2
  ON l1.customer_id = l2.customer_id
 AND l1.loan_type = l2.loan_type
 AND l1.loan_id > l2.loan_id;

-- 5. Customers who never took a loan
SELECT c.name
FROM customers c
LEFT JOIN loans l ON c.customer_id = l.customer_id
WHERE l.loan_id IS NULL;

-- 6. Month-wise disbursed loan totals (trend analysis)
SELECT DATE_FORMAT(disbursed_on, '%Y-%m') AS month, SUM(amount) AS total
FROM loans
GROUP BY DATE_FORMAT(disbursed_on, '%Y-%m')
ORDER BY month;

-- 7. Top 3 customers by total loan amount
SELECT c.name, SUM(l.amount) AS total_amount
FROM customers c
JOIN loans l ON c.customer_id = l.customer_id
GROUP BY c.name
ORDER BY total_amount DESC
LIMIT 3;
Why each pattern matters: 1–2. Nth highest value — tests whether you understand LIMIT/OFFSET vs subquery approaches, and how to handle duplicate values correctly with DISTINCT. 3–4. Finding/removing duplicates — a very common real-world data-quality task, e.g., cleaning up accidental double-entry of the same loan. 5. Anti-join (LEFT JOIN + IS NULL) — used to find “missing” relationships, e.g., customers who never converted into loan holders (useful for sales/marketing targeting). 6. Time-based trend analysis — used constantly in dashboards and reports to track business growth month over month. 7. Top-N per business metric — used for identifying high-value customers, e.g., for relationship management or risk monitoring.
---
## 23. NULL Handling
Definition: NULL represents a missing or unknown value — it is not the same as zero or an empty string, and it doesn’t equal anything, not even another NULL (must use IS NULL / IS NOT NULL, never = NULL).
Why we use these functions: Real data is often incomplete — a loan might not have a status set yet, or a payment might have an unrecorded date. If NULLs aren’t handled explicitly, they silently break comparisons, calculations, and reports (e.g., SUM() skips NULLs, = NULL never matches). These tools let you detect and gracefully handle missing data instead of letting it cause silent bugs.
•	IS NULL / IS NOT NULL — the correct way to test for NULL values. Why: = and != never work with NULL due to SQL’s three-valued logic, so this is the only reliable way to check for missing data.
•	COALESCE() — returns the first non-NULL value from a list of expressions. Why: used to provide a sensible fallback/default when displaying or calculating with data that might be missing.
•	IFNULL() — MySQL-specific shorthand for a two-argument COALESCE. Why: a quick, readable option when you only need one fallback value, common in MySQL codebases.
SELECT * FROM loans WHERE status IS NULL;

SELECT COALESCE(status, 'Unknown') AS status FROM loans;

SELECT IFNULL(interest_rate, 0) FROM loans;   -- MySQL specific
---
## 24. Views
Definition: A view is a virtual table based on the result of a stored SELECT query. It doesn’t store data itself — it runs the underlying query each time it’s accessed — and is used to simplify complex queries, restrict access to specific columns/rows, or present data in a reusable format.
Why we use it: Complex, frequently-reused queries (e.g., a multi-join “active loans with customer details” query) are error-prone to rewrite every time. A view packages that logic once, under a simple name, so other queries/reports/applications can reuse it consistently. Views are also useful for security — e.g., exposing only non-sensitive columns to certain users without giving them access to the full underlying table.
CREATE VIEW active_loans AS
SELECT l.loan_id, c.name, l.amount, l.interest_rate
FROM loans l
JOIN customers c ON l.customer_id = c.customer_id
WHERE l.status = 'Active';

SELECT * FROM active_loans;  -- query it like a normal table
---
## 25. Temporary Tables
Definition: A temporary table is a table that exists only for the duration of a session (or transaction) and is automatically dropped when that session ends. It’s created with CREATE TEMPORARY TABLE and behaves like a normal table otherwise (can be inserted into, joined, indexed).
Why we use them: Some reports or multi-step processes need an intermediate “scratch space” to store partial results before producing a final output — e.g., first computing each customer’s total loan amount into a temp table, then joining that against other data for a final report. Doing this avoids repeating an expensive calculation multiple times in one query, and keeps the scratch data invisible to other sessions and automatically cleaned up (no manual cleanup needed).
CREATE TEMPORARY TABLE temp_customer_totals AS
SELECT customer_id, SUM(amount) AS total_amount
FROM loans
GROUP BY customer_id;

-- Use it like a normal table for the rest of the session
SELECT c.name, t.total_amount
FROM temp_customer_totals t
JOIN customers c ON c.customer_id = t.customer_id
WHERE t.total_amount > 1000000;

-- Automatically dropped when the session ends, or explicitly:
DROP TEMPORARY TABLE IF EXISTS temp_customer_totals;
Note: Temporary tables are session-specific — two users running the same script won’t see or interfere with each other’s temporary table, since each gets its own private copy.
---
## 26. Stored Procedures & Functions
Definition: - A stored procedure is a precompiled, reusable block of SQL statements stored in the database and executed with CALL. It can accept input parameters, perform multiple operations (including DML), and doesn’t have to return a value. - A function (user-defined function) is similar but must return a single value and can be used directly inside a SELECT statement, unlike a procedure.
Why we use them: Some logic (e.g., “fetch all loans for a customer” or “calculate late fee for a payment”) is used repeatedly by many parts of an application. Storing that logic once in the database means every caller gets the same consistent behavior, avoids duplicating SQL logic across the codebase, and can reduce back-and-forth network calls since the whole operation runs inside the database in one call.
### 26.1. Basic procedure (no parameters returned)
DELIMITER //
CREATE PROCEDURE GetCustomerLoans(IN cust_id INT)
BEGIN
    SELECT * FROM loans WHERE customer_id = cust_id;
END //
DELIMITER ;

CALL GetCustomerLoans(1);
### 26.2. Procedure with IN, OUT, and INOUT parameters
Definition: IN passes a value into the procedure (default), OUT returns a value back to the caller, and INOUT does both.
Why we use them: Real procedures often need to hand data back to the caller — e.g., returning the total outstanding amount for a loan after processing a payment, so the calling application can immediately act on it without a second query.
DELIMITER //
CREATE PROCEDURE GetLoanBalance(
    IN p_loan_id INT,
    OUT p_balance DECIMAL(12,2)
)
BEGIN
    DECLARE v_amount DECIMAL(12,2);
    DECLARE v_paid DECIMAL(12,2);

    SELECT amount INTO v_amount FROM loans WHERE loan_id = p_loan_id;
    SELECT COALESCE(SUM(amount_paid), 0) INTO v_paid
    FROM payments WHERE loan_id = p_loan_id;

    SET p_balance = v_amount - v_paid;
END //
DELIMITER ;

CALL GetLoanBalance(101, @balance);
SELECT @balance;
### 26.3. Procedure with conditional logic and a loop
Why we use this: Batch-style operations (e.g., applying a late fee to every overdue loan) are common in banking backends, and are far more efficient run once inside the database than row-by-row from application code.
DELIMITER //
CREATE PROCEDURE ApplyLateFeeToOverdueLoans()
BEGIN
    UPDATE loans
    SET amount = amount + 500
    WHERE status = 'Active'
      AND disbursed_on < CURDATE() - INTERVAL 90 DAY;
END //
DELIMITER ;

CALL ApplyLateFeeToOverdueLoans();
### 26.4. User-Defined Function
Why we use it: Unlike a procedure, a function can be dropped directly into a SELECT statement — useful for a reusable calculation you want available inline in reports, like computing a loan’s outstanding balance per row.
DELIMITER //
CREATE FUNCTION GetOutstandingBalance(p_loan_id INT) RETURNS DECIMAL(12,2)
DETERMINISTIC
BEGIN
    DECLARE v_balance DECIMAL(12,2);
    SELECT l.amount - COALESCE(SUM(p.amount_paid), 0) INTO v_balance
    FROM loans l
    LEFT JOIN payments p ON l.loan_id = p.loan_id
    WHERE l.loan_id = p_loan_id
    GROUP BY l.amount;
    RETURN v_balance;
END //
DELIMITER ;

-- Usable directly inside a SELECT, unlike a procedure
SELECT loan_id, GetOutstandingBalance(loan_id) AS balance FROM loans;
### 26.5. Dropping procedures/functions
DROP PROCEDURE IF EXISTS GetCustomerLoans;
DROP FUNCTION IF EXISTS GetOutstandingBalance;
### 26.6. Cursors & Exception Handling (inside procedures)
Definition: - A cursor lets a procedure process a query’s result row by row, instead of operating on the whole set at once. - A handler (DECLARE ... HANDLER) defines what a procedure should do when a specific error or condition occurs (e.g., “no more rows found,” or a constraint violation).
Why we use them: Most SQL operations are set-based (act on many rows at once), which is usually faster — but sometimes business logic genuinely needs to inspect and act on one row at a time (e.g., applying a different late-fee calculation per loan based on its individual history). Cursors enable this row-by-row processing inside a procedure. Exception handlers make procedures resilient — instead of a batch job crashing entirely on the first bad row, a handler can catch the issue, log it, and let the procedure continue or exit gracefully.
DELIMITER //
CREATE PROCEDURE ProcessOverdueLoans()
BEGIN
    DECLARE done INT DEFAULT FALSE;
    DECLARE v_loan_id INT;
    DECLARE v_amount DECIMAL(12,2);

    -- Cursor: selects overdue loans one at a time
    DECLARE overdue_cursor CURSOR FOR
        SELECT loan_id, amount FROM loans
        WHERE status = 'Active' AND disbursed_on < CURDATE() - INTERVAL 90 DAY;

    -- Handler: runs when the cursor has no more rows to fetch
    DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = TRUE;

    -- Handler: runs on any general SQL exception, so the procedure doesn't just crash
    DECLARE EXIT HANDLER FOR SQLEXCEPTION
    BEGIN
        ROLLBACK;
        SELECT 'An error occurred while processing overdue loans' AS error_message;
    END;

    START TRANSACTION;
    OPEN overdue_cursor;

    read_loop: LOOP
        FETCH overdue_cursor INTO v_loan_id, v_amount;
        IF done THEN
            LEAVE read_loop;
        END IF;

        UPDATE loans SET amount = v_amount + 500 WHERE loan_id = v_loan_id;
    END LOOP;

    CLOSE overdue_cursor;
    COMMIT;
END //
DELIMITER ;
Key points: - DECLARE ... CURSOR FOR — defines the query the cursor will iterate over. - OPEN / FETCH / CLOSE — start the cursor, pull one row at a time, and release it when done. - CONTINUE HANDLER — lets the procedure keep running after the condition (e.g., end of rows). - EXIT HANDLER — stops the procedure immediately after handling the condition (e.g., a real error).
---
## 27. Triggers
Definition: A trigger is a block of SQL code that automatically executes in response to a specific event (INSERT, UPDATE, or DELETE) on a table — either before or after that event occurs. Unlike a stored procedure, a trigger is never called directly; it fires automatically.
Why we use them: Some rules must be enforced no matter which application or user changes the data — even a manual query run directly against the database should respect them. Triggers guarantee this by attaching the logic to the table itself, rather than relying on every application to remember to run it. Common real-world uses: auto-maintaining an audit trail, keeping a summary/balance column in sync, enforcing business rules that constraints alone can’t express, and preventing invalid changes before they happen.
Trigger timing & event types: | Type | Fires | Typical use | |—|—|—| | BEFORE INSERT | Before a new row is added | Validate or auto-fill values before they’re saved (e.g., default status) | | AFTER INSERT | After a new row is added | Log the new row into an audit table, update a related summary table | | BEFORE UPDATE | Before a row is modified | Block or adjust an update that violates a business rule | | AFTER UPDATE | After a row is modified | Record what changed into a history/audit table | | BEFORE DELETE | Before a row is removed | Prevent deletion under certain conditions (e.g., can’t delete a loan with pending payments) | | AFTER DELETE | After a row is removed | Log the deletion for audit purposes |
### 27.1. AFTER INSERT — audit logging
Why: Banking systems typically require an audit trail of every change for compliance — a trigger guarantees this happens automatically, every time, without relying on application code to remember to log it.
CREATE TABLE loan_audit (
    audit_id     INT AUTO_INCREMENT PRIMARY KEY,
    loan_id      INT,
    action       VARCHAR(20),
    action_time  DATETIME
);

DELIMITER //
CREATE TRIGGER trg_after_loan_insert
AFTER INSERT ON loans
FOR EACH ROW
BEGIN
    INSERT INTO loan_audit (loan_id, action, action_time)
    VALUES (NEW.loan_id, 'INSERTED', NOW());
END //
DELIMITER ;
### 27.2. BEFORE UPDATE — enforcing a business rule
Why: Stops invalid data changes at the database level, even if the application layer has a bug that would otherwise allow it — e.g., preventing a loan amount from ever being set to zero or negative.
DELIMITER //
CREATE TRIGGER trg_before_loan_update
BEFORE UPDATE ON loans
FOR EACH ROW
BEGIN
    IF NEW.amount <= 0 THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Loan amount must be greater than zero';
    END IF;
END //
DELIMITER ;
### 27.3. AFTER UPDATE — tracking status changes
Why: Useful for a full history of how a loan’s status changed over time (e.g., Active → Closed → Defaulted), which is valuable for both compliance and troubleshooting disputes.
CREATE TABLE loan_status_history (
    history_id    INT AUTO_INCREMENT PRIMARY KEY,
    loan_id       INT,
    old_status    VARCHAR(20),
    new_status    VARCHAR(20),
    changed_at    DATETIME
);

DELIMITER //
CREATE TRIGGER trg_after_loan_status_update
AFTER UPDATE ON loans
FOR EACH ROW
BEGIN
    IF OLD.status <> NEW.status THEN
        INSERT INTO loan_status_history (loan_id, old_status, new_status, changed_at)
        VALUES (OLD.loan_id, OLD.status, NEW.status, NOW());
    END IF;
END //
DELIMITER ;
### 27.4. BEFORE DELETE — protecting related data
Why: Prevents accidental data-integrity issues — e.g., stops someone from deleting a loan that still has payment records linked to it, which would otherwise leave orphaned rows in the payments table.
DELIMITER //
CREATE TRIGGER trg_before_loan_delete
BEFORE DELETE ON loans
FOR EACH ROW
BEGIN
    IF (SELECT COUNT(*) FROM payments WHERE loan_id = OLD.loan_id) > 0 THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Cannot delete a loan with existing payment records';
    END IF;
END //
DELIMITER ;
Key points on NEW and OLD: - NEW — refers to the row’s values after the change (available in INSERT and UPDATE triggers) - OLD — refers to the row’s values before the change (available in UPDATE and DELETE triggers)
### 27.5. Dropping a trigger
DROP TRIGGER IF EXISTS trg_after_loan_insert;
Caution (worth knowing for interviews): Triggers run silently and automatically, which is powerful but can make debugging harder — a change to loans might trigger a cascade of side effects that aren’t obvious from the query that caused them. They’re best used sparingly, for rules that genuinely must be enforced at the database level (audit trails, hard business constraints) rather than general application logic.
---
## 28. Keys & Constraints
Definition: Keys identify rows uniquely and define relationships between tables. Constraints enforce rules on the data allowed in a column.
Why we use them: Without keys, there’s no reliable way to identify “this exact row” or link related data across tables — you’d risk duplicate/ambiguous records and broken relationships (e.g., a loan pointing to a customer_id that doesn’t exist). Constraints go a step further and stop bad data from ever entering the database in the first place, rather than relying on application code to catch every mistake.
Term	Definition	Why it matters
Primary Key	Uniquely identifies each row in a table; cannot be NULL, must be unique	Guarantees every row can be reliably referenced/updated/deleted without ambiguity
Foreign Key	A column that references the Primary Key of another table, enforcing referential integrity	Prevents “orphan” records — e.g., stops a loan from being created for a customer_id that doesn’t exist
Composite Key	A primary key made up of two or more columns together	Needed when no single column is unique on its own, but a combination of columns is (e.g., student_id + course_id in an enrollment table)
Candidate Key	Any column (or set of columns) that could qualify as the primary key	Helps during design to identify all valid uniqueness options before picking the best one as Primary Key
Unique Key	Ensures all values in a column are distinct, but (unlike Primary Key) allows one NULL	Used for columns that must be unique but aren’t the main identifier — e.g., a customer’s email or PAN number
NOT NULL	Disallows NULL/missing values in a column	Used for fields the business logic can’t function without — e.g., a loan must have an amount
CHECK	Restricts values in a column to satisfy a specific condition	Enforces business rules at the database level — e.g., loan amount must always be positive
DEFAULT	Assigns a default value to a column when none is provided on insert	Reduces repetitive application code and ensures a sensible fallback — e.g., new loans default to ‘Active’ status
CREATE TABLE loans (
    loan_id       INT PRIMARY KEY,
    customer_id   INT NOT NULL,
    amount        DECIMAL(12,2) CHECK (amount > 0),
    status        VARCHAR(20) DEFAULT 'Active',
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
);
### 28.1. Foreign Key Actions (ON DELETE / ON UPDATE)
Definition: These clauses define what should automatically happen to child rows (e.g., rows in loans) when the parent row they reference (e.g., a row in customers) is deleted or its key is updated.
Why we use them: Without an explicit action, deleting a customer who still has loans would either fail (blocked by the foreign key) or, worse, leave orphaned loan records pointing to a customer that no longer exists. These clauses let you decide the correct, intentional behavior for your business rules up front, rather than leaving it to chance or requiring extra application code to handle it manually.
Action	What it does	Example use case
CASCADE	Automatically deletes/updates child rows when the parent is deleted/updated	Deleting a loan automatically deletes its related payment records
SET NULL	Sets the foreign key column in child rows to NULL	A branch is deleted, but existing loans just lose their branch reference rather than being deleted
RESTRICT / NO ACTION	Blocks the delete/update if child rows still exist	Prevent deleting a customer who still has active loans, forcing a deliberate decision first
SET DEFAULT	Sets the foreign key column to its default value	Less commonly used; resets the reference to a predefined fallback value
CREATE TABLE payments (
    payment_id    INT PRIMARY KEY,
    loan_id       INT,
    amount_paid   DECIMAL(12,2),
    payment_date  DATE,
    FOREIGN KEY (loan_id) REFERENCES loans(loan_id)
        ON DELETE CASCADE   -- deleting a loan also deletes its payments
        ON UPDATE CASCADE   -- if loan_id ever changes, payments follow along
);
In practice: Financial systems often prefer RESTRICT for records like customers or loans (you generally want deletion blocked until related data is deliberately handled), while CASCADE is more common for tightly-owned child data like payment line items that have no meaning without their parent loan.
---
## 29. Data Types (common ones)
Definition: A data type defines what kind of value a column can hold and how it’s stored.
Why we use them: Choosing the right data type isn’t just formality — it directly affects data accuracy, storage size, and performance. Using the wrong type causes real bugs: e.g., storing money as FLOAT can introduce rounding errors in interest calculations, which is unacceptable in banking systems — DECIMAL is used specifically to avoid that.
Type	Meaning	Why it’s used
INT	Whole numbers	For counts, IDs, and any value with no fractional part
DECIMAL(p,s)	Exact fixed-point number (p = total digits, s = digits after decimal) — used for money	Guarantees exact precision for financial values, avoiding floating-point rounding errors
VARCHAR(n)	Variable-length text, up to n characters	Efficient storage for text that varies in length, like names or addresses
CHAR(n)	Fixed-length text, padded with spaces	Used for values that are always the same length, e.g., a fixed-length code, for slightly faster fixed-size lookups
DATE / DATETIME / TIMESTAMP	Date-only / date+time / date+time with timezone tracking	Lets the database validate and sort dates correctly, and support date arithmetic (e.g., loan tenure calculations)
BOOLEAN	True/false value (often stored as TINYINT(1) in MySQL)	Clearly models a yes/no flag, e.g., is_active, more explicitly than using 0/1 in an INT column
TEXT	Large variable-length text block	Used for long free-form content, like notes or descriptions, that wouldn’t fit well in a VARCHAR
---
## 30. Auto-Increment / Sequences
Definition: An auto-increment column (MySQL: AUTO_INCREMENT; PostgreSQL: SERIAL/sequences; Oracle/SQL Server: SEQUENCE/IDENTITY) automatically generates a unique, incrementing numeric value for each new row, without the application having to supply one.
Why we use them: Primary keys need to be unique, but coming up with a guaranteed-unique value manually (especially with multiple application instances inserting data at once) is error-prone and adds unnecessary complexity. Auto-increment offloads this to the database, which can generate unique IDs safely even under concurrent inserts — this is why most surrogate (artificial) primary keys, like loan_id or customer_id, are defined this way rather than being manually assigned.
CREATE TABLE branches (
    branch_id   INT AUTO_INCREMENT PRIMARY KEY,
    branch_name VARCHAR(100)
);

INSERT INTO branches (branch_name) VALUES ('Bangalore Main');
-- branch_id is generated automatically — no need to specify it

SELECT LAST_INSERT_ID();  -- retrieve the ID just generated, e.g., to use in a related insert
PostgreSQL equivalent (sequence-based):
CREATE TABLE branches (
    branch_id   SERIAL PRIMARY KEY,
    branch_name VARCHAR(100)
);
Note: Auto-increment values are not guaranteed to be perfectly sequential with no gaps (a rolled-back transaction can “use up” a number), so they should be treated purely as unique identifiers — never relied on for counting rows or inferring business meaning like order of importance.
---
## 31. JSON Functions
Definition: Modern relational databases (MySQL 5.7+, PostgreSQL) support a native JSON data type and functions to store, query, and update semi-structured JSON data directly inside a normal table column.
Why we use them: Some data doesn’t fit neatly into a fixed set of columns — e.g., a flexible “additional_details” field for a loan that might have different attributes depending on loan type (collateral details for a home loan, employer details for a personal loan). Rather than creating many nullable columns or a separate table for every variation, JSON lets you store this flexible data directly, while still being able to query specific fields when needed — combining the flexibility of a document store with the reliability of a relational database.
CREATE TABLE loan_details (
    loan_id   INT PRIMARY KEY,
    metadata  JSON
);

INSERT INTO loan_details (loan_id, metadata)
VALUES (101, '{"collateral": "Property", "value": 5000000, "co_applicant": "Anita Rao"}');

-- Extract a specific field from the JSON column
SELECT loan_id, metadata->>'$.collateral' AS collateral_type
FROM loan_details;

-- Filter rows based on a JSON field's value
SELECT * FROM loan_details
WHERE metadata->>'$.collateral' = 'Property';

-- Update a specific field within the JSON without rewriting the whole value
UPDATE loan_details
SET metadata = JSON_SET(metadata, '$.value', 5500000)
WHERE loan_id = 101;
When to use JSON vs. normal columns: JSON is great for genuinely flexible/variable attributes, but if a field is queried or filtered on constantly, it’s usually better performance-wise as a real column (with an index) rather than buried inside JSON.
---
## 32. Entity-Relationship (ER) Concepts
Definition: These are database-design concepts used before writing SQL, to model how data relates.
Why we use them: Before creating tables, it helps to first think in terms of real-world objects and how they connect — this avoids designing a schema that’s hard to query or prone to redundancy later. Getting the relationship type right (one-to-many vs many-to-many) determines whether you need a foreign key on one table or a whole separate junction table.
•	Entity — a real-world object represented as a table (e.g., Customer, Loan). Why: gives a clear, intuitive mapping between the business domain and the database structure.
•	Attribute — a property of an entity, represented as a column (e.g., name, amount). Why: breaks an entity down into the specific facts that need to be stored about it.
•	Relationship — how entities are connected:
–	One-to-One — one row in Table A relates to exactly one row in Table B. Why: used to split optional or sensitive data into a separate table, e.g., a customer and their KYC document details.
–	One-to-Many — one row in Table A relates to many rows in Table B (e.g., one customer → many loans). Why: the most common real-world relationship — avoids repeating customer info on every loan row.
–	Many-to-Many — rows in Table A relate to many rows in Table B and vice versa (needs a junction/bridge table). Why: needed when neither side can be reduced to “one owns many” — e.g., many customers can hold many types of products, and each product can have many customers.
---
## 33. Quick-Reference Cheat Sheet
Task	Query Pattern
Filter rows	WHERE column = value
Remove duplicates	SELECT DISTINCT
Combine tables	JOIN ... ON
Aggregate + filter groups	GROUP BY ... HAVING
Top N rows	ORDER BY ... LIMIT N
Conditional column	CASE WHEN ... THEN ... END
Quick two-branch condition	IF(condition, val1, val2)
Turn a value into NULL	NULLIF(expr1, expr2)
Fallback for NULL	COALESCE(col, 'default')
Running totals / ranking	Window functions (OVER, PARTITION BY)
Named reusable sub-query	WITH name AS (...) SELECT ... (CTE)
Hierarchical/tree data	WITH RECURSIVE name AS (...)
Subtotal + grand total	GROUP BY col WITH ROLLUP
Combine text	CONCAT(a, b)
Days between dates	DATEDIFF(date1, date2)
Add/subtract time	DATE_ADD(date, INTERVAL n unit)
Round a number	ROUND(n, decimals)
Safe multi-step changes	START TRANSACTION ... COMMIT/ROLLBACK
Control strictness of concurrency	SET SESSION TRANSACTION ISOLATION LEVEL ...
Scratch space for a report	CREATE TEMPORARY TABLE ...
Reusable multi-step logic	CREATE PROCEDURE ... CALL procedure_name()
Reusable inline calculation	CREATE FUNCTION ... RETURNS ...
Row-by-row processing	DECLARE CURSOR ... OPEN/FETCH/CLOSE
Catch errors in a procedure	DECLARE EXIT HANDLER FOR SQLEXCEPTION
Auto-run logic on data change	CREATE TRIGGER ... BEFORE/AFTER INSERT/UPDATE/DELETE
Auto-generated unique ID	AUTO_INCREMENT / SERIAL
Control access/permissions	GRANT ... TO user / REVOKE ... FROM user
Cascade delete to child rows	FOREIGN KEY ... ON DELETE CASCADE
Store flexible/variable data	JSON column + JSON_EXTRACT/->>/JSON_SET
---
Tips for practicing
•	Recreate the schema above in MySQL Workbench / a local SQLite/PostgreSQL instance and run every query yourself.
•	For interviews, be ready to explain why you’d use a JOIN vs subquery vs CTE vs window function for the same problem — that reasoning matters more than syntax recall.
•	Practice writing the “Nth highest value” and “find duplicates” patterns from memory — they come up constantly.
•	Be ready to explain SQL’s logical execution order (FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT) — it’s a favorite conceptual interview question because it explains so many “why doesn’t this work” moments.
•	Know the difference between a stored procedure and a trigger by when they run: a procedure runs when explicitly called; a trigger runs automatically in response to a data change.
