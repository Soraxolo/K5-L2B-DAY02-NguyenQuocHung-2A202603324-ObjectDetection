# Mini guideline — bài làm cá nhân

> Điền nội dung này trước khi bắt đầu gán nhãn. Giữ lại các quyết định sau khi self-QC.

## Định nghĩa lớp

| Lớp | Định nghĩa của bạn | Ví dụ/điểm phân biệt |
| --- | --- | --- |
| `car` | Xe du lịch dưới 9 chỗ, thân liền một khối, chiều cao xấp xỉ hoặc thấp hơn chiều dài khoang hành khách: sedan, hatchback, SUV, crossover, taxi, xe bán tải (pickup). | Gán `car` cho cả xe bán tải vì cabin nhỏ và khung gầm xe con. Không gán `car` cho xe buýt, xe tải thùng, mô tô. |
| `truck` | Xe chở hàng có thùng/khoang hàng tách rời cabin, hoặc xe đầu kéo kéo rơ-moóc; trục sau thường bánh đôi. | Phân biệt với `car`: `truck` có cabin tách khỏi thùng hàng. Phân biệt với `van`: `van` kín thân liền, `truck` có thùng riêng hoặc thùng hở. |
| `bus` | Xe chở khách thân dài, nhiều hàng cửa sổ chạy dọc thân, sàn cao: xe buýt thành phố, xe khách, minibus chở khách. | Phân biệt với `van`: `bus` có từ ba hàng cửa sổ hành khách trở lên và thân cao đều từ đầu tới đuôi. |
| `van` | Xe thùng kín thân liền dùng chở hàng hoặc chở người cỡ nhỏ: minivan, van chở hàng, xe 7–16 chỗ dạng hộp. | Phân biệt với `car`: `van` cao hơn, mui vuông, khoang sau dạng hộp. Phân biệt với `bus`: `van` ngắn, ít cửa sổ, sàn thấp. |

## Quy tắc chung

- Box bám sát vật thể nghĩa là: hình chữ nhật nhỏ nhất bao trọn **phần thân xe nhìn thấy được**, bốn cạnh chạm mép ngoài cùng của xe (kể cả gương, thùng hàng nhô ra), không lấy thêm mặt đường, bóng đổ, dải phân cách hay thân xe bên cạnh. Mỗi xe một box riêng, kể cả khi hai xe cùng lớp đỗ sát nhau.
- Vật thể bị che khuất: vẫn gán box nếu phần nhìn thấy còn đủ để xác định lớp; box chỉ ôm **phần nhìn thấy**, không đoán phần bị cột đèn, cây hay xe khác che. Xe bị chia thành hai mảnh rời do vật che thì vẫn là **một** box bao phần nhìn thấy, không tách thành hai.
- Vật thể bị cắt bởi rìa ảnh: vẫn gán; cạnh box đặt đúng tại mép ảnh (x = 0, y = 0, x = W hoặc y = H), không đoán phần nằm ngoài khung.
- Vật thể rất nhỏ/mơ hồ: nếu phần nhìn thấy quá nhỏ hoặc quá nhòe để xác định chắc chắn thuộc lớp nào trong bốn lớp, **không đoán** — bỏ qua và ghi lại ảnh đó vào bảng dưới để hỏi giảng viên. Không gán người đi bộ, xe đạp, mô tô/xe máy, biển báo, đèn giao thông, xe trong ảnh quảng cáo hay xe phản chiếu trên kính.

## Ghi lại quyết định cho ít nhất ba ca mơ hồ

| Ảnh | Tình huống | Quyết định cuối | Lý do theo schema |
| --- | --- | --- | --- |
| `drive_008`, `drive_022` | Xe buýt khớp nối: hai khoang nối với nhau bằng khớp accordion, nhìn như hai thân xe rời. Schema không nói một xe khớp nối tính là một hay hai vật thể. | Một box bao cả hai khoang. | Schema phân lớp theo **loại vật thể**, mà hai khoang này chỉ có một buồng lái, một số tuyến và không tách rời khi vận hành — vậy là một chiếc xe. Vẽ hai box sẽ đếm thành hai `bus`, sai số lượng vật thể trong ảnh. |
| `drive_022` | Xe thùng trắng bên phải ảnh: thân cao dạng hộp giống `van`, nhưng phóng to thấy rõ khe và bậc giữa nóc cabin với nóc thùng, thùng đặt trên khung gầm lộ thanh dầm. | `truck`. | Tiêu chí tách `truck` khỏi `van` trong bảng định nghĩa là cabin có rời khỏi thùng hàng hay không. `van` phải là thân liền một khối; xe này cabin và thùng là hai phần riêng nên thuộc `truck` dù kích thước nhỏ. |
| `drive_035` | Dãy xe đỗ phía sau nhà chờ xe buýt: bị trụ nhà chờ, pa-nô quảng cáo và hành khách đứng che thành nhiều mảng rời. Mức che mỗi xe một khác — có chiếc lộ gần trọn đầu xe qua khe giữa hai trụ, có chiếc chỉ ló một vệt nóc phía trên mái nhà chờ. | Gán những xe còn nhận ra hình dáng (chiếc sedan bạc bên trái, chiếc bạc sau tấm pa-nô), mỗi xe một box ôm phần nhìn thấy. Bỏ chiếc chỉ còn vệt nóc trắng. | Guideline cho phép gán xe bị che khi phần nhìn thấy đủ xác định lớp, nên mức che chứ không phải việc có bị che quyết định. Hai chiếc kia còn thấy đường nóc, kính lái và vòm bánh nên xếp `car` chắc chắn; chiếc chỉ ló nóc thì không phân biệt được `car` với `van`, gán vào là đoán. Box chỉ ôm phần nhìn thấy, không nối qua trụ nhà chờ và không đoán phần khuất. |
