# Case study: Marker dừng ở `SpawnError: docker binary not found`

## Bối cảnh

Lệnh đã chạy tại thư mục gốc của repo:

```powershell
uv run marker_single "z_file-pdf\scr\He-dieu-hanh-2015.pdf" --output_dir "z_file-pdf\Output" --output_format markdown --mode balanced
```

PDF đầu vào có tồn tại. Lỗi xảy ra trước khi Marker đọc hoặc chuyển đổi nội dung PDF.

## Nguyên nhân đã xác nhận

Traceback đi đến `surya.inference.backends.vllm._resolve_docker_binary()` và dừng bằng `SpawnError: docker binary not found`.

`--mode balanced` yêu cầu backend vLLM của Surya để chạy layout/OCR trên GPU NVIDIA. Backend này tự khởi động bằng Docker. Kiểm tra trên máy cho kết quả:

```text
docker: NOT FOUND
GPU: NVIDIA GeForce RTX 2060 SUPER, 8192 MiB VRAM
```

Vì vậy lỗi hiện tại không phải do tên PDF, câu lệnh `marker_single`, `uv`, hay file đầu ra: Docker chưa sẵn sàng nên Surya không thể khởi động server GPU.

## Lưu ý quan trọng về RTX 2060 SUPER 8 GB

Không nên chỉ cài Docker rồi chạy lại lệnh `balanced`. Surya trong môi trường hiện tại đặt mặc định `VLLM_GPU_TYPE=4090`, VRAM 24 GB và `VLLM_DTYPE=bfloat16`. RTX 2060 SUPER là GPU Turing 8 GB; `bfloat16` không phù hợp, và 8 GB cũng thấp hơn mức thấp nhất (T4 16 GB) mà cấu hình vLLM của Surya hỗ trợ. Sau khi giải quyết Docker, khả năng cao sẽ gặp lỗi CUDA/vLLM hoặc hết VRAM.

## Cách chạy được ngay: PDF có lớp chữ tốt

Không cần Docker/GPU khi PDF chọn-copy được chữ đúng. Dùng chế độ không OCR; nó không khởi chạy server Surya:

```powershell
cd D:\Mega_SYNC\Code_Proeject\Github-and-VSCode\marker
uv run marker_single "z_file-pdf\scr\He-dieu-hanh-2015.pdf" --output_dir "z_file-pdf\Output" --output_format markdown --mode fast --disable_ocr
```

Kết quả là `z_file-pdf\Output\He-dieu-hanh-2015.md`. Kiểm tra nhanh:

```powershell
Get-Content "z_file-pdf\Output\He-dieu-hanh-2015.md" -Encoding utf8 -TotalCount 40
```

Nếu chữ tiếng Việt, bảng hoặc trang scan bị thiếu/sai, không dùng kết quả này làm bản cuối.

## Khi PDF là bản scan hoặc text lỗi

Để dùng `--force_ocr` hay `--mode balanced` tại chỗ, cần một GPU có VRAM phù hợp với Surya/vLLM (tối thiểu cấu hình T4 16 GB được repo liệt kê; 24 GB trở lên an toàn hơn), Docker đang chạy và GPU được Docker nhận diện.

Trên Windows, cần cài Docker Desktop, bật backend WSL 2 và xác nhận Docker hoạt động. Sau đó kiểm tra:

```powershell
docker version
docker run --rm --gpus all nvidia/cuda:12.4.1-base-ubuntu22.04 nvidia-smi
```

Chỉ khi hai lệnh trên thành công mới thử Marker GPU. Với RTX 2060 SUPER 8 GB, không khuyến nghị tiếp tục đường `balanced`; cần GPU mạnh hơn hoặc dùng một dịch vụ/máy khác có server Surya tương thích.

## Kết luận

Lỗi trực tiếp: thiếu Docker. Hạn chế kế tiếp: RTX 2060 SUPER 8 GB không đáp ứng cấu hình vLLM GPU hiện tại của repo. Với PDF có lớp text tốt, dùng ngay `--mode fast --disable_ocr`; với PDF scan, cần hạ tầng OCR khác hoặc GPU có VRAM lớn hơn.
