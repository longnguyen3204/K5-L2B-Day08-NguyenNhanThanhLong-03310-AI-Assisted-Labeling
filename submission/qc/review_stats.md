# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 504; đối tượng sau duyệt: 454; ngưỡng accepted: IoU >= 0.9
- Đã **loại 20 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 357 | 357/504 = 70.8% | 504 box máy |
| edited (sửa) | 48 | 48/504 = 9.5% | 504 box máy |
| deleted (xoá) | 66 | 66/504 = 13.1% | 504 box máy |
| added (vẽ thêm) | 49 | 49/454 = 10.8% | 454 đối tượng sau duyệt |

Trong số deleted + added, **19 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 30.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 204 | 13 | 11 | 11 |
| bicycle | 5 | 1 | 3 | 5 |
| motorbike | 95 | 23 | 12 | 12 |
| car | 45 | 3 | 5 | 9 |
| bus | 2 | 0 | 1 | 3 |
| truck | 0 | 0 | 6 | 3 |
| three_wheeler | 6 | 8 | 28 | 6 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 56 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 50 | 19 | 6 | 72.5% (KTC 95%: 61–82%) | 89.3% (KTC 95%: 79–95%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 10/12 = 83.3% (KTC 95%: 55–95%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - extra: 3/3
  - wrong_class: 1/3
  - missing: 3/3
  - loose: 3/3
  - lọt: img_005.jpg (wrong_class), img_018.jpg (wrong_class)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
