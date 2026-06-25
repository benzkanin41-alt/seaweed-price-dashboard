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

- Dashboard coverage: `2015-01` to `2026-05`
- Latest month: `May 2026`
- Latest monthly price: `23,462.1944 $/Ton`
- Latest YTD: `2026 YTD Jan-May`
- Latest YTD price: `23,449.2806 $/Ton`

## Growth Rules

- Monthly view: show only `MoM` and `YoY`; `QoQ` is not shown and is stored as `null`
- Quarterly view: show only `QoQ` and `YoY`; `MoM` is not shown and is stored as `null`
- Yearly view: show only `YoY`; `MoM` and `QoQ` are not shown and are stored as `null`

อ่านรายละเอียดทั้งหมดได้ใน [`Seaweed price.md`](Seaweed%20price.md).
