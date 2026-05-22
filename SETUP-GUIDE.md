# Marketing Audit + Lead Dashboard — Setup Guide

คู่มือติดตั้งระบบเก็บข้อมูล Lead สำหรับ DG Media House

## 📁 ไฟล์ในชุด

1. **DGMediaHouse-Marketing-Audit.html** — แบบทดสอบ Marketing Audit (วางบนเว็บไซต์)
2. **DGMediaHouse-Lead-Dashboard.html** — Dashboard ดูข้อมูล Lead (เปิดในเครื่องคุณ)
3. **SETUP-GUIDE.md** — ไฟล์นี้

---

## 🚀 Quick Start (5 นาที)

### ขั้นที่ 1: เปิดดู Dashboard ก่อน
ดับเบิ้ลคลิกเปิดไฟล์ `DGMediaHouse-Lead-Dashboard.html` ในเบราว์เซอร์
จะเห็นว่ายังไม่มีข้อมูล — ปกติค่ะ เพราะยังไม่มีใครทำแบบทดสอบ

### ขั้นที่ 2: ทดสอบแบบทดสอบ
เปิดไฟล์ `DGMediaHouse-Marketing-Audit.html` แล้วทำตั้งแต่ต้นจนจบ
กรอกชื่อ-อีเมลเพื่อทดสอบ

### ขั้นที่ 3: กลับมาเปิด Dashboard
จะเห็นข้อมูลที่เพิ่งกรอกปรากฏใน Dashboard แล้ว
✅ ระบบทำงานแล้ว!

---

## ☁️ ต่อ Google Sheets (สำหรับ Production)

เมื่อจะใช้จริงบนเว็บไซต์ ต้องส่งข้อมูลไป Google Sheets เพื่อ:
- ดูข้อมูลจากเครื่องไหนก็ได้ (ไม่ผูกกับ browser เดียว)
- ทีมงานหลายคนเข้าถึงได้
- ไม่หายเวลาเคลียร์ cache

### Step 1: สร้าง Google Sheet
1. เข้า [sheets.google.com](https://sheets.google.com) สร้างสเปรดชีตใหม่
2. ตั้งชื่อ เช่น "DGMH Marketing Audit Leads"
3. ใน Row 1 ใส่ headers ดังนี้ (copy-paste ได้):

```
Timestamp	Name	Brand	Email	Phone	Score	Tier	Anonymous	Referrer	Q1_Category	Q1_Answer	Q1_Score	Q2_Category	Q2_Answer	Q2_Score	Q3_Category	Q3_Answer	Q3_Score	Q4_Category	Q4_Answer	Q4_Score	Q5_Category	Q5_Answer	Q5_Score	Q6_Category	Q6_Answer	Q6_Score	Q7_Category	Q7_Answer	Q7_Score	Q8_Category	Q8_Answer	Q8_Score	Q9_Category	Q9_Answer	Q9_Score	Q10_Category	Q10_Answer	Q10_Score
```

### Step 2: เพิ่ม Apps Script
1. ใน Google Sheet เมนู **Extensions → Apps Script**
2. ลบโค้ดเดิมทิ้ง วางโค้ดนี้แทน:

```javascript
function doPost(e) {
  try {
    const data = JSON.parse(e.postData.contents);
    const sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();

    const row = [
      data.timestamp_th || data.timestamp,
      data.name || '',
      data.brand || '',
      data.email || '',
      data.phone || '',
      data.score || 0,
      data.tier || '',
      data.anonymous ? 'Yes' : 'No',
      data.referrer || ''
    ];

    // เพิ่มคำตอบ 10 ข้อ
    const details = data.details || [];
    for (let i = 0; i < 10; i++) {
      const d = details[i] || {};
      row.push(d.category || '', d.answer || '', d.score || '');
    }

    sheet.appendRow(row);

    return ContentService.createTextOutput(JSON.stringify({ status: 'ok' }))
      .setMimeType(ContentService.MimeType.JSON);
  } catch (err) {
    return ContentService.createTextOutput(JSON.stringify({ status: 'error', message: err.toString() }))
      .setMimeType(ContentService.MimeType.JSON);
  }
}

function doGet() {
  return ContentService.createTextOutput('Marketing Audit Webhook is running');
}
```

3. กด **Save** (💾 icon) ตั้งชื่อ project เช่น "MarketingAuditWebhook"

### Step 3: Deploy เป็น Web App
1. มุมบนขวา กด **Deploy → New deployment**
2. กดเฟือง ⚙️ ข้างคำว่า "Select type" → เลือก **Web app**
3. ตั้งค่าดังนี้:
   - **Description**: Marketing Audit Webhook v1
   - **Execute as**: Me (อีเมลของคุณ)
   - **Who has access**: **Anyone** ⚠️ สำคัญมาก (ต้องเป็น Anyone เพื่อให้เว็บไซต์ส่งข้อมูลเข้ามาได้)
4. กด **Deploy**
5. กด **Authorize access** → เลือก Google account → กด **Advanced → Go to ... (unsafe)** → กด Allow
6. **คัดลอก Web app URL** ที่ได้ (จะขึ้นต้นด้วย `https://script.google.com/macros/s/.../exec`)

### Step 4: ใส่ URL ในไฟล์ HTML
1. เปิด `DGMediaHouse-Marketing-Audit.html` ด้วย text editor (เช่น Notepad, VS Code)
2. หาบรรทัด (อยู่ประมาณกลางไฟล์):
   ```javascript
   const SHEETS_WEBHOOK_URL = '';
   ```
3. ใส่ URL ที่ copy ไว้:
   ```javascript
   const SHEETS_WEBHOOK_URL = 'https://script.google.com/macros/s/XXXXXX/exec';
   ```
4. Save ไฟล์

### Step 5: ทดสอบ
1. เปิดไฟล์ HTML ที่แก้แล้ว
2. ทำแบบทดสอบจบ + กรอกข้อมูล
3. เปิด Google Sheet ดู — ควรเห็นข้อมูลปรากฏใน Row 2 ภายใน 2-3 วินาที
✅ เสร็จเรียบร้อย!

---

## 📊 เชื่อม Dashboard กับ Google Sheet

### Step 1: Publish Sheet เป็น CSV
1. ใน Google Sheet เมนู **File → Share → Publish to web**
2. ตั้งค่า:
   - **Link**: Entire Document
   - **Format**: Comma-separated values (.csv)
3. กด **Publish** → ติ๊ก Confirm
4. **คัดลอก URL** ที่ได้ (จะขึ้นต้นด้วย `https://docs.google.com/spreadsheets/d/.../pub?output=csv`)

### Step 2: ใส่ URL ใน Dashboard
1. เปิด `DGMediaHouse-Lead-Dashboard.html`
2. วาง URL ในช่องด้านบนสุด (Setup banner สีทอง)
3. กดปุ่ม **บันทึก**
4. กดปุ่ม **↻ Sync จาก Google Sheet** เพื่อดึงข้อมูลล่าสุด

ตอนนี้ทุกครั้งที่เปิด Dashboard ระบบจะดึงข้อมูลล่าสุดจาก Sheet ให้อัตโนมัติ

---

## 🌐 เผยแพร่บนเว็บไซต์

วิธีง่ายที่สุดสำหรับใช้จริง:

### ตัวเลือก A: อัปโหลดไป Web Hosting ที่มีอยู่
- ถ้ามี Wordpress: ใช้ plugin "Insert HTML Snippet" หรือ upload เป็น page ผ่าน FTP
- ถ้ามี Wix/Squarespace: ใช้ Embed HTML block
- Upload ไฟล์ `DGMediaHouse-Marketing-Audit.html` ไปยัง path ที่ต้องการ เช่น `yoursite.com/audit/`

### ตัวเลือก B: Netlify (ฟรี + ง่ายสุด)
1. ไปที่ [netlify.com](https://netlify.com) สมัครฟรี
2. เมนู **Sites → Add new site → Deploy manually**
3. ลากไฟล์ `DGMediaHouse-Marketing-Audit.html` ลงไป
4. ได้ URL แบบ `your-audit.netlify.app` มาใช้งานทันที
5. ผูก domain ของตัวเองก็ได้

### ตัวเลือก C: ใส่ใน LINE OA / Email Marketing
- ลิงก์ URL ตรงไปยังหน้าที่ host ไว้

---

## 🔒 Privacy & Security Notes

- ข้อมูลที่เก็บทั้งหมด: ชื่อ, แบรนด์, อีเมล, เบอร์/LINE, คำตอบ 10 ข้อ, คะแนน, เวลาที่ทำ, browser info, referrer
- ข้อมูลถูกเก็บใน:
  1. **localStorage** ในเครื่อง user (จะถูกล้างเมื่อ user clear cache)
  2. **Google Sheets** ของคุณ (ถ้าเซ็ตค่า webhook)
- แนะนำเพิ่ม Privacy Notice ที่หน้าแรกถ้าใช้ใน production
- ปฏิบัติตาม PDPA: ไม่ส่งข้อมูล Lead ไปให้ third party โดยไม่ขออนุญาต

---

## 🎨 การปรับแต่งเพิ่มเติม

### เปลี่ยนโทนสี
ใน HTML แก้ที่ `:root` variables:
```css
--gold: #d4af37;     /* สีหลัก */
--bg-primary: #0a0a0f; /* พื้นหลัง */
```

### เปลี่ยนคำถาม
ใน Quiz HTML หา `const questions = [...]` แล้วแก้ตามต้องการ
ต้องคงโครงสร้าง:
```js
{
  category: "หมวด",
  pill: "01 · ชื่อหัวข้อ",
  title: "คำถาม?",
  sub: "คำอธิบาย",
  options: [
    { text: "ตัวเลือก 1", score: 10 },  // คะแนนสูง = ระบบดี
    { text: "ตัวเลือก 2", score: 6 },
    { text: "ตัวเลือก 3", score: 3 },
    { text: "ตัวเลือก 4", score: 0 }
  ]
}
```

### เปลี่ยน CTA ปลายทาง
หา function `bookConsult()` แล้วเปลี่ยนปลายทาง เช่น:
- เปลี่ยนเป็น Calendly: `window.open('https://calendly.com/your-link', '_blank')`
- เปลี่ยนเป็น LINE OA: `window.location.href = 'https://lin.ee/xxx'`

---

## ❓ Troubleshooting

**Q: ข้อมูลไม่ขึ้นใน Google Sheet**
- ตรวจสอบว่า "Who has access" ใน Deploy เป็น **Anyone**
- เปิด Apps Script → **Executions** ดู error log
- ตรวจสอบ URL ใน HTML ว่าครบ ลงท้ายด้วย `/exec`

**Q: Dashboard ดึง CSV ไม่ได้**
- ตรวจสอบว่า Sheet ทำ "Publish to web" แล้ว
- URL ต้องลงท้ายด้วย `output=csv`
- ลอง paste URL ใน browser ดูว่าโหลด CSV จริงไหม

**Q: ข้อมูลใน Dashboard หาย**
- ปกติ — เพราะ localStorage จะหายเมื่อ clear browser cache
- ป้องกัน: ใช้ Google Sheets sync (ข้อมูลใน Sheet ไม่หาย)
- หรือใช้ปุ่ม "ส่งออก CSV" สำรองข้อมูลเป็นระยะ

---

## 📞 Support

ติดต่อทีม DG Media House: dg.marketing@dgmediahouse.co.th

Built for DG Media House · Data-Driven Marketing Agency
