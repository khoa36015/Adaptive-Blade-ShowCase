# Adaptive-Blade C2 Framework: Technical ShowCase

Adaptive-Blade là một khung làm việc Chỉ huy và Điều khiển (Command and Control - C2) bậc cao, được thiết kế cho các hoạt động mô phỏng Red Team chuyên nghiệp. Hệ thống tập trung vào việc vượt qua các giải pháp EDR/AV hiện đại (như Kaspersky Endpoint Security) và duy trì tính vững chắc trong môi trường doanh nghiệp.

Lưu ý: Đây là dự án nghiên cứu nhằm mục đích bảo mật và đánh giá năng lực hệ thống. Toàn bộ các kỹ thuật mô tả dưới đây đều đã được triển khai thực tế và kiểm chứng ổn định.

## 1. Thành phần Hệ thống

Hệ thống được cấu trúc thành 3 phần chính để tối ưu hóa hiệu suất và khả năng ẩn mình:

- Team Server (Backend): Phát triển trên nền tảng Python, quản lý phiên (session) qua token đặc hiệu (X-Session-Token), tích hợp engine biên dịch động với kỹ thuật chèn junk code metamorphic.
- CLI Client: Giao diện điều khiển dòng lệnh (Interactive CLI) viết bằng Python, hỗ trợ tự động hóa (Auto-Injection) và quản lý Agent linh hoạt.
- Agent (Implant): Module thực thi viết bằng C++, tối ưu hóa cho môi trường Windows với kiến trúc không phụ thuộc vào IAT (Import Address Table) cho các API nhạy cảm.

## 2. Kỹ thuật Lẩn tránh Nâng cao (Advanced Evasion)

Điểm mạnh cốt lõi của Adaptive-Blade nằm ở khả năng "tàng hình" trước các cơ chế phát hiện hành vi (Behavioral Analysis):

- Indirect Syscalls (DJB2 Hashing): Sử dụng lệnh gọi hệ thống gián tiếp để thao tác với bộ nhớ và tiến trình. Kỹ thuật này hoàn toàn vô hiệu hóa các cơ chế Hooking của EDR/AV tại lớp User-mode (ntdll.dll).
- Memory Injection (NtMapViewOfSection): Thay vì sử dụng WriteProcessMemory truyền thống dễ bị phát hiện, Agent sử dụng Sections để map vùng nhớ, kết hợp với NtQueueApcThread để thực thi code, giúp tránh khỏi các dấu vết "Private/Unbacked Memory".
- PPID Spoofing: Khởi tạo tiến trình con (như werfault.exe) dưới danh nghĩa các tiến trình tin cậy (như explorer.exe) để duy trì một cây tiến trình (Process Tree) hợp lệ.
- Sleep Masking: Mã hóa vùng nhớ thực thi của Agent trong thời gian chờ (Sleep) để tránh các cơ chế quét bộ nhớ (Memory Scanning) của AV.

## 3. Giao thức Truyền tải và OPSEC (Network Stealth)

Hệ thống giao tiếp được thiết kế để hòa nhập hoàn toàn vào lưu lượng mạng thông thường:

- WinHTTP Native Engine: Sử dụng stack mạng WinHTTP nguyên bản của Windows thay vì các thư viện bên thứ ba, giúp tạo ra dấu vân tay mạng (JA3/TLS) giống hệt với các tiến trình hệ thống hợp lệ.
- Malleable C2 Profiles: Cấu hình linh hoạt Header, URI và cấu trúc dữ liệu (HTML/JSON) thông qua profile.json. Toàn bộ dữ liệu beacon được mã hóa AES và che giấu trong các yêu cầu HTTP tiêu chuẩn.
- Stealth Gating: Hệ thống chỉ chấp nhận các kết nối có chữ ký và token hợp lệ. Các yêu cầu truy cập bất thường từ Sandbox hoặc Crawler sẽ nhận phản hồi 404 giả lập.

## 4. Chống phân tích Pháp y (Anti-Forensics)

- Payload Inflation: Tự động thổi phồng kích thước payload (.bin) từ 5MB đến 10MB với entropy biến thiên để vượt qua các engine quét tĩnh giới hạn kích thước.
- Bait String Injection: Chèn các chuỗi ký tự từ các thư viện nổi tiếng (như Qt5, SQLite) vào payload để làm giả lập một phần mềm thương mại.
- Advanced Melt: Cơ chế tự hủy (Self-Deletion) xóa sạch dấu vết Agent trên đĩa cứng ngay sau khi hoàn thành nhiệm vụ hoặc theo lệnh điều khiển.

## 5. Video Showcase (Demonstration)

Vị trí dưới đây trình bày khả năng thực thi của Adaptive-Blade trên môi trường có cài đặt Kaspersky Endpoint Security (KES). Video bao gồm các thao tác:
- Khởi tạo và kết nối Agent (Beaconing).
- Thực thi lệnh hệ thống (Shell Command).
- Tải lên/Tải xuống tệp tin (Upload/Download) mà không bị chặn bởi engine phát hiện hành vi của KES.
![DEMO](./showcase.mp4)

## 6. Thông tin Liên hệ

Dự án này được phát triển cho mục đích khảo sát và tuyển dụng chuyên môn. Quý nhà tuyển dụng hoặc chuyên gia bảo mật quan tâm đến kiến trúc chi tiết và mã nguồn có thể liên hệ:

- Email: khoa36015@gmail.com
- Lĩnh vực: Red Team Operations, Malware Development, Evasion Techniques.

Kết luận: Adaptive-Blade không chỉ là một công cụ C2, mà là một minh chứng cho việc vận dụng các kỹ thuật Low-level để vượt qua các rào cản bảo mật hiện đại nhất hiện nay.
