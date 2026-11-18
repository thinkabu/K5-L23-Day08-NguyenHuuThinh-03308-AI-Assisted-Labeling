# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 489; đối tượng sau duyệt: 470; ngưỡng accepted: IoU >= 0.9
- Đã **loại 23 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 442 | 442/489 = 90.4% | 489 box máy |
| edited (sửa) | 17 | 17/489 = 3.5% | 489 box máy |
| deleted (xoá) | 20 | 20/489 = 4.1% | 489 box máy |
| added (vẽ thêm) | 11 | 11/470 = 2.3% | 470 đối tượng sau duyệt |

Trong số deleted + added, **0 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 11.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 262 | 2 | 6 | 1 |
| bicycle | 4 | 1 | 3 | 1 |
| motorbike | 122 | 4 | 2 | 3 |
| car | 36 | 2 | 0 | 0 |
| bus | 3 | 0 | 0 | 0 |
| truck | 5 | 0 | 2 | 0 |
| three_wheeler | 10 | 8 | 7 | 6 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 55 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 34 | 26 | 21 | 56.7% (KTC 95%: 44–68%) | 61.8% (KTC 95%: 49–73%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 11/12 = 91.7% (KTC 95%: 65–99%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - loose: 3/3
  - wrong_class: 3/3
  - missing: 2/3
  - extra: 3/3
  - lọt: img_013.jpg (missing)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
