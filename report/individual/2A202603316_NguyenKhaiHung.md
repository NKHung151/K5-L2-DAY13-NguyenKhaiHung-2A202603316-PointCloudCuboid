# Báo cáo Day 13 — Nguyen Khai Hung, 2A202603316

## 1. Kết luận trước
- Mã số: 2A202603316. Task: `D13-2A202603316`.
- 10 frame: s01f28, s01f44, s02f10, s02f27, s03f00, s03f18, s04f24, s04f52, s05f22, s05f40.
- Hệ điều hành: Windows 11. Phiên bản CVAT: 2.82.0.
- Trình duyệt: coccoc. MSSV bạn ngồi cạnh:.

Tôi gán tổng cộng 58 cuboid trên 10 frame, phần lớn là Car (44), còn lại 10 Cyclist, 2 Truck và 2 Bus. Hai frame đông nhất là s01f28 và s03f18 (8 cuboid mỗi frame), sáu frame còn lại chỉ có 5. Chỗ tôi yên tâm nhất là các xe con ở gần: hướng và kích thước khá ổn, xe cùng chiều và ngược chiều tách bạch rõ. Chỗ tôi chưa yên tâm là nhóm xe hai bánh ở s03f18 (hướng không đều, đáy hơi lơ lửng) và vài xe ở xa hoặc ít điểm.

- **Đổi góc nhìn:** lăn chuột để zoom, kéo chuột để xoay hoặc dời khung nhìn; khi chỉnh vị trí và hướng thì chuyển sang ô **Top** cho dễ nhìn.
- **Vẽ và chỉnh cuboid:** chọn công cụ cuboid, bấm **Shape**, nháy đúp vào đám điểm để đặt box, rồi kéo cạnh và xoay trong ba ô **Top / Side / Front** đến khi box ôm sát điểm và đáy chạm đường.
- **Kiểm tra hướng và ảnh:** xoay để vạch đỏ ngắn chỉ về đầu xe, nhìn ảnh camera để chắc đâu là đầu, đâu là đuôi; cuối cùng chạy `preview` để xem box chiếu lên ảnh có khớp không.

## 2. Đọc frame (Bước 2)
Hai frame đọc kỹ. Ảnh: `t1-<frame>-3d`, `t1-<frame>-top`.

| Frame | Phía trước xe ở cạnh nào của ảnh nhìn từ trên xuống; +x, +y chỉ về đâu | Mặt đường (phẳng/dốc; thấp hơn LiDAR ~ m) | Vật xa nhất đọc được (~ m) | Loại vật thấy | Vật phía sau / hai bên | Cụm chưa đọc được |
|---|---|---|---|---|---|---|
| s01f28 | Chưa ghi lại (xem `t1-s01f28-top`); phía trước là −y, +x bên trái, +y phía sau | Không bằng lắm, chỗ xa hơi thấp xuống; thấp hơn LiDAR khoảng 1,7–1,9 m | ~35 m | 7 Car, 1 Truck | Phía sau có 4 xe con; bên trái có xe tải và một xe con | Chưa ghi lại |
| s01f44 | Chưa ghi lại (xem `t1-s01f44-top`); phía trước là −y, +x bên trái, +y phía sau | Khá phẳng, thấp hơn LiDAR khoảng 1,75 m | ~16 m | 5 Car | Một xe ngay phía sau, một xe sát bên trái, ba xe phía trước | Chưa ghi lại |

**Chỉ camera cho biết:**
1. Dáng và màu xe. Ví dụ một chiếc to ở s05f40, nhìn point cloud thì không biết là van chở người hay xe chở hàng, phải nhìn ảnh mới phân biệt được.
2. Vạch kẻ đường, biển báo, đèn giao thông, đèn phanh của xe phía trước.
3. Vật ở xa: LiDAR chỉ còn lác đác vài điểm, nhưng trên ảnh vẫn nhận ra đó là xe gì.

**Chỉ LiDAR cho biết:**
1. Khoảng cách thật: ví dụ ở s01f44 xe phía trước cách khoảng 10 m.
2. Kích thước thật và độ cao so với mặt đường: ví dụ chiếc xe tải ở s01f28 dài gần 6 m.
3. Những gì nằm ở góc mù giữa các camera. LiDAR quét đủ 360°, nên 4 xe phía sau ở s01f28 thấy ngay trên khung 3D, dù không có trong camera phía trước.

Tám frame lướt:

| Frame | Phía trước ở cạnh nào của ảnh nhìn từ trên xuống | Điều lạ (nếu có) |
|---|---|---|
| s02f10 | Chưa ghi lại | Có một xe buýt dài; hai xe hai bánh bên phải đi ngang so với dòng xe, có thể đang băng qua đường |
| s02f27 | Chưa ghi lại | Xe xa nhất có đáy thấp hơn hẳn các xe khác, nhưng là do đường dốc xuống chứ box không bị lún |
| s03f00 | Chưa ghi lại | Có một xe khá to, cỡ van chở người, vẫn để là Car |
| s03f18 | Chưa ghi lại | Năm xe hai bánh đi gần thành hàng nhưng hướng mỗi chiếc một kiểu, có chiếc ngược hẳn; vài box hơi lơ lửng |
| s04f24 | Chưa ghi lại | Một xe con to hơn bình thường; box xe hai bánh hơi rộng |
| s04f52 | Chưa ghi lại | Một xe con bị xe phía trước che gần hết; xe hai bánh ở xa chỉ còn rất ít điểm |
| s05f22 | Chưa ghi lại | Box xe hai bánh gần như vuông và hướng lệch hẳn so với dòng xe |
| s05f40 | Chưa ghi lại | Xe buýt ở rất gần; một xe to không rõ là Car hay Truck; một xe bên trái ít điểm, đáy hơi thấp |

## 3. Ba cuboid có hướng dẫn (Bước 3)
Ảnh: `t2-a-car`, `t2-b-bus-truck`, `t2-c-ped-cyclist`.

| Vật | Frame, idx | Class | Mũi tên yaw đúng hướng? | Đáy chạm đường? | Đã kiểm Top/Side/Front + ảnh? | Kích thước trong bảng 4.3? |
|---|---|---|---|---|---|---|
| (a) xe con gần | s03f00, #3 (khoảng 4 m, trước-trái) | Car | Có, cùng hướng với các xe trong làn | Có | Có — Top/Side/Front + `preview` | Có, đúng cỡ sedan |
| (b) xe buýt hoặc xe tải | s05f40, #3 (khoảng 10 m, trước-trái) | Bus | Có, cùng hướng với hai xe con bên cạnh | Có | Có — Top/Side/Front + `preview` | Dài và cao thì có; bề rộng hơi lớn, vượt sai số cho phép |
| (c) người đi bộ hoặc xe hai bánh | s04f24, #4 (khoảng 15 m, phía sau) | Cyclist | Có, đi cùng chiều xe thu dữ liệu | Có | Có — Top/Side/Front + `preview` | Nếu là xe máy thì nằm trong sai số; nếu là xe đạp thì quá rộng. Chưa chắc là loại nào |

Vòng hiệu chuẩn: không có kết quả đối chiếu với lớp để ghi lại. Vật bỏ qua vì không đủ cơ sở: không ghi lại danh sách.

## 4. Bốn hiện tượng chất lượng (Bước 4)
Nguyên nhân ghi là **giả thuyết**. Frame mượn thì ghi rõ nguồn: task chung, hoặc MSSV của bạn cho mượn.

| Hiện tượng | Frame / tên ảnh / nguồn | Nguyên nhân (giả thuyết) | Thấy gì trên point cloud | Hệ quả khi gán (theo guideline) |
|---|---|---|---|---|
| 1. Bóng che | s04f52 Car #3 / `t4-1-bong-che` / task của tôi | Xe phía trước nằm cùng hướng, chắn mất tia laser | Xe phía sau chỉ còn một phần điểm, ít hơn nhiều so với xe cùng khoảng cách | Bật Occluded (bảng A đang để 0, cần sửa), box vẫn vẽ đủ cả xe |
| 2. Thưa ở xa | s04f52 Cyclist #4 / `t4-2-thua-o-xa` / task của tôi | Ở xa khoảng 30 m, vật lại nhỏ nên tia laser trúng rất ít | Chỉ vài chục điểm, box ra hình dạng hơi lạ | Chỉ vẽ khi thấy đủ phần vật hoặc ảnh camera rõ; ghi LOW_VISIBILITY |
| 3. Phản xạ thấp / ít điểm bất thường | s05f40 Car #4 / `t4-3-phan-xa-thap` / task của tôi | Có thể xe màu tối hoặc thân xe bóng, phản xạ kém | Ở khoảng cách gần mà số điểm chỉ bằng một nửa các xe khác | Vẫn vẽ nếu ảnh cho thấy rõ là xe; ghi LOW_VISIBILITY |
| 4. Điểm lạ | s02f35 / `t4-4-diem-la` / task chung (theo gợi ý trong docs/PHENOMENA.md) | Có thể tia laser phản xạ nhiều lần qua bề mặt bóng; chưa xác minh | Một cụm điểm nhỏ phía sau bên trái nằm thấp hẳn dưới mặt đường | Không gán thành vật |

## 5. Số từ cuboid của tôi (Bước 5)
Lấy từ `submission/D13-2A202603316-bangA.csv`; số điểm lấy từ cột `points_in` của `-tukiem.csv`. Mặt đường khoảng −1,7 m: z đáy lệch hơn 0,3 m là lơ lửng hoặc chìm.

| Cuboid (frame, idx, class) | z | h | z đáy = z − h/2 | dist_m | r = √(x²+y²+z²) | yaw_deg | Nhận xét (lơ lửng / chìm / chạm đường) |
|---|---|---|---|---|---|---|---|
| s03f00, #3, Car | −1,038 | 1,491 | −1,784 | 3,83 | 3,97 | 270,9 | Chạm đường |
| s03f18, #1, Cyclist | −0,452 | 1,674 | −1,289 | 9,08 | 9,10 | 274,2 | Lơ lửng |
| s05f40, #4, Car | −1,356 | 1,645 | −2,179 | 14,02 | 14,09 | 89,9 | Có vẻ bị chìm |

(a) Gần nhất là một xe con ở s05f22 (`dist_m` 3,69 m, r 3,81 m), xa nhất là một xe con ở s01f28 (`dist_m` 35,18 m, r 35,20 m). Ở gần thì r và `dist_m` chênh nhau thấy rõ, còn ở xa gần như bằng nhau, vì độ cao của LiDAR lúc đó không còn đáng kể. (b) Xe con s03f00 #3: dài 4,15 m, rộng 1,86 m, cao 1,49 m, so với sedan (3,8–5,0 × 1,70–1,90 × 1,40–1,60 m) thì cả ba chiều đều nằm trong khoảng. Xe hai bánh s04f24 #4: dài 1,72 m, rộng 1,04 m, cao 1,73 m, so với xe máy có người (1,8–2,3 × 0,70–0,95 × 1,40–1,80 m) thì chiều cao nằm trong khoảng, còn chiều dài hơi ngắn và chiều rộng hơi rộng nhưng vẫn trong sai số ±10%, vì box có thể ôm cả tay chân người lái nên bị rộng ra, còn đuôi xe ít điểm nên kéo box hơi ngắn. (c) Xe cùng chiều phía trước (s01f44 #4) có yaw 270,8°, gần 270°; xe ngược chiều (s01f28 #5) có yaw 90,4°, gần 90°.

Mật độ theo khoảng cách (4 cuboid ở các `dist_m` khác nhau):

| Cuboid (frame, idx, class) | dist_m | Số điểm (`points_in`) |
|---|---|---|
| s05f22, #1, Car | 3,7 | 3311 |
| s01f44, #4, Car | 10,4 | 799 |
| s02f27, #4, Car | 21,7 | 349 |
| s01f28, #8, Car | 35,2 | 120 |

Nhìn chung xe ở xa gấp đôi thì số điểm chỉ còn khoảng một phần ba. So với đường cong trên slide, ở tầm 10–20 m số của tôi khá sát, nhưng ở xa hơn thì xe của tôi lại nhiều điểm hơn một chút, có lẽ vì mấy xe này không bị che và quay hông về phía LiDAR. Tự kiểm: `selfcheck` hiện không cờ cuboid nào. Tôi đã sửa các box bị cảnh báo rồi chạy lại `sync`, nhưng quên ghi lại đã sửa những box nào. Những chỗ tôi vẫn còn nghi: một xe con ở s01f28 hơi lơ lửng, mấy xe hai bánh ở s03f18 lơ lửng khoảng 0,3 m, và ba box không tính được khoảng hở với mặt đường.

Đổi kiểm: chưa đổi kiểm với bạn nào.

## 6. Ý nghĩa của nhãn, cách dùng AI và điều còn chưa chắc
(a) Model học từ nhãn này cần biết vật thật cách xe bao xa và to cỡ nào để còn quyết định phanh hay né, nên nhãn phải tính bằng mét; pixel thì to nhỏ tùy vật gần hay xa, không nói lên kích thước thật. Còn vật thưa điểm thì cần quy tắc riêng, vì với vài chục điểm mỗi người sẽ đoán một kiểu, và model sẽ học phải những nhãn không nhất quán.

(b) Dùng AI:
- Công cụ: Claude.
- Câu hỏi chính: cách dùng CVAT, lệnh và quy tắc lab.

(c) Còn chưa chắc: chiếc xe to ở s05f40 là Car hay Truck; hướng của mấy xe hai bánh ở s03f18 và s02f10; đáy chiếc xe ít điểm ở s05f40; xe bị che ở s04f52 có cần bật occluded hay không; và xe hai bánh ở s04f24 là xe máy hay xe đạp.
