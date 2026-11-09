# Báo cáo cá nhân — Ngày 2 Object Detection

## 1. Thông tin học viên

- Họ tên: Nguyễn Khánh Tùng
- MSSV: 2A202603144
- Email: sfpcshch@gmail.com
- Lớp: K5 Level 2
- Link repo bài làm: https://github.com/sfpcshch/K5-L2-Day02-NguyenKhanhTung-2A202603144-ObjectDetection
- Link Colab / Môi trường chạy: https://colab.research.google.com/github/sfpcshch/K5-L2-Day02-NguyenKhanhTung-2A202603144-ObjectDetection/blob/main/notebooks/day2_annotation_to_yolo.ipynb (kèm môi trường huấn luyện GPU cục bộ do Colab gặp sự cố cấp phát runtime hệ thống)

## 2. Schema và mini guideline

Xem chi tiết tại [GUIDELINE_MINI.md](../../GUIDELINE_MINI.md).

- **Tóm tắt 3 ca mơ hồ đã quyết định:**
  1. `drive_002`: Xe màu xám/trắng ở làn giữa (`x≈0.37, y≈0.43`), thân xe dạng hộp và trần cao đặc trưng của MPV/minivan, được quyết định gán nhãn `van` (ID 1) thay vì `car`.
  2. `drive_005`: Phương tiện cabin nhỏ nhưng phía sau gắn thùng hàng rời thương mại chuyên dụng (`x≈0.67, y≈0.26`), được quyết định gán `truck` (ID 2) thay vì `van`.
  3. `drive_009`: Xe buýt lớn công cộng đỏ trắng (`x≈0.66, y≈0.55`) bị che một phần bởi dòng xe, vẽ box bao quanh toàn bộ phần thân nhìn thấy được và gán `bus` (ID 3).
- **Phần cần giảng viên làm rõ:** Có sự không đồng nhất về thứ tự ID giữa tài liệu văn bản ban đầu (`0: car, 1: truck, 2: bus, 3: van`) và thứ tự thực tế trong ground truth `gold_train_labels` cùng tập `dataset/labels/eval/` (`0: car, 1: van, 2: truck, 3: bus`). Cần thống nhất quy ước chung cho các buổi lab sau.

## 3. Annotation và self-QC

- Số file nhãn train: 50 file (`drive_001.txt` đến `drive_050.txt`).
- Kết quả checker và lỗi đã xử lý: Chạy `python3 tools/check_yolo_labels.py` ĐẠT 50/50 file ngay từ đầu; tất cả tọa độ đều chuẩn hóa trong `[0, 1]`, không có box nào vượt ngoài biên ảnh.
- Danh sách 10 ảnh đã rà bằng mắt: `drive_001`, `drive_002`, `drive_005`, `drive_008`, `drive_009`, `drive_011`, `drive_012`, `drive_019`, `drive_026`, `drive_030`.
- Lỗi bỏ sót/box/class phát hiện khi rà và cách sửa: Ban đầu gán khá dư thừa các phương tiện ở tít chân trời phía xa và hàng xe đậu sát lề đường ngoài luồng giao thông; độ ôm sát box (tightness) rất tốt nhưng cần thu hẹp phạm vi vào vùng lưu thông chính (ROI).
- Commit khóa nhãn ban đầu: `3fa94ae`

## 4. Gold QC và rework

| Chỉ số | Trước sửa | Sau sửa (Rework) |
|---|---:|---:|
| TP | 383 | 497 |
| FP | 462 | 32 |
| FN | 270 | 156 |
| Precision | 0.4533 | 0.9395 |
| Recall | 0.5865 | 0.7611 |
| F1 | 0.5113 | 0.8409 |
| mean_iou_of_matches | 0.7172 | 0.7311 |
| passed | false | true |

Kết quả chi tiết được lưu tại `outputs/annotation_evaluation.json`.

**Phân tích nguyên nhân và quá trình Rework:**
- **Nguyên nhân chính dẫn đến FP/FN ban đầu:**
  1. *Lệch thứ tự class ID*: Export từ CVAT gán nhãn theo thứ tự khởi tạo khác với nhãn gold/eval của đề bài, khiến nhiều xe vẽ đúng vị trí nhưng bị lệch tên lớp (`bus` vs `van` vs `truck`), vừa mất TP vừa tăng cả FP lẫn FN.
  2. *Over-labeling (Gán thừa xe ngoài vùng ROI)*: Tập ảnh UA-DETRAC có vùng bỏ qua (*ignore region*) ở phía chân trời xa xăm ($y < 0.25$) và xe đậu sát hè. Sinh viên gán chi tiết toàn bộ các xe nhỏ này dẫn đến hơn 300 lỗi False Positive.
- **Biện pháp xử lý Rework:**
  - Chuẩn hóa ánh xạ lại ID 4 lớp đồng bộ hoàn toàn với ground truth và tập eval (`0: car, 1: van, 2: truck, 3: bus`).
  - Lọc bỏ các box vẽ thừa ở vùng chân trời/ngoài làn đường chính theo đúng quy định ROI.
  - Kết quả sau rework: **Precision tăng vọt từ 45.3% lên 93.95%**, **Recall đạt 76.11%**, **Mean IoU đạt 0.7311** (vượt ngưỡng 0.70) và **F1 đạt 0.8409** (chính thức ĐẠT ngưỡng $\ge 0.80$).

## 5. Kết quả YOLO

- Runtime, GPU/CPU, phiên bản Ultralytics và Torch: NVIDIA GeForce RTX 3050 Ti Laptop GPU (CUDA 12.4), PyTorch 2.6.0+cu124, Ultralytics 8.4.208 (được huấn luyện và nghiệm thu đồng nhất theo đúng cấu hình notebook của bài).
- Các tham số thay đổi so với notebook nếu có và lý do: Giữ nguyên hoàn toàn cấu hình chuẩn của bài (`epochs=50`, `imgsz=640`, `model=yolo26n.pt`, `seed=20260909`, `plots=True`, `conf=0.25`).
- Đường dẫn best checkpoint trong phiên chạy: `outputs/yolo26_runs/vehicle_annotation/weights/best.pt`

| Chỉ số trên 10 ảnh eval | Giá trị |
|---|---:|
| Precision | 0.5996 (59.96%) |
| Recall | 0.6011 (60.11%) |
| mAP50 | 0.5376 (53.76%) |
| mAP50–95 | 0.3016 (30.16%) |

*Đã giải nén và lưu đầy đủ 2 thư mục `assets/eval_predictions/` và `assets/yolo26_runs/` cạnh báo cáo.*

**Chi tiết chỉ số theo từng lớp:**
- `car`: 94 instances | P: 0.807 | R: 0.847 | mAP50: 0.891 | mAP50-95: 0.465
- `van`: 5 instances | P: 0.589 | R: 0.307 | mAP50: 0.328 | mAP50-95: 0.200
- `truck`: 2 instances | P: 0.435 | R: 0.500 | mAP50: 0.247 | mAP50-95: 0.148
- `bus`: 12 instances | P: 0.567 | R: 0.750 | mAP50: 0.683 | mAP50-95: 0.393

**Trả lời bốn câu hỏi cuối notebook:**

1. **Lớp nào recall thấp nhất? Ảnh nào minh họa?**
   - Lớp có Recall thấp nhất là **`van`** (Recall = 0.307, tức 30.7%, chỉ phát hiện được 2/5 xe van), tiếp theo là **`truck`** (Recall = 0.500). Minh họa rõ nét qua ảnh [drive_051.jpg](assets/eval_predictions/visuals/drive_051.jpg) (bỏ sót xe van nhỏ) hoặc [drive_055.jpg](assets/eval_predictions/visuals/drive_055.jpg). Nguyên nhân trực tiếp là do hiện tượng mất cân bằng dữ liệu nghiêm trọng (class imbalance): trong tập dữ liệu huấn luyện và đánh giá, số lượng mẫu `van` (5 xe) và `truck` (2 xe) quá ít so với `car` (94 xe con), khiến mô hình chưa đủ dữ liệu đa dạng để học khái quát đặc trưng nhận diện của van và truck.
2. **Nếu mAP50 cao nhưng mAP50–95 thấp, cần kiểm điều gì trên box?**
   - Cần kiểm tra lại **độ ôm sát (tightness) và độ chuẩn xác hình học của các cạnh bounding box**. Chỉ số mAP50 đo lường ở ngưỡng trùng khớp lỏng (IoU ≥ 0.50) nên dễ đạt điểm cao; tuy nhiên mAP50–95 tính trung bình trên các ngưỡng khắt khe từ 0.50 đến 0.95. Nếu box dự đoán chỉ hơi lệch mép, vẽ hơi rộng bao cả bóng đổ hoặc lấn sang mặt đường, IoU sẽ tụt xuống dưới 0.70 và bị coi là trượt ở các ngưỡng IoU cao, làm mAP50–95 giảm mạnh.
3. **Lỗi class nào có thể liên quan guideline? Dữ kiện nào hỗ trợ nhận xét?**
   - Lỗi nhầm lẫn phân loại giữa **`car` (SUV lớn) và `van`**, hoặc giữa **`van` và `truck` thùng kín**. Dữ kiện hỗ trợ trực tiếp là bảng ma trận nhầm lẫn tại [confusion_matrix.png](assets/yolo26_runs/vehicle_eval/confusion_matrix.png): góc nhìn camera giám sát giao thông từ trên cao (overhead CCTV) làm phẳng phối cảnh, khiến nóc xe và phần đuôi của dòng MPV/minivan nhìn rất giống SUV cỡ lớn nếu guideline không có quy ước phân biệt rõ về chiều cao thân xe và kết cấu cửa/kính.
4. **Vì sao không dùng 10 ảnh eval để train? Vai trò validation của chúng trong notebook là gì?**
   - Tuyệt đối không đưa 10 ảnh eval vào tập huấn luyện nhằm ngăn chặn hiện tượng rò rỉ dữ liệu (Data Leakage) và học vẹt (Overfitting). Vai trò của 10 ảnh eval trong notebook là tập Validation độc lập: cung cấp thước đo khách quan để theo dõi khả năng tổng quát hóa (generalization) của mạng nơ-ron qua 50 epoch, đồng thời làm căn cứ lựa chọn checkpoint tối ưu nhất (`best.pt`) dựa trên mAP của dữ liệu chưa từng thấy trong quá trình cập nhật trọng số gradient.

## 6. Khó khăn và cách xử lý

| Bước/thời điểm | Lỗi quan sát | Cách đã thử | Kết quả/còn thiếu |
|---|---|---|---|
| Khởi tạo CVAT | Thiếu đồng bộ thứ tự class với ground truth | Phân tích phân bố nhãn và cấu trúc box để phát hiện sai lệch ID | Chuẩn hóa ánh xạ lại thứ tự ID (car=0, van=1, truck=2, bus=3) |
| Chấm nhãn Gold QC | F1 ban đầu chỉ đạt 0.51 do quá nhiều False Positive | Rà soát các ảnh có FP cao, phát hiện quy ước ignore region cho xe nhỏ ở chân trời | Rework lọc bỏ các box ngoài vùng lưu thông chính, F1 tăng lên 0.8409 (ĐẠT) |
| Chạy huấn luyện | Google Colab gặp sự cố sập cấp phát runtime hệ thống | Tận dụng phần cứng GPU NVIDIA RTX 3050 Ti có sẵn trên máy để huấn luyện và đánh giá | Huấn luyện thành công 50 epoch, trích xuất đầy đủ 100% metrics và assets đánh giá |

## 7. Tự đánh giá theo rubric

- Hoàn thành và định dạng YOLO (10đ): ĐẠT 50/50 file nhãn hợp lệ, đúng cấu trúc 5 cột chuẩn hóa.
- Đầy đủ vật thể (20đ): Bao phủ tốt các phương tiện trên luồng giao thông chính, xử lý được xe bị che khuất và chạm mép ảnh.
- Độ bám sát và hình học box (20đ): Mean IoU đạt 0.7311 (vượt ngưỡng yêu cầu 0.70).
- Phân lớp nhất quán (15đ): Phân biệt rõ 4 lớp xe theo schema, không gán đối tượng ngoài phạm vi.
- Mini guideline và quyết định mơ hồ (10đ): Đầy đủ quy tắc box và 3 ca mơ hồ cụ thể có căn cứ.
- Gold-set annotation QC (15đ): Có kết quả đánh giá rõ ràng, vòng rework chi tiết đưa F1 từ 0.5113 lên 0.8409.
- Pipeline YOLO và phân tích (10đ): Chạy đầy đủ 50 epochs, lưu trữ đầy đủ toàn bộ assets ảnh dự đoán/đồ thị đánh giá và trả lời sâu sắc cả 4 câu hỏi phân tích.

## 8. Công cụ AI và tài liệu tham khảo

- Công cụ AI: Sử dụng Antigravity Coding Assistant để hỗ trợ phân tích ma trận nhầm lẫn nhãn, kiểm tra lỗi cú pháp và chạy script tự động hóa đánh giá đối chiếu với gold set. Học viên tự thực hiện gán nhãn thủ công 50 ảnh trên CVAT và kiểm duyệt nội dung báo cáo.
- Tài liệu tham khảo: Tài liệu hướng dẫn thực hành AI Thực chiến Ngày 2, chuẩn dữ liệu UA-DETRAC dataset.
