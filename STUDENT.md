# Thông tin học viên

- Họ và tên: Nguyễn Nhân Thành Long
- MSHV: 2A202603310
- Lớp: L2B
- Tài khoản GitHub: longnguyen3204
- Repo bài nộp (URL): https://github.com/longnguyen3204/K5-L2B-Day08-NguyenNhanThanhLong-03310-AI-Assisted-Labeling
- Commit chốt bài (hash): chưa ghi nhận
- Máy: có log chạy YOLO trên CUDA; Windows 11. SAM r04 chạy CPU.
- CVAT (Docker): 2.81.0

## Kết quả tóm tắt

- Máy: accepted 357/504 = 70.8%, edited 48/504 = 9.5%, deleted 66/504 = 13.1% (mẫu số: 504 box máy); added 49/454 = 10.8% (mẫu số: 454 đối tượng sau duyệt).
- So với gold: precision máy 72.5% (50 TP / 69 dự đoán), recall máy 89.3% (50/56 vật gold).
- Bắt bẫy: 10/12 = 83.3%; còn lọt hai bẫy sai lớp ở `img_005` và `img_018`.
- Thời gian: gán tay 15/4 = 3.75 phút/ảnh; review có pre-label 90/24 = 3.75 phút/ảnh.
- Lỗi QC: 0/8 ảnh = 0%; 0/169 đối tượng = 0%, theo phiếu QC hiện tại. Cận trên Wilson 95% cho tỷ lệ ảnh lỗi là 32% do mẫu nhỏ.
- **Quyết định:** Release theo kết quả QC hiện tại và ngưỡng bắt bẫy; kết quả 8 ảnh không chứng minh được tỷ lệ lỗi toàn lô dưới 2%.
