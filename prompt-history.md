# 📜 Prompt History — Gacha Drop Rate Simulator

เอกสารบันทึก **Prompt ทั้งหมด** ที่ใช้สื่อสารกับ AI (Claude Code) ตั้งแต่เริ่มต้นจนจบ
แต่ละ prompt ระบุ **ใช้ทำอะไร** + **เหตุผลที่ออกแบบแบบนี้** + **ข้อความเต็ม** เพื่อให้เห็นกระบวนการคิดและการ prompt engineering

- **โปรเจกต์:** เว็บจำลองกาชา (Gacha Drop Rate Simulator) — single HTML file
- **เครื่องมือ:** Claude Code (CLI) + plugin `superpowers`
- **โมเดล:** Claude Opus 4.8 (effort: high)
- **ผลลัพธ์:** `gacha-simulator.html`

> 💡 หมายเหตุการอ่าน: หัวข้อ `##` แต่ละอันคือ 1 prompt — เทียบได้กับ "เซลล์หัวสีฟ้า" ในเทมเพลต Sheets

---

## ⚙️ ส่วนที่ 0 — Setup / Config

**ใช้ทำอะไร:** เตรียมสภาพแวดล้อมก่อนเริ่มงานจริง — ติดตั้ง plugin, ตั้ง working directory, เลือกโมเดลและระดับความละเอียด
**เหตุผลที่ออกแบบแบบนี้:** เลือก Opus 4.8 + effort high เพราะงานต้องเขียนโค้ดยาวไฟล์เดียวและต้องการความถูกต้องของ logic การคำนวณ; ติดตั้ง `superpowers` เพื่อใช้ flow brainstorming → plan → implement

```text
/plugin install superpowers@claude-plugins-official
/reload-plugins
cd C:\Users\koomo\Documents\internship\open
/model claude-opus-4-8
/effort high
```

---

## 🟦 Prompt #1 — Project Spec (เริ่มโปรเจกต์ด้วยสเปกเต็ม)

**ใช้ทำอะไร:** ส่ง requirement ทั้งหมดของแอปในครั้งเดียว ผ่าน `/superpowers:brainstorming` เพื่อให้ AI ออกแบบ logic + UI + การคำนวณ แล้วสร้างทั้งแอปจาก requirement ชุดเดียว
**เหตุผลที่ออกแบบแบบนี้:** จัด requirement เป็น **ตาราง 3 ระดับ (Must / Advanced / Bonus)** + ใส่ **ค่าเริ่มต้นชัดเจน** (SSR 1%, SR 9%, R 30%, N 60%) + **expected behavior รายข้อ** + **ผัง UI** + **ข้อจำกัดเทคนิค** (vanilla JS, ไฟล์เดียว, CSV ผ่าน Blob, ห้าม hardcode) — ทำให้ AI ไม่ต้องเดา ลด scope creep และได้ผลลัพธ์ตรงสเปทที่วัดผลได้

```text
สร้างเว็บจำลองกาชา (Gacha Drop Rate Simulator) แบบไฟล์ HTML เดียว ที่รันได้ทันทีในเบราว์เซอร์
โดยใช้ AI ช่วยออกแบบ logic, UI และการคำนวณทั้งหมด

Requirement | สิ่งที่ต้องทำ | สถานะ
Rate Setting | ตั้งค่า Rate ของ SSR / SR / R / N ได้ และ Rate รวมต้องเท่ากับ 100% | Must Have
Item Pool Management | เพิ่มรายชื่อ Item/ตัวละคร เลือก Rarity ได้ มีปุ่ม + Add Item และลบ/แก้ไขได้ | Must Have
Single Simulation | กรอกจำนวนครั้งสุ่มและราคาต่อครั้ง แล้วสุ่มผลลัพธ์ตาม Rate จริง | Must Have
Result Display | แสดง Summary, Table, Item Result, ค่าใช้จ่าย และ Rarity ที่ออกมากที่สุด | Must Have
Export CSV | Export ผลลัพธ์ล่าสุดและ Player POV Summary เป็น CSV ได้ | Must Have
Validation | ตรวจ rate, input, item pool และค่าที่ผิดชัดเจน | Must Have
Player POV Simulator | กรอกงบเติมเงิน แล้วจำลอง Monte Carlo เพื่อดูโอกาสได้ SSR/Item | Advanced Requirement
Free Roll Rule | ตั้งค่าสุ่มครบ X ครั้ง ได้สุ่มฟรี Y ครั้ง โดยนับจาก paid rolls เท่านั้น | Advanced Requirement
Pity / History / Chart | ระบบ pity, history, chart, insight เพิ่มคะแนน | Bonus

หมวด | ระดับ | Requirement | รายละเอียด / Expected Behavior
Rate Setting | Must | มีค่าเริ่มต้น SSR 1%, SR 9%, R 30%, N 60% | ผู้ใช้แก้ไข rate ได้, rate รวมต้องเท่ากับ 100%, ห้ามค่าติดลบ
Item Pool | Must | เพิ่ม item ได้หลายรายการ | มี input ชื่อ item, dropdown rarity, ปุ่ม + Add Item, list/table item ที่เพิ่มแล้ว, แก้ไข/ลบได้
Item Pool Validation | Must | ถ้า rarity ใดมี rate > 0 ต้องมี item อย่างน้อย 1 ชิ้น | ป้องกันการสุ่มแล้วไม่เจอ item ใน pool
Single Simulation Input | Must | กรอกจำนวนครั้งที่สุ่มและราคาต่อครั้ง | จำนวนครั้งต้อง > 0, ราคาต่อครั้งต้อง >= 0
Single Simulation Logic | Must | สุ่ม rarity ก่อนตาม rate แล้วค่อยสุ่ม item ใน rarity นั้น | ใช้ Math.random() หรือ logic เทียบเท่า ไม่ fix ผลลัพธ์
Single Simulation Output | Must | แสดงจำนวน SSR/SR/R/N, ค่าใช้จ่ายรวม, rarity ที่ออกมากที่สุด | มีตารางสรุป rate ที่ตั้งไว้ vs สัดส่วนจริง
Item Result Output | Must | แสดงรายชื่อ item ที่สุ่มได้ | อาจแสดงเป็นรายการทั้งหมดหรือ summary count ต่อ item
CSV Export | Must | Export Latest Pull Result CSV | CSV ควรมี roll no., rarity, item name, cost type หรือข้อมูลสำคัญของผลลัพธ์
Player POV Input | Advanced | กรอกงบเติมเงิน, ราคาต่อ roll, จำนวน simulation | เช่น budget 3,000 บาท, price 30 บาท, simulation 1,000 รอบ
Free Roll Rule | Advanced | ตั้งค่าสุ่มครบ X ครั้ง ได้สุ่มฟรี Y ครั้ง | free rolls คำนวณจาก paid rolls เท่านั้น และ free rolls ไม่สร้าง free rolls ซ้ำ
Player POV Logic | Advanced | จำลองหลายรอบแบบ Monte Carlo | แต่ละรอบใช้ total rolls = paid rolls + free rolls แล้วนับผล SSR/SR/R/N
Player POV Output | Advanced | แสดง chance ได้ SSR อย่างน้อย 1, chance ได้ 0 SSR, average count | มี best/worst SSR result และ insight เป็นภาษาคน
Player POV CSV | Advanced | Export Player POV Summary CSV | Export ค่า budget, paid rolls, free rolls, total rolls, simulation count, probability, averages
Insight Text | Bonus | ระบบสรุปผลแบบอ่านง่าย | เช่น เติม 3,000 บาท มีโอกาสได้ SSR อย่างน้อย 1 ประมาณ 67%
Chart / Visual | Bonus | มีกราฟแท่งหรือ visual summary | ไม่จำเป็นต้องใช้ library; ใช้ CSS bar ก็ได้
History | Bonus | เก็บประวัติ simulation ล่าสุด | แสดง 5-10 รอบล่าสุด และ reset ได้

(UI Layout)
Header | Title / Subtitle | ชื่อเว็บ เช่น Gacha Drop Rate Simulator พร้อมคำอธิบายสั้น ๆ
Rate Settings | Rarity Rate Inputs | input สำหรับ SSR/SR/R/N พร้อมแสดง rate รวมว่าครบ 100% หรือไม่
Item Pool | Add Item Form | กรอกชื่อ item, เลือก rarity, กด + Add Item
Item Pool | Item Table | แสดง item ที่เพิ่มแล้ว พร้อมปุ่ม edit/delete
Single Simulation | Roll Settings | จำนวนครั้งที่สุ่ม, ราคาต่อครั้ง, ปุ่ม Start Simulation
Single Simulation | Result Cards | จำนวน roll, total cost, SSR count, most common rarity
Single Simulation | Result Table | rarity, rate setting, count, actual percentage
Single Simulation | Item Result | รายการ item ที่ได้ หรือ summary count ต่อ item
Player POV | Budget Inputs | งบเติมเงิน, ราคาต่อ roll, จำนวน simulation
Player POV | Free Roll Rule | toggle เปิด/ปิด, paid rolls required, free rolls granted
Player POV | Monte Carlo Summary | paid rolls, free rolls, total rolls, chance >= 1 SSR, chance 0 SSR, averages
CSV | Export Buttons | Export Latest Pull Result CSV และ Export Player POV Summary CSV
UX | Validation Message | แจ้งเตือน input ผิด เช่น rate ไม่ครบ 100%, item pool ว่าง
UX | Responsive Layout | เปิดบนมือถือหรือจอเล็กแล้วยังอ่านรู้เรื่อง

(Tech Constraints)
Frontend | HTML, CSS, JavaScript ในไฟล์เดียว | ไม่ใช้ React/Next.js/Vue ที่ต้อง build | ใช้ vanilla JS เป็นหลัก
CSV Export | Blob / URL.createObjectURL / download attribute | ไม่ส่งข้อมูลไป server | ต้อง export จาก browser โดยตรง
Random Logic | Math.random() หรือ Web Crypto ถ้าต้องการ | ไม่ Hardcode result | สุ่มหลายรอบแล้วผลควรเปลี่ยน
File | gacha-simulator.html | ไม่แยกหลายไฟล์ css/js/assets | ไฟล์เดียวจบ
Browser | Chrome/Edge/Safari รุ่นปัจจุบัน | | double click เปิดไฟล์แล้วควรใช้ได้
```

**ผลลัพธ์จาก AI:** สำรวจโฟลเดอร์ → เริ่มร่างสถาปัตยกรรมแอปตามสเปก

---

## 🟦 Prompt #2 — Action Directive (สั่งลงมือทำเลย)

**ใช้ทำอะไร:** ตัด process ที่ไม่จำเป็น (การถามกลับ / การกู้ไฟล์ที่ถูกลบ) ให้ AI ลงมือสร้างจริงทันที
**เหตุผลที่ออกแบบแบบนี้:** เมื่อสเปกชัดครบแล้ว ผู้ใช้ต้องการความเร็ว — สั่งสั้น กระชับ เพื่อข้าม ceremony และโฟกัสที่ deliverable

```text
ทำตามที่วางimplementมาเลย
```

**ผลลัพธ์จาก AI:**
- สร้าง `gacha-simulator.html` ครบทุกฟีเจอร์ (Rate Settings, Item Pool, Single Simulation, Player POV/Monte Carlo, Pity, History, CSV Export, Validation, Responsive)
- **พิสูจน์ logic ด้วย Node.js:** rate ลู่เข้า 1/9/30/60 · Pity การันตีเป๊ะทุก X ครั้ง · สูตร free roll (฿3000÷฿30 → 100 paid + 10 free = 110) · Monte Carlo ได้ ~67% ตรงตัวอย่างในสเปก

---

## 🟦 Prompt #3 — Main Rubric (ส่งเกณฑ์ให้คะแนนหลัก เพื่อ audit)

**ใช้ทำอะไร:** ส่งเกณฑ์การให้คะแนน (110 คะแนน) เพื่อให้ AI ตรวจสอบว่าผลงานครอบคลุมทุกข้อหรือยัง (rubric-driven QA)
**เหตุผลที่ออกแบบแบบนี้:** ใช้เกณฑ์จริงเป็น checklist ทำให้การตรวจวัดได้และอุดช่องโหว่ก่อนส่ง — เป็นเทคนิค "ให้ AI ตรวจงานตัวเองด้วยเกณฑ์ของผู้ตรวจ"

```text
หมวด | รายละเอียด | คะแนน
File Requirement (ไฟล์ที่ส่ง) | ส่งเป็น HTML ไฟล์เดียว เปิดใน browser ได้ทันที ไม่ต้อง backend/build | 10
Rate Setting (ตั้งค่าอัตรา) | ตั้งค่า SSR/SR/R/N ได้, rate รวมต้อง 100%, validation rate ถูกต้อง | 12
Item Pool Management (จัดการ Item Pool) | เพิ่ม item, เลือก rarity, แสดงรายการ, แก้ไข/ลบ, validate pool | 15
Single Simulation Logic (ตรรกะการสุ่ม) | สุ่ม rarity ตาม rate จริง แล้วสุ่ม item จาก pool ไม่ hardcode | 18
Single Simulation Result (ผลลัพธ์การสุ่ม) | แสดง summary, table, item result, actual percentage, total cost | 12
CSV Export (ส่งออก CSV) | export latest pull result และข้อมูลเปิดอ่านได้จริง | 10
Player POV Simulator (จำลองฝั่งผู้เล่น) | คำนวณ budget → paid rolls/free rolls/total rolls และ run Monte Carlo | 15
Player POV Output (ผลลัพธ์ฝั่งผู้เล่น) | แสดง chance ≥1 SSR, chance 0 SSR, averages, best/worst, insight | 10
Validation & Error Handling (ตรวจสอบ Input) | ป้องกัน input ผิด เช่น ค่าติดลบ, simulation เกิน limit, pool ว่าง | 5
UI / UX (หน้าตาและการใช้งาน) | อ่านง่าย ไม่รก ใช้งานจริงได้ และ responsive พื้นฐาน | 3
```

**ผลลัพธ์จาก AI:** audit เทียบทุกข้อ → ยืนยันครบ 110/110 พร้อมชี้หลักฐานในโค้ด + เสนอจุด optional 2 อย่าง

---

## 🟦 Prompt #4 — Bonus Rubric (ส่งเกณฑ์คะแนนโบนัส)

**ใช้ทำอะไร:** ส่งเกณฑ์โบนัส (20 คะแนน) ให้ AI ตรวจซ้ำในส่วน Pity / History / Chart / Advanced CSV / Polish
**เหตุผลที่ออกแบบแบบนี้:** แยกโบนัสออกจากเกณฑ์หลัก เพื่อโฟกัสการยกระดับเฉพาะส่วนที่ได้คะแนนเพิ่ม และหาช่องที่ยัง "ดี" แต่ยังไม่ "ดีมาก"

```text
หมวด | รายละเอียด | คะแนนเพิ่ม
Pity System (ระบบ Pity) | มีระบบ pity เปิด/ปิดได้ และทำงานถูกต้องร่วมกับ simulation | 5
History (ประวัติการสุ่ม) | แสดงประวัติการสุ่มหลายรอบล่าสุด | 4
Chart / Visual (กราฟและภาพสรุป) | มีกราฟหรือ visual summary ที่อ่านง่าย | 4
Advanced CSV (ส่งออก CSV เพิ่มเติม) | export Player POV Summary CSV ครบถ้วน | 4
Polish (ความละเอียดของ UI) | UI ดีมาก มี micro-interaction/empty state/คำแนะนำชัด | 3
```

**ผลลัพธ์จาก AI:** audit โบนัส → ครอบคลุม ~19–20/20 (รวม ~130/130) + ระบุว่า Polish (3 คะแนน) ยกระดับได้ด้วย inline edit / empty state / toast

---

## 🟦 Prompt #5 — Approve Polish (อนุมัติให้ปรับปรุง)

**ใช้ทำอะไร:** อนุมัติให้ AI ลงมือทำ 3 จุด Polish ที่เสนอไว้
**เหตุผลที่ออกแบบแบบนี้:** สั่งสั้นเพราะรายละเอียดถูกตกลงไว้ในข้อความก่อนหน้าแล้ว — ใช้บริบทต่อเนื่อง ไม่ต้องเขียนซ้ำ

```text
ทำเลยที่แนะนำเลย
```

**ผลลัพธ์จาก AI:**
- **Inline edit** ในตาราง item (แทน `prompt()`) + Enter/Esc + ตรวจชื่อซ้ำ
- **Empty state** ให้ผลลัพธ์ Single Sim และ Player POV
- **Toast notification** (success/error/info) ทุกปุ่มสำคัญ + helper `fail()` แสดง error ทั้ง inline + toast
- ตรวจ: JS syntax ผ่าน · ไม่มีโค้ดเก่าหลงเหลือ · DOM reference ครบทุกจุด

---

## 🟦 Prompt #6 — Prompt History (ขอเอกสารฉบับนี้)

**ใช้ทำอะไร:** ขอให้รวบรวม prompt ทั้งหมดของ session เป็นไฟล์เอกสาร (deliverable บังคับ)
**เหตุผลที่ออกแบบแบบนี้:** ต้องการหลักฐานกระบวนการใช้ AI ตั้งแต่ต้นจนจบ ในรูปแบบที่อ่านเข้าใจง่ายและตรวจสอบได้

```text
Prompt History ทั้งหมด
```

(ต่อมา) ขอให้จัดรูปแบบเอกสารตามเทมเพลตตัวอย่าง: **หัวข้อ + ใช้ทำอะไร + เหตุผลที่ออกแบบแบบนี้ + ข้อความเต็ม**

**ผลลัพธ์จาก AI:** จัดทำเอกสารฉบับนี้ (`prompt-history.md`) ตามเทมเพลตที่กำหนด

---

## 📊 สรุปภาพรวม (Timeline)

| # | Prompt | ประเภท | ผลลัพธ์ |
|---|---|---|---|
| 0 | Setup/Config | ตั้งค่า | plugin + Opus 4.8 + effort high |
| 1 | Project Spec | สเปกเต็ม | กำหนด requirement ทั้งหมด |
| 2 | "just do what i ask" | directive | สร้าง `gacha-simulator.html` + verify logic |
| 3 | Main Rubric | QA | audit 110/110 |
| 4 | Bonus Rubric | QA | audit โบนัส ~20/20 |
| 5 | "ทำเลย" | directive | inline edit + empty state + toast |
| 6 | Prompt History | เอกสาร | เอกสารฉบับนี้ |

**ปรัชญาการใช้ AI:** ผู้ใช้ให้ **สเปกละเอียดครบตั้งแต่ต้น** → ใช้ **เกณฑ์ให้คะแนนเป็นตัวขับการปรับปรุง (rubric-driven)** → AI ออกแบบ logic + UI + การคำนวณ และ **พิสูจน์ความถูกต้องด้วยการรันทดสอบจริง** ในทุกขั้นสำคัญ
