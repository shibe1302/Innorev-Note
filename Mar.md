### 1.Cài Mar bị lỗi (Ubuntu)
Bước 1: Vào thư mục bộ cài
bash
```
cd "$HOME/mars_1.0.0_585_cte_lab_20260330.075918(for
vendor)/mars_installer"
```
Bước 2: Bật tùy chọn bỏ chặn trên terminal hiện tại
Bash
```
export PIP_BREAK_SYSTEM_PACKAGES=1
```
Bước 3: Chạy lại lệnh cài đặt
Bash
```
./install.sh
```