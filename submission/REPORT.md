# Báo cáo Day 08 — 03308 - Nguyễn Hữu Thịnh

Dữ liệu, mask, nhật ký và số đo trong báo cáo này là kết quả có sẵn trong bộ dữ liệu mẫu. Các quyết định calibration bên dưới được đề xuất từ những ca trong bộ tham chiếu. Các mã lần chạy r02–r05 và thời gian ở mục 4 lấy từ nhật ký dữ liệu mẫu trong `submission/`. Chưa có nhật ký đo riêng cho 03308.

## 1. Model và recipe (Part 2–3)

| Lớp | Model | Ngưỡng gợi ý | Lý do |
|---|---|---|---|
| car | YOLO26m (r02, r03) | 0.50 | Recall 7/7 trên gold trong lần quét |
| person, motorbike | YOLO26m (r02, r03) | 0.30 | Các ngưỡng thử đến 0.50 chưa đạt mục tiêu recall 90% |
| bicycle, bus, truck | YOLO26m (r02, r03) | 0.30 | Mỗi lớp dưới 5 đối tượng gold |
| three_wheeler | YOLOE-26s (r04), prompt `three_wheeler` | 0.30 | Recall tốt nhất trong lần quét là 7/14 (50%) |

`qc/threshold_sweep.md` gợi ý `--thr "car=0.5" --conf 0.3`. Báo cáo nguồn ghi lệnh tạo bộ tham chiếu dùng một bộ ngưỡng khác:

```powershell
python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "person=0.35,bicycle=0.3,motorbike=0.35,car=0.4,bus=0.4,truck=0.4,three_wheeler=0.25" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json
```

Lần thử r05 dùng SAM 2.1 (`sam2.1_b.pt`) để kiểm tra mask sinh từ box. Ở `img_013.jpg`, YOLO26 gọi tuk-tuk là `car` với điểm 0.42. Đổi ngưỡng chỉ thay đổi việc giữ box, còn lớp phải được xác nhận từ ảnh hoặc đề xuất lại bằng YOLOE. Nhật ký nguồn ghi YOLOE tạo một box xe ba bánh điểm 0.40 ở `img_002.jpg` nhưng cắt mất mép phải. Ngưỡng được chọn và đánh giá trên cùng 4 ảnh gold nên số đo có thể lạc quan.

## 2. AI error profile

Theo `qc/review_stats.md`, đối chiếu pre-label 03308 với nhãn batch có sẵn sau khi loại 23 đối tượng quanh bẫy:

| Trạng thái | Số / mẫu số | Tỉ lệ |
|---|---|---|
| accepted | 442/489 box máy | 90,4% |
| edited | 17/489 box máy | 3,5% |
| deleted | 20/489 box máy | 4,1% |
| added | 11/470 đối tượng sau duyệt | 2,3% |

Ở lớp `three_wheeler`, 10/25 box máy được giữ, 8/25 được chỉnh và 7/25 bị xóa. Người duyệt thêm 6 box, chiếm 6/11 lượt thêm trên toàn bộ các lớp. Box gọi nhầm tuk-tuk trong `img_013.jpg` là một ví dụ ở lần thử model riêng. Trên gold, precision là 34/60 = 56,7% và recall là 34/55 = 61,8%.

## 3. Thiên kiến tự động hóa và ảnh bẫy

Khi đối chiếu bộ bẫy 03308 với nhãn batch có sẵn, 11/12 bẫy (91,7%) được khắc phục theo tiêu chí của script. Kết quả theo loại: loose 3/3, wrong_class 3/3, extra 3/3, missing 2/3. Trường hợp còn lệch là missing ở img_013.jpg. Số đo phản ánh mức khớp với nhãn tham chiếu; chưa có lượt duyệt mù riêng để đánh giá thiên kiến của học viên.

## 4. Máy có đáng dùng không?

Nhật ký tham chiếu ghi gán tay 21 phút cho 4 ảnh (5,25 phút/ảnh) và duyệt 72 phút cho 24 ảnh (3,00 phút/ảnh). Đây là hai lô khác nhau, nên chênh lệch còn phụ thuộc mật độ vật và mức độ quen cảnh. Kết quả gợi ý dùng máy để đề xuất box, nhất là lớp phổ biến, và ưu tiên kiểm tra xe ba bánh.

Ba quy tắc kiểm tra khi duyệt:

1. Ẩn toàn bộ box máy và quét từng vùng ảnh trước khi bật box trở lại.
2. Tìm vật bị bỏ sót theo từng lớp, đặc biệt ở mép ảnh và vùng có xe che nhau.
3. Sau khi xác nhận lớp, phóng to và kiểm tra cả bốn cạnh box theo phần vật nhìn thấy.

## 5. Mask SAM cho 3 vật

Các quan sát và số click sau lấy từ báo cáo nguồn.

| Ảnh | Phiên bản | Điểm dương/âm | Quan sát | File |
|---|---|---|---|---|
| img_013.jpg | SAM 2.1 | 2/1 | Mask dính người, hàng và một phần nền | `tuktuk_img013.jpg` |
| img_021.jpg | SAM 2.1 | 2/1 | Vùng bánh nan hoa có khoảng hở cần đối chiếu quy tắc lỗ của mục 4.2 | `threewheeler_img021.jpg` |
| img_005.jpg | SAM 2.1 | 1/0 | Mask lấy thân xe, thiếu gương | `motorbike_img005.jpg` |

Dùng mục 4.2 để kiểm phần vật nhìn thấy, lỗ lớn và bóng đổ. Với tuk-tuk, kiểm mép mái và xử lý riêng người lái khi mask xe dính sang người.

## 6. Calibration

Ba quyết định trong `qc/calibration_decisions.csv` gồm cách nhận diện xe ba bánh khi detector gọi nhầm lớp (CAL-01), cách xử lý xe máy đỗ dày gần camera (CAL-02) và cách rà biên theo từng cạnh khi gặp bẫy box lỏng (CAL-03). Mỗi quyết định có ảnh ví dụ trong guideline v1.1.

## 7. QC và quyết định

Đã dựng lại release candidate từ nhãn batch sẵn có: 24 ảnh, 491 đối tượng gồm cả ignore_region. Codex rà trực quan 5 ảnh ngẫu nhiên (seed 42), gồm 99 đối tượng tính QC. Ghi nhận một lỗi box dư nền ở xe máy #138 trong img_014.jpg: 1/99 đối tượng (1,0%) và 1/5 ảnh (20,0%). Cận trên Wilson 95% theo ảnh xấp xỉ 62%. Việc rà ảnh có box hỗ trợ phát hiện lỗi dễ thấy, còn vật nhỏ hoặc bị che trong ảnh đông vẫn có thể bị bỏ sót.

**Quyết định: Rework**, ngày 2026-11-19. Tỷ lệ ảnh lỗi quan sát vượt ngưỡng 2%. Cần chỉnh box đã ghi trong phiếu và kiểm lại mẫu. Bộ bẫy 03308 được đối chiếu với nhãn có sẵn từ lô cũ.

## 8. Ghi chú cá nhân

Codex tổng hợp file tham chiếu, kiểm tra cấu trúc COCO và đối chiếu phiếu QC với báo cáo tự động. Chưa có bằng chứng xác nhận thời điểm gán gold của 03308, môi trường CVAT dùng để gán, hoặc lần chạy model độc lập của học viên. Ở vòng thực hành cá nhân, nên ghi thời gian ngay khi bắt đầu và kết thúc mỗi task, đồng thời lưu tên ảnh khi phát hiện ca khó.
