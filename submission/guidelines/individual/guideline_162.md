# Guideline gán nhãn — v1.0-cá nhân — 162

> Bản **độc lập** Pha 1, viết từ khung `guideline/TEMPLATE_guideline.md` và `guideline_v0_starter.md`.
> Không sửa lại sau khi họp nhóm; quyết định nhóm chốt ở `guideline_v1.0_agreed.md`.

- Phiên bản: v1.0 (bản cá nhân 162)
- Ngày: 2026-11-16
- Người viết: thành viên 162

## 1. Mục tiêu

Nhãn bounding box trên ảnh đường phố dùng huấn luyện mô hình **phát hiện người và phương tiện**
(camera giao thông / hỗ trợ lái). Sai nguy hiểm nhất: bỏ sót vật thật trong làn đường (person,
motorcycle, car đang di chuyển) và gán nhầm vật không tồn tại thật (phản chiếu, tượng, ma-nơ-canh)
thành vật thật — hai loại này tạo dương tính/âm tính giả khi mô hình triển khai thực tế.

## 2. Lớp và thuộc tính

### Lớp

| Lớp | Định nghĩa (một câu) | Ví dụ | Phản ví dụ / dễ nhầm với |
|---|---|---|---|
| `car` | Ô tô con/SUV có 4 bánh trở lên, thân kín, chở người. | Xe trắng đang chạy trong img_015 | Tuk-tuk chở người (xem R-08); xe bán tải chở hàng → cân nhắc `truck`; phản chiếu xe trong gương (R-06) |
| `truck` | Xe tải có khoang/thùng chở hàng tách biệt, kể cả rơ-moóc. | Đầu kéo + dàn chở xe trong img_002 | Xe khách, tuk-tuk chở hàng (R-08); hàng hóa chất trên nóc không phải xe |
| `van` | Xe thùng kín cỡ nhỏ, mui liền, không có khoang hàng tách rời. | Xe minivan xám nền img_005 | `truck` (thùng rời), `car` (SUV con) |
| `motorcycle` | Xe máy/xe ga/mô tô, có hoặc không có người lái. | Dàn xe máy dựng trong img_030 | Xe đạp thồ hàng (img_010 — không có lớp, xem mục "không thuộc lớp nào"); phần đầu xe máy bị che gần hết |
| `person` | Người thật: đi bộ, đứng, ngồi trên xe; tính cả người chỉ thấy đầu/helmet. | Người bán hàng trong img_010 | Ma-nơ-canh, tượng, ảnh người trên áp phích/biển quảng cáo (R-06) |
| `ignore_region` | Vùng không cần gán từng vật; bỏ sót ở đây không tính lỗi. | Đám xe máy chồng chi chít không tách nổi từng chiếc ở hậu cảnh img_030 | Vật thật rõ ràng (phải gán lớp) |

**Vật không thuộc lớp nào** (xe đạp, xích lô/thồ, ô tô chở xe đạp, hàng hóa, động vật): **không gán lớp**;
nếu chiếm vùng lớn gây nhầm với vật trong lớp thì khoanh `ignore_region`, còn lại bỏ qua tự nhiên.
Không tự thêm lớp mới — ghi vào mục 4 để nhóm bàn.

### Thuộc tính

| Thuộc tính | Các mức cố định | Nghĩa và ngưỡng cụ thể |
|---|---|---|
| `occluded_level` | `0`,`1`,`2`,`3` | Tính trên **diện tích phần thân dự kiến của vật**, không tính mép ảnh: `0` = che < 10%; `1` = 10–30% (vd người đeo khẩu trang+goggles như img_005 vẫn tính thêm mặt nạ che); `2` = 30–60% (xe bị cột/trước xe khác che quá 1/3); `3` = > 60% nhưng **còn nhận dạng được lớp**, nếu không nhận dạng được thì bỏ vật. |
| `truncated` | `no`,`yes` | `yes` khi thân vật bị **cắt bởi biên ảnh** (vd hai xe đỏ/xám áp hai mép img_015). Bị vật khác che **không** phải truncated. |
| `can_review` | tick / không | Tick khi: lớp còn nghi ngờ (R-08), mức che sát ngưỡng ±5%, hoặc box phụ thuộc quyết định chưa chốt. Không dùng thay cho việc đọc lại rule. |

## 3. Quy tắc vẽ box

- **R-01 — Ôm sát phần nhìn thấy.** Box vừa khít phần vật **thật sự nhìn thấy** (kể cả phần bị che một phần nhưng suy ra được vị trí liền mạch); không box "phantom" phần bị che hoàn toàn sau xe khác. Gương chiếu hậu, giá đỡ, người lái **không** tính vào box xe; baga chở hàng chất cao trên xe (img_035) **không** kéo dài box xe.
- **R-02 — Người và xe là hai box.** Người ngồi trên xe máy: một box `person` (từ mũ bảo hiểm tới phần thân nhìn thấy, không gồm tay lái/xe) + một box `motorcycle` (cả xe, không gồm người). Xe đang chạy hay dựng đều gán cả hai nếu đều thấy (img_005, img_010).
- **R-03 — Xe chở xe.** Đầu kéo + rơ-moóc/dàn nâng là **một** box `truck` ôm sát khung xe (không gồm bánh xe chở trên đó trừ khi liền khối). Mỗi chiếc xe con chở trên dàn: gán `car` riêng nếu thấy ≥ phần đầu/đáy rõ; đặt `occluded_level` theo mức bị dàn nâng che.
- **R-04 — Cắt mép ảnh.** Vật bị biên ảnh cắt: box tới sát biên, `truncated = yes`; chỉ gán khi còn nhận dạng được lớp (R-01).
- **R-05 — Quá nhỏ.** Box cao < 20 px (theo `min_box_height_px` của taxonomy): **không gán**; nếu cả cụm nhỏ gây nhiễu thì `ignore_region`.
- **R-06 — Phản chiếu / ảnh / tượng / đồ chơi.** Ảnh xe trong gương chiếu hậu, cửa kính, vũng nước: **không gán lớp xe**; tượng, ma-nơ-canh, mô hình, ảnh người trên áp phích: **không gán `person`**. Nếu vật giả chiếm vùng lớn dễ nhầm, khoanh `ignore_region` (vd áp phích góc trên img_030).
- **R-07 — Đám đông / dàn xe chồng.** Đếm và gán từng chiếc/người tách được bằng ≥ 2 dấu hiệu nhận dạng; phần hậu cảnh chồng chi chít không tách nổi: một `ignore_region` phủ vùng (img_030).
- **R-08 — Xe ba bánh (tuk-tuk/auto-rickshaw).** Bản cá nhân này tạm gán: chở người → `car`, kéo theo rơ-moóc hàng → `truck`, cả hai tick `can_review` vì taxonomy v0 không có lớp "other" (xem mục 4, img_035). **Ca cần hỏi nhóm**, không phải quyết định loại theo quy tắc.
- **R-09 — Xe chủ (ego vehicle).** Phần xe của người quay (táp-lô, capo sát mép dưới) không gán; nếu chiếm > 1/4 khung thì `ignore_region`.

## 4. Ca khó

| Ảnh | Tình huống | Rule ID | Quyết định | Lý do |
|---|---|---|---|---|
| img_002 | Xe tải hai tầng chở 8+ ô tô con | R-03 | 1 box `truck` (cả đầu kéo + dàn) + mỗi xe con 1 box `car`, mức che theo dàn | Xe chở vẫn là phương tiện thật, mô hình cần học dạng "xe nằm yên trên giá" |
| img_005 | Cluster 5–6 người đi xe máy sát nhau, đeo khẩu trang/mũ | R-02, R-07 | Mỗi người thấy được: 1 `person` + xe của họ 1 `motorcycle` (nếu thấy thân xe); mặt nạ/khẩu trang tính vào `occluded_level` của person | Người là vật nguy hiểm nhất, thà tách thừa theo dấu hiệu mũ+hành vi lái |
| img_010 | Người ngồi sau trên xe máy + người bán hàng đứng cạnh + xe đạp thồ hàng giữa ảnh | R-02, mục "không thuộc lớp" | Người ngồi sau vẫn `person` (2 box người trên 1 xe); xe đạp thồ **không gán**, vùng hàng hóa đè gây nhầm thì `ignore_region` | Xe đạp không có trong lớp v0; người ngồi sau là lỗi bỏ sót kinh điển |
| img_015 | Hai xe ô tô áp sát cắt cả hai mép ảnh | R-04 | Gán cả hai `car`, `truncated = yes`, box tới biên | Còn nhận dạng rõ là ô tô con; cắt mép không phải lý do bỏ |
| img_020 | Ảnh xe trong gương chiếu hậu + giọt nước | R-06 | **Không gán** xe trong gương; bản thân gương/đầu gương không gán | Phản chiếu không phải vật thật — dương tính giả nếu gán |
| img_030 | Bãi xe máy dựng san sát, áp phích người mẫu góc trên, người bảo vệ | R-07, R-06 | Tiền cảnh: gán từng xe tách được; hậu cảnh: 1 `ignore_region`; áp phích bỏ qua (không cần ignore nếu nhỏ); người bảo vệ gán `person` | Ngưỡng tách từng chiếc = ≥2 dấu hiệu; áp phích là ảnh người, không phải người |
| img_035 | Tuk-tuk chở bao hàng + xe máy kéo rơ-moóc + xe máy người lái | R-08, R-01 | Tuk-tuk: `car` + `can_review`; hàng hóa trên nóc không kéo box; mỗi xe máy + người theo R-02 | Lớp "other" không tồn tại trong taxonomy v0 — **cần họp nhóm chốt**, đây là ca mơ hồ chứ không phải ca loại theo quy tắc |

## 5. QC và phiên bản

- **Ngưỡng áp dụng** (giữ nguyên `rules` của taxonomy): IAA IoA `iaa_iou = 0.85`, đồng thuận lớp `iaa_class = 0.95`; khớp box `match_iou = 0.5`; box lớn ≥ 50 px dùng `iou_thr_large = 0.85`, nhỏ hơn dùng `iou_thr_small = 0.7`; tỉ lệ lỗi ảnh tối đa `error_rate_img_max = 2%`.
- **Ai quyết ca mơ hồ:** người điều phối tổng hợp; ca tick `can_review` đưa vào họp calibration, kết quả ghi `calibration_decisions.csv` kèm Rule ID dẫn chiếu.
- **Nhật ký QC** dùng mã thành viên (162, …), không dùng họ tên/MSHV.

| Phiên bản | Thay đổi | Lý do (ca nào) |
|---|---|---|
| v0 | (bản khởi đầu giảng viên) | — |
| v1.0-cá nhân-162 | Viết lại 5 phần; thêm ngưỡng che/cắt/quá nhỏ; chốt R-01→R-09; đánh dấu tuk-tuk (img_035) và đám xe máy (img_030) là ca cần nhóm | img_002, img_005, img_010, img_015, img_020, img_030, img_035 |
