# Báo cáo QC — Nhóm T-046

## 1. Mục đích, dữ liệu và phiên bản

- Mục đích và giới hạn sử dụng: Đặc tả taxonomy, quy chuẩn gán nhãn và kiểm soát chất lượng (QC) bounding box 2D cho bài toán nhận diện đối tượng giao thông đường bộ (6 lớp: car, truck, van, motorcycle, person, ignore_region) trên tập 40 ảnh thực hành Day 07; giới hạn phạm vi trong điều kiện ánh sáng và góc nhìn của lô ảnh mẫu.
- Nguồn và phép sử dụng dữ liệu: Dữ liệu ảnh gốc lưu tại data/images/ với đầy đủ thông tin tác giả, giấy phép và đường dẫn nguồn trong data/ATTRIBUTION.csv; chỉ sử dụng cho mục đích học tập và đánh giá nội bộ khóa VinAI K5.
- Manifest và checksum nhãn cuối: submission/qc/batch_manifest.json (SHA-256: 4dd97972b2d4782f58cb8cf351d16436d2f29666211a2a49a4a1d1dcebefe371); file nhãn cuối submission/qc/labels_calibrated.json (SHA-256: 127d33a9d2d22cb7c55edbe9f7c0a73ff9dd6506e198c04f2608155a0d92dbf0).
- Guideline và taxonomy khi gán / khi bàn giao: Gán nhãn độc lập Pha 3 dùng submission/guidelines/guideline_v1.0_agreed.md và submission/taxonomy.json (v1.0); bàn giao cuối dùng submission/guidelines/guideline_v1.1_calibrated.md (cập nhật editorial theo submission/guideline_change.json) và giữ nguyên taxonomy 6 lớp.

## 2. Phân công và đồng thuận trước calibration

- Lượt hoàn thành / được giao: Lượt gán đủ **80/80**.
- Số cặp box ghép, IoU TB, đồng thuận lớp và mẫu số: IoU trung bình và % cùng lớp **chỉ tính trên các cặp box đã ghép** (IoU >= 0.5, ghép 1-1 không phân biệt lớp). Box chưa ghép được thống kê riêng ở cột bỏ sót/thừa. (chi tiết từng ảnh tại submission/qc/image_scores.csv và submission/qc/iaa_summary.json)
- Box chưa ghép, loại do ignore, bất đồng vùng ignore: IoU trung bình và % cùng lớp **chỉ tính trên các cặp box đã ghép** (IoU >= 0.5, ghép 1-1 không phân biệt lớp). Box chưa ghép được thống kê riêng ở cột bỏ sót/thừa.
- Ca hai người có thể cùng sai sau khi đối chiếu tham chiếu: img_002.jpg (xe tải chở 6 ô tô con trên giàn, quy tắc R-01), img_004.jpg (xe tuk-tuk đỗ ban đêm, quy tắc R-01), img_017.jpg và img_027.jpg (tượng đá và vật thể mô phỏng xe dễ gây nhận nhầm, quy tắc R-03); đã rà soát trực tiếp toàn bộ 40 ảnh gốc.

## 3. Calibration và kiểm lại

- Ca đã xử lý, rule ID, bản sửa và người xác nhận: Đã xử lý và đóng 434/434 ca (C001 đến C434) trên 33 ảnh gắn cờ trong submission/qc/calibration_decisions.csv theo các quy tắc R-01 đến R-08; người chốt chính 101 (Nguyễn Hoàng Diệu), kiểm lại chéo bởi 102 và 103, xuất bản hiệu chỉnh submission/qc/labels_calibrated_4d7d7207260c420b.json.
- Tác động v1.1 lên nhãn và ảnh: Loại hình cập nhật editorial (submission/guideline_change.json), làm rõ ranh giới ôm sát vật thể che khuất, ngưỡng kích thước nhỏ và vùng ignore_region trên 33 ảnh phạm vi calibration; gộp thành 495 annotation chuẩn trong submission/qc/labels_calibrated.json và tắt toàn bộ cờ can_review.
- Tham chiếu bằng chứng khi Rework: Không phát sinh vòng Rework; ảnh chồng nhãn trước calibration lưu tại work/viz/, ảnh hiệu chỉnh tại work/calib/images/ và ảnh overlay kiểm tra vòng 1 tại work/review/round-01/.

## 4. Kết quả duyệt

- Vòng QC: 1
- Người review và ngày: 101 (Nguyễn Hoàng Diệu), 102, 103 — ngày 2026-11-17
- Đã duyệt / cả lô: 40/40 ảnh (ca_kho: 33/33, con_lai: 7/7)
- Ảnh lỗi / ảnh đã duyệt: 0/40 (0.0%)
- Annotation đối tượng hiện có bị lỗi / annotation đối tượng đã duyệt: 0/495 (0.0%)
- Vật bỏ sót, ảnh và vùng: 0 vật bỏ sót trên 40/40 ảnh (ghi nhận đầy đủ trong submission/qc/reviews/round-01/qc_review.csv)
- Ca khó / phần còn lại, lỗi lặp và ví dụ: Nhóm ca_kho đạt 33/33 ảnh sạch (0.0% lỗi), nhóm con_lai đạt 7/7 ảnh sạch (0.0% lỗi); không ghi nhận lỗi lặp hệ thống trên phiếu submission/qc/reviews/round-01/qc_review.csv.
- Một lỗi máy chặn được và một lỗi cần người đọc ảnh: Lỗi máy chặn được là phát hiện ảnh thừa ngoài danh sách phân công (img_009.jpg trong 102.json) và box trùng lặp tọa độ qua validate_labels.py / qc_compare.py; lỗi cần người đọc ảnh là phân biệt vật thể giả không có tính chất tham gia giao thông (tượng, ma-nơ-canh, biển quảng cáo tại img_017.jpg, img_018.jpg, img_033.jpg) so với người và phương tiện thật.

## 5. Quyết định bàn giao

- Quyết định: Release
- Lý do, người nhận và phạm vi sử dụng: Đạt 100% tiêu chí nghiệm thu vòng 1 (40/40 ảnh đã duyệt toàn lô, tỷ lệ lỗi theo ảnh 0.0% <= 2%, 0/495 annotation lỗi, 0 box còn cờ can_review); bàn giao cho giảng viên/mentor chương trình VinAI K5 (lớp L2A) và đội ngũ huấn luyện mô hình nhận diện 2D trong phạm vi bài thực hành Day 07.
- Nếu còn việc: Không còn hạng mục tồn đọng; toàn bộ nhãn cá nhân, nhãn hợp nhất, ghi chú cá nhân và hồ sơ QC vòng 1 đã sẵn sàng bàn giao.
- Kiểm ngược từ báo cáo tới ảnh, rule và nhãn: Ca CAL-C001 (img_001.jpg, rule R-07) và CAL-C005 (img_002.jpg, rule R-01) truy vết từ submission/qc/calibration_decisions.csv -> ảnh overlay work/review/round-01/img_001.jpg và img_002.jpg -> nhãn cuối submission/qc/labels_calibrated.json (xác nhận error_type=khong trong submission/qc/reviews/round-01/qc_review.csv).
