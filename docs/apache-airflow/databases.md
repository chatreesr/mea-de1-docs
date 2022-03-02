# Databases

## การเชื่อมต่อ Databases

วิธีการเชื่อมต่อ

** เขียน Python Function และใช้กับ `PythonOperator` **

- **แนะนำ** ใช้ `sqlalchemy` ใช้ได้กับหลาย Database เช่น PostgreSQL, MySQL, Oracle, MSSQL
- PostgreSQL ใช้ `psycopg2`
- MSSQL ใช้ `pymssql`
- MySQL ใช้ `mysql-connector-python`
- MongoDB ใช้ `pymongo`
- Neo4j ใช้ `neo4j`
- Redis ใช้ `redis`
- ถ้า Database สนับสนุน ODBC ใช้ `pyodbc`
- ฐานข้อมูล TIBCO Data Virtualization ใช้ `pyodbc`

** ใช้ Airflow Operators **

- [PostgreSQL](https://airflow.apache.org/docs/apache-airflow-providers-postgres/2.2.0/operators/postgres_operator_howto_guide.html)
- [MongoDB](https://airflow.apache.org/docs/apache-airflow-providers/operators-and-hooks-ref/software.html#mongodb)
- [MSSQL](https://airflow.apache.org/docs/apache-airflow-providers/operators-and-hooks-ref/software.html#microsoft-sql-server-mssql)
- [MySQL](https://airflow.apache.org/docs/apache-airflow-providers/operators-and-hooks-ref/software.html#mysql)
- [Redis](https://airflow.apache.org/docs/apache-airflow-providers/operators-and-hooks-ref/software.html#redis)
- [Neo4j](https://airflow.apache.org/docs/apache-airflow-providers/operators-and-hooks-ref/software.html#neo4j)

## การเชื่อมต่อ Data Platform

รายละเอียดอ่านที่ [Data Platform Apache Airflow](https://dataplatform-portal.vercel.app/docs/use/apache-airflow) แต่โดยปกติ Operators ของ Airflow จะใช้กับระบบเราไม่ได้ เพราะติดปัญหาเรื่อง Policies และ Security Kerberos โดยสรุป

- HDFS ใช้ [WebHDFS Hook](https://airflow.apache.org/docs/apache-airflow-providers/operators-and-hooks-ref/apache.html#webhdfs)
- Hive/Impala ใช้ [JDBCOperator](https://airflow.apache.org/docs/apache-airflow-providers/operators-and-hooks-ref/protocol.html#java-database-connectivity-jdbc)
- Spark ใช้ [LivyOperator](https://airflow.apache.org/docs/apache-airflow-providers/operators-and-hooks-ref/apache.html#apache-livy)
- Sqoop ใช้ [SSHOperator](https://airflow.apache.org/docs/apache-airflow-providers/operators-and-hooks-ref/protocol.html#secure-shell-ssh)
- HBase ใช้ `PythonOperator` ร่วมกับ HBase REST API หรือใช้งานผ่าน Apache Phoenix ด้วย `phoenixdb`
