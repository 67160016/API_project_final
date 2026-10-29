# Manus — Project Phase 3
รหัสนิสิต 67160016 | Pno7 | Final Integration & Project Defense

Manus เป็นเว็บจัดการพนักงาน แผนก ตำแหน่ง โปรเจกต์ และการมอบหมายงาน อยู่ในหน้าเดียว ใช้โทนดำ–ม่วงตามแบบที่เลือก ชุดส่งนี้ใช้ API จริงและเพิ่ม Workload Microservice ผ่าน HTTP แยก process

## เปิดงานบน Windows
ต้องมี Node.js 24 ขึ้นไป และใช้อินเทอร์เน็ตในการติดตั้งครั้งแรก เปิด Terminal ในโฟลเดอร์นี้:

    npm.cmd ci
    npm.cmd run demo

หรือดับเบิลคลิก start-demo.cmd ซึ่งติดตั้ง package เมื่อยังไม่มีแล้วเปิด demo ให้

- เว็บไซต์: http://localhost:3000/
- Swagger UI: http://localhost:3000/api-docs/
- Main API health: http://localhost:3000/health
- สรุปงานจาก microservice: http://localhost:3000/insights/workload
- Workload service health: http://localhost:4001/health
- เอกสาร microservice: services/workload/openapi.yaml และ http://localhost:4001/openapi.json

demo เปิด API และ microservice เป็นคนละ process ใช้ SQLite ที่ data/demo.sqlite ข้อมูลยังอยู่เมื่อปิดแล้วเปิดใหม่ ให้เปิด Terminal ค้างไว้ กด Ctrl+C เพื่อหยุด

หน้าเว็บชุดนี้จะรอข้อมูลจาก API หากเชื่อมไม่ได้จะแสดง loading/error และปุ่มลองใหม่ ไม่มีการแทนที่ด้วยข้อมูล localStorage การสาธิตการเชื่อม API ให้ใช้ localhost:3000 จากชุดนี้ ลิงก์ Sites ที่ทำก่อนหน้านี้เป็นตัวอย่างดีไซน์และโหมดทดลองแยกจากฐานข้อมูลในชุดส่ง

## ใช้ MySQL
ออกแบบ schema สำหรับ MySQL ตาม Phase 1 ใช้ฐานเดิมต่อได้ หากยังไม่มีให้สร้างด้วย schema.sql และ seed.sql ตาม docs/phase2-readme.md
คัดลอก .env.example เป็น .env ตั้งค่า DB_HOST, DB_PORT, DB_USER, DB_PASSWORD, DB_NAME แล้วรัน:

    npm.cmd start

start ใช้ MySQL เป็นค่าเริ่มต้นและเปิด microservice ด้วย ยังไม่ได้ทดสอบกับ MySQL server จริงในสภาพแวดล้อมที่จัดชุดส่ง ผลทดสอบที่แนบใช้ SQLite

## สถาปัตยกรรม
Browser frontend → Express API :3000 → db.js → SQLite หรือ MySQL
Express API → HTTP → Workload Microservice :4001
Express API → HTTP → Open-Meteo (ข้อมูลอากาศสำนักงาน)

microservice รับเฉพาะรหัสและวันที่โปรเจกต์เพื่อคำนวณจำนวนพนักงานที่มี/ไม่มีโปรเจกต์ที่กำลังดำเนินการและจำนวนการมอบหมายแต่ละแผนก ไม่รับชื่อและอีเมลพนักงาน ไม่มีฐานข้อมูลซ้ำใน microservice

โค้ด microservice และ tests ส่วนนี้เพิ่มใหม่ใน Phase 3 เนื่องจากผู้จัดทำยังไม่มีไฟล์งานสัปดาห์ 13 และ test suite สัปดาห์ 9/13 แยกมาให้ เอกสารไม่ได้อ้างว่าเป็นชุดงานเดิมจากสัปดาห์เหล่านั้น หากชั้นเรียนกำหนด API เฉพาะของสัปดาห์ 13 ต้องเทียบกับโจทย์นั้นเพิ่มเติม

## สิ่งที่หน้าเว็บเรียกจริง
- GET /health, /departments, /positions, /employees, /projects
- GET /employees/{id}/projects
- POST /employees, PUT /employees/{id}, DELETE /employees/{id}
- CRUD ของแผนก ตำแหน่ง และโปรเจกต์
- POST /employee-projects และ DELETE /employee-projects/{employeeId}/{projectId}
- GET /departments/{id}/weather
- GET /insights/workload

จึงครอบคลุมมากกว่า 3 endpoint และมีทั้ง GET และการเปลี่ยนแปลงข้อมูล
ค้นหาและกรองรายชื่อที่โหลดมาแล้วในเบราว์เซอร์ ส่วน API รองรับ GET /employees?search=... เพิ่มด้วย

## Microservices และ failure
API หลักมี 28 operations ใน openapi.yaml ซึ่งสร้างจาก contracts.js ที่ระบบใช้จริง
Workload Microservice มี GET /health และ POST /analyze เอกสารแยกที่ services/workload/openapi.yaml ใช้ server port 4001

Workload client: timeout 2000 ms, retry เพิ่ม 2 ครั้ง, backoff 100/200 ms
Open-Meteo: timeout 2000 ms, retry เพิ่ม 2 ครั้ง, backoff 250/500 ms

เมื่อบริการย่อยใช้งานไม่ได้ API ยังตอบคำขอได้ด้วย available=false และ summary=null หรือ weather=null พร้อมข้อความผู้ใช้ ไม่มีการแต่งข้อมูลสรุปหรืออากาศขึ้นมา หน้าเว็บยังจัดการข้อมูลหลักต่อได้

## เอกสาร API และ Postman
สร้างเอกสารใหม่:

    npm.cmd run docs:generate

คำสั่งนี้สร้าง openapi.json, openapi.yaml และ services/workload/openapi.yaml จาก runtime contracts
Import postman_collection.json แล้วตั้ง baseUrl=http://localhost:3000 และ workloadBaseUrl=http://localhost:4001
รันโฟลเดอร์ Endpoints และ Phase 3 Microservice หลังเปิด npm.cmd run demo
โฟลเดอร์จำลอง failure ต้องรัน tests เพื่อเปิด provider จำลองตามที่ระบุใน failure-scenario-results.md

## ผลทดสอบและ coverage
    npm.cmd test
    npm.cmd run coverage

coverage-report.md มีเปอร์เซ็นต์และ uncovered lines ที่วัดด้วย Node.js V8 จริง พร้อมข้อจำกัด
- ชุด API เดิมที่ต่อยอด: 118 HTTP requests ครอบคลุม 28 main operations และ 9 weather provider scenarios
- ชุด Phase 3: microservice calculation, validation, failure isolation และการเปิดแยก process
- Frontend: import JavaScript ตัวจริง เรียก API/SQLite ผ่าน HTTP จริง ครอบคลุม GET, POST, PUT, DELETE, assignment, validation, filter และ reconnect error
- Frontend tests ใช้ DOM test double จึงยังไม่ยืนยันภาพ responsive หรือการคลิกผ่าน browser จริง

หลักฐาน: test-results.json, phase3-test-results.json, frontend-api-evidence.json, coverage/coverage-output.txt
ผล Open-Meteo สดจาก Phase 2 ที่ได้รับมาเก็บแยกใน docs/phase2-live-test-results.json; หากต้องการตรวจใหม่ใช้ npm.cmd run test:live

## โครงสร้างส่ง
src/ — API และ client ของบริการภายนอก
frontend/ — Manus หน้าเดียวที่ใช้ API จริง
services/workload/ — HTTP microservice แยก process และเอกสารของบริการ
db.js — ตัวเชื่อมฐานข้อมูลหลัก; src/db.js เป็น compatibility wrapper
scripts/ — ตัวเปิดระบบ สร้าง OpenAPI และวัด coverage
test/ — ชุดทดสอบ
postman_collection.json / openapi.yaml / coverage-report.md / failure-scenario-results.md
requirement.md / erd.png / schema.sql / seed.sql / package.json

## เตรียมนำเสนอ 5–7 นาที
อ่าน DEFENSE-GUIDE.md สำหรับ demo 3–4 นาที และคำถาม 2–3 นาที
ระบบยังไม่มี authentication/authorization ภายในแอปตามขอบเขตเดิม
