# Annotation guideline — Vùng xe chạy được (Drivable Area) trên ảnh BDD100K

**Version:** v2

<!--
File này là thứ nhóm peer nhận nguyên văn trong blind pack và là Guide dán vào CVAT. Peer KHÔNG nhận
edge_case_cards.md, gold_decisions.csv hay sample_pack.csv. Rule nào peer cần biết phải nằm ở đây.
No hidden rules: rule chỉ giải thích bằng miệng thì coi như không tồn tại.
Ví dụ trong guideline chỉ dùng ảnh split example hoặc calibration, không dùng ảnh blind.
-->

## 1. Objective + scope

- **Mục tiêu downstream:** Cung cấp dữ liệu phân vùng mặt đường chuẩn xác cho hệ thống nhận thức (Perception) và lập kế hoạch quỹ đạo (Motion Planning / Trajectory Generation) của xe tự hành (ego-vehicle). Hệ thống cần phân biệt tức thời giữa:
  1. *Làn đường có quyền ưu tiên di chuyển tức thời (`direct`)* để duy trì hướng chạy.
  2. *Làn đường có thể chuyển sang an toàn và hợp pháp (`alternative`)* khi cần vượt xe hoặc chuyển hướng.
  3. *Các khu vực cấm hoặc không thể lưu thông* (làn ngược chiều, làn xe bus/xe đạp, vỉa hè, vật cản) để đảm bảo không bao giờ lập quỹ đạo va chạm hoặc phạm luật.

- **Trong scope (bắt buộc gán nhãn):**
  - Mọi diện tích mặt đường xe chạy (nhựa đường, bê tông, đá lát) thuộc chiều lưu thông hợp pháp của xe chủ mà xe có thể đi vào trực tiếp hoặc có thể chuyển làn sang hợp pháp.
  - Phân tách thành 2 mức độ ưu tiên theo chuẩn BDD100K:
    1. `direct`: Làn đường hiện tại mà ego-vehicle đang chiếm giữ và có quyền ưu tiên di chuyển thẳng/theo luồng (không cần thực hiện thao tác chuyển làn).
    2. `alternative`: Các làn đường cùng chiều khác mà ego-vehicle có thể chuyển làn sang một cách hợp pháp và an toàn (qua vạch đứt, làn rẽ cùng chiều, khu vực mở rộng làn tại giao lộ).
  - Vạch đi bộ qua đường (crosswalk) cắt ngang lòng đường xe chạy: VẪN thuộc scope (được gán nhãn bao trùm vào `direct` hoặc `alternative` tùy thuộc vào làn đường nó cắt qua).
  - Giao lộ (intersections): Toàn bộ không gian ngã ba / ngã tư mà luồng giao thông cùng chiều của ego được phép đi thẳng hoặc rẽ vào.

- **Ngoài scope (Exclusion / Tuyệt đối không vẽ polygon):**
  - Vỉa hè (sidewalk), gờ bó vỉa (curb), rãnh thoát nước biên, lề đường đất / đá sỏi không rải nhựa (unpaved shoulder).
  - Dải phân cách cứng (median barriers), con lươn bê tông, đảo giao thông tam giác / đảo phân luồng (traffic islands), bùng binh trung tâm.
  - Làn đường ngược chiều (opposing traffic lanes) ngăn cách bởi vạch liền màu vàng, dải phân cách hoặc luồng xe chạy ngược lại.
  - Làn đường chuyên dụng chỉ dành riêng cho xe buýt (bus-only lane có chữ "BUS", "BUS ONLY" hoặc vạch sơn/biển báo cấm xe con).
  - Làn đường dành riêng cho xe đạp (bike lane / cycle track có sơn biểu tượng xe đạp hoặc cọc phân làn riêng).
  - Vùng kẻ vạch xương cá / vạch chữ V phân tách làn (gore / chevron markings tại các nhánh nhập/tách làn cao tốc).
  - Khu vực đỗ xe riêng biệt ngoài đường (bãi đỗ có kẻ ô vuông góc ngoài lòng đường chính), lối ra vào nhà dân (driveways).
  - Làn đường hoặc khu vực bị phong tỏa kiên cố bởi rào chắn công trường (construction barriers) hoặc hàng rào ngăn cấm hoàn toàn.

## 2. Annotation unit

- **Loại unit:** Image-level static annotation (từng ảnh tĩnh 2D trích từ tập BDD100K, độ phân giải 1280x720).
- **Hình thức gán nhãn:** Region-based segmentation bằng đối tượng hình học dạng đa giác (**Polygon**).
- **Đơn vị instance / region:**
  - Mỗi vùng mặt đường liên tục về mặt hình học và cùng loại thuộc tính `areaType` được biểu diễn thành **01 Polygon riêng biệt**.
  - **Quy tắc tách rời polygon (Disjoint regions):** Tuyệt đối KHÔNG vẽ 01 polygon bắc cầu (bridge) qua vùng không được phép chạy (như bắc qua dải phân cách, băng ngang qua làn `direct`, hoặc nhảy qua đảo giao thông). Nếu có 2 làn `alternative` nằm ở hai bên của làn `direct`, bắt buộc phải tạo **2 Polygon `alternative` riêng biệt**.
  - **Xe cộ trên mặt đường:** Drivable area thể hiện diện tích mặt đường chức năng. Đối với các xe đang lưu thông bình thường hoặc xe đỗ tạm thời trên làn xe chạy, polygon drivable area vẫn bao quát toàn bộ vùng làn xe (không khoét lỗ vụn quanh từng bánh xe hay gầm xe), TRỪ KHI có vật cản lớn cố định chắn toàn bộ lòng đường khiến làn đó trở thành không thể đi qua (impassable).

## 3. Geometry rule

- **Định dạng hình học:** Đa giác khép kín (**Polygon**), biểu diễn chính xác vùng diện tích bề mặt đường 2D.
- **Tiêu chuẩn biên (Visible boundary):**
  - Vẽ bám sát ranh giới nhìn thấy thực tế của mặt đường, không vẽ suy đoán theo hình khối ẩn (không vẽ amodal xuyên qua vật cản lớn).
  - **Mép ngoài (Outer boundary):** Bám sát mép tiếp giáp giữa mặt đường nhựa/bê tông với chân gờ bó vỉa (curb) hoặc vạch sơn biên mép đường (solid edge line).
  - **Mép phân làn (Lane boundary):** Đường ranh giới phân tách giữa làn `direct` và làn `alternative` chạy dọc theo tâm của vạch kẻ phân làn (lane marking).
- **Quy tắc đặt điểm:**
  - Đoạn thẳng: Đặt điểm thưa tại các điểm chuyển hướng chính, tránh cắm điểm dày đặc gây gồ ghề biên.
  - Đoạn cong (đoạn đường cua, bo góc giao lộ): Đặt điểm dày hơn (khoảng cách 15–30 px) để đường biên cong mượt mà theo thực tế.
- **Tolerance (Dung sai cho phép):**
  - Sai số lệch biên tối đa **≤ 3 px** so với vạch kẻ phân làn hoặc mép vỉa hè rõ ràng.
  - Không để hở khe trống (gap) giữa các polygon tiếp giáp và không được đè chồng lấn lớn lên nhau (cho phép gối nhẹ biên ≤ 2 px).

## 4. Taxonomy

Bảng ontology chi tiết đồng bộ hoàn toàn với `03_ontology_and_cvat_setup.md` và `03_cvat_labels.json`:

| Name | Geometry | Type | Allowed values | Default | Mutable? | Rationale |
|---|---|---|---|---|---|---|
| `drivable_area` | polygon | class | N/A | N/A | false | Class chính biểu diễn vùng mặt đường xe chạy được. |
| `areaType` | N/A | attribute | `__undefined__`, `direct`, `alternative` | `__undefined__` | false | Phân biệt quyền ưu tiên của làn đường. Default `__undefined__` để bắt buộc annotator chủ động chọn, tránh thiên lệch (bias). |
| `needs_review` | N/A | attribute | `false` | `false` | false | Checkbox đánh dấu trường hợp có nghi vấn cần QA review lại. Khi tick, giá trị chuyển thành `true`. |
| `image_context` | tag | class | N/A | N/A | false | Tag mức toàn ảnh để ghi nhận điều kiện môi trường quan sát. |
| `weather` | N/A | attribute | `__undefined__`, `clear`, `partly cloudy`, `overcast`, `rainy`, `snowy`, `foggy`, `undefined` | `__undefined__` | false | Thời tiết của cảnh chụp ảnh. |
| `timeofday` | N/A | attribute | `__undefined__`, `daytime`, `dawn/dusk`, `night`, `undefined` | `__undefined__` | false | Thời điểm trong ngày (ban ngày, chạng vạng, ban đêm). |
| `image_escalate` | tag | class | N/A | N/A | false | Tag mức toàn ảnh dùng khi ảnh bị mất hoàn toàn thông tin quan sát. |

- **Phân định Class và Attribute:**
  - **Class `drivable_area` (Polygon):** Đối tượng hình học chính thể hiện vùng không gian mặt đường xe có thể lưu thông.
    - `areaType = direct`: Gán cho làn đường hiện tại mà xe ego đang chiếm giữ và có quyền ưu tiên di chuyển trực tiếp (hướng chuyển động tức thời, không cần chuyển làn).
    - `areaType = alternative`: Gán cho các làn đường cùng chiều hợp lệ liền kề mà xe ego có thể chuyển làn sang an toàn và đúng luật (qua vạch đứt, làn rẽ cùng chiều).
    - `areaType = __undefined__`: Giá trị khởi tạo mặc định. Mọi polygon bắt buộc phải được chuyển sang `direct` hoặc `alternative`.
    - `needs_review = true`: Kích hoạt khi ranh giới hoặc ngữ cảnh có sự mơ hồ (vạch kẻ mờ, tuyết che, xe đang đè vạch phân làn).
  - **Class `image_context` (Tag):** Nhãn mức toàn ảnh để ghi nhận thông tin môi trường (`weather`, `timeofday`), phục vụ phân tầng dữ liệu và đánh giá hiệu năng mô hình theo điều kiện thực tế.
  - **Class `image_escalate` (Tag):** Nhãn mức toàn ảnh chỉ áp dụng khi điều kiện quan sát bị hỏng hoàn toàn (hầm tối đen mất đèn, lóa đèn pha > 60% diện tích mặt đường) khiến không thể xác định được vùng xe chạy.

## 5. Inclusion / exclusion

- **Quy tắc bắt buộc vẽ (Inclusion):**
  1. Mặt đường làn `direct` từ sát mũi xe (mép dưới ảnh) kéo dài ra xa đến giới hạn tầm nhìn.
  2. Các làn `alternative` hợp pháp cùng chiều (làn kế cận ngăn bởi vạch đứt trắng).
  3. Mặt đường tại giao lộ / ngã tư: Toàn bộ vùng ngã tư mà xe ego có thể đi thẳng hoặc rẽ hợp pháp.
  4. Vạch đi bộ qua đường (crosswalk) nằm trên lòng đường: Vẽ trùm qua crosswalk nếu làn đường tiếp diễn qua đó.
  5. Làn rẽ (turn-only lanes): Nếu cùng chiều và xe ego có thể chuyển sang được thì gán là `alternative`.
- **Quy tắc bỏ qua / Không vẽ (Exclusion):**
  1. Làn ngược chiều (opposing lanes): Bất kể mặt đường có cùng chất liệu, nếu là làn của chiều xe đối diện (phân cách bởi vạch vàng liền, đảo giao thông hoặc hàng xe chạy ngược lại), BỎ QUA không vẽ.
  2. Làn chuyên dụng chỉ dành cho xe bus hoặc xe đạp (Bus-only / Bike lanes): Các làn có vạch sơn liền, biển báo cấm xe con, chữ "BUS", "BUS ONLY" hoặc sơn biểu tượng xe đạp mà xe ego không được phép chuyển vào theo luật: BỎ QUA không vẽ.
  3. Vùng vạch xương cá (gore / chevron markings): Khu vực phân tách nhánh đường cao tốc: BỎ QUA không vẽ.
  4. Vỉa hè, thảm cỏ, dải đất phân cách, gờ bê tông: Tuyệt đối không vẽ lấn lên.
  5. Làn đường bị chặn hoàn toàn: Khu vực thi công có rào chắn cứng ngang qua toàn bộ mặt đường.
  6. Bãi đỗ xe bên ngoài đường lộ: Khu vực đậu xe có kẻ ô vuông góc ngoài phạm vi lưu thông chính.

## 6. Visibility / occlusion

- **Tầm nhìn xa (Far vanishing boundary):**
  - Dừng vẽ polygon tại vị trí xa nhất mà mắt thường còn phân biệt được ranh giới mặt đường hoặc vạch kẻ (thường không vượt quá điểm tụ / chân trời, khoảng 50–80m).
  - Không vẽ suy đoán kéo dài polygon vào vùng sương mù đặc hoặc bóng tối mù mịt không thấy mặt đường.
- **Bị che khuất bởi xe cộ (Vehicle occlusion):**
  - **Xe lưu thông bình thường:** Drivable area thể hiện diện tích mặt đường có thể đi được; không khoét rỗng theo từng gầm xe hoặc bánh xe của các phương tiện đang lưu thông phía trước. Vẽ polygon ôm theo biên làn đường tự nhiên.
  - **Xe tải / bus lớn chiếm trọn tầm nhìn:** Nếu xe tải lớn chắn ngay trước mũi xe và che khuất hoàn toàn mặt đường phía sau nó, polygon làn đường dừng lại tại chân tiếp đất của bánh xe sau chiếc xe tải đó.
- **Thời tiết bất lợi & Ánh sáng:**
  - **Trời mưa / Cần gạt nước (Rain / Wiper - ví dụ BDD17):** Bỏ qua vùng bị che khuất cơ học bởi gạt nước hoặc mờ nhòe ở góc camera; chỉ vẽ vùng mặt đường nhìn rõ qua kính chắn gió.
  - **Đêm tối (Night - ví dụ BDD18, BDD26):** Chỉ vẽ vùng mặt đường được chiếu sáng bởi đèn pha xe hoặc đèn đường công cộng nhìn rõ ranh giới. Không vẽ vào mảng đen tối mịt mù.
  - **Tuyết phủ (Snowy - ví dụ BDD23, BDD24):** Nếu tuyết phủ lấp vạch kẻ đường nhưng lộ vệt bánh xe (wheel tracks), căn biên theo vệt bánh xe và mép tuyết đùn bên lề. Nếu tuyết phủ trắng xóa hoàn toàn không phân biệt được lề đường, chỉ vẽ dải hẹp an toàn trước đầu xe và tick `needs_review = true`.

## 7. Ambiguity / escalation (Thư viện quy tắc xử lý Edge Case)

Dưới đây là 10 tình huống biên (Edge Cases) thường gặp nhất trên tập dữ liệu BDD100K và quy tắc quyết định bắt buộc cho annotator:

### 1. Xe ego đang đè vạch chuyển làn (Lane Change in Progress)
- **Tình huống:** Thân xe hoặc nắp capo của ego đang nằm đè lên vạch phân làn giữa 2 làn đường.
- **Quy tắc:** Quan sát diện tích nắp capo nằm ở bên nào nhiều hơn:
  - Nếu diện tích nắp capo bên làn A $> 50\%$: Gán làn A là `direct`, làn B bên cạnh là `alternative`.
  - Nếu nằm chính giữa cân bằng (50/50): Gán làn mà đầu xe đang hướng mũi vào là `direct`, làn còn lại là `alternative`.
- **Thao tác CVAT:** Vẽ 2 polygon riêng biệt cho 2 làn, **bắt buộc tick `needs_review = true`** trên cả 2 polygon.

### 2. Vạch người đi bộ qua đường (Crosswalk) cắt ngang làn (ví dụ BDD18)
- **Tình huống:** Vạch sơn kẻ người đi bộ (dạng ngựa vằn hoặc vạch đôi) cắt ngang trước đầu xe.
- **Quy tắc:** Mặt đường nơi có vạch crosswalk vẫn là diện tích lưu thông hợp lệ của xe cơ giới khi qua giao lộ hoặc đoạn đường thẳng.
- **Thao tác CVAT:** Vẽ polygon `direct` (hoặc `alternative`) **bao trùm liên tục xuyên qua vạch crosswalk**. Tuyệt đối KHÔNG cắt đứt polygon trước crosswalk. Dừng biên ngoài polygon tại mép vỉa hè (không vẽ tràn lên vỉa hè dành cho người đi bộ).

### 3. Giao lộ / Ngã tư không có vạch phân làn (Intersections)
- **Tình huống:** Đi vào khu vực ngã ba/ngã tư, toàn bộ vạch kẻ sơn phân làn biến mất.
- **Quy tắc:**
  - Vùng không gian đi thẳng tiếp nối luồng xe của ego được vẽ là `direct` (chiều rộng bằng khẩu độ làn trước khi vào ngã tư).
  - Toàn bộ vùng mặt đường ngã tư cho phép rẽ cùng chiều (rẽ phải, rẽ trái hợp pháp) được vẽ thành các polygon `alternative`.
  - Tuyệt đối KHÔNG vẽ lấn vào luồng đường của chiều xe đối diện rẽ hoặc chạy qua.
- **Thao tác CVAT:** Gán `areaType` tương ứng, tick `needs_review = true` cho các vùng giao lộ không có vạch dẫn hướng.

### 4. Xe ô tô đỗ thành hàng dài sát lề đường (Parked Cars on Shoulder/Curb - ví dụ BDD10)
- **Tình huống:** Nhiều xe con đỗ song song hoặc vuông góc sát vỉa hè bên phải.
- **Quy tắc:**
  - Nếu các xe đỗ tạo thành chướng ngại vật cố định chiếm trọn một làn: Tuyệt đối KHÔNG vẽ polygon luồn vào khoảng trống giữa các xe hoặc vẽ vào sau đuôi xe đỗ. Làn đó bị coi là làn đỗ / bị chặn.
  - Đường biên polygon của làn xe chạy bên cạnh (`direct` hoặc `alternative`) sẽ vẽ men theo mép ngoài thân của hàng xe đang đỗ (cho phép cách thân xe đỗ khoảng 0.2–0.5m để tạo khoảng an toàn).
- **Thao tác CVAT:** Không vẽ lên thân xe hay chỗ đỗ xe. Chỉ vẽ làn lưu thông thông suốt.

### 5. Đường khu dân cư hai chiều không có vạch tim đường (Unmarked 2-Way Street - ví dụ BDD20)
- **Tình huống:** Phố hẹp trong khu dân cư chỉ có mặt đường nhựa, không có vạch sơn tim đường màu vàng hay vạch phân làn.
- **Quy tắc:**
  - Tự động chia đôi mặt đường theo trục dọc tâm đường tưởng tượng.
  - Nửa mặt đường bên phải (theo hướng nhìn của xe ego) được vẽ là `direct`.
  - Nửa mặt đường bên trái mặc định thuộc về luồng xe chạy ngược chiều $\rightarrow$ **IGNORE (Không vẽ)**.
  - Ngoại lệ: Nếu có biển báo đường 1 chiều rõ ràng hoặc toàn bộ xe lưu thông cùng hướng: Vẽ toàn bộ mặt đường (nửa phải là `direct`, nửa trái là `alternative`) và tick `needs_review = true`.

### 6. Tuyết phủ hoặc bùn đất che lấp hoàn toàn vạch kẻ (Snowy Roads - ví dụ BDD23, BDD24)
- **Tình huống:** Tuyết phủ trắng xóa lòng đường, không nhìn thấy vạch sơn hay mép bó vỉa hè.
- **Quy tắc:**
  - Nếu thấy vệt bánh xe (wheel tracks/ruts) đè trên tuyết: Căn ranh giới làn theo vệt bánh xe. Làn chứa vệt bánh xe của ego là `direct`, làn kế cận cùng chiều là `alternative`. Mép ngoài bám theo gờ tuyết đùn cao (snow banks). Tick `needs_review = true`.
  - Nếu tuyết phủ dày không có bất kỳ vệt bánh xe nào: Chỉ vẽ một dải an toàn hình thang ngay trước đầu xe (chiều rộng khoảng 3.5m, chiều dài nhìn thấy khoảng 15–20m) gán là `direct` và **tick `needs_review = true`**.

### 7. Hầm chui hoặc ban đêm thiếu sáng nghiêm trọng (Tunnel / Dark Night - ví dụ BDD16, BDD18, BDD26)
- **Tình huống:** Đi vào hầm tối hoặc đường ban đêm không có đèn cao áp, ánh sáng đèn pha chỉ rọi được một đoạn ngắn phía trước.
- **Quy tắc:**
  - Polygon chỉ vẽ tới ranh giới xa nhất mà ánh sáng đèn pha rọi rõ mặt đường (điểm cut-off sáng/tối). Tuyệt đối KHÔNG vẽ suy đoán vào khoảng tối đen mù mịt.
  - Nếu hầm quá tối hoặc camera bị hỏng, lóa đèn pha $> 60\%$ diện tích khung hình không thấy gì: KHÔNG vẽ polygon. Gán tag **`image_escalate`** cho toàn bộ ảnh.

### 8. Làn rẽ mở rộng, vùng nhập làn và vạch xương cá (Merge, Split & Gore Areas - ví dụ BDD01, BDD03)
- **Tình huống:** Làn nhánh chuẩn bị tách khỏi cao tốc hoặc nhập vào, có vùng vạch kẻ chéo hình chữ V (vạch xương cá / gore area).
- **Quy tắc:**
  - Làn rẽ cùng chiều trước điểm chia tách: Gán là `alternative`.
  - Vùng vạch xương cá (gore / chevron markings): Thuộc phạm vi **NGOÀI SCOPE**, tuyệt đối KHÔNG vẽ đè polygon lên vùng này.
  - Sau điểm chia tách bằng dải phân cách cứng: Nhánh đường rẽ ra coi như không thể tiếp cận $\rightarrow$ BỎ QUA.

### 9. Làn đường xe buýt chuyên dụng (Bus-Only Lane)
- **Tình huống:** Làn đường ngoài cùng bên phải có vạch sơn liền dày và sơn chữ lớn "BUS" hoặc "BUS ONLY".
- **Quy tắc:** Xe con không được phép lưu thông hợp pháp trên làn này. Mặc định là **NGOÀI SCOPE** $\rightarrow$ **IGNORE (Không vẽ)**.
- **Thao tác CVAT:** Không tạo polygon cho làn xe buýt. Nếu chữ "BUS" bị mờ không thể khẳng định 100%, vẽ polygon `alternative` và bắt buộc tick `needs_review = true`.

### 10. Rào chắn tạm thời hoặc cọc tiêu nón chóp công trường (Construction Cones)
- **Tình huống:** Có hàng cọc tiêu phản quang màu cam hoặc rào chắn tạm thời lấn vào làn đường.
- **Quy tắc:**
  - Nếu làn chỉ bị lấn một phần: Vẽ đường biên của polygon bẻ lượn vòng né ra ngoài hàng cọc tiêu để đảm bảo luồng di chuyển an toàn, không vẽ xuyên qua cọc.
  - Nếu làn đường bị chặn hoàn toàn: Polygon kết thúc ngay trước vị trí cọc tiêu đầu tiên. Phần đường phía sau cọc tiêu không được vẽ.

---

### Bảng tóm tắt ma trận quyết định CVAT

| Tình huống | Quyết định | Thể hiện trong CVAT | Ghi chú |
|---|---|---|---|
| Làn đường rõ ràng, đủ bằng chứng | **LABEL** | `drivable_area` (`areaType = direct` hoặc `alternative`), `needs_review = false` | Bình thường. |
| Vỉa hè, làn ngược chiều, làn xe bus/xe đạp, vạch xương cá | **IGNORE** | **Không vẽ** polygon lên vùng này | Ngoài scope. |
| Xe đang chuyển làn đè vạch phân làn | **LABEL + REVIEW** | Vẽ 2 polygon riêng biệt (`direct` làn $>50\%$, `alternative` làn còn lại), tick `needs_review = true` | Edge Case 1. |
| Crosswalk cắt ngang lòng đường | **LABEL** | Vẽ polygon trùm xuyên suốt qua crosswalk | Edge Case 2. |
| Giao lộ không có vạch phân làn | **LABEL + REVIEW** | Kéo thẳng luồng `direct`, các nhánh rẽ cùng chiều là `alternative`, tick `needs_review = true` | Edge Case 3. |
| Đường 2 chiều không vạch tim đường | **LABEL + REVIEW** | Nửa phải là `direct`, nửa trái IGNORE (không vẽ), tick `needs_review = true` | Edge Case 5. |
| Tuyết che vạch nhưng có vệt bánh xe | **LABEL + REVIEW** | Vẽ theo vệt bánh xe, tick `needs_review = true` | Edge Case 6. |
| Hầm tối đen mất điện / lóa đèn pha $>60\%$ | **ESCALATE** | Không vẽ polygon, gán tag **`image_escalate`** | Báo hỏng dữ liệu. |

## 8. Temporal rule

**Không áp dụng — task ảnh tĩnh.**
Tập dữ liệu BDD100K trong bài toán này bao gồm các ảnh tĩnh độc lập (single frames). Không có thuộc tính mutable theo thời gian và không liên kết tracking giữa các frame.

## 9. Examples

Các ví dụ tham chiếu từ tập ảnh thực tế của BDD100K trong `data/catalog.csv` (thuộc split `example` và `calibration`):

| sample_id | Thấy gì | Expected output | Rule áp dụng |
|---|---|---|---|
| `BDD03` | Cao tốc cong ban ngày, xe ego đang chạy ở làn giữa, có làn bên trái và bên phải ngăn bởi vạch đứt trắng. | Vẽ 03 polygon riêng biệt: 01 polygon `direct` cho làn ego đang chạy; 02 polygon `alternative` cho 2 làn kế cận hai bên. Không vẽ qua dải phân cách bê tông bên trái. | Tách rời từng region; phân biệt `direct` vs `alternative` theo làn ưu tiên của ego; không gộp polygon. |
| `BDD10` | Phố đô thị ban ngày, có nhiều xe ô tô đỗ song song sát lề đường bên phải. | Vẽ 01 polygon `direct` cho làn xe chạy. Đối với khu vực có xe đỗ: polygon drivable area chỉ bao phủ luồng đường thông suốt, không vẽ lách vào khoảng trống hẹp giữa các xe đỗ hoặc sau lưng xe đỗ nếu không phải làn chạy. | Loại trừ chướng ngại vật cố định và khu vực đỗ xe không thể lưu thông thông suốt (Edge Case 4). |
| `BDD16` | Xe đi vào hầm chui, ánh sáng yếu dần về phía xa, vạch kẻ phân làn mờ trong bóng tối. | Vẽ polygon `direct` và `alternative` bám theo phần mặt đường nhìn thấy; dừng polygon tại vị trí vách hầm tối đen không còn nhận dạng được mặt đường. Tick `needs_review = true` nếu biên mờ. | Giới hạn tầm nhìn xa trong điều kiện thiếu sáng; không vẽ mò vào vùng đen (Edge Case 7). |
| `BDD17` | Trời mưa, gạt nước kính chắn gió để lại vệt mờ ở rìa kính, phản chiếu đèn đường trên mặt đường ướt. | Vẽ polygon theo mặt đường nhìn xuyên qua kính; bỏ qua phần góc ảnh bị che bởi khung cần gạt nước. Tag `image_context`: `weather = rainy`. | Visibility: Bỏ qua vật che cơ học trước camera, chỉ gán nhãn vùng nhìn thấy thực tế. |
| `BDD18` | Phố ban đêm có vạch kẻ người đi bộ (crosswalk) cắt ngang trước đầu xe. | Vẽ polygon `direct` bao phủ trùm lên toàn bộ vạch crosswalk cắt ngang làn xe. Tuyệt đối không dừng cắt đứt polygon trước vạch crosswalk. Tag `image_context`: `timeofday = night`. | Inclusion: Crosswalk cắt ngang mặt đường xe chạy vẫn là vùng drivable hợp lệ (Edge Case 2). |
| `BDD20` | Đường phố khu dân cư hẹp ban ngày, không có vạch kẻ tim đường, có xe đỗ rải rác. | Vẽ nửa đường bên phải làm polygon `direct`, bỏ qua nửa đường bên trái (làn ngược chiều). Tick `needs_review = true`. Tag `image_context`: `weather = overcast`, `timeofday = daytime`. | Đường hai chiều không vạch kẻ tim đường (Edge Case 5). |
| `BDD23` | Khu dân cư ngập tuyết trắng xóa, tuyết phủ lấp mép đường nhưng có vệt bánh xe lún rõ. | Vẽ polygon `direct` bám theo vệt bánh xe của ego; dừng biên ngoài tại mép tuyết đùn cao. Bắt buộc tick `needs_review = true`. Tag `image_context`: `weather = snowy`. | Bám theo vệt bánh xe khi tuyết phủ (Edge Case 6). |

## 10. Common mistakes & Self-QC Checklist

### 10.1 Các lỗi thường gặp nhất cần tuyệt đối tránh
1. **Lỗi gộp các làn `alternative` thành một polygon lớn cắt ngang qua làn `direct`:**
   - *Hậu quả:* Hệ thống hiểu sai cấu trúc hình học, gây lỗi lập quỹ đạo chuyển làn.
   - *Cách khắc phục:* Mỗi vùng không gian liên tục phải là 1 polygon độc lập. Làn `alternative` bên trái và bên phải của làn `direct` bắt buộc phải là 2 polygon riêng biệt.
2. **Lỗi vẽ lấn sang làn ngược chiều (Opposing traffic lanes):**
   - *Hậu quả:* Đây là lỗi nguy hiểm mức độ **Critical** (xe tự hành có thể lập kế hoạch đâm vào luồng xe đối diện).
   - *Cách khắc phục:* Quan sát kỹ vạch sơn kép màu vàng, hướng đầu xe của các phương tiện phía trước, biển báo hoặc dải phân cách. Tuyệt đối không gán `alternative` cho làn xe đối diện.
3. **Lỗi cắt đứt polygon khi gặp vạch đi bộ qua đường (Crosswalk):**
   - *Hậu quả:* Báo hiệu sai cho xe tự hành rằng đường bị cụt / hết đường xe chạy.
   - *Cách khắc phục:* Xe ô tô vẫn được phép chạy đè qua vạch crosswalk, do đó phải vẽ trùm polygon qua vạch crosswalk theo luồng giao thông.
4. **Lỗi vẽ polygon đè chồng chéo (overlapping) lên nhau giữa `direct` và `alternative`:**
   - *Hậu quả:* Một vị trí không gian bị gắn 2 nhãn mâu thuẫn về mức độ ưu tiên.
   - *Cách khắc phục:* Chỉnh biên của 2 polygon bám khít dọc theo tâm của vạch kẻ phân làn (cho phép gối nhẹ biên $\le 2\text{ px}$, không đè lấn rộng).
5. **Lỗi kéo polygon vào khoảng tối không nhìn thấy hoặc vẽ xuyên qua xe tải lớn:**
   - *Hậu quả:* Sai lệch hình học nghiêm trọng do tưởng tượng thay vì gán nhãn theo dữ liệu cảm biến.
   - *Cách khắc phục:* Tuân thủ nguyên tắc "Chỉ vẽ những gì nhìn thấy được". Dừng polygon tại chân bánh xe của vật cản lớn che khuất toàn bộ mặt đường.
6. **Lỗi để sót thuộc tính `areaType = __undefined__`:**
   - *Hậu quả:* Polygon vô nghĩa, bị hệ thống kiểm tra tự động đánh trượt ngay lập tức.
   - *Cách khắc phục:* Luôn chọn giá trị cụ thể (`direct` hoặc `alternative`) ngay khi vừa tạo xong mỗi polygon.
7. **Lỗi gán nhãn làn xe buýt chuyên dụng hoặc làn xe đạp thành `alternative`:**
   - *Hậu quả:* Xe tự hành sẽ lập kế hoạch chuyển làn vào làn cấm, gây xung đột giao thông và vi phạm luật.
   - *Cách khắc phục:* Chú ý quan sát mặt đường để tìm vạch liền dày, chữ sơn "BUS", hoặc biểu tượng xe đạp. Nếu là làn chuyên dụng cấm xe ô tô, bỏ qua (không vẽ).

### 10.2 Bảng kiểm tra tự kiểm (Self-QC Checklist trước khi bàn giao)
Trước khi lưu và nộp bài, annotator tự kiểm tra theo checklist 5 bước sau:
- [ ] Không có polygon nào còn giữ giá trị mặc định `areaType = __undefined__`.
- [ ] Không có polygon nào vẽ lấn sang làn ngược chiều, làn xe bus chuyên dụng, làn xe đạp hoặc vỉa hè.
- [ ] Mọi làn `alternative` tách biệt đều là các polygon riêng biệt (không gộp chung).
- [ ] Đã gán tag `image_context` với đầy đủ thuộc tính `weather` và `timeofday` cho từng ảnh.
- [ ] Mọi trường hợp nghi vấn (vạch mờ, tuyết che, xe đè vạch) đều đã được đánh dấu `needs_review = true`.
