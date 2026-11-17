# Tóm tắt IAA (taxonomy v1.0)

- Ảnh được so sánh (có >=2 người gán): **40**; số người gán: 3
- Lượt gán đủ **80/80**.
- IoU trung bình và % cùng lớp **chỉ tính trên các cặp box đã ghép** (IoU >= 0.5, ghép 1-1 không phân biệt lớp). Box chưa ghép được thống kê riêng ở cột bỏ sót/thừa.
- Mẫu số: 330 cặp box ghép (tổng mọi cặp người gán, ảnh chung).

| Cặp | Ảnh | Cặp box ghép | IoU TB | Cùng lớp | Cùng occluded_level | Cùng truncated | Chưa ghép (A/B) | Bỏ qua do ignore | Đạt IAA |
|---|---|---|---|---|---|---|---|---|---|
| **TỔNG** | 40 | 330 | 0.900 | 98.5% | 70.0% | 97.9% | 63/102 | 0 | ĐẠT |
| 101|102 | 13 | 128 | 0.948 | 98.4% | 74.2% | 99.2% | 12/17 | 0 | ĐẠT |
| 101|103 | 13 | 125 | 0.874 | 98.4% | 69.6% | 95.2% | 25/58 | 0 | ĐẠT |
| 102|103 | 14 | 77 | 0.862 | 98.7% | 63.6% | 100.0% | 26/27 | 0 | ĐẠT |

Ngưỡng IAA: IoU TB >= 0.85, đồng thuận lớp >= 95%.

Số ca bất thường: 434 ({'sai_thuoc_tinh': 103, 'bo_sot_thua': 149, 'co_can_review': 81, 'iou_thap': 72, 'bat_dong_ignore': 24, 'sai_lop': 5}), trên 33 ảnh cần calibration.
