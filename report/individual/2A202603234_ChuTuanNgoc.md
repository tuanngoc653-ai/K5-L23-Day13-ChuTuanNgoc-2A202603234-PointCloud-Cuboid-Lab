# Báo cáo Day 13 — Chu Tuan Ngoc, 2A202603234

## 1. Kết luận trước
Mã số `2A202603234`, task `D13-2A202603234`, 10 frame: s01f39, s01f57, s02f15, s02f52, s03f33, s03f50, s04f09, s04f52, s05f04, s05f37. Windows 11, CVAT 2.74.1 chạy local (Docker trong WSL; `cvat-sdk` 2.83 báo cảnh báo lệch phiên bản nhưng tạo task và xuất bảng A chạy bình thường).

Trình duyệt: Chrome. Không có bạn ngồi cạnh.

Tổng cộng **110 cuboid**. Ít nhất là s03f33 và s05f37 (5 hộp mỗi frame): ở s03f33 dải cây giữa đường che kín phía trái; ở s05f37 cảnh ít vật và điểm thưa nhất. `selfcheck` còn cờ **3/110** cuboid.

- **Tin nhất: đáy.** Mỗi hộp được đặt theo điểm mặt đường đo quanh chính nó; các cảnh báo về đáy mình đều kiểm lại bằng điểm bánh xe hoặc mặt đường.
- **Khá chắc: hướng của xe gần.** Yaw lấy từ hình dạng cụm điểm và làn đường, đối chiếu đầu xe trên camera cho xe gần; xe xa và xe bị che thì không thấy rõ đầu.
- **Yếu nhất: vật xa (trên 35 m) và vật bị che.** Chỉ thấy một mặt hoặc phần nóc, nên kích thước lấy theo xe chuẩn (4,6 × 2,0 × 1,5 m) và hướng suy theo làn, không thấy đầu xe. Mình đã xoá 5 hộp loại này vì nhận nhầm bụi cây là xe.
- **Class còn hai chỗ chưa chắc:** xe van (mình gán Truck) và xe máy đỗ không người (mình bỏ qua).

Cách dùng CVAT:

- **Đổi góc nhìn:** kéo chuột trái để xoay, kéo chuột phải để dời, lăn con lăn để zoom. Nhìn từ trên xuống: kéo chuột trái theo chiều dọc tới khi nhìn thẳng xuống. Nháy đúp vào chỗ trống để về góc nhìn mặc định.
- **Vẽ và chỉnh cuboid:** nút cuboid ở thanh trái → chọn label → **Shape** → nháy đúp lên cụm điểm để đặt hộp. Ở **Top**: kéo để dời, kéo cạnh để đổi dài/rộng, kéo chấm xanh để xoay. Ở **Side**, **Front**: chỉnh chiều cao và cho đáy chạm đường. Xong thì `Ctrl+S`.
- **Mũi tên hướng:** panel phải, **Appearance → Cuboid orientation**; vạch đỏ là +X của cuboid, chỉ về đầu xe. Xem cuboid trên 7 ảnh camera bằng `python scripts/lab.py preview` (ảnh trong `work/xem/`).

## 2. Đọc frame (Bước 2)
Hai frame đọc kỹ. Ảnh: `t1-<frame>-3d`, `t1-<frame>-top` (ảnh nhìn từ trên xuống, có đánh dấu phía trước, +x, +y).

| Frame | Phía trước xe ở cạnh nào của ảnh nhìn từ trên xuống; +x, +y chỉ về đâu | Mặt đường (phẳng/dốc; thấp hơn LiDAR ~ m) | Vật xa nhất đọc được (~ m) | Loại vật thấy | Vật phía sau / hai bên | Cụm chưa đọc được |
|---|---|---|---|---|---|---|
| s01f39 | Mũi tên xanh lá (+y) chỉ lên, mũi tên đỏ (+x) chỉ sang trái, nên phía trước xe (−y) là **cạnh dưới**. Theo cam01: +x là bên trái xe, +y là phía sau | Dọc hướng đi gần như phẳng (~0,01%); ngang hơi nghiêng, thấp dần sang trái ~1,2%. Thấp hơn LiDAR **~1,76 m**; vỉa hè phải cao hơn mặt đường ~0,55 m (đáy xe đạp ở −1,15 m) | Xe con ~39 m (55 điểm); ở ~50 m vẫn thấy một xe nhưng chỉ còn 25 điểm | Xe con, người đi bộ, người đi xe đạp | Trước: 3 xe con, 3 người đi bộ, 3 người đi xe đạp trên vỉa hè phải. Trái: xe con trắng sát xe, ~3,5 m. Phải: 1 người đi xe đạp. Sau: 2 xe con, 1 người đi xe đạp. Làn ngược chiều (x ~12–22 m): trống | Cụm thấp ~1 m sát lề phải ở (−6, −26); cụm dài 4,3 m, cao 2,8 m trên dải phân cách ở (11, 4), có thể là cây/bụi; các khối dài 11–15 m ở x ~25–32 m: hàng cây hoặc nhà phía xa |
| s01f57 | Giống frame trước (cạnh dưới là phía trước), vì cảm biến gắn cố định trên xe | Phẳng, nghiêng ngang ~0,4%; thấp hơn LiDAR **~1,80 m** | Xe van rõ ở ~35 m; xe xa nhất đã gán nhãn ~37 m nhưng chỉ thấy nóc; quá ~45 m chỉ còn vài điểm | Xe con, xe van, người đi bộ, người đi xe đạp | Trước: 10 xe con xếp hàng chờ đèn đỏ (nhiều xe chỉ thấy nóc vì bị xe trước che), 1 xe van trắng-đỏ, 1 người đi bộ và 1 người đi xe đạp trên vỉa hè. Trái: 1 xe con. Sau: 2 xe con | Bãi xe đạp công cộng ở (−11,5, −10,8), cao 1,1 m, không phải người nên không gán nhãn; vạch dài 4 m, cao 0,8 m ở lề phải (−5, −27), có thể là rào chắn |

**Chỉ camera cho biết:**
1. Màu sắc và chữ: đèn tín hiệu đang đỏ, biển chỉ làn, vạch kẻ đường.
2. Đâu là đầu, đâu là đuôi xe (đèn pha hay đèn hậu), từ đó biết xe cùng chiều hay ngược chiều.
3. Vật là gì: người hay cột, xe van hay xe tải, xe đạp đỗ hay có người đi.

**Chỉ LiDAR cho biết:**
1. Khoảng cách và kích thước thật bằng mét: tâm, dài, rộng, cao, `dist_m`.
2. Độ cao: mặt đường thấp hơn LiDAR ~1,75 m, vỉa hè cao hơn ~0,55 m, phía làn ngược chiều thấp hơn 0,3–0,7 m.
3. Vật bị che phía sau một vật khác (nóc xe nằm sau xe trước), và quét đủ 360° trong một lần.

Tám frame lướt (cả 8 frame: phía trước là **cạnh dưới**, cùng góc nhìn như trên):

| Frame | Phía trước ở cạnh nào của ảnh nhìn từ trên xuống | Điều lạ (nếu có) |
|---|---|---|
| s02f15 | Cạnh dưới | Hàng rào giữa đường tách làn ngược chiều; 7 xe ngược chiều, gần nhất ~4 m |
| s02f52 | Cạnh dưới | Mặt đường nghiêng sang trái nhiều nhất (~3,8%), phía làn ngược chiều thấp hơn ~0,3 m; nhiều bụi hoa che |
| s03f33 | Cạnh dưới | Dải cây giữa che kín phía trái nên rất ít vật (5 hộp); có xe máy có người và người đứng trên vỉa hè |
| s03f50 | Cạnh dưới | Hàng xe chờ phía trước, phần lớn chỉ thấy nóc; xe buýt ở xa ngoài 50 m |
| s04f09 | Cạnh dưới | Xe buýt xanh dương bên trái ~21 m, đáy ở −2,4 m: phía bên đó thấp hơn ~0,6 m; xe tải ngược chiều ở ~38 m |
| s04f52 | Cạnh dưới | Ngã tư: có xe đang rẽ (yaw ~285°) và người mặc áo phản quang; nhiều xe nằm ngoài 50 m |
| s05f04 | Cạnh dưới | Xe tải ben ngay phía sau; bồn cây nhô cao 0,6 m trên dải phân cách dễ nhầm là xe, mình đã xoá 2 hộp ở đây |
| s05f37 | Cạnh dưới | Ít vật nhất và điểm thưa nhất; xe công trình màu vàng sau dải cây đứng trên nền cao, đáy ở −1,3 m |

## 3. Ba cuboid có hướng dẫn (Bước 3)
Ảnh: `t2-a-car`, `t2-b-bus-truck`, `t2-c-ped-cyclist`.

| Vật | Frame, idx | Class | Mũi tên yaw đúng hướng? | Đáy chạm đường? | Đã kiểm Top/Side/Front + ảnh? | Kích thước trong bảng 4.3? |
|---|---|---|---|---|---|---|
| (a) xe con gần | s01f39, idx 1 | Car (xe trắng bên trái, ~3,5 m) | Đúng: yaw 272,4°; cam05 thấy đầu xe hướng về trước, cam09 thấy đuôi | Có: z đáy −1,79 m, mặt đường quanh xe ~−1,83 m | Có: Top/Side/Front trên CVAT; cam05, cam06, cam09 | 4,82 × 2,00 × 1,50 m. Dài, cao trong khoảng sedan (3,8–5,0; 1,40–1,60); rộng 2,00 m vượt 1,90 m vì tính cả gương |
| (b) xe buýt hoặc xe tải | s05f04, idx 3 | Truck (xe ben ngay phía sau, ~11,7 m) | Đúng: yaw 270,0°; cam09 thấy cabin quay về phía xe mình, xe chạy cùng chiều | Có: z đáy −1,68 m, mặt đường ~−1,71 m | Kiểm trên ảnh xuất ra (`preview`) và cam09 | 8,00 × 3,01 × 3,32 m. Dài (ước lượng, chỉ thấy mặt trước) và cao trong khoảng xe tải nặng (6,0–12,0; 3,00–4,00); rộng 3,01 m vượt 2,60 m vì tính cả gương |
| (c) người đi bộ hoặc xe hai bánh | s03f33, idx 4 | Pedestrian (đứng trên vỉa hè phải, ~11,4 m) | Chưa xác nhận: yaw 270,0° để mặc định, cam08 không đủ rõ hướng mặt | Có: z đáy −1,71 m, mặt đường ~−1,71 m | Kiểm trên ảnh xuất ra (`preview`) và cam08 | 0,60 × 0,60 × 1,79 m: cả ba chiều trong khoảng người lớn (0,4–0,9; 0,5–0,9; 1,50–1,95) |

Vòng hiệu chuẩn: **không tham gia**. Cuboid trong task của tôi được tạo với sự hỗ trợ của AI (đọc point cloud qua API CVAT, tính tham số hộp bằng script, đối chiếu ảnh camera), không vẽ trong buổi có coach, nên không có số liệu so với các bạn.

Các vật bỏ qua vì không đủ cơ sở:

| Frame | Vị trí (x, y) m | Mô tả | Loại nghi vấn |
|---|---|---|---|
| s01f39 | (−6,0; −26,4) | Cụm thấp ~1 m sát lề phải, 33 điểm | UNCERTAIN_CLASS |
| s01f39 | (−13,2; −19,9) | Cụm dài trên vỉa hè, giống rào hoặc bụi cây | UNCERTAIN_CLASS |
| s01f39 | (6,2; −49,8) | Xe con ở 50,2 m | Ngoài phạm vi 50 m |
| s01f57 | (−11,5; −10,8) | Bãi xe đạp công cộng đỗ, không người | UNCERTAIN_SCOPE |
| s01f57 | (−10,8; −39,5), (−14,4; −42,4) | Người đi bộ xa, ~20 điểm | LOW_VISIBILITY |
| s02f15 | (−6,1; −15,9) | Xe máy đỗ trên vỉa hè, không người | UNCERTAIN_SCOPE |
| s02f15 | (−6,6; 44,9) | Xe đỗ chéo ở 45 m, không rõ hướng | LOW_VISIBILITY |
| s02f52 | (−3,7; −49,2) | Xe ở 49 m, vài điểm lẫn tán cây | LOW_VISIBILITY |
| s03f50 | (−8,3; −22,2) | Người đi xe đạp, ~10 điểm | LOW_VISIBILITY |
| s03f50 | (11,4; −19,8) | Nóc xe sau dải cây, không thấy mặt đường quanh nó | LOW_VISIBILITY |
| s04f52 | xa phía trước | Xe trắng và xe buýt ngoài 50 m | Ngoài phạm vi 50 m |
| s05f04 | (12,6; −6,4), (12,9; 8,0) | Vật trên bồn cây nhô cao 0,6 m, ban đầu nhầm là xe | UNCERTAIN_CLASS |
| s05f37 | (13,5; −9,1) | Vật sau dải cây, như trên | UNCERTAIN_CLASS |

## 4. Bốn hiện tượng chất lượng (Bước 4)
Nguyên nhân ghi là **giả thuyết**. Frame mượn thì ghi rõ nguồn: task chung, hoặc MSSV của bạn cho mượn.

| Hiện tượng | Frame / tên ảnh / nguồn | Nguyên nhân (giả thuyết) | Thấy gì trên point cloud | Hệ quả khi gán (theo guideline) |
|---|---|---|---|---|
| 1. Bóng che | s01f57, idx 13, (0,1; −20,8); `t4-1-bong-che` | Xe đứng sau xe idx 3 (10,2 m), tia laser bị xe trước chặn | Chỉ 63 điểm, phần lớn là cung nóc xe; xe phía trước có 921 điểm; sau mỗi xe có vùng mặt đường không có điểm | Bật occluded; hộp bao phần suy luận được: đáy ở mặt đường, chiều cao bao hết điểm nóc; kích thước là ước lượng |
| 2. Thưa ở xa | s04f52, idx 10, (−2,6; 49,3); `t4-2-thua-o-xa` | Càng xa các tia càng cách nhau thưa, vật nhận ít tia | 48 điểm ở 49,4 m (so với 2196 điểm của xe ở 3,5 m, s01f39 idx 1), chỉ vài cung ở mặt hướng về LiDAR | Thấy khoảng 15–20% thể tích nên vẫn vẽ, kích thước lấy theo xe chuẩn; cụm thưa hơn (~10 điểm ở s03f50) thì bỏ, ghi LOW_VISIBILITY |
| 3. Phản xạ thấp / ít điểm bất thường | s03f50, idx 1, (−0,3; −9,7), xe con màu đen; `t4-3-phan-xa-thap` | Giả thuyết, chưa khẳng định: sơn đen và kính phản xạ laser kém. Phản chứng: có thể chỉ do LiDAR thấy một mặt xe | 1121 điểm ở 9,7 m, ~57% trung vị slide (~1968 ở 0–10 m); vùng kính cửa trống điểm ở khung Side | Ghi là giả thuyết; đặt kích thước theo mép ngoài cụm điểm và ảnh camera, không thu hộp theo chỗ trống |
| 4. Điểm lạ | s02f15, quanh (1,6; 28,7); `t4-4-diem-la` | Giả thuyết: phản xạ nhiều lần (tia dội qua bề mặt bóng như thân xe, hàng rào kim loại rồi mới về) | ~4 điểm ở z ≈ −9,9 m, thấp hơn mặt đường ~8 m. Tương tự: s04f52 (−4,7; 22,8) z ≈ −8,0 m; s05f04 dưới đầu xe tải z ≈ −2,7 m | Không gán thành vật; không kéo đáy hộp xuống để bao các điểm này |

Cả bốn hiện tượng lấy trong 10 frame của tôi, không mượn.

## 5. Số từ cuboid của tôi (Bước 5)
Lấy từ `submission/D13-<MÃ SỐ>-bangA.csv`; số điểm lấy từ cột `points_in` của `-tukiem.csv`. Mặt đường khoảng −1,7 m: z đáy lệch hơn 0,3 m là lơ lửng hoặc chìm.

| Cuboid (frame, idx, class) | z | h | z đáy = z − h/2 | dist_m | r = √(x²+y²+z²) | yaw_deg | Nhận xét (lơ lửng / chìm / chạm đường) |
|---|---|---|---|---|---|---|---|
| s01f39, 1, Car | −1,043 | 1,501 | −1,793 | 3,52 | 3,67 | 272,4 | Chạm đường |
| s05f04, 3, Truck | −0,016 | 3,320 | −1,676 | 11,67 | 11,67 | 270,0 | Chạm đường |
| s03f33, 4, Pedestrian | −0,813 | 1,787 | −1,706 | 11,37 | 11,40 | 270,0 | Chạm đường |

(a) **Gần nhất:** s05f04, idx 1, Car (xe trắng bên phải): `dist_m` = 3,31 m; r = 3,42 m. **Xa nhất:** s04f52, idx 10, Car (phía sau): `dist_m` = 49,42 m; r = 49,43 m. Ở gần, z làm r lớn hơn `dist_m` một chút; ở xa, z không đáng kể nên r ≈ `dist_m`.

(b) Người đi bộ s03f33, idx 4: 0,60 × 0,60 × 1,79 m, cả ba chiều trong khoảng người lớn của bảng 4.3. Xe con s01f39, idx 1: 4,82 × 2,00 × 1,50 m; dài và cao trong khoảng sedan, dài > rộng; rộng 2,00 m vượt 1,90 m vì tính cả gương, LiDAR chỉ thấy một bên xe nên bề rộng neo theo mặt gần rồi kéo sang theo kích thước chuẩn.

(c) Xe cùng chiều phía trước: s01f57, idx 3, `yaw_deg` = 270,0°. Xe ngược chiều: s04f09, idx 9, `yaw_deg` = 90,0° (và s02f52, idx 3, 90,0°). Cả hai đúng kỳ vọng ~270° và ~90°.

Mật độ theo khoảng cách (4 cuboid ở các `dist_m` khác nhau):

| Cuboid (frame, idx, class) | dist_m | Số điểm (`points_in`) |
|---|---|---|
| s01f39, 1, Car | 3,5 | 2196 |
| s01f57, 3, Car | 10,2 | 921 |
| s01f39, 3, Car | 28,3 | 175 |
| s01f39, 6, Car | 39,2 | 80 |

Quy luật: xa gấp ~11 lần (3,5 → 39,2 m) thì số điểm còn ~1/27; xa gấp đôi thì còn khoảng 1/4–1/5, gần với tỉ lệ nghịch bình phương khoảng cách. So với slide: 2196 (0–10 m) ≈ 1968; 921 (10–20 m) cao hơn 718 vì xe ở gần mép dưới khoảng; 175 (20–30 m) thấp hơn 214 vì xe này chỉ thấy đuôi; 80 (30–40 m) ≈ 72.

Tự kiểm, các cuboid **đã sửa** sau `selfcheck`:
- s01f57, idx 12, Car: cảnh báo "rất ít điểm (2)". Hộp cũ cao 1,5 m cắt ngang dưới nóc xe; tăng h lên 1,78 m, hộp giờ có 25 điểm.
- s02f52, idx 3, Car: rộng 2,33 m vượt 2,3 m; thu còn 2,15 m (tính gương), giữ mép gần LiDAR. Sửa tương tự cho các hộp rộng quá khổ ở s02f52, s04f09, s04f52.
- Xoá 5 hộp không đủ cơ sở (vật trên bồn cây, sau dải cây bị nhận nhầm là xe; ở s02f52, s03f50, s05f04 ×2, s05f37) và nâng đáy xe công trình s05f37, idx 4 lên −1,30 m. Tổng từ 115 xuống 110 cuboid, cờ từ 12 xuống 3.

Cảnh báo **giữ nguyên**:
- s05f04, idx 7, Car, "đáy cao hơn mặt đường 0,63 m": ngay dưới đáy hộp có điểm bánh xe ở ~−1,64 m, mặt đường quanh xe ~−1,62 m. Phía làn ngược chiều thấp hơn làn của tôi; giả thuyết là `selfcheck` ước lượng mặt đường bằng một mặt phẳng chung cho cả frame.
- s01f39, idx 12, Cyclist, "20 điểm sát ngay ngoài mép": các điểm này (x ~−9,85 đến −9,59 m) thuộc người đi bộ idx 11 đứng ngay cạnh, tâm cách tâm hộp Cyclist ~0,51 m. Trong hộp Cyclist vẫn có 20 điểm, từ mặt vỉa hè (~−1,18 m) lên ~0,33 m. Nới hộp ra sẽ trùm sang người bên cạnh, sai quy tắc "không bao dư", nên giữ nguyên: hai vật đứng sát nhau.
- s05f04, idx 11, Car, "height = 1,14 m": xe ngược chiều sau dải phân cách, cách 40,65 m, 62 điểm, đã bật occluded. Hộp bao trọn điểm quan sát được, từ sát mặt đường (~−2,04 m) lên ~−0,98 m; phía trên không có điểm nên không có số đo để kéo cao thêm. Giả thuyết: ở khoảng cách này tia laser thưa nên nóc không bị quét trúng. Cách hiểu khác cũng có lý: theo "bao phần suy luận được" thì có thể nâng lên ~1,45 m (xe con chuẩn); nếu coach chốt theo hướng này thì sửa.
- Các hộp có z đáy ~−1,15 m (người, xe đạp trên vỉa hè cao hơn đường ~0,55 m) và −2,0 đến −2,4 m (làn ngược chiều thấp hơn): đo mặt đất quanh từng hộp thì khớp, không phải lơ lửng hay chìm.

Đổi kiểm: không có bạn ngồi cạnh nên không đổi kiểm chéo.

## 6. Ý nghĩa của nhãn, cách dùng AI và điều còn chưa chắc
(a) **Vì sao nhãn 3D đo bằng mét:** point cloud là toạ độ thật quanh cảm biến, một xe dài ~4,6 m dù cách 3 m hay 40 m, còn trên ảnh cùng chiếc xe chiếm số pixel rất khác tuỳ khoảng cách và camera. Nhãn bằng mét cho phép so kích thước với bảng chuẩn, kiểm đáy chạm mặt đường (~−1,7 m), tính `dist_m` và yaw, dùng chung một nhãn cho cả 7 camera, và cho model phát hiện 3D học vị trí thật để xe tự lái tính đường đi; pixel chỉ có nghĩa trong một ảnh và không cho biết độ sâu.

**Vì sao vật thưa điểm cần quy tắc riêng:** số điểm giảm rất nhanh theo khoảng cách (2196 điểm ở 3,5 m, 175 ở 28,3 m, 48 ở 49,4 m), vật bị che chỉ lộ phần nóc, nên không đo trực tiếp được đủ dài, rộng, cao, hướng và người gán phải suy đoán; mỗi người đoán một kiểu thì nhãn không nhất quán, và một cụm vài điểm có thể chỉ là bụi cây hoặc điểm lạ (5 hộp đã phải xoá vì nhầm bồn cây là xe). Vì vậy cần quy tắc thống nhất: thấy ~15–20% thể tích thì vẽ, bật occluded, kích thước theo chuẩn; ít hơn thì bỏ qua và ghi LOW_VISIBILITY / UNCERTAIN_SCOPE.

(b) Dùng AI:
- Công cụ: một công cụ AI tạo cuboid qua API CVAT và soạn nháp nội dung; Claude Code (trợ lý lập trình trong terminal).
- Câu hỏi chính: cài môi trường `.venv`, điền `configs/lab.ini`, bật CVAT bằng Docker, chạy các lệnh `lab.py` (`dry`, `task`, `sync`, `check_submission.py`), giải thích ý nghĩa cảnh báo selfcheck, phạm vi gán nhãn (360°, ≤ 50 m, cả làn ngược chiều), commit/push.
- Dùng kết quả thế nào: cuboid được tạo với sự hỗ trợ của một công cụ AI (đọc point cloud qua API CVAT, tính tham số hộp bằng script, đối chiếu ảnh camera), rồi sửa lại theo cảnh báo `selfcheck`. Nội dung mục 2–6 do công cụ đó soạn nháp; Claude Code trình bày vào mẫu báo cáo và đối chiếu số với bảng A, phiếu tự kiểm.

(c) Điều còn chưa chắc: kích thước và hướng của vật xa (trên 35 m) và vật bị che, vì chỉ thấy một mặt hoặc phần nóc; hướng người đi bộ s03f33 idx 4 chưa xác nhận bằng camera; class xe van (đang gán Truck); chiều cao s05f04 idx 11 nên giữ theo điểm quan sát hay nâng theo kích thước chuẩn.
