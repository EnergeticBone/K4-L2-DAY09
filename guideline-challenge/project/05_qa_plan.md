# QA plan + quality gates

Không được viết "reviewer kiểm tra lại". Phải có sampling, metric, threshold và action khi fail. Thay mọi placeholder
mới là xong (gate G6).

## Flow

Guideline → Calibration → Production → Self-QC → Review → Rework → Quality Gate. Ghi cụ thể cho project của nhóm:

- **Ai review, review bao nhiêu:** 
  - QA Lead và Annotator thực hiện review chéo (Cross-review), tuyệt đối không tự review bài của chính mình.
  - Tỷ lệ lấy mẫu kiểm tra:
    - **100%** đối với các ảnh có nguy cơ cao (ảnh điều kiện thời tiết tuyết/mưa/đêm, ảnh có annotator gắn cờ `needs_review = true`, và ảnh thuộc đợt gán nhãn của annotator mới).
    - **25%** lấy mẫu ngẫu nhiên (Random sampling) đối với các batch ảnh ban ngày trong điều kiện bình thường (`clear`/`partly cloudy`).
- **Chọn sample theo rule nào:**
  - Áp dụng phương pháp lấy mẫu phân tầng theo rủi ro (Risk-stratified sampling):
    - *Tầng 1 (Bắt buộc 100%):* Mọi ảnh có ít nhất 01 polygon được tick `needs_review = true`.
    - *Tầng 2 (Bắt buộc 100%):* Ảnh thuộc các nhóm rủi ro thị giác cao theo tag `image_context`: `weather` $\in$ {`snowy`, `rainy`, `foggy`} hoặc `timeofday = night`.
    - *Tầng 3 (50%):* Ảnh của annotator có tỷ lệ lỗi ở batch liền trước $> 5$.
    - *Tầng 4 (25% ngẫu nhiên):* Các ảnh thông thường còn lại trên cao tốc hoặc phố ban ngày trời quang để giám sát chất lượng nền.
- **Issue được ghi ở đâu, đóng thế nào:**
  - *Ghi nhận lỗi:* Reviewer mở tính năng **Issue / Comment trực tiếp trên từng polygon trong CVAT**, gắn nhãn mức độ nghiêm trọng (`Critical`, `Major`, `Minor`, `Question`) và mô tả sai lệch cụ thể. Đồng thời cập nhật tóm tắt vào bảng theo dõi chất lượng nội bộ.
  - *Quy trình xử lý và đóng issue:* 
    1. Annotator nhận thông báo lỗi trên task CVAT và tiến hành chỉnh sửa (Rework).
    2. Sau khi sửa xong, annotator chuyển trạng thái Issue sang `Resolved`.
    3. Reviewer kiểm tra lại (Re-check). Nếu đạt chuẩn dung sai và thuộc tính, reviewer chuyển trạng thái sang `Closed`. Batch chỉ được tính là hoàn thành khi 100% Issues ở trạng thái `Closed`.
- **Khi phát hiện guideline gap thì update và version ra sao:**
  - Khi phát hiện bất đồng mà guideline hiện tại chưa có quy tắc phân định rõ ràng (Guideline Gap), annotator tick `needs_review = true` và ghi câu hỏi vào `clarification_log.csv`.
  - QA Lead triệu tập họp kỹ thuật nhanh (10–15 phút) cùng các annotator để thống nhất quy tắc giải quyết dựa trên downstream contract ở `01_problem_statement.md`.
  - Cập nhật quy tắc mới và ví dụ trực quan vào `02_guideline.md`, đồng thời tăng phiên bản guideline (`v1` $\rightarrow$ `v2` hoặc `v2` $\rightarrow$ `v3`).
  - Ghi nhận chi tiết vào `08_revision_log.md` (Phiên bản, ngày cập nhật, nội dung sửa đổi, lý do kỹ thuật).
  - QA Lead thông báo quy tắc mới cho toàn bộ đội ngũ và tiến hành rà soát hồi tố (Retroactive check) các sample tương tự đã gán nhãn trước đó.

## Defect severity

Nhóm được đổi mapping nếu downstream contract khác, nhưng phải giải thích và chốt trước khi QA.


| Severity | Định nghĩa cho project này                                                                                                                                                                                                                       | Ví dụ                                                                                                                                                                                               | Action mặc định                                                                                                                                                      |
| -------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Critical | **False Positive nguy hiểm:** Gán nhãn vùng cấm chạy thành `drivable_area`, hoặc gán nhầm luồng xe ngược chiều / dải phân cách / vùng cấm cao tốc. Hậu quả: Xe tự hành lập quỹ đạo va chạm nghiêm trọng.                                         | Vẽ lấn polygon lên vỉa hè (curb), vẽ trùm lên vạch xương cá (gore area), vẽ đè lên làn dừng khẩn cấp (shoulder), hoặc vẽ sang làn xe chạy ngược chiều.                                              | **REJECT toàn bộ batch**. Annotator phải làm lại 100% batch đó và được QA coaching lại quy tắc trước khi tiếp tục.                                                   |
| Major    | **False Negative hoặc sai thuộc tính phân loại:** Bỏ sót hoàn toàn diện tích mặt đường xe chạy được hợp pháp, hoặc gán sai quyền ưu tiên làn di chuyển. Hậu quả: Xe tự hành di chuyển rụt rè, phanh gấp bất thường hoặc nhầm lẫn khi chuyển làn. | Gán nhầm làn ego đang chạy từ `direct` thành `alternative`; cắt đứt polygon trước vạch kẻ người đi bộ (crosswalk); bỏ sót hoàn toàn một làn rẽ hợp pháp cùng chiều.                                 | **REWORK cục bộ**. Annotator phải sửa lại các sample có lỗi trong vòng 2 giờ; tăng tỷ lệ review batch tiếp theo lên 50%.                                             |
| Minor    | **Lệch dung sai hình học nhẹ hoặc thiếu metadata:** Sai lệch ranh giới nhưng không làm biến đổi bản chất hay ngữ nghĩa của vùng lưu thông.                                                                                                       | Biên polygon lệch $4 - 6\text{ px}$ so với mép vỉa hè/vạch kẻ (vượt ngưỡng $\le 3\text{ px}$); gối biên đè nhau nhẹ $> 2\text{ px}$; quên chọn `weather` hoặc `timeofday` trên tag `image_context`. | **QUICK FIX**. Annotator tự chỉnh sửa lại các điểm biên bị lệch trước khi merge dữ liệu; không cần reject cả batch.                                                  |
| Question | **Trường hợp mơ hồ cao (Data Ambiguity):** Dữ liệu bị che khuất nghiêm trọng, vạch sơn mờ hoặc điều kiện môi trường cực đoan chưa được quy định rõ trong guideline.                                                                              | Mặt đường bị bùn đất/tuyết phủ che khuất không rõ mép lề; ánh sáng đèn pha lóa mạnh nhưng vẫn lờ mờ thấy mặt đường.                                                                                 | Annotator giữ nguyên polygon ước lượng, bắt buộc tick `needs_review = true` và mở Issue chờ QA Lead quyết định; nếu mất hoàn toàn tầm nhìn thì gán `image_escalate`. |


## Metrics


| Metric                                  | Cách tính                                                                                                                     | Vì sao phù hợp với bài toán                                                                                                                                                                      |
| --------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **mIoU (Mean Intersection-over-Union)** | `mIoU = (1/C) * Σ [Area(Pred ∩ GT) / Area(Pred ∪ GT)]` với `C ∈ {direct, alternative}` (tính IoU từng lớp rồi lấy trung bình) | Thước đo chuẩn mực cho bài toán Semantic / Region Segmentation; đánh giá chính xác mức độ trùng khớp diện tích hình học giữa polygon gán nhãn và vùng mặt đường chuẩn.                           |
| **Attribute Accuracy (`areaType`)**     | `Acc = (Số polygon gán đúng areaType / Tổng số polygon drivable_area) * 100%`                                                 | Đo lường độ chuẩn xác trong việc phân định quyền ưu tiên di chuyển (làn đang chạy `direct` vs làn có thể chuyển `alternative`), quyết định trực tiếp tới an toàn của thuật toán Motion Planning. |
| **Boundary F-score (BF-score)**         | `BF-score` là điểm F1-score tính trên tập điểm biên hình học thỏa mãn sai số khoảng cách `≤ 3 px` so với biên ground truth    | Đánh giá độ sắc nét và chính xác tại các ranh giới nhạy cảm (mép gờ bó vỉa, vạch kẻ phân làn), nơi thuật toán điều khiển bánh xe cần độ chính xác cao nhất.                                      |


Metric high-risk tách riêng (ví dụ critical defect escape rate): 

- **Critical Defect Escape Rate (CDER):** 
  - *Công thức:* `CDER = (Số lỗi Critical lọt qua vòng Self-QC / Tổng số sample được QA review) * 100%`
  - *Chỉ tiêu bắt buộc:* `CDER = 0%`. Tuyệt đối không chấp nhận bất kỳ polygon nào lấn lên vỉa hè hoặc làn ngược chiều lọt vào tập dữ liệu bàn giao (Zero-tolerance for safety-critical defects).

## Quality gate

Threshold là đề xuất của nhóm, không phải chuẩn ngành. Giải thích trade-off cost/risk.

```text
PASS if:
  - Critical Defects = 0 (100% không có lỗi False Positive vào vỉa hè, gore area, làn ngược chiều).
  - mIoU >= 90% tính trên toàn bộ tập sample được QA review.
  - Attribute Accuracy (areaType) >= 95%.
  - 100% polygon đã được gán nhãn (0% polygon giữ giá trị mặc định areaType = __undefined__).
  - 100% Issues và Comments trên CVAT đã chuyển sang trạng thái Closed.

REWORK if:
  - Xuất hiện từ 1 - 2 lỗi Major (sai phân loại areaType, cắt đứt polygon trước crosswalk).
  - Hoặc mIoU nằm trong khoảng 80% - 89% (sai lệch biên hình học nhưng chưa sai bản chất vùng).
  - Hoặc Minor Defect Rate > 5% (nhiều lỗi lệch điểm biên nhỏ hoặc thiếu tag image_context).
  - Thời hạn hoàn thành rework: tối đa 2 giờ kể từ khi nhận phản hồi.

REJECT / ESCALATE if:
  - Xuất hiện >= 1 lỗi Critical (vẽ lên vỉa hè, gore area, làn ngược chiều, hoặc shoulder).
  - Hoặc mIoU < 80% hoặc Attribute Accuracy < 85% (cho thấy annotator chưa nắm vững guideline).
  - Hoặc phát hiện mâu thuẫn dữ liệu nghiêm trọng / guideline gap không thể giải quyết nội bộ -> ESCALATE lên QA Lead để họp cập nhật guideline mới.
```

Trade-off: 

- **Độ an toàn (Risk) vs. Chi phí thẩm định (Cost):** Chúng tôi đặt ngưỡng **Critical = 0** và quy trình Reject toàn bộ batch khi xuất hiện lỗi Critical. Quyết định này chấp nhận đánh đổi tăng thời gian review và chi phí đào tạo lại nhân sự (Cost) để triệt tiêu hoàn toàn rủi ro xe tự hành lập quỹ đạo va chạm vào vỉa hè hoặc dải phân cách (Safety-first).
- **Độ chi tiết biên ở tầm xa vs. Tốc độ gán nhãn (Speed):** Tại khu vực cự ly gần (0–20m trước mũi xe), chúng tôi áp dụng kiểm soát nghiêm ngặt dung sai $\le 3\text{ px}$. Tuy nhiên, ở tầm nhìn xa ($>40\text{m}$) hoặc trong điều kiện sương mù/đêm tối nơi độ phân giải camera bị suy giảm mạnh, dung sai được nới lỏng lên $\le 5\text{ px}$. Điều này giúp annotator không mất thời gian nắn chỉnh từng pixel ở vùng dữ liệu nhiễu cao, duy trì tốc độ sản xuất dữ liệu ổn định mà vẫn đảm bảo tính an toàn điều khiển tức thời cho xe.

