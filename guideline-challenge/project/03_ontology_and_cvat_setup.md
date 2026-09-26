# Ontology + CVAT setup

Bảng ontology là **source of truth** cho schema CVAT: `03_cvat_labels.json` phải khớp từng dòng ở đây. Thay mọi
placeholder mới là xong (gate G2).

## Ontology table

| Name | Geometry | Type (class / attribute) | Allowed values | Default | Mutable? | Rationale |
|---|---|---|---|---|---|---|
| drivable_area | polygon | class | N/A | N/A | false | Class chính biểu diễn vùng mặt đường xe chạy được. |
| areaType | N/A | attribute | __undefined__, direct, alternative | __undefined__ | false | Phân biệt quyền ưu tiên của làn đường (làn xe ego đang chạy vs làn có thể chuyển sang). |
| needs_review | N/A | attribute | false, true | false | false | Đánh dấu vùng biên mơ hồ hoặc điều kiện bất lợi cần QA review lại. |
| weather | N/A | attribute | __undefined__, clear, partly cloudy, overcast, rainy, snowy, foggy, undefined | __undefined__ | false | Thuộc tính môi trường thời tiết gắn trên tag image_context. |
| timeofday | N/A | attribute | __undefined__, daytime, dawn/dusk, night, undefined | __undefined__ | false | Thuộc tính thời điểm trong ngày gắn trên tag image_context. |
| image_context | tag | class | N/A | N/A | false | Tag mức toàn ảnh ghi nhận ngữ cảnh môi trường (weather, timeofday). |
| image_escalate | tag | class | N/A | N/A | false | Tag mức toàn ảnh báo hiệu ảnh bị mất hoàn toàn thông tin quan sát (hầm tối, lóa đèn). |

## Class hay attribute

- **`drivable_area` là Class:** Đây là đối tượng hình học duy nhất cần phân đoạn (polygon 2D) phục vụ bài toán drivable area downstream.
- **`areaType` là Attribute:** Là thuộc tính phân loại mức độ ưu tiên (`direct` vs `alternative`) trên cùng một thực thể mặt đường. Dùng attribute giúp tránh nhân đôi số lượng class và giữ schema gọn gàng.
- **`needs_review` là Attribute (checkbox):** Cờ đánh dấu phục vụ quy trình QA nội bộ, không làm thay đổi ngữ nghĩa hình học của mặt đường.
- **`image_context` và `image_escalate` là Class dạng Tag:** Áp dụng cho toàn bộ khung hình, không gắn với bất kỳ polygon cụ thể nào.
- **Default gây bias:** Nếu để default của `areaType` là `direct`, annotator rất dễ quên đổi khi vẽ làn `alternative` $\rightarrow$ gây bias nặng về nhãn `direct`. Do đó default bắt buộc là `__undefined__` để buộc annotator phải chủ động lựa chọn.

## CVAT

- **Phiên bản CVAT** (`make cvat-status`): `CVAT 2.74.1`
- **Tên task calibration** (có version guideline, ví dụ `team07-calib-v1`): `team30-calib-v1`
- **Guide của task đã dán `02_guideline.md`?**: Có (đã dán toàn văn nội dung `02_guideline.md` vào tab Guide của task)
- **Nhóm dùng Track hay Shape, vì sao:** Dùng **Shape** vì tập dữ liệu là các ảnh tĩnh độc lập (2D single frame), không có mối liên kết tracking theo thời gian giữa các frame.

## Setup test

Một thành viên **chưa tham gia setup** mở task và trả lời: label gì, dùng tool nào, gán attribute nào, khi nào
escalate. Ghi lại ai test và chỗ họ vấp:

- **Người thực hiện test:** Nguyễn Đinh Nhật Trường (Guideline owner).
- **Kết quả trả lời:**
  - *Label & Tool:* Dùng tool Polygon vẽ class `drivable_area`.
  - *Attribute:* Chọn `areaType` là `direct` cho làn đang chạy, `alternative` cho làn bên cạnh; tick `needs_review = true` khi gặp vạch mờ/tuyết phủ.
  - *Escalate:* Khi hầm tối đen hoặc lóa đèn $>60\%$ thì không vẽ polygon mà gán tag `image_escalate`.
- **Chỗ vấp & Khắc phục:** Ban đầu annotator quên gán tag `image_context` cho toàn ảnh $\rightarrow$ Đã khắc phục bằng cách bổ sung bước gán context vào mục 10 (Self-QC checklist) của guideline trước khi lưu bài.
