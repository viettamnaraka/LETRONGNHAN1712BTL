# 🛡️ Secure FTP Client with Virus Scanning via ClamAVAgent

## 📚 Course Info

* **Môn học**: Mạng máy tính
* **Lớp**: 17-12
* **Nhóm thực hiện**:

  * Lê Trọng Nhân – MSSV: 1771020521

---

## 🔎 Overview

Dự án mô phỏng hệ thống truyền file an toàn, nơi mọi file cần được quét virus bằng ClamAV trước khi được upload lên FTP Server. Hệ thống gồm:

* **FTP Client**: Chương trình chính, cung cấp các lệnh FTP-client.
* **ClamAVAgent**: Dịch vụ quét virus hoạt động qua socket.
* **FTP Server**: Máy chủ nhận file sạch để lưu trữ.

---

## ⚙️ Thành phần hệ thống

### 1. `ftp_client.py`

* Giao tiếp với FTP Server và ClamAVAgent.
* Hỗ trợ các lệnh như `ls`, `cd`, `put`, `mput`, `get`, `mget`, v.v.
* Tất cả các lệnh upload phải qua quét ClamAV trước.

### 2. `clamav_agent.py`

* Chạy như một server nhận file từ client.
* Dùng `clamscan` để quét virus.
* Gửi kết quả `OK` hoặc `INFECTED` về cho FTP Client.

### 3. `FTP Server`

* Dùng phần mềm như FileZilla Server.
* Chỉ nhận file nếu đã qua kiểm duyệt từ ClamAVAgent.

---

## ⚙️ Cài đặt và cấu hình

### 🔹 <img src="https://www.clamav.net/assets/clamav-trademark.png" alt="ClamAV Logo" width="20"/> Cài đặt ClamAV trên Windows

#### `Giới thiệu`

ClamAV là công cụ chống virus mã nguồn mở, đa nền tảng. Hướng dẫn này sẽ giúp bạn cài đặt ClamAV trên Windows với hai chế độ:

* ClamScan: Chế độ quét cơ bản
* ClamDScan: Chế độ daemon cho tốc độ quét nhanh hơn

#### PHẦN 1: CÀI ĐẶT CLAMSCAN

##### Bước 1: Tải về ClamAV

1. Truy cập trang chủ: [https://www.clamav.net/downloads](https://www.clamav.net/downloads)
2. Chọn **Windows** → Tải file `clamav-1.4.3.win.x64.zip`

##### Bước 2: Cài đặt

1. Giải nén file vào nơi dễ nhớ (giả sử C:\ClamAV\clamav-1.4.3.win.x64):
- unzip clamav-1.4.3.win.x64.zip 
2. Di chuyển vào thư mục cài đặt:
- cd C:\ClamAV\clamav-1.4.3.win.x64
  
##### Bước 3: Cấu hình

1. Sao chép file cấu hình mẫu từ thư mục `conf_examples` sang thư mục chính.
2. Đổi tên file:

```
clamd.conf.sample → clamd.conf
freshclam.conf.sample → freshclam.conf
```

3. Mở file và xóa dòng chứa `Example` (thường là dòng số 8).
4. Lưu lại các thay đổi.

##### Bước 4: Cập nhật cơ sở dữ liệu
- Mở cmd, gõ:
```sh
cd C:\ClamAV\clamav-1.4.3.win.x64
freshclam.exe
```

(Chờ quá trình tải database hoàn tất)

#### PHẦN 2: CÀI ĐẶT CLAMDSCAN (DAEMON)

##### So sánh ClamScan và ClamD

| Tính năng           | ClamScan   | ClamD        |
| ------------------- | ---------- | ------------ |
| Thời gian khởi động | 10–60 giây | 0.1–0.5 giây |
| Tài nguyên          | Cao        | Thấp         |
| Hiệu suất           | Chậm       | Nhanh        |

##### Bước 1: Cấu hình `clamd.conf`
- Mở file clamd.conf trong C:\ClamAV\clamav-1.4.3.win.x64:
1. Kết nối TCP:

```
TCPSocket 3310
TCPAddr 127.0.0.1
```

2. Đường dẫn log:

```
LogFile "C:\ClamAV\clamav-1.4.3.win.x64\clamd.log"
LogTime yes
LogFileMaxSize 5M
```

3. Thư mục database:

```
DatabaseDirectory "C:\ClamAV\clamav-1.4.3.win.x64\database"
```

4. Tối ưu hiệu năng (tuỳ chọn):

```
ScanOLE2 no
ScanPDF no
ScanSWF no
```

> Đảm bảo không còn dấu `#` comment trước các dòng trên.

##### Bước 2: Cài đặt daemon
- Mở cmd, gõ:
```sh
cd C:\ClamAV\clamav-1.4.3.win.x64
clamd.exe --config-file="clamd.conf"
```

Chạy đến khi xuất hiện: `Self checking every 600 seconds` là thành công.

#### PHẦN 3: CÀI ĐẶT Filezilla Server
##### Bước 1: Tải FileZilla Server
1. Truy cập trang chính thức: https://filezilla-project.org/download.php?type=server
2. Nhấn nút Download FileZilla Server phù hợp với hệ điều hành (thường là Windows 64-bit).
3. Chạy file .exe để bắt đầu cài đặt.
##### Bước 2: Cài đặt FileZilla Server
- Trong giao diện setup, nhấn:
1. I Agree
2. Next -> Next
3. Để listening port như vậy, gõ mật khẩu bạn muốn (khuyến khích mật khẩu mạnh) -> Next -> Install -> Ok
4. Sau khi cài xong -> Close -> Connect to server -> Gõ lại password (chọn save the password) -> Yes
- Trong giao diện Administration interface, góc trái trên cùng màn hình:
1. Chọn server -> Configure...
2. Trong Rights Management, chọn Users -> OK
3. Trong giao diện Rights Management/Users, chọn Add -> đặt tên cho users
4. Trong ô chữ nhật Mount points: Đặt tên cho Virtual path ví dụ : /test (phải bắt đầu bằng /)
5. Tiếp tục, Paste đường dẫn tuỳ thích trong máy tính để làm đường dẫn cho server trong ô Native path, ví dụ : C:\Users\tn421\Downloads\newfolder
6. Trong giao diện chính chọn server -> Configure -> Server listeners ->Ở cột protocol chọn Explicit FTP over TLS and insecure plain FTP
- Tiếp theo, set port cho Server:
1. Mở dải port Passive trên Firewall (VD: 49152–49160)
```sh
netsh advfirewall firewall add rule name="FileZilla Passive Ports" dir=in action=allow protocol=TCP localport=49152-49160
```
- (Tùy chọn: Mở outbound nếu cần kết nối từ bên ngoài)
```sh
netsh advfirewall firewall add rule name="FileZilla Passive Outbound" dir=out action=allow protocol=TCP localport=49152-49160
```
2. 🎯 Nhớ cấu hình FileZilla Server để sử dụng dải port Passive này trong phần server -> Configure -> Protocal settings -> FTP and FTP over TLS (FTPS) -> vào Passive Mode -> chọn dải from 49152 to 49160
## 🚀 Cách chạy hệ thống

1. Mở 1 terminal:
- cd đường_dẫn_tới_clamav_agent.py trong project
- gõ:
```sh
python clamav_agent.py
```

2. Mở 1 terminal khác để chạy chương trình:
- cd đến virtual path bạn đã set trong FileZilla
- gõ các lệnh theo hướng dẫn sau:
```sh
        Kết nối:
          open       - Kết nối tới FTP server
          close      - Đóng kết nối
          quit/bye   - Thoát chương trình
        
        Thư mục & File:
          ls [path]  - Liệt kê nội dung thư mục
          cd <path>  - Thay đổi thư mục trên server
          lcd <path> - Thay đổi thư mục cục bộ
          pwd        - Xem thư mục hiện tại trên server
          mkdir <dir>- Tạo thư mục mới
          rmdir <dir>- Xóa thư mục
          delete <f> - Xóa file
          rename <o> <n> - Đổi tên file/thư mục
        
        Truyền file:
          get <file> - Tải file từ server
          put <file> - Upload file lên server (có quét virus)
          mget <pat> - Tải nhiều file (vd: *.txt)
          mput <pat> - Upload nhiều file
        
        Cài đặt:
          ascii      - Chuyển sang chế độ truyền văn bản
          binary     - Chuyển sang chế độ truyền nhị phân
          passive [on|off] - Bật/tắt chế độ passive
          prompt [on|off] - Bật/tắt xác nhận khi mget/mput
          status     - Xem trạng thái hiện tại
        
        Khác:
          help/?     - Hiển thị trợ giúp này
```
3. Trong file config.py sửa lại FTP_PASS và FTP_USER theo tên tài khoản và mật khẩu bạn đã set.
---

### 🔹 FTP Server

* Cài đặt FileZilla Server.
* Tạo user và cấp quyền thư mục.
* Kích hoạt chế độ Passive nếu cần (mport).

---



### Ví dụ các lệnh mẫu và đầu ra mong đợi:
* `open` -> đầu ra:
```sh
ftp> open
<<< 220-FileZilla Server 1.10.3
<<< 220 Please visit https://filezilla-project.org/
220 Please visit https://filezilla-project.org/
>>> USER ftpuser
<<< 331 Please, specify the password.
>>> PASS NguyenToan2k6@123
<<< 230 Login successful.
>>> PWD
<<< 257 "/" is current directory.
📂 Thư mục hiện tại: /
```
* `ls` -> đầu ra:
```sh
ftp> ls
>>> PASV
<<< 227 Entering Passive Mode (127,0,0,1,192,6)
>>> LIST
<<< 150 Starting data transfer.
=== DANH SÁCH ===
drwxrwxrwx 1 ftp ftp               0 Aug 01 01:38 new

<<< 226 Operation successful
```
* `cd <path>` -> đầu ra:
```sh
ftp> cd /new
>>> CWD /new
<<< 250 CWD command successful
>>> PWD
<<< 257 "/new" is current directory.
📂 Thư mục hiện tại: /new
```
* `pwd` -> đầu ra:
```sh
ftp> pwd
>>> PWD
<<< 257 "/new" is current directory.
📂 Thư mục hiện tại: /new
 ```
* `mkdir <dir>` -> đầu ra:
```sh
ftp> mkdir new
>>> MKD new
<<< 257 "/new/new" created successfully.
✅ Đã tạo thư mục: new
```
* `rmdir <dir>` -> đầu ra:
```sh
ftp> rmdir new
>>> RMD new
<<< 250 Directory deleted successfully.
✅ Đã xóa thư mục: new
```
* `delete <file>` -> đầu ra:
```sh
ftp> delete LÝ THUYẾT ĐẠI SỐ TUYẾN TÍNH.docx
>>> DELE LÝ THUYẾT ĐẠI SỐ TUYẾN TÍNH.docx
<<< 250 File deleted successfully.
✅ Đã xóa file: LÝ THUYẾT ĐẠI SỐ TUYẾN TÍNH.docx
```
* `rename <o> <n>` -> đầu ra:
```sh
ftp> rename demo.txt test.txt
>>> RNFR demo.txt
<<< 350 File exists, ready for destination name.
>>> RNTO test.txt
<<< 250 File or directory renamed successfully.
✅ Đã đổi tên demo.txt → test.txt
```
* `get <file>` -> đầu ra:
```sh
ftp> get test.txt
>>> PASV
<<< 227 Entering Passive Mode (127,0,0,1,192,4)
>>> RETR test.txt
<<< 150 Starting data transfer.
<<< 226 Operation successful
✅ Tải thành công: test.txt → D:\PYTHONSOCKET\socket_project\test.txt
📊 Kích thước: 14 bytes | Thời gian: 0.00s | Tốc độ: 17.31 KB/s
```
* `put <file>` -> đầu ra:
```sh
ftp> put hello.txt
🔍 Đang quét virus: hello.txt
>>> PASV
<<< 227 Entering Passive Mode (127,0,0,1,192,5)
>>> STOR hello.txt
<<< 150 Starting data transfer.
<<< 226 Operation successful
✅ Upload thành công: D:\PYTHONSOCKET\socket_project\hello.txt → hello.txt
📊 Kích thước: 9 bytes | Thời gian: 0.00s | Tốc độ: 10.62 KB/s
```
* `mget <pat>` -> đầu ra:
```sh
ftp> mget *.txt
>>> PASV
<<< 227 Entering Passive Mode (127,0,0,1,192,6)
>>> NLST
<<< 150 Starting data transfer.
<<< 226 Operation successful
🔍 Tìm thấy 2 file:
  1. hello.txt
  2. test.txt
Bạn có muốn tải tất cả? (y/n): y
⬇️  Đang tải: hello.txt
>>> PASV
<<< 227 Entering Passive Mode (127,0,0,1,192,2)
>>> RETR hello.txt
<<< 150 Starting data transfer.
<<< 226 Operation successful
✅ Tải thành công: hello.txt → D:\PYTHONSOCKET\socket_project\hello.txt
📊 Kích thước: 9 bytes | Thời gian: 0.00s | Tốc độ: 8.43 KB/s
⬇️  Đang tải: test.txt
>>> PASV
<<< 227 Entering Passive Mode (127,0,0,1,192,7)
>>> RETR test.txt
<<< 150 Starting data transfer.
<<< 226 Operation successful
✅ Tải thành công: test.txt → D:\PYTHONSOCKET\socket_project\test.txt
📊 Kích thước: 14 bytes | Thời gian: 0.00s | Tốc độ: 17.86 KB/s
✅ Đã tải thành công 2/2 file
```
* `mput <pat>` -> đầu ra:
```sh
ftp> mput *.txt
🔍 Tìm thấy 3 file:
  1. eicar.txt
  2. hello.txt
  3. test.txt
Bạn có muốn upload tất cả? (y/n): y
⬆️  Đang upload: eicar.txt
🔍 Đang quét virus: eicar.txt
>>> PASV
<<< 227 Entering Passive Mode (127,0,0,1,192,0)
>>> STOR eicar.txt
<<< 150 Starting data transfer.
<<< 226 Operation successful
✅ Upload thành công: D:\PYTHONSOCKET\socket_project\eicar.txt → eicar.txt
📊 Kích thước: 60 bytes | Thời gian: 0.00s | Tốc độ: 79.48 KB/s
⬆️  Đang upload: hello.txt
🔍 Đang quét virus: hello.txt
>>> PASV
<<< 227 Entering Passive Mode (127,0,0,1,192,8)
>>> STOR hello.txt
<<< 150 Starting data transfer.
<<< 226 Operation successful
✅ Upload thành công: D:\PYTHONSOCKET\socket_project\hello.txt → hello.txt
📊 Kích thước: 9 bytes | Thời gian: 0.00s | Tốc độ: 20.92 KB/s
⬆️  Đang upload: test.txt
🔍 Đang quét virus: test.txt
>>> PASV
<<< 227 Entering Passive Mode (127,0,0,1,192,1)
>>> STOR test.txt
<<< 150 Starting data transfer.
<<< 226 Operation successful
✅ Upload thành công: D:\PYTHONSOCKET\socket_project\test.txt → test.txt
📊 Kích thước: 14 bytes | Thời gian: 0.00s | Tốc độ: 16.91 KB/s
✅ Đã upload thành công 3/3 file
```
* `ascii` -> đầu ra:
```sh
ftp> ascii
>>> TYPE A
<<< 200 Type set to A
✅ Đã chuyển sang chế độ ASCII
```
* `binary` -> đầu ra:
```sh
ftp> binary
>>> TYPE B
<<< 501 Unsupported type. Supported types are I, I N, A, A N and L 8.
✅ Đã chuyển sang chế độ BINARY
```
* `passive` -> đầu ra:
```sh
ftp> passive
✅ Đã TẮT chế độ passive
```
* `prompt` -> đầu ra:
```sh
ftp> prompt
✅ Đã TẮT chế độ xác nhận
```
* `status` -> đầu ra:
```sh
ftp> status
🌐 Đã kết nối: ✅
📂 Thư mục hiện tại: /new
💻 Thư mục cục bộ: D:\PYTHONSOCKET\socket_project
🛁 Chế độ passive: TẮT
📦 Chế độ truyền: BINARY
📢 Chế độ xác nhận: TẮT
📡 Địa chỉ server: 127.0.0.1:21
👤 Người dùng: ftpuser
```
* `help` -> đầu ra:
```sh
ftp> help
        =================== TRỢ GIÚP FTP CLIENT ===================

        Kết nối:
          open       - Kết nối tới FTP server
          close      - Đóng kết nối
          quit/bye   - Thoát chương trình

        Thư mục & File:
          ls [path]  - Liệt kê nội dung thư mục
          cd <path>  - Thay đổi thư mục trên server
          lcd <path> - Thay đổi thư mục cục bộ
          pwd        - Xem thư mục hiện tại trên server
          mkdir <dir>- Tạo thư mục mới
          rmdir <dir>- Xóa thư mục
          delete <f> - Xóa file
          rename <o> <n> - Đổi tên file/thư mục

        Truyền file:
          get <file> - Tải file từ server
          put <file> - Upload file lên server (có quét virus)
          mget <pat> - Tải nhiều file (vd: *.txt)
          mput <pat> - Upload nhiều file

        Cài đặt:
          ascii      - Chuyển sang chế độ truyền văn bản
          binary     - Chuyển sang chế độ truyền nhị phân
          passive [on|off] - Bật/tắt chế độ passive
          prompt [on|off] - Bật/tắt xác nhận khi mget/mput
          status     - Xem trạng thái hiện tại

        Khác:
          help/?     - Hiển thị trợ giúp này

        ===========================================================
```
---

## 📐 Sơ đồ kiến trúc hệ thống

```plaintext
 ┌──────────────────────┐        ┌─────────────────────────┐        ┌───────────────────────┐
 │      FTP Client      │        │      ClamAV Server      │        │       FTP Server      │
 │                      │        │                         │        │      (FileZilla)      │
 └─────────┬────────────┘        └───────────┬─────────────┘        └───────────┬───────────┘
           │                                 │                                  │
  [1] Gửi tên file + data                    │                                  │
      tới quét virus qua TCP socket ─────────►                                  │
           │                                 │                                  │
           │                         [2] Lưu file tạm và chạy:                  │
           │                         `clamdscan <file>`                         │
           │                                 │                                  │
           ◄─────────────────────── [3] Trả kết quả: OK / INFECTED / ERROR      |
           │                                 │                                  │
 [4]If OK: Upload file via FTP ───────────────────────────────────────────────► |
           │                                 │                                  │
           │                                 │                                  │
```
---

## 📜 Các lệnh được hỗ trợ

### 📁 File và thư mục

* `ls` – Liệt kê file/thư mục trên server
* `cd` – Đổi thư mục
* `pwd` – Hiển thị thư mục hiện tại
* `mkdir`, `rmdir` – Tạo/Xoá thư mục
* `delete` – Xoá file
* `rename` – Đổi tên file

### ⬇️⬆️ Tải lên / Tải xuống

* `put`, `mput` – Upload 1 hay nhiều file (phải quét virus)
* `get`, `mget` – Tải file từ server
* `prompt` – Bật/tắt xác nhận khi dùng mget, mput

### 🧭 Quản lý phiên

* ascii / binary – Chuyển chế độ truyền file (văn bản hoặc nhị phân)
* status – Hiển thị trạng thái phiên làm việc hiện tại
* passive – Bật/tắt chế độ Passive FTP
* open, close – Kết nối / ngắt kết nối với FTP server
* quit, bye – Thoát chương trình FTP client
* help, ? – Hiển thị hướng dẫn các lệnh
