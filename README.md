# ระบบเวรพยาบาลตรวจการ (Supervisor)

แอปบันทึก/แสดงข้อมูลเวรพยาบาลตรวจการ สถาบันบำราศนราดูร — ใช้ Google Sheets เป็นฐานข้อมูล, Google Apps Script เป็น backend, hosted ได้ทั้ง GAS Web App และ GitHub Pages

---

## โครงสร้างระบบ

```
[Google Sheet] ⇆ [Apps Script Web App] ⇄ [index.html (GAS หรือ GitHub Pages)]
```

- **Apps Script (Code.gs)** = backend ทำหน้าที่อ่าน/เขียน Sheet + อัปโหลดรูปเข้า Drive
- **index.html** = หน้าเว็บ — เปิดได้จาก 2 ที่:
  1. URL ของ GAS Web App ตรงๆ (เร็ว เปิดได้ทันที)
  2. GitHub Pages (ใช้ index.html เดียวกัน เรียก API กลับมาที่ GAS ผ่าน JSONP/FormData)

---

## ขั้นตอน Deploy

### 1) Deploy Apps Script Web App (จำเป็นต้องทำก่อน)

1. เปิด Google Sheet ของระบบ → เมนู **Extensions → Apps Script**
2. วางโค้ดจาก `Code.gs` ลงใน `Code.gs`
3. กดไอคอน **+** ข้างคำว่า Files → **HTML** → ตั้งชื่อ `index` → วางโค้ดจาก `index.html`
4. กด **Save** (รูปดิสเก็ต)
5. เมนู **Deploy → New deployment**
   - ⚙ → **Web app**
   - Execute as: **Me**
   - Who has access: **Anyone**
6. กด **Deploy** → ครั้งแรกจะให้ Authorize → Allow
7. คัดลอก **Web app URL** ไว้ (รูปแบบ `https://script.google.com/macros/s/AKfycb.../exec`)

> **ใช้ URL นี้เปิดแอปได้ทันที** — ระบบทำงานเต็มที่ตั้งแต่ตรงนี้

---

### 2) (เลือกได้) วาง index.html บน GitHub Pages

ถ้าอยากมี backup ที่ host แยกบน GitHub

1. เปิดไฟล์ `index.html` แก้ค่าตัวแปร `GAS_WEB_APP_URL` ที่ด้านบนสุดของ `<script>`:
   ```js
   const GAS_WEB_APP_URL = 'https://script.google.com/macros/s/AKfycb.../exec';
   ```
   วาง URL ที่ได้จากขั้น 1)

2. สร้าง GitHub repo (ส่วนตัวหรือสาธารณะก็ได้)
3. Upload ไฟล์ `index.html` ไปไว้ใน **root** ของ repo (อย่าใส่ใน subfolder)
4. ไป **Settings → Pages**
   - Source: **Deploy from a branch**
   - Branch: **main** / **(root)**
   - กด Save
5. รอ 1–2 นาที → จะได้ URL ประมาณ `https://<username>.github.io/<repo-name>/`

> **ห้ามอัปโหลด `Code.gs` ไปด้วย** — มันรันบน GAS เท่านั้น GitHub Pages รันไม่ได้

---

## วิธีอัปเดทโค้ดภายหลัง

### แก้ Apps Script (Code.gs / index.html ฝั่ง GAS)

1. แก้โค้ดใน Apps Script editor → **Save**
2. Deploy → **Manage deployments** → กดไอคอน ✏ ดินสอที่ deployment ของคุณ → **Version: New version** → Deploy
3. URL เดิมใช้ได้ต่อ ไม่ต้องเปลี่ยน

### แก้ index.html ที่ GitHub Pages

แก้ใน GitHub → commit → GitHub Pages จะอัปเดทอัตโนมัติใน 1–2 นาที

---

## Troubleshooting

| อาการ | สาเหตุ / วิธีแก้ |
|---|---|
| เปิด GitHub Pages เห็นแค่คำว่า "Supervisor" | ไม่มีไฟล์ `index.html` ที่ root → render `README.md` แทน (ที่มีหัวข้อ `# Supervisor`) ให้ upload `index.html` |
| "ข้อผิดพลาดการเชื่อมต่อ ไม่สามารถเข้าถึง Google Apps Script" | ไม่ได้ตั้งค่า `GAS_WEB_APP_URL` ใน `index.html` หรือ URL ผิด |
| API call timeout | GAS Web App ยังไม่ deploy หรือยังไม่ Authorize / สิทธิ์ไม่ใช่ "Anyone" |
| รูปอัปโหลดไม่ขึ้น | โฟลเดอร์ Drive ID ใน `Code.gs` (`DRIVE_FOLDER_ID`) ไม่ถูกต้องหรือไม่มีสิทธิ์เข้าถึง |
| แก้ Code.gs แล้วไม่เห็นผล | ลืม **Deploy → Manage deployments → New version** (เซฟอย่างเดียวไม่พอ) |

---

## ข้อมูลโครงสร้างชีต (ย่อ)

| ชีต | หน้าที่ |
|---|---|
| `Data` | ข้อมูลหลักรายเวร (date + shift + Sup + doctors) |
| `IPD` | ลูกเวร IPD แยกตามตึก (ยอดผู้ป่วย, ออกซิเจน, ช่วยเหลือ) |
| `OPD` | ลูกเวร OPD แยกตามห้องตรวจ |
| `สรุป` | aggregated by formula — ใช้สำหรับ Dashboard |
| `Dashboard` | aggregated by formula — ใช้สำหรับ Dashboard (alternative) |
| `HR` | รายชื่อบุคลากร (Sup, RN, PN, NA, พนักงาน) |
| `Sup` | พยาบาลตรวจการ |
| `Doc` | รายชื่อแพทย์ (MED/ER/PED/ENT/SUR/ORTHO/EYE/URO/INTERN) |
| `แผนก` | รายชื่อตึก/แผนก + ประเภท IPD/OPD + เบอร์โทร |

---

## License

ใช้งานภายในสถาบันบำราศนราดูร — ไม่อนุญาตให้นำไปใช้เชิงพาณิชย์
