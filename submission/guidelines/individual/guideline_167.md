# Guideline gán nhãn — v0.1 (cá nhân) — 167

- Phiên bản: v0.1 (bản cá nhân, Pha 1)
- Ngày: 2026-11-16
- Người viết / duyệt: 167
- Loại nhãn: Bounding box 2D trên ảnh đường phố (CVAT, xuất COCO 1.0)

---

## 1. Mục tiêu

Nhãn được dùng để huấn luyện và đánh giá mô hình **phát hiện vật thể 2D (2D Object Detection)** cho bài toán camera giao thông và xe tự hành, nhằm nhận diện **người** và **phương tiện cơ giới** đang lưu thông hoặc đỗ trong không gian đường phố thực tế.

**Lỗi nào gây nguy hiểm cho mô hình:**
- **Bỏ sót (False Negative)** người đi bộ, người đi xe máy, ô tô, xe tải thật trên đường → mô hình không cảnh báo va chạm.
- **Gán nhầm vật giả thành vật thật (False Positive)** như tượng, ma-nơ-canh, ảnh người trên biển quảng cáo, mô hình đồ chơi trong tủ kính → mô hình phanh gấp hoặc đánh lái sai.
- **Box lỏng lẻo hoặc gộp sai đối tượng** (ví dụ gộp chung người lái và xe máy thành 1 box, hoặc vẽ box trùm cả hàng hóa cồng kềnh) → sai lệch vị trí, kích thước và ước lượng khoảng cách.

---

## 2. Lớp và thuộc tính

### Lớp (mỗi vật thuộc đúng một lớp)

Giữ nguyên 6 lớp của `submission/taxonomy.json` :

| Lớp | Định nghĩa (một câu) | Ví dụ | Phản ví dụ / dễ nhầm với |
|---|---|---|---|
| `car` | Ô tô con chở người (sedan, hatchback, SUV, MPV, xe bán tải phổ thông). | Xe 4–7 chỗ, taxi, SUV, xe bán tải (`img_012`). | Xe `van` thùng kín chuyên chở; `truck` (`img_001`, `img_002`, `img_036`); xe đồ chơi mô hình (`img_038`). |
| `truck` | Xe tải, xe đầu kéo, xe chuyên chở hàng hóa có thùng hàng hoặc sàn chở tách biệt với cabin. | Xe tải (`img_001`), xe chở ô tô (`img_002`), xe tải giao hàng (`img_007`), xe tải nhỏ kei-truck (`img_036`). | Xe bán tải cá nhân (`car`); xe `van` liền thân thùng kín. |
| `van` | Xe thùng kín cỡ nhỏ/trung có thân liền khối dùng chở hàng hoặc chở khách kiểu minibus/cứu thương. | Xe van giao hàng liền khối, minivan chở hàng không có kính hông sau. | `car` (SUV/MPV gia đình có hàng ghế và cửa kính đầy đủ); `truck` (cabin và thùng tách rời). |
| `motorcycle` | Xe cơ giới 2 bánh (hoặc 3 bánh dạng sidecar gắn liền) có động cơ. | Xe máy, xe tay ga, mô tô phân khối lớn (`img_005`, `img_030`), xe máy có sidecar (`img_022`). | Người lái xe máy (`person`); xe đạp / xích lô đạp (`img_006`); tuk-tuk (`img_004`). |
| `person` | Người thật (đi bộ, đứng, ngồi, điều khiển hoặc ngồi sau phương tiện). | Người đi bộ (`img_014`), người lái/ngồi sau xe máy (`img_005`, `img_026`), người mặc đồ hóa trang thật (`img_011`). | Tượng (`img_017`), ma-nơ-canh (`img_033`), ảnh người trên biển quảng cáo (`img_018`), búp bê (`img_038`). |
| `ignore_region` | Vùng bỏ qua (không phải vật đơn lẻ), nơi các đối tượng quá nhỏ hoặc chồng lấn dày đặc không thể tách box chính xác; bỏ sót trong vùng này không tính là lỗi. | Đám đông xa ở chợ/lễ hội, bãi giữ xe máy xếp san sát phía hậu cảnh không tách được từng xe. | Một nhóm nhỏ 2–3 người đứng gần nhau vẫn nhìn rõ đường biên từng người. |

**Xử lý vật không thuộc lớp nào (xe buýt, tuk-tuk/auto rickshaw, xích lô, xe đạp):**
- Trong bản cá nhân v0.1, **không tự bịa lớp mới** để tránh lệch schema `taxonomy.json`.
- **Không gán** phương tiện thô sơ không động cơ (xích lô đạp, xe đạp) và xe buýt cỡ lớn nếu chưa có lớp `bus`; tuy nhiên vẫn **gán `person`** cho người đạp xích lô / hành khách / người đi xe đạp nhìn thấy rõ.
- Với **tuk-tuk / auto rickshaw** (`img_004`, `img_035`) và **xe buýt** (`img_032`): tạm thời **không gán** vào `car`/`motorcycle` (tránh làm nhiễu định nghĩa lớp), ghi nhận thành **ca cần hỏi reviewer / đưa vào họp nhóm Pha 2** để thống nhất bỏ qua hay đề xuất bổ sung quy tắc.

### Thuộc tính

| Thuộc tính | Giá trị cố định | Ý nghĩa & ngưỡng cụ thể |
|---|---|---|
| `occluded_level` | `0`, `1`, `2`, `3` | **Mức độ bị che khuất** (tính theo % diện tích ước lượng của vật bị vật khác che mất):<br>• `0`: Thấy rõ hoàn toàn hoặc bị che **< 10%**. <br>• `1`: Bị che ít, từ **10% đến < 40%** diện tích vật.<br>• `2`: Bị che nhiều, từ **40% đến < 80%** diện tích vật (vẫn nhận diện chắc chắn là lớp nào).<br>• `3`: Bị che rất nặng (**≥ 80%**) hoặc chỉ thấy một mảnh nhỏ khó xác định rõ hình dáng toàn vật. |
| `truncated` | `no`, `yes` | **Vật bị cắt bởi mép khung ảnh:**<br>• `no`: Toàn bộ phần nhìn thấy nằm trọn bên trong khung ảnh (cách mép ≥ 2 px hoặc không có cạnh bị cắt).<br>• `yes`: Một phần thân vật bị cắt cụt trực tiếp bởi cạnh trái/phải/trên/dưới của ảnh (ước lượng mất ≥ 5% vật). |
| `can_review` | `false` (không tick) / `true` (tick) | Tick `true` khi gặp ca mơ hồ về lớp, ranh giới box hoặc cần reviewer/nhóm chốt thêm ở Pha 2 / Pha 5. Mặc định `false`. |

---

## 3. Quy tắc vẽ box

- **`R-01` — Box ôm sát phần nhìn thấy (Visible-only):** Vẽ box khít nhất có thể quanh phần nhìn thấy được của vật (sai số mép ≤ 2–3 px). **Không** kéo rộng box ra ngoài vùng bị che hoặc ngoài mép ảnh để đoán phần bị khuất.
- **`R-02` — Vật bị che khuất:** Nếu một vật bị chia đôi bởi cột điện/cây nhỏ nhưng rõ ràng là cùng một vật, vẽ **1 box** bao toàn bộ các phần nhìn thấy của vật đó và đặt `occluded_level` theo tổng % bị che. Nếu bị che **> 90%** và không còn nhận diện được hình dáng lớp, bỏ qua (không gán).
- **`R-03` — Vật bị cắt mép ảnh:** Vẫn gán box cho phần nằm trong ảnh nếu chiều cao phần nhìn thấy **≥ 20 px** và nhận diện được lớp; đặt `truncated = yes`.
- **`R-04` — Người đi xe máy & vật nhiều phần:**
  - Tách riêng **`person`** (người lái và từng người ngồi sau) và **`motorcycle`** (xe máy). Không gộp người và xe thành 1 box.
  - Mũ bảo hiểm, ba lô đang đeo tính vào box `person`.
  - Hàng hóa chở cồng kềnh trên yên/baga (`img_029`): **không** kéo box `motorcycle` trùm lên hàng hóa rời (tránh làm biến dạng kích thước chuẩn của xe máy); chỉ bao phần thân xe (bánh, khung, tay lái, đèn, biển số).
  - Xe máy có thùng bên (`sidecar`, `img_022`): gộp phần `sidecar` gắn cố định vào cùng box `motorcycle`; người ngồi trong `sidecar` vẫn gán box `person` riêng.
- **`R-05` — Xe chở xe (`car carrier`):** Với xe tải chuyên dụng chở nhiều ô tô (`img_002`), gán 1 box `truck` cho xe tải/rơ-moóc chở, và gán riêng từng box `car` cho mỗi ô tô con nằm trên sàn chở nếu thấy rõ và đạt kích thước tối thiểu.
- **`R-06` — Phân biệt `car`, `van`, `truck` với xe bán tải (pickup) & xe tải:**
  - Xe bán tải phổ thông (`img_012`) → gán **`car`**.
  - Xe tải có cabin và thùng/sàn chở hàng tách biệt (kể cả xe tải cỡ vừa/nhỏ như `img_001`, `img_036`) → gán **`truck`**.
  - Xe bán tải lắp nắp thùng kín (`truck cap`, `img_037`) → tạm xếp vào **`car`** và tick `can_review` (vì dễ nhầm với `van`/`truck`, cần cả nhóm thống nhất ở Pha 2).
- **`R-07` — Phản chiếu, ảnh in, tượng, ma-nơ-canh, đồ chơi, xe giả tưởng:**
  - **Loại theo quy tắc (không gán):** Tượng người/động vật (`img_017`), tượng đài mô phỏng xe (`img_027`), ma-nơ-canh (`img_033`), hình người/xe trên biển quảng cáo (`img_018`), búp bê và mô hình xe đồ chơi trong tủ kính (`img_038`), hình phản chiếu trong gương chiếu hậu hoặc mặt kính (`img_020`).
  - **Người thật hóa trang** (`img_011` - người mặc trang phục Predator): vẫn gán `person` vì là con người thật đang đứng trong không gian giao thông.
  - **Xe diễu hành / xe giả tưởng** (`img_019` - xe mô hình nhân vật): tạm thời **không gán** (hoặc tick `can_review` nếu có khung xe thật bên dưới) để đưa ra họp nhóm Pha 2.
- **`R-08` — Xe chủ (Ego vehicle):** Gương chiếu hậu, capo hoặc khung cửa thuộc chính phương tiện đang gắn camera (`img_020`) thì **không gán**.
- **`R-09` — Vật quá nhỏ & đám đông:**
  - Theo `taxonomy.json`, chỉ gán box đơn lẻ khi chiều cao box **≥ 20 px** (`min_box_height_px = 20`). Vật nhỏ hơn 20 px đứng lẻ thì bỏ qua.
  - Nếu có **≥ 5 người hoặc ≥ 5 xe** tụ tập/xếp san sát ở xa, chồng lấn mạnh không thể tách từng box chính xác, vẽ 1 box `ignore_region` bao vùng đó thay vì cố vẽ từng box sai lệch.

---

## 4. Ca khó

| Ảnh | Tình huống | Rule ID | Quyết định | Lý do |
|---|---|---|---|---|
| `img_001` | Xe tải đỗ trước quán ăn + khách ngồi ăn sát lề đường | `R-06`, `R-01` | Gán `truck` cho xe tải; gán `person` cho từng người thấy rõ ≥ 20 px | Là xe tải chuyên chở có thùng hàng phía sau chứ không phải xe bán tải; người ngồi quán vẫn là người thật trong khung cảnh đường phố. |
| `img_002` | Xe tải `Renault 380` chở nhiều ô tô con trên sàn | `R-05` | Gán 1 box `truck` cho xe chở; gán riêng từng `car` trên sàn nếu ≥ 20 px | Mô hình cần nhận diện cả xe tải lớn lẫn từng ô tô con nhìn thấy rõ. |
| `img_004` | Hai chiếc `tuk-tuk` xanh đỗ trên vỉa hè | `R-04` *(ngoài taxonomy)* | **Cần hỏi reviewer** (tạm không gán, ghi chú cho Pha 2) | Tuk-tuk là xe 3 bánh chở khách, không khớp hẳn `motorcycle` hay `car` trong v0. |
| `img_005`, `img_026`, `img_030` | Nhiều người đi xe máy đi sát nhau / chở đôi | `R-04`, `R-02` | Tách riêng `person` (từng người) và `motorcycle`, đặt `occluded_level` theo % che | Đảm bảo nhất quán số lượng box cho người và xe khi chồng lấn. |
| `img_006`, `img_009`, `img_023`, `img_034`, `img_039` | Xe xích lô (`Cyclo`) chở khách trên đường | *(ngoài taxonomy)* | Gán `person` cho người đạp và khách; **không gán** khung xe xích lô (đưa vào Pha 2) | Xích lô là xe thô sơ không động cơ, không thuộc 5 lớp phương tiện cơ giới của v0. |
| `img_011` | Người mặc đồ hóa trang `Predator` đứng cạnh dàn mô tô | `R-07` | Gán `person` cho người hóa trang; gán `motorcycle` cho từng xe | Bên trong trang phục là người thật có kích thước và chuyển động như người đi bộ. |
| `img_012` | Xe bán tải chở đầy thùng hàng/túi ở thùng sau | `R-06`, `R-01` | Gán `car` (ghi chú đưa ra Pha 2 để nhóm thống nhất `car` vs `truck` cho pickup); box bao cả phần hàng nằm gọn trong thùng | Theo quy ước xe bán tải thuộc lớp `car`; hàng nằm trong lòng thùng thuộc biên dạng tổng thể của xe. |
| `img_017` | Tượng Công tước Wellington cưỡi ngựa trên bệ | `R-07` | **Loại theo quy tắc** (không gán `person`) | Là tượng điêu khắc tĩnh trên bệ, không phải người thật tham gia giao thông. |
| `img_018` | Ảnh chân dung người trên biển quảng cáo bầu cử | `R-07` | **Loại theo quy tắc** (không gán `person`) | Là hình ảnh 2D in trên biển quảng cáo, tránh gây False Positive. |
| `img_019` | Xe mô hình `Dangermouse` trong lễ diễu hành | `R-07` | **Cần hỏi reviewer** (tạm không gán, ghi chú cho Pha 2) | Xe giả tưởng/mô hình diễu hành dễ gây nhiễu đặc trưng của lớp `car`. |
| `img_020` | Gương chiếu hậu của xe đang chụp (ego) có phản chiếu | `R-08`, `R-07` | **Loại theo quy tắc** (không gán gương xe chủ hay hình phản chiếu) | Gương thuộc xe gắn camera; ảnh trong gương là phản chiếu ảo. |
| `img_022` | Xe máy gắn thùng bên (`sidecar`) | `R-04` | Gán 1 box `motorcycle` bao cả xe và sidecar; người ngồi gán `person` riêng | Sidecar gắn liền cơ học với xe máy thành một phương tiện duy nhất. |
| `img_027` | Tượng đài xe bán tải (`Pickup Truck Monument`) | `R-07` | **Loại theo quy tắc** (không gán `car`/`truck`) | Là tượng đài cố định, không phải phương tiện giao thông thật. |
| `img_029` | Xe máy chở hàng hóa cồng kềnh vượt quá thân xe | `R-04` | Gán `motorcycle` ôm phần thân/bánh xe thật; không kéo box trùm hết đống hàng rời | Tránh làm sai lệch tỉ lệ khung hình chuẩn của lớp `motorcycle`. |
| `img_032` | Xe buýt tại trạm dừng giờ cao điểm (`ETS Bus Stop`) | *(ngoài taxonomy)* | **Cần hỏi reviewer** (tạm không gán vào `truck`/`van`, đưa ra Pha 2) | Xe buýt không có trong 6 lớp v0; cần nhóm thống nhất bỏ qua hay quy ước riêng. |
| `img_033` | Ma-nơ-canh (`Mannequin`) đứng trên phố / tủ kính | `R-07` | **Loại theo quy tắc** (không gán `person`) | Là hình nộm, không phải người thật. |
| `img_035` | Xe `Auto Rickshaw` chở quá tải nhiều người | *(ngoài taxonomy)* | Gán `person` cho người nhìn thấy rõ; xe 3 bánh đưa vào **cần hỏi reviewer** ở Pha 2 | Tương tự `tuk-tuk`, chưa có định nghĩa rõ trong v0. |
| `img_036` | Xe tải nhỏ đầu bằng (`CMC Veryca truck`) | `R-06` | Gán `truck` | Có cabin và thùng chở hàng tách biệt rõ ràng. |
| `img_037` | Xe bán tải lắp nắp thùng kín (`Master 75 Truck Cap`) | `R-06` | Tạm gán `car` + tick `can_review` để chốt ở Pha 2 | Bản chất là xe bán tải (`car`) gắn thêm nắp thùng, dễ nhầm giữa `car`, `van` và `truck`. |
| `img_038` | Tủ kính cửa hàng chứa nhiều búp bê, mô hình ô tô đồ chơi | `R-07` | **Loại theo quy tắc** (không gán `person` hay `car`) | Toàn bộ là đồ chơi kích thước nhỏ sau mặt kính cửa hàng. |

---

## 5. QC và phiên bản

- **Ngưỡng QC áp dụng** (theo `submission/taxonomy.json`, không thay đổi mục `rules`):
  - Chiều cao box tối thiểu: `min_box_height_px = 20` px.
  - Ngưỡng IoU chuẩn theo kích thước (`large_px = 50` px): `iou_thr_large = 0.85` (box ≥ 50 px), `iou_thr_small = 0.70` (box < 50 px).
  - Ngưỡng đồng thuận liên người gán (IAA): `iaa_iou = 0.85`, `iaa_class = 0.95`, `match_iou = 0.50`.
  - Tỉ lệ lỗi tối đa theo ảnh để đạt **Release**: `error_rate_img_max = 0.02` (≤ 2% số ảnh).
- **Quy trình xử lý ca mơ hồ:**
  - Trong Pha 3 (gán độc lập): người gán tick `can_review` trên box chưa chắc hoặc ghi lại ca cấp ảnh.
  - Ở Pha 2 và Pha 5: cả nhóm họp chốt quy tắc (gắn `Rule ID`), người điều phối/reviewer ghi nhận vào `calibration_decisions.csv` và cập nhật phiên bản guideline.
- **Lịch sử thay đổi:**

| Phiên bản | Thay đổi | Lý do (ca nào) |
|---|---|---|
| `v0` | Bản khởi đầu của giảng viên | Thiếu ngưỡng thuộc tính và quy tắc ca khó |
| `v0.1` | Bổ sung ngưỡng `occluded_level`/`truncated`, quy tắc `R-01`..`R-09` và bảng 20+ ca khó trên 40 ảnh | Chuẩn bị thảo luận thống nhất `v1.0` ở Pha 2 (`img_001`..`img_039`) |
