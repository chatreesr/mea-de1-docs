# PostgreSQL

PostgreSQL = Relational Databases

## Use Cases

- ครอบจักรวาล

## psql

ใส่ Password เป็น `devpassword`

```bash
docker compose exec postgres psql -U dev
```

## Create

### Database

```sql
CREATE DATABASE message_board;
```

ใช้คำสั่ง `\connect message_board` ในการเชื่อมต่อ Database

### Table

```sql
CREATE TABLE users(
    user_id INTEGER PRIMARY KEY GENERATED ALWAYS AS IDENTITY,
    username VARCHAR(25) UNIQUE NOT NULL,
    email VARCHAR(50) UNIQUE NOT NULL,
    full_name VARCHAR(100) NOT NULL,
    last_login TIMESTAMP,
    created_on TIMESTAMP NOT NULL
);
```

### Ingest Sample Data

ดาวโหลดไฟล์ SQL จาก [Sample PostgreSQL Data](assets/sample-postgresql.sql)

### Insert

```sql
INSERT INTO users (userna, email, full_name, created_on)
VALUES ('jonathan', 'jona@lol.com', 'Jonathan Montgemery', NOW());
```

## Read

### Limit

```sql
SELECT *
FROM users
LIMIT 10;
```

### Projections

```sql
SELECT username, full_name
FROM users
LIMIT 5;
```

### Filters

```sql
SELECT username, email, user_id
FROM users
WHERE last_login IS NULL AND created_on < NOW() - interval '6 months'
LIMIT 10;
```

### Sort

```sql
SELECT user_id, email, created_on
FROM users
ORDER BY created_on DESC
LIMIT 10;
```

### Subqueries

```sql
SELECT comment_id, user_id, LEFT(comment, 20)
FROM comments
WHERE user_id = (
    SELECT user_id
    FROM users
    WHERE full_name='Maynord Simonich'
);
```

## Update

```sql
UPDATE users
SET full_name='Jonathan Montgemery', email='jonas@sem.com' WHERE user_id=9 RETURNING *;
```

## Delete

```sql
DELETE FROM users WHERE user_id=1000;
```

## Aggregation & Joins

```sql
SELECT boards.board_name, COUNT(*) AS comment_count
FROM comments
NATURAL INNER JOIN boards
GROUP BY boards.board_name
ORDER BY comment_count ASC
LIMIT 10;
```

## Indexes

```sql
SELECT comment_id, user_id, time, LEFT(comment, 20)
FROM comments
WHERE board_id=39
ORDER BY time DESC
LIMIT 40;
```

```sql
EXPLAIN SELECT comment_id, user_id, time, LEFT(comment, 20)
FROM comments
WHERE board_id=39
ORDER BY time DESC
LIMIT 40;
```

```sql
CREATE INDEX ON comments (board_id);
```
