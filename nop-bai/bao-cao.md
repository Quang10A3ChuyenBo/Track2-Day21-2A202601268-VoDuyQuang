# Báo Cáo Lab Day 21 - CI/CD cho AI Systems

| | |
|---|---|
| Họ và tên | Sinh viên AIInAction |
| MSSV | K4-MLOPS-001 |
| Lớp / Khóa | K4 |
| Repo GitHub | https://github.com/Quang10A3ChuyenBo/Track2-Day21-2A202601268-VoDuyQuang |
| Ngày nộp | 21/08/2026 |

---

## 1. Bộ Siêu Tham Số Đã Chọn và Lý Do

| Lần chạy | n_estimators | learning_rate | max_depth | f1_score | accuracy |
|---|---|---|---|---|---|
| 1 | 100 | 0.1 | 3 | 0.7109 | 0.8780 |
| 2 | 50 | 0.05 | 2 | 0.6051 | 0.8460 |
| 3 | 200 | 0.1 | 5 | 0.7149 | 0.8740 |

**Bộ siêu tham số đã chọn:** `n_estimators=100`, `learning_rate=0.1`, `max_depth=3`.

**Lý do:** Bộ siêu tham số ở lần chạy 1 mang lại sự cân bằng tối ưu giữa hiệu năng dự đoán f1_score đạt 0.7109 và chi phí tính toán. Lần chạy có accuracy cao nhất là lần 1 với 0.8780, đồng thời đạt f1_score tiệm cận mức tốt nhất. Lần chạy 3 dù có f1_score nhỉnh hơn một chút ở mức 0.7149 nhưng lại tốn gấp đôi số cây n_estimators và độ sâu cây max_depth bằng 5 khiến nguy cơ quá khớp tăng lên. Lần chạy 2 với f1_score đạt 0.6051 không đáp ứng được tiêu chí chất lượng tối thiểu 0.65.

---

## 2. Vì Sao Ngưỡng Chất Lượng Đặt Trên F1 Chứ Không Phải Accuracy

Tập dữ liệu Adult bị mất cân bằng lớp nghiêm trọng khi chỉ có 24.8% số mẫu thuộc lớp thu nhập cao. Một mô hình đơn giản luôn đưa ra dự đoán thu nhập thấp cho mọi mẫu đầu vào vẫn đạt được chỉ số accuracy bằng 0.752 nhưng điểm f1_score của lớp dương sẽ bằng 0. Con số accuracy 0.752 gây hiểu nhầm vì mô hình hoàn toàn không thể phát hiện bất kỳ người nào thuộc nhóm thu nhập cao. Chỉ số f1_score đo lường dung hòa giữa precision và recall trên lớp dương, phản ánh đúng hiệu quả phân loại của mô hình. Việc không áp dụng phương pháp tính average macro hay average weighted giúp giữ nguyên bản chất đánh giá khắt khe trên lớp dương mà không bị lớp đa số kéo tăng điểm giả tạo.

---

## 3. Khó Khăn Gặp Phải và Cách Giải Quyết

| Khó khăn | Nguyên nhân | Cách giải quyết |
|---|---|---|
| Pipeline GitHub Actions không kéo được dữ liệu từ cloud | Chưa cấu hình hoặc thiếu biến môi trường credentials trong runner | Đưa nội dung service account vào secret và lưu thành file sa-key.json tạm |
| Service API trên máy ảo không nạp được mô hình mới | Đường dẫn file mô hình chưa khớp hoặc thiếu khai báo bucket môi trường | Cập nhật file cấu hình systemd service bổ sung đầy đủ biến ARTIFACT_BUCKET |
| Lỗi dvc pull khi commit dữ liệu mới ở bước 3 | Commit file con trỏ lên repository trước khi đẩy dữ liệu thật | Đẩy dữ liệu thật lên cloud bằng dvc push trước khi git push |

---

## 4. So Sánh Bước 2 và Bước 3 (bắt buộc, 2 - 3 câu)

| | f1_score | accuracy |
|---|---|---|
| Bước 2 (chỉ `train_batch1`) | 0.7109 | 0.8780 |
| Bước 3 (thêm `train_batch2`) | 0.7014 | 0.8740 |

**Nhận xét:** Khi bổ sung batch dữ liệu mới ở bước 3, chỉ số f1_score giảm nhẹ từ 0.7109 xuống 0.7014. Điều này xảy ra do cả hai tập dữ liệu được chia ngẫu nhiên từ cùng một nguồn nên mang phân bố tương đồng, mô hình đã học hầu hết thông tin từ tập ban đầu. Kết quả thể hiện quy trình huấn luyện liên tục tự động hoạt động chính xác từ khâu đẩy dữ liệu đến triển khai sản phẩm.
