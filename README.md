# SQL Interview Notes - Quick Reference

## 1. What is SQL?
**SQL** = Structured Query Language. Used to manage databases - retrieve, insert, update, delete data.

### Main Parts:
- **DDL** - Create/modify tables (CREATE, ALTER, DROP)
- **DML** - Add/change/delete data (INSERT, UPDATE, DELETE)
- **DQL** - Get data (SELECT)
- **DCL** - Control access (GRANT, REVOKE)

---

## 2. SQL vs NoSQL

| Feature | SQL | NoSQL |
|---------|-----|-------|
| Data Type | Structured (tables) | Unstructured (flexible) |
| Schema | Fixed | Flexible |
| Consistency | Strong (ACID) | Eventual |
| Scaling | Vertical | Horizontal |
| Examples | MySQL, PostgreSQL | MongoDB, Redis, Cassandra |

### Strong vs Eventual Consistency
- **Strong (SQL)**: Everyone sees update immediately (like bank ATM)
- **Eventual (NoSQL)**: Update spreads slowly across servers (like social media likes)

---

## 3. Basic SQL Commands

```sql
CREATE TABLE name (id INT PRIMARY KEY);  -- Create table
INSERT INTO table VALUES (...);          -- Add data
SELECT * FROM table WHERE condition;     -- Get data
UPDATE table SET column = value;         -- Modify data
DELETE FROM table WHERE id = 1;          -- Remove data
DROP TABLE name;                         -- Delete table
ALTER TABLE name ADD column;             -- Modify table structure
```

---

## 4. SELECT Statement
Gets data from database. Key clauses:
- **WHERE** - Filter rows
- **ORDER BY** - Sort results
- **LIMIT** - Show only X rows
- **GROUP BY** - Group similar data
- **HAVING** - Filter after grouping
- **JOIN** - Combine tables

---

## 5. WHERE vs HAVING
| WHERE | HAVING |
|-------|--------|
| Filters BEFORE grouping | Filters AFTER grouping |
| Works on individual rows | Works on grouped data |
| Can't use COUNT, SUM, AVG | Can use COUNT, SUM, AVG |

---

## 6. JOINs (Combining Tables)
- **INNER JOIN** - Only matching records from both tables
- **LEFT JOIN** - All from left table + matching from right
- **RIGHT JOIN** - All from right table + matching from left
- **FULL OUTER JOIN** - All records from both tables

---

## 7. Aggregate Functions
Count/calculate across groups:
- **COUNT(*)** - How many rows
- **SUM(column)** - Total
- **AVG(column)** - Average
- **MIN/MAX(column)** - Lowest/highest value

---

## 8. GROUP BY
Groups rows with same values. Used with aggregate functions.
```
Example: COUNT employees per department
```

---

## 9. Keys in SQL
- **Primary Key** - Unique ID for each row (cannot be NULL, only 1 per table)
- **Foreign Key** - Links to another table's primary key (creates relationships)

---

## 10. INDEX
Makes searches faster (like book's index). Trade-off: faster reads but slower writes.

---

## 11. ACID (Database Safety Rules)

**A - Atomicity**: Transaction is all-or-nothing. Both steps succeed or both fail.
- Example: Bank transfer - deduct AND add must both happen, not just one.

**C - Consistency**: Data stays valid. All rules/constraints are enforced.
- Example: Balance can't be negative.

**I - Isolation**: Transactions don't interfere with each other.
- Example: Two people withdrawing from same account at same time don't cause problems.

**D - Durability**: Once saved, data stays saved even if power fails.
- Example: Your order confirmation saved to disk permanently.

---

## 12. SQL Injection & Prevention

**What is it?** Hacker inserts SQL code into input to manipulate database.

**Example:** Username field: `admin' OR '1'='1` bypasses login.

### How to Prevent (3 Ways):

**1. Parameterized Queries** (BEST - Use this!)
- Separate SQL code from user data using placeholders (?)
- Database treats user input as data, not code
- Even if hacker enters malicious text, it's just a string

**2. Input Validation**
- Check if input matches expected format (length, characters)
- Extra security layer on top of parameterized queries
- Don't rely on this alone

**3. Use ORM**
- ORM automatically uses parameterized queries
- You don't write SQL manually, so no injection risk
- (See section 13)

---

## 13. Normalization

**What is it?** Organizing database into separate tables to reduce data redundancy and improve integrity.

### Why Normalize?
- Prevents data duplication (save storage)
- Reduces update anomalies (fix data once, not multiple places)
- Improves data integrity
- Makes queries more efficient

### The Normal Forms:

**0NF (Unnormalized)** - All data in one messy table
```
ID | Name  | Invoice No | Item1 | Item2 | Item3
1  | John  | INV001     | A,B,C | Q1,Q2,Q3 | P1,P2,P3
(Multiple values in single cell - BAD!)
```

**1NF (First Normal Form)** - Atomic values (no repeating groups)
```
Customers Table:
ID | Name
1  | John

Invoices Table:
InvoiceNo | CustomerID | Date
INV001    | 1          | 2024-01-01

Items Table:
InvoiceNo | ItemNo | Description | Qty
INV001    | 1      | Product A   | 5
INV001    | 2      | Product B   | 3
(Each cell has single value)
```

**2NF (Second Normal Form)** - 1NF + All non-key columns depend on ENTIRE primary key
```
In Items Table:
PRIMARY KEY = (InvoiceNo, ItemNo)
Both Description and Quantity depend on both parts of key ✓
```

**3NF (Third Normal Form)** - 2NF + No transitive dependencies (non-key columns depend ONLY on primary key)
```
Invoices Table:
InvoiceNo | CustomerID | InvoiceDate
(No other column depends on CustomerID directly) ✓

Bad example (needs to be split):
InvoiceNo | CustomerID | CustomerName | CustomerCity
(CustomerName depends on CustomerID, not InvoiceNo - needs separate table)
```

### Real Usage:
- Most production databases target **3NF** (good balance)
- Higher forms (BCNF, 4NF) for complex scenarios needing extreme integrity
- Always start with 3NF, denormalize only if needed for performance

---

## 14. Denormalization

**What is it?** Intentionally breaking normalization rules to improve performance. Trade-off: Speed vs. Data Consistency.

### When to Use:
- **Reporting/Analytics** - Complex reports spanning many tables (flatten to single table for faster queries)
- **High-traffic sites** - E-commerce checkout (accept slight delay in sales figures update)
- **Read-heavy apps** - More reads than writes (like search engines, news sites)
- **Distributed systems** - NoSQL databases store redundant data across servers

### How It Works:

**Normalized (slow for reporting):**
```
Need 5 JOINs to get order details with customer + product + warehouse info
SELECT ... FROM Orders 
JOIN Customers ... JOIN Products ... JOIN Warehouses ... JOIN Pricing ...
(5 joins = slower query)
```

**Denormalized (fast for reporting):**
```
Orders_Denormalized Table:
OrderID | CustomerName | CustomerCity | ProductName | ProductCategory | 
WarehouseName | Price | InventoryCount

(All in one table = no joins = fast query)
```

### Trade-offs:
- ✅ Faster reads, simpler queries
- ❌ Slower writes (must update in multiple places)
- ❌ More storage space (duplicate data)
- ❌ Risk of inconsistency (data out of sync)

### When NOT to Use:
- Systems requiring strong data consistency (banking)
- Write-heavy applications
- When storage space is critical

---

## 15. Indexes

**What is it?** Pointer/shortcut to find data faster. Like a book's index instead of reading every page.

### How They Work:
```
Without index - scan 1 million rows:
SELECT * FROM users WHERE email = 'john@example.com'
(Checks EVERY row = 1 million lookups)

With index - jump directly:
(Database uses B-Tree to jump to "john@..." = ~20 lookups)
```

### Index Types:

| Index | Use Case |
|-------|----------|
| **B-Tree** | Most common. Best for range queries (WHERE age > 25) |
| **Hash** | Fast for exact matches (WHERE id = 5). Not good for ranges |
| **Bitmap** | Columns with few unique values (gender: M/F) |
| **R-Tree** | Geographic data (maps, GPS coordinates) |

### When to Index:

✅ **DO index:**
- Primary keys (automatic)
- Foreign keys (for JOINs)
- Columns in WHERE clauses frequently queried
- Columns used in ORDER BY or GROUP BY
- High cardinality (many unique values)

❌ **DON'T over-index:**
- Too many indexes slow down INSERT/UPDATE/DELETE (must update all indexes)
- Low cardinality columns (gender, status - only few values)
- Small tables (whole table fits in memory)

### Code Example:
```sql
-- Create index
CREATE INDEX idx_user_email ON users(email);

-- Create composite index (multiple columns)
CREATE INDEX idx_user_city_age ON users(city, age);

-- View indexes on a table
SHOW INDEXES FROM users;

-- Drop index if not needed
DROP INDEX idx_user_email ON users;

-- Index usage in query
EXPLAIN SELECT * FROM users WHERE email = 'john@example.com';
(Shows if index was used)
```

### Best Practices:
- Monitor index performance regularly
- Remove unused indexes (save space and write performance)
- Keep index count low (3-5 per table usually enough)
- Test queries with EXPLAIN to verify index usage

---

## 16. ORM (Object-Relational Mapping)

**What is it?** Write database code using objects instead of raw SQL. ORM automatically converts objects to SQL.

### Why Use ORM?
- **Security** - Prevents SQL injection automatically (parameterized queries built-in)
- **Less code** - 1 line vs 10 lines of SQL
- **Database agnostic** - Switch from MySQL to PostgreSQL without code changes
- **Type-safe** - Catch errors before runtime
- **Faster development** - Focus on logic, not SQL

### How It Works:

```javascript
// Without ORM (Raw SQL)
const query = "SELECT * FROM users WHERE id = ? AND status = ?";
connection.query(query, [userId, 'active'], (err, results) => {
    if (err) console.log(err);
    const user = results[0];
    console.log(user.name);
});

// With ORM (Sequelize)
const user = await User.findOne({ where: { id: userId, status: 'active' } });
console.log(user.name);
```

### Popular Node.js ORMs:
- **Sequelize** - Best for MySQL/PostgreSQL, beginner-friendly
- **Prisma** - Modern, auto-generates types, cleanest API
- **TypeORM** - Good if using TypeScript
- **Mongoose** - Specific for MongoDB

### Code Example (Sequelize):

```javascript
// Define model (like database blueprint)
const User = sequelize.define('User', {
    id: { type: DataTypes.INTEGER, primaryKey: true },
    name: { type: DataTypes.STRING, allowNull: false },
    email: { type: DataTypes.STRING, unique: true },
    age: { type: DataTypes.INTEGER }
});

// Use in routes
// CREATE
app.post('/users', async (req, res) => {
    const user = await User.create(req.body);
    res.json(user);
});

// READ
app.get('/users/:id', async (req, res) => {
    const user = await User.findByPk(req.params.id);
    res.json(user);
});

// UPDATE
app.put('/users/:id', async (req, res) => {
    const user = await User.findByPk(req.params.id);
    await user.update(req.body);
    res.json(user);
});

// DELETE
app.delete('/users/:id', async (req, res) => {
    await User.destroy({ where: { id: req.params.id } });
    res.json({ message: 'Deleted' });
});
```

### ORM vs Raw SQL:

| Scenario | ORM | Raw SQL |
|----------|-----|---------|
| Simple CRUD | ✅ Use ORM | ❌ Too verbose |
| Complex JOINs (5+) | ⚠️ Works but slow | ✅ Better |
| Maximum performance | ❌ Overhead | ✅ Faster |
| MVP/Learning | ✅ Fast build | ❌ Slow |
| Data integrity critical | ✅ Built-in | ⚠️ Manual |

---

## 17. SQL Injection Prevention

**What is it?** Attacker inserts malicious SQL code through input fields to manipulate database.

### Attack Example:
```
Username input: admin' OR '1'='1
Query becomes: SELECT * FROM users WHERE username = 'admin' OR '1'='1'
Result: Returns all users, bypassing password check!
```

### Prevention Methods:

**1. Parameterized Queries (BEST - ALWAYS USE THIS)**
```javascript
// Placeholder (?) separates SQL from data
const query = "SELECT * FROM users WHERE username = ? AND password = ?";
connection.query(query, [username, password], (err, results) => {
    // Safe! Even if username = "admin' OR '1'='1", it's treated as literal string
});
```

**2. Input Validation (Extra layer)**
```javascript
// Validate before querying
if (!/^[a-zA-Z0-9_]{3,20}$/.test(username)) {
    return res.json({ success: false, message: "Invalid username" });
}
if (password.length < 6) {
    return res.json({ success: false, message: "Invalid password" });
}
// Then do parameterized query
```

**3. Use ORM (Easiest)**
```javascript
// ORM automatically parameterizes everything
const user = await User.findOne({ where: { username, password } });
// SQL injection impossible - ORM handles it
```

### Why Parameterized Queries Work:
```
Database sees: 
SQL: SELECT * FROM users WHERE username = ? AND password = ?
Data: ['admin\' OR \'1\'=\'1', 'anything']

Database KNOWS the second part is DATA, not SQL code
So it looks for username EXACTLY matching that string (won't find it)
Injection prevented!
```

### In Your Node.js Website:
```javascript
// Option 1: Parameterized queries (mysql2)
connection.query("SELECT * FROM users WHERE id = ?", [userId], callback);

// Option 2: ORM (Sequelize) - RECOMMENDED
const user = await User.findByPk(userId);

// NEVER do this:
connection.query("SELECT * FROM users WHERE id = " + userId);  // DANGEROUS!
```

---

## Quick Interview Answers

**Q: What is SQL?**
A: Language to manage databases. Used for CRUD operations, data integrity, and reporting.

**Q: SQL vs NoSQL?**
A: SQL is structured with strong consistency (ACID). NoSQL is flexible with eventual consistency. SQL scales vertically, NoSQL scales horizontally.

**Q: What is ACID?**
A: Rules for safe transactions. Atomicity (all or nothing), Consistency (valid data), Isolation (no interference), Durability (permanent).

**Q: What is Normalization?**
A: Organizing database into separate tables to reduce redundancy. Most databases aim for 3NF. Prevents data duplication and inconsistency.

**Q: What is Denormalization?**
A: Intentionally combining tables to improve performance (faster reads). Trade-off: More storage and risk of inconsistency. Used for reporting and analytics.

**Q: What is an Index?**
A: Shortcut to find data faster (like book's index). Makes SELECT queries faster but slows down INSERT/UPDATE/DELETE. Use on columns frequently searched.

**Q: What is SQL Injection?**
A: Hacker inserts SQL code through input to manipulate database. Prevent with parameterized queries, input validation, or ORM.

**Q: What is ORM?**
A: Write database operations using objects instead of SQL. ORM converts to SQL automatically. Cleaner, safer, prevents SQL injection, faster to build.

**Q: WHERE vs HAVING?**
A: WHERE filters before grouping, HAVING filters after. Use HAVING with aggregate functions like COUNT or SUM.

**Q: What is a foreign key?**
A: Links to another table's primary key. Creates relationships between tables and enforces referential integrity.

**Q: Difference between Primary Key and Index?**
A: Primary key is unique ID for each row (only 1 per table). Index is a shortcut to find data faster (many per table). Primary key is constraint, index is performance tool.

**Q: When would you denormalize?**
A: For reporting/analytics (flatten tables for faster queries), high-traffic sites (accept slight inconsistency for speed), or read-heavy applications.

**Q: How do you know if an index is helping?**
A: Use EXPLAIN in SQL to see if index is used. Monitor query performance before/after adding index. Remove unused indexes.

---

## Interview Tips
✅ Keep answers concise with examples
✅ Explain real-world trade-offs (speed vs consistency, storage vs performance)
✅ Master topics: ACID, SQL Injection, Normalization, ORM, JOINs
✅ Know when to normalize vs denormalize
✅ Understand indexes (when to use, when to avoid)
✅ Can explain your design decisions clearly
✅ Ask clarifying questions
✅ Show you think about performance and security
