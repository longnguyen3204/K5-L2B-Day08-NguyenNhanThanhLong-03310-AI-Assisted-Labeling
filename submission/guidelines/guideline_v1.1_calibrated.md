# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603310 - Nguyễn Nhân Thành Long

> Bổ sung quy tắc vận hành so với guideline v1.0; các ảnh minh họa và quyết định tương ứng nằm trong `qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 5 | Kính chắn gió/cửa sổ có thuộc mask xe không? | Kính và khung cửa gắn liền xe thuộc mask phương tiện. Người nhìn xuyên qua kính hoặc qua cạnh hở vẫn được gán `person` riêng; không gộp người vào mask xe. | `img_013` | `CAL-03` |

Các câu hỏi mở còn lại (xe đạp điện, cụm xe máy gần camera, hàng nhô ra, ngưỡng nhìn thấy người, box chồng trong ảnh đêm và ngưỡng conf) chưa được quyết định ở bản này vì chưa có ca đủ rõ để chốt.

## 2. Ca khó mới

| Ảnh | Tình huống | Máy/người duyệt làm gì | Quyết định | Quy tắc bổ sung |
|---|---|---|---|---|
| `img_003`, `img_013`, `img_035` | Xe ba bánh địa phương có thể bị nhầm với xe COCO thông thường | Review stats ghi nhận nhiều box `three_wheeler` bị sửa/xóa; cần xác nhận từng lớp theo ảnh | Xe ba bánh địa phương là `three_wheeler`, kể cả có mui hoặc chở hàng; không dùng điểm conf để chốt lớp | Mục 2: đối chiếu cấu tạo và số bánh nhìn thấy; nếu không rõ thì đánh dấu `can_review` |
| `img_005`, `img_018` | Bẫy sai lớp lọt qua review | `person`→`bicycle` và `bicycle`→`motorbike` bị bỏ sót | Rà lớp phương tiện riêng; phân biệt xe đạp đạp chân với xe có động cơ; người ngồi/dắt xe là `person` riêng | Mục 2 và checklist review: sau bước quét ảnh, kiểm tra lớp từng phương tiện trước khi accept |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca, `case_id`) |
|---|---|---|
| v1.0 | Bản guideline ban đầu | — |
| v1.1 | Làm rõ phân lớp xe ba bánh, thêm lượt rà lớp để xử lý bẫy sai lớp lọt, chốt kính/cửa sổ thuộc mask xe và tách người thành đối tượng riêng | `CAL-01`, `CAL-02`, `CAL-03` |
