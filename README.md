# SQL Interview Guide - Concise Version

## 1. What is SQL?

**SQL** = Structured Query Language. It's used to talk to databases - to get data, add data, change data, and delete data.

### Main Parts of SQL:
- **DDL** - Create/modify tables (CREATE, ALTER, DROP)
- **DML** - Add/change/delete data (INSERT, UPDATE, DELETE)
- **DQL** - Get data from database (SELECT)
- **DCL** - Control who can access what (GRANT, REVOKE)

### What You Do with SQL:
- Retrieve data
- Add new records
- Update existing records
- Delete records
- Create tables
- Set data rules and constraints

---

## 2. SQL vs NoSQL

| Feature | SQL | NoSQL |
|---------|-----|-------|
| **Data Type** | Structured data (tables) | Unstructured data (flexible) |
| **Schema** | Fixed schema | No fixed schema |
| **Query Language** | SQL | Different APIs (varies) |
| **Consistency** | Strong (ACID) | Eventual consistency |
| **Scaling** | Vertical (bigger servers) | Horizontal (more servers) |
| **Examples** | MySQL, PostgreSQL | MongoDB, Redis, Cassandra |

### Strong Consistency vs Eventual Consistency

#### Strong Consistency (SQL - ACID)
Data is **immediately consistent** across all servers. When you write something, everyone sees the same thing right away.

**Example**: Bank transfer - You withdraw $100, and instantly all ATMs show your new balance. No waiting.

```sql
-- You update once, everyone sees it immediately
UPDATE Account SET Balance = Balance - 100 WHERE ID = 1;
-- Everyone queries this account sees the new balance right now
SELECT Balance FROM Account WHERE ID = 1;  -- All users see $900
```

✅ Best for: Banking, finance, critical data

#### Eventual Consistency (NoSQL - BASE)
Data is **copied to multiple servers in the background**. There's a small delay before everyone sees the update, but it "eventually" catches up.

**Example**: Social media - You like a post. Your friend might see 100 likes while you see 101, but after a few seconds everyone sees 101.

```
Time 0: You click Like
Server 1: Updates like count to 101 ✓
Server 2: Still shows 100 (hasn't updated yet)
Server 3: Still shows 100 (hasn't updated yet)

Time 2 seconds later:
Server 1: 101
Server 2: 101
Server 3: 101
-- Now all servers are "eventually consistent"
```

⚡ Best for: Social media, real-time feeds, high-traffic apps

### Why This Trade-Off?

**SQL (Strong Consistency)**:
- Slower writes (waits for all servers to agree)
- More reliable (no data conflicts)
- Doesn't scale horizontally as well

**NoSQL (Eventual Consistency)**:
- Super fast writes (doesn't wait)
- Better for scaling globally
- Brief period where data might differ across locations

### When to Use:
- **SQL**: Banking, business apps, data with relationships, where accuracy is critical
- **NoSQL**: Big data, real-time apps, social media, where speed matters more than instant accuracy

### Modern Exception:
Some modern databases blur the lines:
- **MongoDB** - NoSQL with optional ACID transactions
- **Google Spanner** - SQL-like but scales horizontally with strong consistency
- **Cassandra** - NoSQL but you can tune consistency per request

---

## 3. Main SQL Commands

### Data Retrieval (DQL)
```sql
SELECT * FROM table_name;
SELECT column1, column2 FROM table WHERE condition;
```

### Create (DDL)
```sql
CREATE TABLE Employee (
    ID INT PRIMARY KEY,
    Name VARCHAR(50),
    Age INT
);
```

### Insert (DML)
```sql
INSERT INTO Employee VALUES (1, 'John', 25);
```

### Update (DML)
```sql
UPDATE Employee SET Age = 26 WHERE ID = 1;
```

### Delete (DML)
```sql
DELETE FROM Employee WHERE ID = 1;
```

### Drop Table (DDL)
```sql
DROP TABLE Employee;
```

---

## 4. SELECT Statement

The **SELECT** statement gets data from a table.

### Basic Syntax:
```sql
SELECT column_name FROM table_name WHERE condition;
```

### Example:
```sql
SELECT Name, Age 
FROM Employee 
WHERE Age > 25 
ORDER BY Age;
```

### Common Clauses:
- **WHERE** - Filter rows
- **ORDER BY** - Sort results
- **LIMIT** - Show only X rows
- **GROUP BY** - Group similar data
- **JOIN** - Combine data from multiple tables

---

## 5. WHERE vs HAVING

| WHERE | HAVING |
|-------|--------|
| Filters **before** grouping | Filters **after** grouping |
| Works with individual rows | Works with grouped data |
| Used with any column | Used with aggregate functions |
| Cannot use COUNT, SUM, AVG | Can use COUNT, SUM, AVG |

### Example:
```sql
-- WHERE - filter before grouping
SELECT Department, COUNT(*) as Count
FROM Employee
WHERE Age > 25          -- WHERE filters rows first
GROUP BY Department
HAVING COUNT(*) > 3;    -- HAVING filters groups second
```

---

## 6. JOINs Explained

### INNER JOIN
Shows only matching records from both tables.
```sql
SELECT * FROM Employee 
INNER JOIN Department ON Employee.DeptID = Department.ID;
```

### LEFT JOIN
Shows all records from left table + matching records from right table.
```sql
SELECT * FROM Employee 
LEFT JOIN Department ON Employee.DeptID = Department.ID;
```

### RIGHT JOIN
Shows all records from right table + matching records from left table.

### FULL OUTER JOIN
Shows all records from both tables.

---

## 7. Aggregate Functions

Used to calculate something on a group of rows.

```sql
-- COUNT - how many rows
SELECT COUNT(*) FROM Employee;

-- SUM - total of a column
SELECT SUM(Salary) FROM Employee;

-- AVG - average of a column
SELECT AVG(Salary) FROM Employee;

-- MIN/MAX - lowest/highest value
SELECT MIN(Salary), MAX(Salary) FROM Employee;
```

---

## 8. GROUP BY

Groups rows that have the same values. Used with aggregate functions.

```sql
-- Count employees per department
SELECT Department, COUNT(*) as Count
FROM Employee
GROUP BY Department;

-- Total salary per department
SELECT Department, SUM(Salary) as Total
FROM Employee
GROUP BY Department;
```

---

## 9. Keys in SQL

### Primary Key
- Unique identifier for each row
- Cannot be NULL
- Only one per table

```sql
CREATE TABLE Employee (
    ID INT PRIMARY KEY,
    Name VARCHAR(50)
);
```

### Foreign Key
- Links to another table's primary key
- Creates relationships between tables

```sql
CREATE TABLE Employee (
    ID INT PRIMARY KEY,
    DeptID INT,
    FOREIGN KEY (DeptID) REFERENCES Department(ID)
);
```

---

## 10. INDEX

Speeds up search on a column. Like a book's index - faster to find information.

```sql
-- Create index
CREATE INDEX idx_name ON Employee(Name);

-- Drop index
DROP INDEX idx_name;
```

**Trade-off**: Faster searches but slower inserts/updates.

---

## 11. Common Interview Questions

### Q: What is ACID?
**A**: Rules that ensure database transactions are reliable and safe. It's about making sure data doesn't get corrupted when multiple operations happen.

#### Atomicity (All or Nothing)
Means a transaction is "atomic" - it either completes fully or doesn't happen at all. There's no in-between state.

**Example**: When you transfer $100 from Account A to Account B:
- Step 1: Deduct $100 from Account A
- Step 2: Add $100 to Account B

Both steps must succeed. If Step 2 fails after Step 1 completes, the database rolls back and undoes Step 1. You won't lose money stuck in limbo.

```sql
BEGIN TRANSACTION;
UPDATE Account SET Balance = Balance - 100 WHERE ID = 1;
UPDATE Account SET Balance = Balance + 100 WHERE ID = 2;
COMMIT;
-- If any error occurs, ROLLBACK automatically
```

#### Consistency (Data Stays Valid)
The database moves from one valid state to another valid state. All rules and constraints are enforced.

**Example**: Say you have a rule: "Account balance cannot be negative"
- If you try to withdraw $500 but only have $100, the transaction fails
- The database rejects it because it would break the consistency rule
- Data remains in a valid state

```sql
-- Constraint ensures data consistency
CREATE TABLE Account (
    ID INT PRIMARY KEY,
    Balance DECIMAL(10, 2) NOT NULL CHECK (Balance >= 0)
);

-- This update will fail - violates constraint
UPDATE Account SET Balance = -50 WHERE ID = 1;
-- Database rejects it automatically
```

#### Isolation (Transactions Don't Interfere)
When multiple transactions run at the same time, they don't mess with each other. Each transaction works as if it's alone.

**Example**: Two people withdraw from the same account at the same time:
- Without Isolation: Both might read Balance = $1000, both withdraw $500, result = $500 (WRONG, should be $0)
- With Isolation: One transaction completes first (Balance = $500), then the other sees the updated balance and fails (can't withdraw $500 from $500)

```
Transaction 1:              Transaction 2:
Read Balance ($1000)        Read Balance ($1000)
Deduct $500 → $500         Deduct $500 → $500
Write $500                  Write $500 ❌ WRONG!
                            Should be $0

With Isolation:
Transaction 1 completes first
Transaction 2 sees Balance = $500, safely processes
```

Different isolation levels control how much transactions can see:
- **Dirty Read**: Can see uncommitted changes (risky)
- **Read Committed**: Only see committed changes (safe)
- **Repeatable Read**: Consistent reads within same transaction
- **Serializable**: Complete isolation (slowest, safest)

#### Durability (Data is Permanently Saved)
Once a transaction is committed, it stays committed even if there's a power failure, crash, or disaster. The data is safely written to disk.

**Example**: You buy something online and get an order confirmation. Even if the server crashes right after, your order exists in the database because it was committed to disk.

```sql
BEGIN TRANSACTION;
INSERT INTO Orders VALUES (123, 'Laptop', '2024-03-08');
INSERT INTO OrderDetails VALUES (456, 123, 1500);
COMMIT;
-- Data is now on disk. Even if power goes off, data survives
```

Without Durability: Data might only be in memory, and a crash = lost data.

#### Real-World Example: Bank Transfer

```sql
-- Transfer $500 from Account A to Account B
BEGIN TRANSACTION;

-- Step 1: Deduct from Account A
UPDATE accounts SET balance = balance - 500 WHERE account_id = 'A';

-- Step 2: Add to Account B  
UPDATE accounts SET balance = balance + 500 WHERE account_id = 'B';

-- Commit everything
COMMIT;

-- ACID Guarantees:
-- ✓ Atomicity: Both updates happen or neither happens
-- ✓ Consistency: Balances stay valid (no negative, no lost money)
-- ✓ Isolation: Other customers' transactions don't interfere
-- ✓ Durability: Once committed, money transfer is permanent
```

**Why ACID Matters**: Without ACID, you could lose money, corrupt data, or have inconsistent information. Banks, hospitals, and financial systems depend on ACID for safety.

### Q: What is Normalization?
**A**: Organizing data to reduce duplicates and keep data consistent.
- **1NF**: Remove repeating groups
- **2NF**: Remove partial dependencies
- **3NF**: Remove transitive dependencies

### Q: What is a VIEW?
**A**: A virtual table based on a SELECT query. Looks like a table but doesn't store data.

```sql
CREATE VIEW HighEarners AS
SELECT * FROM Employee WHERE Salary > 50000;
```

### Q: What is a TRIGGER?
**A**: Code that runs automatically when something happens (INSERT, UPDATE, DELETE).

```sql
CREATE TRIGGER update_timestamp
AFTER UPDATE ON Employee
FOR EACH ROW
BEGIN
    UPDATE Employee SET LastModified = NOW();
END;
```

---

## Quick Tips for Interview

✅ **Know basic commands**: SELECT, INSERT, UPDATE, DELETE, CREATE, DROP
✅ **Understand JOINs**: Most asked topic
✅ **Know the difference**: WHERE vs HAVING, SQL vs NoSQL
✅ **Practice queries**: Write simple and complex queries
✅ **Explain clearly**: Use simple words, give examples
✅ **Ask if unsure**: Better to ask than guess wrong
