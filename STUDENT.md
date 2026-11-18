# Thông tin học viên

- Họ và tên: Doan Thi Anh Hau
- MSHV: 2A202603282
- Lớp: L2B (suy ra từ tên workspace)
- Tài khoản GitHub: DoanAnhHau (theo remote Git)
- Repo bài nộp: https://github.com/DoanAnhHau/K5-L2B-Day08-DoanThiAnhHau-2A202603282-AI-Assisted-Labeling
- Commit chốt bài tại thời điểm rà soát: `00fbe1a` (working tree còn thay đổi chưa commit)
- Máy: Windows; các run model ghi CPU, không dùng CUDA.
- CVAT (Docker): phiên bản chưa được ghi trong artifact.

## Kết quả tóm tắt (nguồn: `submission/qc/`)

- Máy: accepted 384/430 = 89.3%, edited 19/430 = 4.4%, deleted 24/430 = 5.6% (mẫu số: 430 box máy); added 34/438 = 7.8% (438 đối tượng sau duyệt).
- So với gold: precision máy 42/62 = 67.7%; recall máy 42/57 = 73.7% (`review_stats.md`).
- Bắt bẫy: 11/12 = 91.7%; lọt một bẫy missing ở `img_005.jpg`.
- Thời gian gán tay 8/4 = 2 phút/ảnh và review 10/24 ≈ 0.42 phút/ảnh (ước tính hồi cứu do người học cung cấp). SAM: 0.43 phút runtime cho 3 ảnh, suy từ terminal 8.55 giây/ảnh; chưa gồm đặt click/kiểm tra. QC/calibration/report: 45 phút ước tính hồi cứu.
- Lỗi QC: 80.0% theo ảnh (4/5), 6.4% theo đối tượng (6/94).
- **Quyết định:** Rework — 4/5 ảnh lỗi vượt ngưỡng 2%; cần sửa các box được liệt kê trong `qc_review.csv` rồi chạy vòng QC mới.
