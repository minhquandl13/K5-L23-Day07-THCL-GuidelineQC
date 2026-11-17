# Báo cáo QC — Nhóm 4 người (142, 162, 167, 171)

## 1. Mục đích, dữ liệu và phiên bản

- Mục đích và giới hạn sử dụng: QC nhãn 2D (car/truck/van/motorcycle/person + ignore_region) trên 40 ảnh phố trong `data/images`. Nhãn dùng cho đo IAA và làm bộ mẫu Day 07; KHÔNG dùng cho bài toán khác (biển số, tín hiệu giao thông, đếm lưu lượng) vì taxonomy không có lớp đó và 40 ảnh không đại diện cho toàn bộ điều kiện giao thông.
- Nguồn và phép sử dụng dữ liệu: ảnh lấy theo `data/ATTRIBUTION.csv`; có ảnh chứa người thật (kể cả cosplay) nên chỉ dùng trong phạm vi học tập nội bộ, không phát hành lại ảnh gốc.
- Manifest và checksum nhãn cuối: `submission/qc/batch_manifest.json` (SHA-256 `a3fe72e313fd0b91b078c2eee6ec5ba0f73b0aab7f85b3aeff241e9493b82460`); bản nhãn bàn giao `submission/qc/labels_calibrated.json` (SHA-256 `e6f5747046e1e43b6b22a51b6e7a3887c2274f9deaec35489866bbb70b179e77`), chép từ `labels_calibrated_f4b49896ff730d0b.json` (SHA-256 `d222b99f5065e12dbac0049912b6c6b08d7c8622179468ed4e3e6458ae659c23`), tên file do calibration sinh.
- Guideline và taxonomy khi gán / khi bàn giao: khi gán dùng `guideline_v1.0_agreed.md` + `taxonomy.json` v1.0 (6 lớp, 3 thuộc tính); khi bàn giao dùng `guideline_v1.1_calibrated.md` + cùng taxonomy v1.0; thay đổi là semantic, khai trong `submission/guideline_change.json` (R-08, R-11, phạm vi `lo`).

## 2. Phân công và đồng thuận trước calibration

- Lượt hoàn thành / được giao: 80/80 lượt (mỗi thành viên 20/20 ảnh), 40/40 ảnh có đúng K=2 người gán độc lập theo `assignment.csv`.
- Số cặp box ghép, IoU TB, đồng thuận lớp và mẫu số: 234 cặp ghép trên 80 lượt ảnh-người; IoU trung bình 0.828; đồng thuận lớp 94.9%; `occluded_level` 66.2%; `truncated` 96.6%.
- Box chưa ghép, loại do ignore, bất đồng vùng ignore: 169 box của người A và 39 box của người B không ghép được; 4 box `ignore_region` bỏ qua khi đo; 123 bất đồng `bat_dong_ignore`.
- Ca hai người có thể cùng sai sau khi đối chiếu tham chiếu: `img_019` và `img_034` (vùng đám đông chọn "một ignore_region trùm" so với "gán từng người"), `img_008` (dải xe máy đứng sát nhau), `img_037` (bus/xe thùng ở mép trái). Cách kiểm: đối chiếu ảnh gốc + mốc neo §2.2.1 của v1.1, không dựa vào nhãn của đa số.

## 3. Calibration và kiểm lại

- Ca đã xử lý, rule ID, bản sửa và người xác nhận: 471 ca trong `submission/qc/calibration_decisions.csv` (6 nhóm: 123 `bat_dong_ignore`, 123 `bo_sot_thua`, 111 `iou_thap`, 81 `sai_thuoc_tinh`, 21 `co_can_review`, 12 `sai_lop`); mỗi dòng ghi `phieu_sua = SUA-<case_id>`, `nguoi_chot = 171`, `phien_ban = v1.1`, `kiem_lai` theo thành viên liên quan.
- Tác động v1.1 lên nhãn và ảnh: R-08 đổi sang "ba câu hỏi + một ignore chỉ trùm một cụm", R-11 thêm mốc neo `occluded_level` và cấm cờ `Q` của CVAT; phạm vi `lo` nên áp cho cả 40 ảnh. Nhãn cuối còn 435 annotation, 0 box `can_review=true`, 0 box cao dưới 20 px.
- Tham chiếu bằng chứng khi Rework: vòng 0 là nhãn gốc 4 thành viên trong `submission/labels/*.json` (đã khoá hash); vùng sửa chỉ ra theo `object_id` trong `submission/qc/reviews/round-01/qc_review.csv`; vòng mới chạy `qc_sample.py --round 2` trên bản nhãn đã sửa.

## 4. Kết quả duyệt

- Vòng QC: 1
- Người review và ngày: 171 (Đặng Minh Quân), 2026-11-17
- Đã duyệt / cả lô: 40/40 ảnh (vòng full, không lấy mẫu).
- Ảnh lỗi / ảnh đã duyệt: 9/40 = 22.5%; nhóm `ca_kho` 9/38 = 23.7%, nhóm `con_lai` 0/2.
- Annotation đối tượng hiện có bị lỗi / annotation đối tượng đã duyệt: 13/429 = 3.0%.
- Vật bỏ sót, ảnh và vùng: 0 ca bỏ sót; các khung 0 box (`img_017`, `img_020`, `img_027`, `img_033`, `img_038`) đã đối chiếu và đúng theo R-05/R-06/R-14.
- Ca khó / phần còn lại, lỗi lặp và ví dụ: lỗi lặp nhiều nhất là `sai_thuoc_tinh` (9/13 = 69%) — `truncated` khai `no` trong khi box chạm đúng mép khung hình, tức chính nét vẽ đã nói vật tiếp tục ra ngoài ảnh; tiếp theo là `sai_vi_tri_kich_thuoc` (4 ca) — box trùm hai đến ba vật: `img_030` id 316 ôm hai xe máy (trùng id 312, IoU 0.59), `img_032` id 355 ôm ba người (trùng id 354, IoU 0.75), `img_012` id 168 rộng 710 px phủ cả mặt đường, `img_005` id 62 trùm hai người.
- Một lỗi máy chặn được và một lỗi cần người đọc ảnh: lỗi máy chặn được là box chạm mép khung hình mà khai `truncated=no` — chỉ cần so toạ độ box với `width/height`, không cần nhìn ảnh; lỗi cần người đọc ảnh là box trùm nhiều vật — máy chỉ thấy hai box chồng nhau (IoU ≥ 0.55) và không phân biệt được "hai vật sát nhau" với "một box vẽ ẩu", phải mở ảnh mới kết luận.
- Ghi chú phương pháp: bước sàng lọc tự động đề xuất 23 nghi vấn trên 14 ảnh; sau khi mở từng vùng cắt từ ảnh gốc, 10 nghi vấn bị loại (4 ca `sai_lop` là người ngồi trên xe — định nghĩa `person` trong taxonomy đã bao gồm người điều khiển phương tiện nên box người và box xe vốn chồng nhau hợp lệ; 3 ca `truncated` có box cách mép 2–3 px; 3 ca box xe máy đứng sát nhau trong đêm). Phiếu cuối giữ 13 phát hiện, tất cả đều kiểm được trên ảnh.

## 5. Quyết định bàn giao

- Quyết định: Rework
- Lý do, người nhận và phạm vi sử dụng: 22.5% số ảnh có lỗi, vượt ngưỡng lớp học `error_rate_img_max = 0.02`, và một loại lỗi (`sai_thuoc_tinh`) chiếm 69% tổng số lỗi nên là lỗi hệ thống của cách khai thuộc tính, không phải nhiễu ngẫu nhiên. Bản nhãn không bàn giao cho bước huấn luyện; người nhận là 4 thành viên, phạm vi sửa là 13 đối tượng trong 9 ảnh của phiếu vòng 1.
- Nếu còn việc: đồng bộ lại `truncated` cho 9 box chạm mép (người gán gốc của từng ảnh sửa, 171 kiểm lại), tách box trùm nhiều vật ở 4 ca thành box một vật; bằng chứng cần kiểm lại là `qc_review.csv` vòng 1 và ảnh phủ `work/review/round-01/`, sau đó mở vòng 2 bằng `qc_sample.py --round 2`.
- Kiểm ngược từ báo cáo tới ảnh, rule và nhãn: (1) `img_008` id 95 — mở `work/review/round-01/img_008.jpg.png`, box nằm ở góc phải-dưới, chạm cả hai mép, thuộc tính ghi `truncated=no`; dòng tương ứng `img_008.jpg,95,sai_thuoc_tinh,171,R-11` trong `qc_review.csv`; cùng `object_id` trong `labels_calibrated.json`. (2) `img_030` id 316 — ảnh cắt `work/review/round-01/qc_crops/img_030.png` cho thấy một box ôm hai xe máy; dòng `img_030.jpg,316,sai_vi_tri_kich_thuoc,171,R-07`. (3) `img_037` id 409 — box `van` bắt đầu đúng x=0 trong khi xe bị cắt bởi mép trái, thuộc tính `truncated=no`; dòng `img_037.jpg,409,sai_thuoc_tinh,171,R-11`.
