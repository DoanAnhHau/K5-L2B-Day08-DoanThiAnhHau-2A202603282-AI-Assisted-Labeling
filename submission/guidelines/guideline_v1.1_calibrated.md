# Label contract — Phố châu Á — BỔ SUNG v1.1

> Bản bổ sung cho `guideline/guideline_v1.0_label_contract.md`. Áp dụng cho bộ 36 ảnh trong `data/images/`; giữ nguyên các quy tắc v1.0 không được nhắc lại ở đây.

## 1. Chốt các câu hỏi mở ở v1.0

| # | Câu hỏi | Quyết định v1.1 | Ví dụ trong bộ ảnh | Mã ca |
|---|---|---|---|---|
| 1 | Xe đạp điện hay xe máy điện? | Gán `motorbike` nếu xe có động cơ điện (tay ga/bàn đạp phụ không làm đổi lớp). Chỉ gán `bicycle` khi xe di chuyển bằng lực đạp, không có động cơ. Không xác định được từ ảnh thì gán lớp có khả năng cao nhất và tick `can_review`; không suy luận chỉ từ dáng khung xe. | img_011 (pilot; ảnh chạy thử có sẵn trong `work/smoke/`) | CAL-01 |
| 2 | Cụm xe gần camera, dày đặc | Vẫn gán riêng từng vật nhìn thấy và tách được đường bao chính, kể cả khi các box sát/chồng nhẹ. Dùng một `ignore_region` chỉ khi không thể xác định số cá thể hoặc ranh giới của ít nhất 5 vật trong một cụm; không dùng vì thao tác vẽ bất tiện. | img_011 | CAL-02 |
| 3 | Hàng hóa nhô khỏi xe | Box/polygon xe chỉ bao phần phương tiện và hàng được gắn cố định vào kết cấu xe. Hàng rời hoặc tải nhô ra không nhập vào nhãn xe, bất kể dài bao nhiêu; hàng hóa không thuộc taxonomy hiện tại nên không gán nhãn riêng. | img_011 | CAL-03 |
| 4 | Người thấp thoáng sau mui xe ba bánh | Chỉ gán `person` khi nhìn thấy phần cơ thể đủ để nhận diện người và box phần nhìn thấy đạt chiều cao tối thiểu 20 px. Không ước lượng phần trăm diện tích cơ thể bị che. Nếu có dấu hiệu là người nhưng chưa đủ điều kiện vẽ box, bỏ qua; tick `can_review` cho ca chưa chắc. | img_011 | CAL-04 |
| 5 | Kính/cửa sổ xe trong mask | Mask xe bao gồm khung, thân và các bề mặt kính thuộc cấu trúc xe; phần nhìn xuyên qua kính (người, nội thất hoặc cảnh nền) không thuộc mask. Vùng kính trong suốt vẫn nằm trong polygon bao ngoài của xe, không tạo lỗ. | img_011 | CAL-05 |
| 6 | Box xe máy chồng nhau trong ảnh đêm | Không gộp các xe nhận diện được thành một box. Mỗi xe là một instance; box bám phần nhìn thấy. Chỉ dùng `ignore_region` theo ngưỡng cụm tại câu 2 nếu không thể tách vật. Điểm `conf` thấp/cao không thay thế việc kiểm tra ảnh. | img_011 (ảnh pilot/ảnh smoke; kiểm ảnh gốc trước khi áp dụng) | CAL-06 |
| 7 | Ngưỡng conf để nhận box máy | Không đặt ngưỡng `conf` tối thiểu chung trong guideline. `conf` chỉ hỗ trợ sắp thứ tự review; mọi box phải được xác nhận theo ảnh và quy tắc hình học/lớp. Ngưỡng lọc trước khi đưa vào CVAT là recipe theo lớp của lần chạy, ghi trong `model_log.csv`, không phải tiêu chuẩn chấp nhận nhãn. | img_011 (`work/smoke/run.json`, conf 0.30) | CAL-07 |

## 2. Quy tắc áp dụng khi hiệu chỉnh nhãn máy

- Model COCO dự đoán `car`, `truck` hoặc `motorbike` cho xe ba bánh không làm thay đổi taxonomy: phân loại theo cấu tạo nhìn thấy và dùng `three_wheeler` cho tuk-tuk, xích lô, xe ba gác ba bánh. Nếu không xác định được cấu tạo thì tick `can_review`.
- Không chấp nhận box chỉ vì điểm `conf` cao. Kiểm tra đủ lớp, vật bị bỏ sót, box ôm phần nhìn thấy, vật nhỏ dưới 20 px và vùng che khuất theo checklist v1.0.
- Không gộp các instance chỉ để giảm chồng lấn. Chồng lấn giữa box người và xe, hoặc giữa xe ở góc nhìn thực tế, được giữ khi phản ánh đúng ảnh.

## 3. Bổ sung từ các ca đã rà soát (v1.1)

| Mã ca | Nguồn / ảnh | Quan sát và quy tắc áp dụng |
|---|---|---|
| CAL-08 | `img_013.jpg`, lỗi máy r05 trong `model_log.csv` | YOLO26 gán tuk-tuk thành `car` (conf 0.42). Không thể sửa nhầm lớp bằng cách tăng/giảm ngưỡng; dựa cấu tạo ba bánh để gán `three_wheeler`, và đánh dấu `can_review` nếu không xác định được. |
| CAL-09 | `img_031.jpg`, QC box ID 391; câu hỏi mở v1.0 mục 6 câu 3 | Box xe đạp chở hàng không gộp hàng rời/trái cây nhô khỏi kết cấu xe. Chỉ phần phương tiện và bộ phận gắn cố định thuộc box xe; hàng rời không có lớp trong taxonomy thì không gán riêng. |
| CAL-10 | `img_005.jpg`, bẫy `missing` lọt trong `review_stats.md` | Trước khi xem box, quét độc lập toàn ảnh theo tiền cảnh/nền và các mép ảnh để tìm vật bị bỏ sót. Rà box có sẵn không thể phát hiện vật không có box. |

## 4. Lịch sử phiên bản

| Phiên bản | Thay đổi | Lý do |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | Chốt câu hỏi mở về xe điện, cụm xe dày, hàng rời, người bị che, kính xe và conf; bổ sung quy tắc cho lỗi lớp xe ba bánh, box hàng hóa và quét tìm vật bị bỏ sót. | CAL-08 (`img_013`), CAL-09 (`img_031`, ID 391), CAL-10 (`img_005`); chi tiết trong `submission/qc/calibration_decisions.csv`. |
