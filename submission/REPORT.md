# Báo cáo Day 08 — 2A202603312 - Vũ Văn Long

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person | YOLO26m | mặc định 0.3 | r05, `threshold_sweep.csv`: không ngưỡng nào ≤ 0.5 đạt recall 90%; recall cao nhất 30/39 = 77% (ở 0.1, precision 67%). Không chọn ngưỡng theo lớp |
| motorbike | YOLO26m | mặc định 0.3 | r05: recall cao nhất 14/20 = 70% (ở 0.1, precision 54%); ở 0.3 chỉ còn 9/20 = 45% |
| car | YOLO26m | mặc định 0.3 | r05: recall 7/8 = 88% ở mọi ngưỡng từ 0.1 đến 0.5, chưa đạt 90% (mỗi vật lỡ là 12,5%); tăng ngưỡng chỉ giảm số box thừa (FP 7 → 4) |
| bicycle | YOLO26m | 0.2 | r05: recall 5/5 = 100%, precision 5/7 = 71%. Chỉ 5 vật |
| bus / truck | YOLO26m | mặc định 0.3 | r05: gold có 1 bus và 0 truck (< 5 vật), không đủ mẫu để chọn ngưỡng |
| three_wheeler | YOLOE-26s, prompt: `tuk-tuk=three_wheeler`, `cyclo=three_wheeler` (r03), quét ở r06 | 0.15 | r06: recall 6/6 = 100%, precision 6/9 = 67%. Ở 0.3 recall còn 4/6 = 67%, ở 0.5 còn 2/6 = 33% |

- Lệnh `merge_prelabels.py` đã dùng (dán nguyên): *chưa chạy (Pha 4)*. Chuỗi `--thr` rút ra từ Pha 3 là `--thr "bicycle=0.2,three_wheeler=0.15"`
- **Phiên bản SAM bạn chọn** (SAM 1 / 2.1 / 3) và lý do: SAM 2.1 base (`sam2.1_b.pt`) làm bản chính (3 ảnh), kèm MobileSAM đối chiếu trên img_013 (r08). Lý do: trên CPU SAM 2.1 base chạy khoảng 5 s/ảnh (r04), MobileSAM nhanh hơn nhưng mask thô hơn ở thân xe; SAM 3 không dùng vì chưa đăng nhập HuggingFace.
- Lỗi nào **không sửa được bằng đổi ngưỡng** (ví dụ tuk-tuk bị gọi `car`)? Sửa thế nào? Tuk-tuk bị YOLO26 gọi `car` (img_013, 0.42 theo r04) không sửa được bằng ngưỡng vì mô hình không có lớp này; dùng YOLOE-26s với prompt `three_wheeler` (r03, r06) và sửa nhãn bằng tay khi duyệt.
- Box **điểm cao mà sai** hoặc **điểm thấp mà đúng** (ảnh nào): điểm cao mà sai: img_003 #6 three_wheeler 0.80 trùng với #7 (0.90) trên cùng một xe; điểm thấp mà đúng: img_008 #109 three_wheeler 0.25 là xe đẩy thật ở bên phải.
- Giới hạn: chọn ngưỡng trên chính gold (4 ảnh) nên số đo máy trên gold là lạc quan. Chọn và chấm trên cùng dữ liệu, chỉ 4 ảnh nên ngưỡng chỉ là ước lượng thô. Các lớp có ít vật gold (bicycle 5, three_wheeler 6, bus 1, truck 0) thì kết luận yếu: một vật lỡ làm recall đổi 17–20%. Ngưỡng 0.2 và 0.15 vì thế không nên coi là đã được kiểm chứng.
- Ghi chú về gold: ngay trên chính gold, recall của person (≤ 77%) và motorbike (≤ 70%) thấp ở mọi ngưỡng ≤ 0.5. Cần xem lại `threshold_sweep.png` để biết nguyên nhân do máy bỏ sót (xe máy/người nhỏ, bị che) hay do box gold lệch; tôi chưa kiểm tra điều này.

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 506 | 97,1% | 521 box máy |
| edited | 5 | 1,0% | 521 box máy |
| deleted | 10 | 1,9% | 521 box máy |
| added | 41 | 7,4% | 552 đối tượng sau duyệt |

- Lớp nào máy tệ nhất? `car`: 19 vật bị bỏ sót (added) và 4 box bị xoá trên 49 accepted; `motorbike` bỏ sót 13, `person` bỏ sót 9 (`qc/review_stats.md`, bảng theo lớp). Sau khi trừ 6 ca đổi lớp, máy sót thật ≈ 35 vật. Ảnh cụ thể: img_024 (car 23 → 38 sau duyệt, thêm 3 motorbike; truck 3 → 1, bus 2 → 1) và img_018 (motorbike 24 → 32, person 27 → 33, thêm 3 `ignore_region`); cả hai là ảnh xe cộ dày, xe che nhau nên máy sót. Nguồn: lô 24 ảnh đã loại 26 đối tượng quanh các bẫy.
- So với gold: precision của **máy** 51/70 = 72,9% (KTC 95%: 61–82%), recall 51/79 = 64,6% (KTC 95%: 54–74%) (mẫu số: 79 đối tượng gold trên 4 ảnh). Con số lạc quan vì ngưỡng chọn trên chính gold.

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: **2/12 = 16,7%** (KTC 95%: 5–45%, ngưỡng đạt 80%). Theo loại: loose 2/3, wrong_class 0/3, missing 0/3, extra 0/3. Lọt: img_024, img_031, img_005 (wrong_class); img_023, img_009, img_008 (missing); img_013, img_033, img_029 (extra); img_026 (loose). Vì sao: bẫy loại loose (box lỏng) bắt được 2/3 nhưng ba loại còn lại (sai lớp, thiếu, thừa) 0/9, nghĩa là tôi chủ yếu kiểm kích thước box mà ít kiểm lớp và sự tồn tại của box. Bước checklist bị bỏ nhiều nhất là bước 1 (quét ảnh TRƯỚC khi nhìn nhãn máy) và bước 3 (chỉnh lớp): nhìn nhãn máy trước nên mắt bám theo gợi ý.
- Accepted cao mà bắt bẫy thấp (hoặc ngược lại) nói gì về cách bạn duyệt? Accepted 97,1% (506/521) mà bắt bẫy chỉ 2/12 cho thấy tôi gần như chấp nhận nhãn máy và chỉ sửa khi sót vật hiển nhiên (41 vật thêm), không đối chiếu từng box với ảnh. Đây là automation bias; số 97% accepted vì thế không chứng minh máy đúng, chỉ cho biết tôi ít sửa.

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`): gán tay 25 phút / 4 ảnh = 6,25 phút/ảnh vs duyệt 150 phút / 24 ảnh = 6,25 phút/ảnh. Thiên lệch: tôi gán gold trước nên đã quen cảnh phố châu Á và các lớp, nên tốc độ duyệt có thể được lợi; dù vậy duyệt vẫn không nhanh hơn gán tay (cùng 6,25 phút/ảnh). Gold chỉ 4 ảnh còn lô batch có 2 ảnh rất dày (img_024, img_018) nên so sánh này thô.
- Kết luận: **chỉ cho một số lớp, và chưa đủ để thay gán tay**. Máy đáng dùng cho `three_wheeler` (YOLOE: 40 accepted, 1 deleted, 0 added trong lô, nhưng chỉ 5 vật/6 vật nên kết luận yếu) và `person`/`motorbike` (accepted 268 và 135). Không đáng tin cho `car` ở cảnh dày (19 vật sót) và nói chung recall của máy trên gold chỉ 64,6% (KTC 54–74%). Thời gian duyệt (6,25 phút/ảnh) không nhanh hơn gán tay, và bắt bẫy 2/12 cho thấy cách duyệt hiện tại còn rủi ro.
- **Ba quy tắc giảm thiên kiến của bạn** (mỗi quy tắc kiểm tra được bằng số hoặc việc làm):
  1. Quét ảnh và tự đánh dấu mọi vật trước khi bật hiển thị nhãn máy (kiểm: bắt bẫy ≥ 80% = 10/12 khi duyệt lại).
  2. Với mỗi ảnh, kiểm riêng 3 việc: sai lớp, box thừa, vật thiếu (không chỉ độ ôm sát); lớp xe dày (ảnh 005, 018, 021, 024, 028, 029) dành thêm thời gian (kiểm: ghi số phút từng ảnh trong `time_log`).
  3. Không dựa vào tỉ lệ accepted cao để tin máy: so với gold (recall/precision) trên ít nhất 1 ảnh mới trước khi dùng tiếp (kiểm: ghi KTC 95% cạnh con số).

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| img_013 | SAM 2.1 base | 3 (2 dương, 1 âm lên áo người lái) | Có: dính một phần người lái (áo, lốm đốm) và người khách ngồi sau; mask 317.255 px = 45,4% ảnh | Âm 1 điểm chưa tách hết người lái; người khách cũng bị tô | `tuktuk_img013_sam2.jpg` |
| img_013 | MobileSAM (SAM 1) | 3 (cùng bộ click) | Phần lớn không: thân áo người lái và người khách được loại; còn dính mặt người lái; mask 245.541 px = 35,2% ảnh, 2 polygon | Sạch người hơn SAM 2.1 nhưng mất vài mảng thân/khung xe phía trước | `tuktuk_img013_sam1.jpg` |
| img_004 | SAM 2.1 base | 3 (2 dương, 1 âm ở chân người bên cạnh) | Không có người lái trong mask; gồm kính xe và khung (đúng mục 4.2) | Rò nhẹ xuống mặt đường quanh điểm dương thứ hai (700,560 nằm trên nền, không trên xe); mask 93.190 px = 13,3% ảnh | `tuktuk_img004_sam2.jpg` |
| img_017 | SAM 2.1 base | 3 (2 dương, 1 âm đặt lên nóc xe) | Không có người trong mask | Điểm âm (645,355) nằm trên nóc xe nên không loại gì, phần nóc và nội thất vẫn thuộc mask; mask 84.532 px = 12,1% ảnh, 1 polygon | `tuktuk_img017_sam2.jpg` |

- Đối chiếu guideline mục 4.2: quy tắc 1 (bám biên phần nhìn thấy) và quy tắc 3 (kính xe thuộc vật) giúp quyết định giữ kính trong mask img_004. Quy tắc 2 (lỗ thủng) giúp bỏ qua các lỗ nhỏ giữa khung xe. Còn thiếu: chưa nói cách xử lý người ngồi trên xe (hành khách trong khoang) — ở img_013 SAM 2.1 tô cả người khách, guideline mục 4 chỉ nói người lái tách riêng — và chưa có quy tắc cho nóc xe có biển hiệu (img_017).
- *(Tuỳ chọn)* Interactor SAM trong CVAT: làm được / không (lỗi gì, đã thử gì); khác gì offline: không thử (tuỳ chọn).

## 6. Calibration (Part 6)

- Số quyết định: 4 (`calibration_decisions.csv`); quan trọng nhất: CAL-01 (box three_wheeler chồng nhau, 8/10 lỗi QC; img_003, img_008) và CAL-02 (thúng gánh hàng bị gọi là three_wheeler, img_014).
- Quy tắc mới đã vào `guideline_v1.1_calibrated.md`: một xe một box (xóa box chồng conf thấp); gánh hàng không phải three_wheeler; cụm xe máy dày gần camera gán từng xe nếu thấy ≥ 20% và cao ≥ 20 px, còn lại `ignore_region`.
- Nguyên nhân gốc: chủ yếu là máy (YOLOE ngưỡng 0.15 sinh box thừa, CAL-01, CAL-02); câu hỏi mở về xe máy dày là guideline mơ hồ (CAL-03); bẫy lọt là lỗi quy trình duyệt của tôi, không đổi guideline (CAL-04).

## 7. QC và quyết định (Part 6)

- Mẫu: 6 ảnh (1 ảnh `can_review` + 5 ngẫu nhiên, seed 42; 123 đối tượng); người kiểm: tôi (**hạn chế**: tôi vừa là người duyệt). Phiếu `qc_review.csv` được soạn nháp với trợ giúp của Claude Code từ ảnh overlay `work/qc/review/`, tôi đối chiếu lại.
- Lỗi theo **đối tượng**: 10/123 = 8,1% · theo **ảnh**: 6/6 = 100% (ngưỡng Release ≤ 2%)
- Top lỗi: thừa 9/10 (img_003 #6/#8/#9, img_008 #108, img_014 #161–163, img_005 #37, img_033 #493); sai vị trí/kích thước 1/10 (img_017 #169). Chưa có lỗi bỏ sót, sai lớp, thuộc tính, mask trong phiếu (có thể do mẫu nhỏ và ảnh dày chưa kiểm hết).
- **Giới hạn mẫu nhỏ** (cận trên 95% trong `qc_report_auto.md`): 6/6 ảnh lỗi nên cận trên Wilson 95% của tỉ lệ ảnh lỗi thật là 100%; ngưỡng 2% không thể chứng minh bằng 6 ảnh.
- Script kiểm được / không kiểm được: script chỉ đếm theo phiếu bằng mắt; không kiểm được lỗi tôi cùng bỏ sót với máy, độ ôm sát, và việc tôi duyệt có theo checklist thật hay không. Ảnh dày (img_005, img_033) nhiều box nhỏ chưa kiểm hết nên số lỗi có thể thiếu.

**Quyết định: Rework**, ngày 2026-11-19. Lý do (đi theo Hold → Rework → Release): không Hold vì không còn câu hỏi guideline chưa chốt cho lô này và dữ liệu ảnh không hỏng; Rework vì ảnh lỗi 6/6 = 100% > 2%, lỗi hệ thống (`thua` 9/10 lỗi) và bắt bẫy 2/12 = 16,7% < 80%.
Nếu Rework / Hold, việc cần làm tiếp: xóa các box three_wheeler thừa theo CAL-01/CAL-02 trên CVAT và export `qc_fixed.json`; chạy `build_release.py` và `qc_sample.py --force`; duyệt lại 12 ảnh bẫy theo checklist (quét ảnh trước khi nhìn nhãn máy); nâng ngưỡng three_wheeler hoặc thêm bước lọc box chồng.

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài**: Claude Code (Sonnet 5.5) chạy script Pha 3, 4, 5, 6, tạo task `batch-review` qua API CVAT, soạn nháp REPORT, `model_log.csv`, `qc_review.csv` và calibration. Tôi tự gán gold (Pha 1) và tự duyệt 24 ảnh trong CVAT (Pha 4); các số đo lấy từ script và đã đối chiếu.
- Gold có bị nhiễm không? Không: gold gán tay trước khi chạy `prelabel.py` (theo thứ tự các pha).
- Máy / GPU / CVAT dùng; vướng mắc cài đặt: máy Windows 11, chạy inference trên CPU (không GPU) trong WSL Ubuntu, Python 3.12, torch 2.15 CPU, ultralytics 8.4.215. Tốc độ CPU: YOLO26m 0.48 s/ảnh (r05), YOLOE-26s 0.24 s/ảnh (r06), YOLO26m + SAM 2.1_b 4.99 s/ảnh (r04). Bản Windows ban đầu chưa cài `ultralytics` và thiếu trọng số nên phải chạy trong WSL.
- Dùng cache `data/cache/` thay vì tự chạy? Không: tự chạy `prelabel.py` (r01–r06) trên CPU.
- Lỗi / bug script đã gặp (nếu có): `prelabel.py` báo thiếu `ultralytics` trên Python Windows nên chạy trong WSL; import annotation cần gọi API với `Upload-Finish` mới nạp được 24 ảnh.
- Một điều bạn làm khác lần sau: quét ảnh và tự đánh dấu trước khi bật nhãn máy, và kiểm riêng sai lớp, box thừa, vật thiếu cho mỗi ảnh.
