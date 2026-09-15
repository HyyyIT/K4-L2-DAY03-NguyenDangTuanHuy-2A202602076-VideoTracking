# Mini annotation guideline — Ngày 3 (tracking)

> Điền file này **trong lúc gán nhãn**, không phải sau khi xong. Mỗi lần bạn dừng
> lại nghĩ "cái này tính sao nhỉ?" thì đó là một dòng phải ghi vào đây.
>
> Đây là tài liệu mà người gán nhãn tiếp theo sẽ đọc để làm giống bạn. Nếu hai
> người trong nhóm gán khác nhau, gần như luôn là vì file này chưa nói rõ — chứ
> không phải vì ai kém.

Nhóm / tên: `Nguyễn Đăng Tuấn Huy`
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

Bổ sung của nhóm (nếu có): `...`

## 2. Luật ID — phần quan trọng nhất

| Tình huống | Luật của nhóm | Vì sao |
| --- | --- | --- |
| Xe bị che một phần rồi hiện lại | giữ nguyên ID nếu bị che **dưới 25 frame** (mặc định của lab: 25 frame = 2 giây @ 12.5 fps) | `Nó vẫn xuất hiện sau 25 frame thì vẫn nên giữ ID vì nó vẫn nằm trong tầm của mình` |
| Xe bị che lâu hơn ngưỡng trên | `Ta nên chọn ID mới` | `thời gian mất dấu dài, không đủ chắc chắn đó là cùng một xe` |
| Xe rời khung hình rồi quay lại | mặc định: **track mới** | `không biết xe quay lại có phải đúng đối tượng cũ hay không` |
| Hai xe cắt nhau / chồng lên nhau | `giữ ID dựa trên vị trí, hướng di chuyển và đặc điểm hình dạng; không đổi ID chỉ vì hai xe chồng lên nhau` | `tránh hoán đổi ID giữa hai xe sau khi chúng tách ra` |

## 3. Luật bbox

| Tình huống | Luật của nhóm |
| --- | --- |
| Xe bị cắt bởi rìa ảnh | bbox chạm đúng rìa, không đoán phần ngoài ảnh |
| Xe bị xe khác che một phần | bbox ôm phần **nhìn thấy được** |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | bắt đầu track từ frame đầu tiên xác định được là xe bốn bánh; ngưỡng nhóm chọn: `...` |
| Xe đang đỗ, không di chuyển | `...` |
| Keyframe đặt dày ở đâu | `...` |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

Ghi **frame cụ thể** và **ID cụ thể**, không ghi chung chung.

### Ca 1
- Clip / frame / ID: `01/111/8`
- Tình huống: `Xe buýt đi vào giữa khung hình`
- Quyết định: `Ta phải điều chỉnh bbox sát xe`
- Lý do: `tránh bbox lệch khỏi xe`

### Ca 2
- Clip / frame / ID: `01/137/7`
- Tình huống: `xe từ ngoài khung hình vào`
- Quyết định: `vẽ bbox bám sát mép hình`
- Lý do: `Để bbox track theo xe`

### Ca 3
- Clip / frame / ID: `02/8/03`
- Tình huống: `xe đi ra mép khung hình`
- Quyết định: `bbox phải ẩn`
- Lý do: `khi xe đã ra khung hình`

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

Luật nào trong file này hoá ra còn thiếu hoặc còn mơ hồ? Viết lại cho rõ:

- `Nếu xe bị che dưới 25 frame thì giữ nguyên ID; từ 25 frame trở lên hoặc không chắc đó là cùng xe thì tạo ID mới.`
- `Khi xe đi vào, đi ra hoặc bị che, bbox chỉ ôm phần nhìn thấy; cần kiểm tra lại frame đầu, frame cuối và frame giữa để tránh bbox bị lệch.`
