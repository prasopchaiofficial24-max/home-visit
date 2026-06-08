# ระบบเยี่ยมบ้านนักเรียน — โรงเรียนจิกดู่วิทยา

## ภาพรวมโปรเจกต์

เว็บแอป Single-page HTML สำหรับบันทึกและสรุปการเยี่ยมบ้านนักเรียน  
สังกัด สพม.อุบลราชธานี อำนาจเจริญ

- **ไฟล์หลัก:** `index.html` (ไฟล์เดียว ~1,900+ บรรทัด)
- **Deploy:** GitHub Pages
- **Database:** Firebase Firestore (collection: `studentVisits`)
- **Offline fallback:** localStorage (`sv_bk` = records, `sv_cfg` = config)

---

## ข้อมูลโรงเรียน (hardcode ใน JS)

```js
const SCHOOL    = 'โรงเรียนจิกดู่วิทยา';
const AMPHOE    = 'หัวตะพาน';
const PROVINCE  = 'อำนาจเจริญ';
const GRADES    = ['ม.1','ม.2','ม.3','ม.4','ม.5','ม.6'];
const COL       = 'studentVisits';
const PRINCIPAL = 'นายโกวิท สุพรรณ์';
```

## Firebase Config

```js
const FIREBASE_CONFIG = {
  apiKey:            "AIzaSyDXJIhFdFT8_9Uz5iyxvo0Tze5sYHrI4iM",
  authDomain:        "school-student-home-visit.firebaseapp.com",
  projectId:         "school-student-home-visit",
  storageBucket:     "school-student-home-visit.firebasestorage.app",
  messagingSenderId: "639982697383",
  appId:             "1:639982697383:web:55488f484c1f80a4bbd482"
};
```

---

## รายชื่อครูและชั้น (dropdown login)

| ชั้น | ครู |
|------|-----|
| ม.1 | นางสาววันทนีย์ ไชยมงคล, นางสาวไอรินทร์ อัครศักดิ์ศรี, นางสาวอนัญญา หามทอง |
| ม.2 | นางสาวพรอุมา จิตตัง, นางสาวอรวรรณ สุดาชม |
| ม.3 | นายเทียนชัย ทาเงิน, นายณัฐภัทร สังข์ขาว, นางสาวอรอิริยา ศิริวาลย์ |
| ม.4 | นายฉัตรชัย สนิทชัย, นายวสันต์ สานัดถ์ |
| ม.5 | นายประสพชัย มหาวงศ์, นางสาวจิราภรณ์ เชิงหอม |
| ม.6 | นายมงคล แสงย้อย, นายปฏิรูป เธียรประมุข |

---

## โครงสร้าง Pages (5 หน้า)

| Page ID | Tab | ฟังก์ชันหลัก |
|---------|-----|------------|
| `page-dashboard` | 📊 แดชบอร์ด | `renderDash()` |
| `page-form` | ➕ กรอกข้อมูล | `submitForm()`, `getFormData()` |
| `page-records` | 📋 รายชื่อ | `renderRecords()`, `exportExcel()` |
| `page-district` | 📄 รายงานเขต | `renderDistrict()`, `aggByGrade()` |
| `page-present` | 🎯 นำเสนอ | `renderPresent()` |

---

## Data Model (Firestore document)

```js
{
  id, studentName, studentNo, studentPhone,
  gender,          // 'male' | 'female'
  gradeLevel,      // 'ม.1'...'ม.6'
  teacher,
  fatherName, fatherJob, fatherPhone,
  motherName, motherJob, motherPhone,
  parStatus,       // 'together'|'separate'|'divorced'|'father_dead'|'mother_dead'|'both_dead'
  parentName, parentRel, parentJob, parentPhone,
  address,
  needHelp,        // string[]
  parentConcern,
  householdSize, income, allowance,
  burden,          // string[] — 'disabled'|'elderly'|'single_par'|'unemployed'
  houseType,       // 'own'|'rent'|'other'
  visitDate, year, semester,
  visited,         // boolean
  nvReasons,       // string[]
  rH,rS,rV,rT,rX,rP,rSf,rO,  // risk flags (0|1)
  pL,pH,pE,pS,pD,pO,          // problem flags (0|1)
  famBg,           // string[] — 'both_dead'|'one_dead'|'divorced'|'not_with_par'|'half_sib'
  family,          // 'close'|'conflict_some'|'conflict_often'|'distant'|'viol_some'|'viol_often'|'other'
  house,           // string[] — 'good'|'old'|'material'|'no_toilet'|'bad_area'
  transport,       // 'parent_drive'|'motorbike'|'bus'|'school_bus'|'bicycle'|'walk'|'other'
  distance, travelTime,
  urgent,          // boolean
  notes,
  school, amphoe, province,
  ts               // timestamp
}
```

---

## Key Functions

### Firebase / Data
- `initFirebase()` — init Firestore
- `load()` — โหลดจาก Firestore หรือ localStorage fallback
- `upsert(data)` — save/update record
- `del(id)` — ลบ record

### Form
- `getFormData()` — อ่านค่าจากฟอร์มทั้งหมด
- `setFormData(r)` — ใส่ค่าเข้าฟอร์ม (ตอน edit)
- `clearForm()` — ล้างฟอร์ม
- `submitForm()` — validate + save
- `selPill(groupId, val)` — เลือก radio pill
- `togCheck(el, event)` — toggle checkbox pill (**ต้องส่ง event เสมอ** ไม่งั้น checkbox กดไม่ติด)
- `togUrgent(event)` — toggle urgent pill

### Reports
- `aggByGrade()` — รวมสถิติรายชั้น (returns array พร้อม parStatus, burden counts)
- `renderDistrict()` — สร้างรายงานเขต ข้อ 1–23
- `renderPresent()` — สร้างหน้านำเสนอ
- `printStudent(id)` — เปิด modal พิมพ์รายนักเรียน
- `exportExcel()` — export xlsx 2 sheets

### Report Metadata (localStorage `rpt_meta`)
- `saveRptMeta()` / `loadRptMeta()` — บันทึก/โหลด metadata ระดับรายงาน
- Fields: `round, reporter, reporterPos, reporterPhone, orgs[], otherOrg, urgentDesc, helpedCount, usage, parentConcern, issues, comments`

---

## CSS Variables (theme ขาว-แดง)

```css
--pri:   #c8102e;   /* แดงหลัก */
--pri-d: #9b0a23;   /* แดงเข้ม */
--pri-l: #fde8ec;   /* ชมพูอ่อน (hover/bg) */
--sec:   #16a34a;   /* เขียว (success) */
--dan:   #dc2626;   /* แดง danger */
--warn:  #ea580c;   /* ส้ม warning */
--bg:    #f7f7f8;   /* พื้นหลัง */
--card:  #fff;
--bdr:   #e8e8ea;
--txt:   #18181b;
--muted: #71717a;
```

---

## Print CSS

- `@page { size: A4 portrait; margin: 2cm 2.5cm; }`
- พิมพ์เฉพาะ `#page-district` — ซ่อนทุก `.page` อื่น
- Font: `TH SarabunPSK` → `Sarabun` → `Arial`
- ฟอร์มรายนักเรียน: ใช้ `body.printing-student` class (set/remove ก่อน/หลัง print)

---

## สิ่งที่ยังค้างอยู่ (Pending)

1. **Google Drive link รูปภาพ** — รอ link โฟลเดอร์จากงานวิชาการ  
   เมื่อได้ link ให้เพิ่มในส่วน ข้อ 22 ของ `renderDistrict()` และใน section 7 ของฟอร์ม

2. **Firebase Security Rules** — ตอนนี้เปิด public (`allow read, write: if true`)  
   ควรจำกัดถ้า deploy จริง

---

## Known Issues / Notes

- **togCheck ต้องส่ง event** — `onclick="togCheck(this,event)"` เสมอ ถ้าไม่ส่ง checkbox state จะถูก browser toggle ซ้ำแล้วผิด
- **ไฟล์ยาวมาก** — Edit tool อาจตัดไฟล์กลางทาง ควรใช้ Python append แทนถ้าจะต่อท้าย
- **Transport values** ใน HTML: `parent_drive, motorbike, bus, school_bus, bicycle, walk, other` (ต้องตรงกับ `transportLabel` ใน exportExcel และ chart labels ใน renderDash)

---

## External Libraries (CDN)

```html
firebase-app-compat.js    v10.7.0
firebase-firestore-compat.js v10.7.0
Chart.js                  v4.4.1
xlsx (SheetJS)            v0.18.5
```
