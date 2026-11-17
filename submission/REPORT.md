# Báo cáo Day 08 — 2A202603310 - Nguyễn Nhân Thành Long

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person / bicycle / motorbike / car / bus / truck | YOLO26m | person 0.35; bicycle 0.20; motorbike/car/bus/truck mặc định 0.30 | r01→r02: tăng conf chung 0.25→0.50 giảm box pilot 215→139 (76 box), nhưng sweep chọn theo lớp để giữ recall mục tiêu. r01/r02 là thử pilot; số sweep có 4 ảnh gold. |
| three_wheeler | YOLOE-26s, lớp/prompt `three_wheeler` | 0.15 | r03: 8 box trong pilot 8 ảnh; r06: 125 box trên 36 ảnh. Sweep: recall 7/7 = 100%, precision 7/(7+2) = 77.8% trên 7 vật gold. |

- Recipe gộp tái dựng từ `threshold_sweep.md`: `python scripts/merge_prelabels.py work/prelabel/y26_all/pre.coco.json work/prelabel/yoloe_all/pre.coco.json --conf 0.3 --thr "person=0.35,bicycle=0.2,three_wheeler=0.15" --out work/prelabel/merged_review/pre.coco.json`. `model_log.csv`/`run.json` không lưu lệnh merge thực tế; tài liệu sweep nêu ngưỡng mặc định 0.3 cho lớp không có ngưỡng riêng.
- **SAM 2.1** (`sam2.1_b.pt`): r04 tạo 16 mask/2 ảnh, nhưng chạy CPU mất 9.06 giây/ảnh. Chọn bản này cho thử mask vì đã có artifact; dữ liệu không có so sánh định lượng chất lượng/tốc độ với SAM 1.
- Đổi ngưỡng không sửa được lỗi phân loại, ví dụ tuk-tuk bị gọi `car`/`motorbike`; cần xác nhận hình thái ba bánh và đổi lớp thành `three_wheeler` khi phù hợp. Threshold sweep chỉ đo detection theo lớp, không giải quyết nhầm lớp.
- Ví dụ điểm cao nhưng sai: box `truck` conf 0.88 ở `img_001` không khớp box gold cùng lớp (IoU < 0.5); điểm thấp nhưng đúng: `motorbike` conf 0.14 ở `img_001` khớp gold IoU 0.62. Đây là đối chiếu thô từ `y26_all` với gold, trước ngưỡng gộp.
- Giới hạn: sweep chọn ngưỡng trên chính 4 ảnh gold (56 vật), nên precision/recall trên gold lạc quan; mẫu nhỏ, đặc biệt bus (1 vật) và truck (0 vật), không đủ để chốt ngưỡng riêng. Nguồn: `qc/threshold_sweep.md`, `qc/review_stats.md`.

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---:|---:|---|
| accepted | 357 | 357/504 = 70.8% | 504 box máy |
| edited | 48 | 48/504 = 9.5% | 504 box máy |
| deleted | 66 | 66/504 = 13.1% | 504 box máy |
| added | 49 | 49/454 = 10.8% | 454 đối tượng sau duyệt |

- Lớp máy tệ nhất theo số xóa là `three_wheeler`: 28/42 box máy của lớp bị xóa (66.7%; 42 = 6 accepted + 8 edited + 28 deleted). Điều này phù hợp với khó khăn phân biệt xe ba bánh và các lớp xe COCO; cần xem lại từng box, không quy hết lỗi cho ngưỡng.
- So với gold: precision máy 50/(50+19) = 72.5%; recall 50/56 = 89.3% (56 vật gold, 4 ảnh). Kết quả in-sample, khoảng tin cậy 95% lần lượt 61–82% và 79–95%; xem `qc/review_stats.md`.

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt đúng 10/12 = 83.3% bẫy; lọt 2/12: `wrong_class` ở `img_005` (person → bicycle) và `img_018` (bicycle → motorbike). Cả hai đều là lỗi lớp; có thể bước kiểm tra lớp (bước 3 trong checklist) chưa đủ hiệu quả, nhưng log không ghi bước nào đã bị bỏ qua nên đây là nhận định, không phải dữ kiện.
- Accepted 359/504 = 71.2% trong khi bắt bẫy 10/12 = 83.3% cho thấy phần lớn box được giữ, nhưng lỗi cài sẵn vẫn lọt. Tỉ lệ accepted không chứng minh đã quét độc lập ảnh trước khi xem nhãn máy; hai bẫy lọt cùng loại kiểm tra lớp nhấn mạnh cần rà soát nhãn riêng.

## 4. Máy có đáng dùng không?

- `time_log.csv`: gán tay 4 ảnh gold mất 15 phút = 3.75 phút/ảnh; review 24 ảnh mất 90 phút = 3.75 phút/ảnh. Thời gian trung bình/ảnh bằng nhau trong log này (chênh lệch 0%), nhưng so sánh có thiên lệch vì 4 ảnh gold được gán tay trước nên đã quen cảnh.
- Kết luận: pre-label có thể dùng làm điểm bắt đầu để review, nhưng theo log này chưa cho thấy tiết kiệm thời gian: gán tay 15/4 = 3.75 phút/ảnh, review 90/24 = 3.75 phút/ảnh. Cơ sở chất lượng: accepted 357/504 = 70.8%, edited 48/504 = 9.5%, deleted 66/504 = 13.1% và added 49/454 = 10.8%; lớp `three_wheeler` cần kiểm tra kỹ (28/42 box bị xóa). So sánh thời gian chỉ mang tính tham khảo do tập ảnh và mức độ quen cảnh khác nhau.
- Ba quy tắc cho lượt review tiếp theo (có thể kiểm tra bằng thao tác/log):
  1. Trước khi hiện pre-label, quét ảnh và ghi số lượng ước tính theo lớp cho đủ 24/24 ảnh.
  2. Với mọi box xe ba bánh, kiểm tra lớp `three_wheeler` và đối chiếu số bánh/hình dạng trước khi chấp nhận; ghi riêng số box đổi lớp.
  3. Sau mỗi 10 ảnh nghỉ 3–5 phút và ghi giờ bắt đầu/kết thúc vào time log.

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hóa? | Lỗi quan sát | File trong `masks/` |
|---|---|---:|---|---|---|
| img_013 | SAM 2.1 | 3 (2 dương, 1 âm) | Có: vùng mask phủ cả người lái; không thấy hàng hóa riêng | Dính người lái vào xe; biên/lỗ quanh khung xe chưa sạch | `tuktuk_img013_sam2.jpg` |
| img_004 | SAM 2.1 | 3 (2 dương, 1 âm) | Không thấy mask người/hàng hóa tách riêng | Mask xe còn răng cưa/thiếu vùng nhỏ ở phần bánh và gầm | `tuktuk_img004_sam2.jpg` |
| img_017 | SAM 2.1 | 4 (3 dương, 1 âm) | Không thấy mask người/hàng hóa tách riêng | Biên quanh mui, bánh và gầm chưa đều; cần sửa thủ công | `tuktuk_img017_sam2.jpg` |

- Đối chiếu guideline mục 4.2: quy tắc 1 (bám biên phần nhìn thấy) giúp đánh giá mask; quy tắc 4 (hai vật chạm nhau phải có polygon riêng, không chồng) còn thiếu trong kết quả `img_013` vì mask nhập người lái vào xe. Quy tắc 2 về lỗ trong khung xe cũng cần áp dụng nhất quán; vùng hở lớn nên tách khỏi mask, còn lỗ nhỏ có thể bỏ qua theo ngưỡng guideline.
- Interactor SAM trong CVAT: không có log xác nhận đã thử, nên chưa thể kết luận làm được hay lỗi gì. Offline có lưu overlay và điểm click trong các ảnh ở `masks/`; không đủ dữ liệu để so sánh trực tiếp thao tác với CVAT.

## 6. Calibration (Part 6)

- Có 3 quyết định trong `qc/calibration_decisions.csv`: `CAL-01`–`CAL-03`. Quan trọng nhất: `CAL-02`, hai bẫy sai lớp lọt ở `img_005` và `img_018` dẫn đến bước rà lớp xe riêng trước khi chấp nhận.
- Quy tắc được bổ sung trong `guideline_v1.1_calibrated.md`: rà phân lớp xe ba bánh và kiểm tra xe đạp/xe có động cơ; kính/cửa sổ cấu thành xe thuộc mask, người nhìn thấy vẫn là nhãn `person` riêng.
- Nguyên nhân gốc: chủ yếu là lỗi phân loại của mô hình và bước kiểm tra lớp chưa bắt được hai bẫy; `CAL-03` xử lý câu hỏi còn mở của guideline về mask kính. Nguồn: `calibration_decisions.csv`, `review_stats.md`.

## 7. QC và quyết định (Part 6)

- Mẫu gồm 8 ảnh: 3 ảnh `can_review` và 5 ảnh chọn ngẫu nhiên, seed 42; tổng 169 đối tượng. Người kiểm: Nguyễn Nhân Thành Long. Nguồn: `qc_report_auto.md`, `qc_sample.csv`.
- Phiếu hiện tại ghi nhận 0/169 đối tượng lỗi = 0% và 0/8 ảnh có lỗi = 0%. Đây là kết quả theo các dòng `khong` trong `qc_review.csv`; nên đối chiếu lại các ảnh trước khi chốt vì QC là đánh giá bằng mắt.
- Top lỗi: không ghi nhận lỗi trong mẫu. Các lỗi sai lớp `CAL-02` là bẫy bị lọt ở lượt review trước, không phải lỗi được ghi nhận trong mẫu QC hiện tại.
- Giới hạn mẫu nhỏ: với 0/8 ảnh lỗi, cận trên Wilson 95% là 32%; mẫu này không đủ chứng minh tỷ lệ lỗi thật đạt ngưỡng 2%.
- Script kiểm được cấu trúc và thống kê nhãn, nhưng không xác nhận box có ôm sát vật, lỗi mà cả người và máy cùng bỏ sót, hoặc người duyệt có thực sự làm theo checklist.

**Quyết định: Release**, ngày 2026-11-17, theo kết quả QC hiện tại 0/8 ảnh lỗi (ngưỡng ≤2%) và bắt bẫy 10/12 = 83.3% (ngưỡng ≥80%). Đây là quyết định có giới hạn: 8 ảnh là mẫu nhỏ, cận trên Wilson 95% là 32%; kết quả không chứng minh tỷ lệ lỗi toàn bộ lô dưới 2%.

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài:** có dùng Codex để đọc/tóm tắt log, đối chiếu số liệu và soạn nháp báo cáo.
- Gold có bị nhiễm không: không.
- Máy/GPU/CVAT: log chạy YOLO ghi thiết bị CUDA; lần chạy SAM r04 dùng CPU. Model GPU, phiên bản CVAT: 2.81.0.
- Cache `data/cache/`: các file cache YOLO26m và YOLOE-26s.
- Lỗi/bug script: không có bug được ghi nhận trong hồ sơ.
- Lần sau: ghi đủ thời gian từng giai đoạn, rà nhãn lớp bằng checklist độc lập và ghi rõ trạng thái sử dụng cache/GPU để báo cáo có thể kiểm chứng.
