# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603312 - Vũ Văn Long

> **Chỉ viết phần thay đổi so với v1.0** (không copy cả file). Chốt các câu hỏi mở ở mục 6 của v1.0 (hoặc nói rõ câu nào chưa chốt, vì sao)
> và thêm quy tắc mới rút ra từ lỗi của máy / lỗi của bạn. Mỗi ca khó **có ảnh ví dụ** (`img_0xx`) và khớp một dòng trong `qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 2 | Khối xe máy đỗ dày đặc gần camera: từng xe hay `ignore_region`? | Gán từng xe nếu thấy ≥ 20% và cao ≥ 20 px; phần còn lại gom một `ignore_region`. Các câu hỏi 1, 3–7 của v1.0 **chưa chốt** (chưa gặp đủ ca trong lô này). | img_005, img_018, img_024 | CAL-03 |

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của bạn | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| img_003, img_008 | Nhiều box three_wheeler chồng nhau trên cùng một xe ba gác | YOLOE (ngưỡng 0.15) sinh box conf 0.20–0.90 trên cùng xe | Mỗi xe đúng một box, xóa box chồng conf thấp | Mục 4.1: một xe một box (CAL-01) |
| img_014 | Thúng của người gánh hàng bị gọi là three_wheeler | YOLOE, 3 box conf 0.19–0.63 | Chỉ gán `person`, không gán lớp xe | Mục 2: gánh, thúng, xe đẩy tay không bánh không thuộc three_wheeler (CAL-02) |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | Chốt câu hỏi 2 (xe máy dày); thêm quy tắc một xe một box; loại gánh hàng khỏi three_wheeler | CAL-01, CAL-02, CAL-03 |
