# Báo cáo Day 08 — 2A202603282 - Doan Thi Anh Hau

> Tổng hợp recipe, QC và ghi chú. Số liệu lấy từ các file trong `submission/`; phần thời gian không có log nên không ước đoán.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (model log / threshold sweep) |
|---|---|---|---|
| person / bicycle / motorbike / car / bus / truck | YOLO26m | motorbike 0.25; car 0.50; các lớp còn lại mặc định 0.30 | Ngưỡng motorbike đạt recall 92% (12 vật gold); car đạt recall 90%, precision 69% (10 vật gold). person và bicycle không đạt recall mục tiêu ở ngưỡng ≤0.50; bus/truck có quá ít vật gold để chọn ngưỡng riêng. `threshold_sweep.md`. |
| three_wheeler | YOLOE-26s, prompt `tuk-tuk`, `cyclo`, `cargo tricycle` | 0.15 trong sweep gold; pilot r04 thử 0.25 | Sweep gold ghi recall 100%, precision 78% (7 vật gold). YOLOE bổ sung lớp COCO không có; ở pilot r04 có box đúng 0.93 nhưng cũng có box thừa 0.37/0.40. |

- Lệnh gộp theo recipe được ghi trong GUIDE/threshold sweep: `python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "motorbike=0.25,car=0.5,three_wheeler=0.15" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json`. Chuỗi ngưỡng theo `submission/qc/threshold_sweep.md`; file đầu ra được dùng tạo pre-label và bẫy.
- **SAM chọn:** SAM 2.1 base (`sam2.1_b.pt`). Có weights cục bộ và chạy được CPU; xuất mask overlay có click dương/âm để rà trực quan. Ba ảnh chính và kết quả polygon nằm trong `submission/masks/`; thời gian thực tế tổng hợp cho mỗi mask không được ghi đầy đủ.
- Lỗi không sửa được bằng ngưỡng: xe ba bánh bị dự đoán `car`/`truck` là nhầm ngữ nghĩa, không phải vấn đề độ nhạy. Dùng YOLOE để đề xuất `three_wheeler`, rồi người duyệt xác định lớp theo cấu tạo; ví dụ `img_013.jpg`, r05.
- Ví dụ conf không thay thế việc kiểm ảnh: YOLOE pilot r04 có box `three_wheeler` đúng 0.93 ở `img_002.jpg` nhưng cũng có box nhầm 0.37/0.40; YOLO26 smoke r01 có tuk-tuk bị gọi truck. Xem `model_log.csv`.
- Giới hạn gold: precision 42/62 = 67.7%, recall 42/57 = 73.7% (`review_stats.md`). Chỉ 4 ảnh gold/57 đối tượng, đồng thời chọn ngưỡng trên chính gold nên kết quả in-sample lạc quan và kém ổn định.

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---:|---:|---|
| accepted | 384 | 384/430 = 89.3% | 430 box máy |
| edited | 19 | 19/430 = 4.4% | 430 box máy |
| deleted | 24 | 24/430 = 5.6% | 430 box máy |
| added | 34 | 34/438 = 7.8% | 438 đối tượng sau duyệt |

- Ba trạng thái đầu là box máy; added là đối tượng sau duyệt. Hai trường hợp deleted/added là đổi lớp, nên số vật máy sót thật ước tính 32/438 đối tượng sau duyệt.
- Lớp khó nhất là `three_wheeler`: có 5 edited, 7 deleted và 6 added trên lô review; model COCO không có lớp này và YOLOE có false positive ở pilot. Với `motorbike`, 11/89 box máy cần sửa theo lớp thống kê.
- So với gold, máy có precision 42/62 = 67.7% (TP+FP = 62 box máy), recall 42/57 = 73.7% (57 đối tượng gold); KTC 95% lần lượt 55–78% và 61–83%. Chỉ 4 ảnh gold, ngưỡng được chọn trên cùng tập đó.

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt được 11/12 bẫy = 91.7%; lọt 1/12, loại `missing` ở `img_005.jpg` (`review_stats.md`). Điều này cho thấy chỉ rà box có sẵn không đủ để thấy vật bị bỏ sót; cần quét toàn ảnh trước khi chấp nhận pre-label.
- Accepted cao 384/430 = 89.3% không chứng minh việc duyệt bắt đủ lỗi: bẫy missing vẫn lọt 1/12. Mức bắt bẫy vượt ngưỡng 80% nhưng mẫu 12 bẫy nhỏ, KTC 95% là 65–99%.

## 4. Máy có đáng dùng không?

- `submission/time_log.csv`: gán gold 8 phút/4 ảnh = 2 phút/ảnh; batch review 10 phút/24 ảnh ≈ 0.42 phút/ảnh; đây là ước tính hồi cứu do người học cung cấp. SAM ghi 0.43 phút cho 3 ảnh, suy từ 8.55 giây/ảnh trong terminal/model log r05 (chưa tính đặt click/kiểm tra). QC/calibration/report là ước tính hồi cứu 45 phút, không phải thời gian thuần từng ảnh.
- Kết luận: dùng model như pre-label để tiết kiệm thao tác ban đầu, không dùng làm nhãn cuối nếu chưa rà từng box và quét vật bị sót. Căn cứ: accepted 384/430 = 89.3%, nhưng 24/430 = 5.6% box máy bị xóa, 19/430 = 4.4% bị sửa và 34/438 = 7.8% đối tượng sau duyệt được thêm.
- Ba cách giảm thiên kiến có thể kiểm tra ở vòng sau:
  1. Quét ảnh một lượt không nhìn box trước khi duyệt; ghi số vật nghi bị sót và đối chiếu sau lượt review.
  2. Rà toàn bộ box theo lớp/đường bao, không accept theo conf; lưu số box edited/deleted theo lớp.
  3. Dùng ít nhất 12 ca bẫy, giữ lại loại bẫy bị lọt và tính caught/12 sau mỗi vòng.

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Click dương / âm (tọa độ pixel) | Polygon | Mask gồm người lái / hàng hóa? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---:|---|---|---|
| img_004 | SAM 2.1 base | + (620,480), (700,560); − (465,450) | 1 | Không thấy người/hàng hóa dính rõ | Vùng xe liền mạch, nhiễu nhẹ ở rìa/đáy | `tuktuk_img004_sam2.jpg` |
| img_013 | SAM 2.1 base | + (600,110), (400,480); − (720,300) | 1 | Có dính người lái; click âm giảm phần thân nhưng đầu/chân còn trong mask | Xe và người chồng lấn, cần thêm click âm hoặc box | `tuktuk_img013_sam2.jpg` |
| img_017 | SAM 2.1 base | + (560,420), (680,500); − (645,355) | 1 | Không thấy người/hàng hóa dính rõ | Mask bám tuk-tuk tiền cảnh; nhiễu cạnh nhẹ | `tuktuk_img017_sam2.jpg` |

Terminal của ba lần chạy chính báo 1 polygon/ảnh. Thử thêm click âm (735,225) và (760,430) trên người lái ở `img_013` tạo 2 polygon nhưng cắt nhầm mái/khung và thân xe nên không chọn. Ảnh SAM 1 `tuktuk_img013_sam1.jpg` cho thấy người lái dính mask rõ hơn bản 2.1. Quy tắc 4.2 về đường biên và vật chạm nhau hữu ích; còn thiếu thao tác lặp click âm có kiểm tra trực quan hoặc box giới hạn để tách người khỏi xe.

- CVAT Interactor: không có log xác nhận đã thử thành công; phần mask trong báo cáo là kết quả chạy offline bằng `sam2_points.py`.

## 6. Calibration (Part 6)

- Có 3 quyết định trong `calibration_decisions.csv`: một `loi_may`, một `cau_hoi_mo` mục 6 v1.0, và một `bay_lot`.
- Trọng tâm CAL-09 (`img_031.jpg`, box ID 391): box phương tiện không bao hàng rời nhô khỏi xe. CAL-08 dùng lỗi xe ba bánh bị gọi `car` ở `img_013.jpg`; CAL-10 dùng bẫy missing lọt ở `img_005.jpg`.
- Các bổ sung và lịch sử `v1.0 → v1.1` được ghi ở `guideline_v1.1_calibrated.md` mục 3–4. Nguyên nhân gồm giới hạn semantic của model và câu hỏi mở của guideline; lỗi missing còn phản ánh quy trình review box-centric.

## 7. QC và quyết định (Part 6)

- Mẫu: 5 ảnh ngẫu nhiên, seed 42, 0 ảnh `can_review`; tổng 94 đối tượng (`qc_sample.csv`). Các overlay trong `work/qc/review/` được rà theo object ID. Reviewer ghi trong `qc_review.csv`; ghi chú hình học cần người nộp xác nhận trên ảnh gốc.
- Lỗi theo đối tượng: 6/94 = 6.4%; ảnh có lỗi: 4/5 = 80.0% (ngưỡng Release ≤ 2%). `img_014.jpg` được ghi `khong`.
- Loại lỗi duy nhất ghi nhận: sai vị trí/kích thước 6/6, ở `img_003` (ID 10), `img_006` (IDs 69, 77), `img_031` (ID 391), `img_035` (IDs 433, 434). Có box rộng/gộp xe, box nhập hàng rời và box tuk-tuk bị chia.
- Cận trên Wilson 95% cho 4/5 ảnh có lỗi là 96% (`qc_report_auto.md`). Mẫu 5 ảnh quá nhỏ để kết luận tỉ lệ thật đạt ngưỡng 2%.
- Script đếm và tổng hợp phiếu, nhưng không xác định box có ôm sát vật thật hay cùng bỏ sót với máy; các việc đó cần mắt người và checklist.

**Quyết định: Rework**, ngày 2026-11-18. Lý do: ảnh lỗi 4/5 = 80.0% vượt ngưỡng 2%, và sai vị trí/kích thước lặp ở 6/6 lỗi. Bắt bẫy 11/12 = 91.7% đạt ngưỡng 80%, nhưng không bù được lỗi QC. Tiếp theo: xác nhận/sửa các ID trong `qc_review.csv` trên ảnh gốc, cập nhật nhãn nguồn, dựng release mới và lấy mẫu QC vòng tiếp theo.

## 8. Ghi chú cá nhân

- AI hỗ trợ tra cứu artifact, đọc overlay/box ID, đối chiếu guideline, điền bản nháp QC/calibration/report; người nộp cần xác nhận các đánh giá bằng ảnh gốc. Các dòng QC có ghi reviewer trong CSV.
- Không có log chứng minh gold có hay không bị xem pre-label trước khi gán xong; trạng thái nhiễm gold chưa xác định được từ artifact hiện có.
- Các run YOLO/SAM ghi `device=cpu` (`work/prelabel/*/run.json`); máy là Windows theo workspace. Phiên bản Docker/CVAT không được ghi trong submission.
- Không thấy bằng chứng dùng `data/cache/`; recipe đọc các run trong `work/prelabel/`.
- Vướng mắc dữ liệu: không bấm giờ trong lúc làm; thời gian trong `time_log.csv` chỉ là ước tính hồi cứu.
- Lần sau sẽ ghi thời gian thực cho từng bước và quét toàn ảnh trước khi nhìn box máy để tránh lọt bẫy missing.
