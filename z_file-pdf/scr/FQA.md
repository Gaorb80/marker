# FQA - JSON trong Marker

## 1. Tại sao Marker tạo file JSON?

JSON là định dạng dữ liệu có cấu trúc. Marker dùng JSON để lưu thông tin mà Markdown không thể thể hiện đầy đủ, ví dụ:

- Số trang và thống kê xử lý từng trang.
- Mục lục được phát hiện.
- Loại block: tiêu đề, đoạn văn, bảng, ảnh, công thức, danh sách.
- Vị trí của block trên trang bằng tọa độ polygon.
- Quan hệ cha-con giữa các block.
- HTML và dữ liệu ảnh khi cần xử lý tiếp bằng chương trình.

Vì vậy JSON phù hợp hơn Markdown khi cần tìm kiếm, phân tích, RAG, hiển thị lại bố cục, hoặc trích xuất riêng bảng/ảnh.

## 2. Hai loại JSON cần phân biệt

### A. File `_meta.json` đi kèm Markdown

Lệnh mặc định với `--output_format markdown` tạo Markdown và một file metadata:

```text
z_file-pdf/Output/nckh-c1.md
```

File `_meta.json` không phải là toàn bộ văn bản. Nó là thông tin mô tả kết quả xử lý.

### B. JSON output của Marker

Nếu chạy:

```powershell
uv run marker_single "z_file-pdf\scr\nckh-c1.pdf" --output_dir "z_file-pdf\output-json" --output_format json
```

Marker sẽ xuất nội dung tài liệu thành các block JSON có cấu trúc cây. Mỗi trang là một block, bên trong có các block con như tiêu đề, đoạn văn, bảng hoặc ảnh.

## 3. Ví dụ file JSON vừa tạo

File thực tế là:

```text
z_file-pdf/Output/nckh-c1_meta.json
```

Phần rút gọn từ file này:

```json
{
  "table_of_contents": [
    {
      "title": "Chương 1\nGIỚI THIỆU CHUNG VỀ NGHIÊN CỨU KHOA HỌC",
      "heading_level": null,
      "page_id": 0,
      "polygon": [
        [174.67, 53.19],
        [534.97, 53.19],
        [534.97, 91.52],
        [174.67, 91.52]
      ]
    }
  ],
  "page_stats": [
    {
      "page_id": 0,
      "text_extraction_method": "pdftext",
      "block_counts": [
        ["Span", 200]
      ]
    }
  ],
  "debug_data_path": null
}
```

Ý nghĩa:

- `table_of_contents`: mục lục Marker phát hiện.
- `title`: tên tiêu đề.
- `heading_level`: cấp tiêu đề; giá trị `null` nghĩa là Marker chưa gán cấp trong kết quả này.
- `page_id`: số trang bắt đầu, tính từ 0; `0` là trang đầu tiên.
- `polygon`: bốn góc vùng chứa tiêu đề trên trang, theo thứ tự theo chiều kim đồng hồ.
- `page_stats`: thống kê xử lý từng trang.
- `text_extraction_method`: cách lấy text, ở đây là `pdftext`.
- `block_counts`: số lượng block theo loại.
- `debug_data_path`: đường dẫn dữ liệu debug nếu bật `--debug`; không có thì có thể là `null`.

Trong file thực tế của `nckh-c1.pdf`, `page_stats` có 4 phần tử, xác nhận tài liệu có 4 trang.

## 4. Ví dụ JSON output theo tác giả Marker

README của tác giả mô tả JSON output là một danh sách các trang. Mỗi trang có thể có các trường:

```json
{
  "id": "/page/0/Page/1",
  "block_type": "Page",
  "html": "<content-ref src='/page/0/SectionHeader/0'></content-ref>",
  "polygon": [
    [0.0, 0.0],
    [612.0, 0.0],
    [612.0, 792.0],
    [0.0, 792.0]
  ],
  "children": [
    {
      "id": "/page/0/SectionHeader/0",
      "block_type": "SectionHeader",
      "html": "<h1>Introduction</h1>",
      "polygon": [
        [100.0, 80.0],
        [300.0, 80.0],
        [300.0, 105.0],
        [100.0, 105.0]
      ],
      "children": null,
      "section_hierarchy": {
        "1": "/page/0/SectionHeader/0"
      },
      "images": {}
    }
  ]
}
```

Theo README của tác giả:

- `id`: ID duy nhất của block.
- `block_type`: loại block, ví dụ `Page`, `Text`, `SectionHeader`, `Table`, `Figure`.
- `html`: nội dung HTML của block; các thẻ `content-ref` trỏ đến block con.
- `polygon`: vị trí block trên trang.
- `children`: các block con, tạo thành cấu trúc cây.
- `section_hierarchy`: block đang thuộc mục nào.
- `images`: ảnh được mã hóa base64 theo ID block.

README cung cấp các ví dụ JSON của `Think Python`, `Switch Transformers` và `Multi-column CNN` trong bảng Examples của repo Marker.

## 5. Khi nào nên dùng Markdown, khi nào nên dùng JSON?

- Dùng Markdown khi mục tiêu là đọc, sửa, tìm kiếm và lưu vào Obsidian.
- Dùng JSON khi cần lập trình đọc kết quả, giữ tọa độ/layout, tách bảng/ảnh, hoặc đưa dữ liệu vào RAG.
- Dùng `--output_format chunks` khi muốn danh sách block phẳng, để chia nhỏ dữ liệu cho RAG.
- Không cần xuất JSON nếu chỉ muốn văn bản để đọc; Markdown đã đủ và ít phức tạp hơn.

## 6. Lệnh tham khảo

Markdown và metadata:

```powershell
uv run marker_single "z_file-pdf\scr\file.pdf" --output_dir "z_file-pdf\Output" --output_format markdown
```

JSON đầy đủ:

```powershell
uv run marker_single "z_file-pdf\scr\file.pdf" --output_dir "z_file-pdf\output-json" --output_format json
```

Chunks cho RAG:

```powershell
uv run marker_single "z_file-pdf\scr\file.pdf" --output_dir "z_file-pdf\output-chunks" --output_format chunks
```

## 7. Nguồn trong repo

- README phần `JSON`: mô tả cấu trúc cây và các trường của JSON output.
- README phần `Metadata`: mô tả `table_of_contents` và `page_stats`.
- README phần `Examples`: liên kết các file JSON mẫu của tác giả Marker.
