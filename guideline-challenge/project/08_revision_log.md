# Revision log

Guideline v1 = bản nháp đầu; v2 = sau calibration nội bộ; v3 = sau blind handoff. Mỗi lần tăng `Version` trong
`02_guideline.md`, thêm một hoặc nhiều dòng vào bảng: đổi gì và vì sao, kèm bằng chứng (sample_id, dòng
calibration report, câu hỏi trong clarification log, feedback của peer).

Cột Version ghi dạng `v1`, `v2`, `v3` — `make status` tìm dòng bảng có `v2` và dòng có `v3`.

| Version | Đổi gì | Vì sao | Bằng chứng |
|---|---|---|---|
| v1 | Bản thảo đầu: xác định scope, taxonomy direct/alternative, quy tắc geometry cơ bản, Self-QC checklist ban đầu | Khởi tạo guideline cho bài toán Drivable Area Segmentation trên BDD100K | — |
| v2 | Bổ sung §7 đường ban đêm + §8 gore area + §9 bus-only lane; cập nhật Self-QC checklist 5 bước; làm rõ quy tắc polygon cho xe đỗ và crosswalk | Sau calibration nội bộ: BDD01/BDD03 (gore area), BDD18 (đêm tối), BDD24 (tuyết); annotator nội bộ sai 4/8 quyết định critical ở calibration round 1 | 06_calibration_report.csv; 06_calibration_measure.csv |
| v3 | Bổ sung §3 hard exclusion boundary cho vật cản cứng (con lươn, lề cỏ dốc) và double yellow rule (điểm đặt polygon tại mép trong vạch vàng); bảng quyết định CVAT thêm 2 dòng escalation ban đêm; checklist tăng lên 7 bước (bổ sung nhắc nhở gán tag image_context trước khi chuyển ảnh + kiểm tra pixel-level vật cản cứng + IGNORE ban đêm khi không rõ xe đỗ/đường ướt) | Blind handoff GTS = 59.9: C = 0/5 critical (geometry) do thiếu hình minh hoạ rõ ranh giới vật cản cứng và vạch vàng kép; peer bỏ sót tag weather=rainy do workflow không nhắc lại bước gán tag; 1 câu hỏi về kỹ thuật CVAT vạch vàng kép | transfer_score.csv; peer_feedback.md §1 Q2/Q3/Q4; clarification_log.csv row 1; BDD08 d3 / BDD14 d2 / BDD17 d3 / BDD20 d2 / BDD26 d2 d3 |
