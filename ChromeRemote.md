### 1.Sửa lỗi cài chrome remote bị thiếu thư viện trên ubutu26.

Bước 1: Cài thư viện còn thiếu. paste cả hai lệnh vào terminal

Bash
```
sudo apt update
sudo apt install -y python3-packaging python3-psutil xbase-clients xserver-
xorg-video-dummy xvfb
```
Bước 2: Sửa chữa những tiến trình cài đặt bị lỗi/ chưa hoàn thành

run
```
sudo apt --fix-broken install
```