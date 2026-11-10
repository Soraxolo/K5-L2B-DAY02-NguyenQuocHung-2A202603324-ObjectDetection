# Báo cáo cá nhân — Ngày 2 Object Detection

## 1. Thông tin học viên

- Họ tên: Nguyễn Quốc Hưng
- MSSV: 2A202603324
- Email: sora.12082004@gmail.com
- Lớp: K5 — Level 2B (L2B)
- Link repo bài làm: https://github.com/Soraxolo/K5-L2B-DAY02-NguyenQuocHung-2A202603324-ObjectDetection
- Link Colab đã chạy: https://colab.research.google.com/drive/1PWrqQz0ZR8BsgH59yBVzpupsaMoXXflj?usp=sharing

## 2. Schema và mini guideline

Guideline cá nhân: [GUIDELINE_MINI.md](../../GUIDELINE_MINI.md).

Tiêu chí phân lớp tôi dùng xoay quanh hai câu hỏi hình học, áp dụng được cho mọi ảnh: **đường nóc có tụt
dốc về đuôi không** (tách `car` khỏi `van`) và **cabin có tách rời thùng hàng không** (tách `truck` khỏi
`van`). `bus` nhận diện bằng dãy cửa sổ hành khách liên tục và thân dài trên 2,5 lần chiều cao.

Ba ca mơ hồ đã quyết định:

| Ảnh | Tình huống | Quyết định | Căn cứ |
| --- | --- | --- | --- |
| `drive_008`, `drive_022` | Xe buýt khớp nối hai khoang | Một box bao cả hai khoang | Schema phân theo loại vật thể; hai khoang chung một buồng lái, không tách rời khi vận hành nên là một xe |
| `drive_022` | Xe thùng trắng nhỏ, thân cao giống `van` | `truck` | Phóng to thấy khe và bậc giữa nóc cabin với nóc thùng, thùng đặt trên khung gầm — `van` phải là thân liền khối |
| `drive_035` | Xe đỗ sau nhà chờ xe buýt, bị che nhiều mức khác nhau | Gán chiếc còn nhận ra hình dáng, bỏ chiếc chỉ ló vệt nóc | Mức che quyết định chứ không phải việc có bị che; không đủ dữ kiện phân biệt `car` với `van` thì không đoán |

Sau vòng QC với gold, tôi bổ sung một quy tắc nữa vào guideline: **không gán xe có cạnh ngắn dưới 20 px**,
cùng lý lẽ với ca `drive_035` — ở cỡ đó không đọc được đường nóc hay cabin.

Điểm cần giảng viên làm rõ: quy ước class ID giữa `data.yaml` và nhãn gold không khớp (chi tiết ở mục 4).

## 3. Annotation và self-QC

- Số file nhãn train: **50/50**, cùng basename với ảnh, định dạng YOLO 5 cột.
- Kết quả checker: `python3 tools/check_yolo_labels.py` → `ĐẠT cấu trúc: 50/50 file nhãn hợp lệ.`
- Công cụ: CVAT 2.74.1 chạy cục bộ, task `Day2-train`, 4 label kiểu `rectangle`, Image quality 100.
  Export `Ultralytics YOLO Detection 1.0`, sau đó ánh xạ class **theo tên** (không theo thứ tự) khi đưa về
  `dataset/labels/train/` để tránh lệch ID.
- Rà bằng mắt: dựng ảnh overlay (vẽ box lên ảnh gốc, phóng 2×) cho cả 50 ảnh và soi kỹ 10 ảnh:
  `drive_001`, `drive_002`, `drive_006`, `drive_007`, `drive_008`, `drive_009`, `drive_014`, `drive_022`,
  `drive_030`, `drive_038`.
- Lỗi tự phát hiện và xử lý:
  - 3 box suy biến do click nhầm (`drive_006` 1,1×2,1 px; `drive_007` 1,3×1,5 px; `drive_038` 4,3×7,3 px) — xóa.
  - 55 box có cạnh ngắn dưới 10 px, 206 box dưới 15 px: so với nhãn gold của 10 ảnh eval (không có box nào
    dưới 10 px, chỉ 2 box dưới 15 px) thì đây là dấu hiệu tôi gán quá tay ở vùng xe rất nhỏ → dẫn tới quy tắc
    ngưỡng 20 px.
  - Rà lại nhóm xe MPV ở `drive_030` để thống nhất ranh giới `car`/`van`.
- Commit khóa nhãn ban đầu: **`6522923`** — *CP4: lock initial annotation (50 YOLO label files) + mini guideline*.

## 4. Gold QC và rework

Chạy `tools/evaluate_annotations.py` với `gold_train_labels/` do giảng viên phát, kết quả lưu ở
[`outputs/annotation_evaluation.json`](../../outputs/annotation_evaluation.json) (bản trước rework giữ ở
`outputs/annotation_evaluation_truoc_rework.json`).

| Chỉ số | Trước sửa | Sau sửa |
|---|---:|---:|
| TP | 468 | 449 |
| FP | 785 | 430 |
| FN | 185 | 204 |
| Precision | 0,3735 | 0,5108 |
| Recall | 0,7167 | 0,6876 |
| F1 | 0,4911 | 0,5862 |
| mean_iou_of_matches | 0,8338 | 0,8370 |
| passed | false | false |

**Nội dung rework:** áp quy tắc ngưỡng 20 px đã ghi trong guideline, gỡ 374 box (1253 → 879). Precision tăng
0,37 lên 0,51; recall giảm nhẹ vì một phần xe nhỏ trong gold cũng bị loại theo. F1 chưa đạt ngưỡng 0,80.

**Phát hiện quan trọng: nhãn gold đánh class ID theo thứ tự khác `data.yaml`.**

`data.yaml` và bảng schema trong README quy định `car=0, truck=1, bus=2, van=3`. Nhãn gold — cả
`gold_train_labels/` lẫn `dataset/labels/eval/` — thực tế dùng `car=0, van=1, truck=2, bus=3`. Ba lớp bằng
chứng độc lập:

1. **Crop trực tiếp nhãn eval có sẵn trong repo**: toàn bộ box class 3 là xe buýt thành phố và buýt hai
   tầng; class 2 là xe tải và xe cẩu; class 1 là microvan. Nếu đọc theo `data.yaml` thì 12 "van" kia đều là
   xe buýt.
2. **Dò toàn bộ 24 hoán vị class** khi chấm nhãn của tôi với gold train: hoán vị `car→0, truck→2, bus→3,
   van→1` cho F1 cao nhất (0,721 so với 0,586 khi giữ nguyên), và FN giảm từ 204 xuống 101.
3. **Chỉ số per-class khi train** (mục 5): `car` — lớp duy nhất có ID trùng nhau giữa hai quy ước — đạt
   mAP50 0,738, trong khi `bus` đúng bằng 0,000.

Chấm lại cùng bộ nhãn theo quy ước của gold: **P 0,628 · R 0,845 · F1 0,721 · mean IoU 0,844**, và số box
sai lớp thật chỉ còn **12 trên 879** (chủ yếu ở ranh giới `car`/`van` của xe MPV). Tôi **không** đổi ID
trong bài nộp vì `data.yaml` mới là schema chính thức và model train theo thứ tự đó; thay vào đó báo giảng
viên để lớp thống nhất.

**Về phần FP còn lại:** tôi lấy mẫu 40 trong số 177 box bị tính FP có cạnh ngắn từ 28 px trở lên và kiểm tra
bằng mắt — gần như tất cả là xe thật thuộc schema (ô tô con, xe buýt, microvan, xe tải nhìn rõ). Gold cũng
thưa hơn đáng kể: 13,1 box/ảnh so với 17,6 box/ảnh của tôi sau rework. Nên một phần lớn FP phản ánh việc
gold bỏ sót chứ không phải tôi vẽ thừa. Tôi giữ nguyên các box này: xóa chúng sẽ làm đẹp F1 nhưng làm bộ
nhãn kém đúng hơn so với schema và guideline.

Không sửa gold, không viết lại lịch sử commit khóa nhãn.

## 5. Kết quả YOLO

- Runtime: Google Colab, GPU **Tesla T4** (CUDA:0, 14913 MiB)
- Phiên bản: Python 3.13.16 / torch 2.14.1+cu130 / Ultralytics 8.4.208
- Tham số: giữ nguyên notebook — `yolo26n.pt`, 50 epoch, `imgsz=640`, `batch=-1` (auto), `seed=20260909`
- Best checkpoint trong phiên chạy: `outputs/yolo26_runs/vehicle_annotation/weights/best.pt` (epoch 26;
  file `.pt` không commit theo quy định)

| Chỉ số trên 10 ảnh eval | Giá trị |
|---|---:|
| Precision | 0,3156 |
| Recall | 0,3066 |
| mAP50 | 0,2740 |
| mAP50–95 | 0,2006 |

Per-class:

| Lớp | Instances | P | R | mAP50 | mAP50–95 |
| --- | ---: | ---: | ---: | ---: | ---: |
| `car` | 94 | 0,629 | 0,777 | 0,738 | 0,521 |
| `truck` | 5 | 0,391 | 0,200 | 0,235 | 0,203 |
| `bus` | 2 | 0,000 | 0,000 | 0,008 | 0,003 |
| `van` | 12 | 0,243 | 0,250 | 0,115 | 0,075 |

Evidence: [ảnh prediction trên 10 ảnh eval](assets/eval_predictions/visuals/),
[log train và biểu đồ](assets/yolo26_runs/vehicle_annotation/) (`results.csv`, `results.png`),
[chỉ số đánh giá](assets/yolo26_runs/vehicle_eval/) (confusion matrix, PR curve).

### Trả lời bốn câu hỏi

**1. Lớp nào recall thấp nhất? Ảnh nào minh họa?**
`bus` với recall 0,000, kế đến là `truck` 0,200. Ảnh minh họa:
`assets/eval_predictions/visuals/drive_053.jpg` — model khoanh chiếc xe buýt xanh ở giữa ảnh và dự đoán
`bus 0.85`, tức nhận dạng hoàn toàn đúng. Nhưng trong nhãn eval, chính chiếc xe đó mang **id 3**, mà
`data.yaml` gọi id 3 là `van`; còn hai instance được tính là `bus` (id 2) trong 10 ảnh eval thực chất là xe
tải. Model không dự đoán `bus` ở vị trí hai chiếc xe tải đó, nên recall của `bus` bằng 0 dù nó bắt đúng mọi
xe buýt trong ảnh. Recall 0 ở đây đo sự lệch tên lớp, không đo khả năng của model. Nếu chỉ nhìn `car` — lớp duy nhất không bị ảnh hưởng — recall là
0,777.

**2. Nếu mAP50 cao nhưng mAP50–95 thấp thì box có vấn đề gì?**
Nghĩa là model tìm đúng vật nhưng cạnh box chưa khớp sát: ở ngưỡng IoU 0,50 vẫn tính là trúng, nhưng lên
IoU 0,75–0,95 thì trượt. Cần kiểm tra box có bị nới rộng ra nền, có bao thêm bóng đổ hay phần xe bên cạnh
không, và biên của vật bị che/cắt khung đặt đúng chưa. Trong bài này `car` có mAP50 0,738 và mAP50–95 0,521
— chênh lệch đúng kiểu đó, hợp lý với dữ liệu CCTV nhiều xe nhỏ, nơi lệch vài pixel đã làm IoU tụt mạnh.

**3. Lỗi class nào liên quan tới guideline?**
Sau khi quy về cùng quy ước ID, lỗi lớp thật chỉ còn 12 box trên 879, toàn bộ ở ranh giới `car`/`van` — đúng
chỗ guideline của tôi dựa vào đường nóc, mà xe hộp cao cỡ nhỏ (kiểu Wagon R) lại vừa có capo của `car` vừa
có nóc phẳng của `van`. Đây là chỗ guideline cần một câu bổ sung: xe con thân hộp dưới 5 chỗ, có capo, xếp
`car`. Còn các lệch lớp quy mô lớn (`bus`→`van`, `van`→`truck`, `truck`→`bus`) **không** bắt nguồn từ
guideline mà từ việc nhãn gold xoay vòng ID, như đã chứng minh ở mục 4.

**4. Vì sao không dùng 10 ảnh eval để train?**
Vì notebook dùng chính tập đó làm validation để chọn `best.pt`. Train trên nó thì model đã nhìn thấy đáp án,
mọi chỉ số đo được sẽ là đo trí nhớ chứ không phải khả năng tổng quát hóa, và việc chọn checkpoint cũng mất
ý nghĩa. Cần nói rõ: 10 ảnh này là **validation**, không phải test set độc lập — chúng đã tham gia vào việc
chọn checkpoint nên con số ở trên vẫn lạc quan hơn thực tế.

## 6. Khó khăn và cách xử lý

| Bước/thời điểm | Lỗi quan sát | Cách đã thử | Kết quả/còn thiếu |
|---|---|---|---|
| Tạo task CVAT | Dán JSON label báo `unknown label type "undefined"`, task tạo ra không có label nào nên không vẽ được | Thêm trường `"type": "rectangle"` cho cả 4 label rồi lưu lại ở tab Raw của task | Vẽ được bình thường; đặt `type` còn khóa công cụ về bounding box |
| Self-QC | Không rõ box nào lỏng hay bỏ sót khi chỉ nhìn file `.txt` | Dựng ảnh overlay cho cả 50 ảnh, thống kê kích thước box và so với nhãn gold eval | Phát hiện 3 box suy biến và xu hướng gán quá tay ở xe rất nhỏ |
| Gold QC | F1 0,491 trong khi mean IoU 0,834 — mâu thuẫn: box rất sát nhưng điểm rất thấp | Tách lỗi thành "thừa box" và "sai lớp"; thử toàn bộ 24 hoán vị class | Tìm ra nhãn gold dùng thứ tự ID khác `data.yaml` |
| Train Colab | mAP tổng thấp, `bus` bằng 0 | Đọc chỉ số per-class và đối chiếu số instance của từng lớp trong nhãn eval | Khẳng định nguyên nhân là lệch quy ước ID, không phải model hay annotation |

## 7. Tự đánh giá theo rubric

| Tiêu chí | Tự đánh giá |
| --- | --- |
| Hoàn thành và định dạng YOLO | Đủ 50 file, checker đạt, 5 cột normalized, class ID theo `data.yaml` |
| Đầy đủ vật thể | Recall 0,845 khi chấm theo quy ước của gold; phần FP đã kiểm mẫu cho thấy nhiều xe thật gold bỏ sót |
| Độ bám sát và hình học box | mean_iou_of_matches 0,844 — mức "xuất sắc" theo bảng chất lượng |
| Phân lớp nhất quán | 12/879 box lệch lớp sau khi quy về cùng quy ước, tập trung ở ranh giới `car`/`van` |
| Mini guideline và quyết định mơ hồ | Có định nghĩa 4 lớp, quy tắc box, 3 ca mơ hồ kèm lý do, cộng quy tắc ngưỡng kích thước bổ sung sau QC |
| Gold-set annotation QC | F1 chưa đạt 0,80 theo `data.yaml` (0,586) nhưng đạt 0,721 theo quy ước của gold; có một vòng rework với số liệu trước/sau và lý do giữ lại phần FP |
| Pipeline YOLO và phân tích | Colab chạy xong, giữ nguyên split, có metrics + ảnh eval + phân tích nguyên nhân per-class |

Chưa kiểm được: chỉ số thật của `truck`/`bus`/`van` — phải chờ giảng viên chốt quy ước ID rồi chạy lại mới
có con số đáng tin.

## 8. Công cụ AI và tài liệu tham khảo

Có dùng **Claude Code (Anthropic)** trong bài này, ở các bước sau:

- Dựng task CVAT (sinh JSON 4 label), chuyển export CVAT về đúng layout `dataset/labels/train/` bằng script
  ánh xạ class theo tên, chạy `check_yolo_labels.py` và `evaluate_annotations.py`, thao tác git.
- Phân tích thống kê bộ nhãn: phân bố kích thước box, so sánh với nhãn gold eval, tách FP/FN, dò 24 hoán vị
  class dẫn tới phát hiện lệch quy ước ID, dựng ảnh overlay và ảnh diff để rà bằng mắt.
- Soạn và biên tập câu chữ cho `GUIDELINE_MINI.md` và báo cáo này.

Phần tôi tự làm: toàn bộ việc vẽ 50 ảnh trong CVAT, quyết định lớp cho từng xe, ba quyết định ca mơ hồ, và
quyết định giữ lại các box bị tính FP thay vì xóa cho đẹp số.

Phần tôi tự kiểm tra lại: mở ảnh overlay và ảnh diff để xác nhận các nhận định thống kê (box suy biến, xe
nhỏ, nhóm MPV), và tự crop nhãn eval để xác nhận bằng mắt chuyện xoay vòng class ID trước khi viết vào báo cáo.

Tài liệu ngoài repo: tài liệu Ultralytics YOLO (train/val) và tài liệu CVAT về định dạng export.
