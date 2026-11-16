# Guideline gán nhãn — v1.0 (bản thống nhất nhóm)

- Phiên bản: **v1.0** (khóa cho Pha 3)
- Ngày: 2026-11-16
- Nhóm: 4 thành viên — mã **142, 162, 167, 171**
- Người viết / duyệt: tổng hợp từ 4 bản cá nhân; **người điều phối xác nhận tại họp Pha 2** trước khi khóa
- Thay thế: `guideline/guideline_v0_starter.md` (v0 không đủ dùng)
- Taxonomy đi kèm: `submission/taxonomy.json` phiên bản `v1.0`, **giữ nguyên 6 lớp và mục `rules`**
- Phân công Pha 3: mỗi ảnh do **2 người** gán độc lập (K=2), seed 42, `assignment.csv` dùng chung

> Tài liệu này hợp nhất `individual/guideline_142.md`, `guideline_162.md`, `guideline_167.md`, `guideline_171.md`.
> Bảng **§4.2** ghi rõ mỗi bất đồng được chốt thế nào và vì sao — đó là phần chấm điểm "giải quyết bất đồng".

## 1. Mục tiêu

Nhãn bounding box 2D trên 40 ảnh đường phố, dùng huấn luyện mô hình **phát hiện người và phương tiện**
(camera giao thông, ADAS). Nhãn mô tả **vị trí phần nhìn thấy và lớp của vật thật**, không tái dựng 3D,
không suy vận tốc, không suy danh tính.

Ba nhóm lỗi quyết định chất lượng:

| Lỗi | Hậu quả | Quy tắc chặn |
|---|---|---|
| Bỏ sót người / người ngồi sau xe | Mô hình không cảnh báo được vật nguy hiểm nhất | R-03, R-08 |
| Gán vật giả (tượng, ma-nơ-canh, hình in, phản chiếu, đồ chơi) | Dương tính giả, mô hình phanh/đánh lái sai | R-05, R-06 |
| Gộp người vào xe, hoặc kéo box theo hàng hóa | Sai biên và kích thước, sai ước lượng khoảng cách | R-01, R-03, R-12 |

Ngưỡng QC của lab là tiêu chuẩn **bài thực hành**, không phải chứng nhận an toàn để triển khai thật.
Ghi công ảnh giữ tại `data/ATTRIBUTION.csv`.

## 2. Lớp và thuộc tính

### 2.1. Lớp — 6 lớp, không thêm lớp mới ở v1.0

| Lớp | Định nghĩa (một câu) | Ví dụ trong bộ ảnh | Phản ví dụ / dễ nhầm |
|---|---|---|---|
| `car` | Ô tô con thân kín, cabin liền khoang khách, **không có thùng hàng riêng**: sedan, hatchback, wagon, SUV/crossover, MPV, taxi. | `img_023` (Honda Civic), `img_012` (Mazda CX-5), `img_015` (2 xe sát mép), `img_031`, `img_040` (taxi) | Bán tải / xe có sàn hàng riêng → `truck`; xe 16 chỗ thân hộp → `van`; tuk-tuk → ngoài phạm vi (R-09); ô tô đồ chơi `img_038` → loại (R-05) |
| `truck` | Xe có **cabin và thùng/sàn chở hàng tách biệt**, gồm bán tải thùng hở, tải nhẹ, tải thùng kín/bạt, xe chở ô tô, đầu kéo + rơ-moóc. | `img_001`, `img_002`, `img_007`, `img_012`, `img_036`, `img_037` | SUV/MPV không thùng riêng → `car`; van liền nóc → `van`; tượng bán tải `img_027` → loại |
| `van` | Xe thân hộp kín, **nóc liền từ cabin ra sau**, không có thùng hàng tách rời: minivan, xe khách/minibus cỡ nhỏ, xe giao hàng liền thân. | `img_005` (Ford Transit), `img_016` (van bạc), `img_009` (minibus — xem §4.3) | Xe buýt dài, ≥ 2 cửa khách dọc thân → ngoài phạm vi (R-09); bán tải gắn nắp thùng `img_037` → `truck` |
| `motorcycle` | Xe 2 bánh gắn động cơ (xe máy, tay ga, mô tô, xe máy điện); mô tô + sidecar gắn liền là **một** đối tượng. | `img_006`, `img_010`, `img_022` (sidecar), `img_026`, `img_030` | Xe đạp, xích lô, xe ba gác → ngoài phạm vi; tuk-tuk 3 bánh có cabin → ngoài phạm vi |
| `person` | Một **người thật** nhìn thấy trực tiếp: đi bộ, đứng, ngồi, làm việc, hoặc đang ngồi/điều khiển phương tiện (kể cả ngồi sau). | `img_014`, `img_032`, `img_005` (từng người), `img_040` | Tượng `img_017`, bia mộ `img_027`, ma-nơ-canh `img_033`, hình in `img_018`, búp bê/mascot `img_019`, `img_038`, phản chiếu (R-06) |
| `ignore_region` | **Vùng bỏ qua**, không phải lớp vật: nơi vật quá nhỏ hoặc chồng lấp không tách được; bỏ sót ở đây không tính lỗi. | `img_021`, `img_030` (chỉ phần nền dày), `img_038` | Không dùng để né vật to/rõ; không phủ cả bãi xe hay cả hàng người (R-08) |

**R-01 — mỗi vật đúng một lớp.** Không có lớp `rider`, `bus`, `bicycle`, `tuk_tuk` ở v1.0.

**R-02 — phân biệt lớp bằng cấu trúc, theo đúng thứ tự này** (không dựa vào số ghế, nhãn hiệu, kích thước):

1. Có **thùng/sàn chở hàng tách khỏi cabin** — bán tải thùng hở, tải thùng kín/bạt, thùng dịch vụ gắn trên sàn — → `truck`.
2. Có **thân hộp kín liền một khối từ cabin ra sau**, không thùng riêng → `van`.
3. **Còn lại** thân ô tô con, cabin và khoang khách liền → `car`.
4. **Phép thử "đường nóc" (dùng khi phân vân `van` ↔ `truck`):**
   nóc cabin và nóc khoang sau **rời nhau, có bậc/seam, hoặc khoang sau cao hơn cabin** → `truck` (`img_037`);
   nóc **liền một đường** từ kính trước tới đuôi, chỉ có cửa hông và kính bên → `van` (`img_009`, `img_016`).
5. Không đủ dấu hiệu → `can_review` + câu hỏi; không đoán. **Trong 40 ảnh này không có ca nào rơi vào mục 5.**

### 2.2. Thuộc tính — mỗi box đủ 3 thuộc tính (trừ `ignore_region`)

| Thuộc tính | Mức | Nghĩa và ngưỡng đo được |
|---|---|---|
| `occluded_level` | `0` | Che **< 10%** (kể cả bị dây, lưới, cột mảnh che ít). |
| | `1` | Che **10% – 30%** (đúng 10% tính mức 1, đúng 30% tính mức 1). |
| | `2` | Che **> 30% – 70%** (đúng 70% tính mức 2). |
| | `3` | Che **> 70%**, **vẫn nhận ra lớp** và tách được một đối tượng. |
| `truncated` | `no` | Vật nằm trọn trong khung; **chạm mép nhưng đường bao kín** vẫn là `no`. |
| | `yes` | Đường bao bị mép ảnh **cắt hở**, thấy rõ vật tiếp tục ra ngoài khung. |
| `can_review` | tick | Lớp chưa chắc; mức che sát ngưỡng (±5%); hàng hóa vượt thân xe; tượng ↔ người hóa trang chưa phân biệt được; nhìn qua kính chưa rõ là xuyên hay phản chiếu. |

**R-11 — cách đo che khuất.** `q = 100 × (diện tích hình dáng bị vật khác che) / (diện tích hình dáng dự kiến **trong khung**)`.
Mẫu số **không** phải diện tích hình chữ nhật; phần ngoài ảnh xử lý bằng `truncated`, không cộng vào `q`.
Trang phục, mũ, khẩu trang, ba lô **của chính người** thuộc hình dáng người → **không tính là che**.
Bị che 100% thì **không được suy ra** để gán. Không dùng mức `3` làm mã "không biết".

## 3. Quy tắc vẽ box

| Rule | Quy tắc | Ca thật |
|---|---|---|
| **R-01** | Box là hình chữ nhật nhỏ nhất bao **mọi phần nhìn thấy** của một vật — kể cả các mảnh rời nhau do bị che ở giữa. Không tái dựng phần khuất, không thêm lề, không ôm bóng/vệt sáng. Box dừng ở mép ảnh. COCO là `[x, y, w, h]`. | `img_013` (người sau cây), `img_001` (khách sau bàn), `img_029` (người bị thanh thép che) |
| **R-02** | Phân biệt `truck` → `van` → `car` theo cấu trúc, đúng thứ tự ở §2.1. | `img_036` (`truck` thùng bạt), `img_005` (`van`), `img_023` (`car`) |
| **R-03** | **Người và xe là 2 đối tượng.** 1 người trên xe → 2 box (`person` + `motorcycle`); 2 người → **3 box**. Sidecar gộp vào box `motorcycle`, người trong sidecar vẫn 1 box `person`. Mũ/ba lô/khẩu trang thuộc box `person`; gương, tay lái, baga thuộc box xe. Box có thể chồng nhau. | `img_016` (2 box), `img_026` (3 box), `img_010`, `img_022` |
| **R-04** | **Xe chở xe:** 1 box `truck` cho cụm vận chuyển (đầu kéo + rơ-moóc = 1 box), + 1 box `car` cho **mỗi** xe được chở, che theo khung/giá. Không nâng box truck lên tới nóc xe con. | `img_002` (~8 xe trên 2 tầng), `img_024` (xe trên nóc — xem §4.3) |
| **R-05** | **Vật không thật → không gán, không tạo ignore:** tượng, bia mộ/kỷ vật, ma-nơ-canh, đồ chơi, mô hình, hình in trên biển/tường, hình vẽ, mascot. Xe **thật** được sơn/trang trí → gán theo thân xe; đồ chơi gắn trên xe → không gán. | `img_017` (tượng), `img_027` (bia mộ hình bán tải), `img_033` (ma-nơ-canh), `img_038` (đồ chơi), `img_018` (chân dung in), `img_019` (xe trang trí) |
| **R-06** | **Phản chiếu → không gán; nhìn xuyên kính → gán.** Gương, kính xe, cửa kính shop, mặt đường ướt, mũ bảo hiểm, vũng nước không tạo đối tượng. Vật thật nhìn **xuyên** kính trong suốt vẫn gán nếu rõ. Xe chủ (ego: gương, ca-pô, khung cửa của camera) **không gán**. Không phân biệt được xuyên/phản chiếu → `can_review`. | `img_020` (chỉ có gương + ego), `img_031`/`img_015` (phản chiếu mặt ướt), `img_040` (mũ), `img_032` (người sau kính nhà chờ → gán) |
| **R-07** | **Vật nhỏ:** đo `h = y_max − y_min` trên ảnh gốc. `h ≥ 20 px` → gán (đúng 20 px vẫn gán); `h < 20 px` → không gán, nếu là cụm thì 1 `ignore_region` gọn có lý do `small<20px`. Không kéo box để đạt 20 px, không cộng chiều cao hai vật. | `img_003`, `img_025`, `img_030` (xe chỉ còn ghi-đông) |
| **R-08** | **Đám đông:** gán **từng vật** khi thấy ≥ 50% thân, tách được bằng ≥ 2 dấu hiệu và `h ≥ 20 px` (`img_032` hàng người, `img_021` lớp gần). `ignore_region` **chỉ** cho cụm đã đo < 20 px hoặc chồng lấp không thể tách — gọn, không phủ vật hợp lệ, có lý do (`small<20px` / `inseparable`). Lưu ý mã QC: box có ≥ 50% diện tích nằm trong vùng ignore được coi là bị phủ → rà để không bỏ sót. | `img_032` (gán từng người), `img_021`, `img_030`, `img_008` |
| **R-09** | **Ngoài taxonomy → không gán lớp, không tạo lớp mới ở v1.0:** xe đạp, xích lô, tuk-tuk/auto-rickshaw, xe buýt (≥ 17 chỗ hoặc ≥ 2 cửa khách dọc thân), xe ba gác, xe đẩy hàng. **Người trên chúng vẫn gán `person`**. Nếu cả vùng chỉ có loại này và dễ nhầm với lớp thật → 1 `ignore_region` có lý do. | `img_003`, `img_006`, `img_010`, `img_023`, `img_029` (xe đạp/xích lô); `img_004`, `img_016`, `img_035`, `img_040` (tuk-tuk); `img_009`, `img_031`, `img_037` (buýt) |
| **R-10** | **Cắt mép:** đường bao bị mép ảnh cắt hở (vật tiếp tục ra ngoài) → `truncated=yes`, box tới mép, vẫn gán nếu `h ≥ 20 px` và nhận ra lớp. Chỉ chạm mép mà đường bao kín → `no`. Bị vật khác che **không** phải truncated. Một vật có thể vừa `occluded_level>0` vừa `truncated=yes`. | `img_012` (bán tải sát camera), `img_005`, `img_015` (2 xe hai mép), `img_026` (bánh cắt đáy), `img_040` |
| **R-12** | **Hàng hóa, phụ kiện:** box xe bao thùng/nắp thùng/giá gắn cố định; **không** kéo box theo hàng rời, quang gánh, thanh dài, tượng chở trên thùng. Hàng nằm gọn trong lòng thùng không làm phình box lên trên; hàng cao vượt thân → `can_review`. | `img_012`, `img_028` (tượng ngựa trong thùng), `img_029` (thanh dài), `img_035` (bao tải cao gấp đôi xe) |
| **R-13** | **Nhòe, thiếu sáng, chi tiết không chắc:** gán khi lớp và biên còn căn cứ, dùng đường bao thân chính — không ôm cả vệt chuyển động. Không dùng `occluded_level=3` hay `ignore_region` để thay cho "ảnh xấu". Không đặt được biên → ghi vấn đề cấp ảnh (`image_issues.csv`). | `img_014` (xe nhòe phía sau), `img_026`, `img_015` |
| **R-14** | **Ảnh không có vật:** ảnh vẫn phải có mặt trong Task và trong COCO, `annotations` rỗng. Không thêm box giả. | `img_038`, `img_017`, `img_020`, `img_027` |

### 3.1. Nguồn của từng rule sau khi hợp nhất

| Rule v1.0 | Lấy từ | Ghi chú |
|---|---|---|
| R-01 | 142 R-05, 162 R-01, 167 R-01/R-02, 171 R-01 | 4/4 giống nhau; thống nhất "mảnh rời vẫn 1 box" |
| R-02 | 142 R-02 (thứ tự cấu trúc) | 167/171 không có phép thử; chấp nhận vì đo được và không phụ thuộc cảm tính |
| R-03 | 142 R-06, 162 R-02, 167 R-04, 171 R-03 | 4/4; thêm sidecar + "mũ/ba lô thuộc person" |
| R-04 | 142 R-07, 162 R-03, 167 R-05, 171 R-04 | 4/4, khác nhau ở chi tiết; chốt: rơ-moóc = 1 truck |
| R-05 | 142 R-09, 162 R-06, 167 R-07, 171 R-05 | 4/4 |
| R-06 | 142 R-09, 162 R-06 (phản chiếu), 167 R-07/R-08, 171 R-06/R-07 | hợp nhất phản chiếu + xuyên kính + ego |
| R-07 | 142 R-10, 162 R-05, 167 R-09, 171 R-08 | 4/4, ngưỡng 20 px |
| R-08 | 142 R-11, 162 R-07, 167 R-09, 171 R-09 | chốt lại theo bản chặt nhất (142 + 171) |
| R-09 | 142 R-03, 162 "không thuộc lớp", 167 "ngoài taxonomy", 171 R-10 | 4/4 cùng hướng: không thêm lớp |
| R-10 | 142 R-08, 162 R-04, 167 R-03, 171 R-02 | chốt "đường bao hở", bỏ ngưỡng % |
| R-11 | 142 R-04 |
| R-12 | 142 R-07, 162 R-01, 167 R-04, 171 R-11 | |
| R-13 | 142 R-12, 162 R-13, 171 R-12 | |
| R-14 | 171 R-15, 142 §5.2 | 2 bản nêu rõ; cần cho độ phủ IAA |

## 4. Ca khó và bất đồng

### 4.1. Ca khó đã chốt (bảng quyết định dùng cho Pha 3–5)

| Ảnh | Tình huống | Rule | Quyết định v1.0 |
|---|---|---|---|
| `img_001` | Bán tải cũ cắt mép phải; ~6 khách sau bàn; người đứng trước xe | R-02, R-10, R-11 | 1 `truck` (`truncated=yes`); mỗi người 1 `person`, che `2` cho người sau bàn |
| `img_002` | Xe chở ô tô 2 tầng, ~8 xe con bị khung cam che | R-04, R-11 | 1 `truck` (đầu kéo + rơ-moóc); mỗi xe 1 `car` che `2`; người cuối xe `person` |
| `img_003` | Xe đạp chở hàng; dãy phố xa; 3 người bán hàng gần | R-07, R-09 | Không gán xe đạp; 1 `ignore_region` cho dãy phố xa; 3 `person` |
| `img_004` | Hai tuk-tuk 3 bánh, tài xế trong cabin | R-09 | Không gán tuk-tuk; 2 `person` cho tài xế, che `2` |
| `img_005` | Dòng xe máy chen nhau; van trắng phía sau; người cắt đáy | R-03, R-08, R-10 | Mỗi xe + mỗi người tách box; 1 `van`; nền dày → `ignore_region`; người cắt đáy `truncated=yes` |
| `img_006` | 3 xe máy từ phía sau (1–2 người/xe); 2 xe đạp phải; ô tô trắng cắt trái | R-03, R-09, R-10 | 3 `motorcycle` + 4 `person`; xe đạp không gán; ô tô `truncated=yes` |
| `img_007` | Bán tải có giá nóc + 2 xe thùng trong kho, xe phải cắt mép | R-02, R-10 | `truck` cho cả 3; xe phải `truncated=yes` |
| `img_008` | Ban đêm, hàng chục xe máy dựng sát nhau; 2 người giao hàng | R-03, R-08 | Gán các xe/người tách được ở lớp gần; nền dày → `ignore_region` |
| `img_009` | Minibus du lịch be `29B-0944` (Ford Transit 16 chỗ, "DULICH HOANMY") | R-02, R-09 | **`van`**, `occluded_level=0`, `truncated=no` — nóc liền một đường từ cabin ra đuôi, không thùng riêng, 1 cửa khách; **không** `can_review` |
| `img_010` | Xe máy 2 người đi xa; xe đạp thồ rau; người ngồi trên xe đỗ bên phải | R-03, R-09 | 2 `motorcycle` + 3 `person`; xe đạp không gán |
| `img_011` | Tượng quái vật khổng lồ (sơn tróc) trên bệ trưng bày; xe điêu khắc sắt vụn; đám đông trên sân | R-05, R-07, R-09 | Tượng + xe điêu khắc: **không box**. Người thật tách được (3 người gần trái + từng người trên sân có `h ≥ 20 px`) → `person`. Van trắng giữa sân đạt 20 px → `van` che `1` (bị tay tượng chắn). Đám xa < 20 px → 1 `ignore_region` `small<20px` |
| `img_012` | Bán tải chở đầy hàng, cắt phải + đáy; CX-5 trái; tải phía sau | R-02, R-10, R-12 | 1 `truck` (`truncated=yes`) không phình theo hàng; 1 `car`; tải xa nếu đạt 20 px |
| `img_013` | Hai người đi bộ ban đêm, mặt đường ướt phản chiếu | R-01, R-06 | `person` cho cả hai; phản chiếu dưới chân không gán |
| `img_014` | Hai người đi bộ nét; xe/người nhòe phía sau | R-13 | 2 `person`; vệt nhòe → `ignore_region`, không gán box |
| `img_015` | Xe đỏ trái, xe bạc phải, đều cắt mép; xe nhòe nền | R-10, R-13 | 2 `car` `truncated=yes`; xe nền nhòe → `ignore_region` |
| `img_016` | Scooter giữa; tuk-tuk hai bên; van; người áo đỏ cắt góc | R-02, R-03, R-09 | 1 `person` + 1 `motorcycle`; `van`; tuk-tuk không gán; người góc `truncated=yes` |
| `img_017` | Tượng đồng người cưỡi ngựa + chim thật | R-05, R-14 | Không gán gì; ảnh vẫn có trong COCO |
| `img_018` | Biển quảng cáo in chân dung; bãi xe sau lưới | R-05, R-06 | Không gán chân dung; `car` cho xe, che `1` do lưới |
| `img_019` | Xe vàng trang trí diễu hành, biển `YR04 NUM`, tài xế ngồi trong; mascot chuột bông; khán giả sau rào | R-03, R-05, R-08, R-10 | **`car`** (thân xe thật, có biển số, đang chạy, có người lái); tài xế `person` che `2`; mascot bông: **không gán**; khán giả hàng đầu (`h ≥ 20 px`, tách được) → `person` từng người; đám phía sau dày → 1 `ignore_region`; đầu người góc dưới–phải (nhòe, cắt khung) → `person` `truncated=yes` |
| `img_020` | Toàn ảnh là gương chiếu hậu + thân ego | R-06, R-14 | Không box nào; ảnh vẫn có trong COCO |
| `img_021` | Dòng xe máy dày đặc | R-03, R-08 | Gán 2–3 lớp gần nhất còn rõ biên; nền → 1 `ignore_region` |
| `img_022` | Mô tô + sidecar, đen trắng, 2 người | R-03 | 1 `motorcycle` (gộp sidecar) + 2 `person` (che `2` cho người trong sidecar) |
| `img_023` | Xích lô chở khách; Civic; xe máy; người bán hàng | R-04, R-09 | Xích lô không gán; `person` cho người đạp + khách; `car` cho Civic; `motorcycle` |
| `img_024` | Bảo tàng Rambler Ranch: wagon đen `Pacer` + thân AMC Pacer vàng tỉ lệ 1:1 trên giá nóc + dãy xe trưng bày | R-04, R-05 | **`car`** cho wagon đen; **`car`** cho thân xe vàng trên nóc (xe thật 1:1, có bánh/lốp thật, đặt trên giá — cùng logic `img_002`); `car` cho từng xe trong dãy phải và sedan kem trái. Không `ignore_region` — đây là vật thật, không phải mô hình |
| `img_025` | Vỉa hè: người gần/xa; xe máy có thùng bên phải | R-03, R-07 | Người/xe đạt 20 px → gán; người xa < 20 px → `ignore_region` |
| `img_026` | 2 người trên xe đỏ ban đêm; bánh cắt đáy; tranh tường | R-03, R-05, R-10 | 2 `person` + 1 `motorcycle` `truncated=yes`; tranh tường không gán |
| `img_027` | Bia mộ bằng đá hình bán tải | R-05, R-14 | Không box nào |
| `img_028` | Bán tải thật chở tượng ngựa; SUV đỏ phía sau | R-02, R-05, R-11 | `truck`; tượng không gán; `car` cho SUV, che `2` |
| `img_029` | Người trên scooter + người trên xe đạp cùng thanh thép dài | R-03, R-09, R-12 | `person` + `motorcycle`; xe đạp và thanh thép không gán; người bị thanh che `1` |
| `img_030` | Bãi xe máy cực dày; áp phích mặt người; người áo xanh | R-05, R-07, R-08 | 3–4 xe gần nhất + `person`; cả bãi → 1 `ignore_region`; áp phích không gán |
| `img_031` | Phố đêm: ô tô, buýt LED, người vỉa hè phải, phản chiếu | R-06, R-09 | `car`/`motorcycle`/`person` đạt ngưỡng; buýt không gán; phản chiếu không gán |
| `img_032` | ~30 người xếp hàng sau rào; ô tô xanh cắt trái | R-08, R-10 | Gán từng người (≥ 50% thân); ô tô `truncated=yes`; **không có xe buýt trong ảnh** (§4.2) |
| `img_033` | Ma-nơ-canh + phản chiếu + thùng ego | R-05, R-06, R-14 | Không box nào |
| `img_034` | Người qua đường; ô tô đen; 2 xe đạp; rider cắt phải | R-03, R-09, R-10 | `person` + `car` + `motorcycle`; xe đạp không gán; rider `truncated=yes` |
| `img_035` | Auto-rickshaw chở bao tải cao gấp đôi xe; xe máy; xe trắng | R-09, R-12 | Rickshaw + hàng không gán; `motorcycle` + `person`; `car` cho xe trắng |
| `img_036` | Tải nhỏ thùng bạt xanh; dãy scooter phía sau | R-02, R-08 | 1 `truck`; dãy scooter bị che → `ignore_region` |
| `img_037` | Chevrolet Silverado gắn thùng dịch vụ kín (nóc thùng cao hơn nóc cabin); 2 xe buýt xanh; 1 người đi bộ áo đen | R-02, R-09, R-12 | **`truck`** — thùng gắn trên sàn, có bậc/seam giữa nóc cabin và nóc thùng (phép thử "đường nóc" R-02 bước 4); `person` cho người áo đen bên trái; buýt: **không gán**; không `can_review` |
| `img_038` | Tủ kính toàn đồ chơi (xe mô hình ~1:43, vịt cao su, gấu bông) + phản chiếu | R-05, R-14 | **Không box nào, không `ignore_region`** — không có vật thật nào trong ảnh; ảnh vẫn phải có trong COCO với `annotations` rỗng |
| `img_039` | Xe đạp giữa đường; người đi bộ; scooter gần/xa | R-07, R-09 | Người/xe đạt ngưỡng gán; xe đạp không gán; vật xa đo trước |
| `img_040` | Người đội mũ sát camera cắt đáy; taxi; tuk-tuk; SUV; buýt | R-03, R-09, R-10 | `person` `truncated=yes`; `car` cho taxi/SUV; tuk-tuk + buýt không gán; phản chiếu trong mũ không gán |

### 4.2. Bảng giải quyết bất đồng giữa 4 bản cá nhân

| # | Vấn đề | 142 | 162 | 167 | 171 | **Chốt v1.0** | Lý do |
|---|---|---|---|---|---|---|---|
| 1 | Bán tải (`img_001`, `012`, `028`) | `truck` | "cân nhắc `truck`" | `car` | `truck` | **`truck`** | Taxonomy v0 định nghĩa `truck` = "xe có thùng hàng riêng"; bán tải có sàn hàng tách cabin. 3/4 bản cùng hướng; phép thử R-02 cho kết quả giống nhau mọi người |
| 2 | Bán tải gắn nắp thùng (`img_037`) | `truck` + review | — | `car` + review | `truck` | **`truck`** + `can_review` | Cùng lớp với bán tải; nắp là phụ kiện gắn liền, không biến xe thành `van` |
| 3 | Tuk-tuk / auto-rickshaw | bỏ qua | tạm `car` + review | bỏ qua + hỏi | bỏ qua (+ignore nếu lớn) | **Bỏ qua, không lớp mới** | 3/4; gán vào `car` làm hỏng định nghĩa 4 bánh thân kín. Người trên xe vẫn `person` |
| 4 | Xe buýt | bỏ qua | van ≤ 16 chỗ | bỏ qua | bỏ qua | **Bỏ qua** (≥ 17 chỗ hoặc ≥ 2 cửa khách dọc thân) | Không có lớp; dùng cấu trúc thay vì đếm ghế (không đo được trên ảnh nhỏ) |
| 5 | Xe đạp / xích lô | bỏ qua | bỏ qua | bỏ qua | bỏ qua | **Bỏ qua**, người vẫn gán | 4/4 |
| 6 | `occluded_level` | 0/30/70 | 10/30/60 | 10/40/80 | 10/30/70 | **<10 / 10–30 / 30–70 / >70** | 142 và 171 trùng nhau; 162 gần; 167 lệch nhiều nhất. Chọn bộ có mẫu số đo được và ít mức "không biết" |
| 7 | `truncated` | cắt là `yes` | cắt là `yes` | mất ≥ 5% | mất ≥ 10% | **Đường bao hở → `yes`** (bỏ ngưỡng %) | Ngưỡng % phải ước lượng → hai người ra hai kết quả. "Đường bao hở" là phép thử nhị phân, đo bằng mắt giống nhau |
| 8 | Đám đông | không phủ cả bãi; ignore có điều kiện | ≥ 2 dấu hiệu | ≥ 5 vật → ignore | ≥ 50% thân + ignore lớp sau | **R-08: gán từng vật nếu ≥ 50% thân, ≥ 2 dấu hiệu, ≥ 20 px; ignore chỉ cho cụm < 20 px hoặc không tách được** | Ngưỡng "≥ 5 vật" của 167 cho phép bỏ việc hợp pháp → mất nhãn và mất điểm "vật bỏ sót". Giữ đúng hướng 142 + 171 |
| 9 | Khẩu trang/mũ tính là che? | không | **có** | không | không | **Không tính** | Nếu tính, mọi người đeo khẩu trang thành mức 1 → phá vỡ đồng thuận thuộc tính và sai thực tế. Mũ/khẩu trang thuộc hình dáng người |
| 10 | Tượng / ma-nơ-canh / đồ chơi / hình in / phản chiếu / ego | bỏ qua | bỏ qua | bỏ qua | bỏ qua | **Bỏ qua, không ignore** | 4/4. Không dùng `ignore_region` để "che" phản ví dụ (142 nêu rõ) |
| 11 | `img_011` nhân vật lớn + xe điêu khắc | REVIEW | — | `person` | bỏ qua | **Bỏ qua (tượng trưng bày)**, không `can_review` | Bề mặt sơn tróc, kích thước gấp nhiều lần người bên dưới, đặt trên bệ trưng bày cạnh xe điêu khắc sắt vụn → tác phẩm, không phải người hóa trang đang đi lại. Người thật trong sân vẫn gán |
| 12 | `img_019` xe vàng diễu hành | REVIEW, tạm `car` | — | bỏ qua | `car` | **`car`**, không `can_review`; mascot không gán | Biển số `YR04 NUM` thật, xe đang chạy trên đường trong đoàn diễu hành, có tài xế ngồi trong khoang → thân xe thật, chỉ là trang trí bề mặt. Đây là ca "trông giả nhưng là thật" |
| 13 | `img_024` xe vàng trên nóc | bỏ qua (mô hình) | — | — | `car` | **`car`**, không `can_review` | Ảnh gốc cho thấy thân AMC Pacer **tỉ lệ 1:1** có bánh và lốp thật, đặt trên giá nóc → xe thật được vận chuyển/trưng bày, cùng logic `img_002`; không phải mô hình đồ chơi như `img_038` |
| 14 | `img_009` minibus | bỏ qua (buýt) | — | bỏ qua | `van` nếu ≤ 16 | **`van`**, không `can_review` | Ảnh gốc: Ford Transit 16 chỗ, nóc liền một đường từ cabin ra đuôi, **một** cửa khách, không thùng hàng tách rời → `van` theo R-02. Không đủ điều kiện "≥ 2 cửa khách dọc thân" để xếp vào buýt |
| 15 | Hàng trong lòng thùng (`img_012`) | không kéo box | không kéo | box bao cả hàng trong thùng | không kéo | **Không phình lên trên; box tới mép thùng** | Hàng nằm trong biên dạng xe không đổi kích thước xe; hàng vượt thân mới `can_review` |
| 16 | Ảnh không có vật | giữ ảnh trong COCO | — | — | giữ ảnh, `annotations` rỗng | **Giữ ảnh** | Cần cho độ phủ IAA (`coverage`) trong `qc_compare.py` |

### 4.3. Đã đóng — 40/40 ảnh có quyết định cuối, không còn ca mở

Ba ca từng để ngỏ đã được kiểm tra trên ảnh gốc. Không sửa lại sau khi khóa; nếu ai thấy khác thì mở ca
qua `calibration_decisions.csv` ở Pha 5, không tự đổi nhãn.

| Ảnh | Bằng chứng trên ảnh gốc | Quyết định cuối |
|---|---|---|
| `img_024` | Thân AMC Pacer vàng có bánh cao su và lốp thật, vành đúng tỉ lệ, đặt trên giá nóc của wagon đen; nền là showroom có dãy xe cổ thật | `car` (xe thật cỡ 1:1) — không phải mô hình |
| `img_009` | Ford Transit 16 chỗ, biển `29B-0944`, nóc liền một khối, một cửa khách phía trước | `van` |
| `img_037` | Nóc thùng dịch vụ **cao hơn** nóc cabin, có bậc và seam rõ; thùng gắn trên sàn bán tải | `truck` |

**Hệ quả cho Pha 3–5:** trong 40 ảnh này **không box nào cần tick `can_review`**. Thuộc tính vẫn giữ trong
taxonomy cho ảnh mới, nhưng nếu ai tick thì phải có lý do và phải xuất hiện trong `calibration_decisions.csv`.

### 4.4. Hiệu chỉnh mô tả ảnh giữa các bản (dùng khi tranh luận)

- `img_032` **không có xe buýt**: đó là hàng ~30 người chờ sau rào chắn + ô tô xanh cắt mép trái. Bản 167 ghi "bus tại trạm dừng" → cần sửa khi trích dẫn.
- `img_009` là **minibus**, không phải xe buýt thành phố; `img_037` mới có buýt thật (xanh, nền trái).
- Hai bánh thô sơ phân biệt rõ: **xe đạp** ở `img_006`, `img_010`, `img_029`, `img_034`, `img_039`; **xích lô** ở `img_023`. Không gộp chung một mô tả.
- `img_003` có xe đạp chở hàng, không phải xích lô.
- `img_022`: sidecar và người ngồi trong sidecar **đều thấy được** → 1 `motorcycle` + 2 `person` (142 để ngỏ, chốt lại ở đây).

## 5. QC và phiên bản

### 5.1. Ngưỡng áp dụng (lấy từ `submission/taxonomy.json`, **không sửa**)

| Trường | Giá trị | Dùng ở đâu |
|---|---|---|
| `min_box_height_px` | 20 | R-07, R-08 |
| `match_iou` | 0.5 | `qc_compare.py` ghép cặp box 1–1 (không phân biệt lớp) |
| `large_px` / `iou_thr_large` / `iou_thr_small` | 50 px / 0.85 / 0.70 | ngưỡng IoU theo chiều cao trung bình của cặp box |
| `iaa_iou` | 0.85 | IoU trung bình trên **các cặp ghép được** |
| `iaa_class` | 0.95 | tỉ lệ cùng lớp trên các cặp ghép được |
| `error_rate_img_max` | 0.02 | 1 ảnh lỗi / 40 = 2,5% → **đã vượt ngưỡng**, dẫn tới Rework chứ không Release |

IAA chỉ tính trên cặp ghép được: **không chứng minh không bỏ sót**, không chứng minh hai người cùng đúng.
Không có cặp → ghi `N/A`, không ghi 100%. Luôn ghi `tử số/mẫu số`.

### 5.2. Quy trình ca mơ hồ

1. Pha 3: box chưa chắc → tick `can_review` (**40 ảnh này: không dùng**); vùng không vẽ được → ghi `submission/qc/image_issues.csv`
   (`image, annotator, region, note`) **sau khi khóa nhãn, trước khi chạy `qc_compare.py`**.
2. Pha 5: mỗi ca bất thường một dòng trong `calibration_decisions.csv`, nêu mã rule (`R-xx`) và phạm vi
   `anh`/`lo`. Rule mới hoặc đổi nghĩa → `guideline_v1.1_calibrated.md` + `guideline_change.json`.
3. Người quyết: người điều phối chốt, người review xác nhận, ghi mã 3 chữ số.

### 5.3. Lịch sử phiên bản

| Phiên bản | Thay đổi | Lý do (ca nào) |
|---|---|---|
| v0 | Bản giảng viên: 6 lớp, 3 thuộc tính, 3 quy tắc | — |
| v0.1 (4 bản cá nhân) | Mỗi thành viên tự viết R-01…R-13/R-15, tự chọn ngưỡng thuộc tính | v0 thiếu ranh giới lớp, ngưỡng thuộc tính và quy tắc ca biên |
| **v1.0 (bản này)** | Hợp nhất 4 bản; chốt 16 bất đồng ở §4.2; 14 rule R-01…R-14; ngưỡng che 10/30/70, `truncated` theo đường bao hở; bán tải = `truck`; tuk-tuk/buýt/xe đạp/xích lô ngoài phạm vi; không thêm lớp, `rules` giữ nguyên; §4.3 đóng bằng kiểm tra ảnh gốc (`img_009`, `img_024`, `img_037`) và bỏ toàn bộ `can_review` | `img_002`, `img_004`, `img_005`, `img_009`, `img_011`, `img_019`, `img_024`, `img_032`, `img_037`, `img_038` |

**Trạng thái: đã chốt cho cả 40 ảnh** — không còn điểm mở (§4.3 đã đóng bằng kiểm tra ảnh gốc).
Họp Pha 2 chỉ còn việc 4 thành viên xác nhận và khóa bản này; không cần tranh luận thêm ca nào.
Sau khi khóa, mọi thay đổi làm đổi lớp/số box/biên/thuộc tính phải tạo phiên bản mới (v1.1) và ghi `kind`
trong `submission/guideline_change.json` là `semantic`, không ghi là sửa chính tả.
