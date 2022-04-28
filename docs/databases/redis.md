# Redis

Redis = In-memory Database/Cache

## Use Cases

- ใช้เก็บ Cache จากการ Query Database เพื่อลด Workload
- ใช้เป็น Message Queue (เช่นที่ใช้กับ Apache Airflow)
- เก็บข้อมูล Session ในการใช้งานเว็บ เช่น จำนวน Page View, User Profiles, Credentials, User-Specific Data
- ใช้ร่วมกับ Apache Kafka ในการทำ Real-time Analytics

## Redis CLI

```bash
docker compose exec redis redis-cli
```

## Create/Update/Read

### SET/GET

ครอบคลุมการใช้งาน Redis > 70%

```text
SET name "Jonathan"
GET name
```

### Namespaces

Redis มี Data Structure หลายแบบแต่ส่วนใหญ่ก็จะใช้งาน SET/GET เราสามารถแบ่ง Namespace เพื่อป้องกันชื่อ Key ซ้ำได้ โดยใช้ `:` (เป็น Convention ที่ใช้กัน ไม่ใช่สิ่งที่กำหนดตายตัวใน Redis)

```text
SET user:jonathan:department DPD
SET user:jonathan:age 46
SET user:jonathan:vehicle Camry
```

### Maths

ใช้ฟังก์ชั่นคณิตศาสตร์พื้นฐานกับ Redis ได้เช่น

```text
SET satisfactory_score 80
INCR satisfactory_score
DECR satisfactory_score

INCRBY satisfactory_score 40
DECRBY satisfactory_score 170
```

### Conditions

ตรวจสอบก่อน `SET` ค่า

- `XX` หมายถึง ถ้าไม่มี ไม่ต้องเซ็ต SET ค่า
- `NX` หมายถึง ถ้าไม่มี ให้ SET ค่า

```text
SET color blue XX
SET color blue NX
```

### Cache Invalidation

ทำให้ Cache หมดเวลาตามกำหนด

`EX 3600` หมายถึง หมดอายุ 3600 วินาที

```text
SET user:jonathan:token 789kkjjxx EX 3600
```

## Delete

```text
DEL user:jonathan:department
```

## Data Types

### Lists

ปกติใช้เก็บพวกรายการต่างๆ เช่น Task Queues

```text
RPUSH airflow:queue "Task 1" "Task 2" "Task 3" "Task 4"
LRANGE airflow:queue 0 -1
```

สามารถดึง Element ออกจาก List ได้ด้วย `LPOP`, `RPOP`

```text
LPOP airflow:queue
RPOP airflow:queue
```

### Hashes

ตัวอย่าง: ใช้เก็บพวก User Profiles สำหรับ Web Application โดย Hash ทุกสิ่งอย่างภายใต้ Key ของ User เวลาอ่านมาแสดงผลใน Web App ทำได้เลยสะดวก

```text
HSET jonathan:profile title "Bullshit Director" company "Submarine Electricity Minority" city "Bangkok"
HGET jonathan:profile title
HGETALL
```

### Sets

ใช้เก็บรายการที่ Unique ตามคุณสมบัติของ Set

```text
SADD colors red blue yellow green black pink brown
SISMEMBER colors dog
SMEMBERS colors
SPOP colors
```
