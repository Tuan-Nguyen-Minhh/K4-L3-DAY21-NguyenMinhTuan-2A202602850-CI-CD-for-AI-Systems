# Báo Cáo Lab Day 21 - CI/CD cho AI Systems

| | |
|---|---|
| Họ và tên | Nguyen Minh Tuan |
| MSSV | 2A202602850 |
| Lớp / Khóa | K4 |
| Repo GitHub | https://github.com/Tuan-Nguyen-Minhh/K4-L3-DAY21-NguyenMinhTuan-2A202602850-CI-CD-for-AI-Systems|
| Ngày nộp | 7/10/2026 |

---

## 1. Bộ Siêu Tham Số Đã Chọn và Lý Do

| Lần chạy | n_estimators | learning_rate | max_depth | f1_score | accuracy |
|---|---|---|---|---|---|
| 1 | 200 | 0.1 | 5 | 0.7149 | 0.874 |
| 2 | 50 | 0.05 | 2 | 0.6051 | 0.846 |
| 3 | 100 | 0.1 | 3 | 0.7109 | 0.878 |

**Bộ siêu tham số đã chọn:** `n_estimators=100`, `learning_rate=0.1`, `max_depth=3`.

**Lý do:** Bộ này đạt f1_score 0.7109, vượt ngưỡng 0.65, và chỉ dùng một nửa số cây so với bộ 200 cây trong khi chênh lệch f1 là không đáng kể (0.7149). Lần có accuracy cao nhất (lần 3, 0.878) không trùng với lần có f1 cao nhất (lần 1), cho thấy accuracy không phản ánh đầy đủ khả năng bắt lớp thu nhập cao. Đối chiếu lần 2 (50 cây, learning_rate 0.05, độ sâu 2) chỉ đạt f1 0.605, cho thấy learning_rate thấp cần nhiều cây hơn mới bù lại được.

---

## 2. Vì Sao Ngưỡng Chất Lượng Đặt Trên F1 Chứ Không Phải Accuracy

Tập Adult có khoảng 24,8% mẫu thuộc lớp thu nhập cao nên phân bố lớp lệch. Một mô hình luôn trả lời "thu nhập thấp" đạt accuracy 0,752 nhưng f1_score của lớp dương bằng 0 do không bắt được mẫu nào, nên accuracy gây hiểu nhầm. F1 của lớp dương kết hợp precision và recall, đo khả năng mô hình vừa không bỏ sót vừa không gán nhầm lớp hiếm. Không dùng average="weighted" hay "macro" vì hai giá trị này bị lớp đa số kéo lên, làm mất ý nghĩa của ngưỡng 0.65.

---

## 3. Khó Khăn Gặp Phải và Cách Giải Quyết

| Khó khăn | Nguyên nhân | Cách giải quyết |
|---|---|---|
| MLflow không import được ở môi trường cục bộ | SQLAlchemy 2.x không tương thích MLflow 2.13 | Pin `SQLAlchemy<2.1` trong requirements.txt |
| VM load model báo lỗi unpickle | scikit-learn trên VM khác phiên bản lúc train | Pin `scikit-learn==1.4.2` trên VM |
| GitHub Actions SSH vào VM bị timeout | Security group chỉ mở cổng 22 cho My IP | Mở cổng 22 cho 0.0.0.0/0 |

---

## 4. So Sánh Bước 2 và Bước 3

| | f1_score | accuracy |
|---|---|---|
| Bước 2 (chỉ `train_batch1`) | 0.7109 | 0.8780 |
| Bước 3 (thêm `train_batch2`) | 0.7014 | 0.8740 |

**Nhận xét:** f1_score giảm nhẹ khoảng 0,01 và accuracy gần như không đổi. Hai nửa dữ liệu được chia ngẫu nhiên từ cùng một nguồn nên có cùng phân phối; gấp đôi dữ liệu không mang thêm thông tin mới, chỉ làm chỉ số dao động nhẹ. Điều được kiểm chứng ở Bước 3 là quy trình tự động chạy đúng từ commit dữ liệu đến triển khai, không phải chỉ số cao hơn.
