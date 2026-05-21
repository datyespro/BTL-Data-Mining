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

## Lưu ý dữ liệu public

`Dataset_quan.csv` được đưa lên repository theo yêu cầu nộp bài. File này có cột `customer_name`, vì vậy cần cân nhắc quyền riêng tư nếu repository để public.

Các file vẫn bị ignore:

- `.env`
- `venv/`, `webapp/.venv/`
- `onnx_export.zip`
- `Dataset_duy.csv`
- `Dataset_quan_relabeled.csv`
