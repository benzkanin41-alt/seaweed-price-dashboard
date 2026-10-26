# Seaweed Price Dashboard

Interactive local/static dashboard สำหรับติดตามต้นทุนสาหร่ายของ TKN ผ่านข้อมูล `Seaweed Export Price from Korea to Thailand ($/Ton)`.

## Online Dashboard

เปิดหน้า GitHub Pages แล้วจะเข้าสู่ dashboard โดยตรงผ่าน `index.html`.

ไฟล์หลัก:

- `index.html` - dashboard สำหรับเปิด online
- `seaweed_cost_dashboard.html` - dashboard copy ต้นฉบับ
- `seaweed_cost_dashboard_data.json` - dataset สำหรับ dashboard
- `seaweed_cost_dashboard_data.csv` - dataset แบบ CSV
- `Seaweed price.md` - build brief และรายละเอียดคำสั่งทั้งหมดสำหรับ recreate dashboard ใน Codex
- `seaweed_dashboard_preview.png` - desktop preview
- `seaweed_dashboard_mobile_preview.png` - mobile preview

## Source

- Korea Customs Service Trade Statistics: https://tradedata.go.kr/cts/index_eng.do
- HS Code: `121221`
- Country: `Thailand`
- Workbook source used in original build: `D:\OneDrive\stock\Valuation หุ้น\TKN\Seaweed Price sep 68.xlsx`

## Current Snapshot

- Checked on: `2026-10-26`, queried through `2026-10`
- Dashboard actual-data coverage: `2015-01` to `2026-09`
- October 2026 monthly data: not yet returned by the official HS 121221 / Thailand endpoint; no October price is fabricated
- Latest month: `Sep 2026`
- Latest monthly price: `21,545.6546 $/Ton`; MoM `+6.1%`, YoY `-5.7%`
- Latest quarter: complete `2026Q3`, price `20,674.6225 $/Ton`; QoQ `-9.3%`, YoY `-12.2%`
- Latest YTD: `2026 YTD Jan-Sep`
- Latest YTD price: `22,985.7903 $/Ton`; YoY `-2.2%`

## Growth Rules

- Monthly view: show only `MoM` and `YoY`; `QoQ` is not shown and is stored as `null`
- Quarterly view: show only `QoQ` and `YoY`; `MoM` is not shown and is stored as `null`
- Incomplete latest quarter: QoQ and YoY compare matching month positions in the prior quarter and matching months in the prior-year quarter, respectively
- Yearly view: show only `YoY`; `MoM` and `QoQ` are not shown and are stored as `null`

อ่านรายละเอียดทั้งหมดได้ใน [`Seaweed price.md`](Seaweed%20price.md).
