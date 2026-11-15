# GuardianCam Home (DEV-12)
**Hệ thống an ninh gia đình On-Device: Phát hiện người lạ & Té ngã người già**

* **Khối / Lĩnh vực:** AI Thiết bị & Edge AI
* **Nhóm thực hiện:**
  1. Trần Đức Mạnh — MSSV: `2A202603615`
  2. Nguyễn Anh Quân — MSSV: `[Điền MSSV]`
  3. Lê Anh Quân — MSSV: `2A202603576`
  4. Nguyễn Hữu Thịnh — MSSV: `[Điền MSSV]`

---

## 1. Tài liệu Gate 1 (Chốt đề tài)
* **1-Page Brief:** [docs/BRIEF.md](./docs/BRIEF.md)
* **Product Requirements Document (PRD):** [docs/PRD.md](./docs/PRD.md)
* **Wireframe & UI Flow:** [docs/WIREFRAME.md](./docs/WIREFRAME.md)
* **AI Logs & Workflow Setup:** [.ai-logs/README.md](./.ai-logs/README.md)

---

## 2. Tổng quan giải pháp & Kiến trúc On-Device
**GuardianCam Home** xử lý **100% luồng video tại thiết bị biên (Edge Hub / Jetson / PC mô phỏng)** nhằm giải quyết đồng thời 3 bài toán:
1. **Bảo vệ quyền riêng tư tuyệt đối:** Không đẩy video thô lên Cloud; tự động làm mờ (Gaussian Blur) khuôn mặt người lạ trên luồng hiển thị.
2. **Giảm báo động giả (False Positives):** Kết hợp **YOLOv8-nano** + **MobileFaceNet** để nhận diện thành viên gia đình và lập luận ngữ cảnh (giờ giới nghiêm, khu vực, loại trừ thú cưng/bóng đổ).
3. **Phát hiện té ngã người cao tuổi:** Sử dụng **MoveNet / BlazePose** kết hợp logic chuỗi thời gian (temporal logic) và lớp khóa liên động an toàn xác định bằng code (Deterministic Safety Guardrail), ưu tiên tối đa chỉ số **Recall**.

### Techstack mục tiêu
* **Edge AI & Vision:** Python, OpenCV, ONNX Runtime / TensorFlow Lite (YOLOv8-nano, MobileFaceNet, MoveNet/BlazePose).
* **Backend & Messaging:** FastAPI, Mosquitto (Local MQTT), SQLite.
* **Frontend:** React, TailwindCSS (Web App phân quyền Admin/Caregiver & Family Member + xác nhận HITL).
* **Deployment & Eval:** Docker, bộ dữ liệu chuẩn **UR Fall Detection** & kịch bản video thực tế.

---

## 3. Cấu trúc Repository

    guardiancam-home-dev12/
    ├── docs/                   # Tài liệu thiết kế Gate 1 (BRIEF.md, PRD.md, WIREFRAME.md)
    ├── .ai-logs/               # Nhật ký tương tác AI & cấu hình quy chuẩn sử dụng AI của đội
    ├── .github/                # Pull Request template phục vụ quy trình >= 10 PRs ở Gate 2
    ├── edge-agent/             # Pipeline xử lý thị giác biên, ẩn danh khuôn mặt & safety guardrails
    ├── backend/                # FastAPI server, MQTT client, Context Agent & HITL API
    ├── frontend/               # React Dashboard giám sát trực tiếp, đăng ký khuôn mặt & telemetry
    └── README.md               # Tài liệu tổng quan dự án
