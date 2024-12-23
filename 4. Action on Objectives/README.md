# Action on Objective Phase

## I. Mục tiêu:
- Ở giai đoạn này chúng ta sẽ điều tra, tìm hiểu các hành động mà attacker đã thực hiện để phá hoại trang web **imreallynotbatman.com**.

- Ở các giai đoạn trước chúng ta đã phát hiện:
  - Hai IP mà attacker sử dụng để tấn công là: `23.22.63.114` và `40.80.148.42`
  - Địa chỉ IP webserver bị tấn công là: `192.168.250.70`

---

## II. Các bước thực hiện:
   - Ở giai đoạn Installation Attacker đã tải lên và thực thi một chương trình độc hại **`3791.exe`** lên webserver để duy trì quyền truy cập và kiểm soát server.

1. **Kiểm tra webserver đã bị attacker kiểm soát và truy cập đến domain độc hại nào**:
   - **Truy vấn**:
     ```spl
     index=botsv1 src=192.168.250.70 sourcetype=suricata
     ```
   - **Giải thích**:
     - Tìm kiếm các sự kiện traffic mạng sinh ra từ webserver có ip `192.168.250.70` trong nguồn Log `suricata`.

   - **Kết quả**:
     - Kiểm tra trong trường `dest_ip`, chúng ta thấy webserver đã bị attacker kiểm soát và truy cập đến ba địa chỉ IP bên ngoài mạng:
       - `40.80.148.42` (81.874%)
       - `23.22.63.114` (10.269%)
       - `108.161.187.134` (0.095%)
         
      ![Hình ảnh](https://github.com/PhucsS24/Incident-handling-with-Splunk/blob/main/4.%20Action%20on%20Objectives/images/1.png)

2. **Kiểm tra giao tiếp giữa webserver đến địa chỉ IP `23.22.63.114` của attacker**:
   - **Truy vấn**:
     ```spl
     index=botsv1 src=192.168.250.70 sourcetype=suricata dest_ip=23.22.63.114
     ```
   - **Giải thích**:
     - Tìm kiếm các sự kiện traffic mạng trong quá trình webserver giao tiếp với IP `23.22.63.114` của attacker trong nguồn Log `suricata`.

   - **Kết quả**:
     - Kiểm tra trong trường `url`, chúng ta phát hiện 2 file PHP và một file JPEG. File `.jpeg` này có thể là hình ảnh mà attacker tải lên để phá hoại trang web.
       
     ![Hình ảnh](https://github.com/PhucsS24/Incident-handling-with-Splunk/blob/main/4.%20Action%20on%20Objectives/images/2.png)

3. **Kiểm tra file `.jpeg` đáng ngờ đến từ đâu**:
   - **Truy vấn**:
     ```spl
     index=botsv1 url="/poisonivy-is-coming-for-you-batman.jpeg" dest_ip="192.168.250.70" | table _time src dest_ip http.hostname url
     ```
   - **Giải thích**:
     - Tìm kiếm các sự kiện liên quan đến hình ảnh `/poisonivy-is-coming-for-you-batman.jpeg`, trích xuất các thông tin thời gian, IP nguồn, IP đích, hostname, URL ra bảng.

   - **Kết quả**:
     - File JPEG đáng ngờ **`poisonivy-is-coming-for-you-batman.jpeg`** đã được tải xuống từ máy chủ của kẻ tấn công với domain:
       - `prankglassinebracket.jumpingcrab.com`.
     - Sau đó, file này được sử dụng để làm hỏng trang web.

     ![Hình ảnh](https://github.com/PhucsS24/Incident-handling-with-Splunk/blob/main/4.%20Action%20on%20Objectives/images/3.png)

---

## III. Trả lời các câu hỏi:

### 1. What is the name of the file that defaced the `imreallynotbatman.com` website?
- **Tên tệp .jpeg**: `poisonivy-is-coming-for-you-batman.jpeg`
- Truy vấn:
    ```spl
    index=botsv1 src=192.168.250.70 sourcetype=suricata dest_ip=23.22.63.114
    ```
  - Ta thấy một file .jpeg “poisonivy-is-coming-for-you-batman.jpeg” trong trường url

  ![Hình ảnh](https://github.com/PhucsS24/Incident-handling-with-Splunk/blob/main/4.%20Action%20on%20Objectives/images/cau1.png)
 
- Fortigate Firewall 'fortigate_utm' detected SQL attempt from the attacker's IP 40.80.148.42. What is the name of the rule that was triggered during the SQL Injection attempt?
  - HTTP.URI.SQL.Injection
  - Truy vấn:
    ```spl
    index="botsv1" sourcetype="fortigate_utm" src_ip="40.80.148.42"
    ```
  - Giải thích:
    - Truy vấn tìm kiếm các sự kiện traffic mạng sinh ra từ IP "40.80.148.42" của attacker trong nguồn Log firewall fortigate_utm.
  - Kết quả:
    - Kiểm tra trong trường “attack” có các rule đã được kích hoạt ta có thể thấy rule liên quan đến tấn công SQL injection là: “HTTP.URI.SQL.Injection”

   ![Hình ảnh](https://github.com/PhucsS24/Incident-handling-with-Splunk/blob/main/4.%20Action%20on%20Objectives/images/cau2.png)


## IV. Kết luận
- Sau khi hoàn thành gia đoạn Action on Objective chúng ta đã biết được cách mà attacker làm hỏng trang web.
  - Tập tin làm hỏng trang web imreallynotbatman.com là:
    - `poisonivy-is-coming-for-you-batman.jpeg`
  - Malicious domain mà attacker tải tệp .jpeg là:
    - `prankglassinebracket.jumpingcrab.com`
  - Địa chỉ IP của Malicious domain là:
    - `23.22.63.114`

