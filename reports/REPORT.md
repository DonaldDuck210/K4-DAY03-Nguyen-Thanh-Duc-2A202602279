# Báo cáo Ngày 3 — Tracking Annotation

Chép file này thành `reports/REPORT.md` rồi điền. Giữ nguyên các tiêu đề.

Họ tên / Solo: `Nguyễn Thanh Đức`
Ngày: `15/9/2026`

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT / khác: `CVAT` |
| Thời gian gán `clip_02` (warm-up) | `30` phút |
| Thời gian gán `clip_01` | `45` phút |
| Số track đã vẽ trong `clip_01` | `7` |
| Số keyframe trung bình mỗi track | `3` |

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:

1. `Ô tô khuất sau xe bus - Khi nào chắc chắn thấy ô tô mới track`
2. `Xe bị mờ do đi quá nhanh - Gán rộng để đảm bảo không bị thiếu`
3. `Xác định khi nào xe bị chắn - Đặt ra một tỉ lệ cho việc xe bị chắn hay không`

## 2. Tự kiểm và kiểm chéo

Ba lượt tua bắt được gì (lượt 1 nhìn ID, lượt 2 frame đầu/cuối, lượt 3 frame giữa):

- Lượt 1: `Kiểm tra tính liên tục của ID, phát hiện các trường hợp một đối tượng có khả năng bị đổi ID hoặc một ID bị gán cho hai đối tượng khác nhau.`
- Lượt 2: `Kiểm tra frame bắt đầu và kết thúc của từng track, nhằm phát hiện track xuất hiện quá sớm, kết thúc quá muộn hoặc bỏ sót frame`
- Lượt 3: `Kiểm tra các frame giữa track, đặc biệt ở những đoạn có chuyển động, che khuất hoặc hai đối tượng đi gần nhau`

Kiểm chéo với: `No one`. Chi tiết ở `reports/review_partner.md`.
Số lỗi bạn tìm được trong bản của bạn ấy: `0`. Số lỗi bạn ấy tìm được trong bản của bạn: `0`.

Ca nào hai người quyết khác nhau, và luật nào còn thiếu trong `GUIDELINE_MINI.md`?

`...`

## 3. Pre-gold lock và chấm trước/sau rework

| Evidence | Giá trị |
| --- | --- |
| SHA-256 từ `evidence/pre-gold/clip_01/manifest.json` | `Chưa có: thư mục pre-gold hiện chưa có manifest.json` |
| Thời điểm khóa | `Chưa khóa pre-gold` |
| Số row / frame / track trước khi mở reference | `Chưa có snapshot độc lập để xác nhận` |

| | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Bản pre-gold | Chưa có evidence | Chưa có evidence | Chưa có evidence | Chưa có evidence | Chưa có evidence | Chưa có evidence | Chưa có evidence | Chưa có evidence | Chưa có evidence | Chưa có evidence |
| Annotation hiện tại (chưa có pre-gold) | 0.803 | 0.783 | 0.826 | 0.873 | 0.962 | 0.921 | 0.857 | 44 | 1 | 0 |

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **có**

Sau khi đọc danh sách lỗi, bạn đã sửa cụ thể những gì? Ghi theo frame và ID:

| Loại lỗi | Frame | ID | Đã sửa thế nào |
| --- | --- | --- | --- |
| Bbox treo trước thời điểm gold xuất hiện | 62-78 | 5 | Chưa sửa trong annotation hiện tại; cần đặt `outside` tại frame trước 62 |
| Bbox treo trước thời điểm gold xuất hiện | 89-100 | 6 | Chưa sửa trong annotation hiện tại; cần đặt `outside` tại frame trước 89 |
| Bbox treo sau khi gold kết thúc | 149-151 | 4 | Chưa sửa trong annotation hiện tại; cần đặt `outside` tại frame 148 |
| Bbox trôi giữa keyframe | 81-92 | 5 | Chưa sửa trong annotation hiện tại; cần thêm keyframe và chỉnh bbox bám phần xe nhìn thấy |

## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ `outputs/model_run_config.json`:

| Mục | Giá trị |
| --- | --- |
| Python / ultralytics / torch / lap | `3.13.15 / 8.4.145 / 2.11.0+cu128 / 0.5.13` |
| weights / hai tracker | `yolo26n.pt / "bytetrack.yaml" và "botsort-reid.yaml"` |
| conf / IoU / imgsz / classes | `0.25 / 0.7 / 960 / [2, 5, 7]` |
| device | `0` |

                    HOTA    DetA    AssA    LocA    IDF1    MOTA    MOTP      FP      FN    IDSW
------------------------------------------------------------------------------------------------
ban_vs_gold        0.803   0.783   0.826   0.873   0.962   0.921   0.857      44       1       0
bytetrack_vs_gold   0.709   0.649   0.776   0.846   0.875   0.749   0.823      88      54       2
reid_vs_gold       0.763   0.711   0.820   0.872   0.900   0.792   0.860      91      26       2
reid_vs_ban        0.737   0.677   0.804   0.880   0.868   0.732   0.870      93      71       1

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

`Với annotation hiện tại, MOTA = 0.921 thấp hơn IDF1 = 0.962. MOTA chủ yếu phạt FP, FN và IDSW theo từng frame; một lỗi đổi ID không bị phạt nặng như mất detection kéo dài. Vì vậy MOTA cao không đồng nghĩa identity hoàn hảo, và IDF1/AssA cần được đọc riêng để đánh giá giữ ID.`

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**

`So với ByteTrack, BoT-SORT + ReID tăng IDF1 từ 0.875 lên 0.900 và AssA từ 0.776 lên 0.820; IDSW vẫn bằng 2. ReID cải thiện association tổng thể nhưng không loại bỏ được mọi lần đổi ID. ByteTrack bị ID switch ở frame 59 (gold ID 4: model 14 -> 15) và frame 94 (gold ID 5: model 23 -> 32); ReID còn switch ở frame 87 (gold ID 5: 17 -> 18) và frame 113 (gold ID 6: 24 -> 31). Đây là so sánh giữa hai tracker implementation khác nhau, nên không cô lập causal effect riêng của ReID.`

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

`DetA tăng từ 0.649 lên 0.711, FN giảm 54 xuống 26 nhưng FP tăng nhẹ 88 lên 91; LocA cũng tăng 0.846 lên 0.872. Vì DetA và FN thay đổi rõ, ReID treatment không chỉ khác ở association mà còn có khác biệt về track coverage/detection output. Tuy vậy AssA và IDF1 cũng tăng, nên lỗi còn lại là kết hợp: detector/coverage gây FN, còn association vẫn gây các switch và fragmentation.`

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**

`Frame 87, gold ID 5: ReID đổi từ tracker ID 17 sang 18. Annotation của tôi vẫn giữ một ID 5 cho cùng xe qua đoạn này; vì vậy ở điểm chuyển ID, annotation giữ identity nhất quán hơn ReID. Đây là finding từ evaluator, không phải kết luận rằng toàn bộ track ReID sai.`

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**

`Frame 113, gold ID 6: ReID đổi từ tracker ID 24 sang 31. Đây là điểm cần xem lại annotation bằng hình ảnh vì có thể là occlusion hoặc association error. Tuy nhiên evaluator chỉ cho biết ID switch; chưa có ảnh/frame review trong repo để kết luận cần sửa annotation. Vì vậy hiện tại evidence nghiêng về lỗi association của model, không đủ căn cứ sửa nhãn.`

## 6. Nếu phải gán thêm 10 clip nữa

Bạn sẽ sửa gì trong `GUIDELINE_MINI.md`, và đổi gì trong quy trình làm việc của mình?

`Nếu phải gán thêm 10 clip, tôi sẽ bổ sung guideline theo hướng cụ thể hóa các trường hợp dễ gây sai lệch identity.`

## 7. Tệp đã nộp

- [x] `annotations/clip_01/gt.txt`
- [x] `annotations/clip_02/gt.txt`
- [ ] `evidence/pre-gold/clip_01/gt.txt` và `manifest.json` (Có gt.txt nhưng không có manifest.json)
- [x] `GUIDELINE_MINI.md` đã điền
- [x] `outputs/eval_vs_gold.json`
- [x] `outputs/model_bytetrack_clip_01.txt`
- [x] `outputs/model_reid_clip_01.txt`
- [x] `outputs/model_run_config.json`
- [x] `outputs/eval_bytetrack_vs_gold.json`, `outputs/eval_reid_vs_gold.json`, `outputs/eval_reid_vs_me.json`
- [x] `reports/review_partner.md`
- [x] `reports/REPORT.md` (file này)
