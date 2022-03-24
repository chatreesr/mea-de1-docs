# Images

เราสามารถสร้าง Image โดยใช้ Image ชาวบ้านเป็นสารตั้งต้น ตัวอย่างการใช้งาน เช่น

- Customize App ตามความต้องการ
- Package App ที่พัฒนาด้วยตนเองให้พร้อมใช้งาน

## Dockerfile

ขั้นตอนในการสร้าง Image ใช้เอง

- เขียน Dockerfile ตามคำแนะนำ [Dockerfile Reference](https://docs.docker.com/engine/reference/builder/)
- Build Image
- Push Image ไปที่ Repository (Docker Hub)
- Run Image เพื่อใช้งาน

## Example 1 - Dev Notebook

ต้องการสร้าง Python Dev Environment ที่มี Package ยอดนิยมครบถ้วนพร้อมใช้งานผ่าน Jupyter Notebook

### Python Dockerfile

ให้ สร้างไฟล์ชื่อ `Dockerfile` (ไม่มี Extension) และ Copy ชุดคำสั่งด้านล่าง

```text
FROM ubuntu:20.04

RUN mkdir /home/workspace

WORKDIR /home/workspace

RUN apt-get update && apt-get install -y python3 python3-pip

RUN pip3 install pandas matplotlib seaborn numpy notebook jupyterlab viola

EXPOSE 8888

CMD ["jupyter", "notebook", "--no-browser", "--ip=0.0.0.0", "--allow-root", "--NotebookApp.token=''", "--NotebookApp.password=''"]
```

### Build Jupyter Image

สร้าง Local Image ด้วยคำสั่ง

```bash
docker build -t chatreesr/python-dev:test .
```

- `-t` คือการ tag ชื่อ image
- `chatreesr/python-dev:test` คือชื่อ Image พร้อม Tag
- `.` คือ ให้อ่าน Dockerfile จาก Current Directory

### Run Jupyter Image

```bash
docker container run \
    -d --name jupyter-dev \
    -p 8888:8888 \
    -v $(pwd)/src:/home/workspace \
    chatreesr/python-dev:test
```

เปิดเว็บไซต์ [http://localhost:8888](http://localhost:8888) เขียนโปรแกรมได้ตามปกติ Source Code จะถูก Sync ที่ `$(pwd)/src`

### Push Jupyter Image

```bash
docker image push chatreesr/python-dev:test
```

ต้องทำการ Login ด้วยคำสั่ง `docker login` ก่อน

## Example 2 - Airflow

บางกรณีจำเป็นต้องสร้าง Custom Image เอง เช่น ลงโปรแกรมเพิ่ม ลงไลบรารี่เพิ่ม ตัวอย่าง Airflow จะแสดงให้เห็นวิธีการลง Plugins ของ Airflow เพิ่มเพื่อให้สามารถเชื่อมต่อ Database ต่างๆได้ ในระบบ Production ตัว Airflow จะลงทุก Connections และ ไลบรารี่ที่ใช้งานบ่อยๆ กระบวนการสร้าง Image จะซับซ้อนกว่านี้นิดหน่อย แต่แนวคิดเดียวกัน

### Create an Airflow Dockerfile

```text
FROM apache/airflow:2.2.4-python3.8
USER root

# Install system packages & clean up
RUN apt-get update \
    && apt-get install -y --no-install-recommends \
        build-essential \
        # MySQL dependency
        default-libmysqlclient-dev \
        # ODBC dependency
        unixodbc-dev \
    && apt-get autoremove -yqq --purge \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Install Python packages
RUN pip install --no-cache-dir \
        apache-airflow-providers-airbyte==2.1.1 \
        apache-airflow-providers-apache-cassandra==2.1.0 \
        apache-airflow-providers-elasticsearch==2.2.0 \
        apache-airflow-providers-docker==2.4.1 \
        apache-airflow-providers-microsoft-mssql==2.1.0 \
        apache-airflow-providers-http==2.0.3 \
        apache-airflow-providers-imap==2.2.0 \
        apache-airflow-providers-influxdb==1.1.0 \
        apache-airflow-providers-jdbc==2.1.0 \
        apache-airflow-providers-postgres==3.0.0 \
        apache-airflow-providers-mongo==2.3.0 \
        apache-airflow-providers-mysql==2.2.0 \
        apache-airflow-providers-neo4j==2.1.0 \
        apache-airflow-providers-odbc==2.0.1 \
        apache-airflow-providers-redis==2.0.1

USER airflow
```

### Build Airflow Image

```bash
docker build -t chatreesr/airflow-py3.8:test .
```

### Push Airflow Image

```bash
docker push chatreesr/airflow-py3.8:test
```

## Example 3 - Python API App

ใช้ขั้นตอนการพัฒนา Python App ปกติ

1. ทำการรันคำสั่ง `python3 -m venv venv` เพื่อสร้าง Virtual Environment ชื่อ `venv`
2. Activate environment ด้วยคำสั่ง `source venv/bin/activate`
3. เขียนโปรแกรม และ ลง Module ด้วยคำสั่ง `pip install <module_name>`
4. หลังจากเสร็จแล้ว พร้อม Deploy ให้ List รายการ Module ที่ต้องการใช้ด้วยคำสั่ง `pip freeze > requirements.txt`

### API Dockerfile

Source code ของโปรแกรม API อยู่ใน Folder `docker-python-app`

```text
FROM python:3.9
RUN mkdir /app
WORKDIR /app

# Install dependencies
COPY requirements.txt .
RUN pip install -r requirements.txt

# Run the applications
COPY main.py .
COPY data.json .
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "9999"]
EXPOSE 9999
```

### Build Python Image

```bash
docker build -t chatreesr/pyapi:test .
```

### Push Python Image

```bash
docker push chatreesr/pyapi:test
```

### Run Python API

รัน API ด้วยคำสั่งด้านล่าง ทดลองเรียก API ที่ [http://localhost:9999](http://localhost:9999) ดู API Docs ได้ที่ [http://localhost:9999/docs](http://localhost:9999/docs)

```bash
docker container run  \
  --name pyapi -d \
  -p 9999:9999 \
  chatreesr/pyapi:test
```
