# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 485; đối tượng sau duyệt: 469; ngưỡng accepted: IoU >= 0.9
- Đã **loại 28 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 446 | 446/485 = 92.0% | 485 box máy |
| edited (sửa) | 12 | 12/485 = 2.5% | 485 box máy |
| deleted (xoá) | 17 | 17/485 = 3.5% | 485 box máy |
| added (vẽ thêm) | 11 | 11/469 = 2.3% | 469 đối tượng sau duyệt |

Trong số deleted + added, **0 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 11.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 263 | 0 | 5 | 1 |
| bicycle | 4 | 1 | 3 | 1 |
| motorbike | 124 | 4 | 2 | 3 |
| car | 36 | 1 | 0 | 0 |
| bus | 3 | 0 | 0 | 0 |
| truck | 5 | 0 | 2 | 0 |
| three_wheeler | 11 | 6 | 5 | 6 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 55 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 34 | 26 | 21 | 56.7% (KTC 95%: 44–68%) | 61.8% (KTC 95%: 49–73%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 10/12 = 83.3% (KTC 95%: 55–95%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - extra: 3/3
  - missing: 2/3
  - loose: 2/3
  - wrong_class: 3/3
  - lọt: img_028.jpg (missing), img_003.jpg (loose)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
