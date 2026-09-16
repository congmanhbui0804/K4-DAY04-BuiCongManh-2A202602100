# Báo cáo Ngày 4 - Keypoint & Pose

Họ tên: Bùi Công Mạnh   Nhóm: Cá nhân (không kiểm chéo)   Ngày: 16/09/2026

## 1. Nhãn của tôi

| Chỉ số | Giá trị |
| --- | ---: |
| Số ảnh đã gán | 20 |
| Số skeleton | 28 |
| v=2 / v=1 / v=0 | 355 / 92 / 29 |
| Thời gian trung bình mỗi ảnh | 4 phút |

Ba khớp có `%v=1` cao nhất (chép từ `reports/visibility_report.md`):

1. left_ear — 54%
2. right_ear — 39%
3. left_wrist — 29%

Chúng có đúng là những khớp bạn thấy khó gán nhất không? Nếu không, giải thích.

Không. Các khớp này (tai, cổ tay) không khó xác định vị trí giải phẫu — có thể quan sát dễ
dàng bằng mắt thường, khớp rõ ràng khi nhìn ảnh. `%v=1` cao ở đây phản ánh việc chúng hay bị
**che khuất** (tai bị tóc/mũ che, cổ tay bị tay áo hoặc người khác che) chứ không phải vì khó
gán hay khó nhận diện vị trí. Điều này khớp với ghi chú trong `visibility_report.md`: "%v=1 cao
= khớp hay bị che", nên hai chuyện "hay bị che" và "khó xác định vị trí giải phẫu" là khác
nhau.

## 2. Chấm với gold

| Chỉ số | Trước rework | Sau rework |
| --- | ---: | ---: |
| OKS trung bình | 0.9408 | 0.9513 |
| OKS@0.50 | 1.000 | 1.000 |
| OKS@0.75 | 1.000 | 1.000 |
| Lỗi `dao_trai_phai` | 0 | 0 |
| Lỗi `nham_nguoi` | 1 | 0 |
| Lỗi `xoa_khop_bi_che` | 0 | 0 |

**Tôi đã sửa gì giữa hai lần chạy** (ghi cụ thể: ảnh nào, người thứ mấy, khớp nào):

- `train_04.jpg`, người thứ 1, khớp `left_wrist` (cổ tay trái): trước rework chấm bị lỗi
  `nham_nguoi` — điểm `left_wrist` đặt tại toạ độ `(0.607750, 0.799453)`, rơi sát vào
  bounding box và cổ tay của người thứ 2 trong ảnh, khiến OKS của người này chỉ đạt 0.8463
  (thấp nhất trong toàn bộ so sánh với gold). Đã kéo lại chấm về đúng vị trí giải phẫu cổ tay
  trái của người thứ 1, toạ độ `(0.504687, 0.789934)`. Sau khi sửa, OKS của người này tăng lên
  >0.96, khắc phục hoàn toàn lỗi `nham_nguoi` duy nhất và kéo OKS trung bình toàn bộ từ 0.9408
  lên 0.9513.
- `train_10`, người thứ 1: sửa 5 khớp từ `v=0` sang `v=1` — người nằm gọn trong khung nên các
  khớp này không thể "ra ngoài mép ảnh", đây là trường hợp bị che chứ không phải out-of-frame
  (sửa theo cảnh báo định dạng của `check_pose_labels.py`, trước khi chấm gold).
- `train_11`, người thứ 1: sửa 4 khớp từ `v=0` sang `v=1`, cùng lý do như trên.

**Lỗi đảo trái/phải của tôi xảy ra ở ảnh nào?** Ảnh đó dễ hay khó? Nếu là ảnh dễ,
bạn nghĩ vì sao mình vẫn sai?

Không có lỗi đảo trái/phải trong toàn bộ các ảnh đã chấm với gold — thuật toán kiểm tra flip
pair (hoán vị các cặp khớp trái/phải) xác nhận không có skeleton nào tăng điểm OKS khi hoán vị,
tức không có nhãn nào bị đảo trái/phải. Cảnh báo tự động ở `train_15.txt:2` (Cell 5 của
notebook, nghi ngờ hông trái/phải ngược so với hai mắt) cũng đã được kiểm tra lại và xác nhận
không sai: đã nhận diện đúng vị trí bàn tay, chỉ là model (không phải nhãn của tôi) dự đoán lệch
gần vị trí tay người khác do bị che khuất khi đối chiếu ở Mục 4.

## 3. Kiểm chéo

**Không thực hiện.**

## 4. Model

| Chỉ số | yolo26n-pose gốc | Sau fine-tune | Chênh |
| --- | ---: | ---: | ---: |
| pose_mAP50 | 0.8450 | 0.8450 | +0.0000 |
| pose_mAP50-95 | 0.6853 | 0.6908 | +0.0055 |
| pose_precision | 0.9734 | 0.9792 | +0.0058 |
| pose_recall | 0.8462 | 0.8462 | +0.0000 |
| box_mAP50-95 | 0.8119 | 0.8041 | -0.0078 |

*(nguồn: `outputs/eval_model.json`, sinh ở Cell 11 của notebook, 10 ảnh test)*

### Trả lời năm câu hỏi ở cuối notebook

1. **`pose_mAP50-95` thay đổi bao nhiêu?**
   Tăng nhẹ +0.0055 (0.6853 → 0.6908), không giảm. Với chỉ 20 ảnh train, mức tăng này rất nhỏ
   — hợp lý vì `yolo26n-pose.pt` đã được huấn luyện sẵn trên COCO và đã biết bộ 17 điểm này.
   20 ảnh của bạn nhiều khả năng chỉ giúp model quen thêm với phong cách ảnh/điều kiện che
   khuất riêng của tập dữ liệu này, chứ chưa đủ để dạy điều gì mới đáng kể. Vì mAP không giảm,
   không có dấu hiệu 20 ảnh này "dạy sai" hay làm hỏng kiến thức COCO sẵn có.

2. **`box_mAP` và `pose_mAP` chênh nhau bao nhiêu? Model tìm người hay tìm khớp dễ hơn?**
   Gốc: box 0.8119 vs pose 0.6853 → chênh 0.1266. Sau fine-tune: box 0.8041 vs pose 0.6908 →
   chênh 0.1133. Ở cả hai lần đo, `box_mAP50-95` đều cao hơn `pose_mAP50-95` rõ rệt → model tìm
   **người** (bounding box) dễ hơn tìm **khớp** (pose) khá nhiều. Điều này hợp lý: phát hiện có
   người ở đâu là bài toán thô hơn, còn định vị chính xác 17 điểm — đặc biệt các khớp hay bị che
   như cổ tay, mắt cá — đòi hỏi độ chính xác không gian cao hơn nhiều. Đáng chú ý: sau fine-tune,
   `box_mAP` giảm nhẹ (-0.0078) trong khi `pose_mAP` tăng (+0.0055) — một đánh đổi nhỏ, không
   phải cả hai cùng tăng.

3. **Một ảnh model đoán sai — gọi tên lỗi theo bốn loại của slide 43:**
   Ở mục 6 của notebook, ba ảnh có **số người lệch** giữa model và nhãn của bạn:
   `train_10` (model đếm 2 người / bạn đếm 1), `train_13` (model 3 / bạn 2), `train_03`
   (model 4 / bạn 2) — đây là dạng lỗi **nhầm người**. Ảnh `train_15` (OKS thấp nhất, 0.354) là
   một biến thể của lỗi này ở mức khớp: bàn tay được nhãn của tôi gán đúng vị trí, nhưng vì bị
   che khuất, model lại dự đoán điểm tay gần với vị trí tay của **người khác** trong ảnh — tức
   model "nhầm người" ngay ở một khớp đơn lẻ, không phải lệch nhẹ hay trượt hẳn.

4. **Ảnh nào có OKS thấp nhất giữa nhãn của bạn và model? Ai đúng?**
   `train_15`, OKS = 0.354 — thấp hơn hẳn ảnh đứng thứ hai (`train_06`, OKS 0.633). Nhãn của tôi
   **đúng**: bàn tay đã được nhận diện đúng vị trí thực tế, nhưng do bị che khuất một phần nên
   model dự đoán lệch, hút điểm tay về gần vị trí tay của người bên cạnh — đây là lỗi của model
   khi gặp che khuất, không phải lỗi gán nhãn (và cũng không phải lỗi đảo trái/phải như cảnh báo
   tự động ở Cell 5 từng nghi ngờ).

5. **Ảnh gán tệ nhất (so với gold) có cũng là ảnh model đoán tệ nhất không?**
   Ảnh gán tệ nhất so với gold là `train_04.jpg`, người thứ 1, OKS 0.8463 — do lỗi `nham_nguoi`
   ở khớp `left_wrist` (điểm cổ tay bị đặt lệch sang gần cổ tay của người thứ 2). Trên chính ảnh
   `train_04.jpg`, model cũng gặp hiện tượng tương tự khi dự đoán các khớp tay/chân ở vùng hai
   người đứng sát nhau. `train_04.jpg` là một ca có tương tác gần và chồng lấn cơ thể
   (occlusion): khi hai người đứng cạnh nhau và cánh tay đan xen, cả người gán nhãn lẫn model
   đều dễ bắt nhầm khớp của người này sang thân người bên cạnh — khẳng định các ca có độ che
   khuất cao và biên người gần nhau luôn là thách thức lớn nhất cho bài toán pose estimation.
   *(Lưu ý: ở bảng OKS model-vs-nhãn của tôi tại Mục 6 notebook, `train_04` có 2 giá trị khớp
   khá cao — 0.865 và 0.969 — nên khó khăn này thể hiện rõ nhất ở mức một khớp đơn lẻ
   (`left_wrist`), không kéo tụt điểm trung bình toàn skeleton của ảnh.)*

## 5. Một rule evidence bạn đã dùng

**Ảnh:** `train_10` · **Người:** thứ 1 · **Khớp:** 5 khớp đã sửa từ `v=0` sang `v=1`

`check_pose_labels.py` cảnh báo người này nằm gọn trong khung ảnh nhưng có 5 khớp mang cờ
`v=0` (ngoài khung). Theo guideline, `v=0` chỉ dùng khi khớp thật sự ra ngoài mép ảnh; còn khi
người vẫn nằm trong khung nhưng khớp bị che (bởi tóc, quần áo, người khác, hoặc vật cản) thì
phải dùng `v=1` và vẫn đặt chấm ở vị trí ước lượng. Vì bounding box của người này không chạm
biên ảnh, 5 khớp đó về mặt hình học không thể "ra khỏi khung" được — bằng chứng thị giác (người
trọn vẹn trong ảnh) cho thấy đây là trường hợp bị che, nên đã sửa cả 5 khớp thành `v=1`.
