# Sổ tay Vận hành & Khởi động AutoService (Phiên bản Trạm công tác chuyên dụng CVC)

## SỔ TAY VẬN HÀNH · OPERATIONS MANUAL

## AutoService – Sổ tay khởi động và vận hành

Khởi động mặc định cho trạm thông thường · CVC 1upCombo · CVC RuninLog / RuninAuto · CVC giá làm mát

| Mục | Nội dung |
| --- | --- |
| Mã tài liệu | OM-AS-CVC-001 |
| Phiên bản / Ngày cập nhật | V1.0 / 06-11-2026 (Bản chuyển đổi từ Word ngày 09-11-2026) |
| Đối tượng áp dụng | Nhân sự vận hành tại hiện trường (Ops), kỹ thuật viên gỡ lỗi (debug) và nhân sự tích hợp dự án |
| Phạm vi áp dụng | Dịch vụ trạm kiểm tra tự động hóa công nghiệp AutoService; hướng dẫn khởi động, vận hành nhiều dịch vụ đồng thời, quản lý biểu tượng và xử lý sự cố cho ba loại trạm đặc biệt: CVC 1upCombo, CVC Runin, và giá làm mát CVC |
| Hình thức sử dụng | Có thể đọc ngoại tuyến, không cần tài nguyên bên ngoài. Các lệnh trong tài liệu chỉ là ví dụ, không tự động được thực thi. Việc đọc tài liệu không khởi động dịch vụ và cũng không thay đổi cấu hình thiết bị. |

| Vai trò | Họ tên | Ngày | Chữ ký |
| --- | --- | --- | --- |
| Soạn thảo |  Lâm Trịnh | 10/11 | Lâm | 
| Kiểm duyệt | Lâm Trịnh | 10/11 | Lâm |
| Phê duyệt | Lâm Trịnh | 10/11 | Lâm |

Các đường dẫn, tên dự án và mã thiết bị trong sổ tay chỉ là ví dụ minh họa quy tắc. Khi triển khai thực tế, phải sử dụng tên dự án, `StationID` và sơ đồ ánh xạ phần cứng tương ứng tại hiện trường.

---

## Những điểm cần nắm trước khi đọc

Có bốn cách khởi động: trạm thông thường dùng `python run.py`; riêng `CVC 1upCombo` dùng tham số `--station-type` để chọn dự án; `CVC Runin` và `giá làm mát CVC` dùng `--config` để chỉ định file cấu hình cố định. Hai tham số `--config` và `--station-type` không được sử dụng đồng thời.

Các dịch vụ chạy đồng thời phải có `client_id` riêng và sử dụng các cổng nghiệp vụ khác nhau. `run.py` hiện luôn trả về mã thoát `0`, vì vậy không thể chỉ dựa vào mã thoát để kết luận khởi động thành công; cần kiểm tra log đường dẫn đầy đủ, giao diện (UI) hoặc trạng thái nghiệp vụ.

---

## 01 Chọn đúng phương thức khởi động trước

Khả năng khởi động đặc biệt lần này xuất phát từ 3 yêu cầu hiện trường của dự án CVC. Hãy chọn điểm vào (entry point) dựa trên danh sách trạm trước, sau đó chuẩn bị cấu hình và biểu tượng theo các chương tiếp theo.

Quy tắc mặc định: Các trạm chưa được đánh dấu đều sử dụng khởi động thông thường. Lô này chỉ có 3 loại trạm đặc biệt gồm `CVC 1upCombo`, `CVC Runin`, và `giá làm mát CVC` là sử dụng các điểm vào mở rộng trong bảng dưới đây. Các trạm khác chưa đánh dấu sẽ thống nhất sử dụng `python run.py` cùng biểu tượng khởi động thông thường vốn có, không chuyển đổi phương thức khởi động chỉ vì framework hỗ trợ tham số mở rộng.

| Trạm áp dụng / Nguồn gốc yêu cầu | Phương thức khởi động chỉ định | Lý do thích ứng |
| --- | --- | --- |
| Trạm thông thường: Tất cả các trạm chưa đánh dấu khác | `python run.py` | Kế thừa danh tính Mars, thư mục cấu hình trạm và quy trình khởi động thông thường ban đầu. |
| Thiết bị R1 · CVC 1upCombo | `python run.py --station-type <Tên dự án>` | Khách hàng cần tương thích 1up Type-C / 2up NFC; dựa trên số lượng Group hiện tại của Mars, tự động chọn một / hai nhóm cấu hình khi khởi động. |
| Thiết bị R2 · CVC Runin | `python run.py --config "<Đường dẫn file cấu hình service>"` | Trên cùng một thiết bị, cùng định danh và số lượng, cần thực thi riêng biệt hai bộ logic chương trình RuninLog / RuninAuto; mỗi biểu tượng liên kết với một cấu hình cố định. |
| R3 · giá làm mát CVC | `--config` + `MarsConfig.virtual: true` | giá làm mát không có Mars, mượn máy tính của thiết bị lân cận để chạy dịch vụ làm mát độc lập; mô phỏng thời lượng kiểm tra và hiển thị màn hình đếm thời gian thông qua Mars ảo. |

---

## Phân biệt tham số khởi động với loại trạm thực tế

- `--station-type`: Chỉ định phân đoạn cấu hình dự án (Project) hoặc hậu tố thư mục nhóm (Group), nhưng mã trạm thực tế (Station ID) vẫn phải tuân theo file JSON.
- `--config`: Chỉ định trực tiếp file cấu hình dịch vụ (`service JSON`). Có thể liên kết với nhánh Mars thực (như CVC Runin) hoặc cấu hình làm mát cho Mars ảo (như giá làm mát CVC). Mã trạm thực tế được tải vẫn do `station_type` trong file JSON quyết định.

Tổng quan 4 loại điểm vào: Khởi động thông thường (phương thức mặc định cho trạm chưa đánh dấu), khởi động theo `station-type` (`CVC 1upCombo`: Type-C / NFC), khởi động theo `config` (`CVC Runin`: Log / Auto), và khởi động `IMUCLocal` (`giá làm mát CVC`: gắn phụ thuộc trên máy tính lân cận).

---

## 02 Nguồn gốc yêu cầu, trạm áp dụng và quy tắc định danh

Chương này giải thích nguồn gốc yêu cầu và phương thức thích ứng của 3 loại trạm đặc biệt CVC, đồng thời cung cấp các quy tắc thống nhất cho định danh cấu hình, đặt tên thư mục và ngữ nghĩa ví dụ sử dụng.

### 2.1 Nguồn gốc yêu cầu và danh sách thích ứng trạm đặc biệt

| Nguồn gốc yêu cầu | Yêu cầu thực tế từ khách hàng / hiện trường | Thích ứng được áp dụng trong lô này |
| --- | --- | --- |
| R1 · CVC 1upCombo | Một thiết bị Combo cần tương thích với 2 giải pháp kiểm tra: 1up Type-C và 2up NFC. Khách hàng yêu cầu có thể tự động chọn phiên bản tương ứng dựa trên giải pháp hiện tại. | Sử dụng `--station-type` cho một điểm vào dự án; đọc số lượng Group hiện tại của Mars, 1 Group chọn `one_group`, 2 Group chọn `two_group`. |
| R2 · CVC Runin | RuninLog và RuninAuto sử dụng chung một thiết bị, danh tính và số lượng thiết bị giống nhau, nhưng bắt buộc phải chạy riêng biệt hai bộ logic chương trình. | Sử dụng hai điểm vào `--config` cố định, cấu hình `station_type` trong file JSON lần lượt là `RuninLog`, `RuninAuto`, chọn thông qua hai biểu tượng riêng biệt. |
| R3 · giá làm mát CVC | Bản thân giá làm mát không có Mars, chỉ có thể gắn phụ trên máy tính của thiết bị lân cận, cần kiểm tra mô phỏng thời lượng kiểm tra, màn hình đếm thời gian và kết quả hoàn tất làm mát. | Máy tính lân cận khởi động thêm tiến trình `IMUCLocal` độc lập, sử dụng `--config` và `virtual=true`, không kết nối với Mars thật của trạm bên cạnh. |

Ngữ nghĩa hiện tại của việc "tự động chuyển đổi": Việc tự động lựa chọn của R1 diễn ra mỗi khi khởi động: chọn phiên bản dựa trên số lượng Group trong cấu hình kiểm tra hiện tại của Mars. Không thực hiện chuyển đổi nóng (hot-switch) mượt mà trong khi đang chạy; sau khi thay đổi giải pháp tương ứng với Type-C / NFC, hãy hoàn thành tác vụ và thoát dịch vụ, sau đó khởi động lại.

---

### 2.2 Quy tắc định danh cấu hình và thư mục

| Tên gọi | Nguồn gốc / Vị trí | Tác dụng và yêu cầu điền |
| --- | --- | --- |
| Mars StationID | `station_id` trong `mars.json` | Định danh thiết bị để dịch vụ thực tế đối chiếu; `station_id` trong JSON, `station_name.mars_id` trong YAML và tên file sử dụng cùng một giá trị. Phân biệt hoa thường hoàn toàn khớp nhau. |
| Mars StationName | `mars_config.identity.station_name` | Tên thư mục trạm của khởi động thông thường; sau khi chỉ định `--station-type`, nó không còn quyết định thư mục gốc của dự án nữa. |
| Tên dự án cấu hình | Giá trị tham số `--station-type` | Quyết định thư mục dự án và hậu tố thư mục con Group; không dựa vào đây để sửa đổi mã số thiết bị thực tế. |
| Loại trạm thực tế | `station_type` trong service JSON | Quyết định các module Control / Service dưới `StationControl`, ví dụ: `Camera_one_group`, `Camera_two_group`, `IMUCLocal`. |
| Định danh trạm trên hệ thống điều phối | `client_id` trong service JSON | Tương ứng với `stationCode` của ATCS, đồng thời quyết định tên phiên chạy (instance) của khóa file (file lock), cơ sở dữ liệu và bộ nhớ đệm chế độ (mode cache). |
| Cổng dịch vụ (Service Port) | `service_prot` trong service JSON | Hai dịch vụ chạy đồng thời phải khác nhau. Tên trường tiếp tục sử dụng lỗi chính tả hiện tại `service_prot`, không đổi thành `service_port`. |
| Thư mục theo dây chuyền (Line ID) | Ký tự cuối cùng của `line_id` trong Mars identity | Ví dụ: `F7-4F-G01` $\rightarrow$ `L1`. `line_id` trong service JSON là trường dây chuyền của ATCS, không dùng nó để suy ra thư mục cấu hình. |

Định danh không thay đổi theo tên thư mục: Ngay cả khi thư mục dự án được gọi là `FATP-CAM-COMBO-1`, Mars StationName / StationID vẫn có thể là giá trị thực tương ứng với `COMBO-2`. Việc lựa chọn dự án sẽ không tự động đổi số "2" trong mã số thiết bị thành số "1".

---

### 2.3 Mối quan hệ giữa ví dụ tài liệu và cấu hình hiện trường

Sổ tay này giữ lại các ví dụ đường dẫn hiện tại của framework như `FATP-CAM-COMBO-1`, `DEVICE_01`,... để thuận tiện giải thích quy tắc tìm kiếm; các ví dụ này không đồng nghĩa với các thư mục chính thức đã được tạo tại hiện trường CVC. Hiện trường nên sử dụng tên dự án triển khai thực tế, StationID thực, `station_type` tương ứng với từng giải pháp và ánh xạ phần cứng thực tế.

---

## 03 Môi trường, thư mục và chuẩn bị trước khi vận hành

Chương này cung cấp bản sao dự án, các file định danh Mars và cấu trúc cấu hình nghiệp vụ phải được xác nhận trước khi vận hành, cũng như hướng dẫn điểm vào cho lần cài đặt đầu tiên.

### 3.1 Xác nhận đang chạy bản sao dự án nào

Các lệnh Windows sau đây mặc định được thực thi tại thư mục gốc mã nguồn. Nếu đang chạy bản sao bàn giao đã giải nén, hãy vào thư mục gốc của bản sao đó trước; `run.py` và thư mục `configs` của nó phải đồng bộ với nhau.

```powershell
Set-Location -LiteralPath "E:\项目\P27\AutoService 模拟专用"
python --version
```

Thư mục mã nguồn và bản sao `dist` độc lập với nhau: Chỉnh sửa mã nguồn sẽ không tự động cập nhật `dist/AutoService`. `run.py` trùng tên hoặc tên biểu tượng giống nhau đều không chứng minh được rằng bạn đang sử dụng cùng một bộ mã. Trước tiên hãy kiểm tra thư mục hiện tại của terminal, mục tiêu của shortcut (đường tắt) và thư mục làm việc. Sổ tay này không sửa đổi `dist` cũng như không tạo cấu hình chính thức.

---

### 3.2 Các file cần thiết cho dịch vụ Mars thực tế

`Thư mục chính của người dùng hiện tại/`
- `mars.json`
- `.mars_preference/`
  - `Local.json`
  - `Các cấu hình kiểm tra khác được ánh xạ bởi MarsProfileConfig.json`

Cấu hình kiểm tra hiện được chọn do `active_test_config` trong `mars.json` quyết định. Điểm vào thực tế cũng sẽ kiểm tra xem automation đã được bật hay chưa, đồng thời đối chiếu các quy tắc hiện có như chế độ kiểm tra được ánh xạ, số lượng slot mỗi group, `use_trigger`, v.v.

---

### 3.3 Cấu trúc chuẩn của một bộ cấu hình nghiệp vụ

`Thư mục cơ sở cấu hình được chọn bất kỳ/`
- `service/`
  - `DEVICE_01.json`
- `station/`
  - `DEVICE_01.yaml`
- `MarsProfileConfig.json` *(Cần thiết cho nhánh Mars thực)*

`DEVICE_01` là mã số ví dụ thống nhất trong sổ tay này, hiện trường bắt buộc phải thay thế bằng mã số thiết bị thực tế. Tên file và định danh bên nội bộ phải đồng nhất. Các ví dụ đường dẫn xuất hiện bên dưới chỉ mang tính chất minh họa quy tắc, không có nghĩa là cấu hình chính thức tương ứng đã được tạo.

---

### 3.4 Cài đặt và sử dụng lần đầu

Môi trường đã cài đặt có thể sử dụng trực tiếp Python hiện có. Lần cài đặt đầu tiên sử dụng điểm vào cài đặt gốc:

```powershell
python install_autoservice.py
```

Cài đặt không tham số sẽ đối chiếu môi trường, cài đặt các dependency ngoại tuyến (offline) và tạo biểu tượng AutoService mặc định trên màn hình nền (desktop). Trình cài đặt ngoại tuyến giới hạn cho x86-64, Python 3.8 / 3.10 / 3.12 bản 64-bit và pip khả dụng. Điểm vào cài đặt khác với điểm vào "chỉ tạo biểu tượng", xem phần 09.

---

## 04 Khởi động thông thường: Tìm kiếm theo Mars StationName

Chương này giải thích quy tắc tìm kiếm cấu hình, quy trình thực thi và cách đánh giá thành công của điểm vào mặc định, áp dụng cho tất cả các trạm chưa đánh dấu.

Trạm áp dụng: Tất cả các trạm chưa được đánh dấu khác. Ngoại trừ `CVC 1upCombo`, `CVC Runin`, và `giá làm mát CVC` được đánh dấu rõ ràng trong sổ tay này, các trạm còn lại thống nhất thực hiện `python run.py` theo chương này, giữ lại biểu tượng thông thường ban đầu và quy trình tìm kiếm cấu hình cũ. Phù hợp cho trường hợp tiếp tục sử dụng cấu hình thiết bị ban đầu, không cần chuyển đổi dự án thông qua tham số.

```powershell
python run.py
```

### 4.1 Quy tắc tìm kiếm

`configs/L<Ký tự cuối dây chuyền Mars thực tế>/<Mars StationName thực tế>/service/<Mars StationID thực tế>.json`

Ví dụ: Ký tự cuối của dây chuyền Mars là `1`, StationName là `FATP-CAM-COMBO-2`, StationID là `DEVICE_01`:

`configs/L1/FATP-CAM-COMBO-2/`
- `service/DEVICE_01.json`
- `station/DEVICE_01.yaml`
- `MarsProfileConfig.json`

---

### 4.2 Quy trình thực thi

1. **Đọc định danh Mars**: Đọc `mars.json` của người dùng hiện tại, lấy thông tin dây chuyền (line), StationName, StationID.
2. **Xác định vị trí thư mục cấu hình gốc**: Đọc trực tiếp các thư mục `service`, `station`, `MarsProfileConfig` dưới thư mục gốc của trạm.
3. **Thực thi kiểm tra khởi động gốc**: Kiểm tra automation, định danh thiết bị, khóa file (file lock), cổng nghiệp vụ và cấu hình kiểm tra Mars.
4. **Tải trạm thực tế**: Nhập (import) Control / Service theo `station_type` trong file JSON, chuyển vào quy trình khởi tạo và nghiệp vụ ban đầu.

Khởi động thông thường không tự động chọn thư mục con Group: Điểm vào nghiệp vụ `python run.py` không tự động chuyển sang `one_group` / `two_group`. Chỉ khi có cấu hình lồng nhau, mới nên sử dụng `--station-type` hoặc `--config`. Quy tắc đọc thư mục lồng nhau của bản thân WebUI không thay thế được quy tắc điểm vào nghiệp vụ.

---

### 4.3 Đánh giá thành công

Kiểm tra log khởi tạo trạm, trang web tương ứng hoặc giao diện nghiệp vụ đã sẵn sàng hay chưa. Mã thoát cuối cùng của `run.py` hiện tại luôn được cố định là `0`, do đó không thể chỉ dựa vào việc "lệnh trả về 0" để đánh giá khởi động thành công.

---

## 05 Khởi động bằng `station-type`: chọn thư mục theo dự án và số Group

Chương này giải thích điểm vào tự động chọn thư mục dựa trên tên dự án và số lượng Group, áp dụng cho thiết bị `CVC 1upCombo`.

Thích ứng trạm đặc biệt R1: `CVC 1upCombo`. Chọn dự án cấu hình thông qua biểu tượng chuyên dụng; khi khởi động sẽ đọc số lượng Group của giải pháp Mars hiện tại và tự động tải phiên bản tương ứng. Các trạm thông thường chưa đánh dấu không sử dụng điểm vào mở rộng này.

| Giải pháp kiểm tra của khách hàng | Số lượng Group hiện tại của Mars | Thư mục cấu hình tương ứng |
| --- | --- | --- |
| 1up Type-C | 1 Group | `one_group_<Tên dự án>` |
| 2up NFC | 2 Group | `two_group_<Tên dự án>` |

Hai file cấu hình service JSON phải trỏ lần lượt đến mã nguồn trạm thực tế của giải pháp tương ứng. Các tên dự án, tên file và tên module log dưới đây là các ví dụ về quy tắc đường dẫn, hiện trường điền theo cấu hình thực tế. Điểm vào này trong lô này được sử dụng cho thiết bị `CVC 1upCombo`: đáp ứng nhu cầu tương thích với 2 giải pháp 1up Type-C / 2up NFC của khách hàng.

```powershell
python run.py --station-type FATP-CAM-COMBO-1
```

---

### 5.1 Công thức đường dẫn chính xác của phiên bản hiện tại

`configs/L<Ký tự cuối dây chuyền Mars thực tế>/<Tên dự án tham số>/<Tiền tố Group>_<Tên dự án tham số>/`
- 1 Group $\rightarrow$ `one_group_<Tên dự án tham số>`
- 2 Group $\rightarrow$ `two_group_<Tên dự án tham số>`

Khi Mars StationName là `FATP-CAM-COMBO-2`, tham số là `FATP-CAM-COMBO-1`, hiện tại có 2 Group:

```text
configs/L1/FATP-CAM-COMBO-1/
└─ two_group_FATP-CAM-COMBO-1/
   ├─ service/DEVICE_01.json
   ├─ station/DEVICE_01.yaml
   └─ MarsProfileConfig.json
```

Tại đây sẽ không dùng lẫn lộn `COMBO-2` làm thư mục dự án vòng ngoài nữa. Cả hai cấp thư mục đều do tham số quyết định, các file thiết bị vẫn được tìm kiếm theo Mars StationID thực tế.

---

### 5.2 Quy trình thực thi

| Điều kiện | Hành vi |
| --- | --- |
| Có nhiều Fixture hoặc Slot trong 1 Group | Vẫn chọn `one_group`; đếm số lượng Group, không phải tổng số slot. |
| 0 hoặc nhiều hơn 2 Group | Phân nhánh thư mục tự động này báo không hỗ trợ, không tự động chọn thư mục khác. |
| Thư mục dự án hoặc file thiết bị không tồn tại | Hiển thị đường dẫn cần truy cập và báo thất bại, không quay về thư mục Mars StationName. |
| Thay đổi số lượng Mars Group khi đang chạy | Tiến trình này không tự động đổi bố cục; hãy hoàn thành tác vụ, thoát dịch vụ và khởi động lại. |
| `station_type` trong JSON khác với tham số dự án | Cho phép; mã trạm thực tế căn cứ theo file JSON. |

#### Các bước thực thi khi khởi động:

1. **Đọc Mars thực tế**: Lấy danh tính thực, status automation và `active_test_config`.
2. **Đọc bố cục kiểm tra hiện tại**: Mở cấu hình kiểm tra hiện được chọn dưới `.mars_preference`, đếm số lượng `station_layout.groups`.
3. **Ghép đường dẫn dự án**: Tham số quyết định thư mục gốc dự án; Số lượng Group quyết định `one_group` hay `two_group`.
4. **Đọc JSON / YAML**: Giữ nguyên việc xác thực StationID thực tế; `station_type` trong JSON có thể khác với tên dự án.
5. **Kiểm tra và thực thi**: Tiếp tục kiểm tra khóa file, cổng, chế độ và khớp slot Mars đầy đủ, sau đó tải trạm.

---

### 5.3 Nhật ký (log) khởi động dự kiến

```text
Mars StationName: FATP-CAM-COMBO-2
Cấu hình dự án: FATP-CAM-COMBO-1
Mars Cấu hình kiểm tra: C:\Users\Hiện tại\.mars_preference\Local.json
Mars Số lượng Group: 2
Thư mục cấu hình: ...\configs\L1\FATP-CAM-COMBO-1\two_group_FATP-CAM-COMBO-1\
station_type thực tế: Camera_two_group
```

"Thư mục cấu hình" là căn cứ quan trọng nhất để gỡ lỗi/xử lý sự cố. "`station_type` thực tế" được in ra sau khi đọc file JSON, thể hiện tên module dự kiến tải, không đại diện cho phần cứng đã khởi tạo thành công.

Thao tác chuyển đổi hiện trường: Sau khi chuyển sang giải pháp tương ứng với Type-C / NFC trong Mars, trước tiên hãy hoàn thành tác vụ cũ, thoát dịch vụ cũ, sau đó nhấp vào cùng một biểu tượng dự án; Lần khởi động này sẽ đánh giá lại số lượng Group. Không nên hiểu "tự động chọn khi khởi động" thành tự động thay đổi phần cứng và logic chương trình khi đang chạy.

---

## 06 Khởi động bằng `config`: chỉ định cố định một bộ file cấu hình

Chương này giải thích điểm vào khởi động theo file cấu hình service cố định, áp dụng cho CVC Runin và cấu hình giá làm mát.

Thích ứng trạm đặc biệt R2: `CVC Runin`. Biểu tượng RuninLog liên kết cố định với service của RuninLog; biểu tượng RuninAuto liên kết cố định với service của RuninAuto. Hai bộ cấu hình không phụ thuộc vào số lượng Group để chuyển đổi qua lại, logic chương trình thực tế do `station_type` trong JSON tương ứng quyết định.

Điểm vào cấu hình thực tế cố định lần này được sử dụng cho thiết bị `CVC Runin`: Cùng thiết bị, cùng định danh và số lượng, nhưng chạy riêng biệt hai bộ logic chương trình RuninLog / RuninAuto.

```powershell
python run.py --config "configs/L1/RuninLog/service/DEVICE_01.json"
```

---

### 6.1 Cách tìm kiếm file đi kèm

- Chỉ định: `.../RuninLog/service/DEVICE_01.json`
- Đồng bộ: `.../RuninLog/station/DEVICE_01.yaml`
- Thực tế: `.../RuninLog/MarsProfileConfig.json`

Đường dẫn tương đối được giải thích theo thư mục làm việc khi khởi động, khuyến nghị thực thi tại thư mục gốc dự án hoặc sử dụng đường dẫn tuyệt đối. Tên file JSON phải bằng `station_id` bên trong nó. Sau khi chỉ định file, không tìm thư mục cấu hình theo số lượng Group nữa.

---

### 6.2 Phân nhánh Thực tế / Ảo

Hai bộ cấu hình của CVC Runin lần này sử dụng nhánh Mars thực tế, `virtual` mặc định trống hoặc là `false`. CVC giá làm mát sử dụng nhánh ảo, xem chương 07. Trạm chưa đánh dấu tiếp tục khởi động thông thường; không đổi điểm vào chỉ vì framework có khả năng `--config`.

| Cấu hình của service | Thực thi thực tế |
| --- | --- |
| `MarsConfig.virtual` mặc định trống hoặc `false` | Đọc cấu hình chỉ định trước, sau đó đối chiếu danh tính Mars thực, automation và cấu hình kiểm tra của user hiện tại; vẫn cần `MarsProfileConfig.json` cùng thư mục. |
| `MarsConfig.virtual: true` | Sử dụng cấu hình chỉ định để tạo chế độ kiểm tra ảo, kiểm tra khóa thể hiện này và cổng nghiệp vụ, chạy trực tiếp dịch vụ; không đọc file Mars thực, không tạo kết nối Mars thực. |

---

### 6.3 CVC Runin: Cùng một thiết bị, khởi động riêng RuninLog / RuninAuto

```text
configs/L1/
├─ RuninLog/
│  ├─ service/DEVICE_01.json  # station_type: RuninLog
│  ├─ station/DEVICE_01.yaml
│  └─ MarsProfileConfig.json
└─ RuninAuto/
   ├─ service/DEVICE_01.json  # station_type: RuninAuto
   ├─ station/DEVICE_01.yaml
   └─ MarsProfileConfig.json
```

Hai bộ file giữ lại cùng một StationID thực tế và bố cục cần thiết giống nhau, mỗi bên tự điền loại trạm thực tế và các tham số phần cứng đi kèm. RuninLog, RuninAuto dùng chung `client_id` và cổng nghiệp vụ ban đầu theo thỏa thuận hiện tại, luân phiên chạy; sau khi thoát một bộ logic mới nhấp vào biểu tượng khác, không chạy đồng thời hai dịch vụ Mars thực tế.

```powershell
python run.py --config "configs/L1/RuninLog/service/DEVICE_01.json"
python run.py --config "configs/L1/RuninAuto/service/DEVICE_01.json"
```

Hai tham số loại trừ lẫn nhau: `--config` và `--station-type` không thể viết trong cùng một lệnh khởi động. Phương án cấu hình cố định không cần tiền tố `one_group` / `two_group`; Các thư mục có các tiền tố này cũng có thể sử dụng trực tiếp `--config` để chỉ định service bên trong.

---

## 07 IMUCLocal: đếm thời gian làm mát bằng Mars ảo

Chương này giải thích quan hệ triển khai, cấu hình mẫu, quy trình đếm thời gian và hành vi trang giao diện của dịch vụ Mars ảo trên giá làm mát.

Thích ứng trạm đặc biệt R3: `CVC giá làm mát`. Nguồn gốc yêu cầu: Bản thân giá làm mát không có Mars, chỉ có thể gắn phụ thuộc trên máy tính của thiết bị lân cận. Sử dụng dịch vụ `IMUCLocal` độc lập, dùng `--config` cố định và `virtual=true`; không kết nối với Mars thực của thiết bị lưu host, không thay thế tiến trình nghiệp vụ gốc của máy host.

Quan hệ triển khai —— giá làm mát mượn máy tính thiết bị lân cận:

| Vị trí chạy | Dịch vụ gốc trên host | Dịch vụ làm mát thêm mới |
| --- | --- | --- |
| Máy tính thiết bị lân cận | Tiếp tục sử dụng phương thức khởi động bình thường gốc, Mars thực và logic trạm cũ. | Chạy thêm tiến trình `IMUCLocal` độc lập, cung cấp mô phỏng thời gian kiểm tra, màn hình đếm thời gian và xử lý hoàn tất làm mát. |
| Cách ly thể hiện (Instance) | Giữ lại `client_id`, cổng nghiệp vụ và thuộc tính phần cứng gốc của host. | Sử dụng `client_id` độc lập, cổng nghiệp vụ khác, và cấu hình slot / phần cứng của riêng giá làm mát. |

Các cấu hình mẫu hiện có dưới đây dùng để giải thích hình thức cấu hình, cổng, thời lượng, danh tính điều phối và điểm phần cứng vẫn cần đối chiếu theo cấu hình hiện trường của giá làm mát CVC. `IMUCLocal` tiếp tục sử dụng quy trình quản lý vật tư, giao diện điều phối và lưu kết quả của framework. Mars ảo chỉ mô phỏng thời lượng kiểm tra và trả về `PASS`. Khi hoàn tất làm mát, trạm chuyển trạng thái thành `nextStep=12`.

---

### 7.1 Vị trí cấu hình hiện có trong repository

```text
configs/L1/IMUCLOAL/
├─ service/IMUC_LOCAL_01.json
└─ station/IMUC_LOCAL_01.yaml
```

`IMUCLOAL` là chính tả thư mục thực tế hiện tại; tên module là `IMUCLocal`, không nhầm lẫn giữa hai tên này. Sử dụng lệnh dưới đây mà không cần đổi tên thư mục:

```powershell
python run.py --config "configs/L1/IMUCLOAL/service/IMUC_LOCAL_01.json"
```

---

### 7.2 Tham số mẫu hiện tại

| Tham số | Giá trị hiện tại / Ý nghĩa |
| --- | --- |
| `station_type` | `IMUCLocal`: Hiển thị trang làm mát Tkinter |
| `station_id` / `client_id` | `IMUC_LOCAL_01` / `IMUC_1`: Định danh thiết bị và định danh điều phối là 2 trường khác nhau |
| Cổng nghiệp vụ / Thời gian làm mát | `9908` / `30` giây |
| Bố cục / Chế độ phần cứng | 8 nhóm, mỗi nhóm 1 Slot; 2 hàng × 4 cột; `simulation` trong YAML là `true` |
| Địa chỉ ATCS | `127.0.0.1:12345` là cấu hình demo hiện tại; hiện trường phải đổi thành địa chỉ điều phối, dây chuyền và công đoạn thực tế |

---

### 7.3 Ví dụ file cấu hình service hoàn chỉnh

Dưới đây nhất quán với các trường quan trọng của mẫu hiện tại, chủ yếu dùng để hiểu cấu hình. Hãy đổi địa chỉ và định danh thành giá trị hiện trường trước khi sử dụng; không dùng trực tiếp địa chỉ điều phối demo làm địa chỉ hiện trường.

```json
{
  "client_id": "IMUC_1",
  "line_id": "CFN4",
  "section_id": "IMUT",
  "service_prot": 9908,
  "station_type": "IMUCLocal",
  "ATCS_ip": "127.0.0.1",
  "ATCS_port": 12345,
  "station_id": "IMUC_LOCAL_01",
  "MarsConfig": {
    "virtual": true,
    "virtual_test_time": 30,
    "group_slot_list": [1, 1, 1, 1, 1, 1, 1, 1],
    "router_enable": true,
    "result": {},
    "next_step": {},
    "extra_info": {}
  },
  "holderManage": {
    "row": 2,
    "col": 4,
    "group_list": [0, 1, 2, 3, 4, 5, 6, 7],
    "slot_list": [0, 0, 0, 0, 0, 0, 0, 0]
  }
}
```

---

### 7.4 Ví dụ file cấu hình station đi kèm

```yaml
station_name:
  mars_id: IMUC_LOCAL_01
  simulation: true
  plc_ip: 127.0.0.1
  control_adder:
    dut_1_status: 0
    dut_2_status: 1
    dut_3_status: 2
    dut_4_status: 3
    dut_5_status: 4
    dut_6_status: 5
    dut_7_status: 6
    dut_8_status: 7
```

Chế độ PLC thực tế điền IP thực và các điểm cảm biến; Control của `IMUCLocal` hiện tại sử dụng ánh xạ cảm biến 8 đường của M28. Không tự cho rằng phần cứng thực tế hỗ trợ thêm cảm biến chỉ bằng cách tăng số nhóm cấu hình.

---

### 7.5 Hai công tắc lần lượt kiểm soát cái gì

| Cấu hình | File chứa | Tác dụng |
| --- | --- | --- |
| `MarsConfig.virtual: true` | service JSON | Không kết nối Mars thực, sử dụng đếm thời gian kiểm tra ảo. Điểm vào ảo hoàn toàn offline sử dụng `--config`. |
| `MarsConfig.virtual_test_time: 30` | service JSON | Mỗi dịch vụ một thời lượng kiểm tra thống nhất, các Group đếm thời gian độc lập; phải là số dương hữu hạn, đơn vị giây. |
| `simulation: true` | station YAML | `IMUCLocal` không kết nối PLC, cảm biến vật tư sử dụng cache điều phối hiện có. Không tự động mô phỏng nạp liệu ATCS hay tay máy (robot arm). |
| `simulation: false` | station YAML | `IMUCLocal` đọc tín hiệu vật tư PLC thực tế; Mars vẫn có thể là ảo. |

---

### 7.6 Quy trình đếm thời gian và thực thi kết quả

1. **Điều phối nạp liệu (Loading)**: Cung cấp thông tin vật tư hợp lệ theo giao thức điều phối ban đầu và giải phóng khóa robot / điều phối.
2. **Phát hiện vật tư mới và lấy khóa nhóm (Group Lock)**: Trạm tiếp tục quy trình đánh giá vật tư và khóa ban đầu; chỉ mở cửa sổ giao diện sẽ không bắt đầu đếm thời gian.
3. **Kích hoạt bắt đầu kiểm tra**: Hàm `mars_test_start` ban đầu gọi `trigger_test_start`, Mars ảo ghi lại thời điểm kết thúc đồng hồ đơn điệu (monotonic clock) của nhóm này.
4. **Đếm giờ độc lập từng nhóm**: Trang Tkinter hiển thị dồn tích số giây đã dùng; các nhóm có thể bắt đầu lệch giờ, thời gian hoàn thành không đè lên nhau.
5. **Nhận PASS ảo**: Sau khi đạt thời lượng, nghiệp vụ sẽ xử lý thông qua vòng lặp kiểm tra định kỳ (polling), không phụ thuộc vào việc làm mới cửa sổ Tkinter để kích hoạt hoàn thành.
6. **Hoàn thành làm mát và báo cáo**: Trạm thiết lập `nextStep=12`, `testStatus=1`, tiếp tục dùng `add_holder_test_data` để lưu kết quả, sau đó mở khóa và báo cáo (report).

---

### 7.7 Giao diện, thao tác đóng và khởi động lại

- **Giao diện làm mát Tkinter**: Tự động mở khi khởi động, hiển thị trạng thái vị trí làm mát 8 nhóm dạng 2x4. Mỗi nhóm hiển thị mã vật tư và thời gian đã trôi qua.
- **Tính toán kết quả độc lập**: Không sử dụng biến tổng thể để xác định trạng thái làm mát, tránh trường hợp một nhóm hoàn thành làm ảnh hưởng đến các nhóm khác. Mỗi nhóm có trạng thái riêng.
- **Tắt ngang ứng dụng**: Khi đóng cửa sổ, tiến trình nghiệp vụ sẽ bị hủy, dữ liệu đếm thời gian chưa hoàn thành trong bộ nhớ đệm sẽ bị giải phóng.
- **Xử lý sự cố / Reset**: Nếu phát sinh lỗi, hệ thống sẽ gọi `set_error` để dọn dẹp trạng thái và phát tín hiệu cảnh báo cho hệ thống điều phối.
- **Khởi động lại**: Sau khi khởi động lại dịch vụ và nạp lại vật tư, quy trình đếm thời gian sẽ được tính lại từ đầu với đầy đủ thời lượng, không đếm tiếp thời gian đã chạy trước đó.

`scan_sn` ảo hiện tại được cố định là `false` trong code: `TestMain.py` $\rightarrow$ `virtual_profiles()` tạo các profile cho chế độ ảo đều dùng `scan_sn=False`. Chỉ sửa `MarsConfig.scan_sn` trong service JSON sẽ không đổi giá trị này. `scan_sn` của Mars thực lấy từ cấu hình kiểm tra thực tế. Tắt flag này không sửa lỗi ánh xạ Group/Slot ở tầng nghiệp vụ, cũng không thay thế thông tin vật tư.

---

## 08 Chạy nhiều dịch vụ trên cùng máy và quy trình chuyển đổi

Phạm vi áp dụng lô này: `RuninLog` / `RuninAuto` của CVC Runin là 2 bộ logic chạy luân phiên trên cùng 1 thiết bị; CVC giá làm mát thêm dịch vụ ảo độc lập trên máy tính thiết bị lân cận, có thể chạy đồng thời với dịch vụ gốc của máy host. Các trạm khác chưa đánh dấu giữ nguyên khởi động thông thường.

| Kết hợp | Thỏa thuận hiện tại | Yêu cầu Định danh / Cổng (Port) |
| --- | --- | --- |
| Dự án thực A $\leftrightarrow$ Dự án thực B | Chạy luân phiên | Kế thừa `client_id`, cổng nghiệp vụ cũ; dùng khóa hiện tại để chống mở nhiều instance. |
| Dịch vụ thực A + IMUCLocal ảo | Cho phép chạy đồng thời | Khác `client_id`, khác cổng nghiệp vụ, phần cứng / slot độc lập. |
| Hai dịch vụ Mars thực cùng kết nối tới một Mars | Không thuộc phương án dịch vụ kép hiện tại | Không thể chỉ sửa cổng nghiệp vụ mà cho rằng một Mars hỗ trợ nhiều client thực tế. |

---

### 8.1 Tại sao chỉ sửa Cổng (Port) là chưa đủ

`client_id` đồng thời ảnh hưởng đến `stationCode` của ATCS, khóa file (file lock), cơ sở dữ liệu và bộ nhớ đệm chế độ. Việc hai tiến trình độc lập sử dụng cùng một `StationID` không bị cấm, nhưng các phiên chạy (instance) chạy đồng thời bắt buộc phải có `client_id` và cổng nghiệp vụ riêng biệt. Phía điều phối cần cấu hình `stationCode` tương ứng cho dịch vụ làm mát ảo; không cần sửa mã nguồn routing của ATCS.

---

### 8.2 Quy trình chuyển đổi Dự án thực / Bố cục Group

1. Hoàn thành kiểm tra hiện tại và xác nhận kết quả đã được xử lý theo quy trình cũ.
2. Hoàn thành dỡ liệu (unloading), lấy hết vật tư ra, kết thúc việc chiếm khóa điều phối.
3. Thoát dịch vụ cũ, xác nhận tiến trình tương ứng đã kết thúc, cổng đã được giải phóng.
4. Khi cần thay đổi số lượng Group, hãy chọn cấu hình kiểm tra tương ứng trong Mars.
5. Nhấp vào biểu tượng dự án mục tiêu hoặc thực thi lệnh tương ứng.
6. Đối chiếu log về dự án cấu hình, số lượng Group, `station_type` thực tế và đường dẫn đầy đủ.

Các dự án thực dùng chung `client_id` sẽ tiếp tục dùng chung bộ nhớ đệm hiện có, không thêm cơ chế migration (di chuyển) tác vụ. Không tự động thay đổi đối tượng phần cứng hoặc bố cục khi đang chạy.

---

### 8.3 Chạy đồng thời Dịch vụ thực và Dịch vụ làm mát

1. Đối chiếu riêng biệt hai `client_id` và hai cổng.
2. Xác nhận riêng thuộc tính phần cứng và định danh trạm điều phối.
3. Khởi động dịch vụ thực trước, sau đó khởi động `IMUCLocal`; hoặc ngược lại tùy yêu cầu hiện trường.
4. Xác minh riêng việc nạp liệu, kết quả và báo cáo trạng thái của từng bên, đảm bảo không gửi vật tư nhầm sang trạm khác.
5. Khi đóng cửa sổ làm mát, xác nhận dịch vụ thực vẫn đang chạy bình thường.

---

## 09 Tạo và quản lý biểu tượng trên màn hình nền

Chương này cung cấp lệnh tạo, phương pháp chỉ định trình thông dịch (interpreter) và các lưu ý cập nhật cho các loại điểm vào trên desktop.

Chỉ thêm điểm vào chuyên dụng cho các trạm đặc biệt được đánh dấu: CVC 1upCombo chọn 1 biểu tượng dự án theo Group; CVC Runin có 2 biểu tượng cấu hình cố định `RuninLog`, `RuninAuto`; CVC giá làm mát có 1 biểu tượng `IMUCLocal` ảo. Các trạm chưa đánh dấu giữ nguyên biểu tượng khởi động thông thường ban đầu.

| Biểu tượng / Trạm áp dụng | Phương thức liên kết | Tác dụng tại hiện trường |
| --- | --- | --- |
| Trạm thông thường | Không truyền `--station-type` / `--config` | Mặc định sử dụng `python run.py` |
| CVC 1upCombo | `--station-type <Tên dự án hiện trường>` | Tự động chọn cấu hình tương ứng 1up Type-C / 2up NFC khi khởi động |
| CVC RuninLog | `--config` File service của RuninLog | Thiết bị Runin chạy logic RuninLog |
| CVC RuninAuto | `--config` File service của RuninAuto | Thiết bị Runin chạy logic RuninAuto |
| CVC giá làm mát | `--config` File service làm mát (có `virtual=true`) | Chạy dịch vụ làm mát độc lập trên máy tính thiết bị lân cận |

---

### 9.1 Điểm vào cài đặt gốc: Cài đặt môi trường và tạo biểu tượng mặc định

```powershell
python install_autoservice.py
```

Biểu tượng AutoService mặc định cuối cùng sẽ thực thi `run.py` không tham số, sử dụng quy tắc khởi động thông thường.

---

### 9.2 Chỉ tạo biểu tượng: Không cài đặt dependency hoặc khởi động nghiệp vụ

Thực thi trước tại đúng thư mục gốc dự án, bằng trình thông dịch Python mục tiêu. Tên chỉ là nhãn hiển thị trên desktop, không tham gia vào quá trình chọn cấu hình.

| Loại biểu tượng | Tham số liên kết | Hành vi thực tế |
| --- | --- | --- |
| Điểm vào thông thường | Chỉ có `--create-shortcut "Tên"` | `run.py` không tham số |
| Điểm vào dự án tự động Group | `--station-type <Tên dự án>` | Thư mục gốc dự án + `one_group` / `two_group` |
| Điểm vào cấu hình thực cố định | `--config <File service>` | File cố định, khi `virtual` trống / `false` sẽ kết nối Mars thực |
| Điểm vào làm mát IMUC | `--config <File service làm mát>` | File cố định, `virtual=true`, `station_type=IMUCLocal` |

```powershell
python install_autoservice.py --create-shortcut "Trạm gốc"

python install_autoservice.py --create-shortcut "AutoService" --station-type FATP-CAM-COMBO-2

python install_autoservice.py --create-shortcut "RuninLog" --config "configs/L1/FATP-RUNIN-LOG/RunInLog/service/DEVICE_01.json"

python install_autoservice.py --create-shortcut "CVC RuninAuto" --config "configs/L1/FATP-RUNIN-LOG/RunInAuto/service/DEVICE_01.json"

python install_autoservice.py --create-shortcut "CVC giá làm mát" --config "configs/L1/IMUCLOAL/service/IMUC_LOCAL_01.json"
```

---

### 9.3 Chỉ định trình thông dịch Python (Python Interpreter)

Biểu tượng sẽ liên kết với `sys.executable` tại thời điểm tạo. Trong PowerShell, đối với đường dẫn file thực thi có chứa khoảng trắng, hãy thêm ký tự gọi `&` ở đầu:

```powershell
& "C:\Python310\python.exe" install_autoservice.py --create-shortcut "CVC 1upCombo" --station-type FATP-CAM-COMBO-1
```

Đường dẫn Python ở trên là ví dụ, hãy thay bằng trình thông dịch thực tế tồn tại trên máy. Thao tác "chỉ tạo biểu tượng" sẽ không kiểm tra hay cài đặt đầy đủ bộ dependency, do đó nên khởi động thủ công bằng lệnh để xác minh trước.

---

### 9.4 Các lưu ý khi liên kết và cập nhật biểu tượng

- Windows tạo file `.lnk`, Linux tạo file `.desktop`.
- Tham số `--config` nên lưu dưới dạng đường dẫn tuyệt đối khi tạo biểu tượng để tránh lỗi đường dẫn làm việc.
- Khi cập nhật cấu hình file JSON / YAML, không cần tạo lại biểu tượng trừ khi thay đổi đường dẫn file cấu hình service.
- Không dùng lẫn lộn điểm vào của thư mục mã nguồn (source) và thư mục đóng gói (`dist`).
- Đảm bảo môi trường Python được chỉ định có đủ các thư viện phụ thuộc cần thiết.
- Khóa file và cổng nghiệp vụ sẽ giải phóng khi ứng dụng đóng hoàn toàn; nếu không khởi động được biểu tượng, hãy kiểm tra tiến trình ngầm.

---

### 9.5 Sử dụng trực tiếp script Batch để debug

Trong Windows, tham số đầu tiên của `AutoService.bat` là đường dẫn Python, tiếp theo mới là các tham số nghiệp vụ. Không đặt `--station-type` vào vị trí tham số đầu tiên.

```powershell
& ".\AutoService.bat" "C:\Python310\python.exe" --station-type FATP-CAM-COMBO-1
```

Script khởi động tinh gọn của `IMUCLocal` cũng có thể sử dụng, tham số của nó là đường dẫn service:

```powershell
& ".\StationControl\IMUCLocal\IMUCLocal.bat" "E:\项目\P27\AutoService 模拟专用\configs\L1\IMUCLOAL\service\IMUC_LOCAL_01.json"
```

---

### 9.6 Lệnh tham khảo cho Linux

```bash
python3 -u run.py --station-type FATP-CAM-COMBO-1

python3 install_autoservice.py --create-shortcut "CVC 1upCombo" --station-type FATP-CAM-COMBO-1

python3 install_autoservice.py --create-shortcut "CVC giá làm mát" --config "configs/L1/IMUCLOAL/service/IMUC_LOCAL_01.json"
```

Công cụ tạo shortcut trên Linux cần `bash` cũng như `GNOME Terminal` hoặc `Ptyxis`, môi trường desktop có thể cần cấp quyền "Allow Launching". `AutoService.sh` sẽ thử mở terminal độc lập và kết thúc shell tương tác gọi nó; khi debug có thể dùng trực tiếp lệnh `python3`. Giao diện đồ họa Linux và phần cứng vẫn cần xác minh tại hiện trường.

---

## 10 Phân biệt WebUI với điểm khởi chạy dịch vụ nghiệp vụ

WebUI là một chương trình giám sát (monitoring) độc lập, không phải là trang làm mát Tkinter của `IMUCLocal`, cũng không phải là lớp cơ sở khởi động nghiệp vụ mới.

```powershell
python -m WebUI.launch_ui
python -m WebUI.launch_ui --no-browser --port 18081
```

| Mục | Hành vi thực tế hiện tại |
| --- | --- |
| Địa chỉ lắng nghe (Listen Address) | Chỉ localhost `127.0.0.1`; Cổng UI mặc định `18080`, cổng nghiệp vụ lấy từ `service_prot` |
| Tìm kiếm cấu hình | Định vị trạm theo Mars StationName của user hiện tại; Ưu tiên dùng service gốc nếu có, nếu không sẽ tìm theo Group hiện tại trong thư mục con của trạm đó |
| Tham số lệnh hiện tại | Chỉ có `--port`, `--no-browser`; Chưa thêm tham số chọn dự án `--station-type` / `--config` cho UI |
| Thích ứng chuyển đổi dự án | Điểm vào nghiệp vụ đã hỗ trợ chuyển dự án theo tham số, lựa chọn dự án tương tự trên UI chưa triển khai trong đợt này; Không thể giả định UI tự động đi theo tham số của terminal khác |
| Kiểm tra trước khi khởi động | Kiểm tra service và các file đi kèm đã tải bởi UI; Không kết nối phần cứng, không lấy khóa nghiệp vụ |
| So sánh bố cục | Hàng/Cột, Group, Slot của Holder được kiểm tra riêng; Bố cục test Mars chỉ so sánh với `service.MarsConfig.group_slot_list`, không bắt buộc bằng bố cục Holder |
| Tải lại (Reload) | Tải lại thủ công khi file cấu hình hoặc bố cục chọn thay đổi; Xóa snapshot cũ khi thay đổi, giữ lại cấu hình lần trước và cảnh báo khi thất bại |

Khi làm việc đa dự án, kết quả căn cứ theo log khởi động nghiệp vụ: Nghiệp vụ dùng `--station-type` chọn dự án A, nhưng Mars StationName vẫn trỏ tới trạm B, UI hiện tại vẫn có thể kiểm tra trạm B. Hãy xác nhận thư mục gốc chạy terminal / biểu tượng và đường dẫn cấu hình đầy đủ hiển thị trên UI; Không đổi StationID thực tế chỉ vì UI không tìm thấy file ở thư mục khác. Nếu cần UI hỗ trợ chọn giữa nhiều dự án, chức năng này phải được triển khai riêng.

Sau khi sửa file Python hoặc file giao diện của WebUI, cần khởi động lại WebUI và làm mới trang web; Chỉ chọn "Tải lại cấu hình" (Reload config) sẽ không tải mã nguồn chương trình mới. Đóng WebUI không tự động kết thúc dịch vụ nghiệp vụ.

---

## 11 Log và cách xử lý các sự cố thường gặp

Chương này đưa ra các điểm bắt đầu kiểm tra và hướng xử lý theo hiện tượng sự cố, kèm theo các lệnh chỉ đọc (read-only) để kiểm tra và vị trí file log.

| Hiện tượng | Kiểm tra cái gì trước | Hướng xử lý |
| --- | --- | --- |
| Tìm nhầm thư mục dự án / Thư mục làm việc | Tham số khởi động, thư mục làm việc của lệnh, mục tiêu biểu tượng / Điểm vào thông thường xem Mars StationName; Điểm vào dự án xem `--station-type`; Điểm vào cố định xem đường dẫn tuyệt đối `--config` | Xác nhận chính xác tham số và đường dẫn chạy |
| Xuất hiện đường dẫn kiểu `...COMBO-2/two_group_COMBO-1` | Có đang chạy mã nguồn cũ hoặc bản sao `dist` cũ hay không | Cả hai cấp thư mục của điểm vào dự án đều được xác định bởi tham số; Kiểm tra `run.py` thực sự đang chạy trước |
| Không tìm thấy file cấu hình service / station | Đường dẫn đầy đủ, tên file, `station_id` và `mars_id` trong YAML | Tên file giữ nguyên mã số thiết bị thực tế; Thay đổi thư mục dự án không đổi định danh thiết bị |
| Chỉ đặt thư mục `one_group` / `two_group`, khởi động thông thường thất bại | Có phải đang dùng `run.py` không tham số không | Khởi động thông thường của nghiệp vụ không quét các thư mục này, chuyển sang dùng điểm vào dự án hoặc chỉ định `--config` cố định |
| Báo lỗi định danh thiết bị không đồng nhất | Định danh Mars của user hiện tại và định danh trong file chỉ định | Thống nhất mã số thiết bị thực tế và chữ hoa/thường, không bỏ qua bằng cách nới lỏng kiểm tra định danh |
| Báo lỗi đang chạy / Cổng (port) bị chiếm dụng | `client_id`, tiến trình cũ và cổng lắng nghe tương ứng | Xác nhận dịch vụ cũ đã kết thúc chưa; Các thể hiện làm mát chạy đồng thời phải dùng định danh và cổng độc lập |
| Bố cục Mars không khớp | Số lượng slot mỗi Group trong cấu hình test hiện tại với `service.MarsConfig.group_slot_list` | Mảng đầy đủ và thứ tự của cả hai phải nhất quán; Không lấy số lượng slot cache của Holder làm bố cục Mars |
| Cấu hình kiểm tra Mars báo lỗi | Các file ánh xạ bởi `MarsProfileConfig`, chế độ test, `use_trigger` | Không thể chỉ đối chiếu `Local.json`; Khởi động gốc sẽ quét các file test và yêu cầu cấu hình ánh xạ phải hợp lệ |
| Trang làm mát dừng ở trạng thái chờ | Nạp liệu hợp lệ, bộ nhớ đệm thông tin vật tư, khóa robot, khóa Group và `trigger_test_start` | Giao diện UI không phải công cụ tạo vật tư thủ công; Kích hoạt test theo đúng quy trình điều phối gốc |
| Dịch vụ ảo vẫn yêu cầu file Mars | Có dùng `--config` không, `virtual` trong JSON có phải là `true` boolean không | Không viết chuỗi `"true"`; Điểm vào thực thông thường / dự án không tự động biến thành điểm vào offline hoàn toàn |
| Vật tư PLC không khớp với cache | Điểm I/O thực tế, trạng thái có vật tư thực tế, thông tin nạp/dỡ liệu của hệ thống điều phối | Đối chiếu theo quy trình nghiệp vụ gốc, không xóa CSDL để che giấu trạng thái vật tư |
| Làm mát hoàn thành nhưng kết quả boolean chung là `false` | `nextStep=12`, `testStatus=1`, giai đoạn nghiệp vụ trên trang và dữ liệu gốc | Đánh giá theo giao thức hoàn thành làm mát, không chỉ căn cứ vào `testResult` chung để kết luận làm mát thất bại |
| Chỉ riêng WebUI báo thiếu file ở dự án khác | Thư mục gốc của WebUI và Mars StationName mà nó đang dùng | Loại trừ trường hợp chạy nhầm thư mục `dist` / thư mục cũ; UI hiện tại không nhận tham số chọn dự án của nghiệp vụ |

---

### 11.1 Lệnh kiểm tra chỉ đọc (Read-only) trên Windows

```powershell
Get-NetTCPConnection -State Listen -LocalPort 9909 -ErrorAction SilentlyContinue | Select-Object LocalAddress, LocalPort, OwningProcess

Get-NetTCPConnection -State Listen -LocalPort 9908 -ErrorAction SilentlyContinue | Select-Object LocalAddress, LocalPort, OwningProcess
```

Xem tiến trình chỉ định, thay PID ví dụ bằng kết quả truy vấn được:

```powershell
Get-Process -Id 1234
```

Khi cổng nghiệp vụ đã ở trạng thái Listen, có thể đọc API trạng thái bộ nhớ; khi chưa sẵn sàng có thể trả về lỗi, không thể vì thế mà kết luận phần cứng bị hỏng:

```powershell
Invoke-RestMethod "http://127.0.0.1:9909/autoService/ui/state"
```

---

### 11.2 Vị trí File Log và Dữ liệu thể hiện (Instance Data)

`Thư mục người dùng hiện tại/`
- `AutoServiceLog/`
  - `<station_type thực tế>/<Ngày>/*.info.log, *.error.log`
- `.auto_service/`
  - `<client_id>_service.lock`
  - `<client_id>.db`
  - `<client_id>_mode.json`

Khóa file (file lock) do tiến trình nắm giữ, sẽ được hệ điều hành giải phóng sau khi thoát bình thường hoặc crash. File lock vẫn tồn tại không đồng nghĩa với việc khóa vẫn đang bị chiếm dụng. Hãy kiểm tra tiến trình và cổng trước, không tự ý xóa cơ sở dữ liệu vật tư để xử lý lỗi chạy nhiều phiên bản ứng dụng.

---

## 12 Danh sách kiểm tra và nghiệm thu tại hiện trường

Trước tiên hãy xác nhận trạm có thuộc danh sách trạm đặc biệt trong đợt này hay không.

### 12.1 Mỗi lần khởi động

- **CVC 1upCombo**: Kiểm tra số lượng Group trong cấu hình Mars tương ứng với giải pháp 1up Type-C / 2up NFC lần lượt là 1 / 2.
- **CVC Runin**: Hai biểu tượng RuninLog và RuninAuto có logic chạy khác nhau, luân phiên sử dụng theo thỏa thuận.
- **CVC giá làm mát**: Không cần kết nối Mars thực, kiểm tra định danh `IMUCLocal` và cổng được cách ly với host.
- Xác nhận chạy đúng bản sao dự án và môi trường Python mong muốn.
- Kiểm tra log khởi động để đảm bảo không có lỗi nạp cấu hình.
- Đảm bảo các cổng nghiệp vụ tương ứng không bị chiếm dụng bởi tiến trình lạ.
- Xác nhận các khóa file cũ đã được dọn dẹp hoặc giải phóng.
- Kiểm tra trạng thái kết nối PLC / ATCS nếu chạy chế độ thực tế.
- Đảm bảo `client_id` là duy nhất cho mỗi dịch vụ chạy đồng thời.
- Xác minh giao diện người dùng hiển thị đúng thông số thiết bị.

---

### 12.2 Nghiệm thu chuyển đổi Dự án / Group

- Chuyển đổi thành công giữa các cấu hình dự án mà không bị lẫn lộn dữ liệu.
- Định danh trạm Mars StationName và StationID giữ nguyên tính chính xác.
- Logic kiểm tra tương ứng với 1up (1 Group) và 2up (2 Group) được tải đúng.
- Trạng thái đếm thời gian và lưu log hoạt động chuẩn xác sau khi chuyển đổi.
- Không phát sinh lỗi xung đột tài nguyên khi chuyển đổi qua lại.

---

### 12.3 Nghiệm thu Dịch vụ kép và Làm mát

- Dịch vụ làm mát `IMUCLocal` và dịch vụ chính chạy song song ổn định, không chiếm dụng cổng của nhau.
- Các Group đếm thời gian độc lập, không bị ảnh hưởng lẫn nhau khi nạp/dỡ liệu lệch giờ.
- Kết quả làm mát hoàn thành trả về đúng `nextStep=12` và `testStatus=1`.
- Giao diện Tkinter hiển thị chính xác thời gian trôi qua cho từng vị trí làm mát.
- Khi tắt dịch vụ làm mát ảo, dịch vụ trạm chính vẫn tiếp tục vận hành bình thường.

---

### 12.4 Lệnh Regression Test cho bảo trì

```powershell
python -B -m unittest discover -s WebUI/tests -v
node WebUI/tests/test_frontend.js
```

Các lệnh này dành cho nhân sự bảo trì kiểm tra lại mã nguồn, không thay thế cho việc nghiệm thu an toàn thiết bị. Sổ tay này căn cứ theo quy tắc đối chiếu file hiện tại, không tuyên bố toàn bộ các test hiện có đã pass, cũng chưa thực hiện liên điều chỉnh (commissioning) Mars / PLC / ATCS tại hiện trường.

---

## 13 Tra cứu nhanh lệnh và vị trí mã nguồn

| Trạm áp dụng / Tác vụ | Lệnh / Phương thức |
| --- | --- |
| Tất cả các trạm chưa đánh dấu khác: Khởi động thông thường | `python run.py` |
| CVC 1upCombo: Tự động chọn theo Group khi khởi động 1up Type-C / 2up NFC | `python run.py --station-type <Tên dự án cấu hình hiện trường>` |
| CVC RuninLog: Khởi động cấu hình cố định | `python run.py --config "configs/L1/RuninLog/service/DEVICE_01.json"` |
| CVC RuninAuto: Khởi động cấu hình cố định | `python run.py --config "configs/L1/RuninAuto/service/DEVICE_01.json"` |
| CVC giá làm mát: Khởi động mẫu IMUCLocal | `python run.py --config "configs/L1/IMUCLOAL/service/IMUC_LOCAL_01.json"`; Trong cấu hình có `virtual=true` |
| CVC 1upCombo: Tạo biểu tượng dự án | `python install_autoservice.py --create-shortcut "CVC 1upCombo" --station-type <Tên dự án cấu hình hiện trường>` |
| CVC Runin / giá làm mát: Tạo biểu tượng cấu hình cố định | `python install_autoservice.py --create-shortcut "Tên biểu tượng" --config "Đường dẫn file cấu hình service tương ứng"` |
| Khởi động cửa sổ giám sát WebUI (giữ quy tắc cũ) | `python -m WebUI.launch_ui` |
| Chỉ mở dịch vụ HTTP giám sát WebUI | `python -m WebUI.launch_ui --no-browser --port 18081` |

---

### Vị trí mã nguồn tương ứng với từng chức năng

| Nội dung | Vị trí định vị trong Source Code |
| --- | --- |
| Tham số loại trừ lẫn nhau và Điểm vào tổng | `run.py` |
| Đường dẫn thông thường / dự án, cấu hình cố định, profile ảo | `TestMain.py` trong các hàm `check_mars`, `load_service_config`, `virtual_profiles`, `main` |
| Cấu hình kiểm tra Mars hiện tại và Phân tích Group/Slot | `ServiceFramework/MarsManage/ReadMarsConfig.py` |
| Tải trạm thực tế theo JSON type, Chọn trang Tkinter | `InnorevService.py` |
| Mars Thực / Ảo và Đếm giờ độc lập | `ServiceFramework/MarsManage/MarsService.py` |
| IMUCLocal Nạp/dỡ liệu, đếm thời gian, kết quả làm mát và UI | `StationControl/IMUCLocal/IMUCLocalService.py`<br>`StationControl/IMUCLocal/IMUCLocalControl.py`<br>`StationControl/IMUCLocal/ui.py` |
| Tạo biểu tượng và Điểm vào chỉ tạo biểu tượng | `install_autoservice.py`, đi kèm `AutoService.bat` / `AutoService.sh` |
| WebUI đọc thông tin và Kiểm tra trước khi khởi động | `WebUI/launch_ui.py`, `WebUI/preflight.py` |

Ngày sửa đổi: 06-11-2026 (múi giờ Asia/Shanghai). Phiên bản này đánh dấu nguồn gốc và phạm vi áp dụng cho CVC 1upCombo, CVC Runin, CVC giá làm mát theo các yêu cầu bổ sung, các trạm chưa được đánh dấu khác thống nhất dùng khởi động thông thường. Hành vi khởi động vẫn lấy mã nguồn hiện tại đã kiểm tra trong sổ tay gốc làm chuẩn. Sổ tay chỉ phục vụ mục đích đọc tài liệu, không thay đổi mã nguồn nghiệp vụ, cấu hình chính thức, biểu tượng màn hình hoặc bản sao đóng gói `dist`.
```
