# BÁO CÁO PHÂN TÍCH HIỆU NĂNG HỆ THỐNG VÀ THIẾT KẾ CẤU TRÚC LƯU TRỮ

---

## PHẦN 1: PHÂN TÍCH NGUYÊN NHÂN CHẬM LẠC VÀ MÔ HÌNH IPO

### 1. Chẩn đoán nguyên nhân gây giật lag (Task Manager)

Khi mở 20 tab Chrome cùng lúc với ứng dụng VS Code trên máy tính có 8 GB RAM:

* **Tải RAM (% Memory):** Đạt ngưỡng $90\% - 98\%$. Trình duyệt Chrome là ứng dụng ngốn dung lượng bộ nhớ lớn do mỗi tab chạy như một tiến trình riêng biệt.
* **Tải Disk / SSD (% Disk):** Đột ngột tăng vọt lên $90\% - 100\%$.
* **Hiện tượng Swap File (Virtual Memory):** Khi RAM không đủ sức chứa thêm dữ liệu từ các tab Chrome và VS Code, hệ điều hành Windows kích hoạt cơ chế bộ nhớ ảo (*Pagefile.sys* / *Swap File*). HĐH bắt buộc phải dùng một phần ổ cứng SSD để làm "RAM tạm thời". 
* **Hậu quả:** Tốc độ đọc/ghi của SSD (kể cả SSD NVMe) vẫn chậm hơn RAM nhiều lần. Việc CPU phải liên tục đợi ghi/đọc dữ liệu qua lại giữa RAM và SSD khiến toàn bộ hệ thống bị đứng (freeze).

---

### 2. Sự phối hợp CPU - RAM - Storage trong mô hình IPO

```
+------------------+         +--------------------------+         +------------------+
|   INPUT / READ   |         |      PROCESS & RUN       |         |  OUTPUT / WRITE  |
|                  |         |                          |         |                  |
|  Storage (SSD)   |  ---->  |  RAM (Bàn làm việc)      |  ---->  |  Storage (SSD)   |
|  (Tủ hồ sơ)      |  load   |  CPU (Người xử lý)       |  save   |  (Lưu lâu dài)   |
+------------------+         +--------------------------+         +------------------+
```

#### So sánh Bàn làm việc (RAM) và Ngăn tủ (Storage):

| Tiêu chí | RAM (Bàn làm việc) | Storage / SSD (Ngăn tủ / Tủ tài liệu) |
| :--- | :--- | :--- |
| **Vai trò IPO** | Nơi lưu tạm dữ liệu **đang xử lý** cho CPU truy xuất cực nhanh. | Nơi **lưu trữ lâu dài** dữ liệu tĩnh trước và sau khi xử lý. |
| **Đặc tính** | Bộ nhớ bốc hơi (*Volatile*) – Mất dữ liệu khi tắt máy/mất điện. | Bộ nhớ bền vững (*Non-volatile*) – Dữ liệu tồn tại lâu dài. |
| **Tình huống bài toán** | Bàn làm việc 8GB quá nhỏ, chất đầy 20 tài liệu (Chrome) + 1 bộ dụng cụ (VS Code) $\rightarrow$ Không còn chỗ làm việc. | Ngăn tủ chứa tài liệu gốc, nhưng việc CPU cứ phải chạy ra tủ lấy từng tờ giấy (Swap file) làm lãng phí thời gian. |

---

### 3. Cảnh báo về vị trí lưu trữ `C:\Windows\System32\...`

* **Nguy cơ bảo mật & Quyền truy cập:** Thư mục `System32` là trái tim của Windows. Lưu file tại đây yêu cầu quyền Administrator, dễ gây lỗi phân quyền khi Python tạo virtualenv hoặc ghi file logs/cache.
* **Nguy cơ mất dữ liệu:** Khi dọn dẹp hệ thống hoặc cài lại Windows, toàn bộ thư mục này sẽ bị ghi đè hoặc xóa sạch.
* **Rủi ro đường dẫn:** Đường dẫn chứa khoảng trắng (`Code của tôi`) gây lỗi cú pháp khi thực thi lệnh Terminal / CLI trong Python.

---

## PHẦN 2: THIẾT KẾ CẤU TRÚC LƯU TRỮ CHUẨN TRÊN Ổ D:

Cấu trúc cây thư mục dưới đây chuyển toàn bộ dự án sang phân vùng `D:`, tuân thủ **Quy tắc 3 cấp**, **4 trụ cột quản lý dự án** (Source Code, Assets, Docs, Tests/Builds) và chuẩn đặt tên **kebab-case** (chữ viết thường, phân cách bằng dấu gạch ngang `-`, không dấu, không khoảng trắng).

```text
D:/
└── ptit-projects/                                   <-- Cấp 1: Thư mục gốc dự án PTIT
    └── python-app/                                  <-- Cấp 2: Tên dự án cụ thể (kebab-case)
        ├── assets/                                  <-- Cấp 3: Tài nguyên tĩnh
        │   ├── images/
        │   │   └── logo.png
        │   └── data/
        │       └── raw-sample.json
        ├── docs/                                    <-- Cấp 3: Tài liệu dự án & báo cáo
        │   ├── report.md
        │   └── architecture-diagram.png
        ├── src/                                     <-- Cấp 3: Mã nguồn chính (Source code)
        │   ├── __init__.py
        │   ├── main.py
        │   └── utils.py
        ├── tests/                                   <-- Cấp 3: Kiểm thử tự động
        │   └── test-main.py
        ├── .env.example                             <-- File mẫu cấu hình môi trường
        ├── .gitignore                               <-- Cấu hình chặn đẩy file thừa/mật
        ├── README.md                                <-- Hướng dẫn chạy dự án
        └── requirements.txt                         <-- Danh sách thư viện Python
```

### Quy tắc áp dụng:
1. **Phân vùng an toàn:** Lưu hoàn toàn trên đĩa `D:`, tách biệt với ổ hệ thống `C:`.
2. **Kebab-case Naming:** `ptit-projects`, `python-app`, `raw-sample.json`, `test-main.py` giúp tránh lỗi đứt gãy đường dẫn trên các HĐH khác nhau (Linux/macOS/Windows).
3. **Mô-đun hóa rõ ràng:** Giúp Python dễ dàng định vị đường dẫn tương đối bằng module `pathlib` hoặc `os`.