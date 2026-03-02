# 🔐 Password Strength Checker

Tự động kiểm tra độ mạnh mật khẩu hàng loạt từ file `.txt` và xuất báo cáo Excel.

---
## Kiến trúc & Tech stack

**Vấn đề giải quyết:** Kiểm tra thủ công hàng loạt mật khẩu tốn thời gian, dễ bỏ sót — công cụ này tự động hóa hoàn toàn.

| Thành phần | Lý do chọn |
|------------|-----------|
| **Python** | Đa nền tảng, dễ đọc, stdlib mạnh |
| **openpyxl** | Xuất Excel không cần cài Office, hỗ trợ màu sắc/định dạng |
| **re + logging** | Có sẵn trong stdlib, không phụ thuộc thêm |

## Tiêu chí đánh giá

| # | Tiêu chí |
|---|----------|
| 1 | Độ dài ≥ 8 ký tự |
| 2 | Có chữ hoa (A-Z) |
| 3 | Có chữ thường (a-z) |
| 4 | Có chữ số (0-9) |
| 5 | Có ký tự đặc biệt (`!@#$%^&*...`) |

---

## Luồng hoạt động

```
File txt (account:password)
        │
        ▼
   check.py
        ├── Đọc từng dòng → check_password() × N
        ├── In tóm tắt ra console
        ├── export_excel() → file .xlsx (2 sheet)
        └── Ghi log chi tiết → debug.txt
```

---

## Cài đặt & Chạy

```bash
# 1. Cài thư viện
pip install -r requirements.txt

# 2. Chạy (truyền file qua tham số)
python check.py passwords.txt

# 3. Tự chỉ định tên file Excel output
python check.py passwords.txt bao_cao.xlsx

# 4. Không truyền tham số → chương trình hỏi đường dẫn
python check.py
```

---

## Định dạng file đầu vào

```
# Dòng comment (bỏ qua)
alice:Abc@1234
bob:password123
```

> Mỗi dòng: `taikhoan:matkhau` — tách tại dấu `:` đầu tiên.

---

## File đầu ra

| File | Nội dung |
|------|---------|
| `password_report_<timestamp>.xlsx` | Sheet **Chi tiết** (màu xanh/đỏ) + Sheet **Tóm tắt** |
| `debug.txt` | Log DEBUG đầy đủ, ghi lỗi nếu có |


---
---

## Demo
<img width="1765" height="502" alt="image" src="https://github.com/user-attachments/assets/efb2f733-37b2-45c2-8981-efd0432d149b" />



<img width="1238" height="699" alt="image" src="https://github.com/user-attachments/assets/e53eb0fe-acd1-4f73-974a-4a0ff81f6af4" />

---
