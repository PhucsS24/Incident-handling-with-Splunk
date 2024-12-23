# Weaponization Phase

## I. Mục tiêu:
- Trong giai đoạn Weaponization, attacker sẽ:
  - Tạo các phần mềm/tài liệu độc hại để có quyền truy cập ban đầu.
  - Thiết lập các domain giả mạo tương tự như domain thật để đánh lừa người dùng.
  - Tạo C2 server để kiểm soát và liên lạc.

- Ở các giai đoạn trước, chúng ta đã phát hiện:
  - Malicious domain: `prankglassinebracket.jumpingcrab.com`
  - IP của malicious domain: `23.22.63.114`

- **Mục tiêu ở giai đoạn này:** 
  - Tìm kiếm thêm các thông tin liên quan đến malicious domain `prankglassinebracket.jumpingcrab.com` và địa chỉ IP `23.22.63.114` trên các trang web Threat Intel.

## II. Các bước thực hiện:
1. **Tìm kiếm tên miền `prankglassinebracket.jumpingcrab.com` trên trang web Robtex:**
   - **Kết quả:**
     - Các địa chỉ IP khác liên kết với domain này:
       - `69.197.18.183`
       - `70.39.97.227`
       - `169.47.130.85`
     - Mail server: `mail.jumpingcrab.com`
       ![Hình ảnh](https://github.com/PhucsS24/Incident-handling-with-Splunk/blob/main/6.%20Weaponization%20Phase/images/1.png)

     - Các subdomain liên quan đến domain này:
       ![Hình ảnh](https://github.com/PhucsS24/Incident-handling-with-Splunk/blob/main/6.%20Weaponization%20Phase/images/2.png)

2. **Tìm kiếm địa chỉ IP `23.22.63.114` trên web Robtex:**
   - **Kết quả:**
     - IP `23.22.63.114` được liên kết với một số tên miền trông khá giống với trang web WAYNE Enterprise.
       ![Hình ảnh](https://github.com/PhucsS24/Incident-handling-with-Splunk/blob/main/6.%20Weaponization%20Phase/images/3.png)

3. **Tìm kiếm các thông tin liên quan đến địa chỉ IP `23.22.63.114` trên trang web VirusTotal:**
   - **Kết quả:**
     - Trong phần “RELATIONS”, thấy tất cả các domain được liên kết với IP `23.22.63.114` trông giống với tên công ty Wayne Enterprises.
     - Một domain đặc biệt liên quan đến attacker: `www.po1s0n1vy.com`.
     - Các chương trình độc hại liên quan đến địa chỉ IP này.
      ![Hình ảnh](https://github.com/PhucsS24/Incident-handling-with-Splunk/blob/main/6.%20Weaponization%20Phase/images/4.png)

4. **Thu thập các thông tin liên quan đến domain `www.po1s0n1vy.com` trên trang web VirusTotal:**
   - **Kết quả:**
     - Có nhiều địa chỉ IP khác được liên kết với domain `www.po1s0n1vy.com`.
     - Các subdomain liên quan:
       - `ftp.po1s0n1vy.com`
       - `smtp.po1s0n1vy.com`
       ![Hình ảnh](https://github.com/PhucsS24/Incident-handling-with-Splunk/blob/main/6.%20Weaponization%20Phase/images/5.png)

5. **Thu thập thông tin domain `po1s0n1vy.com` với công cụ `whois.domaintools.com`:**
   - **Kết quả:**
     - Name Server là máy chủ quản lý domain `po1s0nvy.com`:
       - NS: `DomainControl.com`
       - Địa chỉ IP của NS: `34.102.136.180`
       - Địa chỉ thực của NS: Missouri, Kansas City.
       ![Hình ảnh](https://github.com/PhucsS24/Incident-handling-with-Splunk/blob/main/6.%20Weaponization%20Phase/images/7.png)

## III. Trả lời các câu hỏi:

1. **What IP address has P01s0n1vy tied to domains that are pre-staged to attack Wayne Enterprises?**
   - **Đáp án:**
     - `23.22.63.114`
   - **Giải thích:**
     - IP `23.22.63.114` đã thực hiện tấn công brute-force đến webserver Wayne Enterprises.
     - Domain `P01s0n1vy.com` liên kết với IP `23.22.63.114`.
       ![Hình ảnh](https://github.com/PhucsS24/Incident-handling-with-Splunk/blob/main/6.%20Weaponization%20Phase/images/cau1.png)

2. **Based on the data gathered from this attack and common open-source intelligence sources for domain names, what is the email address that is most likely associated with the P01s0n1vy APT group?**
   - **Đáp án:**
     - `lillian.rose@po1s0nvy.com`
   - **Giải thích:**
     - Sử dụng tool `WhoXY` (https://www.whoxy.com), tìm kiếm với domain `po1s0nvy.com`.
     - Phát hiện domain `po1s0nvy.com` đã hết hạn và chưa được đăng ký.
     - Trong các records, thấy domain `po1s0nvy.com` từng có email `lillian@po1s0n1vy.com`.
       ![Hình ảnh](https://github.com/PhucsS24/Incident-handling-with-Splunk/blob/main/5.%20Command%20and%20Control%20Phase/images/cau2.png)

## IV. Kết luận:
- Sau khi hoàn thành giai đoạn Weaponization, chúng ta đã thu thập được các thông tin liên quan đến địa chỉ IP `23.22.63.114` và malicious domain `prankglassinebracket.jumpingcrab.com`.

  - **Domain liên quan đến nhóm attacker:**
    - `www.po1s0n1vy.com`
  - **Địa chỉ IP:**
    - `23.22.63.114`
  - **Địa chỉ email liên quan đến nhóm APT P01s0n1vy:**
    - `lillian.rose@po1s0nvy.com`
