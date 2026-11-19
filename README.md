# SPU Canteen Guide — เชื่อม LINE OA Bot (Chef Bot)

ในโฟลเดอร์นี้มี
- `index.html` — หน้าเว็บ (ปุ่ม LINE ของทุกร้านส่งข้อความเข้า OA ได้จริงแล้ว)
- `Code.gs` — โค้ดบอท สำหรับวางใน Google Apps Script

ภาพรวมการทำงาน

```
ผู้ใช้พิมพ์ใน LINE ──► LINE Platform ──(Webhook)──► Google Apps Script (Code.gs)
                                                     │ วิเคราะห์คำถาม + ค้นข้อมูล 20 ร้าน
ผู้ใช้เห็นคำตอบ + การ์ดร้าน ◄── LINE Reply API ◄────┘
```

---

## ขั้นที่ 1 — สร้าง LINE OA + เปิด Messaging API
1. เข้า https://manager.line.biz → **สร้างบัญชี LINE Official Account** (ฟรี) ตั้งชื่อ เช่น `SPU Canteen Guide`
2. ใน OA Manager ไปที่ **Settings (ตั้งค่า) → Messaging API → Enable Messaging API**
   - ระบบจะให้เลือก/สร้าง **Provider** (ตั้งชื่ออะไรก็ได้ เช่น `SPU Canteen Project`)
3. จด **Basic ID** ของ OA ไว้ (หน้าตาแบบ `@123abcde`) — ใช้ในขั้นที่ 5

## ขั้นที่ 2 — เอา Channel Access Token
1. เข้า https://developers.line.biz/console → เลือก Provider → เลือก Channel ของ OA
2. แท็บ **Messaging API** → เลื่อนลงล่างสุด → **Channel access token (long-lived) → Issue**
3. คัดลอก token เก็บไว้ (**อย่าแชร์/อย่าใส่ในหน้าเว็บ** ใครได้ไปจะส่งข้อความในนามบอทได้)

## ขั้นที่ 3 — วางโค้ดบอทใน Google Apps Script
1. เข้า https://script.google.com → **New project** → ตั้งชื่อ `SPU Canteen Bot`
2. ลบโค้ดเดิมใน `Code.gs` แล้ว **วางโค้ดทั้งหมดจากไฟล์ `Code.gs`** → กด 💾 Save
3. ไปที่ ⚙️ **Project Settings → Script Properties → Add script property** เพิ่ม 2 ค่า

   | Property | Value |
   |---|---|
   | `LINE_CHANNEL_ACCESS_TOKEN` | token จากขั้นที่ 2 |
   | `WEB_URL` | ลิงก์หน้าเว็บของคุณ (ไม่ใส่ก็ได้ ปุ่ม "ดูบนเว็บ" บนการ์ดจะไม่แสดง) |

4. (ทดสอบก่อน deploy) กลับไปหน้า Editor → เลือกฟังก์ชัน `testBot` → **Run** → ดูคำตอบใน **Execution log**
   - ครั้งแรกจะขอสิทธิ์ → Review permissions → เลือกบัญชี → Advanced → Go to project (unsafe) → Allow

## ขั้นที่ 4 — Deploy เป็น Web App แล้วผูก Webhook
1. มุมขวาบน **Deploy → New deployment** → ⚙️ เลือกประเภท **Web app**
   - Execute as: **Me**
   - Who has access: **Anyone** ← สำคัญ ไม่งั้น LINE เรียกไม่ได้
2. กด Deploy → คัดลอก **Web app URL** (ลงท้าย `/exec`)
3. กลับไป LINE Developers Console → แท็บ **Messaging API**
   - **Webhook URL** → วาง URL → Update
   - เปิด **Use webhook** = ON
4. ใน **LINE OA Manager → Settings → Response settings (การตอบกลับ)**
   - **Webhook = ON**
   - **Auto-reply messages (ข้อความตอบกลับอัตโนมัติ) = OFF** (ไม่งั้นจะตอบซ้อนกับบอท)
   - Greeting message ปิดได้ เพราะบอทมีข้อความต้อนรับตอนแอดเพื่อนอยู่แล้ว
5. สแกน QR ในหน้า Messaging API เพื่อแอดบอท แล้วลองพิมพ์ "งบ 50 บาทกินอะไรดี" 🎉

> ⚠️ ปุ่ม **Verify** ใน LINE Developers อาจขึ้น error `302 Found` เพราะ Apps Script redirect คำขอเป็นปกติ
> **ไม่ต้องสนใจ** ให้ทดสอบโดยพิมพ์ในแชทจริงแทน ถ้าบอทตอบได้แปลว่าใช้งานได้

> 🔁 **แก้โค้ดทีหลัง** ต้องไปที่ Deploy → Manage deployments → ✏️ Edit → Version: **New version** → Deploy
> (URL เดิมยังใช้ได้ ไม่ต้องเปลี่ยน Webhook)

## ขั้นที่ 5 — ผูกหน้าเว็บเข้ากับ OA
1. เปิด `index.html` ค้นหา `const LINE_OA_ID = '@spucanteen';` → เปลี่ยนเป็น Basic ID ของคุณ
2. อัปโหลดเว็บขึ้นโฮสต์ฟรี เช่น **GitHub Pages** หรือ **Netlify Drop** (ลากไฟล์ไปวางที่ https://app.netlify.com/drop)
3. เอาลิงก์เว็บไปใส่ใน Script Property `WEB_URL` (ขั้นที่ 3)

เมื่อกดปุ่ม LINE บนการ์ดร้าน → แอป LINE จะเปิดแชท OA พร้อมข้อความ เช่น
"สวัสดีค่ะ ขอทราบข้อมูลร้านครัวคุณแม่ตามสั่ง ล็อก H-01 หน่อยค่ะ" → กดส่ง → บอทตอบข้อมูลร้านนั้นทันที
(ถ้ายังไม่ได้เป็นเพื่อนกับ OA ระบบจะให้แอดก่อน)

---

## บอทตอบอะไรได้บ้าง (ดึงจากข้อมูลชุดเดียวกับเว็บ)
| ผู้ใช้พิมพ์ | บอทตอบ |
|---|---|
| โรงเย็นมีร้านตามสั่งไหม | บอกตรง ๆ ว่าโรงเย็นไม่มี แล้วแนะนำ **ครัวคุณแม่ตามสั่ง (โรงร้อน H-01)** |
| งบ 50 บาทกินอะไรดี | รายชื่อร้านที่ราคาเริ่มต้นไม่เกิน 50 บาท เรียงจากถูกสุด + การ์ดรูป |
| ร้านก๋วยเตี๋ยวโรงร้อนอยู่ตรงไหน | **เจ๊ B ก๋วยเตี๋ยวเรือต้มยำ — ล็อก H-04 (กลางโรงร้อน)** |
| ชาไทยราคาเท่าไหร่ | ชาไทย & เบเกอรี่โฮมเมด 35 - 55 บาท |
| ข้าวซอยเปิดกี่โมง | 08:00 - 16:30 น. |
| H-12 / ข้าวขาหมูอยู่ไหน | ข้อมูลร้านตามรหัสล็อก/ชื่อร้าน/เมนู |
| สุ่มร้านให้หน่อย / หิว | สุ่ม 3 ร้าน |
| โรงร้อน / โรงเย็น / เครื่องดื่ม | กรองตามโซน/หมวด |

ทุกคำตอบมี **การ์ด Flex Message ธีมดำ-ทอง** (รูป ราคา เมนูเด็ด ล็อก เวลาเปิด) และปุ่ม **Quick Reply** ให้กดถามต่อ

## เพิ่ม/แก้ร้าน
แก้ที่อาร์เรย์ `STALLS` ใน `Code.gs` (และ `stallsData` ใน `index.html` ให้ตรงกัน)
- `tags` = หมวดอาหาร (ต้องตรงกับชื่อหมวดใน `CATEGORY_WORDS`)
- `keywords` = คำที่ผู้ใช้อาจพิมพ์ถึงร้านนี้ (ชื่อเล่นร้าน เมนูเด่น)
แก้เสร็จแล้วต้อง Deploy เป็น **New version** ทุกครั้ง

## (ทางเลือก) Rich Menu
ทำปุ่มเมนูด้านล่างแชทได้ใน **OA Manager → Home → Rich menus** โดยไม่ต้องเขียนโค้ด
ตั้ง Action แบบ **Text** เช่น `ร้านโรงร้อน`, `ร้านโรงเย็น`, `งบ 50 บาทกินอะไรดี`, `สุ่มร้านให้หน่อย` และแบบ **Link** ไปที่หน้าเว็บ

## ข้อจำกัดที่ควรรู้
- Apps Script อ่าน header `x-line-signature` ไม่ได้ จึงไม่ได้ตรวจลายเซ็น webhook ซึ่งพอใช้ได้สำหรับโปรเจกต์เรียน
  ถ้าจะใช้งานจริงจัง ควรย้ายไปเป็น Node.js + `@line/bot-sdk` (เช่นบน Render/Vercel/Cloud Functions) ซึ่งตรวจลายเซ็นได้
- แพ็กเกจ LINE OA ฟรี: **Reply message ส่งได้ไม่จำกัด** (บอทนี้ใช้ reply อย่างเดียว) ส่วน Push/Broadcast จะมีโควตาต่อเดือน
- ราคาที่บอทตอบเป็น "ช่วงราคาร้าน" ตามที่มีในเว็บ ถ้าอยากให้ตอบราคารายเมนู ต้องเพิ่มข้อมูลราคาแต่ละเมนูก่อน
