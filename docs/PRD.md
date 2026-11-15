# PRODUCT REQUIREMENTS DOCUMENT (PRD): GUARDIANCAM HOME

* **Mã đề tài:** DEV-12
* **Tên sản phẩm:** GuardianCam Home — Hệ thống an ninh gia đình On-Device
* **Phiên bản:** v1.0 (Gate 1 - Chốt thiết kế)
* **Nhóm thực hiện:**
  1. Trần Đức Mạnh — MSSV: `2A202603615`
  2. Nguyễn Anh Quân — MSSV: `2A202603950`
  3. Lê Anh Quân — MSSV: `2A202603576`
  4. Nguyễn Hữu Thịnh — MSSV: `2A202603726`

---

## 1. Mục tiêu sản phẩm & Phạm vi (Product Scope)
Xây dựng hệ thống Agent thị giác an ninh gia đình vận hành **100% tại thiết bị biên (On-Device / Edge Hub)**, giải quyết 3 bài toán cốt lõi:
1. **Bảo vệ quyền riêng tư tuyệt đối:** Không gửi luồng video thô lên Cloud; tự động ẩn danh (làm mờ) khuôn mặt người ngoài khung hình.
2. **Giảm báo động giả (False Positives):** Phân biệt chính xác thành viên gia đình với người lạ và lọc nhiễu môi trường (thú cưng, bóng đổ, ngược sáng) thông qua lập luận ngữ cảnh.
3. **An toàn chủ động cho người cao tuổi:** Phát hiện té ngã hoặc bất động đột ngột với độ nhạy (Recall) cao và cảnh báo tức thì cho người thân.

---

## 2. Phân quyền người dùng (User Roles & Personas)
Hệ thống Web App & Dashboard hỗ trợ tối thiểu **2 vai trò** đăng nhập rõ ràng:

| Vai trò | Mô tả | Quyền hạn chính |
| :--- | :--- | :--- |
| **1. Quản trị viên / Người chăm sóc (Admin / Caregiver)** | Người quản lý an ninh chính hoặc người thân chăm sóc người cao tuổi | • Đăng ký, cập nhật, xóa dữ liệu khuôn mặt thành viên (Face Enrollment).<br>• Cấu hình vùng giám sát, lịch trình gia đình và khung giờ giới nghiêm.<br>• Xem luồng video giám sát (đã ẩn danh người lạ) và toàn bộ lịch sử cảnh báo.<br>• Thực hiện xác nhận báo động **HITL (Human-in-the-Loop)** (Đúng/Sai).<br>• Xem bảng thông số hiệu năng phần cứng biên (Telemetry). |
| **2. Thành viên gia đình (Family Member)** | Các thành viên sinh sống trong nhà | • Được hệ thống nhận diện là "Người quen" để không kích hoạt báo động xâm nhập.<br>• Xem trạng thái an ninh tổng quan của ngôi nhà.<br>• Nhận thông báo khẩn cấp khi có sự cố té ngã hoặc xâm nhập nghiêm trọng. |

---

## 3. Yêu cầu chức năng (Functional Requirements)

### FR-01: Pipeline Thị giác tại biên (On-Device Vision Pipeline)
* **Đầu vào:** Hỗ trợ luồng camera trực tiếp hoặc video mô phỏng kịch bản thực tế (MP4/RTSP) trên môi trường Edge Hub / Jetson / PC mô phỏng.
* **Nhận diện người & khuôn mặt:**
  * Sử dụng **YOLOv8-nano** (ONNX/TFLite) để phát hiện đối tượng người (`person`) và loại trừ vật nuôi (`cat`, `dog`).
  * Sử dụng **MobileFaceNet** để trích xuất vector đặc trưng khuôn mặt và so khớp với cơ sở dữ liệu khuôn mặt thành viên cục bộ (Cosine Similarity).
* **Phát hiện té ngã (Fall Detection):**
  * Sử dụng mô hình ước lượng tư thế **MoveNet / BlazePose** kết hợp logic theo dõi chuỗi thời gian (temporal logic: kiểm tra góc nghiêng cơ thể, vận tốc hạ thấp trọng tâm và thời gian bất động > `T` giây).
  * **Nguyên tắc ưu tiên:** Tối ưu hóa chỉ số **Recall** để tuyệt đối không bỏ sót tình huống người già té ngã.

### FR-02: Bảo vệ quyền riêng tư & Ẩn danh hóa (Privacy Guardrail)
* Mọi frame hình trước khi hiển thị lên Dashboard hoặc lưu trữ cục bộ phải đi qua bộ lọc ẩn danh:
  * Nếu là **Người lạ (Stranger)**: Tự động áp dụng Gaussian Blur lên vùng bounding box khuôn mặt.
  * Nếu là **Thành viên (Member)**: Hiển thị nhãn nhận diện kèm độ tin cậy (Confidence Score).
* Tuyệt đối **không** truyền tải dữ liệu video thô ra mạng Internet bên ngoài.

### FR-03: Bộ nhớ ngữ cảnh & Agent cảnh báo tự nhiên (Context Memory & Alert Agent)
* **Memory cục bộ:** Lưu trữ vector khuôn mặt thành viên, danh sách khu vực camera (Cửa trước, Cửa sau, Phòng khách) và lịch trình/giờ giới nghiêm (ví dụ: 23:00 – 05:30).
* **Lập luận ngữ cảnh & Sinh cảnh báo:** Agent tổng hợp sự kiện thị giác + ngữ cảnh để phân loại mức độ nghiêm trọng (`LOW`, `MEDIUM`, `CRITICAL`) và sinh mô tả tiếng Việt tự nhiên:
  * *Ví dụ 1 (Xâm nhập):* `"CẢNH BÁO CAO: Phát hiện người lạ xuất hiện tại khu vực Cửa sau lúc 02:15 sáng (ngoài giờ an toàn)."`
  * *Ví dụ 2 (Té ngã):* `"KHẨN CẤP: Phát hiện người cao tuổi có dấu hiệu té ngã và bất động tại Phòng khách lúc 14:30."`
  * *Ví dụ 3 (Loại nhiễu):* `"THÔNG TIN: Nhận diện thành viên [Trần Đức Mạnh] về nhà tại Cửa trước lúc 20:10."` (Không bật còi/báo động).

### FR-04: Xác nhận báo động Human-in-the-Loop (HITL)
* Trên giao diện cảnh báo, người chăm sóc có 2 nút hành động cho mỗi sự kiện: **[Xác nhận đúng sự cố]** hoặc **[Báo động giả (False Positive)]**.
* Dữ liệu phản hồi HITL được lưu lại để tính toán tự động các chỉ số **Precision**, **Recall**, và **False Positive Rate (FPR)** phục vụ đánh giá mô hình.

---

## 4. Yêu cầu phi chức năng & An toàn (Non-Functional & Safety Guardrails)

### NFR-01: Khóa liên động an toàn xác định (Deterministic Safety Guardrail)
* Tích hợp lớp kiểm tra bằng code thuần túy (rule-based verification) trước khi kích hoạt bất kỳ hành động cảnh báo khẩn cấp nào:
  * Yêu cầu sự kiện phải duy trì liên tục qua tối thiểu `N` frames (ví dụ: 5 frames liên tiếp đối với té ngã) để chống nhiễu rung giật khung hình.
  * Không cho phép mô hình ngôn ngữ (SLM) tự ý tắt chế độ giám sát hoặc bỏ qua cảnh báo té ngã mức `CRITICAL`.

### NFR-02: Vận hành Offline-First & Hiệu năng phần cứng biên
* Hệ thống phải hoạt động độc lập **100% khi ngắt kết nối Internet** (Offline mode).
* Hỗ trợ đo lường và xuất báo cáo Telemetry phần cứng thực tế:
  * Tốc độ xử lý (**FPS**).
  * Độ trễ xử lý end-to-end (**p95 latency** tính bằng ms).
  * Mức tiêu thụ bộ nhớ (**RAM / VRAM**) và dung lượng file mô hình (**MB**).
  * Bảng so sánh đánh đổi hiệu năng và độ chính xác giữa các mức lượng tử hóa mô hình (**FP32 vs Q8 vs Q4**).

### NFR-03: Bảo mật lưu trữ cục bộ (Gate 3 Target)
* Các đoạn clip sự kiện quan trọng (đã làm mờ mặt người lạ) khi lưu trữ tại thiết bị biên phải được mã hóa cục bộ (AES-256).

---

## 5. Kiến trúc hệ thống & Techstack

```text
[Camera / Video Input]
        │
        ▼ (100% On-Device Processing)
┌───────────────────────────────────────────────────────────┐
│ EDGE AGENT (Python, OpenCV, ONNX Runtime / TFLite)        │
│  ├─ YOLOv8-nano (Person Detection & Pet Filtering)        │
│  ├─ MobileFaceNet (Member vs Stranger Classification)     │
│  ├─ MoveNet/BlazePose + Temporal Logic (Fall Detection)   │
│  └─ Privacy Filter (Blur Stranger Face) & Safety Guard    │
└───────────────────────────┬───────────────────────────────┘
                            │ Local MQTT / WebSocket
                            ▼
┌───────────────────────────────────────────────────────────┐
│ BACKEND & CONTEXT AGENT (FastAPI, SQLite/Local Memory)    │
│  ├─ Context Reasoning (Time, Zone, Member Schedule)       │
│  ├─ Vietnamese Alert Generator & Metrics Tracker          │
│  └─ Auth & Role Management (Admin / Family Member)        │
└───────────────────────────┬───────────────────────────────┘
                            │ REST API / WebSocket
                            ▼
┌───────────────────────────────────────────────────────────┐
│ FRONTEND DASHBOARD (React)                                │
│  ├─ Live Anonymized Stream & Hardware Telemetry (FPS/RAM) │
│  ├─ Vietnamese Alert Feed + HITL Verification Buttons     │
│  └─ Member Face Enrollment & Curfew Schedule Settings     │
└───────────────────────────────────────────────────────────┘
