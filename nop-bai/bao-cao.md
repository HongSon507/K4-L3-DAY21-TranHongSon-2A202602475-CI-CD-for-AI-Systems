# Báo Cáo Lab Day 21 - CI/CD cho AI Systems

| | |
|---|---|
| Họ và tên | Trần Hồng Sơn |
| MSSV | 2A202602475 |
| Lớp / Khóa | K4 |
| Repo GitHub | https://github.com/HongSon507/K4-L3-DAY21-TranHongSon-2A202602475-CI-CD-for-AI-Systems |
| Ngày nộp | 07/10/2026 |

---

## 1. Bộ Siêu Tham Số Đã Chọn và Lý Do

| Lần chạy | n_estimators | learning_rate | max_depth | f1_score | accuracy |
|---|---|---|---|---|---|
| 1 | 200 | 0.1 | 5 | 0.7149 | 0.8740 |
| 2 | 100 | 0.1 | 3 | 0.7109 | 0.8780 |
| 3 | 50 | 0.05 | 2 | 0.6051 | 0.8460 |

**Bộ siêu tham số đã chọn:** `n_estimators=200`, `learning_rate=0.1`, `max_depth=5`.

**Lý do:** Bộ siêu tham số này đạt chỉ số `f1_score` cao nhất (0.7149) trên tập holdout, vượt trội rõ rệt so với bộ siêu tham số yếu ở lần chạy 3 (0.6051). Đáng chú ý, lần chạy có accuracy cao nhất lại là lần chạy 2 (0.8780 so với 0.8740), cho thấy accuracy cao không đồng nghĩa với khả năng nhận diện tốt lớp thiểu số thu nhập cao. Trong mô hình Gradient Boosting, sự kết hợp giữa số lượng cây đủ lớn (`n_estimators=200`), độ sâu vừa phải (`max_depth=5`) và tốc độ học hợp lý (`learning_rate=0.1`) tạo ra sự cân bằng tối ưu giữa việc nắm bắt các quan hệ phi tuyến phức tạp mà không dẫn đến overfitting.

---

## 2. Vì Sao Ngưỡng Chất Lượng Đặt Trên F1 Chứ Không Phải Accuracy

Tập dữ liệu Adult Income có sự mất cân bằng lớp rõ rệt với tỷ lệ lớp thu nhập cao (>50K) chỉ chiếm khoảng 25%, trong khi lớp thu nhập thấp chiếm tới 75%. Một mô hình ngây thơ (naive) chỉ cần luôn dự đoán nhãn "thu nhập thấp" cho mọi mẫu dữ liệu đã dễ dàng đạt được accuracy 75%, nhưng mô hình này hoàn toàn vô dụng trên thực tế vì tỷ lệ phát hiện trường hợp thu nhập cao bằng 0. Do đó, accuracy tạo ra ảo giác về hiệu năng cao và gây hiểu nhầm nghiêm trọng.

Chỉ số `f1_score` của lớp dương là trung bình điều hòa giữa Precision (độ chính xác khi dự đoán thu nhập cao) và Recall (độ bao phủ các ca thu nhập cao thực tế). F1 đòi hỏi mô hình vừa phải tìm đúng vừa không được bỏ sót lớp mục tiêu. Khi tính toán cho bài toán này, tuyệt đối không sử dụng `average="weighted"` hay `average="macro"` vì trọng số của lớp đa số sẽ kéo chỉ số tổng thể lên cao, làm lu mờ hoàn toàn khuyết điểm của mô hình đối với lớp quan trọng cần dự đoán.

---

## 3. Khó Khăn Gặp Phải và Cách Giải Quyết

| Khó khăn | Nguyên nhân | Cách giải quyết |
|---|---|---|
| DVC pull báo lỗi `401 Invalid Credentials` trên CI runner | Secret `STORAGE_CREDENTIALS` bị lỗi ký tự escape chuỗi JSON qua lệnh echo và đường dẫn DVC tương đối | Dùng script Python nạp trực tiếp JSON hợp lệ và đặt đường dẫn tuyệt đối cho remote DVC |
| Unit test thất bại với `PermissionError: [Errno 13] '/D:'` | Thư mục `mlruns` cục bộ chứa đường dẫn ổ đĩa Windows tuyệt đối bị commit lên GitHub | Thêm `mlruns/` vào `.gitignore` và dùng thư mục `tmp_path` cô lập cho pytest |
| Service `income-api` crash liên tục khi khởi động trên VM | Thiếu thư viện FastAPI và lệch phiên bản `scikit-learn` giữa môi trường train (1.4.2) và VM (1.7.2) | Cài đặt đầy đủ các thư viện và đồng bộ phiên bản `scikit-learn==1.4.2` trên VM |

---

## 4. So Sánh Bước 2 và Bước 3 (bắt buộc, 2 - 3 câu)

| | f1_score | accuracy |
|---|---|---|
| Bước 2 (chỉ `train_batch1`) | 0.7149 | 0.8740 |
| Bước 3 (thêm `train_batch2`) | 0.7354 | 0.8820 |

**Nhận xét:** Khi bổ sung 22.361 mẫu dữ liệu mới từ `train_batch2` (tổng cộng 44.722 mẫu), kích thước tập huấn luyện tăng gấp đôi giúp mô hình học thêm được nhiều biến thể và đặc trưng đại diện của nhóm thu nhập cao (>50K). Nhờ đó, chỉ số `f1_score` tăng đáng kể từ 0.7149 lên 0.7354 và `accuracy` tăng từ 0.8740 lên 0.8820, chứng minh tập dữ liệu mới mang lại giá trị thông tin tích cực giúp mô hình tổng quát hóa tốt hơn trên tập đánh giá.
