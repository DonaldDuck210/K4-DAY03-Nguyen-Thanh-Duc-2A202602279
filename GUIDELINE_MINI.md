# Mini annotation guideline — Ngày 3 (tracking)

> Điền file này **trong lúc gán nhãn**, không phải sau khi xong. Mỗi lần bạn dừng
> lại nghĩ "cái này tính sao nhỉ?" thì đó là một dòng phải ghi vào đây.
>
> Đây là tài liệu mà người gán nhãn tiếp theo sẽ đọc để làm giống bạn. Nếu hai
> người trong nhóm gán khác nhau, gần như luôn là vì file này chưa nói rõ — chứ
> không phải vì ai kém.

Solo / tên: `Nguyễn Thanh Đức`
Clip: `clip_01`, `clip_02`

---

## 1. Phạm vi: gán cái gì, không gán cái gì

Một lớp duy nhất: **`vehicle`** — xe bốn bánh (xe con, van, xe buýt, xe tải).

| Gán | Không gán |
| --- | --- |
| xe con, SUV, taxi, xe bán tải | người đi bộ |
| van, minivan | xe đạp |
| xe buýt, minibus | **xe máy / mô tô** |
| xe tải, xe đầu kéo | xe trong ảnh quảng cáo, trong gương, dưới bóng nước |

Bổ sung của nhóm (nếu có): chỉ gán xe bốn bánh có thể xác định từ hình ảnh; không suy đoán xe bị che hoàn toàn hoặc chỉ còn một vùng không đủ nhận dạng.

## 2. Luật ID — phần quan trọng nhất

| Tình huống | Luật của nhóm | Vì sao |
| --- | --- | --- |
| Xe bị che một phần rồi hiện lại | giữ nguyên ID nếu bị che **dưới 25 frame** và còn đủ dấu hiệu liên tục về vị trí, hướng di chuyển hoặc hình dáng | tránh tách một xe thành nhiều ID khi occlusion ngắn |
| Xe bị che lâu hơn ngưỡng trên | dùng track mới khi không còn căn cứ chắc chắn để nối với ID cũ; không nối chỉ vì xe xuất hiện gần vị trí cũ | giảm nguy cơ gộp nhầm hai xe giống nhau |
| Xe rời khung hình rồi quay lại | mặc định: **track mới** | sau khi ra khỏi khung không còn đủ bằng chứng để đảm bảo đó là cùng xe |
| Hai xe cắt nhau / chồng lên nhau | giữ ID theo quỹ đạo trước khi cắt; đối chiếu vị trí, hướng và đặc điểm xe trước/sau giao nhau | không đổi ID chỉ vì bbox chồng lấn trong vài frame |

## 3. Luật bbox

| Tình huống | Luật của nhóm |
| --- | --- |
| Xe bị cắt bởi rìa ảnh | bbox chạm đúng rìa, không đoán phần ngoài ảnh |
| Xe bị xe khác che một phần | bbox ôm phần **nhìn thấy được** |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | bắt đầu track từ frame đầu tiên xác định được là xe bốn bánh và bbox còn ít nhất khoảng **15 x 15 pixel**; nếu chưa chắc thì chờ frame rõ hơn |
| Xe đang đỗ, không di chuyển | vẫn giữ bbox ở các frame xe còn nhìn thấy; đặt `outside` ngay khi xe rời khung hoặc bị che hoàn toàn lâu hơn 25 frame |
| Keyframe đặt dày ở đâu | thêm keyframe mỗi 5-10 frame ở đoạn xe nhanh, đổi hướng, bị che hoặc cắt nhau; đoạn ổn định có thể giãn 15-20 frame |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

Ghi **frame cụ thể** và **ID cụ thể**, không ghi chung chung.

### Ca 1
- Clip / frame / ID: `clip_01 / 62-78 / ID 5`
- Tình huống: xe mới xuất hiện nhưng còn khó xác định và bbox bị evaluator xem là xuất hiện sớm hơn track tham chiếu.
- Quyết định: chỉ giữ track khi nhận dạng rõ xe bốn bánh; với bản hiện tại cần đặt `outside` trước frame 79 nếu chưa đủ bằng chứng ở các frame 62-78.
- Lý do: tránh bbox treo và tránh bắt đầu track từ vùng mờ/che khuất.

### Ca 2
- Clip / frame / ID: `clip_01 / 81-92 / ID 5`
- Tình huống: xe chuyển động nhanh, bbox nội suy giữa các keyframe bị lệch khỏi phần xe nhìn thấy.
- Quyết định: thêm keyframe dày hơn và chỉnh bbox theo phần nhìn thấy, không kéo theo quỹ đạo cũ một cách máy móc.
- Lý do: evaluator ghi nhận IoU thấp nhất quanh frame 83-92; hình học bbox cần ưu tiên hơn việc giữ ít keyframe.

### Ca 3
- Clip / frame / ID: `clip_01 / 149-151 / ID 4`
- Tình huống: xe gần rời khỏi khung nhưng track vẫn còn bbox thêm vài frame.
- Quyết định: đặt `outside` tại frame cuối cùng xe còn nhìn thấy, không để bbox treo sau đó.
- Lý do: evaluator phát hiện ID 4 còn bbox sau khi track tham chiếu kết thúc; biên track phải bám hình ảnh thực tế.

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

Luật nào trong file này hoá ra còn thiếu hoặc còn mơ hồ? Viết lại cho rõ:

- Khi xe vừa xuất hiện hoặc bị che, phải ghi rõ frame bắt đầu/kết thúc dựa trên khả năng nhận dạng thực tế; không kéo track qua vùng không chắc chắn.
- Ở đoạn chuyển động nhanh hoặc che khuất, tăng mật độ keyframe và kiểm tra lại cả biên `outside` lẫn hình học bbox sau khi export.
