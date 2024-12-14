# Reconnaissance Phase

## I. Mục tiêu
Ở giai đoạn này, chúng ta cần thu thập các thông tin về mục tiêu bị tấn công như kiến thức về hệ thống đang sử dụng, ứng dụng web, nhân viên hoặc vị trí, v.v.

### Phân tích các nỗ lực trinh sát chống lại máy chủ web "imreallynotbatman.com":

- **Những IP nào đáng ngờ truy cập đến trang web?**
- **Kẻ tấn công sử dụng các Tool dò quét lỗ hổng nào?**
- **Địa chỉ IP của webserver bị tấn công là gì?**
- **Webserver sử dụng CMS nào?**
- **Các lỗ hổng nào mà attacker có thể khai thác?**

## II. Các bước thực hiện
Mục tiêu bị tấn công là một webserver có domain **imreallynotbatman.com**. Đầu tiên chúng ta cần xác định các nguồn Log nào sẽ ghi lại dấu vết các giao tiếp đến webserver qua domain **imreallynotbatman.com**.

### 1. Truy vấn log ban đầu
- **Truy vấn:** `index=botsv1 imreallynotbatman.com`
    - **Giải thích:** Truy vấn tìm kiếm sự kiện (Logs) trong index "botsv1" có chứa thuật ngữ "imreallynotbatman.com".
    - **Kết quả:** Chúng ta tìm thấy 4 nguồn Log chứa dấu vết liên quan đến domain "imreallynotbatman.com":
      - **Suricata:** Log chứa thông tin chi tiết về các cảnh báo từ Suricata IDS.
      - **stream:http:** Log chứa traffic mạng liên quan đến lưu lượng truy cập HTTP.
      - **fortigate_utm:** Log của firewall.
      - **iis:** Log của webserver IIS.

  ![Hình ảnh về các nguồn Log](https://github.com/PhucsS24/Incident-handling-with-Splunk/blob/main/assets/images_phase1/Picture1.png)

### 2. Xác định các địa chỉ IP đáng ngờ
Chúng ta cần xác định các địa chỉ IP đang cố gắng thực hiện hoạt động do thám trên webserver. Hãy bắt đầu với Log chứa traffic truy cập HTTP đến webserver **"stream:http"**.

- **Truy vấn:** `index=botsv1 imreallynotbatman.com sourcetype=stream:http`
    - **Giải thích:** Truy vấn tìm kiếm thuật ngữ "imreallynotbatman.com" trong Log “streams:http”.
    - **Kết quả:** Tìm thấy hai IP đáng ngờ trong trường **src_ip**:
      - **40.80.148.42** (93.402%)
      - **23.22.63.114** (6.598%)

  ![Hình ảnh về các IP đáng ngờ](https://github.com/PhucsS24/Incident-handling-with-Splunk/blob/main/assets/images_phase1/Picture2.png)

### 3. Kiểm tra dấu hiệu tấn công từ IP đáng ngờ
Tiếp theo, chúng ta cần xem xét các dấu hiệu tấn công từ từng IP để xác định liệu chúng có phải là IP malicious hay không.

#### Kiểm tra IP **40.80.148.42**
- **Truy vấn:** `index=botsv1 imreallynotbatman.com sourcetype=stream:http src_ip="40.80.148.42"`
    - **Kết quả:**
        - **form_data:** Phát hiện thực hiện cuộc tấn công khai thác lỗ hổng web như XSS, RCE, SQL Injection, Command Injection, Path Traversal, External Resource Loading.
        - **http_user_agent:** Phát hiện các hoạt động đáng ngờ chứa payload tấn công như `$(nslookup 0GIaVBMt)`, `${@print(md5(acunetix_wvs_security_test))}`, `!(()&&!|*|*|,…`
        - **uri:** Phát hiện các truy cập bất thường đến các URL hệ thống như `/joomla/administrator/index.php`, `/windows/win.ini`, …

  ![Video về dấu hiệu tấn công từ IP 40.80.148.42](https://github.com/PhucsS24/Incident-handling-with-Splunk/blob/main/assets/images_phase1/video%201.mp4)

#### Kiểm tra IP **23.22.63.114**
- **Truy vấn:** `index=botsv1 imreallynotbatman.com sourcetype=stream:http src_ip="23.22.63.114"`
    - **Kết quả:**
        - **uri:** Địa chỉ tấn công brute force là `/joomla/administrator/index.php`.
        - **http_user_agent:** `"Python-urllib/2.7"` – thư viện của Python tự động gửi các HTTP request để thực hiện tấn công brute force.
        - **form_data:** Nhiều thông tin username, password được gửi đi để thực hiện login.

  ![Video về dấu hiệu tấn công từ IP 23.22.63.114]([link-video-ở-đây](https://github.com/PhucsS24/Incident-handling-with-Splunk/blob/main/assets/images_phase1/video%202.mp4))

### 4. Kiểm tra dấu hiệu tấn công từ Log IDS "Suricata"
Tiếp theo, chúng ta thu hẹp phạm vi và đào sâu hơn các hoạt động khai thác lỗ hổng web từ IP **40.80.148.42**.

- **Truy vấn:** `index=botsv1 imreallynotbatman.com src=40.80.148.42 sourcetype=suricata`
    - **Kết quả:** Phát hiện các alert về tấn công XSS, SQL Injection và đặc biệt là tấn công khai thác lỗ hổng **CVE-2014-6271**.

  ![Hình ảnh về alert của Suricata](https://github.com/PhucsS24/Incident-handling-with-Splunk/blob/main/assets/images_phase1/Picture3.png)

## III. Trả lời các câu hỏi

1. **CVE value associated with the attack attempt?**
   - **CVE-2014-6271**
   - **Truy vấn:** `index=botsv1 imreallynotbatman.com src=40.80.148.42 sourcetype=suricata`
     - **Kết quả:** Phát hiện CVE-2014-6271 trong các alert của Suricata.

  ![Hình ảnh về CVE value](https://github.com/PhucsS24/Incident-handling-with-Splunk/blob/main/assets/images_phase1/Picture4.png)

2. **What is the CMS our web server is using?**
   - **Joomla**
   - **Truy vấn:** `index=botsv1 imreallynotbatman.com sourcetype="stream:http"`
     - **Kết quả:** Phát hiện các URL liên quan đến CMS Joomla.

  ![Hình ảnh về CMS](https://github.com/PhucsS24/Incident-handling-with-Splunk/blob/main/assets/images_phase1/Picture5.png)

3. **What is the web scanner, the attacker used?**
   - **Acunetix**
   - **Truy vấn:** `index=botsv1 imreallynotbatman.com sourcetype="stream:http"`
     - **Kết quả:** Dấu hiệu của công cụ quét lỗ hổng **Acunetix** trong **http_user_agent**.

  ![Hình ảnh về web scanner](https://github.com/PhucsS24/Incident-handling-with-Splunk/blob/main/assets/images_phase1/Picture6.png)

4. **What is the IP address of the server imreallynotbatman.com?**
   - **192.168.250.70**
   - **Truy vấn:** `index=botsv1 imreallynotbatman.com sourcetype="stream:http" http_user_agent="\";print(md5(acunetix_wvs_security_test));$a=\""`

  ![Hình ảnh về địa chỉ IP server](https://github.com/PhucsS24/Incident-handling-with-Splunk/blob/main/assets/images_phase1/Picture7.png)

## IV. Kết luận
Sau khi hoàn thành giai đoạn trinh sát, chúng ta đã thu thập được các thông tin quan trọng về mục tiêu và các kẻ tấn công:

- **Các IP malicious thực hiện tấn công:**
  - 40.80.148.42
  - 23.22.63.114

- **Công cụ dò quét mà attacker đã sử dụng:**
  - Acunetix

- **Địa chỉ IP của webserver:**
  - 192.168.250.70

- **CMS mà webserver sử dụng:**
  - Joomla

- **Các lỗ hổng web mà attacker thực hiện tấn công:**
  - CVE-2014-6271
  - Cross-Site Scripting (XSS), Remote Code Execution (RCE), SQL Injection và Command Injection, Path Traversal, External Resource Loading.
  - Tấn công brute force nhắm vào trang “/joomla/administrator/index.php” để chiếm quyền admin của webserver.
