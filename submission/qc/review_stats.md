# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 430; đối tượng sau duyệt: 438; ngưỡng accepted: IoU >= 0.9
- Đã **loại 29 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 384 | 384/430 = 89.3% | 430 box máy |
| edited (sửa) | 19 | 19/430 = 4.4% | 430 box máy |
| deleted (xoá) | 24 | 24/430 = 5.6% | 430 box máy |
| added (vẽ thêm) | 34 | 34/438 = 7.8% | 438 đối tượng sau duyệt |

Trong số deleted + added, **2 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 32.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 239 | 2 | 8 | 7 |
| bicycle | 5 | 0 | 3 | 3 |
| motorbike | 77 | 11 | 1 | 10 |
| car | 45 | 1 | 3 | 7 |
| bus | 3 | 0 | 0 | 0 |
| truck | 5 | 0 | 2 | 1 |
| three_wheeler | 10 | 5 | 7 | 6 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 57 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 42 | 20 | 15 | 67.7% (KTC 95%: 55–78%) | 73.7% (KTC 95%: 61–83%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 11/12 = 91.7% (KTC 95%: 65–99%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - missing: 2/3
  - loose: 3/3
  - wrong_class: 3/3
  - extra: 3/3
  - lọt: img_005.jpg (missing)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
