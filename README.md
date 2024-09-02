# CS360 HW 1 Summary
This is summary for CS360 Assignment 1 "Manual Cloud-based Web Deployment"
### Strapi
---
Strapi เป็น CMS(Content management system) ที่ไว้ใช้สร้าง Backend API
โดยมี Admin panel ที่ developer สามารถใช้ทดสอบและแก้ไข API ได้อย่างง่าย

โดยตัว strapi จะมี 3 ส่วนหลักๆคือ:
* Content type
* Entry
* Media Library

โดยจากความเข้าใจของผม Content type จะเป็นเหมือนกับ Table ใน database, Entry จะทำหน้าที่เป็น row ข้อมูล, และ Media Library เป็นจุดที่ใช้เก็บข้อมูล media ต่างๆที่ใช้ใน API

ตัวของ Content Type ก็จะแบ่งได้อีก 3 แบบคือ
* #### Collection types
    > กลุ่มของของข้อมูลที่ระบุข้อมูลหลายๆแบบไว้ โดยมีลักษณะคล้ายกับ Table ใน database โดยแต่ละชุดข้อมูลจะ๔ูกเก็บไว้ในรูปของ Entry
* #### Single Types 
    > เป็น collection types ที่มีเพียง Entry เดียวและไม่สามารถเพิ่มได้
* #### Components
    > กลุ่มของข้อมูลที่สามารถนำมาใช้ซ้ำได้ ทำหน้าที่เหมือนเป็น relation ใน database
### Ignore file
---
ignore file เป็นไฟล์ที่เอาไว้เก็บชื่อไฟล์ที่เราไม่ต้องการที่จะ commit ไปพร้อมกับตัวโปรเจ็ก ส่วนมากจะเอาไว้ ignore ไฟล์หรือโฟล์เดอร์ modules เช่น node_modules หรือไฟล์ที่ใช้เก็บ API KEYS เช่นไฟล์ .env

เนื้อหาใน ignore file (.gitignore) ของ repository นี้:
```
node_modules/*
.env
```

### วิธีที่ใช้ติดตั้ง
---
สร้าง strapi project ผ่าน npm อ้างอิงจาก [strapi quickstart guide](https://docs.strapi.io/dev-docs/quick-start#step-1-run-the-installation-script-and-create-a-strapi-cloud-account)
```
npx create-strapi-app@latest my-strapi-project --quickstart
```
หลังจากสร้างโปรเจ็กแล้วจะมีหน้า browser เปิดขึ้นมาและให้สร้าง admin account เพื่อใช้งาน strapi

โดยหากทำตามขั้นตอนใน guide ก็จะสรุปได้ว่าการสร้างโปรเจ็กสำเร็จแล้ว แต่ในภายหลังผมได้ลองรันตัวโปรเจ็กใหม่[ตามขั้นตอนใน Part B](https://docs.strapi.io/dev-docs/quick-start#-part-b-build-your-data-structure-with-the-content-type-builder) ของหน้า guide ซึ่งเป็นการเริ่มตัว app ขึ้นใหม่แต่เกิด error ขึ้นดังนี้
```
Error: Middleware "strapi::session": App keys are required. Please set app.keys in config/server.js (ex: keys: ['myKeyA', 'myKeyB'])...
```
ซึ่งหลังจากการ[ค้นหา Error นี้ใน google](https://forum.strapi.io/t/error-middleware-strapi-session-app-keys-are-required/24948/4) ก็พบว่าตัวโปรเจ็กต้องการ API keys ในการเริ่มใช้งานแต่ตัว key จะใช้เป็น string แบบไหนก็ได้ตามด้วย "=="

วิธีที่ผมใช้แก้ไข error นี้คือการสร้างไฟล์ .env ภายในโฟลเดอร์โปรเจ็กที่มีเนื้อหาดังนี้:
```
APP_KEYS=asdfskdlfjdsafjdsalfdskaf==
API_TOKEN_SALT=asdofdsafsadfasdf==
ADMIN_JWT_SECRET=asdfsadfdsafsafd==
```
###### _(All api keys above is literally me mashing keyboard 3 times. It provides no values what so ever, I think.)_
หลังจากสร้างไฟล์ตามข้างต้นเสร็จก็ทำการลองรันอีกครั้งด้วยคำสั่ง
```
npm run develop
```
ในรอบนี้ตัวโปรเจ็กสามารถรันได้สำเร็จและสามารถใช้งานหน้า admin ได้ตามปกติ โดยหลังจากรันในรอบนี้ตัวแอพจะมีการสร้าง key ขึ้นมาใหม่อีก 1 key ตามนี้:
```
APP_KEYS=asdfskdlfjdsafjdsalfdskaf==
API_TOKEN_SALT=asdofdsafsadfasdf==
ADMIN_JWT_SECRET=asdfsadfdsafsafd==
JWT_SECRET=xxxxxxxxxxxxxxxxxx==      <---------- this
```
### Deployment
---
_ตัว spec ของเครื่อง EC2 ที่ใช้ในส่วนนี้จะอ้างอิงจากสไลด์ในสัปดาห์ที่ 2 "Module 02 Cloud-based Deployment" โดยสาเหตุที่อ้างอิงตามนี้เนื่องจากตัว app มี minimum requirement ตรงกับ spec ของเครื่องในสไลด์และตัว app ไม่ต้องการทรัพยากรมากในการรัน tutorial guide_

### ขั้นตอน
1. clone repository เข้าเครื่อง EC2 ที่ใช้เป็น server
2. สร้างไฟล์ .env ในโฟลเดอร์โปรเจ็กบนเครื่อง EC2
โดยไฟล์ .env จะต้องมี credentials ดังนี้
```
APP_KEYS
API_TOKEN_SALT
ADMIN_JWT_SECRET
JWT_SECRET (ถูกสร้างโดยอัตโนมัติ)
```
3. เริ่ม strapi server ด้วยคำสั่ง
```
npm run start
```
4. ตรงหน้า AWS แก้ไข inbound rules ให้ยอมรับการเชื่อมต่อเข้ามาที่ port 1337 (default port ของ strapi application)
5. ทดสอบการเชื่อมต่อจากเครื่อง client โดยการเชื่อมไปที่ {IP ของเครื่อง EC2}:1337
---
:D