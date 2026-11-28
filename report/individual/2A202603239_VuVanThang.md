# Báo cáo Day 13 — Vũ Văn Thắng, 2A202603239

## 1. Kết luận trước

- **Học viên & task:** Vũ Văn Thắng · MSSV `2A202603239` · task `D13-2A202603239` · 10 frame cố định: `s01f11`, `s01f57`, `s02f03`, `s02f23`, `s03f24`, `s03f55`, `s04f02`, `s04f17`, `s05f04`, `s05f28`.
- **Môi trường:** Arch Linux (`Linux 6.18.60-1-lts`, `x86_64`), trình duyệt Chromium, CVAT 3D chạy Docker cục bộ (phiên bản lớp sử dụng: CVAT 3D `v2.83.0`). Làm cá nhân nên chưa đổi kiểm trực tiếp với bạn ngồi cạnh trên lớp.
- **Tổng hợp kết quả:** Bảng A (`submission/D13-2A202603239-bangA.csv`) hiện có **64 cuboid** (`s01f11=6, s01f57=6, s02f03=6, s02f23=7, s03f24=6, s03f55=6, s04f02=7, s04f17=7, s05f04=6, s05f28=7`); các frame ít nhất đều có `6` cuboid (riêng `s01f11` có `6 < 8` cuboid khuyến nghị do các xe phía xa `> 40–45 m` quá thưa điểm nên bỏ qua theo quy tắc `LOW_VISIBILITY`). Phiếu tự kiểm (`submission/D13-2A202603239-tukiem.csv`) gắn cờ **`34/64` cuboid**; tôi tin nhất ở phần **class** và **hướng mũi đỏ `+X` (yaw)** trên 6 frame đầu (`s01f11` → `s03f55`), còn phần yếu nhất là **độ cao đáy `z_bottom` và một số hộp `0` điểm bị che/đặt nhanh** ở các frame `s03f24` → `s04f17`.
- **Thao tác nhanh trên CVAT:**
  1. **Đổi góc nhìn:** dùng chuột trái xoay 3D quanh cảnh, chuột phải dời khung nhìn, cuộn chuột thu/phóng; ở 3 khung phụ chọn `Top` để nhìn thẳng từ trên xuống (`BEV`) rồi zoom gần vật trước khi chỉnh.
  2. **Vẽ và chỉnh cuboid:** chọn nhãn rồi nhấn nút vẽ cuboid ở chế độ **`Shape`** (tuyệt đối không chọn `Track`; phím tắt `N` để đặt nhanh tiếp cùng class), nháy đúp hoặc chọn hộp trên danh sách `Objects` để kéo 6 mặt ở `Top` / `Side` / `Front` sao cho đáy chạm mặt đường và mũi đỏ `+X` hướng đầu xe.
  3. **Bật hướng và đối chiếu camera:** bật `Cuboid orientation` trong mục `Appearance` để hiện trục `+X` đỏ trên mọi hộp; mở đồng thời `cam01` / `cam03` / `cam05` (hoặc đổi sang `cam06`–`cam09`) trong CVAT để phân biệt đầu–đuôi và sau khi **Save** chạy `python scripts/lab.py sync` để xem ảnh chiếu `work/xem/`.

## 2. Đọc frame (Bước 2)

Hai frame đọc kỹ trước khi vẽ (ảnh lưu trong ZIP: `t1-s01f11-3d.png`, `t1-s01f11-top.png`, `t1-s01f57-3d.png`, `t1-s01f57-top.png` với dấu mũi tên `Phia truoc (-y)`, `Phia sau (+y)`, `+x (Trai)`, `-x (Phai)`):

| Frame | Phía trước xe ở cạnh nào của ảnh nhìn từ trên xuống; `+x`, `+y` chỉ về đâu | Mặt đường (phẳng/dốc; thấp hơn LiDAR ~ m) | Vật xa nhất đọc được (~ m) | Loại vật thấy | Vật phía sau / hai bên | Cụm chưa đọc được |
|---|---|---|---|---|---|---|
| `s01f11` | Phía trước (`-y`) ở **cạnh trên** ảnh `Top` (`t1-s01f11-top.png`); trục `+x` hướng **sang trái**, `+y` hướng **xuống dưới** (về phía sau xe) | Đường phẳng gần như nằm ngang; điểm mặt đường quanh các vòng tròn quét nằm ở `z ≈ -1,65` đến `-1,75 m` (thấp hơn cảm biến LiDAR `~ 1,7 m`) | Xe buýt làn cùng chiều phía trước đọc rõ ở `~ 38 m`, cụm thưa xa nhất tới `~ 45 m` | `Car`, `Bus` | Phía sau (`+y`) có 2 xe con chạy cùng chiều bám sau (`~ 20,5 m` và `~ 36,5 m`); bên trái qua dải phân cách có xe làn ngược chiều (`~ 24,8 m`) | Cụm điểm rất thưa phía sau xa (`> 45 m`, thấy trên `cam09`) và vài cụm lẻ phía trên cao/tán cây ven lề không rõ hình khối xe |
| `s01f57` | Phía trước (`-y`) ở **cạnh trên** ảnh `Top` (`t1-s01f57-top.png`); trục `+x` hướng **sang trái**, `+y` hướng **xuống dưới** | Đường phẳng, các vòng tròn quét quanh xe ở `z ≈ -1,65 m` (thấp hơn LiDAR `~ 1,7 m`) | Xe con đi trước cùng chiều đọc được tới `~ 33–35 m` | `Car` (và xe đạp dựng lề phải không người trên camera, không vẽ) | Phía sau (`+y`) có 1 xe con chạy sát phía sau (`~ 6,1 m`); sát bên trái–sau (`~ 3,1 m`) có 1 xe con vượt lên bên làn trái | Cụm điểm dày rối ở vỉa hè bên phải (`cam07`: tủ kỹ thuật cạnh hàng xe hai bánh dựng và người đứng sát khuất chéo) không tách rõ từng thân người/xe |

Ba điều **chỉ camera** cho biết và ba điều **chỉ LiDAR** cho biết trên `s01f11` và `s01f57`:
- **Chỉ camera cho biết:** (1) màu sắc thân xe và nhận biết xe buýt hay xe con ở tầm `~ 38 m` khi point cloud chỉ còn vài nét khung; (2) phân biệt đầu–đuôi xe đang dừng/xa dựa vào đèn pha trước và đèn hậu sau; (3) thấy xe hai bánh dựng ven lề (`s01f57`, `cam07`) có người lái hay không để quyết định có gắn `Cyclist` hoặc bỏ qua.
- **Chỉ LiDAR cho biết:** (1) khoảng cách 3D chính xác (`m`) và kích thước thật (`dài × rộng × cao`) của từng xe không bị biến dạng phối cảnh; (2) độ cao mặt đường (`~ -1,7 m`) và góc lệch phương ngang thực giữa các làn xe trên mặt bằng `Top`; (3) vật ở sát thân xe hoặc góc mù giao thoa giữa các camera (hoặc vùng bóng che hoàn toàn phía sau vật lớn).

Tám frame lướt:

| Frame | Phía trước ở cạnh nào của ảnh nhìn từ trên xuống | Điều lạ (nếu có) |
|---|---|---|
| `s02f03` | Cạnh trên (`-y` hướng lên) | Xe buýt lớn ở bên trái–sau (`~ 7,2 m`) tạo mảng bóng che đen rộng quét qua làn ngược chiều |
| `s02f23` | Cạnh trên (`-y` hướng lên) | Nhiều xe ngược chiều qua dải phân cách bị che khuất một phần (`occluded`) và có 1 `Cyclist` bên phải–sau (`~ 16,4 m`) |
| `s03f24` | Cạnh trên (`-y` hướng lên) | Có dải xe nối đuôi bên làn ngược chiều bị hàng rào giữa đường che khuất chân bánh |
| `s03f55` | Cạnh trên (`-y` hướng lên) | Đường đông xe hai bên, có 1 `Cyclist` di chuyển sát làn xe đạp bên phải–trước (`~ 15,1 m`, thấy rõ trên `cam07`) |
| `s04f02` | Cạnh trên (`-y` hướng lên) | Biển cảnh báo phản xạ mạnh bên trái (`~ 9 m`, `cam06`) sáng dày hơn hẳn xe con tối màu cạnh đó |
| `s04f17` | Cạnh trên (`-y` hướng lên) | Xe hai bánh `Cyclist` chạy phía sau bên phải (`~ 16,8 m`), mật độ xe trước tương đối thoáng |
| `s05f04` | Cạnh trên (`-y` hướng lên) | Xe con áp sát ngay bên phải ego (`~ 2,9 m`, dày đặc `4041` điểm) và có xe tải `Truck` phía sau |
| `s05f28` | Cạnh trên (`-y` hướng lên) | Hai làn khá thoáng, các xe ngược chiều gần bên trái (`~ 7,6–10,5 m`) bị dải cây phân cách che thấp |

## 3. Ba cuboid có hướng dẫn (Bước 3)

Ảnh chụp từ CVAT: `t2-a-car.png`, `t2-b-bus-truck.png`, `t2-c-ped-cyclist.png`.

| Vật | Frame, idx | Class | Mũi tên yaw đúng hướng? | Đáy chạm đường? | Đã kiểm `Top`/`Side`/`Front` + ảnh? | Kích thước trong bảng 4.3? |
|---|---|---|---|---|---|---|
| (a) xe con gần | `s01f11, idx 1` | `Car` | Đúng (`yaw_deg = 267,9°`, mũi đỏ `+X` hướng đầu xe) | Chạm đường (`z_bottom = -1,623 m`, `ground_gap_m = 0,15 m`) | Đã kiểm đủ 3 khung + `cam01`/`cam03` + `preview` | `4,500 × 1,850 × 1,480 m` — nằm gọn trong khoảng sedan/hatchback (`3,8–5,0 × 1,70–1,90 × 1,40–1,60 m`) |
| (b) xe buýt hoặc xe tải | `s01f11, idx 2` | `Bus` | Đúng (`yaw_deg = 271,6°`, mũi đỏ `+X` hướng đầu xe) | Chạm đường (`z_bottom = -1,642 m`, `ground_gap_m = 0,18 m`) | Đã kiểm đủ 3 khung + `cam01`/`cam03` + `preview` | `10,800 × 2,600 × 3,000 m` — nằm trong khoảng xe buýt thường (`10,0–12,5 × 2,50–2,60 × 3,00–3,50 m`) |
| (c) người đi bộ hoặc xe hai bánh | `s03f55, idx 1` | `Cyclist` | Đúng (`yaw_deg = 272,3°`, mũi đỏ `+X` hướng đầu xe) | Chạm đường (`z_bottom = -1,682 m`, `ground_gap_m = 0,07 m`) | Đã kiểm đủ 3 khung + `cam07` + `preview` (`occluded = 1`) | `1,800 × 0,650 × 1,700 m` — nằm gọn trong khoảng xe đạp có người (`1,6–1,9 × 0,50–0,70 × 1,60–1,90 m`) |

- **Vòng hiệu chuẩn & vật bỏ qua vì không đủ cơ sở:** Thống nhất quy tắc của guideline: không dùng `Track` mà chỉ dùng `Shape`, xe đang lăn bánh giữ đáy sát lớp mặt đường quanh `-1,7 m`, xe hai bánh dựng bên đường không người lái không gắn `Cyclist`, và giữ nguyên xe không vẽ thừa khi chỉ có `1–3` điểm rời rạc ở rất xa. Các vật đã bỏ qua vì thiếu cơ sở:
  - `s01f57`, lề phải (`cam07`, cụm xe đạp dựng cạnh tủ kỹ thuật và người đứng khuất chéo): nghi vấn `UNCERTAIN_CLASS` / `LOW_VISIBILITY` — xe hai bánh dựng không có người lái không gán `Cyclist`, còn phần thân người đứng bị che khuất hầu như không tách được trên point cloud.
  - `s01f11`, phía sau xa (`> 45 m` trên `cam09`) và các xe quá xa làn đối diện (`> 40 m`): nghi vấn `LOW_VISIBILITY` / `UNCERTAIN_SCOPE` — chỉ còn vài điểm rời rạc không đủ `15–20%` thể tích để xác định kích thước và góc `yaw`.

## 4. Bốn hiện tượng chất lượng (Bước 4)

Cả 4 hiện tượng đều lấy trực tiếp từ 10 frame cá nhân của task `D13-2A202603239` (không mượn frame ngoài).

| Hiện tượng | Frame / tên ảnh / nguồn | Nguyên nhân (giả thuyết) | Thấy gì trên point cloud | Hệ quả khi gán (theo guideline) |
|---|---|---|---|---|
| 1. Bóng che | `s02f03` / `t4-1-bong-che.png` / frame cá nhân `D13-2A202603239` | Giả thuyết do thân xe buýt lớn (`idx 2`, cao `3,000 m`, dài `10,800 m`, cách `7,18 m`) chắn hoàn toàn tia LiDAR phát ngang về phía trái–sau | Mặt quay về ego dày đặc (`3183` điểm) nhưng phía sau nó xuất hiện hình nón đen trống kéo dài sang làn đối diện; các xe bị che phía sau chỉ còn rất ít hoặc `0` điểm (như `idx 4` còn `12` điểm) | Bật `occluded = 1` cho các xe bị chắn và kéo cuboid theo kích thước chuẩn suy luận được từ ảnh camera, không mặc định vùng tối sau xe buýt là đường trống |
| 2. Thưa ở xa | `s01f11` / `t4-2-thua-o-xa.png` / frame cá nhân `D13-2A202603239` | Giả thuyết do chùm 40 tia LiDAR phân kỳ theo khoảng cách (`30–50 m`) nên mật độ điểm trên cùng diện tích giảm nhanh theo cự ly | Xe con `idx 5` ở `36,48 m` chỉ còn `66` điểm rời rạc trên khung `Top`/`Side`/`Front`, còn các xe phía sau xa hơn nữa (`> 45 m` thấy ở `cam09`) gần như mất hẳn đường viền | Áp dụng quy tắc `15–20%` thể tích + kích thước bảng 4.3 để vẽ `idx 5` (`occluded = 1`); các xe xa hơn nữa chỉ còn `0–3` điểm thì bỏ qua và ghi `LOW_VISIBILITY` |
| 3. Phản xạ thấp / ít điểm bất thường | `s04f02` / `t4-3-phan-xa-thap.png` / frame cá nhân `D13-2A202603239` | Giả thuyết do sơn xe tối màu và kính xe (`idx 4`, ở `18,62 m`) hấp thụ tia hồng ngoại mạnh hơn màng phản quang của biển báo ven trái (`~ 9 m`, thấy trên `cam06`) | Biển báo gần bên trái tạo cụm điểm dày và `intensity` sáng rõ hơn hẳn dù diện tích nhỏ, trong khi thân xe tối màu phía trước chỉ có `106` điểm thưa trên nóc/cốp | Chỉ ghi nhận ở mức giả thuyết dựa trên đối chiếu ảnh camera; vẫn bám khung điểm thưa và kích thước chuẩn xe con để đặt hộp, không nới hộp theo quầng sáng biển báo |
| 4. Điểm lạ | `s01f11` / `t4-4-diem-la.png` / frame cá nhân `D13-2A202603239` | Giả thuyết do phản xạ từ giá long môn biển báo cao tốc, tán cây ven đường hoặc nhiễu đa đường | Xuất hiện các điểm/cụm điểm rải rác lơ lửng phía trên nóc xe (`cam01`/`cam03`) hoặc tách rời ngoài lòng đường | Không gán nhãn thành phương tiện hay người đi bộ theo đúng guideline |

## 5. Số từ cuboid của tôi (Bước 5)

Số liệu đối chiếu trực tiếp từ `submission/D13-2A202603239-bangA.csv` và `submission/D13-2A202603239-tukiem.csv` (mốc mặt đường chuẩn `~ -1,7 m`, lệch hơn `0,3 m` là lơ lửng hoặc chìm):

| Cuboid (`frame`, `idx`, `class`) | `z` (`m`) | `h` (`m`) | `z đáy = z − h/2` (`m`) | `dist_m` (`m`) | `r = √(x²+y²+z²)` (`m`) | `yaw_deg` (`°`) | Nhận xét (lơ lửng / chìm / chạm đường) |
|---|---|---|---|---|---|---|---|
| `s01f11, idx 1, Car` | `-0,883 m` | `1,480 m` | `-1,623 m` | `6,03 m` | `6,090 m` | `267,9°` | Chạm đường (`ground_gap_m = 0,15 m`) |
| `s04f17, idx 2, Car` | `-1,649 m` | `1,680 m` | `-2,489 m` | `10,04 m` | `10,178 m` | `272,0°` | Chìm (`ground_gap_m = -0,72 m` do thả nhanh chưa nâng đáy ở `Side`) |
| `s04f17, idx 7, Cyclist` | `-0,052 m` | `1,700 m` | `-0,902 m` | `16,75 m` | `16,748 m` | `272,4°` | Lơ lửng (`ground_gap_m = 0,97 m` do thả nhanh chưa hạ đáy ở `Side`) |

- **(a) Cuboid gần nhất và xa nhất:** Theo Bảng A, dòng nhỏ nhất là `s03f55, idx 6, Car` (`dist_m = 0,00 m`, `r = 0,000 m`) — thực chất đây là một hộp đặt nhầm trùng gốc tọa độ `(0, 0, 0 m)` của xe thu dữ liệu; nếu tính trên vật thật ngoài ego thì cuboid gần nhất là `s05f04, idx 1, Car` (`dist_m = 2,85 m`, `r = 2,961 m`, `4041` điểm). Cuboid xa nhất là `s01f11, idx 2, Bus` (`dist_m = 37,86 m`, `r = 37,862 m`, `299` điểm; xe con xa nhất là `s01f11, idx 5, Car` với `dist_m = 36,48 m`, `r = 36,496 m`).
- **(b) So kích thước với bảng 4.3:** `s03f55, idx 1 (Cyclist)` có `length = 1,800 m`, `width = 0,650 m`, `height = 1,700 m` — cả 3 chiều nằm trong khoảng xe đạp có người (`1,6–1,9 m`, `0,50–0,70 m`, `1,60–1,90 m`); `s01f11, idx 1 (Car)` có `length = 4,500 m`, `width = 1,850 m`, `height = 1,480 m` — cả 3 chiều nằm trong khoảng sedan/hatchback (`3,8–5,0 m`, `1,70–1,90 m`, `1,40–1,60 m`), không có chiều nào ngoài khoảng (riêng `s02f23, idx 7, Cyclist` có `width = 0,800 m` hơi rộng hơn nhóm xe đạp `0,50–0,70 m`, tương ứng nhóm xe máy/xe điện rộng ngang có áo khoác rộng).
- **(c) Góc `yaw_deg` theo chiều di chuyển:** Xe đi cùng chiều phía trước `s01f11, idx 1 (Car)` có `yaw_deg = 267,9°` (gần `270°`); xe làn ngược chiều bên trái `s01f11, idx 6 (Car)` có `yaw_deg = 86,7°` (gần `90°`).

Mật độ theo khoảng cách (chọn 4 cuboid `Car` không bị cờ lỗi ở các `dist_m` tăng dần):

| Cuboid (`frame`, `idx`, `class`) | `dist_m` (`m`) | Số điểm (`points_in`) |
|---|---|---|
| `s01f11, idx 1, Car` | `6,03 m` | `2147` |
| `s02f03, idx 3, Car` | `15,09 m` | `454` |
| `s01f11, idx 3, Car` | `24,17 m` | `178` |
| `s01f11, idx 5, Car` | `36,48 m` | `66` |

- **Quy luật mật độ & đối chiếu slide:** Khi khoảng cách tăng gấp `4,01` lần (từ `6,03 m` lên `24,17 m`) thì số điểm trong hộp giảm `~ 12,1` lần (còn `~ 8,3%`, từ `2147` xuống `178` điểm), và khi xa gấp `6,05` lần (`36,48 m`) chỉ còn `66` điểm (`~ 3,1%`); xu hướng này khớp sát đường cong bài giảng (xe con ở `0–10 m`: `~ 1968`, `10–20 m`: `~ 718`, `20–30 m`: `~ 214`, `30–40 m`: `~ 72`, `60–80 m`: `~ 19` điểm).
- **Tự kiểm (`preview` & `selfcheck`):**
  - **2 cuboid đã sửa:** (1) `s01f11, idx 1, Car` — ban đầu bị đặt chỉ bằng nửa kích thước thật (`~ 2,25 × 0,91 × 0,72 m`), đã mở rộng và căn lại đúng `4,500 × 1,850 × 1,480 m`, `z_bottom = -1,623 m` ôm trọn cụm điểm chữ L; (2) `s01f11, idx 2, Bus` — chỉnh lại góc quay `yaw_deg` từ lệch sang `271,6°` và kéo sát mép đuôi xe buýt cho hết hở `0,74 m`.
  - **Các cảnh báo giữ nguyên (`34/64` hộp có cờ):** giữ nguyên các cờ *"điểm sát ngay ngoài mép"* ở những xe sát dải phân cách hoặc xe nối đuôi nhau (ví dụ `s01f57, idx 2, 3`, `s02f23, idx 7`, `s05f28, idx 3`) vì điểm bên ngoài thuộc thanh chắn hoặc xe kế bên chứ không phải nới hộp; các cờ đáy cao/thấp và `0` điểm ở `s02f03#5`, `s02f23#2,#4`, `s03f24`–`s04f17` phản ánh trung thực trạng thái các hộp bị che khuất (`occluded = 1`) hoặc nhóm hộp đặt nhanh theo vị trí BEV.
- **Đổi kiểm:** Chưa đổi kiểm trực tiếp với bạn ngồi cạnh trên lớp; đã tự đối chiếu chéo toàn bộ các cặp số đo (`s01f11, idx 1`: `dist_m = 6,03 m`, `yaw_deg = 267,9°`; `s01f11, idx 2`: `dist_m = 37,86 m`, `points_in = 299`) giữa `bangA.csv` và `tukiem.csv`, các số trùng khớp hoàn toàn.

## 6. Ý nghĩa của nhãn, cách dùng AI và điều còn chưa chắc

- **(a) Ý nghĩa đối với mô hình 3D:** Mô hình phát hiện vật thể 3D học hình học không gian thực (`x, y, z, dài, rộng, cao, yaw`) theo đơn vị mét để tính trực tiếp khoảng cách, vận tốc tương đối và quỹ đạo tránh va chạm, điều mà bounding box 2D đo bằng pixel trên ảnh phẳng không khôi phục được chính xác. Với vật thưa điểm ở xa hoặc bị che, cần quy tắc riêng (`15–20%` thể tích, bật `occluded`, bỏ qua khi quá mờ) để mô hình học ranh giới vật lý nhất quán thay vì học đoán bừa kích thước sai hoặc coi nhiễu ngẫu nhiên là vật thật.
- **(c) Điều còn chưa chắc (lỗi đã nhận diện để tự chỉnh thêm nếu có thời gian):**
  - Ở `s03f55`, có một hộp đặt nhầm ngay tại gốc `(0, 0, 0 m)` của xe thu dữ liệu và một số hộp ở `s03f55`–`s03f24` (đặc biệt một hộp bị đặt quá cao so với mặt đường) cùng cao độ đáy của các hộp thả nhanh ở `s04f02` (`idx 2, 4, 5, 6`) và `s04f17` (`idx 1..5, 7`) chưa chỉnh lại trên khung `Side`/`Front`.
  - Một số xe làn ngược chiều bị che hoàn toàn hoặc gần hoàn toàn sau hàng rào/xe buýt ở `s02f03`, `s02f23`, `s03f24`, `s04f02` hiện rơi vào trạng thái `0` điểm bên trong hộp, và frame đầu `s01f11` mới có `6` cuboid thay vì mức khuyến nghị `8` cuboid.
