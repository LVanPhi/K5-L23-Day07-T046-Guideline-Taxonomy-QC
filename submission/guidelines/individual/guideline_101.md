# Guideline gán nhãn — v0.1 — 101

> Bản cá nhân dựa trên v0, bổ sung quy tắc cho các ca chưa rõ sau khi xem cả 40 ảnh.

- Phiên bản: v0.1
- Ngày: 2026-11-16
- Người viết: 101

## 1. Mục tiêu

Nhãn dùng để huấn luyện mô hình **phát hiện người và phương tiện trên ảnh đường phố 2D**, phục vụ camera giao thông hoặc hệ thống xe tự lái.

- Mô hình cần phát hiện **vật thật đang tham gia hoặc hiện diện trên đường/vỉa hè**, không phải vật trang trí, mô hình, hay hình ảnh trên biển quảng cáo.
- Sai sót nguy hiểm nhất: **bỏ sót người hoặc phương tiện thật** trên đường → có thể gây tai nạn.
- Sai sót ít nguy hiểm hơn: gán nhầm tượng/mô hình thành vật thật → chỉ tạo cảnh báo giả.

## 2. Lớp và thuộc tính

### Lớp

| Lớp | Định nghĩa (một câu) | Ví dụ | Phản ví dụ / dễ nhầm với |
|---|---|---|---|
| `car` | Ô tô con, SUV, sedan, hatchback, taxi — xe 4 bánh chở người, cabin kín. | Sedan, SUV (img_012 Mazda CX-5), taxi (img_040) | Pickup có thùng hở → `truck`; van kín thùng → `van` |
| `truck` | Xe tải, xe bán tải (pickup), xe có thùng hàng riêng biệt với cabin. | Xe tải Isuzu (img_007), pickup Hilux (img_007), xe tải chở xe (img_002), xe tải nhỏ có bạt (img_036) | Van kín liền khối → `van` |
| `van` | Xe thùng kín cỡ nhỏ–trung, cabin và thùng liền khối. | Xe van trắng (img_005 phía sau), minibus (img_009) | Xe tải có cabin tách thùng → `truck` |
| `motorcycle` | Xe máy, mô tô, scooter — xe 2 bánh có động cơ. | Honda (img_006), Vespa (img_016), xe máy chở hàng (img_029) | Xe đạp (không có động cơ) → không gán; tuk-tuk (3 bánh) → xem R-01 |
| `person` | Người thật, đang đi bộ, ngồi, đứng, hoặc trên xe — miễn là nhìn thấy phần cơ thể. | Người đi bộ (img_014), người trên xe máy (img_005), người bán hàng (img_010) | Ma-nơ-canh (img_033), tượng (img_017), người trên billboard (img_018) → không gán |
| `ignore_region` | Vùng bỏ qua: nơi có quá nhiều vật chồng nhau không thể gán từng vật, hoặc vùng cần loại trừ. Bỏ sót ở đây không tính lỗi. | Bãi xe máy quá dày (img_030), đám đông xa (img_021 phần xa) | Không dùng cho vật đơn lẻ có thể gán được |

**Vật không thuộc lớp nào** (xe đạp, tuk-tuk, bus, xích lô, auto-rickshaw, xe lam) → **không gán**. Nếu chúng chiếm vùng lớn và che khuất vật khác, cân nhắc `ignore_region`. Xem chi tiết tại R-01.

### Thuộc tính

Áp dụng cho tất cả các lớp trừ `ignore_region`.

| Thuộc tính | Giá trị | Ngưỡng cụ thể |
|---|---|---|
| `occluded_level` | `0` | Thấy ≥ 80% diện tích vật |
| | `1` | Thấy 50–80% — bị che một phần bởi vật khác |
| | `2` | Thấy 20–50% — bị che phần lớn, nhưng vẫn nhận ra được lớp |
| | `3` | Thấy < 20% — gần như bị che hết, không chắc lớp. Luôn kèm `can_review` |
| `truncated` | `no` | Vật nằm hoàn toàn trong ảnh |
| | `yes` | Vật bị cắt ở mép ảnh (bất kỳ cạnh nào) |
| `can_review` | tick | Tick khi không chắc về lớp, box, hoặc quyết định |

## 3. Quy tắc vẽ box

### Nguyên tắc chung
1. **Box ôm sát phần nhìn thấy** của vật, không bao gồm phần bị che.
2. **Mỗi vật thật một box** — không gộp nhiều vật vào 1 box.
3. Chọn đúng lớp và điền đủ thuộc tính.
4. **Kích thước tối thiểu**: box phải cao ≥ 20 pixel (theo `min_box_height_px` trong taxonomy rules). Vật nhỏ hơn → không gán.

### R-01 — Phương tiện ngoài taxonomy
**Quy tắc**: Tuk-tuk, xe đạp, xích lô, bus, auto-rickshaw, xe lam — **không gán** vì không có lớp phù hợp. Không ép vào lớp khác (sẽ gây nhầm cho mô hình).

| Ảnh | Vật | Quyết định |
|---|---|---|
| `img_004` | 2 tuk-tuk | Không gán tuk-tuk. Gán `person` cho tài xế nếu thấy rõ ≥ 20px |
| `img_009` | Minibus | Gán `van` (thùng kín liền cabin, cỡ nhỏ–trung) |
| `img_016` | 2 tuk-tuk + Vespa + van | Tuk-tuk: không gán. Vespa: `motorcycle`. Van: `van` |
| `img_023` | Xích lô | Không gán xích lô. Gán `person` cho người đạp |
| `img_035` | Auto-rickshaw chở hàng | Không gán |
| `img_040` | Tuk-tuk + bus lớn | Tuk-tuk: không gán. Bus: không gán (không có lớp) |
| `img_003` | Xe đạp | Không gán xe đạp. Gán `person` cho người đi xe đạp |
| `img_006` | Xe đạp trong dòng xe | Không gán xe đạp |
| `img_029` | Xe đạp + người vác thanh kim loại | Không gán xe đạp. Gán `person` |
| `img_037` | Bus (GO Transit) | Không gán bus |

**Lý do**: Nếu gán tuk-tuk vào `motorcycle` hay `car`, mô hình sẽ học sai đặc trưng của lớp đó. Tốt hơn là bỏ qua và để lớp sau (Day 08+) bổ sung taxonomy.

### R-02 — Người trên xe
**Quy tắc**: Người trên xe (xe máy, xe đạp, tuk-tuk) → **gán box `person` riêng** nếu phần cơ thể nhìn thấy ≥ 20px chiều cao. Box `motorcycle` chỉ bao xe, không bao người.

- Người lái xe máy: 1 box `person` + 1 box `motorcycle` (2 box chồng nhau là bình thường).
- Hành khách phía sau: 1 box `person` riêng, `occluded_level` theo mức bị che bởi người lái.
- Người trong ô tô kín: **không gán** (không nhìn thấy cơ thể rõ ràng qua kính).

| Ảnh | Quyết định |
|---|---|
| `img_005` | Mỗi người trên xe máy = 1 box `person`. Mỗi xe = 1 box `motorcycle`. Người bị che nhiều: occluded_level 1–2 |
| `img_006` | Tương tự. Người phía sau chỉ thấy lưng/đầu: `person` occluded_level 1 |
| `img_008` | 2 xe máy đỗ, nhiều người. Mỗi người 1 box. Người chỉ thấy vai: occluded_level 2 |
| `img_010` | Người bán hàng đứng: `person` occ 0. Người ngồi trên xe: `person` occ 1. Xe: `motorcycle` |
| `img_026` | 2 người trên xe máy rõ ràng: 2 box `person` + 1 box `motorcycle` |

### R-03 — Tượng / mô hình / ma-nơ-canh / đồ chơi
**Quy tắc**: **Không gán**. Chỉ gán vật thật có thể tham gia giao thông. Tượng, mô hình, đồ chơi, ma-nơ-canh không phải đối tượng mô hình cần phát hiện.

| Ảnh | Vật | Quyết định |
|---|---|---|
| `img_011` | Tượng lớn + xe mô hình kim loại | Không gán. Gán `person` cho người thật ở xa nếu ≥ 20px |
| `img_017` | Tượng đồng cưỡi ngựa | Không gán → ảnh trống (0 annotation), vẫn giữ trong dataset |
| `img_019` | Xe giả tưởng (Danger Mouse) diễu hành | Gán `car` — vì nó **đang chạy trên đường**, có biển số, người thật ngồi trong. Tick `can_review` |
| `img_024` | Xe mô hình trên nóc ô tô + xe trong bảo tàng | Xe mô hình nhỏ: không gán. Xe thật trong bảo tàng: không gán (không trên đường) |
| `img_027` | Tượng xe pickup (nghĩa trang) | Không gán → ảnh trống |
| `img_028` | Tượng ngựa trên xe bán tải | Tượng ngựa: không gán. Xe bán tải thật: `truck`. Xe đỏ phía sau: `car` |
| `img_033` | Ma-nơ-canh cạnh xe van | Ma-nơ-canh: không gán. Phần xe van (gương chiếu hậu): `van`, truncated=yes nếu chỉ thấy 1 phần |
| `img_038` | Hàng chục xe đồ chơi trong tủ kính | Không gán → ảnh trống |

**Ngoại lệ**: Xe diễu hành/xe cosplay **đang chạy trên đường** với người thật bên trong → vẫn gán, kèm `can_review`.

### R-04 — Phản chiếu / hình in / biển quảng cáo
**Quy tắc**: **Không gán** vật trong phản chiếu, gương, biển quảng cáo, poster, tranh tường.

| Ảnh | Vật | Quyết định |
|---|---|---|
| `img_014` | Xe máy bị nhòe (motion blur) phía sau | Nếu vẫn nhận ra là xe/người thật → gán, occluded_level 3, `can_review`. Nếu chỉ là vệt → không gán |
| `img_018` | Người trên billboard | Không gán — hình in, không phải người thật |
| `img_020` | Xe trong gương chiếu hậu | Không gán xe trong gương. Ảnh trống (trừ khi thấy phần xe chủ bên ngoài gương) |
| `img_026` | Poster cảng biển phía sau | Không gán nội dung poster |
| `img_033` | Phản chiếu xe van trên kính | Không gán phản chiếu |

### R-05 — Kích thước tối thiểu
**Quy tắc**: Vật có **chiều cao box < 20 pixel** → **không gán**. Ước lượng bằng mắt hoặc dùng ruler trong CVAT.

| Ảnh | Áp dụng |
|---|---|
| `img_003` | Xe/người cuối đường nếu < 20px → bỏ |
| `img_021` | Người/xe ở phần xa (giữa-sau ảnh) rất nhỏ → hầu hết bỏ, chỉ gán phần gần ở tiền cảnh |
| `img_025` | Người xa trên bờ biển → kiểm 20px |
| `img_032` | Người đứng xa sau barrier → kiểm 20px |

### R-06 — Vật bị cắt mép ảnh (truncated)
**Quy tắc**: Vật bị cắt ở mép ảnh → **vẫn gán** nếu **phần nhìn thấy đủ để nhận ra lớp** (ước lượng ≥ 20% vật trong ảnh). Đặt `truncated=yes`.

- Chỉ thấy mép bánh xe hoặc mảnh nhỏ không nhận ra lớp → **không gán**.
- Box vẽ sát mép ảnh, ôm phần nhìn thấy.

| Ảnh | Quyết định |
|---|---|
| `img_001` | Truck bên phải: thấy phần lớn → gán, truncated=yes. Người bên trái: thấy nửa trên → gán |
| `img_005` | Xe/người bị cắt 2 bên: gán nếu nhận ra lớp |
| `img_012` | SUV trái cắt 1/3: gán. Pickup phải cắt ít: gán |
| `img_015` | Xe 2 bên cắt ít → gán |

### R-07 — Chi tiết occluded_level
Ngưỡng dựa trên **% diện tích vật bị che bởi vật khác** (không tính cắt mép — đó là `truncated`):

| Level | % diện tích nhìn thấy | Ví dụ |
|---|---|---|
| `0` | ≥ 80% | Xe đỗ riêng, người đi bộ thoáng (img_014, img_034) |
| `1` | 50–80% | Người sau tay lái xe máy (img_006), xe bị che 1 phần bởi xe khác (img_007) |
| `2` | 20–50% | Người phía sau bị che bởi người lái (img_008), xe trong đám đông chỉ thấy nóc + kính (img_021) |
| `3` | < 20% | Gần như chỉ thấy mảnh nhỏ, không chắc lớp. Luôn tick `can_review`. Nếu < 20px → không gán |

### R-08 — Đám đông và `ignore_region`
**Quy tắc**: Dùng `ignore_region` khi **không thể phân biệt và vẽ box riêng cho từng vật** vì quá chồng chéo.

- **Quy tắc ngón tay**: Nếu có > 10 vật chồng nhau trong một vùng và mỗi vật chỉ thấy < 30% → dùng `ignore_region` cho cả vùng đó.
- Vật ở rìa đám đông vẫn phân biệt được → gán bình thường.
- `ignore_region` vẽ 1 box lớn bao vùng đông, không cần thuộc tính.

| Ảnh | Quyết định |
|---|---|
| `img_021` | Phần gần (3–4 hàng đầu): gán từng xe/người có thể. Phần xa phía sau: `ignore_region` |
| `img_030` | Bãi xe máy dày đặc: `ignore_region` cho cả bãi, trừ xe rìa ngoài nhìn rõ |
| `img_032` | Đám đông người đứng: gán từng người nếu ≥ 20px và phân biệt được. Phần quá xa/nhỏ: bỏ (dưới 20px) |

## 4. Ca khó

| Ảnh | Tình huống | Rule ID | Quyết định | Lý do |
|---|---|---|---|---|
| `img_002` | Xe tải chở 6+ ô tô | R-03 | Xe tải: `truck`. Xe trên xe tải: gán `car` từng chiếc nếu ≥ 20px, occ theo mức che | Xe trên carrier là xe thật, chỉ đang được vận chuyển |
| `img_004` | 2 tuk-tuk | R-01 | Không gán tuk-tuk. Gán `person` cho tài xế | Tuk-tuk không có lớp |
| `img_009` | Minibus | R-01 | Gán `van` | Thùng kín liền cabin, cỡ trung |
| `img_011` | Tượng + xe mô hình kim loại | R-03 | Không gán | Không phải vật thật |
| `img_017` | Tượng cưỡi ngựa | R-03 | Không gán → ảnh trống | Tượng đồng, không phải người/xe thật |
| `img_019` | Xe Danger Mouse diễu hành | R-03 | Gán `car` + `can_review` | Đang chạy trên đường, có người thật, có biển số |
| `img_020` | Xe trong gương chiếu hậu | R-04 | Không gán → ảnh trống | Phản chiếu, không phải vật thật ngoài đường |
| `img_021` | Hàng trăm xe máy + người | R-08 | Gán phần gần, `ignore_region` phần xa | Quá đông, không thể gán từng vật ở phần sau |
| `img_022` | Xe máy + sidecar cổ, ảnh B&W | R-01 | Gán `motorcycle` | Sidecar là phụ kiện của xe máy, vẫn là motorcycle |
| `img_024` | Xe mô hình + xe bảo tàng | R-03 | Không gán | Không trên đường giao thông |
| `img_027` | Tượng xe pickup nghĩa trang | R-03 | Không gán → ảnh trống | Tượng, không phải xe thật |
| `img_029` | Người vác thanh kim loại trên xe máy | R-02 | `motorcycle` box ôm xe (không bao thanh sắt). `person` box ôm người | Thanh kim loại là hàng hóa, không phải vật cần phát hiện |
| `img_030` | Bãi xe máy dày đặc | R-08 | `ignore_region` | Không thể phân biệt từng xe |
| `img_033` | Ma-nơ-canh + phản chiếu xe van | R-03, R-04 | Không gán cả hai | Ma-nơ-canh không phải người; phản chiếu không phải xe thật |
| `img_035` | Auto-rickshaw chở hàng cao | R-01 | Không gán rickshaw. Gán `motorcycle` + `person` nếu thấy | Không có lớp phù hợp |
| `img_036` | Xe tải nhỏ có thùng bạt | — | Gán `truck` | Cabin tách thùng rõ ràng |
| `img_037` | Bus + xe bán tải lớn có thùng kín | R-01 | Bus: không gán. Pickup có thùng kín: `truck` | Bus không có lớp. Pickup có thùng riêng = truck |
| `img_038` | Xe đồ chơi trong tủ | R-03 | Không gán → ảnh trống | Đồ chơi, không phải xe thật |
| `img_040` | Tuk-tuk + bus + taxi + xe máy | R-01 | Tuk-tuk/bus: không gán. Taxi: `car`. Xe máy: `motorcycle` | Chỉ gán lớp có trong taxonomy |

## 5. QC và phiên bản

Ngưỡng QC (từ `taxonomy.json` — giữ nguyên, không sửa):
- `min_box_height_px`: 20
- `iou_thr_large`: 0.85 (box lớn ≥ 50px)
- `iou_thr_small`: 0.70 (box nhỏ < 50px)
- `iaa_iou`: 0.85
- `iaa_class`: 0.95
- `error_rate_img_max`: 0.02 (2%)
- `match_iou`: 0.5

Ca mơ hồ: tick `can_review` trên box + ghi vấn đề cấp ảnh vào `image_issues.csv`.

| Phiên bản | Thay đổi | Lý do (ca nào) |
|---|---|---|
| v0 | Bản gốc giảng viên | — |
| v0.1 (bản này) | Bổ sung R-01→R-08, ngưỡng occluded, ca khó, quy tắc phương tiện ngoài taxonomy | Xem 40 ảnh: nhiều ca v0 không xử lý được (img_002, 004, 011, 017, 019, 020, 021, 027, 030, 033, 038) |
