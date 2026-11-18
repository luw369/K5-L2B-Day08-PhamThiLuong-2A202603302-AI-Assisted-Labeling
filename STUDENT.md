## Thông tin học viên

- Họ và tên: Phạm Thị Lương
- MSHV: 2A202603302
- Lớp (L2 / L3 / khác): L2B
- Tài khoản GitHub: luw369
- Repo bài nộp (URL): https://github.com/luw369/K5-L2B-Day08-PhamThiLuong-2A202603302-AI-Assisted-Labeling.git
- Commit chốt bài (hash): HEAD
- Máy: hệ điều hành Windows, GPU
- CVAT (Docker): v2.81.0

## Kết quả tóm tắt (điền cuối bài; số lấy từ `submission/qc/*`, kèm mẫu số)

- Máy: accepted / edited / deleted: 446 / 12 / 17 (mẫu số: 485 box máy); added: 11 (mẫu số: 469 đối tượng)
- So với gold: precision / recall của **máy**: 56.7% / 61.8%
- Bắt bẫy: 10/12 = 83.3%
- Thời gian: gán tay 5.3 phút/ảnh; duyệt có pre-label 3.0 phút/ảnh
- Lỗi QC: 4.2% theo ảnh (1/24), 1.5% theo đối tượng (7/469)
- **Quyết định:** Release — lý do một dòng: Tỷ lệ accepted đạt 92.0% và bắt bẫy vượt ngưỡng với 83.3%, chất lượng pre-label tốt cùng quy trình review chính xác, đủ điều kiện phát hành dữ liệu.