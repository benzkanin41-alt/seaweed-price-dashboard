# Seaweed price

เอกสารนี้เป็น build brief สำหรับส่งให้ Codex อีกเครื่องหนึ่งสร้าง dashboard ต้นทุนสาหร่ายย้อนหลังให้ได้เหมือนต้นฉบับ โดยเน้นมุมมองนักลงทุนสำหรับติดตามต้นทุนวัตถุดิบของ TKN จากข้อมูล Korea export price ไป Thailand

Final destination ของไฟล์ brief นี้:

```text
D:\OneDrive\stock\Valuation หุ้น\TKN\Seaweed price.md
```

## ลำดับคำสั่งและการแก้ไขที่ต้อง preserve

1. ผู้ใช้ต้องการ dashboard ต้นทุนสาหร่ายย้อนหลังจาก `Seaweed Export Price from KOREA ($/Ton)` รายเดือน รายไตรมาส รายปี พร้อมกราฟและจุดที่กดดูรายละเอียดได้
2. แหล่งข้อมูลหลักคือ Korea Customs Service Trade Statistics และไฟล์ workbook `Seaweed Price sep 68.xlsx`
3. ต้องใช้ HS Code `121221`, Country `Thailand`, Item/description `Fit for human consumption`
4. ต้องย้อนหลังตั้งแต่ปี 2558 หรือ `2015-01` ถึงเดือนล่าสุดที่ source คืนข้อมูลจริง
5. ผู้ใช้ขอให้เป็น local dashboard จึงต้องเปิดผ่าน local URL ได้ เช่น `http://127.0.0.1:8766/outputs/seaweed_cost_dashboard.html`
6. ผู้ใช้ขอให้เติมข้อมูลถึงเดือนล่าสุด จึงต้อง query live endpoint เพิ่มจาก workbook latest period
7. ผู้ใช้ขอให้กราฟ Growth รายเดือนมี filter เลือก `YoY / MoM / QoQ`
8. ผู้ใช้ตั้งข้อสังเกตสำคัญว่า metric ต้องสัมพันธ์กับ time grain:
   - รายเดือนไม่ควรมี `QoQ`
   - รายไตรมาสไม่ควรมี `MoM`
   - รายปีควรมีแค่ `YoY`
9. Dashboard ล่าสุดจึงต้องแก้เป็น:
   - Monthly growth filter: `MoM`, `YoY`
   - Quarterly growth filter: `QoQ`, `YoY`
   - Yearly growth filter: `YoY`
   - Detail panel และ table ต้องซ่อน metric ที่ไม่ควรใช้ใน grain นั้น
   - Data model ต้องตั้งค่า metric ที่ไม่ควรใช้เป็น `null`

## เป้าหมาย

สร้าง local dashboard สำหรับ `Seaweed Export Price from KOREA to Thailand ($/Ton)` จาก HS Code `121221` โดยแสดงต้นทุนย้อนหลังตั้งแต่ปี 2558 ถึงเดือนล่าสุดที่ตรวจสอบได้ พร้อมกราฟและรายละเอียดแบบ interactive

Dashboard ต้องตอบคำถามหลักเหล่านี้:

- ราคาส่งออกสาหร่ายจากเกาหลีมาไทยล่าสุดอยู่ที่เท่าไรในหน่วย `$/Ton`
- แนวโน้มต้นทุนเป็นอย่างไรในมุมรายเดือน รายไตรมาส และรายปี
- Growth เปลี่ยนแปลงอย่างไร โดยแสดงเฉพาะ metric ที่เหมาะกับแต่ละ time grain
- เมื่อคลิกจุดบนกราฟ ต้องเห็นรายละเอียดของช่วงเวลานั้นทันที
- ต้องมีข้อมูล YoY, MoM, QoQ ตามหลักการที่ไม่ทำให้ตีความผิด

## ชื่อและตำแหน่งไฟล์

ชื่อไฟล์ brief:

```text
Seaweed price.md
```

ตำแหน่งที่ต้อง save:

```text
D:\OneDrive\stock\Valuation หุ้น\TKN\Seaweed price.md
```

ไฟล์ dashboard ที่ควรสร้าง:

```text
seaweed_cost_dashboard.html
seaweed_cost_dashboard_data.json
seaweed_cost_dashboard_data.csv
```

แนะนำให้วาง output dashboard ไว้ในโฟลเดอร์ `outputs` ของ Codex workspace แล้วเปิดผ่าน local server เช่น:

```text
http://127.0.0.1:8766/outputs/seaweed_cost_dashboard.html
```

## แหล่งข้อมูลหลัก

ใช้ไฟล์ Excel ของผู้ใช้เป็นฐานข้อมูลย้อนหลัง:

```text
D:\OneDrive\stock\Valuation หุ้น\TKN\Seaweed Price sep 68.xlsx
```

ใช้ sheet หลัก:

```text
HS CODE_Month
```

อ่านข้อมูลรายเดือนจาก row 7 เป็นต้นไป โดยใช้ข้อมูลจาก Korea Customs Service Trade Statistics ที่อยู่ใน workbook

แหล่งข้อมูล live สำหรับเติมเดือนล่าสุด:

```text
https://tradedata.go.kr/cts/index_eng.do
```

Query logic:

- H.S Code: `121221`
- Country: `Thailand`
- Period: ต่อจากเดือนล่าสุดใน workbook จนถึงเดือนปัจจุบัน
- Item: `Fit for human consumption`
- ใช้ข้อมูล export weight และ export value เพื่อคำนวณราคา `$/Ton`

Live endpoint ที่ใช้ได้:

```text
https://tradedata.go.kr/cts/hmpgEng/retrieveTradeHsCountryEng.do
```

POST parameters:

```text
priodKind=MON
priodFr=<YYYYMM หลังเดือนล่าสุดใน workbook>
priodTo=<YYYYMM เดือนปัจจุบัน>
langTpcd=ENG
ttwgTpcd=1000
selectPaging=1
showPagingLine=100
hsSgnGrpCol=HS6_SGN
hsSgnWhrCol=HS6_SGN
hsSgn=121221
subHsSgn=Y
cntyNm=Thailand
```

ต้องใส่ browser-like headers เช่น `User-Agent`, `Accept`, `X-Requested-With: XMLHttpRequest`, `isAjax: true`, และ `Referer: https://tradedata.go.kr/cts/index_eng.do`

## Coverage ที่ต้องได้

เริ่ม dashboard ตั้งแต่:

```text
2015-01
```

เดือนล่าสุด ณ ตอนสร้างต้นฉบับ:

```text
2026-05
```

หมายเหตุ: อัปเดตล่าสุดวันที่ `2026-06-25` และ query live สำหรับ HS `121221` + Thailand คืนข้อมูลล่าสุดถึง `2026-05`; ยังไม่มี row รายเดือนเต็มของ `2026-06` สำหรับ HS/country นี้

## Data Model

ต้องสร้าง data 3 grain:

```text
month
quarter
year
```

Field หลักที่ควรมีในแต่ละ row:

```text
grain
period
label
sort
year
year_be
quarter
month
period_start
period_end
months_in_period
complete_period
country
item
hs_code
export_weight_ton
export_value_usd_000
import_weight_ton
import_value_usd_000
trade_balance_usd_000
price_usd_per_ton
reported_price_usd_per_ton
formula_price_usd_per_ton
price_reconcile_diff
mom
qoq
yoy
mom_basis
qoq_basis
yoy_basis
source_sheet
source_row
period_note
```

## สูตรคำนวณราคา

คำนวณราคาเป็น `$/Ton` จาก:

```text
price_usd_per_ton = export_value_usd_000 * 1000 / export_weight_ton
```

หาก workbook มี reported price ให้ใช้ตรวจ reconcile กับ formula price

```text
price_reconcile_diff = reported_price_usd_per_ton - formula_price_usd_per_ton
```

Trade balance ให้ normalize จาก:

```text
trade_balance_usd_000 = export_value_usd_000 - import_value_usd_000
```

เพราะ tail rows บางช่วงใน workbook อาจมี trade balance cell ที่ copy หรือ shift ไม่ตรง

## Aggregation

รายเดือน:

- ใช้ actual monthly row
- `months_in_period = 1`
- `complete_period = true`

รายไตรมาส:

- aggregate จาก monthly rows ในไตรมาสเดียวกัน
- ราคาใช้ weighted average จาก export value และ export weight
- ถ้ามีครบ 3 เดือน ให้ `complete_period = true`
- ถ้าเป็นไตรมาสล่าสุดและยังไม่ครบ 3 เดือน ให้เป็น `Quarter-to-date weighted average`

รายปี:

- aggregate จาก monthly rows ในปีเดียวกัน
- ราคาใช้ weighted average จาก export value และ export weight
- ถ้ามีครบ 12 เดือน ให้ `complete_period = true`
- ถ้าเป็นปีล่าสุดและยังไม่ครบ 12 เดือน ให้ label เป็น `YYYY YTD Jan-<Latest Month>`

Weighted average:

```text
weighted_price = sum(export_value_usd_000) * 1000 / sum(export_weight_ton)
```

## Growth Rules

สำคัญมาก: ห้ามโชว์ growth metric ที่ไม่เหมาะกับ grain นั้น แม้จะคำนวณได้ทางเทคนิค

รายเดือน:

- แสดงได้เฉพาะ `MoM` และ `YoY`
- ไม่ต้องแสดง `QoQ`
- ตั้ง `qoq = null`
- ตั้ง `qoq_basis = null`

สูตรรายเดือน:

```text
MoM = current_month_price / previous_month_price - 1
YoY = current_month_price / same_month_previous_year_price - 1
```

รายไตรมาส:

- แสดงได้เฉพาะ `QoQ` และ `YoY`
- ไม่ต้องแสดง `MoM`
- ตั้ง `mom = null`
- ตั้ง `mom_basis = null`

สูตรรายไตรมาส:

```text
QoQ = current_quarter_weighted_price / previous_quarter_weighted_price - 1
YoY = current_quarter_weighted_price / same_quarter_previous_year_weighted_price - 1
```

รายปี:

- แสดงได้เฉพาะ `YoY`
- ไม่ต้องแสดง `MoM`
- ไม่ต้องแสดง `QoQ`
- ตั้ง `mom = null`
- ตั้ง `qoq = null`
- ตั้ง `mom_basis = null`
- ตั้ง `qoq_basis = null`

สูตรรายปี:

```text
YoY = current_year_weighted_price / previous_year_weighted_price - 1
```

สำหรับปีล่าสุดที่เป็น YTD:

```text
YoY = current_ytd_weighted_price / previous_year_same_months_weighted_price - 1
```

ตัวอย่าง validation ล่าสุดจากต้นฉบับ:

```text
Latest monthly period: 2026-05
Monthly MoM: +2.5%
Monthly QoQ: null
Monthly YoY: -0.6%

Latest quarter period: 2026Q2
Quarterly MoM: null
Quarterly QoQ: -2.6%
Quarterly YoY: -2.9%

Latest year period: 2026 YTD Jan-May
Yearly MoM: null
Yearly QoQ: null
Yearly YoY: +0.9%
```

## Dashboard Layout

หน้าแรกต้องเป็น dashboard ใช้งานจริง ไม่ใช่ landing page

ส่วนประกอบหลัก:

1. Header
   - Title: `Seaweed Export Price from Korea to Thailand`
   - Subtitle: `HS 121221 | Country Thailand | Source Korea Customs Service Trade Statistics`
   - Coverage pill เช่น `2015-01 to 2026-05 | latest May 2026`

2. KPI cards
   - Latest selected grain price
   - Latest monthly signal
   - Latest annual/YTD
   - Range since 2558

3. Grain selector
   - รายเดือน
   - รายไตรมาส
   - รายปี

4. Price chart
   - Line chart ของ `price_usd_per_ton`
   - คลิกจุดแล้ว detail panel ต้องเปลี่ยนตาม

5. Growth chart
   - Line chart ของ growth metric ที่เลือก
   - ต้องมี filter ตาม grain:
     - รายเดือน: `MoM`, `YoY`
     - รายไตรมาส: `QoQ`, `YoY`
     - รายปี: `YoY`
   - ห้ามมีปุ่ม `MoM + QoQ`
   - ห้ามโชว์ `QoQ` ในรายเดือน
   - ห้ามโชว์ `MoM` ในรายไตรมาส
   - ห้ามโชว์ `MoM` หรือ `QoQ` ในรายปี

6. Detail panel
   - แสดงช่วงเวลาที่ selected
   - แสดง price
   - แสดงเฉพาะ growth metric ที่ถูกต้องกับ grain
   - แสดง export weight, export value, trade balance
   - แสดง comparison basis ของ metric ที่เลือก

7. Data table
   - Header ต้องเปลี่ยนตาม grain
   - รายเดือนมี columns: `Period`, `หมายเหตุ`, `$/Ton`, `MoM`, `YoY`, `Export Weight`, `Export Value`, `Source Row`
   - รายไตรมาสมี columns: `Period`, `หมายเหตุ`, `$/Ton`, `QoQ`, `YoY`, `Export Weight`, `Export Value`, `Source Row`
   - รายปีมี columns: `Period`, `หมายเหตุ`, `$/Ton`, `YoY`, `Export Weight`, `Export Value`, `Source Row`

8. Source & Methodology
   - ระบุ source URL
   - ระบุ workbook source path
   - ระบุ HS code, country, formula, aggregation, growth rules
   - ระบุ caveat ว่าเดือนล่าสุดขึ้นกับข้อมูลที่ endpoint คืนจริง

## Interaction Requirements

ต้องรองรับ:

- คลิกจุดใน price chart แล้ว detail panel update
- คลิกจุดใน growth chart แล้ว detail panel update และบอกว่าเลือก metric ไหน
- เปลี่ยน grain แล้ว chart, KPI, table, filter และ detail panel update พร้อมกัน
- เปลี่ยน growth filter แล้ว growth chart แสดงเฉพาะ metric นั้น
- Resize window แล้ว chart redraw ได้

## Visual Style

ใช้ dashboard โทนนักลงทุน:

- เรียบ อ่านง่าย ไม่เป็น landing page
- Background อ่อน
- Panel border ชัด
- KPI card กระชับ
- ใช้สีแยก series:
  - Price: blue
  - MoM: red
  - QoQ: amber
  - YoY: teal
- ต้องอ่านได้ทั้ง desktop และ mobile
- ห้ามใช้ decoration ที่ไม่จำเป็น
- ตารางต้อง scroll ได้เมื่อจอเล็ก

## Suggested Implementation

ใช้ Python สร้าง static HTML จะเหมาะที่สุด เพราะเปิด local dashboard ง่ายและส่งต่อได้

Library ที่ใช้:

```text
openpyxl
json
csv
urllib
datetime
pathlib
```

ขั้นตอน build:

1. อ่าน workbook ด้วย `openpyxl.load_workbook(..., data_only=True, read_only=True)`
2. อ่าน monthly rows จาก sheet `HS CODE_Month`
3. Query live TradeData endpoint จากเดือนหลัง workbook latest ถึงเดือนปัจจุบัน
4. Merge live rows โดยไม่ duplicate period ที่มีใน workbook แล้ว
5. คำนวณ monthly growth ตาม rules
6. Aggregate quarter และ year จาก monthly rows
7. คำนวณ quarter/year growth ตาม rules
8. Filter dashboard period ให้เริ่มปี `2015`
9. เขียน data เป็น JSON และ CSV
10. Embed JSON เข้า HTML
11. เปิดผ่าน local HTTP server

Local server example:

```powershell
python -m http.server 8766 --bind 127.0.0.1
```

เปิด dashboard:

```text
http://127.0.0.1:8766/outputs/seaweed_cost_dashboard.html
```

## Validation Checklist

ก่อนส่งมอบต้องเช็ค:

- Dashboard เปิดได้จริงผ่าน local URL
- Coverage แสดง `2015-01` ถึงเดือนล่าสุดที่ source คืนมา
- Latest source month ตรงกับ live endpoint
- JSON มี monthly/quarter/year ครบ
- รายเดือน:
  - มี filter `MoM`, `YoY`
  - ไม่มี filter `QoQ`
  - table ไม่มี column `QoQ`
  - detail panel ไม่มี `QoQ`
  - data field `qoq = null`
- รายไตรมาส:
  - มี filter `QoQ`, `YoY`
  - ไม่มี filter `MoM`
  - table ไม่มี column `MoM`
  - detail panel ไม่มี `MoM`
  - data field `mom = null`
- รายปี:
  - มี filter `YoY` เท่านั้น
  - ไม่มี `MoM` หรือ `QoQ`
  - table ไม่มี `MoM`, `QoQ`
  - detail panel ไม่มี `MoM`, `QoQ`
  - data field `mom = null`, `qoq = null`
- คลิกจุด chart แล้ว detail panel update
- ตัวเลข price คำนวณจาก export value และ export weight
- Reconcile price diff จาก workbook ควรใกล้ 0
- Latest YTD YoY เทียบกับ same months ของปีก่อน ไม่ใช่เต็มปี

## Prompt สำหรับส่งเข้า Codex

คัดลอก prompt นี้ให้ Codex:

```text
สร้าง local dashboard ชื่อ Seaweed Export Price from Korea to Thailand จากไฟล์ D:\OneDrive\stock\Valuation หุ้น\TKN\Seaweed Price sep 68.xlsx โดยอ่าน sheet HS CODE_Month และเติมข้อมูลล่าสุดจาก https://tradedata.go.kr/cts/index_eng.do สำหรับ HS Code 121221, Country Thailand, Period ต่อจากเดือนล่าสุดใน workbook ถึงเดือนล่าสุดที่ endpoint คืนข้อมูลจริง

ต้องสร้าง dashboard ย้อนหลังตั้งแต่ 2015-01 ถึงเดือนล่าสุด โดยมี view รายเดือน รายไตรมาส รายปี มี price chart, growth chart, KPI cards, detail panel, data table และ source methodology

สูตร price = export_value_usd_000 * 1000 / export_weight_ton
รายไตรมาสและรายปีต้อง aggregate จาก monthly rows ด้วย weighted average

Growth rules:
- รายเดือน แสดงเฉพาะ MoM และ YoY เท่านั้น ห้ามแสดง QoQ และ qoq ต้องเป็น null
- รายไตรมาส แสดงเฉพาะ QoQ และ YoY เท่านั้น ห้ามแสดง MoM และ mom ต้องเป็น null
- รายปี แสดงเฉพาะ YoY เท่านั้น ห้ามแสดง MoM/QoQ และ mom/qoq ต้องเป็น null
- ปีล่าสุดที่เป็น YTD ให้ YoY เทียบกับ same months ของปีก่อน

Growth chart ต้องมี filter ตาม grain:
- รายเดือน: ปุ่ม MoM และ YoY
- รายไตรมาส: ปุ่ม QoQ และ YoY
- รายปี: ปุ่ม YoY เท่านั้น

เมื่อคลิกจุดบน price chart หรือ growth chart ต้อง update detail panel พร้อมข้อมูล period, price, growth ที่ถูกต้องตาม grain, export weight, export value, trade balance และ comparison basis

Table header ต้องเปลี่ยนตาม grain และห้ามโชว์ metric ที่ไม่ควรมีใน grain นั้น

ให้สร้างไฟล์:
- outputs/seaweed_cost_dashboard.html
- outputs/seaweed_cost_dashboard_data.json
- outputs/seaweed_cost_dashboard_data.csv

จากนั้นเปิด local server และแจ้ง URL เช่น http://127.0.0.1:8766/outputs/seaweed_cost_dashboard.html

ต้อง validate ว่า dashboard เปิดได้จริง, local URL เสิร์ฟไฟล์ล่าสุด, monthly ไม่มี QoQ, quarterly ไม่มี MoM, yearly มีแค่ YoY, และ source latest month ตรงกับ endpoint จริง
```

## Expected Latest Snapshot จากต้นฉบับ

ใช้ snapshot นี้เป็นจุดเทียบ sanity check เท่านั้น เพราะข้อมูล live อาจเพิ่มเดือนใหม่ในอนาคต

```text
Generated date: 2026-06-10
Dashboard start: 2015-01
Dashboard latest period: 2026-05
Latest month label: May 2026
Latest month price: 23,462.1944 $/Ton
Latest YTD label: 2026 YTD Jan-May
Latest YTD price: 23,449.2806 $/Ton

Monthly rows reviewed: 173
Workbook monthly rows reviewed: 165
Live monthly rows added: 8
Dashboard monthly rows: 137
Dashboard quarter rows: 46
Dashboard year rows: 12
```

ถ้า run ในอนาคตแล้ว source มีเดือนใหม่ เช่น 2026-05 หรือถัดไป ให้ใช้เดือนล่าสุดจริงจาก endpoint แทน snapshot นี้
