# Guideline gán nhãn — v1 — 102

- **Phiên bản:** v1
- **Ngày:** 16/11/2026
- **Người viết / duyệt:** Người viết: Nhóm 102 / Người duyệt: Mentor hoặc Reviewer
- **Dữ liệu:** Ảnh đường phố giao thông
- **Công cụ:** CVAT
- **Phạm vi annotation:** Bounding Box, Polygon và Polyline
- **Định dạng ảnh:** `.jpg`, kích thước tham chiếu 1280 × 720 pixel
- **Nguyên tắc:** Chỉ gán nhãn theo taxonomy trong tài liệu; trường hợp không đủ bằng chứng phải tạo issue/comment để reviewer hoặc mentor xem xét.

---

## 1. Mục tiêu

Guideline này quy định cách gán nhãn các đối tượng giao thông, vùng mặt đường có thể di chuyển và vạch kẻ đường trên ảnh đường phố. Nhãn được sử dụng làm dữ liệu huấn luyện và đánh giá mô hình thị giác máy tính.

Mục tiêu của việc gán nhãn:

1. Xác định đầy đủ các đối tượng thuộc phạm vi bài toán.
2. Gán đúng class và đúng loại shape trong CVAT.
3. Đảm bảo hình học của Bounding Box, Polygon và Polyline bám sát đối tượng hoặc ranh giới quan sát được.
4. Gán đúng thuộc tính `occluded` và `truncated` khi cần.
5. Giữ tính nhất quán giữa các annotator và giữa các ảnh trong cùng batch.

Sai sót cần hạn chế:

- **False Negative:** Bỏ sót đối tượng hoặc vùng cần annotation.
- **False Positive:** Gán nhãn cho vật không tồn tại, không thuộc phạm vi hoặc chỉ là hình ảnh phản chiếu/bóng đổ.
- **Sai class:** Nhận diện nhầm loại đối tượng.
- **Sai geometry:** Box quá rộng/quá chặt; polygon không bám ranh giới; polyline đi lệch vạch hoặc nối qua vùng không có bằng chứng.
- **Sai attribute:** Bật hoặc bỏ sót `occluded` / `truncated`.

### Phạm vi dữ liệu

- Thư mục ảnh đầu vào: `images`.
- Không sử dụng annotation hoặc ground truth có sẵn trong quá trình thực hiện.
- Chỉ annotate các class được liệt kê trong Phần 2.
- Không tự tạo class mới, không đổi tên class và không gộp class theo cảm tính.

---

## 2. Lớp và thuộc tính

### 2.1. Taxonomy và loại shape

| Nhóm | CVAT shape | Class |
|---|---|---|
| Object instance | Rectangle / Bounding Box | `pedestrian`, `rider`, `car`, `truck`, `bus`, `train`, `motorcycle`, `bicycle`, `traffic light`, `traffic sign` |
| Drivable area | Polygon | `area/direct`, `area/alternative` |
| Lane marking | Polyline | `lane/crosswalk`, `lane/double white`, `lane/double yellow`, `lane/double other`, `lane/road curb`, `lane/single white`, `lane/single yellow`, `lane/single other` |

Mỗi object instance được gán một Bounding Box riêng. Drivable area được biểu diễn bằng Polygon khép kín. Lane marking được biểu diễn bằng Polyline không khép kín.

### 2.2. Định nghĩa class Object instance

| Class | Định nghĩa | Ví dụ | Phản ví dụ / dễ nhầm |
|---|---|---|---|
| `pedestrian` | Người xuất hiện trong cảnh giao thông, không phụ thuộc tư thế đứng, đi bộ, chạy hoặc ngồi. | Người đi bộ trên vỉa hè, người đứng chờ qua đường, người ngồi. | Không gán túi xách, ba lô, ô hoặc vật dụng cá nhân thành pedestrian. |
| `rider` | Người điều khiển hoặc ngồi trên phương tiện hai bánh. | Người lái xe máy, người ngồi trên xe đạp. | Không dùng `pedestrian` cho người đang điều khiển hoặc ngồi trên phương tiện hai bánh. |
| `car` | Ô tô con và xe van dùng để chở người. | Sedan, SUV, hatchback, passenger van. | Xe van dùng để chở hàng được xếp vào `truck`. |
| `truck` | Xe tải và xe van dùng để chở hàng. | Xe tải thùng, xe tải nhỏ, cargo van. | Không xếp passenger van vào `truck`. |
| `bus` | Các loại xe buýt. | Xe buýt công cộng, xe buýt cỡ nhỏ. | Không nhầm xe khách hoặc xe tải với bus nếu không đủ bằng chứng; trường hợp không chắc chắn cần review. |
| `train` | Tàu hỏa xuất hiện trong ảnh. | Đầu tàu hoặc toa tàu. | Không gán các phương tiện đường bộ thành train. |
| `motorcycle` | Phương tiện xe máy hoặc mô tô hai bánh. | Xe máy, mô tô. | Người lái xe là `rider`, không phải motorcycle. |
| `bicycle` | Phương tiện xe đạp. | Xe đạp thường, xe đạp thể thao. | Người đi xe đạp là `rider`, không phải bicycle. |
| `traffic light` | Thiết bị đèn tín hiệu giao thông. | Đèn đỏ, vàng, xanh tại nút giao. | Không gán đèn trang trí hoặc nguồn sáng không có chức năng tín hiệu giao thông. |
| `traffic sign` | Biển báo giao thông. | Biển cấm, biển chỉ dẫn, biển cảnh báo. | Không gán bảng quảng cáo hoặc biển hiệu cửa hàng thành traffic sign. |

**Quy tắc phân biệt người và phương tiện:**

- Người điều khiển hoặc ngồi trên phương tiện hai bánh: gán `rider` cho người và gán class phương tiện tương ứng (`motorcycle` hoặc `bicycle`) cho phương tiện nếu phương tiện quan sát được.
- Người đi bộ độc lập: gán `pedestrian`.
- Mỗi object là một instance riêng; không gộp người và phương tiện vào cùng một Bounding Box.
- Nếu không thể xác định người có đang điều khiển/ngồi trên phương tiện hay không, tạo issue `UNCERTAIN_CLASS`.

### 2.3. Định nghĩa Drivable area

| Class | Định nghĩa | Cách xác định | Không gán |
|---|---|---|---|
| `area/direct` | Vùng mặt đường mà xe đang di chuyển hoặc có thể tiếp tục di chuyển trực tiếp, không cần chuyển làn hay đổi hướng. | Phần mặt đường tiếp nối trực tiếp hướng di chuyển hiện tại. | Không mở rộng sang làn ngược chiều hoặc khu vực không thể đi trực tiếp. |
| `area/alternative` | Vùng mặt đường mà xe có thể lựa chọn di chuyển sang nhưng cần thay đổi làn trong trường hợp đường có nhiều làn cùng chiều. | Làn khác cùng chiều có thể chuyển sang. | Không gán đường ngược chiều thành alternative. |

### 2.4. Định nghĩa Lane marking

| Class | Định nghĩa | Quy tắc nhận diện |
|---|---|---|
| `lane/crosswalk` | Vạch kẻ đường dành cho người đi bộ sang đường. | Bám theo các vạch thuộc khu vực sang đường dành cho người đi bộ. |
| `lane/double white` | Vạch kẻ đường đôi màu trắng. | Áp dụng cho cả vạch nét đứt và nét liền. |
| `lane/double yellow` | Vạch kẻ đường đôi màu vàng. | Áp dụng cho cả vạch nét đứt và nét liền. |
| `lane/double other` | Vạch kẻ đường đôi có màu khác trắng hoặc vàng. | Chỉ sử dụng khi xác định được đây là vạch đôi và màu không thuộc hai nhóm trên. |
| `lane/road curb` | Đường biên bó vỉa, phân cách mặt đường với vỉa hè hoặc khu vực liền kề. | Đi theo đường ranh giới bó vỉa quan sát được. |
| `lane/single white` | Vạch kẻ đường đơn màu trắng. | Áp dụng cho cả vạch nét đứt và nét liền. |
| `lane/single yellow` | Vạch kẻ đường đơn màu vàng. | Áp dụng cho cả vạch nét đứt và nét liền. |
| `lane/single other` | Vạch kẻ đường đơn có màu khác trắng hoặc vàng. | Chỉ sử dụng khi xác định được đây là vạch đơn và màu không thuộc hai nhóm trên. |

Nếu không xác định chắc chắn được màu, số lượng vạch hoặc loại vạch, không tự suy đoán; tạo issue để review.

### 2.5. Thuộc tính

| Thuộc tính | Giá trị | Khi nào sử dụng | Ví dụ |
|---|---|---|---|
| `occluded` | `true` / `false` | `true` khi một phần object bị object khác che khuất nhưng object vẫn tồn tại trong scene. `false` khi không bị che khuất. | Xe phía sau xe tải bị che một phần: `true`. |
| `truncated` | `true` / `false` | `true` khi object bị cắt bởi biên ảnh, một phần nằm ngoài frame. `false` khi object nằm trọn trong frame. | Xe chỉ xuất hiện một phần ở mép trái ảnh: `true`. |

- Nếu object vừa bị che bởi object khác vừa bị cắt bởi biên ảnh, bật cả hai thuộc tính.
- Hai thuộc tính trên áp dụng cho Object instance có Bounding Box.
- Không tự thêm giá trị hoặc thuộc tính mới ngoài cấu hình của batch.

---

## 3. Quy tắc vẽ box

### 3.1. Bounding Box – Object instance

1. Mỗi object instance thuộc taxonomy phải có một Bounding Box riêng.
2. Vẽ box hình chữ nhật bao sát object theo bốn extreme points: trái, phải, trên, dưới.
3. Box phải bao phủ phần object nhìn thấy được; không lấy nền xung quanh quá nhiều và không vẽ quá chặt làm mất phần object.
4. Không vẽ box vượt ra ngoài biên ảnh.
5. Các box được phép chồng lấn khi object ở gần nhau hoặc che khuất lẫn nhau.
6. Không gộp nhiều object cùng class vào một box.
7. Không tạo box cho object không thuộc danh sách class.
8. Không annotate reflection, bóng đổ hoặc hình ảnh của object trên billboard/màn hình thành object thật, trừ khi batch có quy định riêng.

### 3.2. Object bị che khuất (Occluded)

- Object vẫn nằm trong scene nhưng một phần bị object khác che: vẫn annotate nếu có đủ bằng chứng xác định object và class.
- Box bao phần object có thể quan sát được, không tự vẽ phần bị che như thể nhìn thấy.
- Bật `occluded = true`.
- Nếu object bị che quá nhiều và không thể xác định đáng tin cậy vị trí hoặc class, tạo issue `UNCERTAIN_CLASS` hoặc `UNCERTAIN_BOUNDARY` tùy vấn đề.

### 3.3. Object bị cắt bởi biên ảnh (Truncated)

- Object bị cắt tại một hoặc nhiều cạnh ảnh nhưng phần còn lại vẫn nhận diện được: vẫn annotate.
- Box chỉ nằm trong phạm vi frame và ôm sát phần object nhìn thấy.
- Bật `truncated = true`.
- Nếu object vừa bị che vừa bị cắt, bật cả `occluded = true` và `truncated = true`.

### 3.4. Ngưỡng xử lý object bị che hoặc bị cắt

- Object bị che khuất hoặc bị cắt khỏi khung hình không quá **80% kích thước ước tính**: cần annotate.
- Nếu tỷ lệ bị che hoặc bị cắt lớn hơn 80%: vẫn khuyến khích annotate nếu còn đủ bằng chứng để xác định vị trí và class.
- Không tự ước lượng phần bị khuất để mở rộng box thành một hình chữ nhật hoàn chỉnh. Geometry phải phản ánh phần quan sát được trong ảnh.
- Nếu không đủ bằng chứng, tạo issue để reviewer/mentor quyết định.

### 3.5. Polygon – Drivable Area

1. Chỉ sử dụng Polygon cho hai class `area/direct` và `area/alternative`.
2. Polygon phải là đường bao khép kín, biểu diễn vùng mặt đường tương ứng.
3. Các điểm phải bám sát ranh giới và extreme points của vùng, không bao quá rộng hoặc quá hẹp.
4. Các đỉnh polygon không được tự giao cắt.
5. Đoạn ranh giới thẳng chỉ cần số điểm vừa đủ; đoạn cong hoặc phức tạp cần tăng mật độ điểm để bám sát biên.
6. Không đưa vào polygon các khu vực không thuộc drivable area.
7. Không suy đoán ranh giới bị che khuất hoặc không nhìn thấy rõ. Khi không đủ bằng chứng xác định boundary, tạo issue `UNCERTAIN_BOUNDARY`.

### 3.6. Polyline – Lane Marking

1. Chỉ sử dụng Polyline cho các class thuộc nhóm Lane marking trong Phần 2.4.
2. Polyline là đường nối các điểm liên tiếp và **không khép kín**.
3. Với vạch kẻ đường, đường polyline phải đi theo đúng tim vạch theo quy ước của bài.
4. Với `lane/road curb`, polyline phải bám theo đường biên bó vỉa quan sát được.
5. Không nối tắt qua đoạn không có vạch hoặc không có bằng chứng thị giác.
6. Kết thúc polyline tại điểm lane marking không còn đủ bằng chứng quan sát.
7. Giữ hướng vẽ nhất quán trong cùng dataset nếu batch có yêu cầu về direction.
8. Không nối hai vạch riêng biệt thành một polyline chỉ vì chúng cùng class.

### 3.7. Các trường hợp ngoài phạm vi hoặc khó quan sát

| Tình huống | Quyết định |
|---|---|
| Vật không thuộc taxonomy | Không annotate, không tự tạo class mới. |
| Reflection hoặc bóng đổ | Không annotate thành object thật. |
| Hình ảnh object trên billboard/màn hình | Không annotate thành object thật. |
| Object rất nhỏ nhưng vẫn đủ bằng chứng nhận diện | Annotate theo class phù hợp. |
| Object quá nhỏ, mờ hoặc thiếu sáng đến mức không thể xác định class | Tạo issue `LOW_VISIBILITY`. |
| Đám đông có nhiều người phân biệt được từng cá thể | Annotate từng object riêng nếu từng người thuộc scope và đủ bằng chứng. |
| Nhiều object che khuất nhau | Annotate từng object nhận diện được; box có thể chồng lấn; bật `occluded` cho object bị che. |
| Không xác định được ranh giới vùng đường hoặc vạch | Tạo issue `UNCERTAIN_BOUNDARY`. |
| Không rõ một object có thuộc phạm vi bài hay không | Tạo issue `UNCERTAIN_SCOPE`. |

Các trường hợp như xe chủ (ego vehicle), tượng, đồ chơi, hình in hoặc vật thể có hình dạng giống phương tiện nhưng không rõ có thuộc scope hay không chưa có quy tắc quyết định riêng trong tài liệu nguồn. Không tự đặt quy tắc mới; cần đưa reviewer/mentor xác nhận.

---

## 4. Ca khó

Khi gặp tình huống không chắc chắn, ưu tiên tạo issue/comment để review thay vì đoán. Có thể sử dụng majority voting / inter-annotator agreement giữa các thành viên trước khi gửi mentor, nhưng không tự chốt quy tắc mới nếu team chưa có kết luận được reviewer/mentor xác nhận.

| Ảnh | Tình huống | Rule ID | Quyết định | Lý do |
|---|---|---|---|---|
| `img_0xx` | Không phân biệt được car/van chở người hay truck/van chở hàng | `UNCERTAIN_CLASS` | Tạo issue, chờ reviewer/mentor xác nhận | Phân loại van phụ thuộc mục đích chở người hay chở hàng. |
| `img_0xx` | Object bị che nhiều, không rõ phạm vi box | `UNCERTAIN_BOUNDARY` | Annotate nếu còn đủ bằng chứng; nếu không, tạo issue | Không suy đoán phần không quan sát được. |
| `img_0xx` | Không chắc object có thuộc danh sách class hay không | `UNCERTAIN_SCOPE` | Không tự tạo class; tạo issue | Chỉ taxonomy được liệt kê mới thuộc phạm vi. |
| `img_0xx` | Không chắc object bị che hay bị cắt bởi biên ảnh | `ATTRIBUTE_CHECK` | Đề nghị reviewer kiểm tra attribute | `occluded` và `truncated` mô tả hai nguyên nhân khác nhau. |
| `img_0xx` | Object khó thấy do thiếu sáng, ngược sáng, mưa, sương mù, kích thước quá nhỏ hoặc motion blur | `LOW_VISIBILITY` | Tạo issue; annotate nếu vẫn đủ bằng chứng | Tránh gán class khi không có đủ bằng chứng thị giác. |
| `img_0xx` | Không xác định được vùng drivable area hoặc ranh giới lane marking | `UNCERTAIN_BOUNDARY` | Không tự nối hoặc mở rộng; tạo issue | Geometry phải dựa trên ranh giới quan sát được. |

### Quy trình xử lý issue

1. Ghi rõ tên ảnh/frame và vị trí object hoặc vùng đang gặp vấn đề.
2. Chọn đúng loại issue:
   - `UNCERTAIN_CLASS`: Không chắc class.
   - `UNCERTAIN_BOUNDARY`: Không rõ boundary/extent.
   - `UNCERTAIN_SCOPE`: Không rõ object có thuộc phạm vi.
   - `ATTRIBUTE_CHECK`: Không chắc `occluded` hoặc `truncated`.
   - `LOW_VISIBILITY`: Khó quan sát do điều kiện ảnh.
3. Ghi mô tả ngắn gọn về điểm chưa chắc chắn và phương án đang cân nhắc.
4. Gửi reviewer/mentor xem xét.
5. Sau khi có quyết định, áp dụng thống nhất cho các ảnh có tình huống tương tự trong batch.

Nếu CVAT batch có hỗ trợ comment/issue, ghi nhận trực tiếp tại task/frame. Nếu không, ghi vào nhật ký issue của nhóm và tham chiếu chính xác ảnh/frame.

---

## 5. QC và phiên bản

### 5.1. Checklist chất lượng trước khi Submit

| STT | Hạng mục | Tiêu chí kiểm tra |
|---|---|---|
| 1 | Taxonomy | Đúng class và đúng shape; không tự tạo, đổi tên hoặc gộp class. |
| 2 | Completeness | Không bỏ sót object/area/lane rõ ràng thuộc scope; không annotation trùng lặp. |
| 3 | Bounding Box | Box ôm sát object, hạn chế nền thừa, không vượt biên ảnh. |
| 4 | Attributes | `occluded` và `truncated` được gán đúng khi cần; trường hợp vừa che vừa cắt phải bật cả hai. |
| 5 | Polygon | Polygon bám đúng ranh giới drivable area, khép kín, không tự giao cắt. |
| 6 | Polyline | Polyline bám theo lane marking/road curb, không nối qua vùng không có evidence. |
| 7 | Consistency | Cùng một tình huống được xử lý nhất quán giữa các annotator và xuyên suốt batch. |
| 8 | Escalation | Case chưa chắc chắn đã có issue/comment và được raise lên reviewer/mentor. |
| 9 | Final check | Đã kiểm tra lại toàn bộ dữ liệu 02 lần trước khi save/submit. |

### 5.2. Tiêu chí đánh giá của Reviewer

| Hạng mục | Reviewer kiểm tra |
|---|---|
| Taxonomy | Đúng class, không có class ngoài scope. |
| Completeness | Không thiếu hoặc thừa object/area/lane rõ ràng. |
| Geometry Accuracy | Bounding Box ôm sát; Polygon và Polyline đúng hình học, bám ranh giới quan sát được. |
| Attributes | `occluded` và `truncated` đúng quy tắc. |
| Consistency | Các annotator áp dụng thống nhất guideline, taxonomy và quy tắc xử lý edge cases. |
| Escalation | Case chưa có rule được đưa reviewer/mentor chốt; annotator không tự invent rule. |

### 5.3. Nguyên tắc QC và xử lý bất đồng

- Reviewer kiểm tra tối thiểu các hạng mục trong checklist trước khi xác nhận dữ liệu.
- Khi phát hiện lỗi, ghi rõ ảnh/frame, class/shape liên quan, loại lỗi và hướng sửa.
- Khi các annotator bất đồng, đối chiếu định nghĩa class và quy tắc trong guideline; có thể dùng majority voting để tổng hợp ý kiến trước khi gửi mentor.
- Quyết định cuối cùng đối với case chưa có rule thuộc reviewer/mentor.
- Sau khi có kết luận, áp dụng thống nhất cho các trường hợp tương tự trong batch.
- Tài liệu nguồn không quy định tỷ lệ pass QC hoặc ngưỡng sai số hình học cụ thể; không tự đặt ngưỡng định lượng khi chưa được reviewer/mentor phê duyệt.

### 5.4. Lịch sử phiên bản

| Phiên bản | Ngày | Thay đổi | Lý do (ca nào) |
|---|---|---|---|
| v0 | Theo guideline khởi đầu | Quy định ban đầu về Bounding Box, các class `car`, `truck`, `van`, `motorcycle`, `person`, `ignore_region` và thuộc tính liên quan. | Bản khởi đầu do giảng viên cung cấp. |
| v1 | 16/11/2026 | Chuẩn hóa theo template; mô tả taxonomy Object instance, Drivable area, Lane marking; bổ sung quy tắc Bounding Box, Polygon, Polyline, thuộc tính `occluded` / `truncated`, xử lý issue và checklist QC. | Hoàn thiện guideline cho batch ảnh đường phố và thống nhất cách annotate theo tài liệu BBox, Polygon & Polyline. |

---

**Lưu ý:** Guideline này sử dụng taxonomy và quy tắc trong tài liệu `BBox_Polygon_Polyline_Annotation_Guideline_V1.docx` làm căn cứ chính cho batch hiện tại. Các class/attribute của bản v0 không xuất hiện trong taxonomy của tài liệu này (ví dụ `ignore_region`, `can_review`, `occluded_level`) không được tự ý thêm vào task CVAT nếu chưa có xác nhận từ reviewer/mentor.
