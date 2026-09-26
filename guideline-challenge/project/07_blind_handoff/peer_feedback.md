# Peer feedback + owner response

Phần 1 do **nhóm peer** trả lời (gửi kèm file export). Phần 2 do **nhóm owner** điền. Thay mọi placeholder mới
là xong (gate G5).

- **Nhóm peer:** Nhóm 999999 (3 thành viên)
- **Người label blind:** Đỗ Trung Kiên

## 1. Peer trả lời

1. **Rule nào rõ nhất / giúp quyết định nhanh nhất?**
   Quy tắc phân biệt `direct` vs `alternative` theo vạch đứt/liền rất rõ — nhìn vạch là biết phân loại ngay.
   Quy tắc "không vẽ lên vỉa hè và gờ bó vỉa" cũng dễ áp dụng vì thường thấy rõ ràng.

2. **Rule nào mơ hồ hoặc phải tự suy diễn?**
   - Vạch vàng kép: guideline nói "bám sát" nhưng không có ảnh cắt ngang cho thấy điểm đặt polygon là mép trong hay mép ngoài của vạch.
   - Đêm tối (BDD26): không biết "quầng rọi sáng của đèn pha" dừng ở đâu chính xác — không có ví dụ ảnh ban đêm trong guideline.
   - Con lươn bê tông và đảo giao thông ban đêm: khó nhận biết ranh giới trong ảnh tối, không có ảnh mẫu.

3. **Sample nào khiến guideline "vỡ"?**
   BDD20 — vạch vàng kép giữa đường phố dân cư + xe SUV đỗ bên phải trong điều kiện mặt đường ướt phản quang. Không chắc polygon có nên bao gồm vùng trước SUV hay không khi đó là xe đỗ chứ không phải xe đang chạy.

4. **Attribute / default nào trong CVAT dễ gây thao tác sai?**
   Attribute `weather` của tag `image_context` — mặc định không hiện trong CVAT nếu không mở đúng panel Attributes. Nhóm bỏ sót hoàn toàn bước gán tag này vì workflow step-by-step trong guideline không nhắc lại sau phần polygon.

5. **Một thay đổi cụ thể giúp annotator mới ít hỏi hơn?**
   Thêm ảnh cắt ngang mặt đường ban đêm và ảnh cắt ngang vạch vàng kép với mũi tên chỉ rõ "đặt điểm polygon ở đây" — một hình bằng nghìn chữ.

## 2. Owner phân loại

Owner không tranh luận để bảo vệ guideline. Mỗi feedback và mỗi decision peer làm sai được xếp vào một hướng xử lý.

| Feedback / decision sai | Nguyên nhân (guideline gap / data ambiguity / execution error) | Xử lý (accept + revise / reject with evidence / add escalation rule) | Bằng chứng |
|---|---|---|---|
| BDD08 d3: mép trái lấn vào con lươn bê tông | guideline_gap — không có ảnh minh hoạ tường minh ranh giới dừng tại vật cản cứng | accept + revise — thêm ảnh cắt ngang con lươn BDD11 với annotation mẫu vào §4.2 guideline v3 | BDD08 d3 correct=0; calibration_report row C1 |
| BDD14 d2: vẽ lấn sang lề cỏ dốc | guideline_gap — "lề cỏ = hard exclusion" được ghi text nhưng không có visual example trên cao tốc | accept + revise — thêm hình minh hoạ lề cỏ dốc vào §3 Exclusion và thêm ví dụ BDD14 vào §5 | BDD14 d2 correct=0; peer feedback Q3 |
| BDD17 d3: bỏ sót tag weather=rainy | execution_error — workflow CVAT §6 không nhắc bước gán image_context tag sau khi vẽ polygon xong | accept + revise — thêm checklist cuối task vào §6.3: "Kiểm tra đã gán đủ tag image_context chưa?" | BDD17 d3 correct=0; peer feedback Q4 |
| BDD20 d2: trùm polygon qua vạch vàng kép | guideline_gap — thiếu ảnh cắt ngang mặt cắt vạch vàng kép nhìn thẳng chỉ rõ mép trong làm ranh giới polygon | accept + revise — thêm hình minh hoạ vạch vàng kép nhìn từ trên xuống với mũi tên "dừng tại đây" vào §4.1 | BDD20 d2 correct=0; clarification_log row 1 |
| BDD20 d3: đè lên SUV đỗ bên phải | data_ambiguity — đêm mưa phản quang khiến xe SUV đỗ khó phân biệt với vùng đường ướt trong ảnh static | add escalation rule — thêm vào §7 Escalation: "Không phân biệt được xe đỗ hay đường ướt → mark IGNORE toàn vùng nghi ngờ, hỏi người phụ trách" | BDD20 d3 correct=0; peer feedback Q3 |
| BDD26 d2: vẽ vào bóng tối | guideline_gap — không có ví dụ ảnh ban đêm trong bộ example; peer không có điểm tham chiếu trực quan | accept + revise — thêm ảnh BDD26 thumbnail (không dùng để gán nhãn) làm visual reference cho "quầng sáng đèn pha" vào §4.3 | BDD26 d2 correct=0; peer feedback Q2 |
| BDD26 d3: mép trái chạm đảo giao thông ban đêm | data_ambiguity — đảo giao thông ban đêm không nhìn rõ ranh giới, dễ nhầm với mặt đường tối | add escalation rule — cùng §7: "Không nhìn rõ đảo giao thông → IGNORE vùng nghi ngờ + ghi note" | BDD26 d3 correct=0; peer feedback Q3 |
