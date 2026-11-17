# Báo cáo QC — Nhóm 4 người (142, 162, 167, 171)

## 1. Mục đích, dữ liệu và phiên bản

- Mục đích và giới hạn sử dụng: QC nhãn 2D (car/truck/van/motorcycle/person + ignore_region) trên 40 ảnh phố trong `data/images`. Nhãn dùng cho đo IAA và làm bộ huấn luyện/demo Day 07; KHÔNG dùng cho bài toán khác (biển số, tín hiệu giao thông, đếm lưu lượng) vì taxonomy không có lớp đó và 40 ảnh không đại diện cho toàn bộ điều kiện giao thông.
- Nguồn và phép sử dụng dữ liệu: ảnh lấy theo `data/ATTRIBUTION.csv`; có ảnh chứa người thật (kể cả cosplay) nên chỉ dùng trong phạm vi học tập nội bộ, không phát hành lại ảnh gốc.
- Manifest và checksum nhãn cuối: `submission/qc/batch_manifest.json` (SHA-256 `a3fe72e313fd0b91b078c2eee6ec5ba0f73b0aab7f85b3aeff241e9493b82460`); bản nhãn bàn giao `submission/qc/labels_calibrated.json` (SHA-256 `e6f5747046e1e43b6b22a51b6e7a3887c2274f9deaec35489866bbb70b179e77`), nguồn chép từ `labels_calibrated_f4b49896ff730d0b.json` (SHA-256 `d222b99f5065e12dbac0049912b6c6b08d7c8622179468ed4e3e6458ae659c23`), tên file do calibration sinh ra.
- Guideline và taxonomy khi gán / khi bàn giao: khi gán dùng `guideline_v1.0_agreed.md` + `taxonomy.json` v1.0 (6 lớp, 3 thuộc tính); khi bàn giao dùng `guideline_v1.1_calibrated.md` + cùng taxonomy v1.0, thay đổi là semantic và đã khai trong `submission/guideline_change.json` (R-08, R-11, phạm vi `lo`).

## 2. Phân công và đồng thuận trước calibration

- Lượt hoàn thành / được giao: 80/80 lượt (mỗi thành viên 20/20 ảnh), 40/40 ảnh có đúng K=2 người gán độc lập theo `assignment.csv`.
- Số cặp box ghép, IoU TB, đồng thuận lớp và mẫu số: 234 cặp ghép trên 80 lượt ảnh-người; IoU trung bình 0.828; đồng thuận lớp 94.9%; đồng thuận `occluded_level` 66.2%; `truncated` 96.6%.
- Box chưa ghép, loại do ignore, bất đồng vùng ignore: 169 box của người A và 39 box của người B không ghép được; 4 box `ignore_region` được bỏ qua khi đo; 123 bất đồng `bat_dong_ignore`.
- Ca hai người có thể cùng sai sau khi đối chiếu tham chiếu: `img_019` và `img_034` (vùng đám đông chọn "một ignore_region trùm" so với "gán từng người"), `img_037` (ranh giới truck/van theo đường nóc), `img_008` (xe đứng cạnh nhau, dễ cùng gán sai lớp). Cách kiểm: đối chiếu ảnh gốc + mốc neo trong §2.2.1 của v1.1, không dựa vào nhãn của đa số.

## 3. Calibration và kiểm lại

- Ca đã xử lý, rule ID, bản sửa và người xác nhận: 471 ca trong `submission/qc/calibration_decisions.csv` (6 nhóm lỗi: 123 `bat_dong_ignore`, 123 `bo_sot_thua`, 111 `iou_thap`, 81 `sai_thuoc_tinh`, 21 `co_can_review`, 12 `sai_lop`); mỗi dòng ghi `phieu_sua = SUA-<case_id>`, `nguoi_chot = 171`, `phien_ban = v1.1`, `kiem_lai` theo thành viên liên quan.
- Tác động v1.1 lên nhãn và ảnh: R-08 đổi sang quy tắc "ba câu hỏi" + "một ignore chỉ trùm một cụm", R-11 bổ sung mốc neo `occluded_level` và cấm dùng cờ `Q` của CVAT; phạm vi thay đổi là `lo` nên được áp cho toàn lô 40 ảnh. Nhãn cuối còn 435 annotation, 0 box `can_review=true`, 0 box cao dưới 20 px.
- Tham chiếu bằng chứng khi Rework: vòng 0 là nhãn gốc của 4 thành viên trong `submission/labels/*.json` (đã khoá hash); vùng sửa được chỉ ra theo `object_id` trong phiếu vòng 1 tại `submission/qc/reviews/round-01/qc_review.csv`; vòng mới phải chạy lại `qc_sample.py --round 2` trên cùng bản nhãn đã sửa.

## 4. Kết quả duyệt

- Vòng QC: 1
- Người review và ngày: 171 (Đặng Minh Quân), 2026-11-17
- Đã duyệt / cả lô: 40/40 ảnh đã duyệt (vòng full, không lấy mẫu).
- Ảnh lỗi / ảnh đã duyệt: 14/40 = 35.0%; nhóm `ca_kho` 14/38 = 36.8%, nhóm `con_lai` 0/2.
- Annotation đối tượng hiện có bị lỗi / annotation đối tượng đã duyệt: 22/429 = 5.1%.
- Vật bỏ sót, ảnh và vùng: 0 ca bỏ sót được ghi; các khung 0 box (`img_017`, `img_020`, `img_027`, `img_033`, `img_038`) đã được đối chiếu và đúng theo R-05/R-06/R-14.
- Ca khó / phần còn lại, lỗi lặp và ví dụ: lỗi lặp nhiều nhất là `sai_thuoc_tinh` (14/23 = 61% số lỗi) — chủ yếu là `truncated` khai `yes/no` không khớp việc box có chạm mép ảnh hay không; kế đến `thua` 5 và `sai_lop` 4. Ví dụ: `img_005` id 62 trùng id 61 (IoU 0.56), `img_039` id 410 person chồng lên id 414 motorcycle (IoU 0.61).
- Một lỗi máy chặn được và một lỗi cần người đọc ảnh: lỗi máy chặn được là box trùng cùng lớp (IoU ≥ 0.55, phát hiện bằng so khớp hình học, không cần nhìn ảnh); lỗi cần người đọc ảnh là `truncated` sai — máy chỉ biết box có chạm mép hay không, còn việc vật có thực sự bị cắt bởi khung hình hay bị che bởi vật khác phải do người phán đoán trên ảnh.

## 5. Quyết định bàn giao

- Quyết định: Rework
- Lý do, người nhận và phạm vi sử dụng: 35.0% số ảnh có lỗi, vượt ngưỡng lớp học `error_rate_img_max = 0.02`, đồng thời một loại lỗi (`sai_thuoc_tinh`) chiếm 61% và lặp 14 lần nên không thể coi là nhiễu ngẫu nhiên. Bản nhãn hiện tại không bàn giao cho bước huấn luyện; người nhận là nhóm 4 thành viên, phạm vi sửa là 14 ảnh được nêu trong phiếu vòng 1.
- Nếu còn việc: sửa 22 đối tượng bị nêu (chủ yếu đồng bộ lại `truncated`, gộp/xoá box trùng, tách box một vật hai lớp), người xử lý là người gán gốc của từng ảnh, người kiểm lại là 171; bằng chứng cần kiểm lại là `qc_review.csv` của vòng 1 và ảnh phủ `work/review/round-01/`.
- Kiểm ngược từ báo cáo tới ảnh, rule và nhãn: (1) `img_005` — mở `work/review/round-01/img_005.jpg.png`, thấy id 62 và id 61 cùng lớp person chồng nhau, đối chiếu `qc_review.csv` dòng `img_005.jpg,62,thua,171,R-08` và `labels_calibrated.json` hai annotation cùng `image_id`; (2) `img_039` — id 410 person và id 414 motorcycle, dòng `img_039.jpg,410,sai_lop,171,R-02`; (3) `img_008` — id 95 box chạm mép phải nhưng `truncated=no`, dòng `img_008.jpg,95,sai_thuoc_tinh,171,R-11`. Cả ba mở được từ `qc_review.csv` → vòng → ảnh phủ → nhãn, không có bước nào phải suy đoán.
