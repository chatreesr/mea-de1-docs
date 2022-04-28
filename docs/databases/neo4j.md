# Neo4j

Neo4j = Graph Database

## Use Cases

- ใช้โมเดลความสัมพันธ์ใน Social Network
- ใช้โมเดลระบบไฟฟ้า ระบบเน็ตเวิร์ก ระบบการจราจร
- ใน กฟน. ยังไม่มีคนใช้งาน Database จริงๆจังๆ แต่มีการโมเดลระบบไฟฟ้าส่วนย่อยด้วย Graph

## Cypher Shell & Query Language

ใช้งาน Cypher Shell

```bash
docker compose exec neo4j cypher-shell
```

Syntax พื้นฐานของ Cypher Query Language

```text
(label {property})-[relationship]->(label {property})
```

## Create

### Create a Node

สร้าง Node ที่มี Label ชื่อ `Person` และ Node ที่มี Label ชื่อ `Department`

```text
CREATE (p:Person {name: 'Jonathan', age: 45 });
CREATE (d:Department {name: 'DPD', floor: 5});
```

### Create a Relationship

สร้าง Relationship หลังสร้าง Nodes

```text
MATCH (p:Person), (d:Department)
WHERE p.name='Jonathan' AND d.name='DPD'
CREATE (p)-[r:WORKS_IN { rank: 'Assistant Director'}]->(d)
RETURN r;
```

หรือจะสร้าง Relationship พร้อม Nodes ใหม่เลย

```text
CREATE (p:Person {name: 'Elizabeth', age: 53})-[r:WORKS_IN {rank: 'C9'}]->(d:Department {name: 'DPD', floor: 5})
```

ก่อนจะทดสอบหัวข้อถัดไป ให้ลบทุกอย่างก่อน

```text
MATCH (x)
DETACH DELETE x
```

## Read

ใช้ Neo4J Browser ในการ Read จะสะดวกกว่า [http://localhost:7474](http://localhost:7474)

พิมพ์ `:play movie` เพื่อ Generate Sample Data

### Find All

```text
MATCH (n)
RETURN n
```

### Find Nodes & Relationships

หานักแสดงชื่อ Tom Hanks

```text
MATCH (p:Person {name: 'Tom Hanks'}) RETURN p;
```

```text
MATCH (p:Person)
WHERE p.name='Tom Hanks'
RETURN p;
```

หาหนังที่ Tom Hanks แสดง

```text
MATCH (p:Person)-[r:ACTED_IN]->(m:Movie)
WHERE p.name='Tom Hanks'
RETURN m;
```

หาบทบาท (Relationship) ที่ Tom Hanks เคยแสดง

```text
MATCH (p:Person)-[r:ACTED_IN]->(m:Movie)
WHERE p.name='Tom Hanks'
RETURN r;
```

แสดงกราฟความสัมพันธ์ของหนังและบทบาทที่ Tom Hanks เคยแสดง ถ้า Displayบนเว็บจะเห็น Relationship `DIRECTED` เพิ่มขึ้นเนื่องจากโดย Default Neo4j Browser จะทำการเชื่อมโยง Nodes โดยอัตโนมัติ หากต้องการปิดโหมดนี้ให้ไปที่ `Settings` > เอาเครื่องหมายถูกออกจาก `Connect result nodes`

```text
MATCH (p:Person)-[r:ACTED_IN]->(m:Movie)
WHERE p.name='Tom Hanks'
RETURN p, r, m;
```

หานักแสดงที่แสดงหนังเรื่องเดียวกับ Keanu Reeves

```text
MATCH (p:Person)-[:ACTED_IN]->(:Movie)<-[:ACTED_IN]-(q:Person)
WHERE p.name = 'Keanu Reeves'
RETURN q.name;
```

หาจำนวนครั้งที่นักแสดงแสดงหนังร่วมกับ Keanu Reeves

```text
MATCH (p:Person)-[:ACTED_IN]->(:Movie)<-[:ACTED_IN]-(q:Person)
WHERE p.name = 'Keanu Reeves'
RETURN DISTINCT q.name, count(*)
ORDER BY count(*) DESC, q.name;
```

หา Shortest Path ระหว่าง Keanu Reeves กับ Tom Hanks

```text
MATCH path = shortestPath(
  (k:Person {name: 'Keanu Reeves'})-[*]-(t:Person {name: 'Tom Hanks'})
)
RETURN path;
```

## Update

### Update a Node

```text
MATCH (p:Person {name: 'Keanu Reeves'})
SET p.name = 'Jonathan Reeves'
RETURN p;
```

### Update a Relationship

```text
MATCH (p:Person {name: 'Jonathan Reeves'})-[r:ACTED_IN]-(m:Movie {title: 'The Matrix'})
SET r.roles = ['Jona NEO']
RETURN p, r, m;
```

## Delete

### Delete All

```text
MATCH (x)
DETACH DELETE x;
```

### Delete a Node with All Relationships

แนวคิดคือ ถ้าต้องการจะลบ Node ต้องลบ Relationships ด้วย วิธีง่ายที่สุดคือการใช้ `DETACH DELETE` เพราะไม่ต้องไปลบ Relationships ก่อน

```text
MATCH (j:Person { name: 'Jonathan Reeves' })
DETACH DELETE j;
```

### Delete Relationships Only

```text
MATCH (t:Person {name: 'Tom Hanks'})-[r:DIRECTED]->()
DELETE r;
```

## Index

ตรวจสอบการ Query

```text
EXPLAIN MATCH (p:Person) WHERE p.born=1967 RETURN p.name;
```

ทดลองสร้าง Index

```text
CREATE INDEX FOR (p:Person) ON (p.born);
```

ดู Query Plan อีกครั้ง

```text
EXPLAIN MATCH (p:Person) WHERE p.born=1967 RETURN p.name;
```
