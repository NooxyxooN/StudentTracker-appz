# ระบบติดตามงานนักเรียน (เว็บแอป + Firebase)

เว็บแอปเดี่ยว (`index.html`) เชื่อมต่อฐานข้อมูล Firebase Firestore แบบเรียลไทม์
เปิดใช้งานได้จากเบราว์เซอร์บนมือถือ โน้ตบุ๊ค หรือไอแพด — ข้อมูลซิงก์กันทันทีทุกอุปกรณ์
ออกแบบไว้สำหรับ **ครูคนเดียวใช้งาน ไม่มีระบบล็อกอิน**

## โครงสร้างข้อมูล (Firestore)

```
classes/{classId}
  - name

classes/{classId}/students/{studentId}
  - no, prefix, name, surname

classes/{classId}/assignments/{assignmentId}
  - name, order

classes/{classId}/submissions/{studentId_assignmentId}
  - sent (true/false), checked (true/false), score (number/null)
```

## ขั้นตอนติดตั้ง (ทำครั้งเดียว)

### 1. สร้างโปรเจกต์ Firebase
1. ไปที่ https://console.firebase.google.com แล้วล็อกอินด้วย Google account
2. กด "Add project" ตั้งชื่อโปรเจกต์ เช่น `student-tracker` แล้วกด Create

### 2. เปิดใช้งาน Firestore
1. ในเมนูซ้ายเลือก **Build > Firestore Database**
2. กด **Create database**
3. เลือกโหมด **Start in production mode** (จะใช้ไฟล์ `firestore.rules` ที่แนบมาแทน)
4. เลือก location ที่ใกล้ (เช่น `asia-southeast1`)

### 3. สร้าง Web App เพื่อขอ config
1. หน้าโปรเจกต์ (Project Overview) กดไอคอน **</>** (Web)
2. ตั้งชื่อแอป แล้วกด "Register app" (ไม่ต้องติ๊ก Firebase Hosting ก็ได้)
3. คัดลอกค่าที่ได้ในช่อง `firebaseConfig` เช่น

```js
const firebaseConfig = {
  apiKey: "AIzaSy...",
  authDomain: "student-tracker.firebaseapp.com",
  projectId: "student-tracker",
  storageBucket: "student-tracker.appspot.com",
  messagingSenderId: "123456789",
  appId: "1:123456789:web:abcdef"
};
```

### 4. วางค่าใน index.html
เปิดไฟล์ `index.html` ด้วยโปรแกรมแก้ไขข้อความ (เช่น Notepad, TextEdit, VS Code)
หาบรรทัด `const firebaseConfig = { ... }` ใกล้ด้านบนของสคริปต์ แล้ววางค่าที่คัดลอกมาแทนที่

### 5. ตั้งค่าความปลอดภัย (Firestore Rules)
1. กลับไปที่ Firebase Console > Firestore Database > แท็บ **Rules**
2. คัดลอกเนื้อหาทั้งหมดจากไฟล์ `firestore.rules` ที่แนบมา วางแทนของเดิม แล้วกด **Publish**

> กติกานี้เปิดให้ใครก็ตามที่มีลิงก์เว็บแอปเข้าถึงข้อมูลได้ (เหมาะกับใช้คนเดียว/ไม่แชร์ลิงก์)
> ถ้าต้องการความปลอดภัยสูงขึ้นในอนาคต ดูหัวข้อ "อัปเกรดเป็นระบบล็อกอิน" ด้านล่าง

### 6. เปิดใช้งาน
**วิธีง่ายสุด (ทดสอบในเครื่อง):** ดับเบิลคลิกเปิดไฟล์ `index.html` ด้วยเบราว์เซอร์ได้เลย

**วิธีใช้งานจริงข้ามอุปกรณ์ (แนะนำ):** เผยแพร่ขึ้น Firebase Hosting เพื่อให้ได้ลิงก์เว็บ เช่น
`https://student-tracker.web.app` เปิดจากมือถือ/ไอแพด/โน้ตบุ๊คได้ทันที

```bash
npm install -g firebase-tools     # ติดตั้งครั้งเดียว
firebase login                    # ล็อกอินด้วย Google account เดียวกับ Firebase Console
cd webapp                         # เข้าโฟลเดอร์นี้
firebase use --add                # เลือกโปรเจกต์ที่สร้างไว้
firebase deploy                   # เผยแพร่ขึ้นเว็บ
```

หลัง deploy เสร็จจะได้ลิงก์เว็บมาใช้งานได้เลย บันทึกไว้เป็น Bookmark หรือ Add to Home Screen
บนมือถือ/ไอแพดเพื่อให้เปิดได้เหมือนแอป

## วิธีใช้งานแอป

- **+ ห้องเรียนใหม่** — สร้างห้องเรียน (คล้ายแท็บชีทในเวอร์ชัน Excel เดิม)
- **+ เพิ่มนักเรียน** — พิมพ์ คำนำหน้า ชื่อ นามสกุล
- **+ เพิ่มงาน** — เพิ่มคอลัมน์งานใหม่ พร้อม checkbox ส่งแล้ว/ตรวจแล้ว และช่องคะแนน
- ติ๊ก checkbox หรือกรอกคะแนนได้ทันที ระบบบันทึกอัตโนมัติและซิงก์ไปทุกอุปกรณ์ที่เปิดหน้านี้อยู่
- คอลัมน์ขวาสุดคำนวณอัตโนมัติ: คะแนนรวม (เฉพาะงานที่ตรวจแล้ว), จำนวนงานที่ส่ง, ที่ตรวจแล้ว, และที่ยังค้างส่ง
- กด ✕ ข้างชื่องาน/ชื่อนักเรียนเพื่อลบ (ข้อมูลที่เกี่ยวข้องจะถูกลบตามไปด้วย)

## อัปเกรดเป็นระบบล็อกอิน (ถ้าต้องการในอนาคต)

ถ้าภายหลังมีครูหลายคนใช้งาน หรืออยากล็อกไม่ให้คนอื่นแก้ข้อมูลได้:
1. เปิดใช้งาน **Firebase Authentication** (เช่น ล็อกอินด้วย Email/Password หรือ Google)
2. แก้ `firestore.rules` ให้ตรวจสอบ `request.auth != null` แทน `if true`
3. เพิ่มหน้าล็อกอินในเว็บแอป (แจ้งได้ ถ้าต้องการให้ช่วยทำส่วนนี้เพิ่ม)

## ค่าใช้จ่าย

Firebase มีแพ็กเกจฟรี (Spark Plan) ซึ่งเพียงพอสำหรับการใช้งานระดับห้องเรียน/โรงเรียนขนาดเล็ก
(อ่านโควตาปัจจุบันได้ที่ https://firebase.google.com/pricing)
