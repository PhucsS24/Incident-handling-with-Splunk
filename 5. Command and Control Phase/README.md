# Command and Control Phase

## I. Mục tiêu:
- Ở giai đoạn Installation chúng ta đã phát hiện attacker tải lên và thực thi một chương trình độc hại “3791.exe” để duy trì quyền truy cập và kiểm soát webserver.

- Ở giai đoạn Action on Objective attacker đã kiểm soát server truy cập đến malicious domain “prankglassinebracket.jumpingcrab.com” tải về tệp `.jpeg` “poisonivy-is-coming-for-you-batman.jpeg” để phá hỏng trang web.

- Ở giai đoạn C2 chúng ta cần điều tra địa chỉ IP của malicious domain là gì?

## II. Các bước thực hiện:

### 1. Tìm kiếm các sự kiện liên quan đến tệp `.jpeg` “poisonivy-is-coming-for-you-batman.jpeg” trong nguồn Log `fortigate_utm`.
   - **Truy vấn:**
     ```spl
     index=botsv1 sourcetype=fortigate_utm "poisonivy-is-coming-for-you-batman.jpeg"
     ```
   - **Giải thích:**
     - Tìm kiếm các sự kiện liên quan đến tệp `.jpeg` “poisonivy-is-coming-for-you-batman.jpeg” trong nguồn Log `fortigate_utm`.
   - **Kết quả:**
     - Chúng ta thấy trong trường hostname có chứa malicious domain và port của attacker.
   ![Hình ảnh](https://github.com/PhucsS24/Incident-handling-with-Splunk/blob/main/5.%20Command%20and%20Control%20Phase/images/1.png)

### 2. Tiếp tục truy vấn với trường hostname chứa malicious domain:
   - **Truy vấn:**
     ```spl
     index=botsv1 sourcetype=fortigate_utm "poisonivy-is-coming-for-you-batman.jpeg" hostname="prankglassinebracket.jumpingcrab.com:1337"
     ```
   - **Giải thích:**
     - Tìm kiếm các sự kiện liên quan đến tệp `.jpeg` “poisonivy-is-coming-for-you-batman.jpeg” và hostname “prankglassinebracket.jumpingcrab.com” trong nguồn Log `fortigate_utm`.
   - **Kết quả:**
     - Chúng ta có thể thấy trường dest_ip của các sự kiện đang trỏ đến malicious domain “prankglassinebracket.jumpingcrab.com” là IP `23.22.63.114`.
     ![Hình ảnh](https://github.com/PhucsS24/Incident-handling-with-Splunk/blob/main/5.%20Command%20and%20Control%20Phase/images/2.png)

     ![Hình ảnh](https://github.com/PhucsS24/Incident-handling-with-Splunk/blob/main/5.%20Command%20and%20Control%20Phase/images/3.png)

### 3. Xác minh malicious domain “prankglassinebracket.jumpingcrab.com” đang trỏ đến IP `23.22.63.114` trong nguồn Log `stream:http`.
   - **Truy vấn:**
     ```spl
     index=botsv1 sourcetype=stream:http dest_ip=23.22.63.114 "poisonivy-is-coming-for-you-batman.jpeg" src_ip=192.168.250.70
     ```
   - **Giải thích:**
     - Tìm kiếm các sự kiện liên quan đến tệp `.jpeg` trong quá trình giao tiếp giữa webserver `192.168.250.70` và IP của attacker `23.22.63.114`.
   - **Kết quả:**
   ![Hình ảnh](https://github.com/PhucsS24/Incident-handling-with-Splunk/blob/main/5.%20Command%20and%20Control%20Phase/images/4.png)

## III. Trả lời các câu hỏi:

- **This attack used dynamic DNS to resolve to the malicious IP. What fully qualified domain name (FQDN) is associated with this attack?**
  - `prankglassinebracket.jumpingcrab.com`

## IV. Kết luận:
- Sau khi hoàn thành giai đoạn C2, chúng ta đã biết được malicious domain và địa chỉ IP của nó:
  - `prankglassinebracket.jumpingcrab.com`
  - `23.22.63.114`
