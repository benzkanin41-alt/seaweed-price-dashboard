# Seaweed price dashboard

คู่มือนี้คือ prompt/spec สำหรับส่งให้เพื่อนหรือส่งเข้า Codex เพื่อสร้าง dashboard ต้นทุนสาหร่ายแบบเดียวกับที่ทำใน chat นี้ให้ครบทั้งข้อมูล, logic การคำนวณ, local dashboard, GitHub Pages และไฟล์รายละเอียดประกอบ

ปลายทางที่ต้อง save คู่มือนี้:

```text
D:\OneDrive\stock\OPPDAY\PROMPT\Seaweed price dashboard.md
```

Dashboard online ที่สร้างไว้แล้ว:

```text
https://benzkanin41-alt.github.io/seaweed-price-dashboard/
```

GitHub repo:

```text
https://github.com/benzkanin41-alt/seaweed-price-dashboard
```

## เป้าหมายของงาน

สร้าง dashboard สำหรับติดตามต้นทุนวัตถุดิบสาหร่ายของ TKN จากข้อมูล `Seaweed Export Price from Korea to Thailand ($/Ton)` โดยใช้ข้อมูล Korea Customs Service Trade Statistics และ workbook ที่ผู้ใช้ให้มา

Dashboard ต้องทำได้ดังนี้:

- แสดงข้อมูลย้อนหลังตั้งแต่ปี 2558 หรือ `2015-01`
- เติมข้อมูลถึงเดือนล่าสุดที่ source คืนข้อมูลจริง
- มี view รายเดือน รายไตรมาส รายปี
- มีกราฟราคา `$/Ton`
- มีกราฟ growth
- คลิกจุดในกราฟแล้ว detail panel เปลี่ยนตาม
- มี KPI cards, data table, source & methodology
- เปิด local dashboard ได้
- อัปโหลดขึ้น GitHub Pages เพื่อเปิด online ได้

## Source ที่ต้องใช้

Workbook หลัก:

```text
D:\OneDrive\stock\Valuation หุ้น\TKN\Seaweed Price sep 68.xlsx
```

Sheet หลัก:

```text
HS CODE_Month
```

Source online:

```text
https://tradedata.go.kr/cts/index_eng.do
```

Live endpoint ที่ใช้เติมข้อมูลล่าสุด:

```text
https://tradedata.go.kr/cts/hmpgEng/retrieveTradeHsCountryEng.do
```

เงื่อนไข source:

- H.S Code: `121221`
- Country: `Thailand`
- Item: `Fit for human consumption`
- Period: ต่อจากเดือนล่าสุดใน workbook ไปจนถึงเดือนปัจจุบัน
- ใช้ export value และ export weight เป็นฐานคำนวณราคา

POST parameters สำหรับ live endpoint:

```text
priodKind=MON
priodFr=<YYYYMM หลังเดือนล่าสุดใน workbook>
priodTo=<YYYYMM เดือนปัจจุบัน>
langTpcd=ENG
ttwgTpcd=1000
selectPaging=1
showPagingLine=100
sortColumn=
sortOrder=
hsSgnGrpCol=HS6_SGN
hsSgnWhrCol=HS6_SGN
hsSgn=121221
subHsSgn=Y
cntyNm=Thailand
```

Headers ที่ควรใส่:

```text
User-Agent: browser-like user agent
Accept: application/json, text/javascript, */*; q=0.01
Content-Type: application/x-www-form-urlencoded; charset=UTF-8
X-Requested-With: XMLHttpRequest
isAjax: true
Referer: https://tradedata.go.kr/cts/index_eng.do
```

## Snapshot ของ dashboard ล่าสุด

วันที่ build dashboard ต้นฉบับ:

```text
2026-06-10
```

วันที่ context ล่าสุดใน thread:

```text
2026-06-14
```

Coverage ที่ได้จาก dashboard ล่าสุด:

```text
Start: 2015-01
Latest source period: 2026-05
Latest month label: May 2026
Latest month price: 23,462.1944 $/Ton
Latest YTD label: 2026 YTD Jan-May
Latest YTD price: 23,449.2806 $/Ton
```

Validation counts:

```text
Monthly rows reviewed: 173
Workbook monthly rows reviewed: 165
Live monthly rows added: 8
Dashboard monthly rows: 137
Dashboard quarter rows: 46
Dashboard year rows: 12
```

หมายเหตุ: อัปเดตล่าสุดวันที่ `2026-06-25` live endpoint คืนข้อมูลล่าสุดสำหรับ HS `121221` + `Thailand` ถึง `2026-05` ยังไม่มี row รายเดือนเต็มของ `2026-06` สำหรับ combination นี้

## Data model ที่ต้องสร้าง

ต้องมี 3 grain:

```text
month
quarter
year
```

Field หลักในแต่ละ row:

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
reported_trade_balance_usd_000
trade_balance_reconcile_diff
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

## สูตรคำนวณ

ราคา:

```text
price_usd_per_ton = export_value_usd_000 * 1000 / export_weight_ton
```

Reconcile ราคา:

```text
price_reconcile_diff = reported_price_usd_per_ton - formula_price_usd_per_ton
```

Trade balance:

```text
trade_balance_usd_000 = export_value_usd_000 - import_value_usd_000
```

เหตุผล: ใน workbook บาง tail rows มี trade balance cell ที่ copy หรือ shift ไม่ตรง จึง normalize ด้วยสูตรนี้

Weighted average สำหรับรายไตรมาสและรายปี:

```text
weighted_price = sum(export_value_usd_000) * 1000 / sum(export_weight_ton)
```

## Aggregation rules

รายเดือน:

- ใช้ actual monthly row
- `months_in_period = 1`
- `complete_period = true`

รายไตรมาส:

- aggregate จาก monthly rows ในไตรมาสเดียวกัน
- ราคาใช้ weighted average
- ถ้าครบ 3 เดือน: `complete_period = true`
- ถ้าไตรมาสล่าสุดยังไม่ครบ 3 เดือน: label/note เป็น `Quarter-to-date weighted average`

รายปี:

- aggregate จาก monthly rows ในปีเดียวกัน
- ราคาใช้ weighted average
- ถ้าครบ 12 เดือน: `complete_period = true`
- ถ้าปีล่าสุดยังไม่ครบ 12 เดือน: label เป็น `YYYY YTD Jan-<Latest Month>`

## Growth rules ล่าสุด

สำคัญมาก: ห้ามแสดง metric ที่ไม่เหมาะกับ grain นั้น แม้จะคำนวณได้ทางเทคนิค เพราะจะทำให้ตีความสัญญาณต้นทุนผิดในมุมมองนักลงทุน

รายเดือน:

- แสดงเฉพาะ `MoM` และ `YoY`
- ไม่แสดง `QoQ`
- ตั้ง `qoq = null`
- ตั้ง `qoq_basis = null`

สูตร:

```text
MoM = current_month_price / previous_month_price - 1
YoY = current_month_price / same_month_previous_year_price - 1
```

รายไตรมาส:

- แสดงเฉพาะ `QoQ` และ `YoY`
- ไม่แสดง `MoM`
- ตั้ง `mom = null`
- ตั้ง `mom_basis = null`

สูตร:

```text
QoQ = current_quarter_weighted_price / previous_quarter_weighted_price - 1
YoY = current_quarter_weighted_price / same_quarter_previous_year_weighted_price - 1
```

รายปี:

- แสดงเฉพาะ `YoY`
- ไม่แสดง `MoM`
- ไม่แสดง `QoQ`
- ตั้ง `mom = null`
- ตั้ง `qoq = null`
- ตั้ง `mom_basis = null`
- ตั้ง `qoq_basis = null`

สูตร:

```text
YoY = current_year_weighted_price / previous_year_weighted_price - 1
```

สำหรับปีล่าสุดแบบ YTD:

```text
YoY = current_ytd_weighted_price / previous_year_same_months_weighted_price - 1
```

Latest validation ตัวอย่าง:

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

## Dashboard UI requirements

หน้าแรกต้องเป็น dashboard ใช้งานจริง ไม่ใช่ landing page

องค์ประกอบ:

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
   - คลิกจุดแล้ว detail panel update

5. Growth chart
   - Line chart ของ growth metric ที่เลือก
   - Filter ต้องเปลี่ยนตาม grain:
     - รายเดือน: `MoM`, `YoY`
     - รายไตรมาส: `QoQ`, `YoY`
     - รายปี: `YoY`
   - ห้ามมีปุ่ม `MoM + QoQ`
   - ห้ามโชว์ `QoQ` ในรายเดือน
   - ห้ามโชว์ `MoM` ในรายไตรมาส
   - ห้ามโชว์ `MoM` หรือ `QoQ` ในรายปี

6. Detail panel
   - แสดง period ที่เลือก
   - แสดง price
   - แสดงเฉพาะ growth metric ที่ถูกต้องกับ grain
   - แสดง export weight, export value, trade balance
   - แสดง comparison basis ของ metric ที่เลือก

7. Data table
   - Header เปลี่ยนตาม grain
   - รายเดือน: `Period`, `หมายเหตุ`, `$/Ton`, `MoM`, `YoY`, `Export Weight`, `Export Value`, `Source Row`
   - รายไตรมาส: `Period`, `หมายเหตุ`, `$/Ton`, `QoQ`, `YoY`, `Export Weight`, `Export Value`, `Source Row`
   - รายปี: `Period`, `หมายเหตุ`, `$/Ton`, `YoY`, `Export Weight`, `Export Value`, `Source Row`

8. Source & Methodology
   - Source URL
   - Workbook source path
   - HS code, country, formula, aggregation, growth rules
   - Coverage caveat

## Interaction requirements

Dashboard ต้องรองรับ:

- คลิกจุดใน price chart แล้ว detail panel update
- คลิกจุดใน growth chart แล้ว detail panel update พร้อมบอก metric ที่เลือก
- เปลี่ยน grain แล้ว chart, KPI, table, filter, detail panel update พร้อมกัน
- เปลี่ยน growth filter แล้ว growth chart แสดงเฉพาะ metric นั้น
- Resize window แล้ว chart redraw ได้
- Table scroll ได้บนจอเล็ก

## Visual style

ใช้โทน dashboard นักลงทุน:

- เรียบ อ่านง่าย
- ไม่ทำ landing page
- Background อ่อน
- Panel border ชัด
- KPI card กระชับ
- สี series:
  - Price: blue
  - MoM: red
  - QoQ: amber
  - YoY: teal
- อ่านได้ทั้ง desktop และ mobile
- ไม่มี decoration ที่ไม่จำเป็น

## Files ที่ต้อง output

ใน workspace:

```text
outputs/seaweed_cost_dashboard.html
outputs/seaweed_cost_dashboard_data.json
outputs/seaweed_cost_dashboard_data.csv
outputs/Seaweed price.md
outputs/seaweed_dashboard_preview.png
outputs/seaweed_dashboard_mobile_preview.png
```

สำหรับ GitHub Pages:

```text
github-site/index.html
github-site/seaweed_cost_dashboard.html
github-site/seaweed_cost_dashboard_data.json
github-site/seaweed_cost_dashboard_data.csv
github-site/Seaweed price.md
github-site/README.md
github-site/.nojekyll
github-site/seaweed_dashboard_preview.png
github-site/seaweed_dashboard_mobile_preview.png
```

## Suggested implementation

ใช้ Python สร้าง static HTML เพราะเปิด local dashboard และ push GitHub Pages ง่าย

Libraries:

```text
openpyxl
json
csv
urllib.parse
urllib.request
datetime
pathlib
collections.defaultdict
```

Build steps:

1. โหลด workbook ด้วย `openpyxl.load_workbook(..., data_only=True, read_only=True)`
2. อ่าน sheet `HS CODE_Month`
3. Parse monthly rows ตั้งแต่ row 7 เป็นต้นไป
4. คำนวณ formula price และ reconcile กับ reported price
5. Query live endpoint จากเดือนหลัง workbook latest ถึงเดือนปัจจุบัน
6. Merge live rows โดยไม่ duplicate period
7. Sort monthly rows ตาม month index
8. คำนวณ monthly growth ด้วย rule ล่าสุด
9. Aggregate quarter/year จาก monthly rows
10. คำนวณ quarter/year growth ด้วย rule ล่าสุด
11. Filter dashboard period ตั้งแต่ `2015-01`
12. เขียน JSON/CSV
13. Embed JSON เข้า HTML
14. เปิด local server
15. ตรวจ UI/logic
16. Copy static site เข้า `github-site`
17. Push ขึ้น GitHub และเปิด GitHub Pages

Local server:

```powershell
python -m http.server 8766 --bind 127.0.0.1
```

Local URL:

```text
http://127.0.0.1:8766/outputs/seaweed_cost_dashboard.html
```

## GitHub Pages publish steps

สร้าง static site folder:

```text
github-site
```

ให้ `index.html` เป็น dashboard:

```text
Copy outputs/seaweed_cost_dashboard.html -> github-site/index.html
```

Init git:

```powershell
git init -b main
git config user.name "Codex"
git config user.email "codex@local"
git add .
git commit -m "Publish seaweed price dashboard"
```

สร้าง repo:

```powershell
gh repo create seaweed-price-dashboard --public --source=. --remote=origin --push --description "Seaweed export price dashboard for TKN raw material cost tracking"
```

เปิด GitHub Pages:

```powershell
gh api --method POST repos/<owner>/seaweed-price-dashboard/pages -f build_type=legacy -f "source[branch]=main" -f "source[path]=/"
```

ตรวจ status:

```powershell
gh api repos/<owner>/seaweed-price-dashboard/pages
curl -I https://<owner>.github.io/seaweed-price-dashboard/
```

ต้นฉบับที่ทำไว้:

```text
Owner: benzkanin41-alt
Repo: seaweed-price-dashboard
Pages URL: https://benzkanin41-alt.github.io/seaweed-price-dashboard/
```

## Validation checklist

ก่อนส่งมอบต้องตรวจ:

- Local dashboard เปิดได้จริง
- GitHub Pages เปิดได้จริง
- HTTP status online เป็น `200 OK`
- GitHub Pages status เป็น `built`
- Coverage แสดง `2015-01` ถึง latest source period
- Latest source month ตรงกับ live endpoint
- JSON มี `month`, `quarter`, `year`
- รายเดือน:
  - filter มี `MoM`, `YoY`
  - ไม่มี `QoQ`
  - table ไม่มี column `QoQ`
  - detail panel ไม่มี `QoQ`
  - data field `qoq = null`
- รายไตรมาส:
  - filter มี `QoQ`, `YoY`
  - ไม่มี `MoM`
  - table ไม่มี column `MoM`
  - detail panel ไม่มี `MoM`
  - data field `mom = null`
- รายปี:
  - filter มี `YoY` เท่านั้น
  - ไม่มี `MoM`, `QoQ`
  - table ไม่มี columns `MoM`, `QoQ`
  - detail panel ไม่มี `MoM`, `QoQ`
  - data fields `mom = null`, `qoq = null`
- คลิก chart แล้ว detail panel update
- Price คำนวณจาก export value และ export weight
- Latest YTD YoY เทียบ same months ของปีก่อน
- Markdown ภาษาไทยอ่านได้แบบ UTF-8

## Prompt สำหรับส่งให้ Codex

คัดลอก prompt นี้ไปใช้:

```text
สร้าง dashboard ชื่อ Seaweed Export Price from Korea to Thailand จากไฟล์ D:\OneDrive\stock\Valuation หุ้น\TKN\Seaweed Price sep 68.xlsx โดยอ่าน sheet HS CODE_Month และเติมข้อมูลล่าสุดจาก https://tradedata.go.kr/cts/index_eng.do สำหรับ HS Code 121221, Country Thailand, Period ต่อจากเดือนล่าสุดใน workbook ถึงเดือนล่าสุดที่ endpoint คืนข้อมูลจริง

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
- outputs/Seaweed price.md

จากนั้นเปิด local server และแจ้ง URL เช่น http://127.0.0.1:8766/outputs/seaweed_cost_dashboard.html

ถ้าต้องการเปิด online ให้ copy static dashboard ไป github-site/index.html แล้ว push ขึ้น GitHub Pages

ต้อง validate ว่า dashboard เปิดได้จริง, local URL เสิร์ฟไฟล์ล่าสุด, GitHub Pages เปิดได้ 200 OK, monthly ไม่มี QoQ, quarterly ไม่มี MoM, yearly มีแค่ YoY, และ source latest month ตรงกับ endpoint จริง
```

## Notes สำหรับเครื่อง Windows/Codex

- ถ้า PowerShell แสดงภาษาไทยเพี้ยนเป็น mojibake ให้ตรวจไฟล์ด้วย `python -X utf8` ก่อนสรุปว่าไฟล์เสีย
- ถ้า shell ขึ้น `CreateProcessAsUserW failed: 5` ให้ใช้ explicit PowerShell path หรือ Python UTF-8 helper แทนคำสั่งยาว
- ถ้าผู้ใช้กำหนด path D: เป็น final destination ต้อง verify จาก path D: จริง ไม่ใช่แค่ไฟล์ใน workspace
