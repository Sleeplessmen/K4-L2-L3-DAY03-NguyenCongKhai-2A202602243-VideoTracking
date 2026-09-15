# Mini annotation guideline — Ngày 3 (tracking)

> Điền file này **trong lúc gán nhãn**, không phải sau khi xong. Mỗi lần bạn dừng
> lại nghĩ "cái này tính sao nhỉ?" thì đó là một dòng phải ghi vào đây.
>
> Đây là tài liệu mà người gán nhãn tiếp theo sẽ đọc để làm giống bạn. Nếu hai
> người trong nhóm gán khác nhau, gần như luôn là vì file này chưa nói rõ — chứ
> không phải vì ai kém.

Nhóm / tên: Nguyễn Công Khải<br>
Clip: `clip_01`, `clip_02`

---

## 1. Phạm vi: gán cái gì, không gán cái gì

Một lớp duy nhất: **`vehicle`** — xe bốn bánh có động cơ (xe con, van, xe buýt, xe tải).

| Gán                           | Không gán                                                |
| ----------------------------- | -------------------------------------------------------- |
| xe con, SUV, taxi, xe bán tải | người đi bộ                                              |
| van, minivan                  | xe đạp                                                   |
| xe buýt, minibus              | **xe máy / mô tô** (dưới 4 bánh)                         |
| xe tải, xe đầu kéo            | xe trong ảnh quảng cáo, phản chiếu gương, dưới bóng nước |

## 2. Luật ID — phần quan trọng nhất

| Tình huống                       | Luật của nhóm                                                                                                            | Vì sao                                                     |
| -------------------------------- | ------------------------------------------------------------------------------------------------------------------------ | ---------------------------------------------------------- |
| Xe bị che một phần rồi hiện lại  | giữ nguyên ID nếu bị che **dưới 25 frame** (mặc định của lab: 25 frame = 2 giây @ 12.5 fps)                              | 2 giây là ngưỡng an toàn để không vỡ track do che ngắn hẳn |
| Xe bị che 25-50 frame            | Giữ nguyên ID nếu có thể dự đoán quỹ đạo từ frame trước/sau (ví dụ: xe che là xe khác, có dấu hiệu chuyển động liên tục) | Tránh vỡ track do che tạm thời                             |
| Xe bị che lâu hơn ngưỡng trên    | **track mới** nếu che > 50 frame (4 giây) mà không có thông tin nào (bán khuất, đổi làn)                                 | Che quá lâu mất tin cậy, tránh gán nhầm                    |
| Xe rời khung hình rồi quay lại   | mặc định: **track mới**                                                                                                  | Không thể khẳng định là cùng xe, tránh sai lầm             |
| Hai xe cắt nhau / chồng lên nhau | giữ ID theo **quỹ đạo chuyển động**, không theo bbox — dùng frame trước/sau để xác định                                  | Bbox chồng nhưng xe vẫn là 2 vật thể riêng                 |

## 3. Luật bbox

| Tình huống                             | Luật của nhóm                                                                                        |
| -------------------------------------- | ---------------------------------------------------------------------------------------------------- |
| Xe bị cắt bởi rìa ảnh                  | bbox chạm đúng rìa, không đoán phần ngoài ảnh                                                        |
| Xe bị xe khác che một phần             | bbox ôm **chặt** phần nhìn thấy được, không bao gồm xe che phía trước                                |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | bắt đầu track từ frame đầu tiên xác định được là xe bốn bánh; ngưỡng: **diện tích ≥ 10% khung hình** |
| Xe đang đỗ, không di chuyển            | **vẫn track bình thường** — không bỏ, không nhấp nháy, giữ ID cho đến khi biến mất                   |
| Keyframe đặt dày ở đâu                 | **mọi 25 frame** + các frame quan trọng: che, cắt nhau, vào/ra khung, đổi làn                        |
| Vật thể bị che khuất                   | Bật cờ occluded = true khi **≥10% diện tích bị che**                                                 |
| Vật thể ra khỏi khung hình             | Bật cờ outside = true khi **<15% diện tích còn nhìn thấy**                                           |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

Ghi **frame cụ thể** và **ID cụ thể**, không ghi chung chung.

### Ca 1

- Clip / frame / ID: clip_01/frame 000187.jpg/ID 3
- Tình huống: Vật thể bị che khuất khoảng bao nhiêu phần trăm thì có thể set occluded cho box không?
- Quyết định:
  - Đặt ngưỡng quy định: Bật cờ `occluded` khi ≥10% diện tích khung hình bị che.
  - Đối với xe con ID 3 tại frame 000187: bắt buộc set `occluded` = `true` (trong frame, xe tải ID 5 đã che khuất một ít phần bánh xe sau và đuôi xe)
- Lý do:
  - Ước lượng rằng phần bị che chiếm hơn 10% diện tích xe.
  - Khớp với luật phần 3 ("Bật cờ occluded = true khi ≥10% diện tích bị che") và dễ đo lường bằng tool.
  - Giúp mô hình phân biệt được giữa vật thể đầy đủ biên dạng và vật thể bị vật cản chèn lên, tránh việc mô hình học nhầm đặc trưng của xe tải phía trước gộp vào xe con phía sau.

### Ca 2

- Clip / frame / ID: clip_01/frame 000171.jpg/ID 6
- Tình huống: Vật thể exit, liệu diện tích vật thể còn lại quá nhỏ và cần set outside cho box không?
- Quyết định:
  - Quy tắc exit threshold: Set outside = true ngay khi <15% diện tích khung hình còn nhìn thấy, hoặc khi kích thước phần còn lại quá nhỏ/không còn đủ đặc trưng nhận dạng.
  - Đối với ID 6 tại frame 000171: Vẫn còn một phần mép đuôi/đèn xe nên vẫn chưa set `outside` = `true` cho object.
- Lý do:
  - Phần mép đuôi/đèn xe còn lại vẫn cung cấp đủ đặc trưng nhận dạng (như hình dạng đuôi xe, màu sắc, đặc trưng đèn) để theo dõi liên tục.
  - Khớp với luật phần 3 ("Bật cờ outside = true khi <15% diện tích còn nhìn thấy").

### Ca 3

- Clip / frame / ID: clip_01/frame 000077.jpg/ID 8
- Tình huống: Vật thể có ID = 8 đã bắt đầu xuất hiện ở frame 000077 nhưng đến frame 000078 mới được gán entry box, vậy quy định kích thước để bắt đầu gán entry box như nào cho hợp lý?
- Quyết định:
  - Quy tắc entry threshold: Chỉ bắt đầu tạo track và gán box đầu tiên khi vật thể xuất hiện trong khung hình đạt tối thiểu 10-15% diện tích tổng thể và mắt thường nhìn thấy được.
  - Ở frame 000077 xe chưa đạt tối thiểu 10% tổng diện tích, quyết định bắt đầu gán từ frame 000078.
- Lý do:
  - Ngưỡng 10-15% diện tích đảm bảo vật thể có đủ thông tin hình dạng (như bánh xe, đầu xe, thân xe) để gán nhãn chính xác.
  - Ở frame 000077, xe ID 8 xuất hiện nhưng diện tích chưa đủ ngưỡng, không đủ đặc trưng để phân biệt với noise hoặc vật thể khác, tránh gán nhầm.
  - Phù hợp với luật "bắt đầu track từ frame đầu tiên xác định được là xe bốn bánh; ngưỡng: diện tích ≥ 10% khung hình" ở phần 3.

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

Luật nào trong file này hoá ra còn thiếu hoặc còn mơ hồ? Viết lại cho rõ: hiện chưa có
