# Báo cáo QC tự động (nháp)

> **Gợi ý** do script tính; bạn xác nhận. Script chỉ đếm theo phiếu kiểm bằng mắt người.

## Mẫu
- 6 ảnh (1 ca khó + 5 ngẫu nhiên), 123 đối tượng.
- taxonomy v1.0

## Số đo
| Chỉ số | Giá trị | Mẫu số |
|---|---|---|
| Lỗi **theo đối tượng** | 10/123 = 8.1% | 123 đối tượng trong mẫu |
| **Ảnh có lỗi** (dùng cho ngưỡng) | 6/6 = 100.0% | 6 ảnh trong mẫu |
| Máy: accepted / edited / deleted | 506/521 = 97.1% / 5/521 = 1.0% / 10/521 = 1.9% | 521 box máy |
| Máy: added | 41/552 = 7.4% | 552 đối tượng sau duyệt |
| Bắt lỗi bẫy | 2/12 = 16.7% | 12 bẫy |

## Top 3 loại lỗi
| Loại lỗi | Số lỗi | Ảnh ví dụ |
|---|---|---|
| thua | 9 | img_003.jpg, img_005.jpg, img_008.jpg |
| sai_vi_tri_kich_thuoc | 1 | img_017.jpg |

**Lỗi hệ thống**: 'thua' lặp lại -> sửa quy tắc / recipe, không chỉ sửa từng ảnh.

## Giới hạn thống kê
- 6 ảnh lỗi trong 6 ảnh mẫu: cận trên 95% (Wilson) của tỉ lệ ảnh lỗi thật là **100%**. Ngưỡng 2% không thể được **chứng minh** bằng mẫu nhỏ như vậy: hãy nói rõ giới hạn này trong báo cáo.

## Gợi ý quyết định
**REWORK**

- ảnh có lỗi 100.0% > 2%
- lỗi hệ thống: 'thua' chiếm 9/10 lỗi
- bắt bẫy 17% < 80% -> lô duyệt chưa đáng tin

Cây quyết định: Hold (guideline chưa rõ / dữ liệu hỏng) -> Rework (> ngưỡng, lỗi lặp, bắt bẫy / nhất quán thấp) -> Release.
