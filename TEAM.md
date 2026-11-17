# Thông tin nhóm

- Tên nhóm: THCL
- Mã thành viên (ba chữ số): Đặng Minh Quân(171), Vũ Duy Khanh(142), Đặng Minh Huy(162), Trịnh Đức Mạnh(167)
- Người điều phối: Đặng Minh Quân
- Người review bản cuối: Đặng Minh Quân
- Commit chốt bài (điền ở kênh nộp sau khi commit): 23446f9

## Kết quả tóm tắt (điền cuối bài)

- Lượt gán hoàn tất / được giao: 80/80 lượt (4 người × 20/20 ảnh, K=2 độc lập, seed 42)
- IAA trước calibration (kèm mẫu số): IoU TB 0.828 (234 cặp ghép / 80 lượt ảnh-người); đồng thuận lớp 94.9%; `occluded_level` 66.2%; `truncated` 96.6%; 169 + 39 box không ghép; 4 box `ignore_region` bỏ qua; kết luận CHƯA ĐẠT
- Số ảnh phải calibration: 38 / 40 (471 ca, 6 nhóm lỗi, khai trong `submission/qc/calibration_decisions.csv`)
- Vòng QC và số ảnh đã duyệt / cả lô: vòng 1 (full), đã duyệt 40/40 ảnh
- Tỉ lệ lỗi: 22.5 % theo ảnh (9/40), 3.0 % theo đối tượng (13/429); 0 vật bỏ sót
- **Quyết định** (Release / Hold / Rework) và lý do một dòng: Rework — 22.5% số ảnh có lỗi vượt ngưỡng lớp học 2%, và một loại lỗi (`sai_thuoc_tinh`, 9 ca) chiếm 69% tổng số lỗi nên là lỗi hệ thống của cách khai thuộc tính; cần sửa 13 đối tượng trong 9 ảnh của phiếu vòng 1.
