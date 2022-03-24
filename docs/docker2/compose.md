# Docker Compose

Docker Compose คือ Tool ที่ใช้สำหรับ Define และ Run Applications ที่ใช้หลาย Containers เช่น Wordpress App Suite ที่ต้องมี Wordpress และ Database

## Compose File

วิธีการ Define Multi-container Applications เริ่มต้นด้วยการสร้างไฟล์ YAML ชื่อ `docker-compose.yml` (หรือชื่ออื่นๆ แต่ต้องระบุชื่อตอนรัน) และใส่นิยาม Containers ที่ต้องรันลงไปในไฟล์

Compose File มีหลายเวอร์ชั่น แต่ละเวอร์ชั่นรองรับ Docker Engine เวอร์ชั่นต่างกัน แนะนำให้ใช้งานตั้งแต่ เวอร์ชั่น 2 ขึ้นไป [ดูรายละเอียดของแต่ละเวอร์ชั่นที่นี่](https://docs.docker.com/compose/compose-file/compose-versioning/)

## Structure

Compose file ประกอบด้วยชิ้นส่วนพื้นฐาน:

```text
version: <compose_file_version>

services:
  <service_name>:
    <container_spec>
  <service_name>:
    <container_spec>
  ...
```

ในกรณีที่มีการใช้งาน Volumes หรือ User Defined Networks จะต้องมี Section `volumes` และ `networks` เพิ่ม รายละเอียดในตัวอย่าง

## Example 1 - Wordpress App

### Wordpress Architecture

![Wordpress Arch](images/wordpress_arch.png)

### Compose Definition

ตัวอย่างอยู่ใน `docker-tutorial-2/wordpress/docker-compose.yml`

```text title='docker-compose.yml'
version: "3"

services:
  db:
    image: mysql:5.7
    volumes:
      - db_data:/var/lib/mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: wordpress
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: wordpress

  wordpress:
    depends_on:
      - db
    image: wordpress:latest
    volumes:
      - wordpress_data:/var/www/html
    ports:
      - "8088:80"
    restart: always
    environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: wordpress
      WORDPRESS_DB_NAME: wordpress
volumes:
  db_data: {}
  wordpress_data: {}
```

### Start Compose

```bash
docker compose up -d
```

โดย Default ถ้าไม่ระบุชื่อ Network ตัว Compose จะตั้งชื่อให้อัตโนมัติเป็น `foldername_default` เช่น ถ้าไฟล์ `docker-compose.yml` อยู่ใน Folder `wordpress` ชื่อ Network จะเป็น `wordpress_default` และใช้ Subnet Default หรือที่เราตั้งค่าไว้ใน Daemon แต่แรก หลังรันคำสั่งแล้ว รอให้ Service Up สักพักแล้วเปิดเว็บ [http://localhost:8088](http://localhost:8088) เพื่อติดตั้ง Wordpress ต่อให้จบกระบวนการ

### Stop Compose

```bash
docker compose down
```

หากต้องการทำลาย Volumes ด้วย

```bash
docker compose down -v
```

### Read Logs

```bash
docker compose logs <service_name>
```

เช่น

```bash
docker compose logs db
```

### Execute Command

```bash
docker compose exec <service_name> <command>
```

เช่น

```bash
docker compose exec wordpress bash
```

## Example 2 - Database Dev Environment

### DB Dev Architecture

![DB Dev Arch](images/dbdev_arch.png)

### DB Dev Compose File

```text title='docker-compose.yml'
version: '3'

services:
  # PostgreSQL: SQL
  postgres:
    image: postgres
    restart: always
    environment:
      POSTGRES_DB: dev
      POSTGRES_USER: dev
      POSTGRES_PASSWORD: devpassword
    volumes:
      - postgres_data:/var/lib/postgresql/data

  cloudbeaver:
    image: dbeaver/cloudbeaver
    restart: always
    ports:
      - '9999:8978'
    volumes:
      - cloudbeaver_data:/opt/cloudbeaver/workspace

  # MongoDB: NoSQL
  mongo:
    image: mongo
    restart: always
    environment:
      MONGO_INITDB_ROOT_USERNAME: dev
      MONGO_INITDB_ROOT_PASSWORD: devpassword
    volumes:
      - mongo_data:/data/db

  adminmongo:
    image: mrvautin/adminmongo
    restart: always
    ports:
      - '1234:1234'
    environment:
      HOST: 0.0.0.0

  # Redis: Key-value store
  redis:
    image: redis
    restart: always
    volumes:
      - redis_data:/data

  insight:
    image: redislabs/redisinsight
    restart: always
    ports:
      - '8001:8001'
    volumes:
      - insight_data:/db

  # Neo4j: Graph
  neo4j:
    image: neo4j
    restart: always
    ports:
      - '7474:7474'
      - '7687:7687'
    volumes:
      - neo4j_data:/data

volumes:
  cloudbeaver_data:
  postgres_data:
  mongo_data:
  redis_data:
  insight_data:
  neo4j_data:
```

### Start DB Dev

```bash
docker compose up -d
```

### Using DV Dev

ใน Compose มี Web UI สำหรับบริหารจัดการ Database แต่ละประเภทให้ ซึ่งทีมเราต้องเข้าไป Support

Database ในองค์กรส่วนใหญ่เป็น SQL 90%, NoSQL 9% ที่เหลือ Key-Value กับ Graph ค่อนข้างน้อย ใน Session ถัดไปเราจะมาลงรายละเอียดวิธี CRUD DB แต่ละประเภทเพื่อซัพพอร์ตงานในอนาคต วิธีการเข้าใช้งาน Web UI

| Name          | Location                                       | Username   | Password   |
| :------------ | :--------------------------------------------- | :--------- | :--------- |
| Cloudbeaver   | [http://localhost:9999](http://localhost:9999) | Set in App | Set in App |
| AdminMongo    | [http://localhost:1234](http://localhost:1234) | None       | None       |
| Redis Insight | [http://localhost:8001](http://localhost:8001) | None       | None       |
| Neo4J UI      | [http://localhost:7474](http://localhost:7474) | None       | None       |

Default Database Credentials

| Name       | Connection    | Username | Password    |
| :--------- | :------------ | :------- | :---------- |
| PostgreSQL | postgres:5432 | dev      | devpassword |
| MongoDB    | mongo:27017   | dev      | devpassword |
| Redis      | redis:6379    | None     | None        |
| Neo4J      | neo4j:7687    | neo4j    | neo4j       |

**เก็บ Compose อันนี้ไว้ก่อน จะใช้ศึกษา Database ภายหลัง**

## Example 3 - Apache Airflow

Airflow สนับสนุนวิธีการ Deploy หลายแบบ ในองค์กรเราใช้ [Celery Executor](https://airflow.apache.org/docs/apache-airflow/stable/executor/celery.html) ในอนาคตจะย้ายไป [Kubernetes Executor](https://airflow.apache.org/docs/apache-airflow/stable/executor/kubernetes.html)

### Airflow Architecture

![Airflow Arch](images/airflow_arch.png)

### Airflow Compose

```text title='docker-compose.yml'
version: '3.7'

# ====================================== AIRFLOW ENVIRONMENT VARIABLES =======================================
x-environment: &airflow_environment
  - AIRFLOW__CELERY__BROKER_URL=redis://redis:6379/1
  - AIRFLOW__CELERY__RESULT_BACKEND=db+postgresql://airflow:airflow@postgres:5432/airflow
  - AIRFLOW__CORE__EXECUTOR=CeleryExecutor
  - AIRFLOW__CORE__LOAD_DEFAULT_CONNECTIONS=False
  - AIRFLOW__CORE__LOAD_EXAMPLES=False
  - AIRFLOW__CORE__SQL_ALCHEMY_CONN=postgresql://airflow:airflow@postgres:5432/airflow
  - AIRFLOW__METRICS__STATSD_HOST=statsd_exporter
  - AIRFLOW__METRICS__STATSD_ON=True
  - AIRFLOW__METRICS__STATSD_PORT=9125
  - AIRFLOW__WEBSERVER__EXPOSE_CONFIG=True

x-airflow-image: &airflow_image apache/airflow:2.0.0-python3.8
# ====================================== /AIRFLOW ENVIRONMENT VARIABLES ======================================

services:
  postgres:
    image: postgres:12-alpine
    environment:
      - POSTGRES_USER=airflow
      - POSTGRES_PASSWORD=airflow
      - POSTGRES_DB=airflow
    ports:
      - "5432:5432"
    volumes:
      - postgres_airflow:/var/lib/postgresql/data

  init:
    image: *airflow_image
    depends_on:
      - postgres
    environment: *airflow_environment
    entrypoint: /bin/bash
    command: -c 'airflow db init && airflow users create --username admin --password admin --firstname Anonymous --lastname Admin --role Admin --email admin@example.org'

  webserver:
    image: *airflow_image
    restart: always
    depends_on:
      - postgres
    ports:
      - "8080:8080"
    volumes:
      - logs:/opt/airflow/logs
    environment: *airflow_environment
    command: webserver

  scheduler:
    image: *airflow_image
    restart: always
    depends_on:
      - postgres
    volumes:
      - logs:/opt/airflow/logs
      - ./dags:/opt/airflow/dags
    environment: *airflow_environment
    command: scheduler

  # docker-compose -f docker-compose-celeryexecutor.yml up --scale worker=3 -d
  worker:
    image: *airflow_image
    restart: always
    depends_on:
      - scheduler
    volumes:
      - logs:/opt/airflow/logs
      - ./dags:/opt/airflow/dags
    environment: *airflow_environment
    command: celery worker

  flower:
    image: *airflow_image
    restart: always
    depends_on:
      - worker
    ports:
      - "5555:5555"
    environment: *airflow_environment
    command: celery flower

  statsd_exporter:
    image: prom/statsd-exporter:v0.18.0
    restart: always
    volumes:
      - ./files/statsd_mapping.yml:/tmp/statsd_mapping.yml
    ports:
      - "9102:9102"
      - "9125:9125/udp"
    command: --statsd.mapping-config=/tmp/statsd_mapping.yml

  prometheus:
    image: prom/prometheus:v2.22.0
    restart: always
    volumes:
      - ./files/prometheus.yml:/etc/prometheus/prometheus.yml
    ports:
      - "9090:9090"
    command:
      - --web.enable-admin-api
      - --web.enable-lifecycle
      - --config.file=/etc/prometheus/prometheus.yml
      - --storage.tsdb.path=/prometheus
      - --web.console.libraries=/usr/share/prometheus/console_libraries
      - --web.console.templates=/usr/share/prometheus/consoles

  grafana:
    image: grafana/grafana:7.2.1
    restart: always
    ports:
      - "3000:3000"

  redis:
    image: redis:5-alpine

  redis_exporter:
    image: oliver006/redis_exporter:v1.5.2-alpine
    ports:
      - "9121:9121"
    command: --redis.addr=redis://redis:6379

volumes:
  postgres_airflow:
  logs:
```

### Start Apache Airflow

```bash
docker compose up -d
```

### Check Scheduler Logs

ใช้ตรวจสอบขั้นตอนที่ Airflow จัดคิวในการประมวลผล Task ตรวจสอบ Logs 100 บรรทัดล่าสุด

```bash
docker compose logs --tail=100 scheduler
```

### Check Webserver Logs

ตรวจสอบ Logs ของ Web UI

```bash
docker compose logs --tail=100 webserver
```

### Check Worker Logs

ตรวจสอบการรัน Task

```bash
docker compose logs --tail=100 worker
```

### Check Monitoring System

- **Queuing System** [Celery Flower](http://localhost:5555)
- **Prometheus Metrics** [Prometheus UI](http://localhost:9090)
- **Dashboard** [Grafana](http://localhost:3000)

### Test Python/Shell Code

ในบางกรณี อาจจะต้องทดสอบ Python/Shell Code ใน Worker วิธีการทดสอบง่ายที่สุดคือการ SSH เข้าไปที่ Worker และ Import Airflow มาใช้งานเพื่อทดสอบได้เลย ส่วนใหญ่ใช้ในกรณี

- ทดสอบว่า Worker สามารถต่อ Database ได้หรือไม่
- ทดสอบว่ามี Module ที่ต้องการหรือยัง
- ทดสอบว่า Worker มี System Library ที่ต้องการหรือยัง เช่น Oracle Database Libary
- ทดสอบว่า Worker สามารถอ่าน Config Bind Mounts ได้ถูกต้อง

```bash title='python'
docker compose exec worker ipython
```

```bash title='bash'
docker compose exec worker bash
```

### Production Differences

ความแตกต่างของตัวอย่างนี้กับ Production Airflow คือ ใน Production ...

- ติดตั้ง Common Python Modules และ Airflow Connection Plugins ทั้งหมดที่มี
- ใช้งาน Service ที่สร้างและ Refresh Kerberos Ticket สำหรับการสื่อสารกับ Cloudera Cluster
- ติดตั้ง Common Drivers เช่น Oracle, Hive JDBC, Impala JDBC, ODBC เป็นต้น
- Integrate กับ Active Directory (ตั้งค่าสิทธิผ่าน AD Group) และ Internal Email Server
- Integrate DAGs กับ MinIO และมีการ Backup & Cleanup รายวัน
- มี Development Storage ให้ทดลอง PostgreSQL, MongoDB, MariaDB และ SFTP
- มีการ Optimize Scheduler & Core Configuration สำหรับการทำงานแบบขนาน
- ใช้ Scheduler 3 ตัว และ Workers 15 ตัว บน vCPU 92 Cores และ RAM 312 GB

ข้อเสียของ Production ปัจจุบัน

- ไม่มี Workload Isolation
- การ Scale ยังเป็น Manual ยังไม่สามารถ Auto Scale ตาม Workload
- ไม่มีการเซ็ต App-Level Logging System (Airflow สนับสนุน ELK, FluentD แต่ยังไม่ได้เซ็ต)
- ไม่ได้ Integrate กับ Git การพัฒนา DAG ลำบากและไม่ Smooth
- ไม่มีกระบวนการ Backup Database อัตโนมัติ
- มีแต่ App-Level ไม่มี VM-Level HA (High Availability)
