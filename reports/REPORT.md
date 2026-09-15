# Báo cáo Ngày 3 — Tracking Annotation

Họ tên / nhóm: Nguyễn Công Khải
Ngày: 2026-09-15

---

## 1. Quá trình gán nhãn

| Mục                               | Giá trị |
| --------------------------------- | ------- |
| Công cụ                           | CVAT    |
| Thời gian gán `clip_02` (warm-up) | 30 phút |
| Thời gian gán `clip_01`           | 50 phút |
| Số track đã vẽtrong `clip_01`     | 8       |
| Số keyframe trung bình mỗi track  | 77      |

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:

1. Frame 187, ID 3: Xe con bị xe tải ID 5 che khuất một phần bánh xe sau và đuôi xe. Xử lý: Đặt ngưỡng occluded = true khi ≥10% diện tích bị che, đủ điều kiện nên set occluded=true.
2. Frame 171, ID 6: Xe đang exit khung hình, còn lại phần mép đuôi/đèn xe. Xử lý: Chưa set outside=true vì phần còn lại vẫn cung cấp đủ đặc trưng (hình dạng đuôi xe, màu sắc, đèn), khớp với luật outside <15% diện tích.
3. Frame 77-78, ID 8: Xe ID 8 mới xuất hiện ở frame 77 nhưng diện tích chưa đủ ngưỡng. Xử lý: Bắt đầu track từ frame 78 khi xe đạt ≥10% diện tích khung hình, đảm bảo đủ thông tin hình dạng để gán nhãn chính xác.

## 2. Tự kiểm và kiểm chéo

Ba lượt tua bắt được gì (lượt 1 nhìn ID, lượt 2 frame đầu/cuối, lượt 3 frame giữa):

- Lượt 1: không có gì
- Lượt 2: không có gì
- Lượt 3: không có gì

Kiểm chéo với: chưa có đối tác. Chi tiết ở `reports/review_partner.md`.
Số lỗi bạn tìm được trong bản của bạn ấy: 0. Số lỗi bạn ấy tìm được trong bản của bạn: 0.

Ca nào hai người quyết khác nhau, và luật nào còn thiếu trong `GUIDELINE_MINI.md`:

Hiện chưa có kiểm chéo, nhưng qua tự kiểm phát hiện ghost tracks (track 4,6,8 có bbox trước/sau khi xuất hiện/rời khung) cần khắc phục.

## 3. Pre-gold lock và chấm trước/sau rework

| Evidence                                             | Giá trị                                                          |
| ---------------------------------------------------- | ---------------------------------------------------------------- |
| SHA-256 từ `evidence/pre-gold/clip_01/manifest.json` | a4aa227f78441aa80cd7f2240aa63aafdc0b338f8e5d95dcb6358a4d4b91f2e9 |
| Thời điểm khóa                                       | 2026-09-15T04:40:26.472188+00:00                                 |
| Số row / frame / track trước khi mở reference        | 615, 190, 8                                                      |

|              |               HOTA |   DetA |  AssA |   LocA |   IDF1 |   MOTA |  MOTP |  FP |  FN | IDSW |
| ------------ | -----------------: | -----: | ----: | -----: | -----: | -----: | ----: | --: | --: | ---: |
| Bản pre-gold |             0.8075 | 0.7927 | 0.824 | 0.8813 | 0.9562 | 0.9092 | 0.869 |  47 |   5 |    0 |
| Sau rework   | **Chưa có rework** |

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **Có**

Sau khi đọc danh sách lỗi, bạn đã sửa cụ thể những gì? Ghi theo frame và ID:

| Loại lỗi    | Frame   | ID  | Đã sửa thế nào                                            |
| ----------- | ------- | --- | --------------------------------------------------------- |
| Ghost track | 80-100  | 6   | Xóa bbox trước khi track 6 xuất hiện (bị ghost 21 frames) |
| Ghost track | 51-53   | 4   | Xóa bbox trước khi track 4 xuất hiện (bị ghost 3 frames)  |
| Ghost track | 149-151 | 4   | Xóa bbox sau khi track 4 đã rời khung (bị ghost 3 frames) |
| Ghost track | 133-135 | 8   | Xóa bbox trước khi track 8 xuất hiện (bị ghost 3 frames)  |
| Ghost track | 169-171 | 8   | Xóa bbox sau khi track 8 đã rời khung (bị ghost 3 frames) |

## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ `outputs/model_run_config.json`:

| Mục                                | Giá trị                                                   |
| ---------------------------------- | --------------------------------------------------------- |
| Python / ultralytics / torch / lap | 3.13.15 / 8.4.145 / 2.11.0+cu128 / 0.5.13                 |
| weights / hai tracker              | yolo26n.pt / ByteTrack control, BoT-SORT + ReID treatment |
| conf / IoU / imgsz / classes       | 0.25 / 0.7 / 960 / [2, 5, 7]                              |
| device                             | 0                                                         |

| So sánh                   |   HOTA |   DetA |   AssA |   LocA |   IDF1 |   MOTA |   MOTP |  FP |  FN | IDSW |
| ------------------------- | -----: | -----: | -----: | -----: | -----: | -----: | -----: | --: | --: | ---: |
| bạn vs gold               | 0.8075 | 0.7927 |  0.824 | 0.8813 | 0.9562 | 0.9092 |  0.869 |  47 |   5 |    0 |
| ByteTrack control vs gold | 0.7085 | 0.6487 | 0.7761 | 0.8463 | 0.8746 | 0.7487 | 0.8226 |  88 |  54 |    2 |
| BoT-SORT + ReID vs gold   | 0.7635 |  0.711 | 0.8204 | 0.8721 | 0.9001 | 0.7923 | 0.8595 |  91 |  26 |    2 |
| ReID vs bạn               | 0.7713 | 0.7156 | 0.8322 | 0.9001 | 0.8843 | 0.7691 | 0.8916 |  81 |  58 |    3 |

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

- MOTA (0.9092) thấp hơn IDF1 (0.9562). Điều này cho thấy annotation có độ chính xác ID rất cao (ít ID switch, FN thấp), nhưng MOTA bị ảnh hưởng bởi FP (47 false positives) và FN (5 false negatives).
- MOTA không phạt nặng lỗi ID vì nó chỉ tính FP, FN, IDSW theo tỷ lệ, trong khi IDF1 là metric chuyên biệt cho association accuracy, nhạy cảm hơn với lỗi ID switch.

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**

- ReID treatment tốt hơn ByteTrack control ở cả IDF1 (0.9001 vs 0.8746, +0.0255) và AssA (0.8204 vs 0.7761, +0.0443), nhưng IDSW đều là 2.
- Frame sequence giải thích: track ID 5 bị fragmented trong ByteTrack với pred_tracks [32, 23] (45+1 frames) và trong ReID là [18, 17] (51+1 frames). Do đó, ReID duy trì track chính dài hơn (51 vs 45 frames), cho thấy khả năng association tốt hơn.
  Tuy nhiên, không thể cô lập causal effect của ReID vì hai tracker có implementation khác nhau (ByteTrack vs BoT-SORT).

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

- So với ByteTrack: ReID cải thiện DetA (0.711 vs 0.6487, +0.0623) và giảm FN đáng kể (26 vs 54, -28), nhưng FP tăng nhẹ (91 vs 88, +3).
- So với annotation của bạn: ReID vs bạn có DetA=0.7156, FP=81, FN=58.
- Lỗi còn lại chủ yếu là detector (FP, FN cao), tuy nhiên AssA của ReID (0.8204 vs gold, 0.8322 vs bạn) cho thấy association cũng còn vấn đề, đặc biệt là ghost tracks do detector phát hiện sai.

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**

- Frame 16-116, ID 7: Trong eval_reid_vs_gold.json có ghost track 7 (43 frames, không khớp track tham chiếu nào), trong khi annotation của bạn không có ghost track 7 (không có trong ghost_pred_tracks của eval_vs_gold.json).
- Lý do: ReID phát hiện sai một vật thể không tồn tại trong gold, trong khi bạn tuân thủ đúng luật không gán nhãn cho vật thể không có trong real scene.

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**

- Frame 87, ID 5: Trong eval_reid_vs_gold.json có ID switch ở frame 87 (gt_track=5, from_track=17, to_track=18). Điều này cho thấy ReID có sự không ổn định trong association ở vùng này.
- Khi xem lại annotation, cần kiểm tra xem track ID 5 có bị fragmented không, hoặc có box nào bị thiếu/kém chính xác không. Nguyên nhân có thể là do ReID bị nhiễu bởi các xe có appearance tương tự trong khu vực có mật độ xe cao.

## 6. Nếu phải gán thêm 10 clip nữa

Bạn sẽ sửa gì trong `GUIDELINE_MINI.md`, và đổi gì trong quy trình làm việc của mình?

Trong `GUIDELINE_MINI.md` sẽ bổ sung:

- Luật kiểm soát ghost tracks: Không được có bbox trước khi track xuất hiện hoặc sau khi track đã rời khung. Set ngưỡng chặt chẽ hơn cho entry/exit frames.
- Luật occluded/outside: Làm rõ cách ước lượng phần trăm diện tích bị che/còn lại, sử dụng công cụ đo lường trong CVAT.

Trong quy trình làm việc:

- Thêm bước tự kiểm ghost tracks sau khi hoàn thành annotation bằng script tự động.
- Tăng tần suất keyframes ở các vùng có mật độ xe cao hoặc có nhiều interaction giữa các xe.
- Sử dụng chế độ interpolate thận trọng hơn, tránh tạo ra các bbox không chính xác.

## 7. Tệp đã nộp

- [x] `annotations/clip_01/gt.txt`
- [x] `annotations/clip_02/gt.txt`
- [x] `evidence/pre-gold/clip_01/gt.txt` và `manifest.json`
- [x] `GUIDELINE_MINI.md` đã điền
- [x] `outputs/eval_vs_gold.json`
- [x] `outputs/model_bytetrack_clip_01.txt`
- [x] `outputs/model_reid_clip_01.txt`
- [x] `outputs/model_run_config.json`
- [x] `outputs/eval_bytetrack_vs_gold.json`, `outputs/eval_reid_vs_gold.json`, `outputs/eval_reid_vs_me.json`
- [x] `reports/review_partner.md`
- [x] `reports/REPORT.md` (file này)
