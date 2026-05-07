# Adaptive-Blade C2 Framework: Technical Overview

Adaptive-Blade là một khung làm việc Chỉ huy và Kiểm soát (Command and Control - C2) được thiết kế cho các hoạt động mô phỏng tấn công nâng cao. Hệ thống tập trung vào việc tối ưu hóa khả năng ẩn mình trước các giải pháp EDR/AV và duy trì tính bí mật trong quá trình truyền tải dữ liệu.

**Lưu ý quan trọng:** Đây là một dự án nghiên cứu tập trung vào thực thi các hoạt động Red Team (Red Team Operations) và mô phỏng lại các hành vi tấn công thực tế. Mục tiêu cốt lõi của dự án không dừng lại ở việc nghiên cứu kỹ thuật lý thuyết chuyên sâu, mà là vận dụng thực tiễn để hiểu rõ phương thức hoạt động của các mối đe dọa, từ đó xây dựng các chiến lược phát hiện và ngăn chặn hiệu quả ("Biết để Ngăn chặn"). Toàn bộ các kỹ thuật được liệt kê dưới đây đều đã được triển khai thực tế, kiểm chứng và vận hành ổn định trong môi trường mô phỏng.

## 1. Giao diện Quản lý (Web UI)

Adaptive-Blade cung cấp một giao diện quản lý hiện đại, cho phép điều khiển tập trung toàn bộ hạ tầng C2:

- **Dashboard:** Theo dõi trạng thái thời gian thực của các Agent, tài nguyên hệ thống và các tiến trình đang thực thi.
- **Task Management:** Giao diện trực quan để đẩy tác vụ, quản lý tệp tin và thực thi lệnh từ xa.
- **Configuration Builder:** Tích hợp bộ tạo Agent tự động với các tùy chọn tùy chỉnh linh hoạt.

<img src="https://i.ibb.co/bM7BQZjG/image.png">

## 2. Kiến trúc Hệ thống

Hệ thống được xây dựng trên mô hình phân tán nhằm giảm thiểu rủi ro bị phát hiện và tăng cường khả năng phục hồi:
- **Team Server:** Trung tâm quản lý viết bằng ngôn ngữ Go, điều phối tác vụ và quản lý trạng thái tác nhân (agent).
- **Agent (Implant):** Module thực thi trên mục tiêu, tối ưu hóa cho môi trường Windows/Linux với thiết kế module hóa.
- **Redirector:** Lớp đệm biên giúp che giấu địa chỉ IP thực của Team Server và lọc lưu lượng truy cập không mong muốn.
- **AI Engine:** Hỗ trợ phân tích hành vi và tự động hóa các phản hồi chiến thuật.
<img src="https://i.ibb.co/7NNFbGTn/image.png">
## 3. Kỹ thuật Lẩn tránh Tĩnh (Static Evasion)

Để vượt qua các cơ chế quét tệp tin và phân tích mã nguồn của các giải pháp bảo mật, Adaptive-Blade áp dụng các kỹ thuật sau (đã được tích hợp sẵn trong core builder):

- **Centralized Obfuscation:** Toàn bộ các biến nhạy cảm (C2 URL, AES Key, User-Agent) được mã hóa bằng thuật toán XOR kết hợp Hex Encoding trước khi biên dịch. Quá trình giải mã chỉ diễn ra trong bộ nhớ tại thời điểm thực thi.
- **Indirect Syscalls:** Thay vì sử dụng các hàm API Windows chuẩn (thường bị EDR hook), agent thực hiện các lệnh gọi hệ thống gián tiếp để thao tác với bộ nhớ và tiến trình.
- **Memory Injection Tactics:** Sử dụng các kỹ thuật tiên tiến như Module Stomping, Stack Spoofing và Mockingjay để thực thi mã trong không gian bộ nhớ của các tiến trình tin cậy.
- **Anti-Analysis Heuristics:** Kiểm tra môi trường thực thi (dung lượng RAM, ổ đĩa, số lượng CPU) để nhận diện môi trường Sandbox hoặc máy ảo của các nhà phát triển bảo mật, từ đó tự động điều chỉnh hành vi hoặc chấm dứt thực thi để bảo vệ mã nguồn.

## 4. Kỹ thuật Lẩn tránh Mạng (Network Evasion)

Giao thức truyền tải được thiết kế để hòa nhập vào lưu lượng mạng thông thường và đã được kiểm chứng qua các hệ thống giám sát lưu lượng (IDS/IPS):

- **WinHTTP Implementation:** Trên nền tảng Windows, agent sử dụng thư viện WinHTTP nguyên bản thay vì stack mạng mặc định của Go. Điều này giúp tránh việc tạo ra các dấu vân tay JA3 đặc trưng, vốn thường bị các hệ thống IDS/IPS phát hiện.
- **Stealth Gating:** Team Server tích hợp cơ chế lọc lưu lượng tại lớp ứng dụng. Chỉ những yêu cầu có User-Agent chính xác, đúng đường dẫn (standardized beacon path) và đi kèm Cookie được mã hóa hợp lệ mới được xử lý. Các yêu cầu không hợp lệ sẽ nhận phản hồi mã lỗi 404 để giả lập một máy chủ web không tồn tại.
- **Traffic Masking:** Dữ liệu giao tiếp được mã hóa AES và đóng gói trong các yêu cầu HTTP tiêu chuẩn, giả lập các hoạt động cập nhật phần mềm hoặc truy xuất API thông thường.
- **Infrastructure Hiding:** Sử dụng các Redirector kết hợp với kỹ thuật Domain Fronting (nếu cần thiết) để chuyển hướng lưu lượng, ngăn chặn việc truy vết ngược lại hạ tầng điều khiển trung tâm.
<img src="https://i.ibb.co/NdLyh7Zy/image.png">
## 4. Cơ chế Duy trì Sự hiện diện (Persistence)

Các phương thức duy trì được lựa chọn dựa trên tiêu chí ít để lại dấu vết (low footprint):
- **Ghost Task:** Tạo các tác vụ lập lịch không hiển thị trong các công cụ quản lý thông thường.
- **Phantom COM:** Sử dụng các đối tượng COM để kích hoạt lại tác nhân khi hệ thống có thay đổi trạng thái.
- **WMI Event Subscription:** Tận dụng Windows Management Instrumentation để thực thi mã khi các sự kiện hệ thống cụ thể xảy ra.

## 5. Chính sách Công bố và Tiếp cận Mã nguồn

Vì tính chất nhạy cảm của các kỹ thuật lẩn tránh nâng cao (Advanced Evasion) được triển khai trong dự án này, mã nguồn đầy đủ của Adaptive-Blade không được công khai rộng rãi. Quyết định này nhằm ngăn chặn việc các tác nhân xấu lợi dụng công cụ cho mục đích tấn công bất hợp pháp, gây ảnh hưởng đến an ninh mạng của các tổ chức.

Mã nguồn và các tài liệu kỹ thuật chi tiết chỉ được cung cấp cho mục đích nghiên cứu học thuật, kiểm thử bảo mật chuyên nghiệp hoặc phục vụ quá trình tuyển dụng trong các môi trường có sự kiểm soát.

## 6. Thông tin Liên hệ

Đối với bộ phận nhân sự (HR) hoặc các chuyên gia bảo mật có nhu cầu trao đổi trực tiếp, tìm hiểu sâu hơn về kiến trúc dự án hoặc đánh giá năng lực chuyên môn, vui lòng liên hệ qua địa chỉ email dưới đây:

- **Email:** khoa36015@gmail.com
- **Mục đích:** Trao đổi công việc, yêu cầu xem mã nguồn dự án (vui lòng đính kèm thông tin tổ chức/nhà tuyển dụng).

## 7. Kết luận

Adaptive-Blade không chỉ là một công cụ thực thi lệnh từ xa mà là một nền tảng nghiên cứu về tính bí mật trong an ninh mạng. Mọi tính năng được phát triển đều ưu tiên yếu tố OPSEC (Operations Security), đảm bảo rằng các cuộc mô phỏng tấn công phản ánh đúng năng lực của các nhóm đe dọa thực tế trong môi trường doanh nghiệp hiện đại.
