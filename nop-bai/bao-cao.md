# Báo Cáo Lab Day 21 - CI/CD cho AI Systems

| | |
|---|---|
| Họ và tên | Nguyễn Nhật Quang |
| MSSV | 2A202601452 |
| Lớp / Khóa | K4 |
| Repo GitHub | https://github.com/Raijuz/Track2-K4-Day21-2A202601452-NguyenNhatQuang |
| Ngày nộp | 21/08/2026 |

---

## 1. Bộ Siêu Tham Số Đã Chọn và Lý Do

| Lần chạy | n_estimators | learning_rate | max_depth | f1_score | accuracy |
|---|---|---|---|---|---|
| 1 | 100 | 0.1 | 3 | 0.7109 | 0.8780 |
| 2 | 10 | 0.1 | 2 | 0.2963 | 0.5250 |
| 3 | 200 | 0.1 | 5 | 0.7149 | 0.8740 |

**Bộ siêu tham số đã chọn:** `n_estimators=200`, `learning_rate=0.1`, `max_depth=5`.

**Lý do:** Bộ tham số này mang lại chỉ số F1 cao nhất (0.7149) trên tập holdout, giúp cây quyết định phân tách tốt hơn các đặc trưng phi tuyến trong bài toán Adult Income. Mặc dù Lần chạy 1 đạt accuracy cao hơn một chút (0.8780 so với 0.8740), nhưng Lần chạy 3 có F1 vượt trội, chứng minh mô hình dự đoán chính xác hơn trên lớp thiểu số (thu nhập > 50K). Việc tăng `n_estimators=200` kết hợp `learning_rate=0.1` và `max_depth=5` giúp mô hình hội tụ ổn định, tránh bị underfitting nghiêm trọng như Lần chạy 2 (`n_estimators=10`, F1 chỉ đạt 0.2963).

---

## 2. Vì Sao Ngưỡng Chất Lượng Đặt Trên F1 Chứ Không Phải Accuracy

Tập dữ liệu Adult có phân bố lớp mất cân bằng rõ rệt với khoảng 75% mẫu thuộc lớp thu nhập thấp (<= 50K) và chỉ 25% mẫu thuộc lớp thu nhập cao (> 50K). Một mô hình tầm thường luôn đoán nhãn "thu nhập thấp" cho tất cả các trường hợp vẫn sẽ đạt độ chính xác (accuracy) lên tới 75%, nhưng mô hình này hoàn toàn vô dụng vì không nhận diện được bất kỳ đối tượng mục tiêu nào. 

Chỉ số F1-score là trung bình điều hòa giữa Precision và Recall, đo lường trực tiếp năng lực nhận diện lớp thiểu số mà accuracy không thể phản ánh. Lab tính F1 trực tiếp trên lớp dương (`pos_label=1`) thay vì dùng `average="weighted"` hay `average="macro"` vì các phương pháp trung bình đa lớp sẽ bị kéo theo bởi lớp đa số (75%), che giấu đi sự yếu kém của mô hình trên lớp dữ liệu quan trọng cần dự đoán.

---

## 3. Khó Khăn Gặp Phải và Cách Giải Quyết

| Khó khăn | Nguyên nhân | Cách giải quyết |
|---|---|---|
| Lỗi tạo Service Account Key trên GCP (`disableServiceAccountKeyCreation`). | Chính sách Organization Policy mặc định của GCP chặn tạo key cho SA. | Vào Organization Policies trên GCP Console, chọn Override policy và chuyển Enforcement sang Off rồi set lại Policty |
| Runner GitHub Actions báo lỗi thiếu module `pkg_resources`. | Phiên bản `setuptools>=72` đã loại bỏ hoàn toàn `pkg_resources`. | Thêm phần `setuptools<72` trong `requirements.txt` và cài lại |
| CI runner không kéo được dữ liệu DVC từ Google Cloud Storage. | DVC cần đường dẫn file credential cục bộ và biến môi trường xác thực. | Thiết lập `dvc remote modify labstore credentialpath` và truyền secret an toàn qua env variable. |

---

## 4. So Sánh Bước 2 và Bước 3 (bắt buộc, 2 - 3 câu)

| | f1_score | accuracy |
|---|---|---|
| Bước 2 (chỉ `train_batch1`) | 0.7149 | 0.8740 |
| Bước 3 (thêm `train_batch2`) | 0.7354 | 0.8820 |

**Nhận xét:** Khi bổ sung thêm `train_batch2` (tổng cộng 44.722 mẫu), F1-score tăng từ 0.7149 lên 0.7354 và accuracy tăng từ 0.8740 lên 0.8820. Do dữ liệu mới cùng phân phối với dữ liệu ban đầu, việc tăng gấp đôi số lượng mẫu giúp Gradient Boosting học được các ranh giới phân tách chi tiết hơn và giảm thiểu phương sai ước lượng, từ đó cải thiện chất lượng dự đoán tổng thể trên tập holdout.
