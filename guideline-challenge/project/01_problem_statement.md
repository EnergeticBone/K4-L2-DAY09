# Problem statement + downstream contract

## Bài toán

Segment **drivable area** (polygon + `areaType`) trên ảnh dashcam, với trọng tâm là các ranh giới mơ hồ:
vỉa hè/bãi đỗ/lề đường trong phố dân cư, gore area & shoulder trên cao tốc, mặt đường bị tuyết phủ, và đường
đêm thiếu vạch kẻ — annotator hợp lý dễ vẽ khác nhau chính ở những ranh giới này.

## Downstream contract

1. **Downstream task / model / user là ai?** Model segment vùng xe được phép di chuyển (drivable-area
   segmentation đầu vào ego-path planning, xe bán tự hành L2); annotator cung cấp ground truth huấn luyện.
2. **Output annotation nào thực sự cần?** Polygon class `drivable_area` phủ mặt đường chạy được, kèm attribute
   `areaType` ∈ {`direct`, `alternative`}; không cần instance, không cần class con.
3. **Failure nào gây hậu quả lớn nhất?** **False positive**: gán vùng không được phép chạy (vỉa hè, bãi đỗ vạch
   sẵn, lề tuyết, gore area cấm) là drivable → model học "lên vỉa hè được" → va chạm. Đây là decision `critical`
   trong gold. False negative chỉ khiến model rụt rè → severity `major`.
4. **Khi ambiguity không resolve được, ai / ở đâu là escalation path?** Tra guideline mục 7 trước; vẫn không đủ
   bằng chứng → vẽ polygon + `areaType=unknown` (hết cách khác trong ảnh tĩnh) hoặc ESCALATE theo điều kiện mục 7,
   mọi chỗ phải hỏi ghi vào `clarification_log.csv`. Không hỏi miệng trong blind window.

## Scope

- **Trong scope (bắt buộc label):** mặt đường mà xe hợp pháp chạy được — làn, intersection, đường phố dân cư
  kể cả phần dùng làm bãi đỗ dọc khi đó là mặt đường chung, shoulder/gore area **chỉ khi** rule guideline cho phép.
- **Ngoài scope (ignore):** vỉa hè, bãi đỗ riêng có vạch, lề cỏ/tuyết không phải mặt đường, xe cộ, người, cột
  đèn; vật thể che mặt đường → ranh giới đi theo phần nhìn thấy được, không suy đoán phía sau.
- **Geometry tolerance:** polygon sát biên mặt đường thấy được, sai lệch ≤ 5 px mỗi đỉnh so với biên thật; ≥ 4
  đỉnh; không cắt qua xe đang che (không extrapolate).

## Output chấm được

- **LABEL:** polygon `drivable_area` + `areaType=direct|alternative` — nhìn thấy qua attribute value trong export.
- **IGNORE:** không vẽ polygon tại vùng ngoài scope — chấm qua *vắng mặt* (gold ghi "không polygon tại vùng X",
  `peer_evidence` mô tả peer vẽ gì).
- **UNKNOWN:** polygon + `areaType=unknown` (attribute khai báo trong CVAT, xuất ra file).
- **ESCALATE:** không có attribute riêng → thể hiện bằng dòng `expected=ESCALATE` trong `gold_decisions.csv` +
  câu hỏi trong `clarification_log.csv`; điều kiện escalate ghi trong guideline mục 7.
- **Geometry:** decision `geometry:` chấm bằng cách mở polygon của peer so tolerance ở trên.

## Dữ liệu và giới hạn

- **Nguồn:** `data/drivable/` — 30 ảnh BDD100K đã chọn tay (DRV01–DRV30), metadata lấy từ nhãn chính thức
  (`catalog.csv`: weather/timeofday/scene), xem `data/ATTRIBUTION.txt`.
- **Phủ cảnh:** 15 city street, 10 highway, 2 trạm xăng, 2 bãi đỗ xe, 1 residential; 18 ngày / 10 đêm / 2
  chạng vạng; 4 mưa, 3 tuyết, 2 sương mù — đủ case `low_visibility` + `edge`/`critical`: vỉa hè–bãi đỗ
  (DRV21–24), biên mờ do sương/tuyết (DRV25–28), reflect nước mưa (DRV30), đêm thiếu vạch (DRV03/07/08).
- **Giới hạn đã biết:** ảnh tĩnh → mục 8 guideline ghi "Không áp dụng"; ảnh hiếm (tuyết/sương) chỉ 5 cái nên
  chỉ dùng cho một trong hai split (calibration hoặc blind, không chia đôi); `data/` không có nhãn drivable
  đi kèm — gold do nhóm tự chốt trước freeze.
