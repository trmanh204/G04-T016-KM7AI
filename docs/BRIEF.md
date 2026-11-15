# 1-PAGE PROJECT BRIEF: GUARDIANCAM HOME

* **Mã đề tài:** DEV-12
* **Tên đề tài:** GuardianCam Home — Hệ thống an ninh gia đình On-Device: Phát hiện người lạ & Té ngã người già
* **Khối / Lĩnh vực:** AI Thiết bị & Edge AI
* **Thành viên thực hiện:**
  1. Trần Đức Mạnh — MSSV: `2A202603615`
  2. Nguyễn Anh Quân — MSSV: `2A202603950`
  3. Lê Anh Quân — MSSV: `2A202603576`
  4. Nguyễn Hữu Thịnh — MSSV: `2A202603726`

---

## 1. Bối cảnh & Vấn đề cốt lõi (Problem Statement)
Các giải pháp camera an ninh gia đình truyền thống hiện tồn tại ba điểm nghẽn lớn khi đưa vào vận hành thực tế:
* **Rủi ro quyền riêng tư & phụ thuộc Cloud:** Việc truyền tải video thô lên máy chủ đám mây tiêu tốn băng thông, gây lo ngại rò rỉ dữ liệu sinh hoạt nhạy cảm và mất khả năng bảo vệ khi gián đoạn kết nối Internet.
* **Báo động giả (False Positives) gây nhiễu:** Cảm biến chuyển động thông thường dễ bị kích hoạt nhầm bởi vật nuôi, bóng đổ, ngược sáng hoặc thành viên trong gia đình đi lại, dẫn đến tình trạng "nhờn" cảnh báo.
* **Thiếu giám sát an toàn chủ động cho người cao tuổi:** Người già sống một mình đối mặt với nguy cơ té ngã hoặc bất động đột ngột nhưng khó được phát hiện kịp thời do thiếu cơ chế phân tích hành vi liên tục tại chỗ.

## 2. Giải pháp đề xuất (Proposed Solution)
**GuardianCam Home** là hệ thống Agent thị giác an ninh vận hành **100% On-Device (Edge AI)** trên thiết bị biên (Edge Hub / Jetson / PC mô phỏng), kết hợp nhận diện đa tác vụ, bộ nhớ ngữ cảnh và lập luận an toàn:
* **Nhận diện người quen / người lạ tại biên:** Tích hợp pipeline phát hiện người và trích xuất đặc trưng khuôn mặt nhẹ (**YOLOv8-nano + MobileFaceNet** định dạng ONNX/TFLite), đối chiếu với bộ nhớ khuôn mặt thành viên đã đăng ký để phân loại chính xác người nhà và người ngoài.
* **Phát hiện té ngã & bất động (Fall Detection):** Ứng dụng ước lượng tư thế (**MoveNet / BlazePose**) kết hợp logic phân tích theo chuỗi thời gian (temporal logic) để nhận diện trạng thái ngã hoặc nằm bất động bất thường, ưu tiên tối đa độ nhạy (**Recall**) để không bỏ sót sự cố của người cao tuổi.
* **Agent lập luận ngữ cảnh & Cảnh báo tiếng Việt tự nhiên:** Đối chiếu sự kiện thị giác với bộ nhớ lịch trình, vị trí camera và khung giờ (ví dụ: giờ giới nghiêm ban đêm) để loại trừ nhiễu (thú cưng, ánh sáng) và xuất cảnh báo ngôn ngữ tự nhiên rõ ràng (ví dụ: *"Phát hiện người lạ tại khu vực cửa sau lúc 02:15 sáng"*).

## 3. Đối tượng sử dụng (Target Users)
* **Quản trị viên / Người chăm sóc (Admin / Caregiver):** Thiết lập vùng giám sát và khung giờ an toàn, đăng ký dữ liệu khuôn mặt thành viên (Face Enrollment), nhận cảnh báo khẩn cấp và thực hiện xác nhận báo động (**Human-in-the-Loop - HITL**).
* **Thành viên gia đình (Family Member):** Được lưu trong danh sách nhận diện an toàn để giảm thiểu báo động giả; có quyền truy cập giao diện theo dõi trạng thái an ninh chung của ngôi nhà.

## 4. Ràng buộc kỹ thuật & Cam kết an toàn (Key Constraints)
* **On-Device Privacy & Ẩn danh hóa:** Toàn bộ dữ liệu hình ảnh được xử lý cục bộ trên thiết bị biên; tự động làm mờ (blur) khuôn mặt người ngoài khung hình và tuyệt đối không gửi video thô lên cloud.
* **Offline-First & Tối ưu phần cứng biên:** Hoạt động ổn định ngay cả khi mất kết nối mạng hoàn toàn; tối ưu hóa đánh đổi giữa dung lượng mô hình lượng tử hóa (**Q4 / Q8**) với độ chính xác và độ trễ phản hồi tức thì trong giới hạn bộ nhớ **RAM/VRAM**.
* **Deterministic Safety Guardrail:** Tích hợp lớp kiểm tra an toàn xác định bằng code trước mọi lệnh điều khiển hoặc kích hoạt cảnh báo khẩn cấp để tránh sai sót từ mô hình AI.

## 5. Kiến trúc & Lộ trình mục tiêu (Scope & Milestones)
* **Techstack cốt lõi:** Python (**FastAPI**, **ONNX Runtime / TFLite**, **OpenCV**), **React** (Web App & Dashboard), **MQTT** (giao tiếp Edge – Backend), **Docker**, cùng bộ dữ liệu kiểm thử tham chiếu (**UR Fall Detection**, video mô phỏng kịch bản thực tế).
* **Mục tiêu Gate 2 (MVP):** Hoàn thiện luồng xử lý end-to-end trên video mô phỏng (người nhà, người lạ ban đêm, người già té ngã, thú cưng) trên Web App phân quyền tối thiểu 2 vai trò (≥ 2 roles), có chức năng làm mờ khuôn mặt, xác nhận HITL và đo lường các chỉ số *Precision, Recall, False Positive Rate*.
* **Mục tiêu Gate 3 (Production Ready):** Tối ưu kiến trúc phối hợp Edge – Cloud, hoàn thiện cơ chế lưu trữ video mã hóa cục bộ và xuất báo cáo benchmark phần cứng thực tế (**FPS, p95 latency, RAM/VRAM, so sánh Q4 vs Q8**) kèm kết quả thử nghiệm với tối thiểu 5 người dùng thực tế (≥ 5 users).
