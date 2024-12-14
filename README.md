# TryHackMe: Incident Handling with Splunk

## A. Kịch bản:
- Một tổ chức doanh nghiệp lớn **Wayne Enterprises** gần đây đã phải đối mặt với một cuộc tấn công mạng, trong đó những kẻ tấn công đã đột nhập vào mạng của họ, tìm đường đến máy chủ web và đã thành công trong việc phá hoại trang web [http://www.imreallynotbatman.com](http://www.imreallynotbatman.com).
- Trang web của họ hiện đang hiển thị logo của những kẻ tấn công với thông báo **“YOUR SITE HAS BEEN DEFACED”**, như hình bên dưới:

![Defaced Website](https://github.com/PhucsS24/Incident-handling-with-Splunk/blob/main/assets/Picture0.png) <!-- Thay # bằng đường dẫn hình ảnh nếu có -->

- Wayne Enterprises đã triển khai **Splunk** và có tất cả các bản ghi Logs sự kiện liên quan đến hoạt động của kẻ tấn công như sau:
  - `wineventlog`
  - `winRegistry`
  - `XmlWinEventLog`
  - `fortigate_utm`
  - `iis`
  - `Nessus:scan`
  - `Suricata`
  - `stream:http`
  - `stream:DNS`
  - `stream:icmp`

## B. Nhiệm vụ:
1. Điều tra cuộc tấn công để:
   - Tìm ra **nguyên nhân gốc rễ**.
   - Xác định các **hành động mà kẻ tấn công đã thực hiện trong hệ thống**.
2. **Index:** `botsv1`
3. **Mục tiêu bị tấn công:** Trang web có domain: **`imreallynotbatman.com`**
