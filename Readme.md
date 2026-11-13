ADAS-23 — WeatherForge - Mô phỏng điều kiện môi trường (mưa, sương, nắng gắt, đêm, mặt đường ướt) và ma trận phủ ODD cho kiểm thử perception
ADAS – Hệ thống hỗ trợ lái & Xe tự hành

2/2 đội chọn
Khối / Lĩnh vực
ADAS – Hệ thống hỗ trợ lái & Xe tự hành
Mã đề
ADAS-23
Tên đề tài
WeatherForge - Mô phỏng điều kiện môi trường (mưa, sương, nắng gắt, đêm, mặt đường ướt) và ma trận phủ ODD cho kiểm thử perception
Mô tả bài toán
📍 Thực trạng: Perception thường được kiểm thử chủ yếu trong thời tiết đẹp vì dữ liệu mưa/sương/đêm khó thu thập có kiểm soát; còn khi mô phỏng thì điều kiện môi trường thường chỉnh thủ công, không có ma trận rõ ràng cho biết các tổ hợp điều kiện (ODD) đã được kiểm thử đủ chưa.

🎯 Vấn đề: Bài toán đặt ra là làm thế nào để xây dựng công cụ sinh điều kiện môi trường có kiểm soát (cường độ mưa, độ dày sương, góc mặt trời/chói, ban đêm, mặt đường ướt phản chiếu) trong mô phỏng hoặc bằng biến đổi ảnh có điều kiện, chạy perception qua ma trận điều kiện, hiển thị độ phủ ODD và độ giảm hiệu năng theo từng ô, và để agent đề xuất các tổ hợp điều kiện chưa được kiểm thử; đồng thời kết hợp mô hình học sâu/VLM với tầng kiểm tra ràng buộc hình học – động học xác định, ước lượng độ bất định có hiệu chuẩn và hỗ trợ kỹ sư khoanh vùng nguyên nhân lỗi hoặc khai thác ca khó tự động.

🔒 Ràng buộc: kỹ sư duyệt định nghĩa ODD và ngưỡng chấp nhận hiệu năng (human-in-the-loop); chỉ chạy trên mô phỏng và dữ liệu công khai, không dùng ảnh biến đổi như thể là dữ liệu thật khi báo cáo; độ thực của ảnh sinh đo bằng FID/KID hoặc so với ảnh thật cùng điều kiện, kèm độ giảm AP của detector; phải ghi rõ giới hạn của phương pháp sinh; kiểm soát chi phí GPU khi sinh dữ liệu hàng loạt.

Techstack gợi ý
• Python, PyTorch
• CARLA weather/lighting API hoặc Isaac Sim
• image-to-image/diffusion có điều kiện để thêm sương/mưa/đêm (có kiểm soát)
• ACDC, BDD100K, nuScenes (cảnh mưa/đêm) làm tham chiếu thực
• detector mẫu (YOLO/RT-DETR)
• clean-fid/torchmetrics để tính FID/KID
• FastAPI backend
• React/Next.js (ma trận ODD dạng heatmap)
• Docker + GPU.
• Dữ liệu & benchmark thực tế tham chiếu: Bộ dữ liệu định vị & bản đồ KITTI Odometry, Argoverse 2 Map Change, NuPlan / Lanelet2 / ASAM OpenDRIVE và OpenStreetMap.

Yêu cầu đầu ra gợi ý
Cơ bản:
• web app định nghĩa ma trận điều kiện (≥3 chiều: ánh sáng, mưa/sương, mặt đường), sinh dữ liệu từ mô phỏng, chạy một detector, heatmap AP theo ô điều kiện và độ phủ ODD, ≥2 vai trò (kỹ sư và reviewer), HITL duyệt ngưỡng chấp nhận.

Nâng cao:
• agent đề xuất tổ hợp điều kiện chưa phủ và tự sinh dữ liệu kiểm thử bổ sung, đánh giá độ thực của ảnh sinh (FID/KID, so với ảnh thật), phân tích loại lỗi tăng mạnh nhất theo điều kiện, đo chi phí - thời gian sinh trên mỗi ô điều kiện
• Kết hợp kiểm chứng ràng buộc vật lý/hình học xác định với bộ benchmark đánh giá độ bền vững trên các ca biên thực tế và mở rộng công cụ chẩn đoán chuyên sâu cho đội ngũ kỹ sư.