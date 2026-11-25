# Báo cáo Day 13 — Nguyễn Thành Tài, 2A202603038

## 1. Kết luận trước

Mã số của tôi là `2A202603038`, task cá nhân là `D13-2A202603038`. Mười frame được giao là `s01f02`, `s01f57`, `s02f03`, `s02f38`, `s03f10`, `s03f55`, `s04f09`, `s04f32`, `s05f07` và `s05f42`. Tôi làm bài trên Windows bằng PowerShell, mở CVAT local trên trình duyệt Microsoft Edge; CVAT Server báo phiên bản 2.74.1.

Tôi tạo 69 cuboid: `s01f02` có 11 hộp, `s01f57` có 6, `s02f03` có 6, `s02f38` có 6, `s03f10` có 6, `s03f55` có 6, `s04f09` có 6, `s04f32` có 6, `s05f07` có 9 và `s05f42` có 7. Các frame ít nhất có 6 hộp; đây là những frame tôi chỉ giữ lại các vật đủ rõ điểm và đủ cơ sở nhận dạng trong phạm vi làm bài. Phiếu tự kiểm còn cảnh báo 9/69 cuboid. Các cảnh báo tập trung vào hộp xa ít điểm, đáy lệch mặt đường, kích thước nằm ngoài khoảng tham khảo hoặc điểm sát mép; cảnh báo là tín hiệu để rà soát, không tự động có nghĩa là cuboid bị trừ điểm.

Tôi tự tin nhất ở việc xác định class và hướng xe nhờ kết hợp ảnh camera với mũi tên yaw. Phần còn yếu nhất là căn đáy và kích thước của vật xa hoặc bị thiếu điểm, vì một vài điểm quan sát được không đủ mô tả trọn hình dạng.

Trong CVAT, tôi xoay khung 3D bằng kéo chuột trái, di chuyển bằng kéo chuột phải và phóng to/thu nhỏ bằng con lăn. Khi vẽ, tôi chọn cuboid rồi chọn **Shape**, nháy đúp vào cụm điểm, sau đó chỉnh hộp ở các khung Top/Side/Front. Tôi bật mũi tên orientation để kiểm tra đầu xe và dùng `preview` để xem lại cuboid trên các ảnh camera.

## 2. Đọc frame (Bước 2)

Hai frame đọc kỹ là `s01f02` và `s01f57`. Ảnh minh chứng tương ứng là `t1-s01f02-3d.png`, `t1-s01f02-top.png`, `t1-s01f57-3d.png` và `t1-s01f57-top.png`.

| Frame | Phía trước xe ở cạnh nào của ảnh nhìn từ trên xuống; +x, +y chỉ về đâu | Mặt đường | Vật xa nhất đọc được | Loại vật thấy | Vật phía sau / hai bên | Cụm chưa đọc được |
|---|---|---|---|---|---|---|
| `s01f02` | Phía trước ở phía trên ảnh; `+x` hướng sang trái, `+y` hướng ra sau/phía dưới ảnh | Đường tương đối phẳng, thấp hơn LiDAR khoảng 1,7 m | Khoảng 29 m; vật xa có rất ít điểm | Chủ yếu là Car và một Pedestrian | Có xe ở trước, hai bên và phía sau vùng quan sát chính | Các điểm rời ở vùng xa và sát mép cảnh chưa đủ rõ để xác định là xe hay kết cấu đường |
| `s01f57` | Phía trước ở phía trên ảnh; `+x` sang trái, `+y` ra sau/phía dưới ảnh | Đường tương đối phẳng, thấp hơn LiDAR khoảng 1,7 m | Khoảng 16 m đối với vật đã đọc rõ | Chủ yếu là Car | Có các xe xếp theo làn và các điểm dọc hai bên đường | Điểm ở biên đường và vùng xa thưa, khó tách thành vật riêng |

Ba điều camera cho biết rõ hơn LiDAR là màu và hình thức bề mặt xe, chi tiết như kính/đèn/biển báo, và mặt trước hay mặt sau của xe khi điểm bị che. Ba điều LiDAR cho biết rõ hơn camera là khoảng cách và vị trí 3D theo mét, chiều cao/hình dạng không phụ thuộc ánh sáng, và các vật nằm ngoài hướng nhìn của từng camera.

Tám frame còn lại được lướt qua như sau:

| Frame | Phía trước ở cạnh nào của ảnh nhìn từ trên xuống | Điều lạ hoặc điểm cần chú ý |
|---|---|---|
| `s02f03` | Phía trên ảnh | Có Bus gần và một Car rất xa, điểm của vật xa thưa |
| `s02f38` | Phía trên ảnh | Có Cyclist gần; một số xe ở khoảng 30 m chỉ còn ít điểm |
| `s03f10` | Phía trên ảnh | Có Truck giữa các xe con; điểm quanh vật xa thưa hơn |
| `s03f55` | Phía trên ảnh | Có Bus ở xa và xe ở khoảng 30–36 m, cần cẩn thận khi đặt đáy |
| `s04f09` | Phía trên ảnh | Các xe xa có ít điểm và một hộp có đáy cần kiểm tra lại |
| `s04f32` | Phía trên ảnh | Có Cyclist ở gần và các xe hai bên làn |
| `s05f07` | Phía trên ảnh | Có Truck và một Car ở khoảng trên 30 m, mật độ điểm giảm rõ |
| `s05f42` | Phía trên ảnh | Có hai Truck và một Cyclist; các vật ở hai phía có hướng khác nhau |

## 3. Ba cuboid có hướng dẫn (Bước 3)

Ảnh minh chứng là `t2-a-car.png`, `t2-b-bus-truck.png` và `t2-c-ped-cyclist.png`.

| Vật | Frame, idx | Class | Mũi tên yaw đúng hướng? | Đáy chạm đường? | Đã kiểm Top/Side/Front + ảnh? | Kích thước trong bảng 4.3? |
|---|---|---|---|---|---|---|
| (a) xe con gần | `s01f02`, idx 1 | Car | Có, `yaw_deg = 273,6°`, gần hướng cùng chiều | Có, `z_bottom = -1,739 m` | Có, xem trong `t2-a-car.png` | `l=3,461 m`, `w=2,078 m`, `h=1,395 m`; chiều cao hơi thấp nhưng vẫn có thể do góc nhìn và phần xe bị khuất |
| (b) xe buýt | `s02f03`, idx 3 | Bus | Có, `yaw_deg = 269,2°` | Gần mặt đường, `z_bottom = -1,774 m` | Có, xem trong `t2-b-bus-truck.png` | `l=12,174 m`, `w=3,200 m`, `h=3,450 m`; chiều dài và chiều cao hợp lý, chiều rộng cần thận trọng vì lớn hơn khoảng tham khảo thông thường |
| (c) xe hai bánh có người | `s02f38`, idx 5 | Cyclist | Có, `yaw_deg = 270,6°` | Có, `z_bottom = -1,651 m` | Có, xem trong `t2-c-ped-cyclist.png` | `l=1,894 m`, `w=0,929 m`, `h=1,564 m`; phù hợp với khoảng tham khảo của xe máy có người |

Điểm quan trọng tôi rút ra từ vòng hiệu chuẩn là cần thống nhất cách hiểu guideline: mỗi vật là một cuboid Shape riêng, cuboid bao đủ phần nhìn thấy và phần bị che có cơ sở suy luận, mũi tên chỉ đúng đầu vật, đáy chạm mặt đường và class phải dựa trên loại vật chứ không chỉ dựa vào hình dáng một phần. Những cụm quá xa hoặc quá ít điểm mà không nhận dạng được thì không nên cố gán nhãn; cần ghi là trường hợp thiếu khả năng quan sát hoặc không chắc class. Trong bài này, các điểm rời không thuộc vật thể không được gán thành cuboid.

## 4. Bốn hiện tượng chất lượng (Bước 4)

| Hiện tượng | Frame / tên ảnh / nguồn | Nguyên nhân (giả thuyết) | Thấy gì trên point cloud | Hệ quả khi gán |
|---|---|---|---|---|
| 1. Bóng che | `s01f02` / `t4-1-bong-che.png` / task cá nhân `D13-2A202603038` | Xe lớn ở gần chắn hướng quét của LiDAR | Phía sau vật che có vùng rất ít điểm, trong khi hai bên vẫn còn điểm; vùng trống là vùng chưa quan sát được chứ không phải chắc chắn không có vật | Bật `occluded` khi có vật bị che; cuboid vẫn bao phần bị che nếu có đủ cơ sở suy luận |
| 2. Thưa ở xa | `s01f02` / `t4-2-thua-o-xa.png` / task cá nhân `D13-2A202603038` | Khoảng cách lớn làm góc quét thưa dần và số điểm trên vật giảm | Vật ở khoảng trên 30 m chỉ còn một số điểm, mép và chiều cao khó xác định | Chỉ vẽ khi còn khoảng 15–20% thể tích hoặc có cơ sở suy luận; nếu không đủ thì bỏ qua và ghi `LOW_VISIBILITY` |
| 3. Phản xạ thấp / ít điểm bất thường | `s02f03` / `t4-3-phan-xa-thap.png` / task cá nhân `D13-2A202603038` | Có thể do bề mặt tối, kính hoặc vật liệu phản xạ tia LiDAR kém | Một bề mặt có ít điểm hơn vật xung quanh ở khoảng cách tương tự; camera giúp kiểm tra đó vẫn là vật thật | Chỉ ghi nguyên nhân là giả thuyết; không nới cuboid theo quầng điểm và cần thận trọng với class/kích thước |
| 4. Điểm lạ | `s03f10` / `t4-4-diem-la.png` / task cá nhân `D13-2A202603038` | Nguyên nhân thu nhận điểm chưa rõ | Có cụm điểm lẻ loi không nối với vật nào và nằm thấp bất thường so với mặt đường xung quanh | Không gán cụm điểm này thành một vật thể |

## 5. Số từ cuboid của tôi (Bước 5)

| Cuboid (frame, idx, class) | z (m) | h (m) | z đáy (m) | dist_m (m) | r (m) | yaw_deg | Nhận xét |
|---|---:|---:|---:|---:|---:|---:|---|
| `s01f57`, idx 1, Car | -0,911 | 1,646 | -1,734 | 3,52 | 3,64 | 269,4° | Đáy gần mặt đường, chạm đường |
| `s02f38`, idx 5, Cyclist | -0,869 | 1,564 | -1,651 | 5,32 | 5,39 | 270,6° | Đáy gần mặt đường, chạm đường |
| `s02f03`, idx 6, Car | -1,429 | 2,167 | -2,512 | 36,69 | 36,71 | 92,4° | Có dấu hiệu chìm; cần kiểm tra lại ở Side/Front vì vật ở xa và điểm thưa |

Cuboid gần nhất trong bảng là `s01f57` idx 1 với `dist_m = 3,52 m` và `r = 3,64 m`. Cuboid xa nhất là `s02f03` idx 6 với `dist_m = 36,69 m` và `r = 36,71 m`.

Với Cyclist `s02f38` idx 5, kích thước `1,894 × 0,929 × 1,564 m` nằm gần khoảng tham khảo của xe máy có người. Với Car `s01f57` idx 1, kích thước `4,858 × 2,223 × 1,646 m`; chiều dài phù hợp xe con/SUV, còn chiều rộng hơi lớn nếu so với sedan nên cần đối chiếu với ảnh camera và gương xe. Một xe cùng chiều phía trước có yaw gần 270°, ví dụ `s01f57` idx 1 có `269,4°`; một xe ngược chiều có yaw gần 90°, ví dụ `s02f03` idx 6 có `92,4°`.

| Cuboid | dist_m (m) | points_in |
|---|---:|---:|
| `s01f57`, idx 1, Car | 3,52 | 3104 |
| `s01f02`, idx 3, Car | 14,55 | 582 |
| `s05f07`, idx 7, Car | 31,84 | 159 |
| `s03f55`, idx 5, Car | 36,51 | 85 |

Bốn ví dụ cho thấy số điểm giảm mạnh khi khoảng cách tăng: từ 3104 điểm ở 3,52 m xuống 85 điểm ở 36,51 m. Đây là xu hướng phù hợp với đường cong mật độ điểm trong bài giảng, nhưng số điểm không chỉ phụ thuộc khoảng cách mà còn phụ thuộc hướng của vật, vật liệu và che khuất.

Sau khi xem `preview` và `selfcheck`, tôi tập trung rà lại các hộp có cảnh báo đáy hoặc kích thước, đặc biệt `s01f02` idx 7 và `s02f03` idx 6; thao tác cần kiểm là chỉnh tâm/đáy ở Top, Side và Front, rồi Save và chạy lại `sync`. Tôi giữ lại các cảnh báo trong phiếu tự kiểm vì chúng chỉ ra các ca khó: hộp xa ít điểm, điểm sát mép, kích thước tham khảo chưa chắc và đáy có khả năng chìm. Tôi không chỉnh cuboid chỉ để làm mất cờ cảnh báo.

Tôi chưa ghi lại MSSV của người kiểm chéo và hai con số được đối chiếu trong workspace; phần này cần bổ sung nếu đã thực hiện kiểm chéo.

## 6. Ý nghĩa của nhãn, cách dùng AI và điều còn chưa chắc

Đối với mô hình, nhãn 3D phải đo bằng mét vì cuboid biểu diễn vị trí và kích thước thật trong không gian, không phụ thuộc trực tiếp vào độ phân giải hay khoảng cách của ảnh camera. Vật ít điểm cần quy tắc riêng vì một hộp có thể bao phần suy luận lớn hơn phần điểm thực tế; nếu không kiểm tra mức độ chắc chắn thì hộp dễ bị quá to, sai class hoặc đặt sai đáy.

Tôi đã dùng AI như sau:

- Công cụ: ChatGPT/Codex trong workspace.
- Câu hỏi chính: cách chạy lệnh CVAT, đọc các khung Top/Side/Front, hiểu các trường trong bảng A và tổ chức báo cáo.
- Cách dùng kết quả: dùng để kiểm tra quy trình và tổng hợp số liệu từ các file sinh ra; các cuboid.

Điều tôi còn chưa chắc là cách đánh giá các vật rất xa khi chỉ còn ít điểm, nguyên nhân chính xác của các vùng phản xạ thấp và việc phân biệt hộp bị chìm với mặt đường dốc trong các cảnh có ít điểm. Tôi sẽ ưu tiên đối chiếu lại các ca này ở Side/Front và với ảnh camera thay vì suy đoán chỉ từ một góc nhìn.
