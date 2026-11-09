# Báo cáo cá nhân — Ngày 2 Object Detection

## 1. Thông tin học viên

- Họ tên: Nguyen Minh Nam Hai
- MSSV: 2A202603076
- Email: namhainguyenminh@gmail.com
- Lớp: K5-L2A
- Link repo bài làm: https://github.com/namhai923/K5-L2A-DAY02-NGUYENMINHNAMHAI-2A202603076-ObjectDetection
- Link Colab đã chạy: https://colab.research.google.com/drive/1cItk5-Hcm5p2qnCk2o-Sdc9w2G-WHZ78

## 2. Schema và mini guideline

Chi tiết ở [GUIDELINE_MINI.md](../../GUIDELINE_MINI.md). Schema giữ nguyên `0 car`, `1 truck`, `2 bus`, `3 van`; pickup và taxi gán `car`; mô tô, xe đạp, người đi bộ, phản chiếu, hình xe quảng cáo không gán.

Ba ca mơ hồ chính:

- `drive_001`: taxi có đèn mào → `car` (loại thân xe là xe con).
- `drive_003`: xe trắng bị cắt ở rìa phải ảnh, dưới xe bus, có thể là truck hoặc van → `truck`; box dừng ở rìa ảnh.
- `drive_005`, `drive_041`: model đề xuất box `truck` cao, hẹp sát rìa ảnh → xóa, vì không phải truck và không đủ phần nhìn thấy để xác định lớp.
- Thêm: `drive_007` → `van` (thân hộp, mái cao, không có cabin tách rời); `drive_009`, `drive_050` → không có van.

Cần giảng viên làm rõ:

- Thứ tự class ID trong gold có vẻ khác schema của README/`data.yaml`.
- Xe đỗ và xe nhỏ ở xa: GUIDE yêu cầu gán xe đỗ, nhưng gold có vẻ không gán nhiều xe này.

## 3. Annotation và self-QC

- Số file nhãn train: 50 file, 721 box (569 car, 26 truck, 88 bus, 38 van) sau rework.
- Quy trình gán nhãn: pre-label bằng YOLO26x pretrained COCO (car/truck/bus; COCO không có van), import vào CVAT, rồi xem lại từng ảnh: sửa lớp, xóa box sai/trùng, thêm xe bị bỏ sót, export YOLO.
- Kết quả checker và lỗi đã xử lý: `check_yolo_labels.py` đạt `50/50 file nhãn hợp lệ`.
- Danh sách ảnh đã rà bằng mắt: rà cả 50 ảnh trên CVAT; kiểm kỹ `drive_001`, `drive_003`, `drive_004`, `drive_005`, `drive_007`, `drive_009`, `drive_014`, `drive_024`, `drive_041`, `drive_050`.
- Lỗi phát hiện khi rà và cách sửa:
  - 18 cặp box trùng (model gán hai lớp cho cùng một xe) → xóa box thừa.
  - Box `truck` cao hẹp ở rìa `drive_005`, `drive_041` và box sai ở `drive_004` → xóa.
  - Van bị model gán `car`/`truck`/`bus` → sửa thành `van` (ví dụ `drive_007`).
  - Pickup → `car`; taxi `drive_001` → `car`.
  - 8 box rất nhỏ không chắc lớp → xóa.
- Commit khóa nhãn ban đầu: `8e50dfd` ("train labels before gold", 1165 box); bản sau rework: `c8b6475` (721 box).

## 4. Gold QC và rework

| Chỉ số | Trước sửa (`8e50dfd`) | Sau sửa (`c8b6475`) |
|---|---:|---:|
| TP | 477 | 385 |
| FP | 688 | 336 |
| FN | 176 | 268 |
| Precision | 0.4094 | 0.5340 |
| Recall | 0.7305 | 0.5896 |
| F1 | 0.5248 | 0.5604 |
| mean_iou_of_matches | 0.8478 | 0.8633 |
| passed | False (gate F1 0.80) | True (gate F1 0.50 theo cập nhật của giảng viên) |

Kết quả sau sửa: [outputs/annotation_evaluation.json](../../outputs/annotation_evaluation.json). Kết quả trước sửa được lưu riêng ở `outputs/annotation_evaluation_v1_before_rework.json`.

Rework:

- **Lỗi đã kiểm:**
  - Mean IoU đạt ngay từ đầu (0.85), nên box đã khớp là sát; vấn đề chính là precision thấp (688 FP).
  - Đã vẽ vị trí FP/FN lên ảnh để xem: phần lớn FP là xe đỗ và xe nhỏ ở xa mà gold không gán. Phần lớn số này đến từ lượt bổ sung xe bị bỏ sót (thêm 444 box).
- **Quyết định sửa:** gỡ 444 box của lượt bổ sung, quay về bộ nhãn đã review từ pre-label (721 box). Precision tăng 0.41 → 0.53, recall giảm 0.73 → 0.59, F1 tăng 0.525 → 0.560.
- **Quyết định giữ:**
  - Giữ các xe đỗ hiện có, vì GUIDE quy định "xe đang đỗ vẫn gán". Thử xóa 83 box xe đỗ thì F1 lên 0.589, nhưng không áp dụng vì trái guideline.
- **Quan sát về class ID:**
  - Các box đã khớp vị trí cho thấy gold dùng thứ tự `0 car, 1 van, 2 truck, 3 bus`, khác schema `0 car, 1 truck, 2 bus, 3 van`. Box `bus` của tôi trùng vị trí với gold class 3, `van` trùng gold class 1, `truck` trùng gold class 2.
  - Vì evaluator ghép theo class, các box này bị tính FP + FN dù vị trí đúng.
  - Tôi giữ nhãn theo schema của README/`data.yaml`, không đổi ID khi chưa có xác nhận của giảng viên.
  - Nhãn `dataset/labels/eval/` có vẻ cùng thứ tự với gold, có thể ảnh hưởng metric theo lớp ở mục 5.

## 5. Kết quả YOLO

- Runtime, GPU/CPU, phiên bản Ultralytics và Torch: Google Colab GPU Tesla T4 (15 GB), Ultralytics 8.4.208, Python 3.13.16, torch 2.14.1+cu130. Train 50 epoch hết 0.024 giờ (~1.5 phút).
- Commit được clone: `c8b6475` (nhãn sau rework, 721 box).
- Các tham số thay đổi so với notebook nếu có và lý do: không đổi (`yolo26n.pt`, 50 epoch, imgsz 640, `batch=-1`, seed 20260909).
- Đường dẫn best checkpoint trong phiên chạy: `/content/Day2-Lab/outputs/yolo26_runs/vehicle_annotation/weights/best.pt`

| Chỉ số trên 10 ảnh eval | Giá trị |
|---|---:|
| Precision | 0.4906 |
| Recall | 0.3063 |
| mAP50 | 0.3255 |
| mAP50–95 | 0.2082 |

Theo từng lớp (tên lớp theo `data.yaml`, khi val `best.pt`):

| Lớp (`data.yaml`) | Instances | P | R | mAP50 | mAP50–95 |
|---|---:|---:|---:|---:|---:|
| car (0) | 94 | 0.363 | 0.809 | 0.674 | 0.439 |
| truck (1) | 5 | 1.000 | 0.000 | 0.243 | 0.175 |
| bus (2) | 2 | 0.000 | 0.000 | 0.009 | 0.005 |
| van (3) | 12 | 0.599 | 0.417 | 0.376 | 0.213 |

Ảnh dự đoán: [assets/eval_predictions/visuals/](assets/eval_predictions/visuals/); log train và ảnh chỉ số: [assets/yolo26_runs/](assets/yolo26_runs/).

Khi predict với `conf=0.25`, model chỉ ra `car` trên cả 10 ảnh, cộng một box `bus` ở `drive_058` (32 car + 1 bus). Model gần như chỉ học được `car`.

**Nhận xét quan trọng — thứ tự class ID của nhãn eval.** Khi vẽ nhãn `dataset/labels/eval/` lên ảnh (chỉ để xem, không train):

- class `3` (tên `van` trong `data.yaml`) nằm trên các xe **bus**. Ví dụ: `drive_058` có 6 box class 3 đều là xe buýt; `drive_053` có xe buýt xanh lá lớn.
- class `2` (tên `bus`) nằm trên **truck**: xe tải thùng ở `drive_051`, xe tải trắng ở `drive_058`.
- class `1` (tên `truck`) nằm trên **van**: xe van trắng ở `drive_059`.

Như vậy nhãn eval dùng thứ tự `0 car, 1 van, 2 truck, 3 bus`, giống gold train (mục 4), còn nhãn train của tôi theo `data.yaml`. Metric theo lớp truck/bus/van ở bảng trên vì thế đang so hai lớp khác nhau. Chỉ số `car` (cùng ID 0) và chỉ số `all` của car là đáng tin.

Trả lời bốn câu hỏi cuối notebook:

1. **Lớp nào recall thấp nhất? Ảnh nào minh họa?**
   - Theo bảng, `truck` (R = 0, 5 instances) và `bus` (R = 0, 2 instances) thấp nhất.
   - Ảnh minh họa: `drive_059`. Nhãn eval có một xe van trắng ở xa, mang class 1, nên được tính là "truck". Model chỉ dự đoán 8 `car` trên ảnh này, không có box nào khác.
   - Nguyên nhân kép:
     - Lệch class ID: class 1 trong eval là van, nhưng model học class 1 là truck.
     - Dữ liệu train quá ít cho lớp hiếm: 26 truck, 88 bus, 38 van so với 569 car.
     - Kết quả: ở `conf=0.25` model gần như chỉ dự đoán `car`.
2. **Nếu mAP50 cao nhưng mAP50–95 thấp, cần kiểm điều gì trên box?**
   - Box khớp vị trí thô (IoU ≥ 0.5) nhưng không sát ở các ngưỡng IoU cao.
   - Ở đây `car` có mAP50 0.674 nhưng mAP50–95 chỉ 0.439. Cần kiểm:
     - cạnh box lỏng hoặc lấn nền/bóng đổ;
     - box xe bị che/cắt rìa không nhất quán (lúc ôm phần nhìn thấy, lúc đoán cả xe);
     - xe nhỏ ở xa, nơi lệch vài pixel đã giảm IoU mạnh.
   - Nhãn train có mean IoU 0.86 với gold, nên phần lớn chênh lệch nhiều khả năng do model nano + 50 ảnh, không phải do box lỏng.
3. **Lỗi class nào có thể liên quan guideline? Dữ kiện nào hỗ trợ nhận xét?**
   - Lỗi lớn nhất là thứ tự class ID: schema trong README/`data.yaml` (`1 truck, 2 bus, 3 van`) khác nhãn gold/eval (`1 van, 2 truck, 3 bus`).
     - Dữ kiện: nhãn eval class 3 nằm trên xe buýt (`drive_058`, `drive_053`).
     - Lớp `van` có R 0.417: model "bắt" được một phần xe buýt eval dưới tên van. Đây là trùng hợp vị trí xe lớn, không phải nhận đúng lớp.
     - Gold QC cũng cho thấy box bus/van/truck của tôi khớp vị trí với gold nhưng khác ID.
   - Thứ hai, ranh giới `van` với `car`/`bus` mơ hồ (GUIDE yêu cầu không tự đoán), và lớp van chỉ có 38 box train. Model khó tách van khỏi car.
   - Thứ ba, guideline gán cả xe đỗ, gold thì không. Điều này làm FP khi QC, nhưng không ảnh hưởng tên lớp.
4. **Vì sao không dùng 10 ảnh eval để train? Vai trò validation của chúng trong notebook là gì?**
   - Nếu train trên eval, model đã thấy chính các ảnh dùng để đo. Metric sẽ bị thổi phồng và không phản ánh khả năng tổng quát.
   - Trong notebook, `data.yaml` dùng eval làm `val`: tính metric mỗi epoch và chọn `best.pt`.
   - Vì eval đã tham gia chọn checkpoint, nó là validation set, không phải test set độc lập.

## 6. Khó khăn và cách xử lý

| Bước/thời điểm | Lỗi quan sát | Cách đã thử | Kết quả/còn thiếu |
|---|---|---|---|
| Pre-label | COCO không có lớp `van`; model gán hai lớp cho một xe | Rà từng ảnh trong CVAT, sửa lớp van thủ công, xóa 18 box trùng | Đã sửa |
| Gold QC lần 1 | F1 0.525 < 0.80, precision 0.41 | Vẽ FP/FN lên ảnh; gỡ 444 box của lượt bổ sung | F1 0.560, đạt gate 0.50 mới |
| Gold QC / Colab | Thứ tự class ID của gold và nhãn eval khác schema | Giữ nhãn theo schema; vẽ nhãn eval lên ảnh để xác nhận (class 3 = bus, 2 = truck, 1 = van) | Metric theo lớp truck/bus/van không so đúng lớp |
| Colab | Model chỉ dự đoán `car` ở `conf=0.25` | Đối chiếu số box train theo lớp (569 car so với 26–88 cho lớp khác) | Mất cân bằng lớp + lệch ID; cần thêm dữ liệu lớp hiếm |
| Gold QC | Gold không gán xe đỗ, trái GUIDE | Thử xóa xe đỗ (F1 0.589) nhưng không áp dụng | Giữ xe đỗ theo GUIDE |

## 7. Tự đánh giá theo rubric

Đối chiếu [RUBRIC.md](../../RUBRIC.md):

- **Định dạng YOLO (10):** 50 file, checker đạt.
- **Đầy đủ vật thể (20) / độ bám sát (20) / phân lớp (15):** mean IoU 0.86 cho thấy box sát. Recall 0.59 so với gold, một phần do thứ tự class ID khác nhau. Sau rework, một số xe nhỏ ở xa không còn được gán.
- **Mini guideline (10):** có định nghĩa 4 lớp, quy tắc box và 6 ca mơ hồ.
- **Gold QC (15):** có JSON trước/sau; đạt gate F1 0.50 đã cập nhật (chưa đạt 0.80 gốc); vòng rework được giải thích ở mục 4.
- **Pipeline YOLO (10):** Colab đã chạy trên commit `c8b6475`, không dùng eval để train. Có metrics và phân tích lỗi (mục 5). Ảnh eval nằm trong `assets/` sau khi giải nén `day2_colab_outputs.zip`.

## 8. Công cụ AI và tài liệu tham khảo

- **YOLO26x (Ultralytics, pretrained COCO):** pre-label 50 ảnh train (`tools/prelabel.py`). Mọi box đều được xem lại và sửa trên CVAT. Không dùng gold hay ảnh eval khi pre-label.
- **CVAT (chạy local):** xem lại, sửa và export nhãn YOLO.
- **Claude Code (Claude, Anthropic):**
  - Hỗ trợ viết script pre-label và sửa nhãn qua CVAT REST API theo các quyết định do tôi duyệt.
  - Hỗ trợ phân tích FP/FN và soạn guideline/báo cáo.
  - Tôi đã tự kiểm tra các quyết định lớp trên ảnh (`drive_001`, `drive_003`, `drive_004`, `drive_005`, `drive_007`, `drive_009`, `drive_041`, `drive_050`).
- **Nguồn ngoài repo:** tài liệu Ultralytics YOLO và CVAT.
