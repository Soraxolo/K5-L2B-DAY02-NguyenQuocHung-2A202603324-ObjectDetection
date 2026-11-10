# Báo cáo cá nhân — Ngày 2 Object Detection

## 1. Thông tin học viên

- Họ tên: Nguyễn Quốc Hưng
- MSSV: 2A202603324
- Email: sora.12082004@gmail.com
- Lớp: K5 — Level 2B (L2B)
- Link repo bài làm: https://github.com/Soraxolo/K5-L2B-DAY02-NguyenQuocHung-2A202603324-ObjectDetection
- Link Colab đã chạy: https://colab.research.google.com/drive/1PWrqQz0ZR8BsgH59yBVzpupsaMoXXflj?usp=sharing

## 2. Schema và mini guideline

Guideline cá nhân của tôi nằm ở [GUIDELINE_MINI.md](../../GUIDELINE_MINI.md).

Khi viết guideline tôi cố quy mọi thứ về câu hỏi hình học, vì ảnh CCTV 640×640 thì đoán theo cảm giác rất dễ lệch giữa ảnh này với ảnh kia. Hai câu hỏi tôi dùng nhiều nhất: đường nóc xe có tụt dốc về phía đuôi không (để tách `car` với `van`), và cabin có tách rời khỏi thùng hàng không (để tách `truck` với `van`). Riêng `bus` thì dựa vào dãy cửa sổ hành khách chạy liên tục dọc thân và tỉ lệ thân dài trên 2,5 lần chiều cao.

Ba ca mơ hồ tôi gặp khi vẽ:

| Ảnh | Tình huống | Quyết định | Căn cứ |
| --- | --- | --- | --- |
| `drive_008`, `drive_022` | Xe buýt khớp nối hai khoang | Một box bao cả hai khoang | Schema phân theo loại vật thể. Hai khoang chung một buồng lái và không tách rời khi chạy, nên tôi coi là một xe |
| `drive_022` | Xe thùng trắng nhỏ, thân cao trông như `van` | `truck` | Phóng to thì thấy khe và bậc giữa nóc cabin với nóc thùng, thùng đặt trên khung gầm. `van` phải là thân liền một khối |
| `drive_035` | Xe đỗ sau nhà chờ xe buýt, bị che ở nhiều mức khác nhau | Gán những chiếc còn nhận ra hình dáng, bỏ chiếc chỉ ló một vệt nóc | Thứ quyết định là mức che chứ không phải việc có bị che. Không đủ dữ kiện phân biệt `car` với `van` thì tôi không đoán |

Sau khi chấm với gold tôi thêm một quy tắc nữa vào guideline: không gán xe có cạnh ngắn dưới 20 px. Lý do giống hệt ca `drive_035`, ở cỡ đó tôi không đọc được đường nóc hay cabin nữa.

Chỗ cần giảng viên làm rõ: quy ước class ID giữa `data.yaml` và nhãn gold không khớp nhau. Tôi trình bày ở mục 4.

## 3. Annotation và self-QC

Số file nhãn train: 50/50, cùng basename với ảnh. Checker chạy ra `ĐẠT cấu trúc: 50/50 file nhãn hợp lệ`.

Tôi gán nhãn bằng CVAT 2.74.1 cài cục bộ, task `Day2-train`, bốn label đều để kiểu `rectangle` và đặt Image quality 100 vì để mặc định 70 thì xe ở xa nhòe, kéo box không sát được. Export theo định dạng `Ultralytics YOLO Detection 1.0`. Khi đưa về `dataset/labels/train/` tôi ánh xạ class theo tên lấy từ `data.yaml` trong file export chứ không theo thứ tự, để tránh lệch ID nếu CVAT sắp xếp khác.

Để rà bằng mắt, tôi vẽ box lên lại ảnh gốc (phóng 2×) cho cả 50 ảnh rồi soi kỹ 10 ảnh: `drive_001`, `drive_002`, `drive_006`, `drive_007`, `drive_008`, `drive_009`, `drive_014`, `drive_022`, `drive_030`, `drive_038`.

Những gì tôi tìm ra và sửa ở bước này:

- Ba box suy biến do click nhầm chuột: `drive_006` 1,1×2,1 px, `drive_007` 1,3×1,5 px, `drive_038` 4,3×7,3 px. Xóa cả ba.
- 55 box có cạnh ngắn dưới 10 px và 206 box dưới 15 px. Đem so với nhãn gold của 10 ảnh eval có sẵn trong repo thì gold không có box nào dưới 10 px, chỉ 2 box dưới 15 px. Tôi hiểu là mình đang gán quá tay ở vùng xe rất nhỏ, và đó là lý do sau này tôi đặt ngưỡng 20 px.
- Rà lại nhóm xe MPV trong `drive_030` cho thống nhất, vì có mấy chiếc dáng gần giống nhau mà tôi gán hai lớp khác nhau.

Commit khóa nhãn ban đầu: `6522923` (*CP4: lock initial annotation (50 YOLO label files) + mini guideline*).

## 4. Gold QC và rework

Tôi chạy `tools/evaluate_annotations.py` với `gold_train_labels/` giảng viên phát, kết quả lưu ở [`outputs/annotation_evaluation.json`](../../outputs/annotation_evaluation.json). Bản trước rework giữ nguyên ở `outputs/annotation_evaluation_truoc_rework.json`.

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

Vòng rework tôi làm là áp quy tắc ngưỡng 20 px đã ghi trong guideline, gỡ 374 box, còn 879. Precision lên từ 0,37 thành 0,51, recall tụt nhẹ vì trong số bị loại cũng có vài xe gold có gán. F1 vẫn chưa tới 0,80.

### Nhãn gold đánh class ID khác `data.yaml`

Lần chạy đầu tiên có một chỗ rất vô lý mà tôi mất một lúc mới nghĩ ra: F1 chỉ 0,491 nhưng `mean_iou_of_matches` lại 0,834. Box vẽ sát tới mức đó thì không thể sai nhiều như vậy. Lúc đầu tôi tưởng mình gán sai lớp hàng loạt, vì bảng lỗi cho thấy 50 box tôi gán `bus` thì gold gọi là `van`, 35 box `van` thì gold gọi `truck`, 19 box `truck` thì gold gọi `bus`. Nhưng mở mấy ca đó ra xem thì toàn xe buýt thành phố thật, không đời nào gold gọi một chiếc buýt hai toa là `van` được.

Hóa ra nhãn gold đánh ID theo thứ tự `car, van, truck, bus`, còn `data.yaml` và bảng schema trong README ghi `car=0, truck=1, bus=2, van=3`. Tôi kiểm lại bằng ba cách độc lập:

1. Crop trực tiếp nhãn eval có sẵn trong repo. Toàn bộ box class 3 là xe buýt thành phố và buýt hai tầng, class 2 là xe tải với xe cẩu, class 1 là microvan. Đọc theo `data.yaml` thì 12 chiếc "van" kia đều là xe buýt, vô lý.
2. Thử cả 24 hoán vị class khi chấm nhãn của tôi với gold train. Hoán vị `car→0, truck→2, bus→3, van→1` cho F1 cao nhất là 0,721 so với 0,586 khi giữ nguyên, FN giảm từ 204 xuống 101. Các hoán vị còn lại đều tệ hơn.
3. Chỉ số per-class sau khi train ở mục 5. `car` là lớp duy nhất mà hai quy ước trùng ID, và nó đạt mAP50 0,738, trong khi `bus` đúng bằng 0,000.

Chấm lại cùng bộ nhãn đó theo quy ước của gold thì được P 0,628, R 0,845, F1 0,721, mean IoU 0,844. Số box sai lớp thật chỉ còn 12 trên 879, gần như đều nằm ở ranh giới `car` với `van` của mấy xe MPV.

Tôi không đổi ID trong bài nộp. `data.yaml` mới là schema chính thức của bài và model cũng train theo thứ tự đó, nên tôi giữ nguyên rồi báo giảng viên để lớp thống nhất.

### Về số FP còn lại

Tôi lấy mẫu 40 trong 177 box bị tính FP có cạnh ngắn từ 28 px trở lên và mở từng cái ra xem. Gần như tất cả đều là xe thật thuộc schema: ô tô con, xe buýt, microvan, xe tải, nhìn rõ mặt mũi. Gold cũng thưa hơn hẳn, 13,1 box mỗi ảnh so với 17,6 box của tôi sau rework. Nên phần lớn FP là do gold bỏ sót chứ không phải tôi vẽ thừa.

Tôi quyết định giữ nguyên những box này. Xóa đi thì F1 đẹp hơn, nhưng bộ nhãn sẽ sai hơn so với schema và guideline của chính mình, mà đó mới là thứ bài học này nhắm tới.

Tôi không sửa gold và không viết lại lịch sử commit khóa nhãn.

## 5. Kết quả YOLO

Runtime: Google Colab, GPU Tesla T4 (CUDA:0, 14913 MiB). Python 3.13.16, torch 2.14.1+cu130, Ultralytics 8.4.208.

Tham số giữ nguyên notebook: `yolo26n.pt`, 50 epoch, `imgsz=640`, `batch=-1`, `seed=20260909`. Best checkpoint rơi vào epoch 26, lưu ở `outputs/yolo26_runs/vehicle_annotation/weights/best.pt` (file `.pt` không commit theo quy định).

| Chỉ số trên 10 ảnh eval | Giá trị |
|---|---:|
| Precision | 0,3156 |
| Recall | 0,3066 |
| mAP50 | 0,2740 |
| mAP50–95 | 0,2006 |

Theo từng lớp:

| Lớp | Instances | P | R | mAP50 | mAP50–95 |
| --- | ---: | ---: | ---: | ---: | ---: |
| `car` | 94 | 0,629 | 0,777 | 0,738 | 0,521 |
| `truck` | 5 | 0,391 | 0,200 | 0,235 | 0,203 |
| `bus` | 2 | 0,000 | 0,000 | 0,008 | 0,003 |
| `van` | 12 | 0,243 | 0,250 | 0,115 | 0,075 |

Evidence: [ảnh prediction trên 10 ảnh eval](assets/eval_predictions/visuals/), [log train và biểu đồ](assets/yolo26_runs/vehicle_annotation/) gồm `results.csv` và `results.png`, [chỉ số đánh giá](assets/yolo26_runs/vehicle_eval/) gồm confusion matrix và PR curve.

### Trả lời bốn câu hỏi

**1. Lớp nào recall thấp nhất, ảnh nào minh họa?**

`bus` với recall 0,000, sau đó là `truck` 0,200. Ảnh minh họa là `assets/eval_predictions/visuals/drive_053.jpg`. Trong ảnh đó model khoanh chiếc xe buýt xanh ở giữa và dự đoán `bus 0.85`, tức là nhận dạng đúng hoàn toàn. Nhưng nhãn eval gán chính chiếc xe ấy id 3, mà `data.yaml` gọi id 3 là `van`. Còn hai instance mang id 2 (`bus`) trong 10 ảnh eval thì thực chất là xe tải, model không dự đoán `bus` ở đó nên recall của `bus` bằng 0. Con số này đo độ lệch tên lớp chứ không đo khả năng của model.

**2. Nếu mAP50 cao nhưng mAP50–95 thấp thì box đang gặp vấn đề gì?**

Model tìm đúng vật nhưng cạnh box chưa khớp sát. Ở ngưỡng IoU 0,50 vẫn tính là trúng, lên 0,75 hay 0,90 thì trượt. Thứ cần kiểm là box có nới ra nền không, có ôm thêm bóng đổ hay thân xe bên cạnh không, và biên của xe bị che hay bị cắt khung đã đặt đúng chưa. Trong bài này `car` có mAP50 0,738 và mAP50–95 0,521, chênh đúng kiểu đó. Với ảnh CCTV nhiều xe nhỏ thì lệch vài pixel đã đủ làm IoU tụt mạnh.

**3. Lỗi class nào liên quan tới guideline?**

Sau khi quy hai bên về cùng quy ước ID, lỗi lớp thật chỉ còn 12 box trên 879 và đều nằm ở ranh giới `car` với `van`. Đây đúng là chỗ guideline của tôi yếu: tôi dựa vào đường nóc, nhưng xe con thân hộp cỡ nhỏ kiểu Wagon R vừa có capo của `car` vừa có nóc phẳng của `van`. Guideline cần thêm một câu cho loại này, chẳng hạn xe dưới 5 chỗ có capo thì xếp `car` dù nóc phẳng. Còn các lệch lớp quy mô lớn (`bus` thành `van`, `van` thành `truck`, `truck` thành `bus`) thì không phải do guideline, mà do nhãn gold xoay vòng ID như tôi đã trình bày ở mục 4.

**4. Vì sao không dùng 10 ảnh eval để train dù chúng có nhãn?**

Vì notebook lấy chính tập đó làm validation để chọn `best.pt`. Train trên nó thì model đã nhìn thấy đáp án, chỉ số đo được chỉ phản ánh trí nhớ chứ không phải khả năng tổng quát hóa, và việc chọn checkpoint cũng mất ý nghĩa. Cũng cần nói rõ 10 ảnh này là validation chứ không phải test set độc lập, chúng đã tham gia chọn checkpoint nên các con số ở trên vẫn lạc quan hơn thực tế.

## 6. Khó khăn và cách xử lý

| Bước/thời điểm | Lỗi quan sát | Cách đã thử | Kết quả/còn thiếu |
|---|---|---|---|
| Tạo task CVAT | Dán JSON label thì báo `unknown label type "undefined"`, task vẫn tạo ra nhưng không có label nào nên mọi công cụ vẽ đều xám | Thêm trường `"type": "rectangle"` cho cả bốn label rồi lưu lại ở tab Raw của task | Vẽ được bình thường. Đặt `type` còn khóa luôn công cụ về bounding box, khỏi lỡ tay vẽ polygon |
| Self-QC | Nhìn file `.txt` thì không biết box nào lỏng hay chỗ nào bỏ sót | Vẽ box lên lại ảnh gốc cho cả 50 ảnh, thống kê kích thước box rồi so với nhãn gold eval | Tìm ra 3 box suy biến và thấy mình gán quá tay ở xe rất nhỏ |
| Gold QC | F1 0,491 mà mean IoU 0,834, hai số này không thể cùng đúng | Tách lỗi thành hai nhóm "thừa box" và "sai lớp", sau đó thử cả 24 hoán vị class | Phát hiện gold dùng thứ tự ID khác `data.yaml` |
| Train Colab | mAP tổng thấp, riêng `bus` bằng 0 | Đọc chỉ số per-class, đối chiếu số instance từng lớp trong nhãn eval, mở ảnh prediction `drive_053` | Xác nhận nguyên nhân là lệch quy ước ID, không phải model hay nhãn của tôi |

## 7. Tự đánh giá theo rubric

| Tiêu chí | Tự đánh giá |
| --- | --- |
| Hoàn thành và định dạng YOLO | Đủ 50 file, checker đạt, 5 cột normalized, class ID theo `data.yaml` |
| Đầy đủ vật thể | Recall 0,845 khi chấm theo quy ước của gold. Phần FP tôi kiểm mẫu cho thấy nhiều xe thật mà gold bỏ sót |
| Độ bám sát và hình học box | mean_iou_of_matches 0,844, thuộc mức xuất sắc theo bảng chất lượng |
| Phân lớp nhất quán | 12/879 box lệch lớp sau khi quy về cùng quy ước, tập trung ở ranh giới `car` với `van` |
| Mini guideline và quyết định mơ hồ | Có định nghĩa bốn lớp, quy tắc box, ba ca mơ hồ kèm lý do, cộng quy tắc ngưỡng kích thước thêm sau QC |
| Gold-set annotation QC | F1 chưa đạt 0,80 nếu đọc theo `data.yaml` (0,586), đạt 0,721 nếu đọc theo quy ước của gold. Có một vòng rework với số liệu trước sau và lý do giữ lại phần FP |
| Pipeline YOLO và phân tích | Colab chạy xong, giữ nguyên split, có metrics, ảnh eval và phân tích nguyên nhân theo từng lớp |

Thứ tôi chưa kiểm được là chỉ số thật của `truck`, `bus`, `van`. Phải chờ giảng viên chốt quy ước ID rồi chạy lại mới có con số đáng tin.

## 8. Công cụ AI và tài liệu tham khảo

Tôi có dùng Claude Code (Anthropic) trong bài này, ở những bước sau:

- Dựng task CVAT (sinh JSON bốn label), viết script chuyển export CVAT về đúng layout `dataset/labels/train/` với ánh xạ class theo tên, chạy `check_yolo_labels.py` và `evaluate_annotations.py`, thao tác git.
- Phân tích thống kê bộ nhãn: phân bố kích thước box, so với nhãn gold eval, tách FP và FN, dò 24 hoán vị class, dựng ảnh overlay và ảnh diff để tôi rà bằng mắt.
- Soạn và biên tập câu chữ cho `GUIDELINE_MINI.md` và báo cáo này.

Phần tôi tự làm: vẽ toàn bộ 50 ảnh trong CVAT, quyết định lớp cho từng xe, ba quyết định ca mơ hồ, và quyết định giữ lại các box bị tính FP thay vì xóa cho đẹp số.

Phần tôi tự kiểm tra lại: mở ảnh overlay và ảnh diff để xác nhận các nhận định thống kê (box suy biến, xe quá nhỏ, nhóm MPV), tự crop nhãn eval xem tận mắt rồi mới tin chuyện xoay vòng class ID, và mở ảnh `drive_053` để kiểm chứng câu trả lời số 1.

Tài liệu ngoài repo: tài liệu Ultralytics về train và val, tài liệu CVAT về định dạng export.
