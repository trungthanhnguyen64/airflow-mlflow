# 📦 DVC Tutorial cho người mới (dùng trong project này)

Tài liệu này hướng dẫn **từng bước** cách sử dụng **DVC (Data Version Control)** cho người mới, áp dụng trực tiếp cho project **ML/NLP** trong repository này.

---

## 1. DVC là gì? Vì sao cần DVC?

Trong các project Machine Learning:

* **Git** rất tốt để version code
* Nhưng **KHÔNG phù hợp** để version file dữ liệu lớn (CSV, audio, image, ...)

👉 **DVC** được dùng để:

* Version dữ liệu giống như Git version code
* Tách **data** và **code** ra khỏi nhau
* Đảm bảo **reproducible experiments** (tái lập thí nghiệm)

> Nguyên tắc: **Git quản lý metadata – DVC quản lý data thật**

---

## 2. Cấu trúc project (ví dụ)

```text
airflow-mlflow/
├── data/
│   ├── train_v1.csv
│   ├── train_v2.csv
│   └── .gitignore
├── notebooks/
│   └── ToxicCommentClassifier.ipynb
├── .dvc/
├── .gitignore
├── README.md
```

---

## 3. Cài đặt DVC

```bash
pip install dvc
```

Kiểm tra:

```bash
dvc --version
```

---

## 4. Khởi tạo DVC (chỉ làm 1 lần)

Tại thư mục root của project:

```bash
dvc init
```

Sau đó commit:

```bash
git add .dvc .dvcignore
git commit -m "Initialize DVC"
```

---

## 5. Cấu hình DVC remote (bắt buộc nếu muốn push/pull)

### 5.1 Dùng local folder (đơn giản, khuyến nghị cho người mới)

```bash
mkdir -p ~/dvc-storage
dvc remote add -d storage ~/dvc-storage
```

Commit config:

```bash
git add .dvc/config
git commit -m "Configure DVC local remote"
```

> Sau này có thể đổi sang S3 / GCS mà không ảnh hưởng code

---

## 6. Version dữ liệu với DVC

### 6.1 Add dataset vào DVC

Ví dụ với 2 file train:

```bash
dvc add data/train_v1.csv
dvc add data/train_v2.csv
```

DVC sẽ:

* Tạo `data/train_v1.csv.dvc`, `data/train_v2.csv.dvc`
* Thêm file CSV thật vào `.gitignore`

### 6.2 Push dữ liệu lên remote

```bash
dvc push
```

### 6.3 Commit metadata bằng Git

```bash
git add data/train_v1.csv.dvc data/train_v2.csv.dvc .gitignore
git commit -m "Add training datasets with DVC"
```

📌 **KHÔNG commit file CSV thật**

---

## 7. Sử dụng dataset trong notebook

Trong notebook, chọn dataset bằng biến:

```python
DATA_FILE = "train_v1.csv"  # hoặc train_v2.csv
train_data_path = os.path.join(tutorial_dir_path, "data", DATA_FILE)
train = pd.read_csv(train_data_path)
```

Có thể log vào MLflow:

```python
mlflow.log_param("dataset", DATA_FILE)
```

---

## 8. Khi dữ liệu thay đổi

### Trường hợp 1: Dataset mới (file mới)

```bash
dvc add data/train_v3.csv
dvc push
git add data/train_v3.csv.dvc
git commit -m "Add train_v3 dataset"
```

### Trường hợp 2: Giữ nguyên tên file (train.csv) nhưng nội dung thay đổi

```bash
dvc add data/train.csv
dvc push
git commit -am "Update training data"
```

---

## 9. Tái lập dữ liệu (reproduce)

Quay lại dataset ở commit cũ:

```bash
git checkout <commit_sha>
dvc pull
```

👉 Dataset, code, model sẽ khớp đúng snapshot đó

---

## 10. Các lệnh DVC hay dùng

| Lệnh             | Ý nghĩa              |
| ---------------- | -------------------- |
| `dvc init`       | Khởi tạo DVC         |
| `dvc add <file>` | Version dữ liệu      |
| `dvc push`       | Push data lên remote |
| `dvc pull`       | Pull data về         |
| `dvc status`     | Kiểm tra trạng thái  |

---

## 11. Nguyên tắc quan trọng cần nhớ

* ❌ Không commit file dữ liệu lớn vào Git
* ✅ Chỉ commit file `.dvc`
* ✅ Mỗi Git commit = 1 snapshot data + code
* ✅ MLflow + DVC = trace được model tạo từ dataset nào

---

## 12. Tóm tắt ngắn gọn

> Project này dùng DVC để version dữ liệu training.
> Git quản lý metadata, DVC quản lý data thật.
> Nhờ đó các thí nghiệm ML có thể tái lập chính xác.

---

