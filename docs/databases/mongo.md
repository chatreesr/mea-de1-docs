# MongoDB

MongoDB = Document Database

## Use Cases

- เก็บข้อมูลประเภท Unstructured ได้ดี ไม่ต้องกังวลเรื่อง Schema ก่อนเก็บข้อมูล
- ใช้งานกับข้อมูลขนาดใหญ่ได้
- ค่อนข้างจะเป็นฐานข้อมูลเอนกประสงค์ เช่น ใช้ในอุตสาหกรรมเกมส์ การพัฒนา Web App การวิเคราะห์ Real Time Analytics ใช้เก็บข้อมูลประเภท Geospatial ใช้เป็น Data Lake
- ภายใน กฟน. มีการใช้งานเก็บข้อมูลจาก Web App หรือ Mobile App และใช้เป็น Data Lake

## Core Concepts

- **Collections** คล้ายกับ Tables ใน RDBMS ใช้จัดเก็บ Documents ต่างกันตรงที่ Collections เป็น Dynamic Schema และไม่มีซัพพอร์ท Foreign Key (สามารถสร้างเองได้แต่ต้องบริหารจัดการเอง)
- **Documents** คล้ายกับ Rows ใน RDBMS เป็นเนื้อข้อมูล อาจจะมองว่าเป็น JSON ก้อนหนึ่งก็ได้โดยในแต่ละ Document ไม่จำเป็นต้องมี Schema เดียวกัน

## Start a Mongo Shell

```javascript
docker compose exec mongo mongosh
```

Login เข้าใช้งาน MongoShell ด้วยคำสั่ง

```javascript
use admin;
db.auth('dev', 'devpassword');
```

## Create a New Database

ใช้คำสั่ง `use database_name` ได้เลย เช่น

```javascript
use company;
```

แสดงรายการ Databases:

```javascript
show dbs;
```

แสดงสถิติ Databases:

```javascript
db.stats()
```

ตรวจสอบว่าทำอะไรกับ Database ได้บ้าง:

```javascript
db.help()
```

## Create

ใช้ `db.collection_name.insert` collection_name ตั้งชื่อได้เลย MongoDB สร้างให้อัตโนมัติ

```javascript
db.managers.insertOne({
  name: 'Jonathan',
  rank: 'Assistant Director',
  age: 45,
  field: 'Electrical Engineering',
})
```

MongoDB ใช้ JavaScript เพราะฉะนั้นเราสามาถใช้ JavaScript ในการสร้างข้อมูลสำหรับการทดลองได้ เช่น

```javascript
db.managers.insertMany(
  Array.from({ length: 30000 }).map((_, idx) => ({
    index: idx,
    name: [
      'Jonathan',
      'Elizabeth',
      'Mohammad',
      'Jack',
      'Jane',
      'Rex',
      'Tyler',
      'Deimos',
      'Kassandra',
    ][idx % 9],
    rank: [
      'Assistant Director',
      'Director',
      'CEO',
      'CTO',
      'CDO',
      'CFO',
      'Engineer',
    ][idx % 7],
    age: (idx % 20) + 40,
    field: [
      'Electrical Engineering',
      'Computer Science',
      'Computer Engineering',
      'Economics',
      'MBA',
    ][idx % 5],
  }))
)
```

## Read

### find

```javascript
db.managers.find()
```

### findOne

```javascript
db.managers.findOne()
```

### Filters

```javascript
db.managers.find({ field: 'Electrical Engineering' })
db.managers.findOne({ field: 'Electrical Engineering' })
db.managers.find({ field: 'MBA', rank: 'CEO' }).limit(5)
```

### Query Operators

หาคนจบสาขา MBA ที่มีอายุ 50+

```javascript
db.managers.countDocuments({ field: 'MBA', age: { $gt: 50 } })
```

หาคนชื่อ Jonathan ที่อายุน้อยกว่าหรือเท่ากับ 45

```javascript
db.managers.find({ name: 'Jonathan', age: { $lte: 45 } })
```

ดูรายละเอียดเพิ่มเติมที่ [Query Operators](https://www.mongodb.com/docs/manual/reference/operator/query/)

### Logical Operators

หาตำแหน่ง CDO ที่อายุตั้งแต่ 45 - 50 ดูรายละเอียด [Logical Operators](https://www.mongodb.com/docs/manual/reference/operator/query-logical/)

```javascript
db.managers.find({
  rank: 'CDO',
  $and: [{ age: { $gte: 45 } }, { age: { $lte: 50 } }],
})
```

### Projections

Projections คือการเลือกเฉพาะข้อมูลที่สนใจ

```javascript
db.managers.find({ rank: 'CDO' }, { name: 1, field: 1 })
db.managers.find({ rank: 'CEO' }, { name: 1, age: 1, _id: 0 })
```

## Update

### updateOne

```javascript
db.managers.updateOne({ index: 999 }, { $set: { name: 'Doctor Strange' } })
```

### updateMany

```javascript
db.managers.updateMany({ rank: 'CEO' }, { $inc: { age: 10 } })
```

### Upsert

Upsert = พยายาม Update Document ถ้าไม่มีให้ทำการ Insert

```javascript
db.managers.updateOne(
  { name: 'Jolly Decks' },
  {
    $set: {
      name: 'Jolly Deck',
      age: 99,
      field: 'Procurement Engineering',
      rank: 'C99',
    },
  },
  { upsert: true }
)
```

## Delete

```javascript
db.managers.deleteMany({ name: 'Jonathan' })
db.managers.findOneAndDelete({ name: 'Jolly Deck' })
```

## Index

ตรวจสอบ Execution Stats ด้วยคำสั่ง **สังเกตเทคนิคในการค้นหา COLSCAN และ จำนวน Documents ที่ค้นหา**

```javascript
db.managers.find({ name: 'Kassandra' }).explain('executionStats')
```

สร้าง Index

```javascript
db.managers.createIndex({ name: 1 })
```

แสดง Index

```javascript
db.managers.getIndexes()
```

แสดง Execution Stats อีกครั้งหลังสร้าง Index **ดูเทคนิคในการค้นหาและจำนวน Documents**

```javascript
db.managers.find({ name: 'Kassandra' }).explain('executionStats')
```

## Aggregation

คล้ายกับ SQL ดูวิธีการเพิ่มเติมที่ [Aggregation Pipeline](https://www.mongodb.com/docs/manual/core/aggregation-pipeline/)

```javascript
db.managers.aggregate([
  {
    $bucket: {
      groupBy: '$age',
      boundaries: [0, 20, 30, 40, 50, 60],
      default: '61+',
      output: {
        count: { $sum: 1 },
      },
    },
  },
])
```

ตัวอย่างเพิ่มเติม

```javascript
db.managers.aggregate([
  {
    $match: { rank: 'CEO' },
  },
  {
    $group: { field: '$field', total: { $sum: 1 } },
  },
])
```
