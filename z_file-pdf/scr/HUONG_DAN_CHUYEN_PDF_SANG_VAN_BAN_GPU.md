# Chuyển `He-dieu-hanh-2015.pdf` thành văn bản bằng Marker

PDF đã ở đúng thư mục `z_file-pdf\scr`. Mở PowerShell tại thư mục gốc của repo `marker`:

```powershell
cd D:\Mega_SYNC\Code_Proeject\Github-and-VSCode\marker
```

Chạy lệnh sau để chuyển PDF sang Markdown - đây là tệp văn bản có giữ tiêu đề, đoạn, bảng, công thức và thứ tự nội dung:

```powershell
uv run marker_single "z_file-pdf\scr\He-dieu-hanh-2015.pdf" --output_dir "z_file-pdf\Output" --output_format markdown --mode balanced
```

Kết quả nằm trong thư mục `z_file-pdf\Output`. Liệt kê và mở kết quả:

```powershell
Get-ChildItem "z_file-pdf\Output" -File
notepad "z_file-pdf\Output\He-dieu-hanh-2015.md"
```

## Dùng GPU NVIDIA để nhanh và chính xác

`--mode balanced` là chế độ tốt nhất khi có GPU phù hợp: Marker dùng mô hình Surya để nhận bố cục và OCR. Lần đầu chạy, repo tự chuẩn bị môi trường qua `uv` và tải model cần thiết.

Trước khi chạy, kiểm tra GPU:

```powershell
nvidia-smi
```

Theo README của Marker, GPU NVIDIA cần Docker Desktop đang chạy và NVIDIA Container Toolkit (trong môi trường Linux/WSL2 của Docker). Surya sẽ tự khởi động máy chủ vLLM ở lần chuyển đổi đầu tiên. Không cần tự cài PaddleOCR hoặc viết script OCR riêng. Xem case study kèm theo nếu nhận lỗi thiếu Docker hoặc dùng RTX 2060 SUPER 8 GB.

Nếu máy có nhiều GPU, chọn GPU số 0 bằng PowerShell rồi chạy lại lệnh chuyển đổi:

```powershell
$env:VLLM_GPUS = "0"
uv run marker_single "z_file-pdf\scr\He-dieu-hanh-2015.pdf" --output_dir "z_file-pdf\Output" --output_format markdown --mode balanced
```

Để dùng nhiều GPU, ví dụ GPU 0 và 1:

```powershell
$env:VLLM_GPUS = "0,1"
```

Không cần đặt `SURYA_INFERENCE_PARALLEL`: Marker tự điều chỉnh số yêu cầu song song theo năng lực GPU. Chỉ đặt biến này khi bạn đã đo đạc và muốn ghi đè mặc định.

## Chọn lệnh theo tình trạng PDF

PDF sách scan hoặc chữ copy ra bị lỗi - ép OCR toàn bộ trang:

```powershell
uv run marker_single "z_file-pdf\scr\He-dieu-hanh-2015.pdf" --output_dir "z_file-pdf\Output" --output_format markdown --mode balanced --force_ocr
```

PDF có sẵn lớp chữ tốt và cần tốc độ tối đa - không chạy OCR/VLM:

```powershell
uv run marker_single "z_file-pdf\scr\He-dieu-hanh-2015.pdf" --output_dir "z_file-pdf\Output" --output_format markdown --mode fast --disable_ocr
```

Không dùng `--disable_ocr` cho PDF scan. Nếu kết quả có chữ sai do lớp OCR cũ của PDF, dùng `--force_ocr` (hoặc `--strip_existing_ocr` khi muốn giữ text gốc nhưng loại phần OCR cũ).

## Kiểm tra và xử lý lỗi

Xem 40 dòng đầu:

```powershell
Get-Content "z_file-pdf\Output\He-dieu-hanh-2015.md" -Encoding utf8 -TotalCount 40
```

Nếu lỗi hết bộ nhớ GPU, chỉ chạy một lệnh Marker tại một thời điểm; với PDF dài, dùng `--page_range` để làm thử hoặc chia nhỏ, ví dụ:

```powershell
uv run marker_single "z_file-pdf\scr\He-dieu-hanh-2015.pdf" --output_dir "z_file-pdf\Output" --output_format markdown --mode balanced --page_range "0-19"
```

Chú ý: số trang của `--page_range` bắt đầu từ `0`. Khi cần chẩn đoán bố cục/OCR, thêm `--debug`; Marker sẽ lưu ảnh trang và dữ liệu bổ sung trong đầu ra.
