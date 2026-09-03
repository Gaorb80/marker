# Hướng dẫn chuyển PDF bằng Marker

## 1. Cấu trúc thư mục

Đặt file PDF cần chuyển vào thư mục `scr`:

```text
marker/
└── z_file-pdf/
    ├── scr/
    │   ├── HUONG_DAN_CHUYEN_PDF_BANG_MARKER.md
    │   └── tai-lieu.pdf
    └── Output/
```

## 2. Lệnh chuyển một file

Mở PowerShell tại thư mục gốc `marker`, sau đó chạy:

```powershell
uv run marker_single "z_file-pdf\scr\tai-lieu.pdf" --output_dir "z_file-pdf\Output" --output_format markdown
```

Thay `tai-lieu.pdf` bằng tên file thật. Ví dụ:

```powershell
uv run marker_single "z_file-pdf\scr\nckh-c1.pdf" --output_dir "z_file-pdf\Output" --output_format markdown
```

## 3. Vị trí kết quả

Marker sẽ tạo:

```text
z_file-pdf/Output/tai-lieu.md
```

File `.md` là văn bản đã chuyển đổi. File `_meta.json` chứa thông tin metadata và thống kê trang.

## 4. Lần chạy đầu tiên

Lần đầu chạy `uv run`, UV sẽ tự động:

1. Tạo môi trường `.venv` cho repo.
2. Cài các gói phụ thuộc của Marker.
3. Tải model OCR/Layout của Surya.
4. Chạy chuyển đổi PDF.

Bước này có thể mất nhiều phút và cần Internet. Các lần sau sẽ nhanh hơn vì gói và model đã được cache.

## 5. Lưu ý quan trọng

- Chỉ chạy một lệnh Marker tại một thời điểm; không mở hai lệnh cùng xử lý một file.
- Không cần chạy lại nếu lệnh đã trả về `returncode=0` và đã có file `.md`.
- Nếu lần chạy bị dừng đột ngột và lần sau báo lỗi `ocr_error_server.lock`, đóng các tiến trình Marker/Surya còn lại rồi chạy lại một lần duy nhất.
- Không dùng `--disable_ocr` nếu PDF là bản scan hoặc cần giữ bố cục; tùy chọn này chỉ phù hợp khi PDF đã có lớp text tốt.

## 6. Kiểm tra nhanh kết quả

```powershell
Get-ChildItem "z_file-pdf\Output" -File
Get-Content "z_file-pdf\Output\tai-lieu.md" -TotalCount 30
```

## 7. Tóm tắt cách đã thực hiện với `nckh-c1.pdf`

- Tìm file đầu vào trong `z_file-pdf/scr`.
- Dùng entry point có sẵn `marker_single` của repo Marker.
- Đặt đầu ra vào `z_file-pdf/Output` và định dạng `markdown`.
- Marker đã tạo Markdown 8.090 byte và metadata xác nhận 4 trang.
