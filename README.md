# Datathon

## Config
pip install optuna pickle scikit-learn pathlib numpy pandas matplotlib.pyplot json
Cấu trúc thư mục

Datathon/
├── Feature Engineering/
│   ├── sales.csv
│   ├── orders.csv
│   ├── order_items.csv
│   ├── products.csv
│   ├── customers.csv
│   ├── payments.csv
│   ├── promotions.csv
│   ├── inventory.csv
│   ├── sample_submission.csv
│   └── clean_FE.ipynb
├── Model/
│   └── hgb_cutoff_safe_recursive_pipeline.ipynb
├── outputs/
└── README.md

## Hướng dẫn chạy
git clone ""
Sau đó đặt hết tất cả các file .csv tải về từ Kaggle vào một tệp tên là "Feature Engineering".
### Feature Engineering
Chạy notebook Feature Engineering/clean_FE.ipynb
Notebook này sẽ tạo ra các tập dữ liệu đặc trưng dùng cho huấn luyện, validation, kiểm tra leakage và giải thích mô hình. Trong đó:
(1) Tập "features_validate.csv" sẽ dùng để tìm ra bộ tham số tối ưu cho mô hình HistGradientBoosting khi forecast qua OptunaSearch. Tập này sẽ dùng để validate qua 3 folds:
| Fold | Train period | Validation period | Mục đích |
|---|---|---|---|
| Fold A | 2012-07-04 → 2021-12-31 | 2022-01-01 → 2022-12-31 | Fold chính, gần giai đoạn forecast nhất |
| Fold B | 2012-07-04 → 2020-12-31 | 2021-01-01 → 2021-12-31 | Kiểm tra độ ổn định |
| Fold C | 2012-07-04 → 2021-06-30 | 2021-07-01 → 2022-06-30 | Kiểm tra horizon 12 tháng |

(2) final_features_forecast.csv
Tập này để train từ 2012-07-04 đến 2022-12-31, và dùng mô hình đó để dự báo từ 2023-01-01 đến 2024-07-01

### Chạy mô hình
Chạy notebook
Model/hgb_forecast.ipynb
Notebook này thực hiện việc:
#### 1. Load dữ liệu đã được xử lý đặc trưng
#### 2. Xây dựng bảng đặc trưng riêng cho từng mốc cutoff
#### 3. Huấn luyện mô hình HistGradientBoostingRegressor
#### 4. Tối ưu siêu tham số bằng Optuna
#### 5. Đánh giá mô hình bằng 3 fold theo thứ tự thời gian
#### 6. Dự báo đệ quy cho giai đoạn 2023-2024
#### 7. Xuất file submission.csv
#### 8. Xuất kết quả Feature Importance và SHAP

## Đảm bảo
- Không tràn dữ liệu từ tương lai cho quá khứ
- Không sử dụng dữ liệu ngoài

## Requirements:
source code, notebook, file submission
