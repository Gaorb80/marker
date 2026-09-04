# Ghi chú nghiên cứu: cấu hình máy cho OCR PDF với Marker

Mục đích: lưu lại đánh giá cấu hình hiện tại và các hướng nâng cấp khi cần chuyển PDF sang văn bản, đặc biệt với PDF scan.

## Cấu hình đã ghi nhận

- Windows 10 Pro 64-bit (build 19045)
- CPU: AMD Ryzen 5 5600X, 6 nhân / 12 luồng
- RAM: 24 GB; tại thời điểm kiểm tra còn khoảng 6 GB trống
- GPU: NVIDIA GeForce RTX 2060 SUPER, 8 GB VRAM

## Kết luận cho Marker

CPU hiện tại vẫn phù hợp cho chuyển PDF có sẵn lớp chữ. Điểm giới hạn là GPU 8 GB: chế độ `balanced` của Marker dùng Surya/vLLM và môi trường repo hiện có cấu hình GPU thấp nhất ở mốc 16 GB VRAM (T4); mặc định còn giả định GPU 24 GB (`VLLM_GPU_TYPE=4090`). Vì vậy không nên cố ép RTX 2060 SUPER chạy `balanced` bằng cách sửa biến môi trường.

Với PDF có thể chọn/copy được chữ đúng, dùng lệnh CPU sau. Lệnh không cần Docker hoặc GPU:

```powershell
cd D:\Mega_SYNC\Code_Proeject\Github-and-VSCode\marker
uv run marker_single "z_file-pdf\scr\He-dieu-hanh-2015.pdf" --output_dir "z_file-pdf\Output" --output_format markdown --mode fast --disable_ocr
```

Không dùng `--disable_ocr` cho PDF scan; các trang ảnh, công thức và nội dung không có lớp chữ sẽ bị bỏ qua hoặc nhận không đầy đủ.

## Tối ưu trước khi nâng cấp

1. Đóng ứng dụng ngốn RAM (trình duyệt nhiều tab, IDE, máy ảo/WSL, game) hoặc khởi động lại trước khi xử lý PDF dài.
2. Để page file Windows ở chế độ System managed trên SSD/NVMe. Nó hỗ trợ khi RAM gần đầy nhưng không thay thế VRAM GPU.
3. Xử lý thử một dải trang để đánh giá đầu ra: thêm `--page_range "0-19"` vào lệnh Marker. Số trang bắt đầu từ 0.
4. Không chạy nhiều tiến trình Marker đồng thời trên máy này.

## Lộ trình nâng cấp nếu thường OCR PDF scan

- Bước 1: nâng RAM lên 32 GB DDR4 bằng kit đồng bộ 2 x 16 GB. Điều này giúp Windows, Docker/WSL và PDF dài ổn định hơn, nhưng không làm GPU 8 GB chạy được vLLM.
- Bước 2: nếu cần OCR GPU cục bộ, chọn GPU từ 16 GB VRAM trở lên. RTX A4000 có 16 GB VRAM và công suất tối đa 140 W; phù hợp khi ưu tiên điện năng/kích thước. GPU 24 GB như RTX 3090 cho biên độ VRAM tốt hơn cho Marker/vLLM.
- Trước khi chọn RTX 3090, kiểm tra nguồn tối thiểu 750 W, hai dây PCIe 8-pin riêng, case có 3 slot trống và đủ chỗ cho card dài khoảng 313 mm.

Sau khi có GPU phù hợp, cài Docker Desktop với backend WSL 2 và bảo đảm Docker truy cập được GPU NVIDIA. Khi đó mới chạy:

```powershell
uv run marker_single "z_file-pdf\scr\He-dieu-hanh-2015.pdf" --output_dir "z_file-pdf\Output" --output_format markdown --mode balanced --force_ocr
```

## Tài liệu liên quan

- `CASE_STUDY_LOI_MARKER_DOCKER_GPU.md`: chẩn đoán lỗi Docker/vLLM đã gặp.
- `HUONG_DAN_CHUYEN_PDF_SANG_VAN_BAN_GPU.md`: các lệnh chuyển đổi bằng repo Marker.
