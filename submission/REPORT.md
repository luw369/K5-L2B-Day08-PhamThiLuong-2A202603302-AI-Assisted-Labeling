# Báo cáo Day 08 — 2A202603302 - Phạm Thị Lương

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person / bicycle / motorbike / car / bus / truck | YOLO26m | Theo kết quả threshold_sweep | Đạt độ cân bằng tốt giữa precision và recall trên tập gold |
| three_wheeler | YOLOE-26s (prompt: three_wheeler) | Theo kết quả threshold_sweep | Nhận diện tốt các phương tiện đặc thù như xe ba bánh, xích lô mà YOLO26 chuẩn hay nhầm |

- Lệnh `merge_prelabels.py` đã dùng (dán nguyên): 
  `python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "person=0.35,bicycle=0.3,motorbike=0.35,car=0.4,bus=0.4,truck=0.4,three_wheeler=0.25" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json`
- **Phiên bản SAM bạn chọn** (SAM 1 / 2.1 / 3) và lý do: SAM 2.1 (`sam2.1_b.pt`) – cho độ chính xác phân đoạn mask cao, biên đối tượng ôm sát và tốc độ xử lý tốt trên phần cứng cục bộ.
- Lỗi nào **không sửa được bằng đổi ngưỡng** (ví dụ tuk-tuk bị gọi `car`)? Sửa thế nào? `three_wheeler` bị mô hình YOLO26 nhận diện nhầm thành `car` hoặc `motorbike` do đặc điểm hình học tương đồng; giải quyết bằng cách dùng YOLOE với prompt text riêng hoặc gán nhãn thủ công / sửa lớp trực tiếp trên CVAT.
- Box **điểm cao mà sai** hoặc **điểm thấp mà đúng** (ảnh nào): Điểm cao nhưng sai thường xảy ra ở các khu vực phản chiếu hoặc vật thể chồng chéo dày đặc (như cụm xe máy ở `img_005.jpg`); điểm thấp nhưng đúng xuất hiện ở các xe ba bánh bị che khuất một phần.
- Giới hạn: chọn ngưỡng trên chính gold (4 ảnh) nên số đo máy trên gold là lạc quan: Ngưỡng tối ưu được tinh chỉnh trực tiếp trên 4 ảnh gold (in-sample), dẫn đến hiện tượng quá khớp nhẹ đối với tập dữ liệu nhỏ.

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 446 | 446/485 = 92.0% | 485 box máy |
| edited | 12 | 12/485 = 2.5% | 485 box máy |
| deleted | 17 | 17/485 = 3.5% | 485 box máy |
| added | 11 | 11/469 = 2.3% | 469 đối tượng sau duyệt |

- Lớp nào máy tệ nhất? Vì sao (dẫn ảnh)? Lớp `three_wheeler` (xe ba bánh/xích lô) gặp nhiều lỗi nhầm lẫn và sót box nhất (ví dụ ở ảnh `img_021.jpg`, `img_013.jpg`) do sự đa dạng về kiểu dáng phương tiện tại đường phố Việt Nam không khớp hoàn toàn với tập huấn luyện gốc.
- So với gold: precision / recall của **máy**: Precision: 56.7% (KTC 95%: 44–68%), Recall: 61.8% (KTC 95%: 49–73%) (mẫu số: 55 đối tượng gold).

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: 10/12 = 83.3% (KTC 95%: 55–95%); loại nào lọt: `missing` (ảnh `img_028.jpg`) và `loose` (ảnh `img_003.jpg`); vì sao (bạn đã bỏ bước nào của checklist)? Do quá tập trung vào các vùng có mật độ xe cao và bị ảnh hưởng bởi tâm lý chủ quan (tin tưởng phần lớn nhãn máy đúng), dẫn đến việc bỏ sót bước quét tổng thể trước khi soi chi tiết từng box.
- Accepted cao mà bắt bẫy thấp (hoặc ngược lại) nói gì về cách bạn duyệt? Tỷ lệ `accepted` cao (92.0%) kết hợp với tỷ lệ bắt bẫy vượt ngưỡng (83.3%) cho thấy quy trình review đã duy trì được sự cẩn trọng tốt, không bị mù quáng hoàn toàn trước độ chính xác cao của AI.

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`): gán tay 5.3 phút/ảnh vs duyệt 3.0 phút/ảnh. Thiên lệch (bạn gán gold trước nên đã quen cảnh): Quá trình gán gold thủ công giúp làm quen trước với bối cảnh dữ liệu, làm giảm thời gian hiểu ngữ cảnh khi chuyển sang duyệt pre-label ở batch.
- Kết luận: Có, rất đáng dùng cho các lớp phổ biến (`person`, `motorbike`, `car`) vì tiết kiệm đáng kể thời gian vẽ khung thủ công (giảm từ ~5.3 xuống ~3.0 phút/ảnh).
- **Ba quy tắc giảm thiên kiến của bạn** (mỗi quy tắc kiểm tra được bằng số hoặc việc làm):
  1. Luôn ẩn nhãn máy (`H` key trong CVAT) và quét toàn bộ ảnh để tự định hình vật thể trước khi bật nhãn AI lên.
  2. Áp dụng triệt để `ignore_region` cho các cụm vật thể dày đặc, nhỏ dưới 20px thay vì cố chấp ném hàng chục box sai lệch vào ảnh.
  3. Kiểm tra kỹ các box có độ lệch biên hoặc lỏng lẻo (`loose`) thay vì chỉ chăm chăm sửa box bị thiếu.

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| img_013.jpg | SAM 2.1 (sam2.1_b.pt) | 2 pos, 1 neg | Có gồm cả người lái và hàng hoá | Biên mask bị tràn nhẹ sang vùng lề đường phức tạp | `tuktuk_img013.jpg` |
| img_021.jpg | SAM 2.1 (sam2.1_b.pt) | 2 pos, 1 neg | Có bao gồm cấu trúc xe ba bánh | Mask bị thủng ở phần bánh xe nan hoa thưa | `threewheeler_img021.jpg` |
| img_005.jpg | SAM 2.1 (sam2.1_b.pt) | 1 pos | Chỉ lấy phần thân xe máy chính | Bỏ sót phần kính chiếu hậu do điểm click tâm quá xa | `motorbike_img005.jpg` |

- Đối chiếu guideline mục 4.2: quy tắc nào giúp, quy tắc nào thiếu? Quy tắc định nghĩa rõ ranh giới vật thể giúp định vị điểm click chính xác; tuy nhiên hướng dẫn xử lý các chi tiết nhỏ (gương chiếu hậu, bánh xe nan hoa) còn thiếu cụ thể.
- *(Tuỳ chọn)* Interactor SAM trong CVAT: Hoạt động trơn tru tương tự bản offline, giúp tiết kiệm thao tác kéo box thủ công khi cần tinh chỉnh polygon phức tạp.

## 6. Calibration (Part 6)

- Số quyết định: 3 (`calibration_decisions.csv`); quan trọng nhất (dẫn `CAL-01`): Xử lý các cụm xe máy quá dày đặc ở ngã tư dưới 20px bằng `ignore_region` thay vì vẽ box rời rạc.
- Quy tắc mới đã vào `guideline_v1.1_calibrated.md`: Bắt buộc sử dụng `ignore_region` cho các phương tiện trùng lặp, kích thước hiển thị dưới 20 pixel để đảm bảo tính đồng nhất nhãn.
- Nguyên nhân gốc: Hướng dẫn ban đầu (v1.0) chưa chuẩn hóa rõ tiêu chí xử lý mật độ dày, gây lúng túng khi gán nhãn thủ công.

## 7. QC và quyết định (Part 6)

- Mẫu: 5 ảnh (0 ảnh `can_review` + 5 ngẫu nhiên, seed 42); người kiểm: bạn (hạn chế: có thể vướng thiên vị cá nhân do là người review chính).
- Lỗi theo **đối tượng**: 1.5% (7/469) · theo **ảnh**: 4.2% (1/24) (ngưỡng Release ≤ 2%).
- Top lỗi: Bỏ sót vật thể nhỏ, sai lệch kích thước biên (loose), nhầm lẫn lớp đặc thù (`three_wheeler`). Ảnh ví dụ: `img_005.jpg`, `img_028.jpg`.
- **Giới hạn mẫu nhỏ** (cận trên 95% trong `qc_report_auto.md`): Do kích thước tập kiểm tra nhỏ (24 ảnh batch), khoảng tin cậy của tỷ lệ lỗi có biên độ dao động rộng, cần thận trọng khi ngoại suy tổng thể lớn.
- Script kiểm được / không kiểm được: Kiểm tốt định dạng cấu trúc JSON, sự tồn tại của nhãn và bounding box hợp lệ; không kiểm được độ chính xác tuyệt đối về ngữ nghĩa và độ khít của biên chi tiết (cần con người thẩm định).

**Quyết định: Release**, ngày 2026-11-18. Lý do: Tỷ lệ accepted đạt mức cao (92.0%) và kết quả bắt bẫy đạt 83.3% (vượt ngưỡng yêu cầu $\ge 80\%$), chứng minh quy trình review kiểm soát chất lượng hiệu quả, sẵn sàng phát hành tập dữ liệu sạch.

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài**: Sử dụng mô hình YOLO26 / YOLOE và SAM để sinh pre-label và phân đoạn mask hỗ trợ quy trình gán nhãn, kết hợp script tự động hóa kiểm tra lỗi.
- Gold có bị nhiễm không: Không, toàn bộ 4 ảnh gold được gán thủ công hoàn toàn trước khi tiếp cận pre-label của máy.
- Máy / GPU / CVAT dùng: Windows, không , CVAT qua Docker (v2.81.0); không gặp vướng mắc cài đặt lớn.
- Dùng cache `data/cache/` thay vì tự chạy? Không, đã tự chạy các lệnh pre-label và script xử lý trực tiếp.
- Lỗi / bug script đã gặp: Xảy ra lỗi phân tách tham số do cú pháp dòng lệnh PowerShell (`\`), đã khắc phục bằng cách gộp dòng hoặc dùng ký tự nối lệnh phù hợp.
- Một điều bạn làm khác lần sau: Sẽ tiến hành quét tổng thể ảnh kỹ hơn theo đúng checklist 4 bước trước khi chỉnh sửa chi tiết để tránh bỏ sót các bẫy tinh vi.