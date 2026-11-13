# WeatherForge (ADAS-23): Kiểm thử độ bền vững của Perception trong điều kiện môi trường bất lợi

## 1. Tổng quan

WeatherForge là nền tảng giúp kỹ sư **kiểm thử mô hình AI Perception** của xe tự hành/ADAS dưới các điều kiện mưa, sương mù, nắng gắt, ban đêm và mặt đường ướt.

Quy trình tổng quát:
1. Tạo tình huống có điều kiện môi trường được kiểm soát.
2. Chạy mô hình nhận diện trên từng tình huống.
3. Đo mức suy giảm hiệu năng.
4. Hiển thị kết quả trên **ma trận ODD**.

Một **AI Agent** phân tích kết quả và đề xuất tình huống cần kiểm thử thêm. Mọi đề xuất và ngưỡng đánh giá đều phải được **kỹ sư phê duyệt**.

## 2. Vấn đề thực tế

Mô hình Perception (nhận diện ô tô, xe máy, người đi bộ, biển báo, làn đường) suy giảm đáng kể khi môi trường xấu đi:

| Điều kiện | Ảnh hưởng |
|---|---|
| Mưa lớn | Ảnh kém chất lượng, vật thể bị che khuất |
| Sương mù | Giảm tầm nhìn, khó phát hiện vật ở xa |
| Ban đêm | Thiếu sáng, khó nhận diện người và xe |
| Nắng gắt, ngược sáng | Chói, mất chi tiết ảnh |
| Đường ướt | Phản chiếu ánh sáng gây nhận diện nhầm |

Dữ liệu thật cho các tình huống này **khó thu thập**: phụ thuộc thời tiết, địa điểm, thời gian, và tốn chi phí gán nhãn. Ngoài ra chưa có cách hệ thống hóa các tổ hợp điều kiện, nên kỹ sư khó trả lời các câu hỏi sau:
- Đã kiểm thử những điều kiện nào, còn tổ hợp nào chưa kiểm thử?
- Điều kiện nào làm mô hình suy giảm nhiều nhất?
- Kết quả có đủ tin cậy không?
- Cần ưu tiên kiểm thử gì tiếp?

## 3. Mục tiêu

> **Giúp kỹ sư xác định giới hạn hoạt động của mô hình Perception:** mô hình yếu ở điều kiện nào, tổ hợp nào chưa được kiểm thử, và có cơ sở đánh giá độ tin cậy trước khi triển khai thực tế.

Để đạt mục tiêu này, hệ thống cần các thành phần:
1. **Mô phỏng có kiểm soát:** tạo các điều kiện thời tiết và ánh sáng.
2. **Ma trận ODD:** chia không gian kiểm thử thành các tổ hợp điều kiện, theo dõi độ phủ.
3. **Đánh giá hiệu năng:** đo độ chính xác và mức suy giảm theo từng điều kiện.
4. **Đánh giá độ tin cậy:** đo độ bất định, phát hiện dự đoán thiếu tin cậy, kiểm tra ràng buộc hình học và vật lý.
5. **AI Agent:** tìm vùng yếu hoặc chưa phủ, đề xuất kiểm thử bổ sung.
6. **Dashboard và HITL:** trực quan hóa kết quả, cung cấp cơ chế phê duyệt của kỹ sư.

## 4. Quy trình cốt lõi

**Định nghĩa ODD → Mô phỏng → Chạy Perception → Đánh giá → Agent phân tích và đề xuất**

**B1. Định nghĩa ma trận ODD.** Mỗi tổ hợp điều kiện là một ô kiểm thử, ví dụ *ban đêm + mưa lớn + đường ướt*.

| Chiều | Các mức |
|---|---|
| Ánh sáng | Ngày, hoàng hôn, đêm, ngược sáng |
| Thời tiết | Quang, mưa nhẹ, mưa lớn, sương mù |
| Mặt đường | Khô, ướt nhẹ, ướt nhiều |

Mỗi ô có một trong các trạng thái: *Chưa kiểm thử / Đã kiểm thử / Đạt / Không đạt*.

**B2. Mô phỏng môi trường.** CARLA cho phép điều chỉnh cường độ mưa, độ dày sương, góc mặt trời, thời gian trong ngày, độ ướt mặt đường và thông số camera, đồng thời xuất ảnh kèm nhãn. Image-to-Image/Diffusion là hướng mở rộng. **Dữ liệu sinh phải được đánh dấu rõ, không trình bày như dữ liệu thật.**

**B3. Chạy Perception.** Dùng YOLO làm baseline để nhận diện ô tô, xe máy, người đi bộ và xe đạp. Lưu lại bounding box, confidence và nhãn thật, rồi tổng hợp theo từng ô ODD.

**B4. Đánh giá.**
- Chỉ số chính: Precision, Recall, AP/mAP, confidence và latency. So sánh với điều kiện bình thường để ra mức suy giảm.
- Phần nâng cao: độ bất định, hiệu chuẩn bằng ECE, phân tích lỗi bỏ sót và nhận nhầm, ràng buộc hình học.
- FID/KID dùng để so ảnh sinh với ảnh thật, nhưng **chỉ mang tính tham khảo**.

**B5. Dashboard.** Gồm heatmap AP theo từng ô, độ phủ ODD, so sánh giữa các điều kiện, ảnh kèm kết quả nhận diện, thống kê lỗi và chi phí GPU.

| Điều kiện | mAP (minh họa) | Trạng thái |
|---|---:|---|
| Ngày + quang + khô | 95% | Đạt |
| Đêm + mưa lớn + ướt | 55% | Không đạt |
| Sương mù + ướt | – | Chưa kiểm thử |

**B6. AI Agent.** Agent đọc kết quả, tìm ô yếu hoặc chưa phủ, phân tích loại lỗi và đề xuất kiểm thử theo mức độ rủi ro.

> Ví dụ đề xuất: *"Đêm + mưa lớn + đường ướt có tỷ lệ bỏ sót người đi bộ cao. Đề xuất sinh thêm 100 tình huống với các mức mưa khác nhau."*

Agent **không tự đổi ngưỡng hay chạy kiểm thử lớn** khi chưa được duyệt.

## 5. Vai trò và HITL

| Engineer | Reviewer |
|---|---|
| Tạo ma trận ODD, thiết lập điều kiện | Duyệt hoặc từ chối đề xuất kiểm thử |
| Chạy mô phỏng và Perception | Thiết lập và phê duyệt ngưỡng hiệu năng |
| Xem heatmap, báo cáo, đề xuất của Agent | Theo dõi độ phủ, lịch sử, nguồn gốc dữ liệu |

**Các hoạt động bắt buộc phải duyệt:** định nghĩa ODD, ngưỡng chấp nhận, đề xuất của Agent, kết luận đánh giá.

## 6. Kiến trúc và công nghệ

| Thành phần | Công nghệ | Vai trò |
|---|---|---|
| Frontend | React/Next.js | Quản lý ODD, heatmap, vai trò, giao diện Agent |
| Backend | FastAPI | API, phân quyền, điều phối tác vụ (worker riêng cho tác vụ nặng) |
| Mô phỏng | CARLA | Tạo môi trường, điều chỉnh thời tiết, xuất ảnh kèm nhãn |
| Perception | PyTorch, YOLO/RT-DETR | Nhận diện và tính chỉ số |
| Đánh giá | torchmetrics, clean-fid | AP, độ phủ, ECE, FID/KID, ràng buộc hình học |
| AI Agent | LLM API / LangGraph | Phân tích và đề xuất |
| Lưu trữ | PostgreSQL, Object Storage | Cấu hình, kết quả, hình ảnh |
| Triển khai | Docker, CUDA | Đóng gói, GPU |

**Dữ liệu tham chiếu:**
- BDD100K, ACDC (thời tiết xấu), nuScenes, KITTI.
- CARLA dùng để sinh dữ liệu mô phỏng.
- Cần kiểm tra tương thích nhãn giữa các bộ trước khi đưa vào pipeline.

## 7. Phạm vi đầu ra

**MVP (bắt buộc):**
- Web app có ≥2 vai trò.
- Ma trận ODD ≥3 chiều.
- Dùng CARLA sinh ảnh kèm nhãn.
- Tích hợp YOLO, tính AP/mAP theo từng ô.
- Heatmap hiệu năng và độ phủ.
- Reviewer duyệt ngưỡng.
- Lưu lịch sử kiểm thử.

**Nâng cao:**
- **A. Agent:** đề xuất ô chưa phủ hoặc ô yếu, ưu tiên theo rủi ro.
- **B. Sinh dữ liệu có mục tiêu:** kiểm soát số lượng và chi phí GPU theo từng ô, kiểm tra tính nhất quán của nhãn.
- **C. Chất lượng ảnh sinh:** dùng FID/KID, so sánh với ảnh thật cùng điều kiện, phân biệt rõ dữ liệu thật và dữ liệu sinh.
- **D. Lỗi và độ tin cậy:** thống kê FP/FN, hiệu chuẩn độ bất định, kiểm tra ràng buộc vật lý.
- **E. Đánh giá mở rộng:** so sánh nhiều mô hình, đánh giá trên dữ liệu thật độc lập, xây benchmark các ca biên.

## 8. Tiêu chí thành công

| Nhóm | Chỉ số |
|---|---|
| Độ phủ | ODD Coverage |
| Hiệu năng | Precision, Recall, AP/mAP, AP Degradation theo ô |
| Độ tin cậy | ECE |
| Ảnh sinh | FID/KID |
| Vận hành | Latency, thời gian và chi phí GPU |
| Phân tích | Khả năng phát hiện ô yếu, chất lượng đề xuất của Agent |

- Tập đánh giá phải **độc lập** với tập huấn luyện. Với video, không để khung hình gần nhau hoặc cùng cảnh rơi vào cả hai tập.
- Ngưỡng đánh giá do kỹ sư đặt và duyệt theo từng mục tiêu thử nghiệm, không dùng một ngưỡng chung cho mọi trường hợp.

> **Thành công = kiểm thử có hệ thống, chỉ ra được điều kiện mô hình yếu, và đủ rõ ràng để kỹ sư ra quyết định cải thiện.**

## 9. Giới hạn

- Chỉ tập trung vào Perception, không xây dựng một xe tự hành hoàn chỉnh.
- Dữ liệu sinh không phải dữ liệu thật, và nhãn của nó không mặc định chính xác tuyệt đối.
- CARLA chưa mô phỏng đúng giọt nước bám ống kính, phản xạ phức tạp hay sương không đồng nhất.
- Diffusion có thể làm thay đổi vật thể trong ảnh, nên phải kiểm tra lại nhãn.
- FID/KID không chứng minh được tính chân thực của ảnh sinh.
- Agent chỉ đề xuất, không thay thế quyết định của kỹ sư.
- Cần kiểm soát số lượng kịch bản và độ phân giải ảnh để giới hạn chi phí GPU.
- Kết quả trên mô phỏng **không đồng nghĩa với an toàn ngoài đời thực**.

## 10. Hướng mở rộng: cải thiện mô hình bằng dữ liệu sinh

**Tìm điều kiện yếu → Sinh dữ liệu bổ sung → Huấn luyện lại (trộn với dữ liệu thật) → Kiểm tra trên tập thật độc lập.**

Các câu hỏi cần trả lời:
- Dữ liệu sinh có giúp mô hình tốt hơn trên ảnh thật không?
- Sinh có mục tiêu có hiệu quả hơn sinh đều không?
- Sinh bao nhiêu thì hiệu quả bão hòa?

*Đây là phần bổ sung, không bắt buộc với ADAS-23.*

## 11. Giá trị và thông điệp

- **Kỹ sư AI/CV:** biết mô hình yếu ở đâu, lỗi gì phổ biến, cần bổ sung dữ liệu nào.
- **Nhóm ADAS:** đánh giá khả năng nhận diện khi điều kiện bất lợi, xác định tình huống mô hình không còn đáng tin.
- **Nhóm kiểm thử và an toàn:** theo dõi độ phủ, quản lý ngưỡng, truy vết nguồn dữ liệu và lịch sử kiểm thử.

> *"Không chỉ kiểm tra AI nhận diện tốt đến đâu, WeatherForge giúp kỹ sư biết AI dễ mắc lỗi trong điều kiện nào, tình huống nào chưa được kiểm thử, và cần làm gì để đánh giá độ tin cậy của Perception trước khi triển khai thực tế."*
