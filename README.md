# BTL Data Mining - Vietnamese Fashion Review ABSA

Dự án phân tích cảm xúc theo khía cạnh (Aspect-Based Sentiment Analysis - ABSA) cho review sản phẩm quần áo tiếng Việt. Model dự đoán 5 khía cạnh: chất liệu, kích cỡ/form, thiết kế, gia công và giá trị thực tế.

## Chức năng chính

- Tiền xử lý dữ liệu review từ CSV.
- Gán nhãn lại dữ liệu bằng OpenAI API theo schema 5 khía cạnh.
- Notebook huấn luyện/xuất model PhoBERT.
- Webapp FastAPI + HTML/JS để demo dự đoán 1 review hoặc xử lý batch CSV.

## Cấu trúc thư mục

```text
.
├── BTL_DataMining.ipynb              # Notebook huấn luyện và export model
├── preprocess_dataset.py             # Làm sạch CSV cho webapp
├── relabel_dataset_openai.py         # Gán nhãn bằng OpenAI API
├── Dataset_duy_clean.csv             # Dataset đã làm sạch, dùng demo batch
├── webapp/
│   ├── backend/                      # FastAPI backend
│   ├── frontend/                     # Giao diện web tĩnh
│   ├── run.bat                       # Chạy webapp trên Windows
│   └── run.sh                        # Chạy webapp trên Linux/macOS
└── onnx_export/                      # Model/tokenizer, tracked bằng Git LFS
```

## Chuẩn bị môi trường

Yêu cầu:

- Python 3.10 trở lên.
- RAM tối thiểu khoảng 2 GB nếu chạy ONNX model.
- File model ONNX và tokenizer đặt đúng vị trí:

```text
onnx_export/
├── multioutput_phobert.onnx
└── tokenizer_phobert/
    ├── added_tokens.json
    ├── bpe.codes
    ├── tokenizer_config.json
    └── vocab.txt
```

Các file model `.onnx` và `.pth` trong `onnx_export/` được quản lý bằng Git LFS vì vượt giới hạn 100 MB/file của GitHub. File `onnx_export.zip` là bản nén trùng lặp nên vẫn bị ignore.

## Cấu hình API key

API key chỉ cần cho script `relabel_dataset_openai.py`, không cần để chạy webapp dự đoán ONNX.

```powershell
copy .env.example .env
```

Sau đó điền `OPENAI_API_KEY` trong `.env`. Không commit `.env`.

## Chạy tiền xử lý dữ liệu

```powershell
python -m venv venv
.\venv\Scripts\activate
pip install pandas tqdm openai python-dotenv pydantic

python preprocess_dataset.py --input Dataset_duy.csv --output Dataset_duy_clean.csv
```

Các tùy chọn hữu ích:

```powershell
python preprocess_dataset.py --min-words 4
python preprocess_dataset.py --no-filter
python preprocess_dataset.py --dump-removed removed_rows.csv
```


## Chạy webapp

Trước khi chạy, đặt `onnx_export/` ở thư mục gốc dự án như mô tả ở trên.

Windows:

```powershell
cd webapp
.\run.bat
```

Linux/macOS:

```bash
cd webapp
bash run.sh
```

Server chạy tại:

```text
http://localhost:8000
```

Các API chính:

| Method | Path | Mô tả |
|---|---|---|
| GET | `/health` | Kiểm tra trạng thái load model |
| POST | `/predict` | Dự đoán 1 review |
| POST | `/predict_batch` | Upload CSV có cột `rating`, `content` để dự đoán hàng loạt |

## Kiểm tra bảo mật trước khi push

Đã cấu hình `.gitignore` để chặn:

- `.env` và mọi API key local.
- `venv/`, `.venv/`, cache Python.
- `onnx_export.zip` và các file model lớn ngoài `onnx_export/`.
- Dataset raw có `customer_name`: `Dataset_duy.csv`, `Dataset_quan.csv`, `Dataset_quan_relabeled.csv`.

Lưu ý quan trọng:

- Repo local từng có khóa API thật trong `.env`; hãy revoke/rotate các khóa đó trước khi public repo.
- `Dataset_duy_clean.csv` đã được giữ lại để demo batch, nhưng vẫn nên rà soát dữ liệu trước khi public.
- GitHub chặn file trên 100 MB; repo này dùng Git LFS cho model trong `onnx_export/`.

Lệnh kiểm tra nhanh trước khi commit:

```powershell
git status --short
git check-ignore -v .env onnx_export.zip Dataset_duy.csv
rg -n --hidden -g '!venv/**' -g '!webapp/.venv/**' -g '!onnx_export/**' "sk-|OPENAI_API_KEY|OPENROUTER_API_KEY|password|secret|token"
```

## Đẩy lên GitHub

Thư mục hiện tại chưa có `.git`. Nếu muốn tạo repo local và push lên remote đã cung cấp:

```powershell
git init
git branch -M main
git remote add origin https://github.com/datyespro/BTL-Data-Mining.git
git add .
git status --short
git commit -m "Prepare project for GitHub"
git push -u origin main
```

Kiểm tra kỹ `git status --short` trước khi commit để chắc chắn `.env`, `venv/`, `webapp/.venv/`, `onnx_export.zip` và dataset raw không nằm trong danh sách staged. `onnx_export/` sẽ được staged qua Git LFS.
