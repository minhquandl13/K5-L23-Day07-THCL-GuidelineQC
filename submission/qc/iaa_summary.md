# Tóm tắt IAA (taxonomy v1.0)

- Ảnh được so sánh (có >=2 người gán): **40**; số người gán: 4
- Lượt gán đủ **80/80**.
- IoU trung bình và % cùng lớp **chỉ tính trên các cặp box đã ghép** (IoU >= 0.5, ghép 1-1 không phân biệt lớp). Box chưa ghép được thống kê riêng ở cột bỏ sót/thừa.
- Mẫu số: 234 cặp box ghép (tổng mọi cặp người gán, ảnh chung).

| Cặp | Ảnh | Cặp box ghép | IoU TB | Cùng lớp | Cùng occluded_level | Cùng truncated | Chưa ghép (A/B) | Bỏ qua do ignore | Đạt IAA |
|---|---|---|---|---|---|---|---|---|---|
| **TỔNG** | 40 | 234 | 0.828 | 94.9% | 66.2% | 96.6% | 169/39 | 4 | CHƯA ĐẠT |
| 142|162 | 6 | 27 | 0.802 | 88.9% | 77.8% | 100.0% | 0/7 | 0 | CHƯA ĐẠT |
| 142|167 | 9 | 66 | 0.850 | 100.0% | 69.7% | 97.0% | 47/4 | 3 | CHƯA ĐẠT |
| 142|171 | 5 | 35 | 0.829 | 80.0% | 62.9% | 94.3% | 50/5 | 0 | CHƯA ĐẠT |
| 162|167 | 5 | 30 | 0.809 | 96.7% | 63.3% | 96.7% | 17/6 | 1 | CHƯA ĐẠT |
| 162|171 | 9 | 49 | 0.822 | 98.0% | 65.3% | 98.0% | 44/10 | 0 | CHƯA ĐẠT |
| 167|171 | 6 | 27 | 0.831 | 100.0% | 55.6% | 92.6% | 11/7 | 0 | CHƯA ĐẠT |

Ngưỡng IAA: IoU TB >= 0.85, đồng thuận lớp >= 95%.

Số ca bất thường: 471 ({'bo_sot_thua': 123, 'iou_thap': 111, 'sai_thuoc_tinh': 81, 'bat_dong_ignore': 123, 'co_can_review': 21, 'sai_lop': 12}), trên 38 ảnh cần calibration.
