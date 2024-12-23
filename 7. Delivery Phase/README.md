# Delivery Phase

## I. Mục tiêu
- Ở giai đoạn Weaponization, chúng ta đã phát hiện ra các địa chỉ IP, domain, và email liên kết với nhóm attacker APT P01s0n1vy:
  - **Email**: lillian.rose@po1s0nvy.com
  - **Domain**: po1s0nvy.com
  - **Địa chỉ IP**: 23.22.63.114

- Trong giai đoạn Delivery, chúng ta sử dụng các thông tin đã thu thập được để tìm bất kỳ phần mềm độc hại nào có liên quan đến attacker thông qua các nền tảng Threat Hunting và trang web OSINT.

## II. Các bước thực hiện

### 1. Điều tra địa chỉ IP `23.22.63.114` với ThreatMiner
- **Kết quả**:
  - Có ba file dạng MD5 liên quan đến IP `23.22.63.114`:
    - `39eecefa9a13293a93bb20036eaf1f5e`
    - `aae3f5a29935e6abcc2c2754d12a9af0`
    - `c99131e0169171935c5ac32615ed6261`
  - Trong đó, hash MD5 `c99131e0169171935c5ac32615ed6261` được các Antivirus phát hiện là một malware.
    ![Hình ảnh](https://github.com/PhucsS24/Incident-handling-with-Splunk/blob/main/7.%20Delivery%20Phase/images/1.png)

### 2. Kiểm tra hash MD5 `c99131e0169171935c5ac32615ed6261` với ThreatMiner
- **Kết quả**:
  - Phát hiện một file thực thi: `MirandaTateScreensaver.scr.exe`
  - File dành cho Windows, kiến trúc Intel 80386, kích thước file ~494 KB.
    ![Hình ảnh](https://github.com/PhucsS24/Incident-handling-with-Splunk/blob/main/7.%20Delivery%20Phase/images/2.png)

### 3. Kiểm tra hash MD5 `c99131e0169171935c5ac32615ed6261` với VirusTotal
- **Kết quả**:
  - Hash được gán nhãn: `trojan.redsip/sanwaicrypt`.
    ![Hình ảnh](https://github.com/PhucsS24/Incident-handling-with-Splunk/blob/main/7.%20Delivery%20Phase/images/3.png)
    
  - Phần "Relations" cho thấy địa chỉ IP `23.22.63.114` liên kết với malware này.
    ![Hình ảnh](https://github.com/PhucsS24/Incident-handling-with-Splunk/blob/main/7.%20Delivery%20Phase/images/4.png)

### 4. Kiểm tra hash MD5 `c99131e0169171935c5ac32615ed6261` với Hybrid-Analysis
- **Kết quả**:
  - Nhận được thông tin chi tiết về malware `MirandaTateScreensaver.scr.exe`.
    ![Hình ảnh](https://github.com/PhucsS24/Incident-handling-with-Splunk/blob/main/7.%20Delivery%20Phase/images/5.png)

## III. Trả lời các câu hỏi

### 1. What is the HASH of the Malware associated with the APT group?
- **Đáp án**: `c99131e0169171935c5ac32615ed6261`
- **Giải thích**:
  - Sử dụng ThreatMiner để tìm kiếm địa chỉ IP `23.22.63.114` liên quan đến APT group Poison Ivy, hash MD5 `c99131e0169171935c5ac32615ed6261` được xác định là malware.
  ![Hình ảnh](https://github.com/PhucsS24/Incident-handling-with-Splunk/blob/main/7.%20Delivery%20Phase/images/cau1.png)

### 2. What is the name of the Malware associated with the Poison Ivy Infrastructure?
- **Đáp án**: `MirandaTateScreensaver.scr.exe`
- **Giải thích**:
  - Kiểm tra hash MD5 `c99131e0169171935c5ac32615ed6261` với Hybrid-Analysis cho thấy tên malware là `MirandaTateScreensaver.scr.exe`.
  ![Hình ảnh](https://github.com/PhucsS24/Incident-handling-with-Splunk/blob/main/7.%20Delivery%20Phase/images/cau2.png)*

## IV. Kết luận
- Sau khi hoàn thành giai đoạn Delivery, chúng ta đã xác định được các phần mềm độc hại mà nhóm attacker sử dụng:
  - **Phần mềm độc hại liên quan nhóm attacker Poison Ivy**:
    - `MirandaTateScreensaver.scr.exe`
  - **Mã HASH của phần mềm độc hại**:
    - **MD5**: `c99131e0169171935c5ac32615ed6261`
