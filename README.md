# Data Mining Project – Nhóm 3

Dự án môn **Khai phá dữ liệu**, thực hiện 4 bài tập thực hành (tiền xử lý, phân lớp, luật kết hợp, gom cụm) trên **3 bộ dữ liệu**:

| Bộ | Dữ liệu | Kỹ thuật thực hiện |
|----|---------|---------------------|
| Bộ 1 | Olist Brazilian E-Commerce | Phân lớp, Gom cụm |
| Bộ 2 | Inside Airbnb – New York City | Luật kết hợp, Gom cụm |
| Bộ 3 | US Accidents | Phân lớp, Luật kết hợp |

> Ghi chú: Luật kết hợp ban đầu dự kiến làm trên Olist, nhưng khảo sát
> thực tế (Mục 1.3 của `khao-sat.ipynb`) cho thấy đa số đơn hàng Olist
> chỉ có 1 sản phẩm/ngành hàng — không đủ đa dạng để sinh luật kết hợp
> có ý nghĩa. Vì vậy nhóm chuyển Luật kết hợp sang Airbnb (mỗi listing
> có nhiều tiện nghi cùng lúc, phù hợp hơn cho Apriori/FP-Growth).

Phân công 6 thành viên: xem chi tiết trong [`HUONG_DAN_LAM_BAI.md`](./HUONG_DAN_LAM_BAI.md).

---

## Đồ án tổng hợp – US Accidents

Đồ án phân tích các tổ hợp điều kiện liên hệ với **mức ảnh hưởng giao thông `Severity` 3-4** trong snapshot US Accidents gồm California và Texas. `Severity` không phải số thương vong. Hai kỹ thuật được kết hợp là mô hình phân lớp nhị phân và luật kết hợp; phân lớp bốn mức chỉ giữ làm nền kỹ thuật. Notebook chính là [`phan-tich-tong-hop.ipynb`](./phan-tich-tong-hop.ipynb).

Phần D.2-D.6 sử dụng nguồn dữ liệu duy nhất:

```text
data/processed/us_accidents/accidents_preprocessed.csv
```

Thứ tự thực thi:

1. Tạo môi trường và cài `requirements.txt`.
2. Mở repository ở thư mục gốc, chọn kernel của môi trường vừa tạo.
3. Mở `phan-tich-tong-hop.ipynb`, chọn **Restart Kernel & Run All**.
4. Kiểm tra artifacts được tạo trong `artifacts/`, đặc biệt `split_us_accidents.csv`, `data_bias_audit.csv`, `model_predictions_baseline.csv`, `rules_train.csv`, `rules_model_test.csv` và `rules_stratified_robustness.csv`.

Notebook tạo split 80/20 chung theo `ID`, stratify theo `Severity`, seed 42. `Distance(mi)` bị loại khỏi feature nhận diện sớm; 7.236 dòng thiếu thời gian được giữ là `Unknown`. Imputation, scaling, calibration, lựa chọn mô hình và khai phá luật chỉ fit trên train; test chỉ dùng để đánh giá cuối, đối chiếu rules-model và kiểm tra độ bền theo Source/State.

Các tài liệu liên quan:

- [`artifacts/data_contract_us_accidents.md`](./artifacts/data_contract_us_accidents.md): schema, nhãn, feature whitelist và quy tắc transaction.
- [`docs/nhat_ky_quyet_dinh.md`](./docs/nhat_ky_quyet_dinh.md): các quyết định phân tích và bằng chứng.
- [`data/README.md`](./data/README.md): nguồn dữ liệu sử dụng trong đồ án.
- Kế hoạch triển khai chi tiết được lưu cục bộ và không thuộc bộ file bàn giao trên Git.

Để chạy lại notebook bằng dòng lệnh:

```powershell
python -m nbconvert --to notebook --execute .\phan-tich-tong-hop.ipynb `
  --output phan-tich-tong-hop.ipynb --ExecutePreprocessor.timeout=900
```

---

## 1. Cài Python

Kiểm tra đã có Python chưa (PowerShell):

```powershell
python --version
```

Nếu chưa có, tải tại: https://www.python.org/ (chọn bản ≥ 3.9, khi cài nhớ tick **Add Python to PATH**). Môi trường đồ án hiện đã kiểm tra trên Python 3.9.13.

---

## 2. Cài Git

```powershell
git --version
```

Nếu chưa có, tải tại: https://git-scm.com/

---

## 3. Clone project

```powershell
git clone https://github.com/<ten-repo-cua-nhom>.git
cd <ten-repo-cua-nhom>
code .
```

---

## 4. Tạo môi trường ảo Python

Trong Terminal của VS Code (PowerShell):

```powershell
python -m venv venv
.\venv\Scripts\activate
```

Nếu kích hoạt thành công, đầu dòng lệnh sẽ hiện `(venv)`.

> Nếu PowerShell báo lỗi không cho chạy script, mở PowerShell với quyền Admin và chạy 1 lần:
> ```powershell
> Set-ExecutionPolicy -Scope CurrentUser RemoteSigned
> ```

---

## 5. Cài thư viện

```powershell
pip install -r requirements.txt
```

Kiểm tra:

```powershell
pip list
```

`requirements.txt` gồm các thư viện xử lý dữ liệu, trực quan hóa, notebook, phân lớp, luật kết hợp và xử lý mất cân bằng; phiên bản cụ thể được khóa trực tiếp trong file.

---

## 6. Chuẩn bị dữ liệu (BẮT BUỘC – mỗi thành viên tự tải về máy)

**Dữ liệu thô KHÔNG được đưa lên GitHub** (đã thêm `data/raw/` vào `.gitignore`). Mỗi người phải tự tải và đặt đúng vị trí bên dưới trước khi chạy notebook.

> ⚠️ Tên thư mục dưới đây phải khớp **chính xác** (kể cả dấu gạch dưới
> `_`) với đường dẫn mà `khao-sat.ipynb` dùng để đọc file
> (`RAW_DIR = '../data/raw'`). Đặt sai tên thư mục (ví dụ dùng gạch
> ngang `-` thay vì gạch dưới `_`) sẽ khiến notebook báo lỗi
> `FileNotFoundError` khi chạy.

### 6.1. Bộ 1 – Olist Brazilian E-Commerce

- Nguồn: https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce
- Cần đăng nhập Kaggle → nút **Download** → giải nén.
- Copy toàn bộ các file `.csv` vào:

```text
data/raw/olist/
```

### 6.2. Bộ 2 – Inside Airbnb, New York City

- Nguồn: http://insideairbnb.com/get-the-data/
- Chọn thành phố **New York City** trong danh sách.
- Tải file **`listings.csv`** bản chi tiết (không dùng bản `listings_summary.csv` vì bị rút gọn thuộc tính).
- Có thể tải thêm `calendar.csv`, `reviews.csv` nếu nhóm cần cho luật kết hợp/gom cụm.
- Giấy phép: Creative Commons Attribution 4.0 International (CC BY 4.0).
- Copy vào:

```text
data/raw/airbnb/
```

### 6.3. Bộ 3 – US Accidents

- Nguồn: https://www.kaggle.com/datasets/sobhanmoosavi/us-accidents
- Cần đăng nhập Kaggle → **Download** → giải nén (đặt tên file: `US_Accidents.csv`).
- Đặt file gốc chưa lọc vào:

```text
data/raw/us_accidents/
```

### 6.4. Lưu ý chung

- Không đổi tên file gốc tải về.
- Không sửa trực tiếp dữ liệu trong `data/raw/`.
- Không upload bất kỳ file nào trong `data/raw/` lên Git.
- Dữ liệu sau xử lý (`data/processed/`) là dữ liệu nhẹ hơn, **có thể** commit nếu nhóm thống nhất, hoặc dùng script tái tạo (`bai1.../khao-sat.ipynb` chạy lại được).

---

## 7. Cấu trúc thư mục

```text
ten-nhom/
├── README.md                       # file này
├── HUONG_DAN_LAM_BAI.md             # hướng dẫn chi tiết làm từng bài
├── requirements.txt
├── .gitignore
│
├── data/
│   ├── raw/                        # dữ liệu gốc — KHÔNG commit
│   │   ├── olist/
│   │   ├── airbnb/
│   │   └── us_accidents/
│   └── processed/                  # dữ liệu sau tiền xử lý
│       ├── olist/
│       ├── airbnb/
│       └── us_accidents/
│
├── bai1-du-lieu-tien-xu-ly/
│   ├── khao-sat.ipynb              # 1 notebook gộp cả 3 bộ (hàm dùng chung + 3 phần Bộ 1/2/3 + ma trận)
│   └── bao-cao-bai1.pdf
│
├── bai2-phan-lop/                  # Olist + US Accidents
│   ├── phan-lop.ipynb
│   └── bao-cao-bai2.pdf
│
├── bai3-luat-ket-hop/              # Airbnb + US Accidents
│   ├── luat-ket-hop.ipynb
│   └── bao-cao-bai3.pdf
│
└── bai4-gom-cum/                   # Olist + Airbnb
    ├── gom-cum.ipynb
    └── bao-cao-bai4.pdf
```

---

## 8. Làm việc với Jupyter Notebook trong VS Code

1. Mở file `.ipynb`.
2. Ở góc trên bên phải, chọn kernel `Python 3.x ('venv')`.
3. Nếu chưa thấy kernel, chạy:

```powershell
python -m ipykernel install --user --name=data-mining-venv
```

4. Chạy từng ô bằng **Shift + Enter**.
5. Trước khi commit/push: **Kernel → Restart & Run All** để đảm bảo notebook chạy lại được từ đầu, không lỗi.

---

## 9. Git & làm việc nhóm

Trước khi bắt đầu làm việc, luôn cập nhật branch từ `main`:

```powershell
git checkout main
git pull origin main
git checkout feature/ten-branch-cua-ban
git merge main
```

Quy tắc:

- Dùng đường dẫn tương đối (`../data/processed/...`), không dùng đường dẫn tuyệt đối.
- Không commit `venv/`, `__pycache__/`, hoặc bất kỳ file nào trong `data/raw/`.
- Không sửa file/notebook của người khác khi chưa trao đổi.
- Mỗi bước tiền xử lý/thuật toán ghi rõ lý do trong comment hoặc markdown cell.
- Đặt tên branch theo mẫu: `feature/<ten>-<bai>` (vd: `feature/an-bai2-airbnb`).

---

## 10. Tài liệu liên quan

- [`HUONG_DAN_LAM_BAI.md`](./HUONG_DAN_LAM_BAI.md) — phân công thành viên, checklist chi tiết cho từng bài, tiêu chí báo cáo.
