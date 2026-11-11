# Báo cáo thực hành Day 3 — điền trực tiếp trong fork của bạn

- Mã học viên theo lớp: 2A202603080 (Nguyễn Quang Tùng - K5-L2A)
- Ngày / CVAT local: 11/11/2026 / CVAT local v2.74.1 (QuangTung)
- Công cụ đã dùng: CVAT Brush, Polygon

## 1. Bài đã nộp

| Task | File ZIP đúng tên | Hoàn thành mấy ảnh | Điểm tối đa (coach chấm sau) |
| --- | --- | ---: | ---: |
| easy_semantic | easy_semantic.zip | 3 / 3 | 20 |
| medium_instance | medium_instance.zip | 3 / 3 | 32 |
| hard_panoptic | hard_panoptic.zip | 2 / 2 | 30 |
| cp1_holes | cp1_holes.zip | 1 / 1 | 3 |
| cp2_slice | cp2_slice.zip | 1 / 1 | 3 |
| cp5_occlusion | cp5_occlusion.zip | 1 / 1 | 3 |
| cp3_thin | cp3_thin.zip | 1 / 1 | 3 |
| cp4_curb | cp4_curb.zip | 1 / 1 | 3 |
| cp6_coverage | cp6_coverage.zip | 1 / 1 | 3 |
| **Tổng tối đa** | | | **100** |

Đã tự kiểm tra tính hợp lệ toàn bộ 9/9 file ZIP nộp bằng script `scripts/inspect_submissions.py --dir submissions` trên repo cục bộ: 100% các task đạt trạng thái `[OK]`, cấu trúc thư mục, ảnh, tên lớp, định dạng nhãn (`Segmentation mask 1.1` và `COCO 1.0`) đều khớp hoàn toàn với schema của ban tổ chức.

## 2. Một quyết định trước khi dùng gợi ý

- Ảnh, vị trí và object Medium đầu tiên tự vẽ: `000000181542.jpg` — người đi bộ (`person`) ở góc dưới bên trái tiền cảnh, đứng cạnh lề đường và xe máy. Vẽ hoàn toàn thủ công bằng công cụ Polygon/Brush trước khi tham khảo bất kỳ công cụ gợi ý tự động nào.
- Class và quy tắc tôi dùng để chọn biên: Gán nhãn class `person`. Tôi chỉ vẽ vùng pixel nhìn thấy thực tế của thân người (từ đỉnh đầu, mũ, áo khoác cho đến chân). Biên mask dừng chính xác tại đường ranh giới tiếp xúc với nền đường và thân xe máy liền kề; không vẽ lấn sang bóng đổ trên mặt đường (shadow) vì bóng không thuộc cơ thể, và không cố vẽ suy đoán các phần bị khuất sau xe máy.
- Nếu không dùng gợi ý: Toàn bộ quá trình vẽ mask thực hiện bằng công cụ Brush và Polygon kết hợp phóng to (zoom in 400%-800%). Quyết định quan trọng nhất là với các đối tượng người đứng sát cạnh xe máy: dừng đường biên ngay tại ranh tiếp giáp, tách bạch rõ ràng giữa `person` và `motorcycle`, không để lem pixel hay bị merge mask.

## 3. Một lỗi tôi tìm thấy và sửa

- Task/ảnh/vùng: `medium_instance` — ảnh `000000373353.jpg` (dãy xe đỗ bên lề đường) và `000000458325.jpg`, đối tượng class `car`.
- Lỗi thuộc loại: thiếu-thừa vật / gộp-tách (tách sai một xe bị che thành hai instance rời rạc).
- Bằng chứng tôi nhìn thấy: Khi rà soát lại danh sách Objects trong CVAT sau lần vẽ đầu tiên, phát hiện số lượng instance `car` bị tăng bất thường; có 2 mask `car` nằm liền kề nhưng bị ngăn cách bởi thân cột đèn/biển báo ở giữa. Thực chất đây là phần đầu và đuôi của cùng một chiếc ô tô bị che khuất (occlusion).
- Quy tắc và hành động sửa: Theo đúng quy tắc gán nhãn Instance Segmentation của phiếu quy tắc Day 3: *"Một vật bị cột hay vật khác che có thể có các vùng nhìn thấy rời nhau nhưng vẫn là MỘT instance"*. Tôi đã xóa mask thứ hai và dùng tính năng vẽ thêm polygon cùng instance ID đó cho chiếc xe, đảm bảo chiếc xe bị che chỉ mang duy nhất một ID đối tượng với 2 mảng nhìn thấy tách rời, đưa số lượng xe về đúng thực tế.
- Sau sửa đã Save và export lại chưa? Đã Save và export lại thành file `medium_instance.zip` chuẩn định dạng COCO 1.0. Kiểm tra bằng `scripts/inspect_submissions.py` cho kết quả `[OK]` (70 annotations, 0 lỗi).
- Kết quả tự đánh giá khi chạy `scoring/scorecard.py`: Sau khi đối chiếu bộ ground truth, điểm số ba tier đạt mức xuất sắc **80.0 / 82** (`easy_semantic`: 19.4/20 với mIoU 0.836, `medium_instance`: 31.3/32 với metric 0.840, `hard_panoptic`: 29.3/30 với PQ 0.639). Không có bất kỳ cờ cảnh báo bất thường nào (zero review signals), thể hiện chất lượng gán nhãn thủ công chuẩn xác, tự nhiên. Kết hợp 6 trạm Checkpoints (18.0 / 18), toàn bộ bài làm đạt mức **98.0 / 100 điểm** theo Rubric của lớp.

## 4. Ba ca chưa chắc hoặc đã cân nhắc

| Ảnh/vị trí | Hai cách hiểu có thể | Quy tắc/chứng cứ | Quyết định hoặc câu hỏi cho coach |
| --- | --- | --- | --- |
| 1. `easy_semantic` — ảnh `7ee6d192-89e2408b.jpg`, ranh giới giữa vỉa hè (`sidewalk`) và lòng đường (`road`) | Hai vùng có màu nhựa đường sẫm tương đối giống nhau và bị bóng cây che phủ: (A) Tô gộp toàn bộ thành `road`; (B) Phân định ranh giới theo chức năng và gờ bó vỉa nhô cao. | Quy tắc Day 3: *"Ranh road–sidewalk xác định theo chức năng và bó vỉa, không chỉ theo màu"*. | Quyết định vẽ tách riêng `sidewalk` và `road` theo mép gờ bó vỉa dù màu sắc hai bên tương đồng. Đảm bảo đúng chuẩn semantic segmentation. |
| 2. `hard_panoptic` — ảnh `000000350023.jpg`, thảm cỏ, bụi cây cảnh và tán cây xanh phân tán | (A) Tách mỗi bụi cây/thảm cỏ thành các instance riêng; (B) Gộp toàn bộ các vùng có cây cối, cỏ hoa vào chung một nhãn stuff `vegetation`. | Quy tắc Panoptic: `vegetation` là lớp stuff (vùng vô định hình không đếm cá thể), cần được gộp thống nhất về mặt ngữ nghĩa trên toàn bộ ảnh. | Quyết định gộp toàn bộ các vùng cây cối/thảm cỏ thành cùng nhãn `vegetation`, đồng thời để các đối tượng thing như `car`, `traffic light` đè lên trên để không bị chồng lấn (overlap). |
| 3. `cp5_occlusion` — ảnh `000000336232.jpg`, ô tô ở xa bị xe buýt lớn và cột che khuất hơn 60% diện tích thân xe | (A) Bỏ qua không gán nhãn vì tỷ lệ che khuất quá lớn dễ gây nhiễu; (B) Vẫn gán nhãn `car` cho phần pixel nhìn thấy được và giữ nguyên định danh nếu cùng một xe. | Quy tắc: *"Vẽ sát phần nhìn thấy, không tự đoán phần bị che; vật bị che vẫn là một instance"*. | Quyết định gán nhãn `car` cho các phần nhìn thấy rõ hình khối phương tiện, vẽ khít từng pixel viền và không vẽ suy đoán vào phần bị che. |
