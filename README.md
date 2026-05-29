# 🎴 Gacha Drop Rate Simulator

เว็บแอปพลิเคชันไฟล์เดียว (Single-page HTML) สำหรับจำลองการสุ่มกาชา วิเคราะห์อัตราดรอปจริง (Actual Drop Rate) และคำนวณโอกาสฝั่งผู้เล่นด้วย **Monte Carlo Simulation** พร้อมระบบการันตี (Pity), ประวัติการสุ่ม และการส่งออกรายงาน CSV

> รันได้ทันทีในเบราว์เซอร์ — ไม่ต้องติดตั้ง, ไม่ต้อง build, ไม่ต้องต่อ backend

---

## 🚀 ฟีเจอร์หลัก

| ฟีเจอร์ | รายละเอียด |
|---|---|
| 📊 **Rate Settings** | ตั้งค่าอัตราสุ่ม SSR / SR / R / N (ค่าเริ่มต้น 1% / 9% / 30% / 60%) พร้อมตรวจผลรวม = 100% และกันค่าติดลบแบบเรียลไทม์ |
| 📦 **Item Pool** | เพิ่ม / แก้ไข (inline) / ลบ ไอเทมรายระดับ + validation: ระดับที่มีอัตรา > 0% ต้องมีไอเทมอย่างน้อย 1 ชิ้น |
| 🎲 **Single Simulation** | สุ่มสูงสุด 100,000 ครั้ง (สุ่ม rarity ตามอัตราจริง → สุ่ม item ใน rarity นั้น) แสดง summary, ตาราง Set vs Actual, กราฟแท่ง CSS และสรุปไอเทม |
| 👤 **Player POV (Monte Carlo)** | ป้อนงบเติมเงิน → คำนวณ paid/free/total rolls → จำลองหลายรอบ แสดงโอกาสได้ SSR ≥1, โอกาส 0 SSR, ค่าเฉลี่ย, best/worst และ insight ภาษาคน |
| 🎟️ **Pity System** | Hard Pity เปิด/ปิดได้ — การันตี SSR เมื่อสุ่มครบ X ครั้ง ทำงานทั้ง Single & Monte Carlo |
| 📜 **History** | เก็บ 10 รอบล่าสุด พร้อม sparkline, Pity badge และคลิกเพื่อเรียกผลลัพธ์กลับมาดู (click-to-restore) |
| 📤 **CSV Export** | ส่งออก Latest Pull Results และ Player POV Summary — เข้ารหัส UTF-8 BOM อ่านภาษาไทยใน Excel/Sheets ได้ทันที |

---

## 📖 วิธีใช้งาน

1. ดาวน์โหลด / clone repo นี้
2. **ดับเบิลคลิก** `gacha-simulator.html` เพื่อเปิดในเบราว์เซอร์ (Chrome / Edge / Safari / Firefox)
3. ใช้งานได้ทันที — ไม่ต้องติดตั้งอะไรเพิ่ม

```bash
git clone https://github.com/momozxmo/Gachaexam.git
cd Gachaexam
# เปิด gacha-simulator.html ในเบราว์เซอร์
```

---

## 🛠️ เทคโนโลยีที่ใช้

- **HTML5 + Vanilla CSS3 + Vanilla JavaScript (ES6+)** ในไฟล์เดียว
- **Zero dependencies** — ไม่ใช้ไลบรารี/เฟรมเวิร์กภายนอก รันออฟไลน์ได้
- การสุ่มใช้ `Math.random()` (ไม่ hardcode ผล), CSV export ผ่าน `Blob` + `URL.createObjectURL`
- ธีมมืด (Glassmorphism) + Responsive รองรับมือถือ + micro-interaction/toast/empty-state
- ฟอนต์ `Plus Jakarta Sans` (Google Fonts) พร้อม fallback ฟอนต์ระบบ

---

## ✅ การตรวจสอบความถูกต้อง (Verified Logic)

logic หลักผ่านการทดสอบด้วย Node.js แล้ว:

- การกระจายอัตราลู่เข้า **1 / 9 / 30 / 60%** (จาก 500,000 ครั้ง)
- Pity การันตี SSR **เป๊ะทุก X ครั้ง**
- สูตร Free Roll: งบ ฿3,000 ÷ ฿30 → **100 paid + 10 free = 110 total**
- Monte Carlo: โอกาสได้ SSR ≥1 ที่ 110 rolls @1% ≈ **67%** (ตรงทฤษฎี `1 − 0.99¹¹⁰`)

---

## 📁 ไฟล์ในโปรเจกต์

| ไฟล์ | คำอธิบาย |
|---|---|
| [`gacha-simulator.html`](gacha-simulator.html) | ตัวเว็บแอป (ไฟล์เดียวจบ) |
| [`prompt-history.md`](prompt-history.md) | บันทึก Prompt ทั้งหมดที่ใช้กับ AI ตั้งแต่ต้นจนจบ |
| [`short-reflection.md`](short-reflection.md) | สรุปการเรียนรู้ระหว่างทำโปรเจกต์ |

---

<sub>สร้างด้วยความช่วยเหลือจาก AI (Claude Code · Opus 4.8) — ออกแบบ logic, UI และการคำนวณ พร้อมพิสูจน์ความถูกต้องด้วยการรันทดสอบจริง</sub>
