# Label contract — Phố châu Á — Bổ sung v1.1

## Câu hỏi mở đã chốt

| Ca | Câu hỏi v1.0 | Quy tắc bổ sung | Ảnh ví dụ |
|---|---|---|---|
| CAL-02 | §6.2 — Xe máy đỗ sát nhau gần camera | Gán riêng từng xe nếu box cao từ 20 px và có thể xác định phần nhìn thấy theo §4.1. Chỉ khoanh `ignore_region` cho phần cụm gồm ít nhất 5 vật sát nhau có box dưới 20 px. Tiếp tục gán riêng các xe đủ điều kiện ở ngay cạnh cụm. | img_005.jpg |

Các câu hỏi mở còn lại chưa chốt trong lần hiệu chỉnh tham chiếu này.

## Ca khó và điểm kiểm tra

| Ca | Ảnh ví dụ | Tình huống | Quy tắc bổ sung |
|---|---|---|---|
| CAL-01 | img_013.jpg | Trong lần thử YOLO26, tuk-tuk có mui được đề xuất là `car`. | Kiểm tra kiểu khung và phần bánh thấy được trước khi chấp nhận lớp. Dùng `three_wheeler` khi nhận diện được xe ba bánh, kể cả xe có mui. Giữ riêng box người đủ điều kiện. |
| CAL-03 | img_003.jpg | Bẫy box lỏng còn sót sau review tham chiếu. | Ở cuối lượt duyệt, rà riêng cả bốn cạnh. Khi một cạnh ôm thêm nền, kéo cạnh đó về điểm ngoài cùng của phần vật nhìn thấy theo §4.1. |

## Lịch sử thay đổi

v1.1 làm rõ phạm vi của `ignore_region` ở cụm hỗn hợp, dấu hiệu xác nhận xe ba bánh và thao tác kiểm tra box lỏng. Các ca tương ứng là CAL-01, CAL-02 và CAL-03.
