# Mini guideline — bài làm cá nhân

> Điền nội dung này trước khi bắt đầu gán nhãn. Giữ lại các quyết định sau khi self-QC.

Schema cố định: `0 car`, `1 truck`, `2 bus`, `3 van`. Quy trình: pre-label bằng YOLO26 (COCO), sau đó xem lại từng box trong CVAT (sửa lớp, xóa box sai, thêm xe bị bỏ sót).

## Định nghĩa lớp

| Lớp | Định nghĩa của bạn | Ví dụ/điểm phân biệt |
| --- | --- | --- |
| `car` | Xe con chở người: sedan, hatchback, SUV, taxi, pickup | Thân thấp, có nắp capo và cốp/thùng ngắn. Taxi vẫn là `car` dù có đèn mào. Pickup gán `car` theo schema. |
| `truck` | Xe chở hàng có cabin tách biệt với thùng hàng hoặc thùng kín cao | Cabin phía trước + thùng hàng/rơ-moóc phía sau; thường cao và dài hơn van. |
| `bus` | Xe chở khách cỡ lớn, thân dài liền khối | Nhiều cửa sổ hành khách xếp hàng dọc thân, cửa lên xuống bên hông, dài hơn rõ rệt so với van. |
| `van` | Xe thân hộp liền khối, cỡ giữa car và bus (van chở hàng, minivan thân hộp) | Không có capo dài như car, không có cabin tách rời như truck, ngắn hơn bus; mái cao, đuôi phẳng đứng. |

## Quy tắc chung

- Box bám sát vật thể nghĩa là: bốn cạnh box chạm đúng mép ngoài của phần thân xe nhìn thấy (gồm gương, cản), không lấy thêm bóng đổ, mặt đường hay xe bên cạnh. Mỗi xe một box; hai xe sát nhau vẽ hai box riêng.
- Vật thể bị che khuất: vẫn gán box quanh phần nhìn thấy nếu xác định được lớp; không vẽ phần đoán sau vật cản.
- Vật thể bị cắt bởi rìa ảnh: box dừng đúng ở rìa ảnh, không đoán phần ngoài khung. Chỉ gán khi phần còn lại đủ để nhận diện lớp.
- Vật thể rất nhỏ/mơ hồ: không gán mảnh xe quá nhỏ hoặc quá mờ ở xa mà không xác định chắc lớp; không tự đoán. Box pre-label của model trên các mảnh như vậy đã được xóa khi review.
- Ngoài scope (không gán): mô tô/xe máy, xe đạp, người đi bộ, biển báo, phản chiếu, hình xe trên bảng quảng cáo.

## Ghi lại quyết định cho ít nhất ba ca mơ hồ

| Ảnh | Tình huống | Quyết định cuối | Lý do theo schema |
| --- | --- | --- | --- |
| `drive_001` | Xe taxi (có đèn mào trên nóc) | `car` | Taxi là xe con chở người; đèn mào không đổi loại thân xe. |
| `drive_003` | Xe màu trắng bị cắt ở rìa phải ảnh (dưới xe bus), có thể là truck hoặc van | `truck` | Sau khi xem lại, phần nhìn thấy được nhận định là truck; box dừng đúng ở rìa ảnh, không đoán phần ngoài khung. |
| `drive_005`, `drive_041` | Model đề xuất box `truck` cao, hẹp sát rìa ảnh | Xóa box, không gán | Khi xem lại, đó không phải truck và không đủ phần nhìn thấy để xác định lớp; không tự đoán. |
| `drive_007` | Xe thân hộp, mái cao, đuôi đứng | `van` | Không có cabin tách rời (không phải truck), ngắn hơn bus, không có capo dài như car. |
| `drive_009`, `drive_050` | Xe nghi là van khi lướt nhanh | Không gán `van`; gán theo loại thân xe thực tế | Xem kỹ thì không có xe nào có thân hộp liền khối của van. |
| Toàn bộ tập | Pickup | `car` | Schema quy định pickup thuộc `car`. |
