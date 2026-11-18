# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 521; đối tượng sau duyệt: 552; ngưỡng accepted: IoU >= 0.9
- Đã **loại 26 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 506 | 506/521 = 97.1% | 521 box máy |
| edited (sửa) | 5 | 5/521 = 1.0% | 521 box máy |
| deleted (xoá) | 10 | 10/521 = 1.9% | 521 box máy |
| added (vẽ thêm) | 41 | 41/552 = 7.4% | 552 đối tượng sau duyệt |

Trong số deleted + added, **6 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 35.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 268 | 2 | 1 | 9 |
| bicycle | 8 | 1 | 0 | 0 |
| motorbike | 135 | 2 | 1 | 13 |
| car | 49 | 0 | 4 | 19 |
| bus | 2 | 0 | 1 | 0 |
| truck | 4 | 0 | 2 | 0 |
| three_wheeler | 40 | 0 | 1 | 0 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 79 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 51 | 19 | 28 | 72.9% (KTC 95%: 61–82%) | 64.6% (KTC 95%: 54–74%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 2/12 = 16.7% (KTC 95%: 5–45%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - wrong_class: 0/3
  - missing: 0/3
  - loose: 2/3
  - extra: 0/3
  - lọt: img_024.jpg (wrong_class), img_023.jpg (missing), img_026.jpg (loose), img_009.jpg (missing), img_008.jpg (missing), img_013.jpg (extra), img_031.jpg (wrong_class), img_033.jpg (extra), img_005.jpg (wrong_class), img_029.jpg (extra)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
