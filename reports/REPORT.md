# Báo cáo Ngày 3 — Tracking Annotation

Chép file này thành `reports/REPORT.md` rồi điền. Giữ nguyên các tiêu đề.

Họ tên / nhóm: `Nguyễn Đăng Tuấn Huy`
Ngày: `15/09/2026`

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT|
| Thời gian gán `clip_02` (warm-up) | `15` phút |
| Thời gian gán `clip_01` | `30` phút |
| Số track đã vẽ trong `clip_01` | `8` |
| Số keyframe trung bình mỗi track | `52` |

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:

1. `1 xe ô tô đi qua nhiều frame` : Phải tracking theo xe điểm đầu frame và cuối frame liên tục
2. `1 xe ô tô đi qua nhiều frame`: Ở giữa frame thường track nó không theo kịp mình phải chỉnh
3. `Đi vào đầu mép hình đầu frame và cuối frame` : phải chỉnh bbox nhỏ dần khi vào đầu khung hình và cuối khung hình

## 2. Tự kiểm và kiểm chéo

Ba lượt tua bắt được gì (lượt 1 nhìn ID, lượt 2 frame đầu/cuối, lượt 3 frame giữa):

- Lượt 1: `ID ổn`
- Lượt 2: `Frame đầu/cuối cần chỉnh bbox cho sát mép object`
- Lượt 3: `frame giữa điều chỉnh bbox ôm trọn object`

Kiểm chéo với: `...`. Chi tiết ở `reports/review_partner.md`.
Số lỗi bạn tìm được trong bản của bạn ấy: `...`. Số lỗi bạn ấy tìm được trong bản của bạn: `3`.

Ca nào hai người quyết khác nhau, và luật nào còn thiếu trong `GUIDELINE_MINI.md`? Làm cá nhân

## 3. Pre-gold lock và chấm trước/sau rework

| Evidence | Giá trị |
| --- | --- |
| SHA-256 từ `evidence/pre-gold/clip_01/manifest.json` | `Chưa xác định: manifest.json không có trong workspace hiện tại` |
| Thời điểm khóa | `Chưa xác định từ file hiện có` |
| Số row / frame / track trước khi mở reference | `573 row / 190 frame / 8 track` |

| | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Bản pre-gold | Chưa có file eval pre-gold | Chưa có file eval pre-gold | Chưa có file eval pre-gold | Chưa có file eval pre-gold | Chưa có file eval pre-gold | Chưa có file eval pre-gold | Chưa có file eval pre-gold | Chưa có file eval pre-gold | Chưa có file eval pre-gold | Chưa có file eval pre-gold |
| Sau rework | 0.8283 | 0.8089 | 0.8522 | 0.8909 | 0.9605 | 0.9232 | 0.8804 | 6 | 38 | 0 |

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **có**

Sau khi đọc danh sách lỗi, bạn đã sửa cụ thể những gì? Ghi theo frame và ID:

| Loại lỗi | Frame | ID | Đã sửa thế nào |
| --- | --- | --- | --- |
| Bbox chưa sát vật thể | Frame đầu/cuối (chưa lưu số cụ thể) | Các ID tương ứng | Chỉnh bbox bám phần xe nhìn thấy, nhất là khi xe đi vào hoặc rời mép ảnh |
| Bbox lệch ở frame giữa | Frame giữa (chưa lưu số cụ thể) | Các ID tương ứng | Chỉnh keyframe để bbox ôm trọn xe và để nội suy bám đúng chuyển động |
| Thiếu nhật ký rework | Chưa lưu số frame | - | Bổ sung quy trình ghi frame/ID ngay khi sửa; không khẳng định thêm lỗi khi chưa có evidence |

## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ `outputs/model_run_config.json`:

| Mục | Giá trị |
| --- | --- |
| Python / ultralytics / torch / lap | `3.13.15 / 8.4.145 / 2.11.0+cu128 / 0.5.13` |
| weights / hai tracker | `yolo26n.pt / bytetrack.yaml / configs/trackers/botsort-reid.yaml` |
| conf / IoU / imgsz / classes | `0.25 / 0.70 / 960 / [2, 5, 7]` |
| device | `0` (GPU) |

| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| bạn vs gold | 0.8283 | 0.8089 | 0.8522 | 0.8909 | 0.9605 | 0.9232 | 0.8804 | 6 | 38 | 0 |
| ByteTrack control vs gold | 0.7085 | 0.6487 | 0.7761 | 0.8463 | 0.8746 | 0.7487 | 0.8226 | 88 | 54 | 2 |
| BoT-SORT + ReID vs gold | 0.7635 | 0.7110 | 0.8204 | 0.8721 | 0.9001 | 0.7923 | 0.8595 | 91 | 26 | 2 |
| ReID vs bạn | 0.7838 | 0.7258 | 0.8470 | 0.8913 | 0.9042 | 0.7911 | 0.8806 | 105 | 8 | 0 |

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

`MOTA = 0.9232 thấp hơn IDF1 = 0.9605. Nhãn của tôi ít lỗi và không bị đổi ID (IDSW = 0). MOTA chủ yếu tính FP, FN và IDSW nên không phạt mạnh lỗi ID bằng IDF1.`

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**

`ReID tốt hơn ByteTrack: IDF1 tăng từ 0.8746 lên 0.9001, AssA tăng từ 0.7761 lên 0.8204. IDSW vẫn là 2. Ở khoảng frame 87 và 113, ReID vẫn còn bị đứt track. Tuy nhiên đây chưa phải kết luận riêng về ReID vì hai tracker dùng implementation khác nhau.`

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

`DetA tăng từ 0.6487 lên 0.7110, FN giảm từ 54 xuống 26 nhưng FP tăng từ 88 lên 91. ReID giúp bắt và nối track tốt hơn, nhưng detector vẫn còn bắt nhầm một số vật thể.`

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**

`Khoảng frame 16-116, ReID tạo track 7 nhưng không khớp với track nào trong reference. Đây là ghost track, nên nhãn của tôi không cần thêm đối tượng này.`

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**

`Tôi sẽ xem lại frame 6 và các đoạn 104-116, 158-178. Tuy nhiên ReID có 105 FP và nhiều ghost track, nên khả năng cao model sai hơn là annotation sai. Tôi chưa sửa nhãn chỉ dựa vào dự đoán của model.`

## 6. Nếu phải gán thêm 10 clip nữa

Bạn sẽ sửa gì trong `GUIDELINE_MINI.md`, và đổi gì trong quy trình làm việc của mình?

`Tôi sẽ bổ sung rõ quy tắc giữ ID khi xe bị che, xe ra khỏi khung hình và hai xe chồng lên nhau. Khi gán nhãn, tôi sẽ kiểm tra lần lượt ID, bbox đầu/cuối và bbox ở giữa video, đồng thời ghi lại frame và ID của các lỗi đã sửa.`

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
- [ ] `reports/review_partner.md`
- [x] `reports/REPORT.md` (file này)
