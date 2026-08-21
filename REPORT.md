# BÁO CÁO THỰC HÀNH LAB MLOPS - DAY 21

## Từ Thực Nghiệm Cục Bộ Đến Triển Khai Liên Tục

* **Họ và tên**: Nguyễn Cao Nam - 2A202601377
* **Repository**: [https://github.com/nguyencaonamm/Day21-Track2-2A202601377-NguyenCaoNam](https://github.com/nguyencaonamm/Day21-Track2-2A202601377-NguyenCaoNam)

---

## 1. Kết Quả Thực Nghiệm & Siêu Tham Số Tối Ưu (Bước 1)

### 1.1. Bảng đối sánh các lần chạy thí nghiệm trên MLflow:

|   Lần chạy   | `n_estimators` | `max_depth` | `min_samples_split` |         Accuracy         |    F1-Score (Weighted)    | Đánh giá                                |
| :-------------: | :--------------: | :-----------: | :-------------------: | :-----------------------: | :-----------------------: | ------------------------------------------ |
| **Run 1** |       100       |       5       |           2           |          0.5640          |          0.5534          | Underfitting do cây quá nông            |
| **Run 2** |        50        |       3       |           2           |          0.5580          |          0.5185          | Độ chính xác thấp nhất               |
| **Run 3** |       200       |      20      |           2           |          0.6840          |          0.6830          | Độ chính xác cao, phân loại tốt     |
| **Run 4** |       300       |      30      |           2           | **0.6820 - 0.6840** | **0.6811 - 0.6830** | **Bộ tham số tối ưu lựa chọn** |

* **Lý do lựa chọn**: Tập dữ liệu Wine Quality chứa 12 đặc trưng hóa lý phức tạp. Việc sử dụng độ sâu `max_depth >= 20` và `n_estimators = 300` giúp rừng cây quyết định nắm bắt tốt các tương quan phi tuyến giữa các đặc trưng và phân loại chính xác 3 mức chất lượng rượu vang.

---

## 2. Hệ Thống CI/CD & Phục Vụ Mô Hình (Bước 2)

* **Data Versioning (DVC)**: Toàn bộ dữ liệu được quản lý phiên bản qua DVC và lưu trữ trên Google Cloud Storage (`gs://mlops-wine-bucket-3690/dvc`).
* **CI/CD Pipeline (GitHub Actions)**: Tự động hóa qua 4 giai đoạn:
  1. `Unit Test`: Kiểm thử độc lập với Pytest trên dữ liệu giả lập.
  2. `Train`: Tải dữ liệu DVC, huấn luyện mô hình, ghi nhận metrics và xuất model artifact.
  3. `Eval Gate`: Kiểm tra chất lượng mô hình tự động trước khi triển khai.
  4. `Deploy`: SSH tự động vào máy chủ Cloud VM (`35.192.174.234`), khởi động lại dịch vụ FastAPI (`systemctl restart mlops-serve`).
* **Model Serving (FastAPI)**: Phục vụ mô hình dạng REST API tại cổng 8000:
  - `GET /health` $\rightarrow$ `{"status": "ok"}`
  - `POST /predict` $\rightarrow$ Trả về nhãn dự đoán thời gian thực (`prediction: 0, label: "thap"`).

---

## 3. Huấn Luyện Liên Tục (Continuous Training - Bước 3)

* Khi dữ liệu mới được bổ sung từ `train_phase2.csv` (tăng tổng số mẫu từ 2,998 lên 5,996 mẫu), chỉ với một commit cập nhật con trỏ DVC (`data: bo sung 2998 mau du lieu moi (train_phase2)`), pipeline GitHub Actions được kích hoạt tự động 100%.
* Mô hình mới được huấn luyện lại trên môi trường sạch và tự động cập nhật lên máy chủ Cloud VM mà không cần bất kỳ can thiệp thủ công nào.

---

## 4. Danh Sách Minh Chứng Đính Kèm

1. `screenshots/1_mlflow_experiments.png`: Giao diện MLflow UI với 4 lần chạy và đối sánh độ đo.
2. `screenshots/2_cloud_storage_bucket.png`: Giao diện Google Cloud Storage Console với Bucket `mlops-wine-bucket-3690`.
3. `screenshots/3_github_actions_pipeline_step2.png`: GitHub Actions Pipeline #1 hoàn thành cả 4 jobs màu xanh lá.
4. `screenshots/4_serving_api_prediction_test.png`: Kết quả kiểm thử REST API (`/health` và `/predict`) trên Cloud VM.
5. `screenshots/5_continuous_training_pipeline_step3.png`: GitHub Actions Pipeline #2 tự động kích hoạt bởi commit dữ liệu mới.
