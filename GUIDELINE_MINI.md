# Mini guideline — bài làm cá nhân

> Điền nội dung này trước khi bắt đầu gán nhãn. Giữ lại các quyết định sau khi self-QC.

## Định nghĩa lớp

| Lớp | Định nghĩa của bạn | Ví dụ/điểm phân biệt |
| --- | --- | --- |
| `car` | Xe con, sedan, SUV, taxi, hatchback, xe bán tải (pickup). | Phương tiện cá nhân, chở ít người, gầm thấp hoặc bán tải cabin kép có thùng hở nhỏ. |
| `van` | Xe van, minivan, xe 16 chỗ chở khách/hàng (Ford Transit, Toyota Hiace, MPV). | Thân xe dạng hộp liền khối, trần cao hơn car thông thường nhưng nhỏ hơn xe buýt công cộng. |
| `truck` | Xe tải chở hàng, xe tải thùng kín/thùng lửng, xe đầu kéo container. | Có kết cấu thùng chở hàng thương mại rõ rệt phía sau, phân biệt với xe con và xe bán tải nhỏ. |
| `bus` | Xe buýt công cộng đô thị, xe khách du lịch cỡ lớn. | Xe thân dài nhiều hàng ghế hoặc nhiều cửa lên xuống, kích thước vượt trội hoàn toàn so với minivan. |

## Quy tắc chung

- Box bám sát vật thể nghĩa là: Bounding box ôm khít các cạnh ngoài cùng của phần thân xe thực sự nhìn thấy được, không bao gồm bóng đổ xe, mặt đường hay khoảng trống xung quanh.
- Vật thể bị che khuất: Chỉ vẽ box bao quanh phần thân xe còn nhìn thấy được (visible region), miễn là còn đủ chi tiết để xác định chắc chắn loại phương tiện; không đoán phần bị khuất hoàn toàn.
- Vật thể bị cắt bởi rìa ảnh: Kéo cạnh box chạm đúng mép biên của khung hình; không vẽ box vượt ra ngoài tọa độ [0, 1].
- Vật thể rất nhỏ/mơ hồ: Các phương tiện ở quá xa tại vùng đường chân trời hoặc cầu vượt trên cao (chiều cao < 20 pixel, mờ nhòe) và xe đậu cố định ngoài luồng lưu thông chính được coi là vùng bỏ qua (ignore region) theo chuẩn tập UA-DETRAC, không gán nhãn để tránh gây nhiễu cho mô hình phát hiện vật thể.

## Ghi lại quyết định cho ít nhất ba ca mơ hồ

| Ảnh | Tình huống | Quyết định cuối | Lý do theo schema |
| --- | --- | --- | --- |
| `drive_002` | Xe màu xám/trắng ở làn giữa (`x≈0.37, y≈0.43`), dễ nhầm giữa SUV cỡ lớn (`car`) và xe `van`. | Gán `van` (class ID 1) | Thân xe dạng hình hộp, trần cao thẳng đứng đặc trưng của dòng MPV/minivan, thể tích khoang lớn hơn SUV thông thường. |
| `drive_005` | Phương tiện có cabin nhỏ nhưng phía sau gắn thùng tải kín chở hàng màu xanh (`x≈0.67, y≈0.26`), phân vân giữa `van` và `truck`. | Gán `truck` (class ID 2) | Có thùng chở hàng thương mại chuyên dụng phía sau tách biệt với cabin, thuộc nhóm xe tải chở hàng theo định nghĩa schema. |
| `drive_009` | Xe buýt cỡ lớn màu đỏ trắng (`x≈0.66, y≈0.55`) đi trên làn chính, bị xe khác che khuất một phần đầu xe. | Gán `bus` (class ID 3) | Xe buýt công cộng kích thước lớn với dải kính hành khách dài, vẽ box ôm trọn phần thân nhìn thấy được; phân biệt rõ với xe minibus/van. |
