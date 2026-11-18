# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603302 - Phạm Thị Lương

> **Chỉ viết phần thay đổi so với v1.0** (không copy cả file). Chốt các câu hỏi mở ở mục 6 của v1.0 (hoặc nói rõ câu nào chưa chốt, vì sao)
> và thêm quy tắc mới rút ra từ lỗi của máy / lỗi của bạn. Mỗi ca khó **có ảnh ví dụ** (`img_0xx`) và khớp một dòng trong `qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 1 | Xe ba bánh chở hàng cồng kềnh/xích lô: phân loại vào `car` hay `three_wheeler`? | Gán thống nhất vào lớp `three_wheeler`, sử dụng YOLOE hoặc chỉnh sửa thủ công để tránh nhầm với `car`. | img_013.jpg | CAL-02 |

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của bạn | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| img_005.jpg | Cụm phương tiện (xe máy) quá dày đặc, kích thước dưới 20px ở ngã tư | YOLO26m vẽ quá nhiều box rời rạc bị lỗi lỏng/thừa | Dùng `ignore_region` bao trùm toàn bộ cụm thay vì vẽ từng vật thủ công | Bổ sung quy định dùng `ignore_region` cho cụm phương tiện nhỏ dưới 20px mật độ cao |
| img_021.jpg | Box phương tiện bị lệch biên, ôm không sát do góc khuất và chồng chéo | YOLO26m sinh box có tọa độ lỏng lẻo (`loose`) | Kéo chỉnh thủ công, dịch chỉnh khung ôm khít biên thực tế của phương tiện | Chuẩn hóa quy tắc bounding box phải ôm sát biên vật thể |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | Chốt nhãn cho `three_wheeler`, bổ sung quy tắc xử lý cụm phương tiện nhỏ dày đặc bằng `ignore_region` và chuẩn hóa biên box. | CAL-01, CAL-02, CAL-03 |