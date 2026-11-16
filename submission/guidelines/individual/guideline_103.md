# Guideline gán nhãn — v0.1 (Bản cá nhân) — 103 - Lê Vân Phi

- **Phiên bản:** v0.1-personal (phát triển từ v0_starter)
- **Ngày:** 2026-11-16
- **Người viết:** 103 - Lê Vân Phi 

---

## 1. Mục tiêu

Tập dữ liệu này phục vụ bài toán **phát hiện đối tượng (2D Object Detection)** trên ảnh đường phố phục vụ hệ thống hỗ trợ lái xe nâng cao (ADAS) và camera giám sát giao thông đô thị (đặc thù đường phố châu Á nhiều xe máy, xe hỗn hợp, hàng quán vỉa hè).

- **Mục đích huấn luyện:** Giúp mô hình định vị và phân loại chính xác các phương tiện lưu thông và người tham gia giao thông.
- **Rủi ro an toàn nếu nhãn sai:**
  - **Bỏ sót (False Negative - FN):** Bỏ sót người đi bộ hoặc xe máy sang đường có thể dẫn tới quyết định không phanh kịp, gây va chạm tai nạn giao thông nguy hiểm.
  - **Gán thừa / Nhận diện sai vật tĩnh (False Positive - FP):** Gán nhầm biển quảng cáo, ma-nơ-canh, tượng đài hoặc hình phản chiếu qua gương thành người/xe thật sẽ gây hiện tượng "phanh ma" (phantom braking), làm xe dừng đột ngột nguy hiểm cho các phương tiện đi sau.
  - **Sai lệch Bounding Box (IoU thấp):** Box vẽ lỏng, lệch trọng tâm hoặc gộp nhiều đối tượng dẫn tới ước tính sai cự ly vật thể và dự báo quỹ đạo chuyển động sai.

---

## 2. Lớp và thuộc tính

### 2.1. Danh mục các lớp (Classes)

Mỗi đối tượng hợp lệ chỉ thuộc về **duy nhất một lớp** dưới đây:

| Lớp | Định nghĩa (một câu) | Ví dụ bao gồm | Phản ví dụ / Dễ nhầm với |
|---|---|---|---|
| `car` | Phương tiện giao thông cơ giới 4 bánh chở người từ 4 đến 9 chỗ ngồi. | Sedan, hatchback, SUV, CUV, xe taxi, coupe, xe bán tải (pickup truck cabin kép dùng đi phố). | Không bao gồm xe van chở hàng (`van`), xe tải thùng (`truck`), xe mô hình đồ chơi, xe đồ chơi trẻ em. |
| `truck` | Phương tiện cơ giới chuyên dụng vận chuyển hàng hóa, có thùng hàng hoặc đầu kéo chuyên biệt. | Xe tải thùng, xe tải đông lạnh, xe ben, xe đầu kéo container, xe bồn, xe cẩu, xe chuyên chở ô tô (car carrier). | Không bao gồm xe bán tải cỡ nhỏ gia đình (xếp `car`), xe van đóng kín (xếp `van`). |
| `van` | Phương tiện giao thông dạng khối hộp khép kín liền khối giữa khoang lái và khoang chở hàng/khách (10–16 chỗ). | Minivan, xe van chở hàng dạng hộp kín (ví dụ: Ford Transit, Hyundai Solati, Toyota HiAce, Suzuki Blind Van). | Không bao gồm SUV lớn (thuộc `car`), xe buýt cỡ lớn > 24 chỗ (cần đưa ra họp nhóm/ignore), xe tải có cabin rời thùng (thuộc `truck`). |
| `motorcycle` | Phương tiện cơ giới hai bánh hoặc ba bánh có gắn động cơ (bao gồm cả xe tay ga, xe gắn máy điện). | Xe máy số, xe tay ga (scooter), xe phân khối lớn, xe ba bánh có động cơ / xe lôi máy, xe máy có thuyền bên (sidecar). | Không bao gồm xe đạp thường, xe xích lô đạp chân, xe đồ chơi mini của trẻ em. |
| `person` | Con người bằng xương bằng thịt còn sống ở mọi lứa tuổi, tư thế. | Người đi bộ, người đứng/ngồi bên đường, người đang điều khiển hoặc ngồi sau trên xe máy, xe đạp, người lái ô tô thấy rõ qua cửa kính. | **Tuyệt đối không gán:** Ma-nơ-canh (mannequin), tượng đài đá/đồng, hình người in trên tranh ảnh/biển quảng cáo, hình phản chiếu qua gương kính. |
| `ignore_region` | Vùng ảnh chứa các đối tượng gây nhiễu, đám đông mờ nhòe ở quá xa, hoặc phương tiện dị biệt chưa thống nhất. | Đám đông người hoặc xe ở xa có chiều cao < 20px, khu vực phản chiếu mờ phức tạp, tranh ảnh quảng cáo lớn gây nhầm lẫn thị giác. | Không dùng để thay thế cho việc gán nhãn các đối tượng rõ ràng phía tiền cảnh. |

#### Quy tắc xử lý vật ngoài taxonomy:
- **Xe đạp (bicycle), xích lô đạp chân (cyclo) (img_006, img_009, img_023):** Không thuộc `motorcycle`. Gán người ngồi trên xích lô/xe đạp là `person`; bản thân chiếc xích lô/xe đạp nếu không nằm trong bảng nhãn thì **không gán box** (hoặc vẽ `ignore_region` nếu có nguy cơ gây bối rối cho mô hình).
- **Xe Tuk-tuk / Auto Rickshaw (img_004, img_035):** Là phương tiện 3 bánh phổ biến ở Thái Lan/Ấn Độ. Trong bản v0.1 cá nhân đề xuất:
  - Nếu có động cơ và kích thước tương đương xe con: Tạm thời gán `car` kèm tick `can_review`, hoặc gán `motorcycle` nếu nhấn mạnh tính chất xe máy 3 bánh. *Quyết định cá nhân: Tạm coi là `car` với cờ `can_review=true` để chờ họp nhóm thống nhất.*
- **Xe buýt công cộng (Bus) (img_032):** v0 thiếu lớp `bus`. Xe buýt công cộng lớn không phải xe van. Đề xuất: gán tạm `truck` hoặc không gán và tick `can_review`, đồng thời đề xuất thêm lớp `bus` ở Pha 2.

---

### 2.2. Thuộc tính (Attributes)

Mỗi bounding box (trừ `ignore_region`) bắt buộc phải được gắn đầy đủ 3 thuộc tính với tiêu chuẩn định lượng rõ ràng:

1. **`occluded_level` (Mức độ che khuất):** Đo bằng tỷ lệ diện tích bề mặt của vật thể bị che khuất bởi các vật thể khác phía trước.
   - `0` (Không bị che): 0% diện tích bị che khuất, toàn bộ đường viền ngoài của vật thể nhìn thấy rõ.
   - `1` (Che khuất nhẹ): Bị che khuất từ **1% đến 35%** diện tích (ví dụ: xe bị người đứng che một góc cản, hoặc xe khác che mất một phần bánh xe).
   - `2` (Che khuất trung bình đến nặng): Bị che khuất từ **36% đến 80%** diện tích (chỉ còn nhìn thấy từ 20% đến 64% thân xe/người, nhưng vẫn nhận dạng chắc chắn được loại đối tượng).
   - `3` (Bị che khuất gần như toàn bộ): Bị che khuất **> 80%** diện tích (chỉ còn lộ một phần nhỏ như chỏm đầu, một góc đèn/gương; chỉ nhận biết được nhờ ngữ cảnh xung quanh).

2. **`truncated` (Bị cắt bởi mép ảnh):**
   - `no`: Vật thể nằm trọn vẹn 100% bên trong khung hình ảnh, không chạm hoặc không bị cắt bởi bất kỳ cạnh nào của ảnh.
   - `yes`: Vật thể bị cắt ngang bởi ít nhất một trong 4 cạnh biên của bức ảnh (trên, dưới, trái, phải).

3. **`can_review` (Đánh dấu cần xem lại):**
   - `false` (mặc định): Tự tin với quyết định gán nhãn theo đúng guideline.
   - `true`: Gán nhãn cho các trường hợp dị biệt, khó phân lớp (ví dụ: xe tuk-tuk, xe chở xe, đồ vật mang vác kỳ lạ, ánh sáng quá tối) để đưa ra thảo luận tại buổi họp Calibration.

---

## 3. Quy tắc vẽ Bounding Box

1. **Nguyên tắc ôm sát (Tight Bounding Box):**
   - Box phải ôm sát điểm cực trị (trên cùng, dưới cùng, ngoài cùng bên trái, ngoài cùng bên phải) của phần vật thể nhìn thấy.
   - Không được để khoảng trống thừa (margin/padding) lớn hơn 2-3 pixel ra ngoài mặt đường hoặc phông nền.
   - Không được cắt lẹm vào thân xe hoặc người.

2. **Vật thể bị che khuất (Occlusion):**
   - Vẽ box bao quanh **phần thực tế nhìn thấy** của đối tượng, không cố tình tưởng tượng vẽ bao trùm cả vật cản phía trước nếu vật cản đó là một đối tượng độc lập.
   - Nếu một đối tượng bị chia cắt làm hai mảnh nhìn thấy (ví dụ xe ô tô bị một cột điện thẳng đứng cắt đôi): vẽ một bounding box lớn bao quanh cả hai phần của chiếc xe đó, và gán thuộc tính `occluded_level = 2` hoặc `3`.

3. **Vật thể bị cắt bởi mép ảnh (Truncation):**
   - Kéo cạnh của bounding box sát mép ảnh (tọa độ chạm viền ảnh), tuyệt đối không để box vượt ra ngoài giới hạn ảnh (x < 0, y < 0, x > W, y > H). Bắt buộc bật `truncated = yes`.

4. **Tách biệt Người (`person`) và Phương tiện (`motorcycle`, `car`):**
   - **Người điều khiển và hành khách trên xe máy / xe đạp (img_005, img_026):**
     - Vẽ 01 box `motorcycle` bao toàn bộ chiếc xe máy (từ bánh xe, ghi đông, thân xe đến đuôi xe; không tính phần người nhô lên trên yên xe).
     - Vẽ các box `person` độc lập cho từng người (người lái, người ngồi sau). Box `person` bao từ đỉnh đầu/mũ bảo hiểm xuống đến phần cơ thể nhìn thấy (chân/mông tiếp xúc với xe).
     - Việc hai box `person` và `motorcycle` chồng lấn (overlap) nhau là hoàn toàn bình thường và bắt buộc.
   - **Người bên trong ô tô:** Nếu ngồi trong ô tô kính trong suốt hoặc cửa mở nhìn rõ toàn bộ thân trên thì không cần vẽ `person` riêng lẻ trừ khi người đó đang nhoài hẳn nửa thân người ra ngoài xe.

5. **Xe chở hàng cồng kềnh hoặc chở phương tiện khác:**
   - **Xe chở xe (Car carrier) (img_002):**
     - Vẽ 01 box `truck` bao toàn bộ chiếc xe tải đầu kéo và giá đỡ rơ-moóc chuyên dụng.
     - Với mỗi chiếc xe ô tô con nằm trên giá chở: Nếu nhìn thấy rõ hình hài xe con, vẽ riêng từng box `car` cho từng chiếc xe đó (thuộc tính `occluded_level = 1` hoặc `2`).
   - **Xe chở hàng vượt khổ (img_012, img_029):**
     - Hàng hóa gắn chặt trên giá/thùng xe: Bounding box của phương tiện bao trùm toàn bộ thân xe và khối hàng hóa gắn liền trên xe (vì đối với hệ thống an toàn, khối hàng là một phần của thể tích va chạm).

6. **Vật thể KHÔNG PHẢI VẬT THỰC (Non-real objects):**
   - **Tượng đài / Điêu khắc (Statues) (img_017, img_027):** KHÔNG gán nhãn `person` hay `truck`/`car` cho tượng đá, tượng đồng.
   - **Hình in / Tranh ảnh quảng cáo (Billboards, Posters) (img_018):** KHÔNG gán nhãn `person` cho ảnh in người trên biển quảng cáo tranh cử, poster dán tường.
   - **Hình phản chiếu qua gương / kính (Reflections) (img_020):** KHÔNG gán nhãn cho xe hoặc người chỉ là hình ảnh ảo phản chiếu qua gương chiếu hậu hoặc bề mặt kính bóng.
   - **Ma-nơ-canh (Mannequins) (img_033):** KHÔNG gán `person` cho ma-nơ-canh trưng bày quần áo.
   - **Mô hình đồ chơi thu nhỏ (Toys / Model cars) (img_038):** KHÔNG gán `car` cho ô tô đồ chơi trong tủ kính.

7. **Kích thước tối thiểu và Đám đông (Small objects & Crowds):**
   - Đối tượng có chiều cao (height) **< 20 pixel**: Không gán box đơn lẻ.
   - Nếu có cụm người/xe dày đặc ở xa không thể tách rời: Vẽ một box `ignore_region` bao quanh cụm đó để tránh mô hình bị phạt lỗi false negative.

---

## 4. Ca khó và Quyết định ban đầu (Case Studies từ 40 ảnh)

Dưới đây là các ca dị biệt điển hình được trích xuất trực tiếp từ 40 bức ảnh trong tập dữ liệu `data/images/`:

| STT | Ảnh | Tình huống thực tế | Quyết định cá nhân | Lý do căn cứ theo mục tiêu mô hình |
|---|---|---|---|---|
| 1 | `img_001.jpg` | Xe tải nhỏ Suzuki Carry (dạng pickup/mini truck) đỗ trước quán ăn; có cô gái đứng che khuất một phần thùng xe. | Gán xe là `truck` (hoặc thảo luận chuyển `car`); người phụ nữ gán `person` (`occluded_level=0`). Box xe gán `occluded_level=1`. | Xe có thùng hàng hở phía sau đặc trưng của xe tải nhẹ chở hàng. Người đứng trước che khuất ~15% thân xe nên xe bị che nhẹ mức 1. |
| 2 | `img_002.jpg` | Xe tải đầu kéo Renault chở 6 chiếc ô tô con trên hai tầng giàn thép. | Vẽ 01 box `truck` lớn bao quanh toàn bộ xe kéo và giàn chở. Vẽ riêng 6 box `car` cho từng chiếc ô tô con trên giàn. | Xe tải đầu kéo là `truck`. Các xe con trên giàn là xe thật hoàn chỉnh, hệ thống thị giác cần nhận biết để tính toán vật cản. |
| 3 | `img_004.jpg` | Hai chiếc xe Tuk-tuk 3 bánh đỗ trên vỉa hè ban đêm. Taxonomy v0 không có lớp tuk-tuk. | Tạm gán `car`, tick `can_review=true`. Đề xuất tại Pha 2 xem xét thêm lớp hoặc quy ước chung. | Tuk-tuk chở khách 3 bánh có động cơ, kết cấu có cabin và mái che gần với ô tô/xe lôi hơn xe máy cá nhân. |
| 4 | `img_005.jpg` | Đoàn người đi xe máy rất đông trên đường phố Sài Gòn, nhiều người ngồi sát nhau. | Tách riêng: mỗi xe máy là 01 box `motorcycle`, mỗi người (cả lái và ngồi sau) là 01 box `person`. | Đảm bảo mô hình học được mối liên kết người - xe máy nhưng vẫn phát hiện được đúng số lượng người tham gia giao thông. |
| 5 | `img_006.jpg` | Đoàn xích lô đạp chân chở khách du lịch tại Huế. | Gán khách và người đạp xích lô là `person`. KHÔNG gán xích lô là `motorcycle` (có thể vẽ `ignore_region` bao quanh xe xích lô nếu gây nhiễu). | Xích lô chạy bằng sức người (xe đạp 3 bánh), không có động cơ cơ giới nên không thuộc `motorcycle`. |
| 6 | `img_011.jpg` | Người mặc trang phục hóa trang quái vật Predator đứng cạnh dàn xe mô tô độ kiểu kỳ dị. | Người hóa trang gán `person` (`can_review=true`). Dàn xe mô tô độ gán `motorcycle`. | Dù mặc trang phục cosplay/quái vật nhưng bên trong là người thật đang di chuyển; xe mô tô dù độ hình thù lạ vẫn là phương tiện 2 bánh tham gia giao thông. |
| 7 | `img_012.jpg` | Xe bán tải nhỏ chở thùng xốp, bao tải hàng hóa chất cao đầy thùng xe. | Box `truck` vẽ bao trùm cả cabin xe và phần hàng hóa trên thùng xe. Gán `occluded_level=0`. | Khối hàng gắn liền trên thùng xe tạo thành một thể tích vật lý thống nhất cần tránh va chạm trong thực tế lái xe. |
| 8 | `img_017.jpg` | Tượng đài công tước Wellington cưỡi ngựa bằng đồng/đá đội nón giao thông tại Glasgow. | **KHÔNG gán nhãn** (không gán `person` cho tượng người cưỡi ngựa). | Đây là công trình kiến trúc / tượng điêu khắc tĩnh, không phải con người thật. Gán nhãn sẽ khiến mô hình nhận diện nhầm tượng là người. |
| 9 | `img_018.jpg` | Biển quảng cáo tấm lớn (billboard) in hình chân dung ứng viên chính trị gia. | **KHÔNG gán nhãn** `person` cho hình ảnh in trên biển quảng cáo. | Tranh ảnh 2D trên biển hiệu không có nguy cơ va chạm giao thông; tránh lỗi phantom braking. |
| 10 | `img_020.jpg` | Cầu vồng và các phương tiện phản chiếu qua mặt gương chiếu hậu xe ô tô. | **KHÔNG gán nhãn** cho hình ảnh xe phản chiếu trong gương. | Hình ảnh ảo trong gương không phải đối tượng thực tế trên mặt đường phía trước của xe. |
| 11 | `img_022.jpg` | Xe máy cổ điển có gắn thùng phụ bên hông (Sidecar / xe thuyền). | Vẽ 01 box `motorcycle` bao cả xe máy và thùng phụ sidecar. Người ngồi lái và ngồi trong thùng gán các box `person` riêng. | Thùng phụ gắn cố định vào khung xe máy tạo thành một phương tiện 3 bánh thống nhất di chuyển cùng vận tốc. |
| 12 | `img_027.jpg` | Bia mộ/đài tưởng niệm hình xe bán tải bằng đá granite tại nghĩa trang Oak Hill. | **KHÔNG gán nhãn** `truck` hay `car` (hoặc vẽ `ignore_region`). | Tác phẩm điêu khắc bằng đá tĩnh tại nghĩa trang, không phải xe cơ giới thật. |
| 13 | `img_029.jpg` | Xe máy chở khối hàng cồng kềnh, cành cây/thùng hàng vượt quá kích thước xe. | Box `motorcycle` bao trùm cả xe và khối hàng hóa; người lái gán `person`. | Khối hàng cồng kềnh là nguồn nguy cơ va chạm trực tiếp với xe tự lái. |
| 14 | `img_032.jpg` | Trạm dừng xe buýt giờ cao điểm có xe buýt công cộng lớn (ETS Bus). | Tạm gán `truck`, tick `can_review=true`. Đề xuất tại Pha 2 thêm lớp `bus`. | v0 thiếu lớp xe buýt; xe buýt lớn gần với kích thước xe tải chở khách hơn là xe van nhỏ. Cần họp nhóm thống nhất. |
| 15 | `img_033.jpg` | Ma-nơ-canh mặc áo quần thời trang đứng trên phố đi bộ Buchanan Street. | **KHÔNG gán nhãn** `person`. | Ma-nơ-canh là vật vô tri giác trưng bày, không phải người sống tham gia giao thông. |
| 16 | `img_038.jpg` | Tủ kính cửa hàng bày bộ sưu tập đồ chơi, búp bê và các mô hình xe ô tô thu nhỏ. | **KHÔNG gán nhãn** `car` hay `person` (vẽ `ignore_region` bao quanh tủ kính nếu cần). | Đồ chơi mô hình tĩnh kích thước nhỏ trong tủ kính không phải phương tiện giao thông đường phố. |

---

## 5. QC và Quy trình cập nhật phiên bản

### 5.1. Ngưỡng kiểm soát chất lượng (QC Thresholds)
- **Độ chính xác vị trí box (IoU):**
  - Đối tượng lớn (chiều cao ≥ 50px): $IoU \ge 0.85$.
  - Đối tượng nhỏ (chiều cao 20px – 50px): $IoU \ge 0.70$.
- **Độ đồng thuận giữa các annotator độc lập (IAA - Inter-Annotator Agreement):**
  - $Mean\ IoU \ge 0.85$ trên toàn bộ các cặp box trùng khớp.
  - Tỷ lệ trùng khớp nhãn lớp (Class Agreement Rate) $\ge 95\%$.
- **Tỷ lệ lỗi tối đa sau QC (Error Rate):**
  - Tỷ lệ ảnh có lỗi $\le 2\%$ để được phép **Release**.

### 5.2. Nguyên tắc cập nhật phiên bản
- Mọi trường hợp mơ hồ gặp phải trong quá trình dán nhãn phải tick `can_review = true` và ghi chép lại.
- Các quyết định thống nhất trong cuộc họp nhóm sẽ được cập nhật vào phiên bản **v1.0 (Agreed)** và **v1.1 (Calibrated)**.

### 5.3. Lịch sử thay đổi (Changelog)

| Phiên bản | Ngày | Người thực hiện | Nội dung thay đổi chính | Căn cứ lý do |
|---|---|---|---|---|
| `v0` | 2026-11-16 | Ban giảng huấn | Bản sơ sài khởi đầu (chỉ liệt kê 5 lớp cơ bản, không có quy tắc định lượng). | Bản khung khởi tạo. |
| `v0.1` | 2026-11-16 | 103 - Lê Vân Phi | - Thêm định nghĩa chi tiết, ví dụ và phản ví dụ cho cả 6 lớp.<br>- Lượng hóa 4 mức `occluded_level` theo % diện tích.<br>- Quy định rõ cách vẽ box ôm sát, tách biệt người và xe máy.<br>- Bổ sung quy tắc loại trừ vật không thật (tượng đá, biển quảng cáo, ma-nơ-canh, đồ chơi, hình phản chiếu qua gương).<br>- Phân tích chi tiết 16 ca khó từ 40 ảnh thực tế (`img_001` đến `img_038`). | Khắc phục hoàn toàn các điểm mơ hồ của v0 để chuẩn bị họp thống nhất Pha 2. |
