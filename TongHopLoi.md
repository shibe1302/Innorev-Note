### 1.Sửa lỗi / Cấu hình lại Host IP trong sources.list (Ubuntu)

Cập nhật file sources.list chính:
Bash
```
sudo sed -Ei.bak 's|http://([0-9]{1,3}\.){3}[0-9]{1,3}/ubuntu|http://archive.ubuntu.com/ubuntu|g' /etc/apt/sources.list
```
Cập nhật các file bổ sung trong thư mục sources.list.d:
Bash
```
sudo sed -Ei.bak 's|http://([0-9]{1,3}\.){3}[0-9]{1,3}/ubuntu|http://archive.ubuntu.com/ubuntu|g' /etc/apt/sources.list.d
````
💡 Giải thích chi tiết lệnh:
sed -Ei.bak:

-E: Sử dụng biểu thức chính quy mở rộng (Extended Regular Expressions).

-i: Chỉnh sửa trực tiếp trên file gốc.

.bak: Tự động tạo một bản sao lưu (backup) của file trước khi thay đổi (phòng trường hợp lỗi).

's|...|...|g': Cú pháp tìm kiếm và thay thế toàn bộ (g - global) các chuỗi khớp với mẫu biểu thức chính quy IP (([0-9]{1,3}\.){3}[0-9]{1,3}) thành archive.ubuntu.com.