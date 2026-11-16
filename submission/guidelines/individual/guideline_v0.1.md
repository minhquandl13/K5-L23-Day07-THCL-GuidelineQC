# Guideline gán nhãn người và phương tiện — v0.1

- **Phiên bản:** v0.1.
- **Ngày:** 2026-11-16.
- **Phạm vi:** 40 ảnh `img_001.jpg`–`img_040.jpg` trong [data/images](../../../data/images/).
- **Loại nhãn:** bounding box chữ nhật song song trục ảnh; dự kiến dùng CVAT, xuất COCO 1.0.
- **Soạn thảo:** Codex hỗ trợ đọc tài liệu, xem 40 ảnh và viết theo yêu cầu trực tiếp; chưa có người dùng/reviewer xác nhận các quyết định. Không có box hoặc pre-label được tạo ở Pha 1.
- **Tên tệp:** giữ `guideline_v0.1.md` theo yêu cầu; không thực hiện phần nhóm hoặc gửi riêng.

## 1. Mục tiêu

Huấn luyện mô hình phát hiện **người thật và các loại phương tiện thật thuộc phạm vi taxonomy** trên ảnh 2D. Đầu ra là vị trí phần nhìn thấy và lớp của từng đối tượng, không phải tái dựng hình dạng 3D, suy đoán vận tốc, chức năng xe hay danh tính người.

Ảnh ban ngày, ban đêm, mưa, bãi đỗ và vật thật đang trưng bày đều được xem xét. Phương tiện đang đỗ hoặc đang được vận chuyển vẫn là đối tượng. Không cần chứng minh xe đang chạy hoặc động cơ hoạt động.

Các sai lệch cần tránh: gộp người vào xe làm mất người; bỏ xe vì đang chở trên xe khác; nhận tượng/đồ chơi/phản chiếu thành vật thật; gán xe ngoài taxonomy vào lớp gần giống; mở rộng box theo phần tưởng tượng. Các lỗi đó làm sai dữ liệu học và có thể khiến mô hình bỏ sót hoặc báo nhầm trong cảnh giao thông.

Quy tắc này phục vụ bài lab. Ngưỡng QC của lab không phải chứng nhận an toàn cho triển khai xe tự lái. Nguồn và ghi công ảnh giữ tại [ATTRIBUTION.csv](../../../data/ATTRIBUTION.csv).

## 2. Lớp và thuộc tính

### 2.1. Lớp — R-01: mỗi đối tượng một lớp

Giữ nguyên tên lớp trong [taxonomy v0](../../taxonomy.json). Không tạo thêm `rider`, `bus`, `bicycle` hoặc `tuk_tuk` ở Pha 1. Quy tắc dưới đây là đề xuất làm rõ nghĩa; taxonomy hiện chưa được nâng phiên bản.

| Lớp | Định nghĩa sử dụng trong v0.1 | Ví dụ thật trong bộ ảnh | Phản ví dụ / phân biệt |
|---|---|---|---|
| `car` | Ô tô con dạng sedan, hatchback, coupe, wagon, SUV/MPV có thân khoang hành khách liền, không có thùng hàng riêng. | SUV bên trái `img_012`; xe đen phía trước `img_024`; taxi ở `img_040`. | Bán tải `img_028` là `truck`; xe thân van `img_005` là `van`; ô tô đồ chơi `img_038` bị loại. |
| `truck` | Xe có cabin và thùng/sàn hàng riêng, gồm bán tải, xe tải nhẹ, tải thùng kín và xe chở ô tô. | Xe chở xe `img_002`; bán tải `img_012`; tải nhỏ `img_036`. | Kích thước nhỏ không làm tải thành `van`; SUV không có thùng hàng riêng là `car`; tượng bán tải `img_027` bị loại. |
| `van` | Xe thân hộp cao, nóc liền từ cabin ra khoang sau, không thấy thùng hàng tách rời; có thể chở khách hoặc hàng. | Xe màu bạc sau người đi xe máy ở `img_005`; xe trắng phía sau ở `img_016`. | Xe buýt thân dài, cửa lên xuống riêng và nhiều cửa sổ dọc thân ở `img_009` nằm ngoài phạm vi; bán tải có nắp thùng `img_037` không tự động là `van`. |
| `motorcycle` | Xe máy/mô tô/scooter có động cơ nhận biết qua cấu trúc xe; gồm mô tô gắn thùng phụ sidecar thành một cụm. | Scooter giữa `img_016`; xe đỏ `img_026`; mô tô sidecar `img_022`. | Xe đạp `img_029` và xích lô `img_023` không phải xe máy; tuk-tuk `img_004` không nằm trong định nghĩa này. |
| `person` | Một người thật nhìn thấy trực tiếp, dù đi bộ, đứng, ngồi hoặc đi trên bất kỳ phương tiện nào. | Người đi bộ `img_034`; hai người trên xe máy `img_026`; người chờ xe `img_032`. | Tượng `img_017`, ma-nơ-canh `img_033`, chân dung in `img_018` bị loại; không suy ra người chỉ từ xe/mũ rời. |
| `ignore_region` | Vùng loại khỏi việc gán từng đối tượng vì vật quá nhỏ hoặc có cụm đối tượng không thể tách đáng tin cậy; đây không phải lớp vật thể. | Cụm xa trong `img_021` hoặc `img_025` **sau khi đo/kiểm tra** theo R-10/R-11. | Không dùng cho tượng, hình in, xe ngoài taxonomy đã xác định chắc chắn; không phủ toàn ảnh chỉ vì đông hoặc tối. |

**R-02 — Trật tự phân biệt xe:** tìm cấu trúc thùng hàng riêng trước (`truck`), rồi thân van liền nóc (`van`), rồi hình dáng ô tô con (`car`). Với van/minibus/bus, dùng cấu trúc nhìn thấy, không suy số ghế, tải trọng, nhãn hiệu hoặc mục đích sử dụng. Không đủ dấu hiệu để chọn một lớp thì áp dụng R-13.

**R-03 — Ngoài taxonomy:** xe buýt, tuk-tuk/auto-rickshaw, xe đạp, xích lô không động cơ, động vật và đồ vật khác được **bỏ qua theo phạm vi**, không có box lớp đối tượng và không tự tạo vùng ignore. Người thật nhìn thấy trên chúng vẫn được xét riêng cho `person`. Bỏ qua vì thiếu lớp khác với không nhận diện được vật: trường hợp thứ hai cần REVIEW. Có thể đề xuất mở rộng taxonomy ở phiên bản sau, nhưng không tự làm trong v0.1.

### 2.2. Thuộc tính — R-04: ghi đủ và đúng mức

Mỗi box đối tượng có đủ ba thuộc tính bên dưới. Không để giá trị mặc định thay cho việc kiểm tra. `ignore_region` không cần diễn giải mức che/cắt như một vật; nếu giao diện bắt buộc, giữ `0`, `no`, `false` và ghi lý do vùng ở ghi chú riêng.

**Cách ước lượng che khuất:** gọi `q` là phần trăm diện tích hình dáng chiếu 2D của vật **nằm trong khung ảnh** bị vật khác che. Ước lượng `q = 100 × diện tích bị che / diện tích hình dáng dự kiến trong khung`. Không lấy diện tích toàn hình chữ nhật làm mẫu số. Phần ngoài ảnh được xử lý bằng `truncated`, không cộng vào che khuất. Đây là ước lượng để chọn mức, không yêu cầu vẽ thêm hình dáng tưởng tượng.

| Thuộc tính | Giá trị cố định | Ranh giới và cách dùng |
|---|---|---|
| `occluded_level` | `0` | `q = 0%`: không thấy bằng chứng vật khác che. Góc nhìn chỉ cho thấy một bên xe không tự tính là che khuất. |
| `occluded_level` | `1` | `0% < q ≤ 30%`. |
| `occluded_level` | `2` | `30% < q ≤ 70%`. |
| `occluded_level` | `3` | `70% < q < 100%`, vẫn nhận ra lớp và tách được một đối tượng. Vật bị che hoàn toàn không được suy ra để gán. |
| `truncated` | `yes` | Đối tượng thực sự kéo dài ra ngoài ít nhất một mép ảnh, dù chỉ một phần nhỏ. Chỉ chạm mép nhưng còn nguyên vẹn thì `no`. |
| `truncated` | `no` | Không có bằng chứng bị khung ảnh cắt; bị vật khác che không phải bị cắt mép. |
| `can_review` | `true` / tick | Có quyết định tạm nhưng chưa chắc lớp, biên box, mức thuộc tính hoặc tính thật/giả; phải ghi rõ ảnh, vùng, điều chưa chắc và R-ID. |
| `can_review` | `false` / không tick | Áp dụng rõ quy tắc, không còn điểm cần hỏi trên box. Không có nghĩa là box đã được reviewer duyệt. |

- Đúng **30% → mức 1**, đúng **70% → mức 2**; không dùng khoảng 1–30/31–70 gây hở các số thập phân.
- Khi ước lượng dao động qua ranh giới 30% hoặc 70%, chọn mức theo ước lượng tốt nhất và tick `can_review`. Nếu không thể xác định cả lớp/biên, ghi vấn đề cấp ảnh theo R-13; không dùng mức `3` làm mã “không biết”.
- Quần áo, mũ đội đầu của chính người nằm trong hình dáng `person`; khẩu trang không làm người thành vật bị che hoàn toàn. Người khác, xe, hàng hóa, cột, hàng rào có thể gây che.
- Kính trong cho thấy vật phía sau không tự gây occlusion. Phản sáng/kính đục che mất hình dáng thì xét phần mất đó; nếu không chắc nhìn xuyên hay phản chiếu, cần REVIEW.
- Bóng tối, nhòe chuyển động và ảnh đen trắng không tự làm tăng `occluded_level`; chúng có thể làm biên/lớp chưa chắc và kích hoạt `can_review`.
- Cờ occluded có sẵn của CVAT là trường khác; lab dùng `occluded_level`.

## 3. Quy tắc vẽ box

### R-05 — Chỉ ôm phần nhìn thấy, không tái dựng phần khuất

Một box là hình chữ nhật nhỏ nhất bao **mọi phần nhìn thấy thuộc cùng một đối tượng**, gồm cả các phần rời nhau do bị che. Không tách đầu/tay/chân thành nhiều người; không bao thêm phần cơ thể hay xe bị khuất chỉ vì có thể đoán được.

- Lấy biên trái/trên/phải/dưới ở cực trị phần nhìn thấy. Không thêm lề cố định, bóng đổ, vệt phản chiếu hoặc vệt sáng kéo dài.
- `person`: gồm tóc, mũ đang đội, quần áo, tay/chân nhìn thấy; không cố kéo box để chứa túi xách, quang gánh, hàng hay xe. Khoảng trống hoặc vật khác nằm trong hình chữ nhật do tư thế là chấp nhận được.
- Phương tiện: gồm thân, bánh, gương và phụ kiện gắn cố định như thùng/nắp thùng, giá xe; không kéo box theo người ngồi hoặc hàng rời nhô ra. Nếu bộ phận gắn cố định chưa nhận diện được thì REVIEW.
- Giới hạn tọa độ theo ảnh gốc: `0 ≤ x_min < x_max ≤ W`, `0 ≤ y_min < y_max ≤ H`. COCO dùng `[x_min, y_min, width, height]`, không phải `[x_min, y_min, x_max, y_max]`.
- Không crop/resize/đổi tên ảnh nguồn để làm vật vượt ngưỡng. Zoom giao diện chỉ phục vụ quan sát; đo pixel trên ảnh gốc.

### R-06 — Người và xe là các đối tượng riêng

Một người trên một xe máy đủ điều kiện → **2 box: 1 `person` + 1 `motorcycle`**. Hai người trên một xe → **3 box: 2 `person` + 1 `motorcycle`**. Không có lớp `rider`; không vẽ box nhóm người–xe. Mỗi box tự xét ngưỡng nhỏ, cắt mép và che khuất.

Box có thể chồng lên nhau. Biên box xe không mở lên đỉnh mũ của người; biên box người không kéo xuống bánh xe. Không suy người ngồi sau nếu chỉ có dấu hiệu không đủ xác định một người.

**Phép thử nhất quán, chỉ đếm cụm được chỉ rõ, không phải tổng box toàn ảnh:**

| Ảnh và vùng thử | Quyết định v0.1 | Phần bao trong box |
|---|---|---|
| [img_016](../../../data/images/img_016.jpg), người áo xanh trên scooter xanh giữa ảnh | 2 box: `person` + `motorcycle`. | Người: mũ, thân, tay/chân nhìn thấy; xe: gương/tay lái, thân và bánh nhìn thấy. |
| [img_026](../../../data/images/img_026.jpg), đôi người trên xe đỏ | 3 box: hai `person` riêng + một `motorcycle`. | Mỗi người chỉ gồm phần thấy của người đó; xe gồm phần xe còn thấy tới mép dưới, `truncated=yes` cho xe. |
| [img_010](../../../data/images/img_010.jpg), người áo hoa trên scooter nâu bên phải | 2 box: `person` + `motorcycle`. | Dù đang dừng, vẫn tách người và xe; không bao sạp hàng vào hai box. |

### R-07 — Xe chở xe và vật có nhiều bộ phận

- `img_002`: đề xuất **1 `truck` cho xe chở + 8 `car` cho các ô tô trên hai tầng** (5 trên, 3 dưới). Đây là 9 box dự kiến cho cụm xe, không phải annotation đã vẽ hay tổng đối tượng toàn ảnh. Người đứng phía phải, nếu đủ điều kiện, là box `person` riêng.
- Box truck bao cabin, khung/sàn chở và bánh; không nâng biên theo nóc xe con. Mỗi car ôm phần nhìn thấy của chính nó. Khung sắt hoặc xe khác chắn car được tính vào che khuất.
- Một đầu kéo và rơ-moóc đang nối thành cụm vận chuyển được coi là một `truck` trong bản này. Không tách sàn/khung/bánh thành lớp khác.
- Mô tô và sidecar gắn liền (`img_022`) là một `motorcycle`; người nhìn thấy trên cụm được xét riêng. Sidecar không phải tuk-tuk độc lập có cabin.
- Hàng rời, quang gánh, thanh dài, tượng chở trên thùng (`img_028`, `img_029`) không làm thay đổi lớp hoặc kéo rộng box xe. Chỉ phần xe thật thuộc taxonomy được gán.

### R-08 — Che khuất và cắt mép độc lập

Nếu còn đủ bằng chứng nhận diện và box phần nhìn thấy cao ít nhất 20 px, vẫn gán khi chỉ thấy một phần vật. Các mảnh nhìn thấy chắc chắn thuộc một vật được gom vào một box. Vật che phía trước có box riêng nếu thuộc phạm vi.

Ví dụ: người sau bàn/xe ở `img_001` có che khuất; xe sát mép ở `img_015` có cắt mép. Một vật có thể đồng thời có `occluded_level>0` và `truncated=yes`. Không kéo box qua mép, không đo chiều cao giả định của phần bị cắt để vượt ngưỡng 20 px.

### R-09 — Tượng, đồ chơi, hình in, phản chiếu và xe chủ

- **Loại theo quy tắc:** tượng người/ngựa `img_017`, tượng bán tải `img_027`, ma-nơ-canh `img_033`, xe mô hình trong tủ `img_038`, chân dung quảng cáo `img_018`. Không box đối tượng, không vùng ignore chỉ để che các phản ví dụ này.
- Hình phản chiếu trong gương, kính hoặc mặt đường không tạo đối tượng mới, dù nhận ra xe/người. Vật thật nhìn xuyên kính vẫn được gán nếu đủ rõ.
- Xe chủ (ego) là xe mang camera, không nằm trong mục tiêu phát hiện của bản này; bỏ phần gương/cửa/thân ego khi có bằng chứng trực tiếp. `img_020`: không gán xe trong gương và các phần ego. Không loại một xe chỉ vì nó to hoặc ở sát máy ảnh; `img_012` vẫn xét truck nhìn thấy.
- Xe thật đang trưng bày vẫn được gán (`img_024`, xe đen phía dưới). Vật thu nhỏ/trang trí khác với xe thật trưng bày. Nếu chưa đủ bằng chứng phân biệt tượng, người mặc trang phục và mô hình/xe thật (`img_011`, mascot `img_019`), ghi REVIEW theo R-13 thay vì kết luận từ tên ảnh.

### R-10 — Vật quá nhỏ: ngưỡng cố định 20 px

Đo `h_visible = y_max − y_min` của box ôm phần nhìn thấy trên ảnh gốc, trước khi quyết định gán.

- `h_visible ≥ 20 px`: gán lớp đối tượng nếu đạt điều kiện nhận diện. Đúng 20 px được gán.
- `0 < h_visible < 20 px`: không vẽ box lớp đối tượng; đặt `ignore_region` sát vùng nhỏ đó hoặc cụm vật nhỏ liền kề, ghi lý do `small<20px`.
- Ngưỡng này áp dụng riêng cho người và xe; không cộng chiều cao của hai vật. Không tăng box lên 20 px để lách kiểm tra.
- Ảnh `img_021`, `img_025`, `img_039` có ứng viên xa cần đo. Bảng ca khó **không khẳng định** vật cụ thể dưới 20 px khi chưa đo box; nhận xét “ở xa” không thay cho phép đo.

### R-11 — Đám đông và vùng ignore

Ưu tiên tách từng người/xe đủ rõ và đủ lớn. Chỉ dùng `ignore_region` cho cụm chồng lấp mà không thể phân biệt số vật/ranh giới sau khi zoom, hoặc cho vật dưới ngưỡng R-10. Không gom toàn đám đông `img_032` hay toàn bãi xe `img_030` vào một box ignore vì tốn công.

Vùng ignore phải gọn, có lý do `small<20px` hoặc `inseparable`, không bao vật hợp lệ đứng cạnh. Nếu không thể đặt vùng mà không che vật hợp lệ, ghi vấn đề cấp ảnh để review, không tùy tiện mở rộng vùng. Trong mã QC, một box có từ **50% diện tích** nằm trong một vùng ignore được coi là được vùng đó phủ; cần rà kỹ để tránh bỏ qua nhầm đối tượng. Bất đồng ignore vẫn phải xử lý, không dùng ignore để nâng IAA.

### R-12 — Nhòe, thiếu sáng và chi tiết không chắc

Vẫn gán khi lớp/biên đủ rõ. Khi thấy vật nhưng biên bị nhòe, dùng đường bao thân chính nhìn thấy, không ôm toàn vệt chuyển động; tick `can_review` nếu còn nhiều biên hợp lý. Không đoán số người từ các vệt sáng hoặc coi mọi chỗ tối là vùng ignore.

Nếu không nhận ra lớp hoặc không đặt được biên có căn cứ, ghi vấn đề cấp ảnh. Reviewer mới chốt gán, loại hay đặt vùng ignore; không dùng `occluded_level=3` như mã chất lượng ảnh kém.

### R-13 — Phân biệt bỏ qua chắc chắn và cần review

| Tình trạng | Hành động khi gán ở pha sau | Ví dụ |
|---|---|---|
| **GÁN**: thuộc lớp, vật thật, đủ kích thước và biên | Box đúng lớp, đủ thuộc tính; không tick review nếu rõ. | Scooter giữa `img_016`. |
| **LOẠI**: xác định rõ ngoài phạm vi | Không box đối tượng, không ignore; giữ lý do/R-ID trong bảng ca. | Tượng `img_027`; xe đạp `img_029`; bus `img_009`. |
| **IGNORE**: đạt điều kiện R-10/R-11 | Chỉ vùng ignore gọn, có lý do. | Cụm vật nhỏ đã đo ở nền xa. |
| **REVIEW có box tạm hợp lý** | Gán lớp/biên dự kiến, `can_review=true`, ghi điều cần chốt; chưa coi là đã duyệt. | Bán tải gắn nắp cao `img_037`: đề xuất `truck`. |
| **REVIEW chưa có box/lớp đáng tin** | Không bịa box hoặc lớp; ghi `image`, vùng mô tả, câu hỏi, quyết định dự kiến và R-ID. | Vật dạng nhân vật Predator ở `img_011`: tượng hay người hóa trang? |

Ở Pha 1, bảng bên dưới chính là sổ ca cần chốt; chưa có box nên chưa tick thuộc tính trên CVAT. Khi bước sang gán thật, ca chưa có box được chuyển thành vấn đề cấp ảnh theo định dạng của lab; không đánh dấu đã xử lý khi reviewer chưa xác nhận.

## 4. Ca khó và nhật ký xem đủ 40 ảnh

Mỗi dòng là **quyết định ban đầu cho vùng nêu rõ**, không phải danh sách đầy đủ đối tượng hoặc kết quả QC của cả ảnh. `REVIEW` tách khỏi `LOẠI`; các điều kiện đo `<20 px` phải thực hiện khi đặt box.

| Ảnh | Tình huống / vùng quan sát | Rule ID | Quyết định ban đầu | Lý do |
|---|---|---|---|---|
| [img_001](../../../data/images/img_001.jpg) | Xe tải nhỏ tiền cảnh, người đứng cạnh thùng và người sau bàn. | R-02, R-05, R-08 | **GÁN:** xe là `truck`; mỗi người thấy rõ có box riêng; cắt mép và che đánh giá riêng. | Thùng hàng riêng; bàn/xe che người không làm mất đối tượng, không đoán chân bị khuất. |
| [img_002](../../../data/images/img_002.jpg) | Xe chở ô tô hai tầng và người đứng phía phải. | R-07, R-08 | **GÁN:** cụm xe dự kiến 1 truck + 8 car; người xét riêng. | Xe được chở vẫn là xe thật; khung sàn không thuộc box từng car. |
| [img_003](../../../data/images/img_003.jpg) | Người gánh hàng giữa ảnh, xe đạp tiền cảnh và xe máy phía sau. | R-03, R-05, R-06 | **GÁN:** người và xe máy riêng; **LOẠI:** xe đạp, giỏ hàng. | Không mở box người ra tận đầu quang gánh; không gọi xe đạp là motorcycle. |
| [img_004](../../../data/images/img_004.jpg) | Hai tuk-tuk và người trong cabin. | R-03, R-08 | **LOẠI:** tuk-tuk khỏi bộ lớp hiện tại; **GÁN:** người nhìn xuyên kính/nhìn trực tiếp đủ rõ. | Cabin ba bánh không nằm trong motorcycle; phương tiện ngoài lớp không kéo theo loại người. |
| [img_005](../../../data/images/img_005.jpg) | Người đi xe máy che nhau, xe bạc thân van phía sau. | R-02, R-04, R-06 | **GÁN:** van phía sau; tách từng person và motorcycle đủ nhận diện; **REVIEW** nếu không gắn được phần tay/thân về đúng người. | Mũ/khẩu trang không phải lý do loại người; không gộp người–xe hoặc đoán số người từ mũ. |
| [img_006](../../../data/images/img_006.jpg) | Người trên xe máy bên trái, người đạp xe bên phải. | R-03, R-06 | **GÁN:** từng người; từng xe máy; **LOẠI:** xe đạp. | Người trên phương tiện ngoài taxonomy vẫn thuộc person. |
| [img_007](../../../data/images/img_007.jpg) | Hai xe hàng trong kho tối, xe bên phải cắt mép. | R-02, R-08, R-12 | **GÁN:** truck theo thùng rời; xe phải `truncated=yes`. | Xe đỗ trong kho vẫn là xe; tối không tự đồng nghĩa che khuất. |
| [img_008](../../../data/images/img_008.jpg) | Hàng xe máy chồng lấp và người áo số 4 tiền cảnh. | R-06, R-08, R-11 | **GÁN:** từng xe/người phân biệt được; **REVIEW:** vùng phía sâu không tách rõ. | Không gộp cả bãi thành ignore; xe dừng vẫn được xét. |
| [img_009](../../../data/images/img_009.jpg) | Xe buýt giữa ảnh, xe máy/người xung quanh. | R-03, R-08 | **LOẠI:** bus; **GÁN:** các đối tượng thuộc lớp, kể cả người trong bus nếu đủ rõ và lớn. | Không ép bus vào van; xét người độc lập. |
| [img_010](../../../data/images/img_010.jpg) | Người trên scooter nâu phải ảnh, nhóm bên xe trái, sạp hàng. | R-05, R-06 | **GÁN:** cụm phải 1 person + 1 motorcycle; từng người cụm trái riêng; **LOẠI:** hàng/xe đạp sạp. | Đây là phép thử R-06 trên ảnh khác; đồ bán không phải phần xe/người. |
| [img_011](../../../data/images/img_011.jpg) | Nhân vật Predator lớn và các vật dạng mô tô trang trí phía trước. | R-09, R-13 | **REVIEW cấp ảnh:** không tự khẳng định nhân vật là người hay tượng; cần xác nhận mô tô là xe thật hay tác phẩm. **GÁN:** người thật/xe thật rõ ở nền. | Hình dáng và tên ảnh không chứng minh người hóa trang hoặc xe vận hành; không loại cả ảnh. |
| [img_012](../../../data/images/img_012.jpg) | Bán tải trắng chở thùng/túi, SUV trái ảnh. | R-02, R-05, R-09 | **GÁN:** truck tiền cảnh, car bên trái; hàng không kéo rộng box truck. | Thùng hàng riêng là dấu hiệu truck; ở sát camera chưa chứng minh là ego. |
| [img_013](../../../data/images/img_013.jpg) | Người trên vỉa hè, mặt đường mưa có phản chiếu. | R-09, R-12 | **GÁN:** người/xe thật; **LOẠI:** bóng phản chiếu dưới đường. | Không đếm hai lần người/xe và không dùng vùng ignore cho phản chiếu rõ. |
| [img_014](../../../data/images/img_014.jpg) | Hai người đi bộ rõ nét; xe/người chạy nhòe phía sau. | R-06, R-12, R-13 | **GÁN:** hai person rõ; **REVIEW:** biên/người–xe trong từng vùng nhòe. | Không gán mức che cao chỉ vì nhòe, không vẽ hộp lớn theo vệt chuyển động. |
| [img_015](../../../data/images/img_015.jpg) | Đầu xe đỏ trái và đuôi xe bạc phải bị cắt; vệt xe giữa nền. | R-08, R-09, R-12 | **GÁN:** hai car sát mép với `truncated=yes`; **REVIEW:** xe nhòe nền; **LOẠI:** phản chiếu mặt đường. | Phần xe nhìn thấy đủ để nhận diện dù không thấy toàn xe. |
| [img_016](../../../data/images/img_016.jpg) | Scooter giữa, tuk-tuk hai bên, van và bán tải ở nền. | R-02, R-03, R-06 | **GÁN:** cụm giữa 1 person + 1 motorcycle; van/truck theo cấu trúc; **LOẠI:** tuk-tuk, xét người trên chúng riêng. | Không gộp người–xe; cùng một ảnh vẫn áp dụng nhiều lớp và phạm vi. |
| [img_017](../../../data/images/img_017.jpg) | Tượng người cưỡi ngựa đội cọc tiêu. | R-03, R-09 | **LOẠI:** tượng; không gán person, không ignore. | Vật mô phỏng người không phải người thật; ngựa không thuộc bộ lớp. |
| [img_018](../../../data/images/img_018.jpg) | Chân dung billboard; xe thật sau hàng rào. | R-08, R-09 | **LOẠI:** chân dung; **GÁN:** các xe thật đủ rõ, đánh giá che do rào. | Hình in không là person; hàng rào không khiến toàn bãi xe bị loại. |
| [img_019](../../../data/images/img_019.jpg) | Xe vàng diễu hành, người lái, mascot và khán giả. | R-05, R-09, R-11, R-13 | **REVIEW có quyết định tạm:** xe vàng đề xuất `car` vì cấu trúc xe đủ cỡ có bánh/khoang lái; mascot cần phân biệt mô hình/người mặc đồ. **GÁN:** người lái và khán giả đủ rõ. | Màu/trang trí không đủ để gọi xe đồ chơi; mascot chưa chắc không được tự gán person. |
| [img_020](../../../data/images/img_020.jpg) | Gương chiếu hậu, xe phản chiếu và phần cửa ego. | R-09 | **LOẠI:** xe trong gương và các phần ego; dự kiến ảnh không có box đối tượng. | Chỉ phản chiếu/xe chủ, không phải xe ngoài nhìn trực tiếp. |
| [img_021](../../../data/images/img_021.jpg) | Dòng xe máy dày, người/xe thu nhỏ về nền. | R-06, R-10, R-11 | **GÁN:** từng người/xe tách được; **IGNORE có điều kiện:** cụm đã đo dưới 20 px hoặc không thể tách. | Không có ngưỡng “quá đông”; chỉ bỏ vùng đạt điều kiện cụ thể. |
| [img_022](../../../data/images/img_022.jpg) | Mô tô với sidecar giữa ảnh đen trắng. | R-07, R-13 | **GÁN:** 1 motorcycle cho cụm gắn liền; **REVIEW:** phần nghi có người sau kính sidecar nếu không phân biệt được. | Sidecar là bộ phận của mô tô; không suy người từ ghế hoặc ảnh mờ. |
| [img_023](../../../data/images/img_023.jpg) | Người đạp xích lô tiền cảnh, car trái và xe máy nền. | R-03, R-06 | **GÁN:** người, car, motorcycle đủ điều kiện; **LOẠI:** xích lô. | Số bánh không đủ để gọi xích lô là motorcycle. |
| [img_024](../../../data/images/img_024.jpg) | Xe đen trưng bày, vật dạng ô tô vàng nhỏ trên nóc. | R-07, R-09 | **GÁN:** car đen và xe thật xung quanh; **LOẠI theo đề xuất:** mô hình vàng trên nóc. Nếu người duyệt thấy bằng chứng xe thật, chuyển REVIEW. | Tỷ lệ nhỏ so với nóc và cấu trúc mô hình khác ô tô thật chở trên `img_002`; ở bảo tàng không tự loại xe thật. |
| [img_025](../../../data/images/img_025.jpg) | Người đi bộ gần/xa, xe máy có thùng bên trên đường phải. | R-07, R-10, R-11 | **GÁN:** người/xe nhận ra; cụm mô tô–thùng gắn liền một motorcycle; **IGNORE có điều kiện:** người xa sau đo `<20 px`. | Không dùng chiều cao màn hình hay độ xa làm số đo pixel. |
| [img_026](../../../data/images/img_026.jpg) | Hai người đi một xe máy đỏ, bánh xe cắt mép dưới. | R-06, R-08 | **GÁN:** 2 person + 1 motorcycle; xe `truncated=yes`. | Giữ đúng quy tắc đã thử ở `img_016`, không có một box rider chung. |
| [img_027](../../../data/images/img_027.jpg) | Tượng bán tải trên bệ trong nghĩa trang. | R-09 | **LOẠI:** không truck/car, không ignore. | Bệ và chất liệu/chi tiết cho thấy vật tưởng niệm; hình dạng giống xe không đủ. |
| [img_028](../../../data/images/img_028.jpg) | Bán tải thật chở tượng ngựa, SUV đỏ nền. | R-02, R-05, R-07 | **GÁN:** truck và car nền; **LOẠI:** tượng ngựa, không kéo box truck lên đỉnh đầu ngựa. | Phân biệt phương tiện thật với hàng/trang trí; đối chứng với tượng xe `img_027`. |
| [img_029](../../../data/images/img_029.jpg) | Người/xe máy và người/xe đạp cùng thanh dài. | R-03, R-05, R-06 | **GÁN:** mỗi người và motorcycle; **LOẠI:** bicycle/thanh dài. | Hàng dài không mở rộng box xe/người; không ép mọi xe hai bánh vào motorcycle. |
| [img_030](../../../data/images/img_030.jpg) | Bãi xe dày, người áo xanh phải ảnh và poster nền. | R-09, R-11, R-13 | **GÁN:** xe tách được, người thật; **LOẠI:** người in trên poster; **REVIEW:** cụm xe không xác định ranh giới. | Đỗ xe không là lý do bỏ nhãn; không dùng một ignore cho toàn bãi. |
| [img_031](../../../data/images/img_031.jpg) | Xe/người ban đêm, bus nền phải, phản sáng mặt đường. | R-03, R-09, R-12 | **GÁN:** car/motorcycle/person rõ; **LOẠI:** bus, hình quảng cáo và phản chiếu. | Đêm/mưa không đổi lớp; vùng thực sự không rõ mới cần REVIEW. |
| [img_032](../../../data/images/img_032.jpg) | Hàng người chờ trạm, kính nhà chờ, ô tô mép trái. | R-08, R-09, R-11 | **GÁN:** từng person tách được; xe trái `truncated=yes`; nhìn xuyên kính vẫn xét vật thật. | Nhiều người không đồng nghĩa không tách được; không gộp toàn hàng người. |
| [img_033](../../../data/images/img_033.jpg) | Ma-nơ-canh, hình người in và phản chiếu trên tủ kính. | R-09 | **LOẠI:** các hình người giả/phản chiếu; dự kiến không có person trực tiếp đủ căn cứ. | Không coi hình dáng người sau kính mặc nhiên là người thật. |
| [img_034](../../../data/images/img_034.jpg) | Người đi bộ giữa, xe và người bên phải bị cắt. | R-06, R-08, R-10 | **GÁN:** person trung tâm; xe/người tách riêng; xét cắt mép từng box và đo đối tượng xa. | Người qua đường và rider đều cùng lớp person; cờ cắt không truyền từ xe sang người. |
| [img_035](../../../data/images/img_035.jpg) | Auto-rickshaw chất hàng cao, motorcycle gần phải. | R-03, R-05, R-06 | **LOẠI:** rickshaw và kiện hàng; **GÁN:** motorcycle + người gần phải, các người/xe thuộc lớp khác đủ rõ. | Chở nhiều hàng không biến tuk-tuk thành truck; người vẫn xét riêng. |
| [img_036](../../../data/images/img_036.jpg) | Xe tải xanh nhỏ, thùng phủ bạt sau cabin. | R-02, R-05 | **GÁN:** truck; box gồm bạt/khung thùng gắn xe. | Dựa thùng hàng riêng, không dựa kích thước xe để gọi van. |
| [img_037](../../../data/images/img_037.jpg) | Bán tải có nắp thùng cao; bus phía sau. | R-02, R-03, R-13 | **REVIEW có quyết định tạm:** truck, `can_review=true` khi gán; **LOẠI:** bus; người thật xét riêng. | Cabin và thùng vẫn tách; nắp cao làm dễ nhầm van. Cần xác nhận tiêu chí, không dựa chữ Master 75. |
| [img_038](../../../data/images/img_038.jpg) | Tủ đồ chơi, búp bê, ô tô mô hình. | R-09 | **LOẠI:** không car/person, không ignore; dự kiến không có box đối tượng. | Đồ chơi vẫn bị loại dù hình trên ảnh cao trên 20 px. |
| [img_039](../../../data/images/img_039.jpg) | Xe đạp giữa đường, người đi bộ, scooter gần/xa. | R-03, R-06, R-10 | **GÁN:** person/motorcycle rõ; **LOẠI:** bicycle; vật xa đo riêng trước quyết định ignore. | Giữ cùng quy tắc như `img_006` và `img_029`. |
| [img_040](../../../data/images/img_040.jpg) | Người đội mũ tiền cảnh, xe máy chỉ còn một phần; taxi, SUV, tuk-tuk và bus phía sau. | R-03, R-06, R-08, R-13 | **GÁN:** person và car rõ; **LOẠI:** tuk-tuk/bus. **REVIEW:** mảnh xe máy tiền cảnh nếu chưa xác định được biên của cùng một xe. | Không gộp người và xe, không suy toàn bộ motorcycle từ một gương; ảnh người in trên taxi không phải person. |

Các ca REVIEW ưu tiên: `img_011` (tượng/người và xe trang trí), `img_014`–`img_015` (biên nhòe), `img_019` (xe diễu hành/mascot), `img_022` (chi tiết phía sidecar), `img_037` (truck/van), `img_040` (mảnh xe tiền cảnh). Với `img_024`, quyết định ban đầu là loại mô hình, có tiêu chí chuyển review nếu bằng chứng trái ngược; không ghi như quyết định đã được người khác xác nhận.

## 5. QC và phiên bản

### 5.1. Ngưỡng giữ nguyên từ lab

Không chỉnh mục `rules` của taxonomy để làm số đo đẹp hơn. Các số dưới đây là **tiêu chuẩn áp dụng về sau**, không phải kết quả đã đạt ở Pha 1.

| Trường / đại lượng | Giá trị | Cách hiểu |
|---|---|---|
| `min_box_height_px` | 20 | Chiều cao box phần nhìn thấy; dưới 20 px xử lý R-10. |
| `match_iou` | 0.5 | Ghép cặp box 1–1 không phụ thuộc lớp, trước khi so lớp/biên. |
| `large_px` | 50 | Khi so hai box, script dùng chiều cao trung bình của cặp để chọn ngưỡng lớn/nhỏ. |
| `iou_thr_large` | 0.85 | Cặp box có chiều cao trung bình ≥50 px. |
| `iou_thr_small` | 0.70 | Cặp box có chiều cao trung bình <50 px. |
| `iaa_iou` | 0.85 | IoU trung bình của các cặp ghép được phải đạt ít nhất 0.85. |
| `iaa_class` | 0.95 | Tỉ lệ cùng lớp trên các cặp ghép được phải đạt ít nhất 95%. |
| `error_rate_img_max` | 0.02 | Tối đa 2% ảnh có ít nhất một lỗi trong tập đã duyệt; Release của lab cần duyệt đủ lô. |

IAA chỉ tính trên cặp ghép được, không chứng minh không bỏ sót và không chứng minh hai người cùng đúng. Không có cặp → ghi N/A, không ghi 100%. Phải xem cả box chưa ghép và bất đồng vùng ignore. Báo cáo luôn có tử số/mẫu số.

Với lô 40 ảnh, `1/40 = 2,5%` đã vượt 2%. Duyệt lần lượt điều kiện **Hold → Rework → Release**: thiếu bằng chứng/chưa chốt đặc tả thì Hold; vượt lỗi hoặc lỗi lặp thì Rework; chỉ Release khi đủ bằng chứng và reviewer xác nhận. Pha 1 chưa tạo nhãn/IAA/QC, **không có quyết định Release cho lô**.

### 5.2. Tự kiểm trước khi dùng guideline

- Tìm được ảnh cho từng ca; phân biệt đề xuất chắc chắn, loại theo phạm vi, ignore có điều kiện và REVIEW.
- Cụm `img_016` cho 2 box, cụm `img_026` cho 3 box theo cùng R-06; cụm `img_002` tách xe chở với xe được chở theo R-07.
- Phân biệt `q=30%`, `q=70%`, `h=20 px`; không dùng chiều cao tưởng tượng hoặc mức che để thay cho độ mờ.
- Mỗi đối tượng đủ điều kiện có một box; người, hàng và bóng không làm box xe phình ra.
- Ca chưa chốt có vùng mô tả và câu hỏi cụ thể. Reviewer/người dùng xác nhận trước khi đưa vào guideline khóa; việc review chưa được thực hiện trong bản này.
- Giữ ảnh không có đối tượng trong danh sách ảnh COCO ở pha gán, không xóa ảnh phản ví dụ khỏi bộ dữ liệu.

### 5.3. Lịch sử và cách sửa

| Phiên bản | Thay đổi | Lý do / ca dẫn chứng |
|---|---|---|
| v0 | Bản khởi đầu của lab; giữ nguyên tại [guideline_v0_starter.md](../../../guideline/guideline_v0_starter.md). | Thiếu ranh giới lớp, thuộc tính, cách bao box và ca khó. |
| v0.1 | Viết đủ 5 phần; R-01–R-13; khoảng che không hở; box phần nhìn thấy; người–xe tách; ngưỡng 20 px; phạm vi âm tính; bảng 40 ảnh và danh sách cần review. | `img_002`, `img_016`, `img_026`, `img_027`, `img_037` và toàn bộ bảng ca. |

Đây là bản cá nhân/đề xuất, không thay thế bản thống nhất v1.0 hay calibrated v1.1. Sau khi dùng làm bản đối chiếu, giữ nguyên v0.1; sửa quy tắc có thể đổi lớp/số box/biên/thuộc tính phải tạo phiên bản mới, ghi R-ID, ảnh dẫn chứng và phạm vi cần rà lại. Sửa cách chọn đối tượng là thay đổi ngữ nghĩa, không ghi thành sửa chính tả. Chưa tạo hoặc khóa các phiên bản pha sau trong nhiệm vụ này.
