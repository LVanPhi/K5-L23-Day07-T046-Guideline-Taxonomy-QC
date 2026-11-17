# Guideline gán nhãn — v1.0 (Thống nhất nhóm)

> Bản đặc tả thống nhất của nhóm trước khi khóa nhãn và gán độc lập ở Pha 3.

- Phiên bản: v1.0
- Ngày: 2026-11-16
- Nhóm thực hiện: T046 (101, 102, 103)

## 1. Mục tiêu

Nhãn dùng để huấn luyện mô hình **phát hiện người và phương tiện trên ảnh đường phố 2D**, phục vụ camera giao thông hoặc hệ thống xe tự lái[cite: 12].

- Mô hình cần phát hiện **vật thật đang tham gia hoặc hiện diện trong không gian giao thông ngoài trời (đường, vỉa hè, bãi đỗ, trên xe chở hàng)**, không gán vật trưng bày trong nhà/tủ kính, vật trang trí, mô hình, hay hình ảnh trên biển quảng cáo[cite: 12].
- Sai sót nguy hiểm nhất: **bỏ sót người hoặc phương tiện thật** trên đường → có thể gây tai nạn[cite: 12].
- Sai sót ít nguy hiểm hơn: gán nhầm tượng/mô hình thành vật thật → chỉ tạo cảnh báo giả[cite: 12].

## 2. Lớp và thuộc tính

### Lớp (chỉ sử dụng Bounding Box)

| Lớp | Định nghĩa (một câu) | Ví dụ | Phản ví dụ / dễ nhầm với |
|---|---|---|---|
| `car` | Ô tô con, SUV, sedan, hatchback, taxi — xe 4 bánh chở người, cabin kín[cite: 12]. | Sedan, SUV (img_012 Mazda CX-5), taxi (img_040)[cite: 12] | Pickup có thùng hở → `truck`; van kín thùng → `van`[cite: 12] |
| `truck` | Xe tải, xe bán tải (pickup), xe có thùng hàng riêng biệt với cabin[cite: 12]. | Xe tải Isuzu (img_007), pickup Hilux (img_007), xe tải chở xe (img_002), xe tải nhỏ có bạt (img_036)[cite: 12] | Van kín liền khối → `van`[cite: 12] |
| `van` | Xe thùng kín cỡ nhỏ–trung, cabin và thùng liền khối[cite: 12]. | Xe van trắng (img_005 phía sau), minibus (img_009)[cite: 12] | Xe tải có cabin tách thùng → `truck`[cite: 12] |
| `motorcycle` | Xe máy, mô tô, scooter — xe 2 bánh có động cơ (bao gồm xe có sidecar)[cite: 12]. | Honda (img_006), Vespa (img_016), xe máy sidecar (img_022), xe máy chở hàng (img_029)[cite: 12] | Xe đạp (không có động cơ) → không gán; tuk-tuk (3 bánh) → xem R-01[cite: 12] |
| `person` | Người thật, đang đi bộ, ngồi, đứng, hoặc trên phương tiện cabin hở — miễn là nhìn thấy phần cơ thể[cite: 12]. | Người đi bộ (img_014), người trên xe máy (img_005), người bán hàng (img_010)[cite: 12] | Ma-nơ-canh (img_033), tượng (img_017), người trên billboard (img_018) → không gán[cite: 12] |
| `ignore_region` | Vùng bỏ qua: nơi có quá nhiều vật chồng nhau không thể gán từng vật, hoặc vùng cần loại trừ[cite: 12]. Bỏ sót ở đây không tính lỗi[cite: 12]. | Bãi xe máy quá dày (img_030), đám đông xa (img_021 phần xa)[cite: 12] | Không dùng cho vật đơn lẻ có thể gán được[cite: 12] |

**Vật không thuộc lớp nào** (xe đạp, tuk-tuk, bus, xích lô, auto-rickshaw, xe lam) → **không gán**[cite: 12]. Nếu chúng chiếm vùng lớn và che khuất vật khác, cân nhắc `ignore_region`[cite: 12]. Xem chi tiết tại R-01[cite: 12].

### Thuộc tính

Áp dụng cho tất cả các lớp trừ `ignore_region`[cite: 12].

| Thuộc tính | Giá trị | Ngưỡng cụ thể |
|---|---|---|
| `occluded_level` | `0` | Thấy ≥ 80% diện tích vật[cite: 12] |
| | `1` | Thấy 50–80% — bị che một phần bởi vật khác[cite: 12] |
| | `2` | Thấy 20–50% — bị che phần lớn, nhưng vẫn nhận ra được lớp[cite: 12] |
| | `3` | Thấy < 20% — gần như bị che hết, không chắc lớp[cite: 12]. Luôn kèm `can_review`[cite: 12] |
| `truncated` | `no` | Vật nằm hoàn toàn trong ảnh[cite: 12] |
| | `yes` | Vật bị cắt ở mép ảnh (bất kỳ cạnh nào)[cite: 12] |
| `can_review` | tick | Tick khi không chắc về lớp, box, hoặc quyết định[cite: 12] |

## 3. Quy tắc vẽ box

### Nguyên tắc chung
1. **Canh theo 4 điểm cực trị (Extreme Points)**: Vẽ BBox hình chữ nhật bám sát 4 biên `left, right, top, bottom` của **phần cơ thể/phương tiện thật nhìn thấy**, không bao gồm phần bị che, bóng đổ, hay hàng hóa/vật dụng mang theo (ba lô, ô, thanh kim loại)[cite: 12, 15].
2. **Mỗi vật thật một box** — không gộp nhiều vật vào 1 box; các BBox được phép chồng lấn khi đối tượng nằm gần hoặc che khuất nhau[cite: 12, 15].
3. Chọn đúng lớp và điền đủ thuộc tính[cite: 12].
4. **Kích thước tối thiểu**: box phải cao ≥ 20 pixel (theo `min_box_height_px` trong taxonomy rules)[cite: 12]. Vật nhỏ hơn → không gán[cite: 12].

### R-01 — Phương tiện ngoài taxonomy
**Quy tắc**: Tuk-tuk, xe đạp, xích lô, bus, auto-rickshaw, xe lam — **không gán** vì không có lớp phù hợp[cite: 12]. Không ép vào lớp khác (sẽ gây nhầm cho mô hình)[cite: 12].

| Ảnh | Vật | Quyết định |
|---|---|---|
| `img_004` | 2 tuk-tuk | Không gán tuk-tuk[cite: 12]. Gán `person` cho tài xế nếu thấy rõ ≥ 20px[cite: 12] |
| `img_009` | Minibus | Gán `van` (thùng kín liền cabin, cỡ nhỏ–trung)[cite: 12] |
| `img_016` | 2 tuk-tuk + Vespa + van | Tuk-tuk: không gán[cite: 12]. Vespa: `motorcycle`[cite: 12]. Van: `van`[cite: 12] |
| `img_023` | Xích lô | Không gán xích lô[cite: 12]. Gán `person` cho người đạp[cite: 12] |
| `img_035` | Auto-rickshaw chở hàng | Không gán[cite: 12] |
| `img_040` | Tuk-tuk + bus lớn | Tuk-tuk: không gán[cite: 12]. Bus: không gán (không có lớp)[cite: 12] |
| `img_003` | Xe đạp | Không gán xe đạp[cite: 12]. Gán `person` cho người đi xe đạp[cite: 12] |
| `img_006` | Xe đạp trong dòng xe | Không gán xe đạp[cite: 12] |
| `img_029` | Xe đạp + người vác thanh kim loại | Không gán xe đạp[cite: 12]. Gán `person`[cite: 12] |
| `img_037` | Bus (GO Transit) | Không gán bus[cite: 12] |

### R-02 — Người trên xe (cabin hở vs cabin kín)
**Quy tắc**:
- **Phương tiện cabin hở / không kính chắn kín** (xe máy, xe đạp, xích lô, tuk-tuk, xe mui trần): gán box `person` riêng cho từng người nếu phần cơ thể nhìn thấy ≥ 20px chiều cao[cite: 12]. Box xe chỉ bao xe, không bao người hay hàng hóa[cite: 12].
- **Phương tiện cabin kín có kính** (ô tô con, xe tải, xe van): **không gán** `person` ngồi bên trong (do kính phản chiếu/che khuất không rõ cơ thể)[cite: 12].

| Ảnh | Quyết định |
|---|---|
| `img_005` | Mỗi người trên xe máy = 1 box `person`[cite: 12]. Mỗi xe = 1 box `motorcycle`[cite: 12]. Người bị che nhiều: occluded_level 1–2[cite: 12] |
| `img_006` | Tương tự[cite: 12]. Người phía sau chỉ thấy lưng/đầu: `person` occluded_level 1[cite: 12] |
| `img_008` | 2 xe máy đỗ, nhiều người[cite: 12]. Mỗi người 1 box[cite: 12]. Người chỉ thấy vai: occluded_level 2[cite: 12] |
| `img_010` | Người bán hàng đứng: `person` occ 0[cite: 12]. Người ngồi trên xe: `person` occ 1[cite: 12]. Xe: `motorcycle`[cite: 12] |
| `img_026` | 2 người trên xe máy rõ ràng: 2 box `person` + 1 box `motorcycle`[cite: 12] |

### R-03 — Tượng / mô hình / ma-nơ-canh / đồ chơi / xe trưng bày
**Quy tắc**: **Không gán** tượng, mô hình, đồ chơi, ma-nơ-canh, hoặc xe trưng bày cố định trong nhà/bảo tàng[cite: 12]. Chỉ gán vật thật đang hiện diện hoặc tham gia trong không gian giao thông ngoài trời (bao gồm cả xe thật đang được vận chuyển trên xe tải chở hàng)[cite: 12].

| Ảnh | Vật | Quyết định |
|---|---|---|
| `img_002` | Xe tải chở 6+ ô tô | Xe tải: `truck`[cite: 12]. Ô tô thật trên carrier: gán `car` từng chiếc nếu ≥ 20px[cite: 12] |
| `img_011` | Tượng lớn + xe mô hình kim loại | Không gán[cite: 12]. Gán `person` cho người thật ở xa nếu ≥ 20px[cite: 12] |
| `img_017` | Tượng đồng cưỡi ngựa | Không gán → ảnh trống (0 annotation), vẫn giữ trong dataset[cite: 12] |
| `img_019` | Xe giả tưởng (Danger Mouse) diễu hành | Gán `car` — vì đang chạy trên đường ngoài trời, có biển số, người thật lái (gán thêm `person` vì cabin hở)[cite: 12]. Tick `can_review`[cite: 12] |
| `img_024` | Xe mô hình trên nóc ô tô + xe trong bảo tàng | Không gán cả hai (mô hình giả + vật trưng bày trong nhà)[cite: 12] |
| `img_027` | Tượng xe pickup (nghĩa trang) | Không gán → ảnh trống[cite: 12] |
| `img_028` | Tượng ngựa trên xe bán tải | Tượng ngựa: không gán[cite: 12]. Xe bán tải thật: `truck`[cite: 12]. Xe đỏ phía sau: `car`[cite: 12] |
| `img_033` | Ma-nơ-canh cạnh kính cửa hàng | Ma-nơ-canh: không gán[cite: 12]. Phần gương xe van ở mép ảnh quá nhỏ (< 20% diện tích xe, không đủ nhận diện thân xe) → không gán (theo R-06) → ảnh trống |
| `img_038` | Hàng chục xe đồ chơi trong tủ kính | Không gán → ảnh trống[cite: 12] |

### R-04 — Phản chiếu / hình in / biển quảng cáo / motion blur
**Quy tắc**: **Không gán** vật trong phản chiếu, gương chiếu hậu, kính cửa hàng, biển quảng cáo, poster, tranh tường[cite: 12, 15].

| Ảnh | Vật | Quyết định |
|---|---|---|
| `img_014` | Xe máy bị nhòe (motion blur) phía sau | Nếu vẫn nhận ra là xe/người thật → gán, occluded_level 3, `can_review`[cite: 12]. Nếu chỉ là vệt màu → không gán[cite: 12] |
| `img_018` | Người trên billboard | Không gán — hình in, không phải người thật[cite: 12] |
| `img_020` | Xe trong gương chiếu hậu | Không gán xe trong gương[cite: 12]. Ảnh trống (trừ khi thấy phần xe chủ bên ngoài gương ≥ 20%)[cite: 12] |
| `img_026` | Poster cảng biển phía sau | Không gán nội dung poster[cite: 12] |
| `img_033` | Phản chiếu xe van trên kính | Không gán phản chiếu[cite: 12] |

### R-05 — Kích thước tối thiểu
**Quy tắc**: Vật có **chiều cao box < 20 pixel** → **không gán**[cite: 12].

| Ảnh | Áp dụng |
|---|---|
| `img_003` | Xe/người cuối đường nếu < 20px → bỏ[cite: 12] |
| `img_021` | Người/xe ở phần xa (giữa-sau ảnh) rất nhỏ → bỏ hoặc gộp `ignore_region`, chỉ gán phần gần ở tiền cảnh[cite: 12] |
| `img_025` | Người xa trên bờ biển → kiểm 20px[cite: 12] |
| `img_032` | Người đứng xa sau barrier → kiểm 20px[cite: 12] |

### R-06 — Vật bị cắt mép ảnh (truncated)
**Quy tắc**: Vật bị cắt ở mép ảnh → **vẫn gán** nếu **phần nhìn thấy đủ để nhận ra lớp** (ước lượng ≥ 20% diện tích vật trong ảnh) và đặt `truncated=yes`[cite: 12].

- Chỉ thấy mép bánh xe, gương chiếu hậu, hoặc mảnh nhỏ không nhận ra lớp (< 20%) → **không gán**[cite: 12].
- Box vẽ sát mép ảnh, ôm phần nhìn thấy[cite: 12].

| Ảnh | Quyết định |
|---|---|
| `img_001` | Truck bên phải: thấy phần lớn → gán `truck`, truncated=yes[cite: 12]. Người bên trái: thấy nửa trên → gán `person`, truncated=yes[cite: 12] |
| `img_005` | Xe/người bị cắt 2 bên: gán nếu nhận ra lớp (≥ 20%), truncated=yes[cite: 12] |
| `img_012` | SUV trái cắt 1/3: gán `car`, truncated=yes[cite: 12]. Pickup phải cắt ít: gán `truck`, truncated=yes[cite: 12] |
| `img_015` | Xe 2 bên cắt ít → gán, truncated=yes[cite: 12] |

### R-07 — Chi tiết occluded_level
Ngưỡng dựa trên **% diện tích vật bị che bởi vật khác** (không tính phần bị cắt bởi mép ảnh — phần đó tính ở `truncated`)[cite: 12]:

| Level | % diện tích nhìn thấy | Ví dụ |
|---|---|---|
| `0` | ≥ 80% | Xe đỗ riêng, người đi bộ thoáng (img_014, img_034)[cite: 12] |
| `1` | 50–80% | Người sau tay lái xe máy (img_006), xe bị che 1 phần bởi xe khác (img_007)[cite: 12] |
| `2` | 20–50% | Người phía sau bị che bởi người lái (img_008), xe trong đám đông chỉ thấy nóc + kính (img_021)[cite: 12] |
| `3` | < 20% | Gần như chỉ thấy mảnh nhỏ, không chắc lớp[cite: 12]. Luôn tick `can_review`[cite: 12]. Nếu < 20px → không gán[cite: 12] |

### R-08 — Đám đông và `ignore_region`
**Quy tắc**: Dùng `ignore_region` khi **không thể phân biệt và vẽ box riêng cho từng vật** vì quá chồng chéo[cite: 12].

- **Quy tắc ngón tay**: Nếu có > 10 vật chồng nhau trong một vùng và mỗi vật chỉ thấy < 30% → dùng `ignore_region` cho cả vùng đó[cite: 12].
- Vật ở rìa đám đông vẫn phân biệt được (≥ 20px) → gán bình thường[cite: 12].
- `ignore_region` vẽ 1 box lớn bao vùng đông, không cần thuộc tính[cite: 12].

| Ảnh | Quyết định |
|---|---|
| `img_021` | Phần gần (3–4 hàng đầu): gán từng xe/người có thể[cite: 12]. Phần xa phía sau: `ignore_region`[cite: 12] |
| `img_030` | Bãi xe máy dày đặc: `ignore_region` cho cả bãi, trừ xe rìa ngoài nhìn rõ[cite: 12] |
| `img_032` | Đám đông người đứng: gán từng người nếu ≥ 20px và phân biệt được[cite: 12]. Phần quá xa/nhỏ: bỏ (dưới 20px)[cite: 12] |

## 4. Bảng tra cứu nhanh ca khó

| Ảnh | Tình huống | Rule ID | Quyết định | Lý do |
|---|---|---|---|---|
| `img_002` | Xe tải chở 6+ ô tô | R-03 | Xe tải: `truck`[cite: 12]. Xe trên xe tải: gán `car` từng chiếc nếu ≥ 20px, occ theo mức che[cite: 12] | Xe trên carrier là xe thật trong không gian giao thông[cite: 12] |
| `img_004` | 2 tuk-tuk | R-01, R-02 | Không gán tuk-tuk[cite: 12]. Gán `person` cho tài xế[cite: 12] | Tuk-tuk không có lớp; cabin hở nên gán tài xế[cite: 12] |
| `img_009` | Minibus | R-01 | Gán `van`[cite: 12] | Thùng kín liền cabin, cỡ trung[cite: 12] |
| `img_011` | Tượng + xe mô hình kim loại | R-03 | Không gán[cite: 12] | Không phải vật thật[cite: 12] |
| `img_017` | Tượng cưỡi ngựa | R-03 | Không gán → ảnh trống[cite: 12] | Tượng đồng, không phải người/xe thật[cite: 12] |
| `img_019` | Xe Danger Mouse diễu hành | R-02, R-03 | Gán `car` + `person` (người lái cabin hở) + `can_review`[cite: 12] | Đang chạy trên đường ngoài trời, có người thật, có biển số[cite: 12] |
| `img_020` | Xe trong gương chiếu hậu | R-04 | Không gán → ảnh trống[cite: 12] | Phản chiếu, không phải vật thật ngoài đường[cite: 12] |
| `img_021` | Hàng trăm xe máy + người | R-08 | Gán phần gần, `ignore_region` phần xa[cite: 12] | Quá đông, không thể gán từng vật ở phần sau[cite: 12] |
| `img_022` | Xe máy + sidecar cổ, ảnh B&W | R-01 | Gán `motorcycle` (bao gồm cả sidecar)[cite: 12] | Sidecar là bộ phận gắn liền của xe máy[cite: 12] |
| `img_024` | Xe mô hình + xe bảo tàng | R-03 | Không gán[cite: 12] | Vật trưng bày trong nhà/không trên đường giao thông[cite: 12] |
| `img_027` | Tượng xe pickup nghĩa trang | R-03 | Không gán → ảnh trống[cite: 12] | Tượng, không phải xe thật[cite: 12] |
| `img_029` | Người vác thanh kim loại trên xe máy | R-02 | `motorcycle` box ôm xe[cite: 12]. `person` box ôm người (không bao thanh sắt)[cite: 12] | Thanh kim loại là hàng hóa, không tính vào extreme points của người[cite: 12, 15] |
| `img_030` | Bãi xe máy dày đặc | R-08 | `ignore_region`[cite: 12] | Không thể phân biệt từng xe[cite: 12] |
| `img_033` | Ma-nơ-canh + phản chiếu/gương xe van | R-03, R-04, R-06 | Không gán → ảnh trống | Ma-nơ-canh giả; phản chiếu không gán; phần gương xe van < 20% không đủ nhận diện[cite: 12] |
| `img_035` | Auto-rickshaw chở hàng cao | R-01 | Không gán rickshaw[cite: 12]. Gán `motorcycle` + `person` nếu thấy[cite: 12] | Không có lớp phù hợp[cite: 12] |
| `img_036` | Xe tải nhỏ có thùng bạt | — | Gán `truck`[cite: 12] | Cabin tách thùng rõ ràng[cite: 12] |
| `img_037` | Bus + xe bán tải lớn có thùng kín | R-01 | Bus: không gán[cite: 12]. Pickup có thùng kín: `truck`[cite: 12] | Bus không có lớp[cite: 12]. Pickup có thùng riêng = truck[cite: 12] |
| `img_038` | Xe đồ chơi trong tủ | R-03 | Không gán → ảnh trống[cite: 12] | Đồ chơi, không phải xe thật[cite: 12] |
| `img_040` | Tuk-tuk + bus + taxi + xe máy | R-01 | Tuk-tuk/bus: không gán[cite: 12]. Taxi: `car`[cite: 12]. Xe máy: `motorcycle`[cite: 12] | Chỉ gán lớp có trong taxonomy[cite: 12] |

## 5. Quy trình QC, phân loại lỗi và phiên bản

### Ngưỡng QC (từ `taxonomy.json` — giữ nguyên, không sửa)
- `min_box_height_px`: 20[cite: 12]
- `iou_thr_large`: 0.85 (box lớn ≥ 50px)[cite: 12]
- `iou_thr_small`: 0.70 (box nhỏ < 50px)[cite: 12]
- `iaa_iou`: 0.85[cite: 12]
- `iaa_class`: 0.95[cite: 12]
- `error_rate_img_max`: 0.02 (2%)[cite: 12]
- `match_iou`: 0.5[cite: 12]

### Phân loại Issue chuẩn hóa (dùng cho `can_review` và `image_issues.csv`)
Khi gặp ca mơ hồ trong lúc gán độc lập ở Pha 3, tick `can_review` trên box[cite: 12] và ghi chú cấp ảnh theo các mã sau:
- `UNCERTAIN_CLASS`: Phân vân giữa 2 lớp (ví dụ: `car` vs `van`, `van` vs `truck`)[cite: 15].
- `UNCERTAIN_BOUNDARY`: Không rõ đường biên box do vật che khuất nặng (`occluded_level = 3`)[cite: 12, 15].
- `UNCERTAIN_SCOPE`: Không chắc đối tượng có thuộc phạm vi bài hay ngoài taxonomy[cite: 15].
- `ATTRIBUTE_CHECK`: Phân vân mức `occluded_level` hoặc trạng thái `truncated`[cite: 12, 15].
- `LOW_VISIBILITY`: Đối tượng khó quan sát do thiếu sáng, ngược sáng, bóng tối, quá nhỏ sát ngưỡng 20px, hoặc motion blur nặng[cite: 12, 15].

### Checklist tự kiểm tra trước khi Export (Pha 3)
- [ ] Đúng lớp và chỉ dùng Bounding Box theo `labels_generated.json`, không tự tạo/đổi tên lớp[cite: 12, 15].
- [ ] Không bỏ sót vật thật ≥ 20px thuộc scope; không vẽ trùng lặp 2 box cho cùng 1 vật[cite: 12, 15].
- [ ] BBox bám sát 4 điểm cực trị của phần nhìn thấy, không vượt biên ảnh[cite: 12, 15].
- [ ] Điền đủ `occluded_level` và `truncated` cho mọi box (trừ `ignore_region`)[cite: 12].
- [ ] Đã rà soát lại 2 lượt toàn bộ tập ảnh được giao trước khi xuất COCO 1.0[cite: 15].

### Lịch sử phiên bản

| Phiên bản | Thay đổi | Lý do (ca nào) |
|---|---|---|
| v0 | Bản gốc giảng viên[cite: 12] | — |
| v0.1 | Bản nháp cá nhân 101: bổ sung R-01→R-08, ngưỡng occluded, bảng ca khó[cite: 12] | Rà soát 40 ảnh đầu vào[cite: 12] |
| v1.0 (bản này) | Chốt thống nhất nhóm: sửa mâu thuẫn `img_033`, làm rõ cabin hở/kín (R-02), chuẩn hóa extreme points, mã `Issue type` và checklist QC[cite: 12, 15] | Khóa đặc tả trước khi sinh `cvat/labels_generated.json` và chia ảnh Pha 3 |