# Installation Phase

## I. Mục tiêu:
- Ở giai đoạn Exploitation chúng ta đã phát hiện attacker sử dụng phương pháp tấn công brute-force nhắm vào trang quản trị admin CMS Joomla của webserver "iamreallynotbatman.com".
  - Attacker sử dụng IP `23.22.63.114` để tấn công brute-force.
  - Attacker sử dụng IP `40.80.148.42` để đăng nhập sau khi có được username và password trong quá trình tấn công brute-force.
  - Username: `admin`, password: `batman`.
  - Mục tiêu bị tấn công brute-force:
    - URI: `/joomla/administrator/index.php`
    - IP webserver: `192.168.250.70`

- Ở giai đoạn Installation, chúng ta sẽ điều tra có bất kỳ payload hay malicious program nào được tải lên từ attacker hay không. Nếu có thì kiểm tra các malicious program đó đã được thực thi trên webserver hay chưa. Thông qua các malicious program thì attacker có thể duy trì quyền truy cập và kiểm soát hệ thống.

---

## II. Các bước thực hiện:

### 1. Kiểm tra malicious program tải lên từ attacker

- **Truy vấn:**
  ```
  index=botsv1 sourcetype=stream:http dest_ip="192.168.250.70" *.exe
  ```
- **Giải thích:**
  - Tìm kiếm tất cả sự kiện traffic HTTP có chứa `.exe` truy cập đến IP webserver `192.168.250.70`.
- **Kết quả:**
  - Tìm kiếm thông tin trong các trường liên quan đến `filename` ta thấy trong trường `part_filename{}` có chứa hai tên tệp: một tệp thực thi `3791.exe` và một tệp PHP `agent.php`.
  ![Hình ảnh](https://github.com/PhucsS24/Incident-handling-with-Splunk/blob/main/3.%20Installation%20Phase/images/part-filename.png)

### 2. Kiểm tra các tệp được tải lên có liên quan đến các địa chỉ IP của attacker thực hiện tấn công brute-force không

- **Truy vấn:**
  ```
  index=botsv1 sourcetype=stream:http dest_ip="192.168.250.70" "part_filename{}"="agent.php" "part_filename{}"="3791.exe"
  ```
- **Giải thích:**
  - Tìm kiếm tất cả sự kiện traffic HTTP có chứa file `3791.exe` và `agent.php` truy cập đến IP webserver `192.168.250.70`.
- **Kết quả:**
  - Kiểm tra trong trường `c_ip` chúng ta phát hiện một IP của attacker `40.80.148.42`.
  ![Hình ảnh](https://github.com/PhucsS24/Incident-handling-with-Splunk/blob/main/3.%20Installation%20Phase/images/IP_up_file.png)

### 3. Xác nhận file `3791.exe` có được thực thi trên webserver chưa

- **Truy vấn:**
  ```
  index=botsv1 "3791.exe"
  ```
- **Giải thích:**
  - Tìm kiếm tất cả các nguồn log có liên quan đến file `3791.exe`.
- **Kết quả:**
  - Kiểm tra trong trường `sourcetype` chúng ta thấy các nguồn log sau có thể ghi lại các sự kiện liên quan đến `3791.exe`:
    - `XmlWinEventLog`: chứa các event logs Sysmon.
    - `WinEventLog`: chứa các event logs Windows.
    - `fortigate_utm`: chứa các event logs firewall.
  ![Hình ảnh](https://github.com/PhucsS24/Incident-handling-with-Splunk/blob/main/3.%20Installation%20Phase/images/source_log_3791exe.png)

#### Kiểm tra với nguồn log `XmlWinEventLog`

- **Truy vấn:**
  ```
  index=botsv1 "3791.exe" sourcetype="XmlWinEventLog" EventCode=1
  ```
- **Giải thích:**
  - Tìm kiếm sự kiện liên quan đến file `3791.exe` được thực thi trong nguồn log `XmlWinEventLog`.
  - `EventCode=1`: Là mã sự kiện, giá trị `1` nghĩa là tiến trình mới đã được tạo (file `3791.exe` đã được thực thi).
- **Kết quả:**
  - Kiểm tra các sự kiện trong trường `CommandLine` ta sẽ thấy các lệnh thực thi, rõ ràng rằng file `3791.exe` đã được thực thi trên webserver.
  ![Hình ảnh](https://github.com/PhucsS24/Incident-handling-with-Splunk/blob/main/3.%20Installation%20Phase/images/command.png)

---

## III. Trả lời các câu hỏi:

### 1. Sysmon also collects the Hash value of the processes being created. What is the MD5 HASH of the program 3791.exe?
- **MD5 Hash:** `AAE3F5A29935E6ABCC2C2754D12A9AF0`
- **Truy vấn:**
  ```
  index=botsv1 "3791.exe" sourcetype="XmlWinEventLog" EventCode=1 CommandLine="3791.exe"
  ```
- **Giải thích:**
  - Tìm kiếm sự kiện liên quan đến file `3791.exe` và lệnh thực thi `3791.exe` trong nguồn log `XmlWinEventLog`.
- **Kết quả:**
  - Kiểm tra trong trường `Hashs` chúng ta thấy xuất hiện mã hash MD5 `MD5=AAE3F5A29935E6ABCC2C2754D12A9AF0`.
  ![Hình ảnh](https://github.com/PhucsS24/Incident-handling-with-Splunk/blob/main/3.%20Installation%20Phase/images/cau1.png)

### 2. Looking at the logs, which user executed the program 3791.exe on the server?
- **User:** `NT AUTHORITY\IUSR`
- **Truy vấn:**
  ```
  index=botsv1 "3791.exe" sourcetype="XmlWinEventLog" EventCode=1 CommandLine="3791.exe" | table User
  ```
- **Giải thích:**
  - Tìm kiếm các sự kiện liên quan đến user, `3791.exe` và lệnh thực thi `3791.exe` trong nguồn log `XmlWinEventLog`.
- **Kết quả:**
  - Chúng ta có thể thấy thông tin về user `NT AUTHORITY\IUSR`.
  ![Hình ảnh](https://github.com/PhucsS24/Incident-handling-with-Splunk/blob/main/3.%20Installation%20Phase/images/cau2.png)

### 3. Search hash on VirusTotal. What other name is associated with this file 3791.exe?
- **Name:** `ab.exe`
- **Thao tác:**
  - Truy cập trang web VirusTotal ([https://www.virustotal.com/gui/home/upload](https://www.virustotal.com/gui/home/upload)).
  - Tìm kiếm với mã hash MD5 `AAE3F5A29935E6ABCC2C2754D12A9AF0`.
  - Chúng ta có thể thấy một tên khác được liên kết với tệp `3791.exe` là `ab.exe`.
  ![Hình ảnh](https://github.com/PhucsS24/Incident-handling-with-Splunk/blob/main/3.%20Installation%20Phase/images/cau3.png)

---

## IV. Kết luận
- Sau khi hoàn thành giai đoạn Installation, chúng ta đã phát hiện ra một file độc hại được attacker tải lên và thực thi trên webserver.
  - **Các payload/malicious program mà attacker đã tải lên webserver:**
    - `3791.exe` và `agent.exe`
  - **Địa chỉ IP mà attacker sử dụng để tải lên:**
    - `40.80.148.42`
  - **File thực thi `3791.exe` đã được attacker thực thi có mã hash MD5:**
    - `AAE3F5A29935E6ABCC2C2754D12A9AF0`
  - **User đã thực thi `3791.exe` là:**
    - `NT AUTHORITY\IUSR`
  - **Tên khác liên kết với `3791.exe` trên VirusTotal:**
    - `ab.exe`
