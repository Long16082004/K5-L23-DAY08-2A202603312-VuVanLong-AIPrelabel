# Thông tin học viên

- Họ và tên: Vũ Văn Long
- MSHV: 2A202603312
- Lớp (L2 / L3 / khác): L2B
- Tài khoản GitHub: Long16082004
- Repo bài nộp (URL): https://github.com/Long16082004/K5-L23-DAY08-2A202603312-VuVanLong-AIPrelabel
- Commit chốt bài (hash): 699fde2 part 6 - calibration, QC, release, bao cao va quyet dinh Rework
- Máy: Windows 11 + WSL Ubuntu, không có GPU (CPU)
- CVAT (Docker): v2.74.1

## Kết quả tóm tắt (điền cuối bài; số lấy từ `submission/qc/*`, kèm mẫu số)

- Máy: accepted / edited / deleted: 506 / 5 / 10 (mẫu số: 521 box máy); added: 41 (mẫu số: 552 đối tượng)
- So với gold: precision / recall của **máy** 72,9% (51/70) / 64,6% (51/79)
- Bắt bẫy: 2/12 = 16,7%
- Thời gian: gán tay 6,25 phút/ảnh; duyệt có pre-label 6,25 phút/ảnh
- Lỗi QC: 100% theo ảnh (6/6), 8,1% theo đối tượng (10/123)
- **Quyết định:** Rework — ảnh lỗi 6/6 > 2%, lỗi hệ thống `thua` 9/10, bắt bẫy 2/12 < 80%
