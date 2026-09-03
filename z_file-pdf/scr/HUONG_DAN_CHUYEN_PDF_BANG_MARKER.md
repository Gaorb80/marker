# Huong dan chuyen PDF bang Marker

## 1. Cau truc thu muc

Dat file PDF can chuyen vao thu muc `scr`:

```text
marker/
└── z_file-pdf/
    ├── scr/
    │   ├── HUONG_DAN_CHUYEN_PDF_BANG_MARKER.md
    │   └── tai-lieu.pdf
    └── output/
```

## 2. Lenh chuyen mot file

Mo PowerShell tai thu muc goc `marker`, sau do chay:

```powershell
uv run marker_single "z_file-pdf\scr\tai-lieu.pdf" --output_dir "z_file-pdf\output" --output_format markdown
```

Thay `tai-lieu.pdf` bang ten file that. Vi du:

```powershell
uv run marker_single "z_file-pdf\scr\nckh-c1.pdf" --output_dir "z_file-pdf\output" --output_format markdown
```

## 3. Vi tri ket qua

Marker se tao:

```text
z_file-pdf/output/tai-lieu/tai-lieu.md
z_file-pdf/output/tai-lieu/tai-lieu_meta.json
```

File `.md` la van ban da chuyen doi. File `_meta.json` chua thong tin metadata va thong ke trang.

## 4. Lan chay dau tien

Lan dau chay `uv run`, UV se tu dong:

1. Tao moi truong `.venv` cho repo.
2. Cai cac goi phu thuoc cua Marker.
3. Tai model OCR/Layout cua Surya.
4. Chay chuyen doi PDF.

Buoc nay co the mat nhieu phut va can Internet. Cac lan sau se nhanh hon vi goi va model da duoc cache.

## 5. Luu y quan trong

- Chi chay mot lenh Marker tai mot thoi diem; khong mo hai lenh cung xu ly mot file.
- Khong can chay lai neu lenh da tra ve `returncode=0` va da co file `.md`.
- Neu lan chay bi dung dot ngot va lan sau bao loi `ocr_error_server.lock`, dong cac tien trinh Marker/Surya con lai roi chay lai mot lan duy nhat.
- Khong dung `--disable_ocr` neu PDF la ban scan hoac can giu bo cuc; tuy chon nay chi phu hop khi PDF da co lop text tot.

## 6. Kiem tra nhanh ket qua

```powershell
Get-ChildItem "z_file-pdf\output" -Recurse -File
Get-Content "z_file-pdf\output\tai-lieu\tai-lieu.md" -TotalCount 30
```

## 7. Tom tat cach da thuc hien voi `nckh-c1.pdf`

- Tim file dau vao trong `z_file-pdf/scr`.
- Dung entry point co san `marker_single` cua repo Marker.
- Dat dau ra vao `z_file-pdf/output` va dinh dang `markdown`.
- Marker da tao Markdown 8,090 byte va metadata xac nhan 4 trang.
