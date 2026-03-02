# Copilot Chat Conversation Export: Password strength checker in one file

**User:** @hkz4
**Thread URL:** https://github.com/copilot/c/341ad9c6-822e-4ffe-9a01-d7a49138ffb0

## @hkz4

Đề: Một script tự động kiểm tra xem độ mạnh của các mật khẩu trong một file text nhập vào có đạt chuẩn hay không (chuẩn: độ dài 8 kí tự, có đầy đủ chữ viết hoa, thường, số, kí tự đặc biệt), file do người dùng truyền vào input, output là số tài khoản thỏa mãn chuẩn, số tài khoản không thỏa mãn chuẩn và xuất 1 file excel chi tiết các tài khoản như vậy, trong file excel cũng có trường vì sao không thỏa mãn. Source code đẩy lên GitHub/GitLab kèm theo file README.md. File README phải hướng dẫn chi tiết cách chạy (run/deploy).Luồng hoạt động chính (có thể dùng một sơ đồ text đơn giản hoặc mô tả flow). Phải có thêm code để debug/ phát hiện lỗi nếu hoạt đôngj sai/ không hoạt động, lỗi cứ đưa hết vào file debug đấy. Viết bằng python để tối ưu, tài liệu Kiến trúc: Trình bày ngắn gọn (tối đa 1 trang):
Vấn đề bạn chọn giải quyết là gì?
Tại sao bạn chọn tech stack / công cụ này?. Nên xuất luôn word hoặc tương tự tối ưu để tôi làm 1 blog trên github

## Copilot

Tôi sẽ tạo đầy đủ source code và tài liệu cho bạn! Đây là toàn bộ project:

---

## 📁 Cấu trúc Project

```
password-checker/
├── main.py
├── checker.py
├── exporter.py
├── debug_logger.py
├── requirements.txt
├── README.md
├── ARCHITECTURE.md
└── sample_passwords.txt
```

---

### File 1: `debug_logger.py`

```python name=debug_logger.py
"""
debug_logger.py
---------------
Module xử lý logging/debug toàn bộ ứng dụng.
Mọi lỗi, cảnh báo, thông tin đều được ghi vào file debug.log
"""

import logging
import os
from datetime import datetime

LOG_FILE = "debug.log"

def setup_logger(name: str = "PasswordChecker") -> logging.Logger:
    """
    Khởi tạo logger với handler ghi ra file và console.
    
    Args:
        name: Tên logger
    Returns:
        logging.Logger instance đã được cấu hình
    """
    logger = logging.getLogger(name)
    
    # Tránh thêm handler trùng lặp nếu gọi lại
    if logger.handlers:
        return logger

    logger.setLevel(logging.DEBUG)

    # Formatter chi tiết
    formatter = logging.Formatter(
        fmt="%(asctime)s | %(levelname)-8s | %(name)s | %(funcName)s:%(lineno)d | %(message)s",
        datefmt="%Y-%m-%d %H:%M:%S"
    )

    # --- File Handler: lưu toàn bộ DEBUG trở lên ---
    file_handler = logging.FileHandler(LOG_FILE, encoding="utf-8", mode="a")
    file_handler.setLevel(logging.DEBUG)
    file_handler.setFormatter(formatter)

    # --- Console Handler: chỉ hiện INFO trở lên ---
    console_handler = logging.StreamHandler()
    console_handler.setLevel(logging.INFO)
    console_handler.setFormatter(formatter)

    logger.addHandler(file_handler)
    logger.addHandler(console_handler)

    # Ghi header phân cách mỗi lần chạy
    with open(LOG_FILE, "a", encoding="utf-8") as f:
        f.write(f"\n{'='*80}\n")
        f.write(f"  SESSION START: {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}\n")
        f.write(f"{'='*80}\n")

    return logger
```

---

### File 2: `checker.py`

```python name=checker.py
"""
checker.py
----------
Module kiểm tra độ mạnh mật khẩu theo các tiêu chí:
  - Độ dài >= 8 ký tự
  - Có chữ hoa (A-Z)
  - Có chữ thường (a-z)
  - Có chữ số (0-9)
  - Có ký tự đặc biệt (!@#$%^&*...)
"""

import re
from dataclasses import dataclass, field
from typing import List

from debug_logger import setup_logger

logger = setup_logger("Checker")

# ---------- Cấu hình tiêu chí ----------
MIN_LENGTH = 8
SPECIAL_CHARS_PATTERN = re.compile(r"[!@#$%^&*()\-_=+\[\]{};:'\",.<>?/\\|`~]")


@dataclass
class PasswordResult:
    """Kết quả kiểm tra một mật khẩu."""
    account: str          # Tên tài khoản (hoặc nhãn)
    password: str         # Mật khẩu gốc
    is_valid: bool = True
    reasons: List[str] = field(default_factory=list)

    def add_reason(self, reason: str):
        self.reasons.append(reason)
        self.is_valid = False

    @property
    def reason_text(self) -> str:
        return "; ".join(self.reasons) if self.reasons else "Đạt chuẩn"


def check_password(account: str, password: str) -> PasswordResult:
    """
    Kiểm tra một mật khẩu theo 5 tiêu chí.

    Args:
        account:  Tên tài khoản hoặc định danh
        password: Chuỗi mật khẩu cần kiểm tra

    Returns:
        PasswordResult chứa trạng thái hợp lệ và lý do (nếu không hợp lệ)
    """
    logger.debug(f"Kiểm tra tài khoản='{account}', độ dài mật khẩu={len(password)}")

    result = PasswordResult(account=account, password=password)

    try:
        # Tiêu chí 1: Độ dài
        if len(password) < MIN_LENGTH:
            msg = f"Độ dài chỉ {len(password)} ký tự (yêu cầu >= {MIN_LENGTH})"
            result.add_reason(msg)
            logger.debug(f"  ✗ {msg}")

        # Tiêu chí 2: Chữ hoa
        if not re.search(r"[A-Z]", password):
            msg = "Thiếu chữ hoa (A-Z)"
            result.add_reason(msg)
            logger.debug(f"  ✗ {msg}")

        # Tiêu chí 3: Chữ thường
        if not re.search(r"[a-z]", password):
            msg = "Thiếu chữ thường (a-z)"
            result.add_reason(msg)
            logger.debug(f"  ✗ {msg}")

        # Tiêu chí 4: Số
        if not re.search(r"[0-9]", password):
            msg = "Thiếu chữ số (0-9)"
            result.add_reason(msg)
            logger.debug(f"  ✗ {msg}")

        # Tiêu chí 5: Ký tự đặc biệt
        if not SPECIAL_CHARS_PATTERN.search(password):
            msg = "Thiếu ký tự đặc biệt (!@#$%^&*...)"
            result.add_reason(msg)
            logger.debug(f"  ✗ {msg}")

        if result.is_valid:
            logger.debug(f"  ✓ Mật khẩu đạt chuẩn")

    except Exception as e:
        logger.error(f"Lỗi không mong đợi khi kiểm tra '{account}': {e}", exc_info=True)
        result.add_reason(f"Lỗi hệ thống khi kiểm tra: {e}")

    return result


def check_file(filepath: str) -> List[PasswordResult]:
    """
    Đọc file text, mỗi dòng có dạng:  account:password
    Trả về danh sách PasswordResult.

    Args:
        filepath: Đường dẫn tới file txt chứa dữ liệu

    Returns:
        Danh sách kết quả kiểm tra
    Raises:
        FileNotFoundError: Nếu file không tồn tại
        ValueError:        Nếu file rỗng hoặc sai định dạng hoàn toàn
    """
    logger.info(f"Đọc file: {filepath}")
    results: List[PasswordResult] = []
    skipped = 0

    try:
        with open(filepath, "r", encoding="utf-8") as f:
            lines = f.readlines()
    except FileNotFoundError:
        logger.error(f"Không tìm thấy file: {filepath}")
        raise
    except UnicodeDecodeError as e:
        logger.error(f"Lỗi encoding khi đọc file '{filepath}': {e}", exc_info=True)
        raise

    if not lines:
        logger.warning(f"File '{filepath}' rỗng!")
        raise ValueError(f"File '{filepath}' không có nội dung.")

    for idx, raw_line in enumerate(lines, start=1):
        line = raw_line.strip()

        # Bỏ qua dòng trống và comment (#)
        if not line or line.startswith("#"):
            logger.debug(f"Dòng {idx}: bỏ qua (trống/comment)")
            skipped += 1
            continue

        # Tách account:password (tách tại dấu : đầu tiên)
        if ":" not in line:
            logger.warning(f"Dòng {idx} sai định dạng (thiếu ':'): '{line}' — bỏ qua")
            skipped += 1
            continue

        account, _, password = line.partition(":")

        if not account.strip():
            logger.warning(f"Dòng {idx}: tên tài khoản rỗng — bỏ qua")
            skipped += 1
            continue

        result = check_password(account.strip(), password)
        results.append(result)

    logger.info(
        f"Hoàn tất đọc file: {len(results)} tài khoản hợp lệ, {skipped} dòng bỏ qua"
    )

    if not results:
        raise ValueError("Không tìm thấy tài khoản hợp lệ nào trong file.")

    return results
```

---

### File 3: `exporter.py`

```python name=exporter.py
"""
exporter.py
-----------
Module xuất kết quả ra file Excel (.xlsx) sử dụng openpyxl.
File Excel gồm:
  - Sheet "Chi tiết": toàn bộ tài khoản, trạng thái, lý do
  - Sheet "Tóm tắt":  tổng số đạt / không đạt
"""

from pathlib import Path
from typing import List

import openpyxl
from openpyxl.styles import (
    Alignment, Font, PatternFill, Border, Side
)
from openpyxl.utils import get_column_letter

from checker import PasswordResult
from debug_logger import setup_logger

logger = setup_logger("Exporter")

# Màu sắc
COLOR_HEADER    = "2F5496"   # Xanh đậm header
COLOR_VALID     = "C6EFCE"   # Xanh lá nhạt — đạt
COLOR_INVALID   = "FFC7CE"   # Đỏ nhạt — không đạt
COLOR_SUMMARY_H = "D9E1F2"   # Xanh nhạt header tóm tắt
COLOR_WHITE     = "FFFFFF"


def _make_border() -> Border:
    thin = Side(style="thin", color="000000")
    return Border(left=thin, right=thin, top=thin, bottom=thin)


def _header_font() -> Font:
    return Font(name="Calibri", bold=True, color="FFFFFF", size=11)


def _cell_font() -> Font:
    return Font(name="Calibri", size=10)


def export_to_excel(results: List[PasswordResult], output_path: str) -> str:
    """
    Xuất danh sách kết quả ra file Excel.

    Args:
        results:     Danh sách PasswordResult
        output_path: Đường dẫn file Excel đầu ra (.xlsx)

    Returns:
        Đường dẫn tuyệt đối của file Excel đã tạo
    Raises:
        PermissionError: Nếu file đang được mở bởi chương trình khác
        Exception:       Mọi lỗi khác đều được log và re-raise
    """
    logger.info(f"Bắt đầu xuất Excel: {output_path}")

    try:
        wb = openpyxl.Workbook()

        # ── Sheet 1: Chi tiết ─────────────────────���───────────────────────
        ws_detail = wb.active
        ws_detail.title = "Chi tiết"

        headers = ["STT", "Tài khoản", "Mật khẩu", "Trạng thái", "Lý do không đạt"]
        col_widths = [6, 25, 30, 14, 60]

        # Ghi header
        for col_idx, (header, width) in enumerate(zip(headers, col_widths), start=1):
            cell = ws_detail.cell(row=1, column=col_idx, value=header)
            cell.font = _header_font()
            cell.fill = PatternFill("solid", fgColor=COLOR_HEADER)
            cell.alignment = Alignment(horizontal="center", vertical="center", wrap_text=True)
            cell.border = _make_border()
            ws_detail.column_dimensions[get_column_letter(col_idx)].width = width

        ws_detail.row_dimensions[1].height = 22

        # Ghi dữ liệu
        valid_count = 0
        invalid_count = 0

        for row_idx, r in enumerate(results, start=2):
            status = "✅ Đạt" if r.is_valid else "❌ Không đạt"
            row_color = COLOR_VALID if r.is_valid else COLOR_INVALID

            row_data = [row_idx - 1, r.account, r.password, status, r.reason_text]

            for col_idx, value in enumerate(row_data, start=1):
                cell = ws_detail.cell(row=row_idx, column=col_idx, value=value)
                cell.font = _cell_font()
                cell.fill = PatternFill("solid", fgColor=row_color)
                cell.alignment = Alignment(
                    horizontal="left" if col_idx > 1 else "center",
                    vertical="center",
                    wrap_text=True
                )
                cell.border = _make_border()

            ws_detail.row_dimensions[row_idx].height = 18

            if r.is_valid:
                valid_count += 1
            else:
                invalid_count += 1

        # Freeze header row
        ws_detail.freeze_panes = "A2"

        logger.debug(f"Sheet 'Chi tiết': {len(results)} dòng dữ liệu")

        # ── Sheet 2: Tóm tắt ──────────────────────────────────────────────
        ws_sum = wb.create_sheet("Tóm tắt")

        sum_headers = ["Chỉ số", "Số lượng", "Tỷ lệ (%)"]
        for col_idx, h in enumerate(sum_headers, start=1):
            cell = ws_sum.cell(row=1, column=col_idx, value=h)
            cell.font = Font(name="Calibri", bold=True, size=11)
            cell.fill = PatternFill("solid", fgColor=COLOR_HEADER)
            cell.font = _header_font()
            cell.alignment = Alignment(horizontal="center", vertical="center")
            cell.border = _make_border()

        total = len(results)
        pct_valid   = round(valid_count / total * 100, 2) if total else 0
        pct_invalid = round(invalid_count / total * 100, 2) if total else 0

        summary_data = [
            ("Tổng số tài khoản",          total,         "100%"),
            ("Số tài khoản ĐẠT chuẩn",    valid_count,   f"{pct_valid}%"),
            ("Số tài khoản KHÔNG ĐẠT",     invalid_count, f"{pct_invalid}%"),
        ]

        row_colors_sum = [COLOR_WHITE, COLOR_VALID, COLOR_INVALID]

        for row_idx, (label, count, pct) in enumerate(summary_data, start=2):
            for col_idx, value in enumerate([label, count, pct], start=1):
                cell = ws_sum.cell(row=row_idx, column=col_idx, value=value)
                cell.font = Font(name="Calibri", bold=(col_idx == 1), size=11)
                cell.fill = PatternFill("solid", fgColor=row_colors_sum[row_idx - 2])
                cell.alignment = Alignment(horizontal="center", vertical="center")
                cell.border = _make_border()

        ws_sum.column_dimensions["A"].width = 35
        ws_sum.column_dimensions["B"].width = 18
        ws_sum.column_dimensions["C"].width = 18
        ws_sum.row_dimensions[1].height = 20

        # Lưu file
        wb.save(output_path)
        abs_path = str(Path(output_path).resolve())
        logger.info(f"Xuất Excel thành công: {abs_path}")
        return abs_path

    except PermissionError as e:
        logger.error(f"Không thể ghi file '{output_path}' (đang mở?): {e}", exc_info=True)
        raise
    except Exception as e:
        logger.error(f"Lỗi khi xuất Excel: {e}", exc_info=True)
        raise
```

---

### File 4: `main.py`

```python name=main.py
"""
main.py
-------
Entry point chính của ứng dụng Password Strength Checker.

Luồng hoạt động:
  1. Nhận đường dẫn file từ tham số CLI hoặc nhập tay
  2. Gọi checker.check_file() → List[PasswordResult]
  3. In tóm tắt ra console
  4. Gọi exporter.export_to_excel() → file .xlsx
  5. Mọi lỗi đều được log vào debug.log
"""

import argparse
import sys
from datetime import datetime

from checker import check_file
from exporter import export_to_excel
from debug_logger import setup_logger

logger = setup_logger("Main")


def print_banner():
    banner = r"""
╔══════════════════════════════════════════════════════════╗
║          🔐  PASSWORD STRENGTH CHECKER  🔐               ║
║      Kiểm tra độ mạnh mật khẩu hàng loạt               ║
╚══════════════════════════════════════════════════════════╝
    """
    print(banner)


def parse_args() -> argparse.Namespace:
    parser = argparse.ArgumentParser(
        description="Kiểm tra độ mạnh mật khẩu từ file text.",
        formatter_class=argparse.RawTextHelpFormatter,
        epilog="""
Ví dụ:
  python main.py -i passwords.txt
  python main.py -i passwords.txt -o ket_qua.xlsx
  python main.py --input passwords.txt --output output/report.xlsx
        """
    )
    parser.add_argument(
        "-i", "--input",
        type=str,
        help="Đường dẫn tới file txt chứa mật khẩu (mỗi dòng: account:password)"
    )
    parser.add_argument(
        "-o", "--output",
        type=str,
        default=None,
        help="Đường dẫn file Excel đầu ra (mặc định: password_report_<timestamp>.xlsx)"
    )
    return parser.parse_args()


def get_input_path(args: argparse.Namespace) -> str:
    """Lấy đường dẫn file đầu vào từ CLI hoặc nhập tay."""
    if args.input:
        logger.debug(f"Input file từ CLI: {args.input}")
        return args.input
    print("\n📂 Nhập đường dẫn file chứa mật khẩu (vd: passwords.txt):")
    path = input("  > ").strip()
    if not path:
        logger.error("Người dùng không nhập đường dẫn file.")
        print("❌ Lỗi: Bạn chưa nhập đường dẫn file.")
        sys.exit(1)
    return path


def get_output_path(args: argparse.Namespace) -> str:
    """Tạo tên file Excel đầu ra."""
    if args.output:
        return args.output
    timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
    return f"password_report_{timestamp}.xlsx"


def print_summary(valid: int, invalid: int, total: int):
    """In tóm tắt kết quả ra console."""
    print("\n" + "─" * 55)
    print("  📊  KẾT QUẢ KIỂM TRA")
    print("─" * 55)
    print(f"  Tổng số tài khoản    : {total:>5}")
    print(f"  ✅ Đạt chuẩn         : {valid:>5}  ({valid/total*100:.1f}%)" if total else "")
    print(f"  ❌ Không đạt chuẩn   : {invalid:>5}  ({invalid/total*100:.1f}%)" if total else "")
    print("─" * 55)


def main():
    print_banner()
    logger.info("=== Khởi động ứng dụng ===")

    args = parse_args()

    # 1. Lấy đường dẫn file đầu vào
    input_path  = get_input_path(args)
    output_path = get_output_path(args)

    logger.info(f"File đầu vào : {input_path}")
    logger.info(f"File đầu ra  : {output_path}")

    # 2. Đọc & kiểm tra mật khẩu
    try:
        print(f"\n🔍 Đang đọc và kiểm tra file: {input_path}")
        results = check_file(input_path)
    except FileNotFoundError:
        print(f"❌ Lỗi: Không tìm thấy file '{input_path}'.")
        print("   Kiểm tra lại đường dẫn hoặc xem chi tiết trong debug.log")
        sys.exit(1)
    except ValueError as e:
        print(f"❌ Lỗi dữ liệu: {e}")
        print("   Xem chi tiết trong debug.log")
        sys.exit(1)
    except Exception as e:
        logger.critical(f"Lỗi không mong đợi khi đọc file: {e}", exc_info=True)
        print(f"❌ Lỗi nghiêm trọng: {e}\n   Xem debug.log để biết chi tiết.")
        sys.exit(1)

    # 3. Thống kê
    valid_count   = sum(1 for r in results if r.is_valid)
    invalid_count = len(results) - valid_count
    print_summary(valid_count, invalid_count, len(results))

    # 4. Xuất Excel
    try:
        print(f"\n📤 Đang xuất file Excel...")
        abs_path = export_to_excel(results, output_path)
        print(f"✅ Xuất thành công: {abs_path}")
    except PermissionError:
        print(f"❌ Không thể ghi file '{output_path}'.")
        print("   Hãy đóng file Excel nếu đang mở, hoặc chọn tên file khác.")
        sys.exit(1)
    except Exception as e:
        logger.critical(f"Lỗi khi xuất Excel: {e}", exc_info=True)
        print(f"❌ Lỗi xuất Excel: {e}\n   Xem debug.log để biết chi tiết.")
        sys.exit(1)

    logger.info("=== Ứng dụng kết thúc thành công ===")
    print("\n🎉 Hoàn tất! Xem chi tiết log tại: debug.log\n")


if __name__ == "__main__":
    main()
```

---

### File 5: `requirements.txt`

```text name=requirements.txt
openpyxl>=3.1.2
```

---

### File 6: `sample_passwords.txt`

```text name=sample_passwords.txt
# Định dạng: account:password
# Dòng bắt đầu bằng # là comment, sẽ bị bỏ qua

alice:Abc@1234
bob:password123
charlie:SHORT1!
david:alllowercase1!
eve:ALLUPPERCASE1!
frank:NoNumber@Here
grace:N0Special
henry:Abcdefg1!
ivan:Tr0ub4dor&3
judy:weak
```

---

### File 7: `README.md`

````markdown name=README.md
# 🔐 Password Strength Checker

> **Tự động kiểm tra độ mạnh mật khẩu hàng loạt và xuất báo cáo Excel chi tiết.**

---

## 📋 Mục lục

- [Giới thiệu](#giới-thiệu)
- [Luồng hoạt động](#luồng-hoạt-động)
- [Yêu cầu hệ thống](#yêu-cầu-hệ-thống)
- [Cài đặt](#cài-đặt)
- [Cách sử dụng](#cách-sử-dụng)
- [Định dạng file đầu vào](#định-dạng-file-đầu-vào)
- [File đầu ra](#file-đầu-ra)
- [Debug & Xử lý lỗi](#debug--xử-lý-lỗi)
- [Cấu trúc project](#cấu-trúc-project)

---

## 🎯 Giới thiệu

**Password Strength Checker** là công cụ dòng lệnh (CLI) viết bằng Python, giúp:

- ✅ Kiểm tra hàng loạt mật khẩu từ file `.txt`
- ✅ Phát hiện mật khẩu yếu theo 5 tiêu chí bảo mật
- ✅ Xuất báo cáo Excel với màu sắc trực quan
- ✅ Ghi log debug đầy đủ để phân tích lỗi

### Tiêu chí đánh giá mật khẩu

| # | Tiêu chí | Ví dụ |
|---|----------|-------|
| 1 | Độ dài ≥ 8 ký tự | `AbCd1234!` (9 ký tự ✅) |
| 2 | Có ít nhất 1 chữ **HOA** (A-Z) | `A...` ✅ |
| 3 | Có ít nhất 1 chữ **thường** (a-z) | `a...` ✅ |
| 4 | Có ít nhất 1 **chữ số** (0-9) | `1...` ✅ |
| 5 | Có ít nhất 1 **ký tự đặc biệt** | `!@#$%^&*...` ✅ |

---

## 🔄 Luồng hoạt động

```
 Người dùng
     │
     ▼
[Nhập file .txt]
     │
     ▼
 main.py ──► parse_args() ──► get đường dẫn file
     │
     ▼
 checker.py ──► check_file(filepath)
     │              │
     │              ├─ Đọc từng dòng  (account:password)
     │              ├─ check_password() × N lần
     │              │       ├─ Kiểm tra độ dài
     │              │       ├─ Kiểm tra chữ hoa
     │              │       ├─ Kiểm tra chữ thường
     │              │       ├─ Kiểm tra số
     │              │       └─ Kiểm tra ký tự đặc biệt
     │              └─ Trả về List[PasswordResult]
     │
     ▼
 main.py ──► In tóm tắt ra console
     │         (Tổng | Đạt | Không đạt)
     │
     ▼
 exporter.py ──► export_to_excel()
     │              ├─ Sheet "Chi tiết": từng tài khoản
     │              └─ Sheet "Tóm tắt": thống kê tổng
     │
     ▼
 File .xlsx + debug.log
```

---

## 💻 Yêu cầu hệ thống

| Yêu cầu | Phiên bản |
|---------|-----------|
| Python  | ≥ 3.8     |
| pip     | ≥ 21.0    |
| OS      | Windows / macOS / Linux |

---

## ⚙️ Cài đặt

### Bước 1: Clone repository

```bash
git clone https://github.com/<your-username>/password-checker.git
cd password-checker
```

### Bước 2: Tạo môi trường ảo (khuyến nghị)

```bash
# Windows
python -m venv venv
venv\Scripts\activate

# macOS / Linux
python3 -m venv venv
source venv/bin/activate
```

### Bước 3: Cài đặt dependencies

```bash
pip install -r requirements.txt
```

---

## 🚀 Cách sử dụng

### Cách 1: Truyền file qua tham số CLI (khuyến nghị)

```bash
python main.py -i passwords.txt
```

Tự chỉ định file output:

```bash
python main.py -i passwords.txt -o bao_cao.xlsx
```

### Cách 2: Nhập tay khi chạy

```bash
python main.py
# Chương trình sẽ hỏi: "Nhập đường dẫn file..."
```

### Xem help

```bash
python main.py --help
```

### Kết quả console mẫu

```
╔══════════════════════════════════════════════════════════╗
║          🔐  PASSWORD STRENGTH CHECKER  🔐               ║
╚══════════════════════════════════════════════════════════╝

🔍 Đang đọc và kiểm tra file: passwords.txt

───────────────────────────────────────────────────────
  📊  KẾT QUẢ KIỂM TRA
───────────────────────────────────────────────────────
  Tổng số tài khoản    :    10
  ✅ Đạt chuẩn         :     3  (30.0%)
  ❌ Không đạt chuẩn   :     7  (70.0%)
───────────────────────────────────────────────────────

📤 Đang xuất file Excel...
✅ Xuất thành công: /home/user/project/password_report_20260302_153000.xlsx
```

---

## 📄 Định dạng file đầu vào

File `.txt`, mỗi dòng theo định dạng:

```
account:password
```

**Ví dụ:**

```
# Đây là comment
alice:Abc@1234
bob:password123
charlie:Tr0ub4dor&3
```

> ⚠️ Lưu ý:
> - Dòng bắt đầu bằng `#` → bỏ qua (comment)
> - Dòng trống → bỏ qua
> - Dòng thiếu dấu `:` → cảnh báo, bỏ qua
> - Mật khẩu có thể chứa dấu `:` (chỉ tách tại dấu `:` đầu tiên)

---

## 📊 File đầu ra

### File Excel (`password_report_<timestamp>.xlsx`)

**Sheet 1 — Chi tiết:**

| STT | Tài khoản | Mật khẩu | Trạng thái | Lý do không đạt |
|-----|-----------|----------|-----------|-----------------|
| 1 | alice | Abc@1234 | ✅ Đạt | Đạt chuẩn |
| 2 | bob | password123 | ❌ Không đạt | Thiếu chữ hoa; Thiếu ký tự đặc biệt |

- 🟢 Màu xanh lá = Đạt chuẩn
- 🔴 Màu đỏ nhạt = Không đạt

**Sheet 2 — Tóm tắt:**

| Chỉ số | Số lượng | Tỷ lệ |
|--------|---------|-------|
| Tổng số tài khoản | 10 | 100% |
| Số tài khoản ĐẠT chuẩn | 3 | 30% |
| Số tài khoản KHÔNG ĐẠT | 7 | 70% |

---

## 🐛 Debug & Xử lý lỗi

Mọi hoạt động đều được ghi vào **`debug.log`**:

```bash
# Xem log
cat debug.log        # Linux/macOS
type debug.log       # Windows
```

**Ví dụ nội dung debug.log:**

```
================================================================================
  SESSION START: 2026-03-02 15:30:00
================================================================================
2026-03-02 15:30:00 | INFO     | Main | main:67 | === Khởi động ứng dụng ===
2026-03-02 15:30:00 | INFO     | Checker | check_file:78 | Đọc file: passwords.txt
2026-03-02 15:30:00 | DEBUG    | Checker | check_password:42 | Kiểm tra tài khoản='bob', độ dài=11
2026-03-02 15:30:00 | DEBUG    | Checker | check_password:55 |   ✗ Thiếu chữ hoa (A-Z)
```

**Các mã lỗi thường gặp:**

| Lỗi | Nguyên nhân | Giải pháp |
|-----|-------------|-----------|
| `FileNotFoundError` | File không tồn tại | Kiểm tra đường dẫn |
| `UnicodeDecodeError` | File không phải UTF-8 | Lưu lại file dưới dạng UTF-8 |
| `PermissionError` | File Excel đang mở | Đóng file Excel và chạy lại |
| `ValueError` | File rỗng hoặc sai định dạng | Kiểm tra định dạng `account:password` |

---

## 📁 Cấu trúc project

```
password-checker/
├── main.py              # Entry point, CLI, điều phối luồng
├── checker.py           # Logic kiểm tra mật khẩu
├── exporter.py          # Xuất kết quả ra Excel
├── debug_logger.py      # Cấu hình logging/debug
├── requirements.txt     # Thư viện cần thiết
├── sample_passwords.txt # File mẫu để test
├── README.md            # Tài liệu này
└── ARCHITECTURE.md      # Tài liệu kiến trúc
```
````

---

### File 8: `ARCHITECTURE.md`

````markdown name=ARCHITECTURE.md
# 🏗️ Tài liệu Kiến trúc — Password Strength Checker

---

## 1. Vấn đề được giải quyết

Trong môi trường doanh nghiệp hoặc quản trị hệ thống, việc kiểm tra thủ công
hàng trăm mật khẩu để đảm bảo chúng đạt chuẩn bảo mật là **tốn thời gian
và dễ sai sót**. Công cụ này tự động hóa quá trình đó: nhận một file danh
sách tài khoản/mật khẩu, kiểm tra theo 5 tiêu chí bảo mật, và xuất báo cáo
trực quan ngay lập tức — thay thế hoàn toàn việc kiểm tra tay.

---

## 2. Tech Stack & Lý do lựa chọn

| Thành phần | Công nghệ | Lý do |
|------------|-----------|-------|
| **Ngôn ngữ** | Python 3.8+ | Dễ đọc, thư viện phong phú, chạy đa nền tảng |
| **Xuất Excel** | `openpyxl` | Thuần Python, không cần cài Office, hỗ trợ định dạng/màu sắc |
| **Regex** | `re` (stdlib) | Kiểm tra pattern ký tự mạnh mẽ, không cần lib ngoài |
| **Logging** | `logging` (stdlib) | Chuẩn công nghiệp, không phụ thuộc thư viện bên ngoài |
| **CLI** | `argparse` (stdlib) | Tích hợp sẵn, tạo help message tự động |

> **Triết lý**: Tối thiểu hóa dependency — chỉ có `openpyxl` là thư viện
> bên ngoài duy nhất. Toàn bộ còn lại dùng Python standard library.

---

## 3. Kiến trúc module

```
┌─────────────────────────────────────────────────────────┐
│                      main.py                            │
│  (Điều phối: CLI → checker → exporter → output)        │
└────────────┬────────────────────────┬───────────────────┘
             │                        │
             ▼                        ▼
    ┌─────────────────┐     ┌──────────────────┐
    │   checker.py    │     │   exporter.py    │
    │  (Business      │     │  (Xuất Excel     │
    │   Logic)        │     │   2 sheets)      │
    └────────┬────────┘     └────────┬─────────┘
             │                       │
             └──────────┬────────────┘
                        ▼
              ┌──────────────────┐
              │ debug_logger.py  │
              │ (Logging tập     │
              │  trung → file)   │
              └──────────────────┘
```

**Nguyên tắc thiết kế:**
- **Single Responsibility**: Mỗi module có một trách nhiệm duy nhất
- **Dependency Injection**: Logger được inject qua `setup_logger()`, không hard-code
- **Fail-fast + Graceful Degradation**: Lỗi nghiêm trọng dừng sớm, lỗi dòng đơn lẻ chỉ cảnh báo

---

## 4. Quyết định thiết kế nổi bật

**Dùng `dataclass` cho `PasswordResult`** thay vì `dict`:
→ Type-safe, IDE autocomplete, dễ mở rộng thêm trường mới.

**Tách `reason_text` khỏi `is_valid`**:
→ Một mật khẩu có thể vi phạm nhiều tiêu chí cùng lúc; ghi hết lý do giúp
người dùng sửa một lần thay vì phải chạy lại nhiều lần.

**Log mỗi session có header phân cách**:
→ Dễ tìm kiếm khi file log tích lũy qua nhiều lần chạy.
````

---

## 🚀 Cách đẩy lên GitHub

```bash name=deploy.sh
# 1. Khởi tạo repo
git init
git add .
git commit -m "feat: initial release - password strength checker"

# 2. Tạo repo trên GitHub rồi push
git remote add origin https://github.com/<your-username>/password-checker.git
git branch -M main
git push -u origin main
```

---

## 📋 Tóm tắt project

| Thành phần | File | Vai trò |
|-----------|------|---------|
| 🚪 Entry point | `main.py` | CLI, điều phối toàn bộ luồng |
| 🔍 Logic kiểm tra | `checker.py` | 5 tiêu chí, dataclass kết quả |
| 📊 Xuất báo cáo | `exporter.py` | Excel 2 sheet có màu sắc |
| 🐛 Debug | `debug_logger.py` | Ghi log tập trung ra `debug.log` |
| 📄 Tài liệu | `README.md` | Hướng dẫn đầy đủ |
| 🏗️ Kiến trúc | `ARCHITECTURE.md` | 1 trang kỹ thuật — phù hợp làm blog |
| 📦 Deps | `requirements.txt` | Chỉ 1 lib ngoài: `openpyxl` |
| 🧪 Mẫu test | `sample_passwords.txt` | 10 tài khoản mẫu đa dạng |

> 💡 **Tip blog**: Dùng `ARCHITECTURE.md` làm nội dung bài viết kỹ thuật trên GitHub Pages — nó đã được viết theo đúng format 1 trang, có bảng so sánh tech stack và sơ đồ kiến trúc ASCII.

## @hkz4

sao nhiều file thực thi vậy, chỉ 1 file check.py bao gồm check và export kết quả, nếu có lỗi xuất debug.txt để biết lỗi

## Copilot

Gọn lại thành **2 file thôi**: `check.py` (toàn bộ logic) + `README.md` (tài liệu).

---

### Cấu trúc tối giản

```
password-checker/
├── check.py               ← File duy nhất chạy
├── requirements.txt
├── sample_passwords.txt
└── README.md
```

---

### `check.py` — File duy nhất: đọc, kiểm tra, xuất Excel, ghi debug

```python name=check.py
"""
check.py
--------
Kiểm tra độ mạnh mật khẩu từ file txt và xuất báo cáo Excel.

Tiêu chí hợp lệ:
  1. Độ dài >= 8 ký tự
  2. Có chữ hoa (A-Z)
  3. Có chữ thường (a-z)
  4. Có chữ số (0-9)
  5. Có ký tự đặc biệt (!@#$%^&*...)

Cách dùng:
  python check.py passwords.txt
  python check.py passwords.txt output.xlsx
  python check.py             ← sẽ hỏi đường dẫn file
"""

import re
import sys
import logging
import traceback
from datetime import datetime
from pathlib import Path

# ── Thư viện bên ngoài (pip install openpyxl) ──────────────────────────────
try:
    import openpyxl
    from openpyxl.styles import Font, PatternFill, Alignment, Border, Side
    from openpyxl.utils import get_column_letter
except ImportError:
    print("[LỖI] Thiếu thư viện 'openpyxl'. Chạy: pip install openpyxl")
    sys.exit(1)


# ════════════════════════════════════════════════════════════════════════════
# 1. CẤU HÌNH DEBUG LOGGER
# ════════════════════════════════════════════════════════════════════════════

DEBUG_FILE = "debug.txt"

def _setup_logger() -> logging.Logger:
    """Tạo logger ghi mọi thứ vào debug.txt, chỉ INFO ra console."""
    logger = logging.getLogger("check")
    if logger.handlers:          # tránh thêm handler trùng khi import lại
        return logger
    logger.setLevel(logging.DEBUG)

    fmt = logging.Formatter(
        "%(asctime)s | %(levelname)-8s | %(funcName)s:%(lineno)d | %(message)s",
        datefmt="%Y-%m-%d %H:%M:%S",
    )

    # File handler — DEBUG trở lên
    fh = logging.FileHandler(DEBUG_FILE, encoding="utf-8", mode="a")
    fh.setLevel(logging.DEBUG)
    fh.setFormatter(fmt)

    # Console handler — INFO trở lên
    ch = logging.StreamHandler()
    ch.setLevel(logging.INFO)
    ch.setFormatter(fmt)

    logger.addHandler(fh)
    logger.addHandler(ch)

    # Phân cách session trong file log
    with open(DEBUG_FILE, "a", encoding="utf-8") as f:
        f.write(f"\n{'='*70}\n")
        f.write(f"  SESSION: {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}\n")
        f.write(f"{'='*70}\n")

    return logger

log = _setup_logger()


# ══════════════���═════════════════════════════════════════════════════════════
# 2. KIỂM TRA MẬT KHẨU
# ════════════════════════════════════════════════════════════════════════════

MIN_LEN       = 8
RE_UPPER      = re.compile(r"[A-Z]")
RE_LOWER      = re.compile(r"[a-z]")
RE_DIGIT      = re.compile(r"[0-9]")
RE_SPECIAL    = re.compile(r"[!@#$%^&*()\-_=+\[\]{};:'\",.<>?/\\|`~]")


def check_password(account: str, password: str) -> dict:
    """
    Kiểm tra một mật khẩu theo 5 tiêu chí.

    Returns:
        dict {
            "account":  str,
            "password": str,
            "valid":    bool,
            "reasons":  str   (rỗng nếu hợp lệ)
        }
    """
    log.debug(f"Kiểm tra: account='{account}' | len={len(password)}")
    reasons = []

    if len(password) < MIN_LEN:
        reasons.append(f"Độ dài {len(password)} < {MIN_LEN}")

    if not RE_UPPER.search(password):
        reasons.append("Thiếu chữ HOA")

    if not RE_LOWER.search(password):
        reasons.append("Thiếu chữ thường")

    if not RE_DIGIT.search(password):
        reasons.append("Thiếu chữ số")

    if not RE_SPECIAL.search(password):
        reasons.append("Thiếu ký tự đặc biệt")

    is_valid = (len(reasons) == 0)
    log.debug(f"  → {'ĐẠT' if is_valid else 'KHÔNG ĐẠT'}: {reasons}")

    return {
        "account":  account,
        "password": password,
        "valid":    is_valid,
        "reasons":  "; ".join(reasons) if reasons else "",
    }


def read_and_check(filepath: str) -> list[dict]:
    """
    Đọc file txt (định dạng account:password mỗi dòng),
    kiểm tra từng mật khẩu, trả về list kết quả.

    Raises:
        FileNotFoundError, UnicodeDecodeError, ValueError
    """
    log.info(f"Đọc file: {filepath}")
    results = []
    skipped = 0

    try:
        lines = Path(filepath).read_text(encoding="utf-8").splitlines()
    except FileNotFoundError:
        log.error(f"Không tìm thấy file: '{filepath}'")
        raise
    except UnicodeDecodeError as e:
        log.error(f"Lỗi encoding file '{filepath}': {e}", exc_info=True)
        raise

    for i, raw in enumerate(lines, 1):
        line = raw.strip()

        # Bỏ qua dòng trống và comment
        if not line or line.startswith("#"):
            log.debug(f"Dòng {i}: bỏ qua (trống/comment)")
            skipped += 1
            continue

        if ":" not in line:
            log.warning(f"Dòng {i} sai định dạng (thiếu ':'): '{line}'")
            skipped += 1
            continue

        account, _, password = line.partition(":")

        if not account.strip():
            log.warning(f"Dòng {i}: tên tài khoản rỗng")
            skipped += 1
            continue

        results.append(check_password(account.strip(), password))

    log.info(f"Kết quả: {len(results)} tài khoản, {skipped} dòng bỏ qua")

    if not results:
        raise ValueError(f"Không tìm thấy tài khoản hợp lệ nào trong '{filepath}'.")

    return results


# ════════════════════════════════════════════════════════════════════════════
# 3. XUẤT EXCEL
# ════════════════════════════════════════════════════════════════════════════

C_HEADER  = "2F5496"   # Xanh đậm
C_VALID   = "C6EFCE"   # Xanh lá nhạt
C_INVALID = "FFC7CE"   # Đỏ nhạt
C_SUM_H   = "D9E1F2"   # Xanh nhạt tóm tắt


def _border():
    s = Side(style="thin", color="000000")
    return Border(left=s, right=s, top=s, bottom=s)

def _hfont():
    return Font(name="Calibri", bold=True, color="FFFFFF", size=11)

def _dfont(bold=False):
    return Font(name="Calibri", size=10, bold=bold)

def _fill(color):
    return PatternFill("solid", fgColor=color)

def _align(h="left"):
    return Alignment(horizontal=h, vertical="center", wrap_text=True)


def export_excel(results: list[dict], output_path: str) -> str:
    """
    Xuất danh sách kết quả ra file Excel 2 sheet:
      - "Chi tiết": từng tài khoản
      - "Tóm tắt":  thống kê

    Returns:
        Đường dẫn tuyệt đối file Excel vừa tạo.
    """
    log.info(f"Xuất Excel: {output_path}")

    wb = openpyxl.Workbook()

    # ── Sheet 1: Chi tiết ────────────────────────────────────────────────
    ws = wb.active
    ws.title = "Chi tiết"

    headers   = ["STT", "Tài khoản", "Mật khẩu", "Trạng thái", "Lý do không đạt"]
    col_widths = [6,      22,          28,          14,            55]

    for ci, (h, w) in enumerate(zip(headers, col_widths), 1):
        c = ws.cell(row=1, column=ci, value=h)
        c.font      = _hfont()
        c.fill      = _fill(C_HEADER)
        c.alignment = _align("center")
        c.border    = _border()
        ws.column_dimensions[get_column_letter(ci)].width = w
    ws.row_dimensions[1].height = 22

    valid_count = invalid_count = 0

    for ri, r in enumerate(results, 2):
        is_valid = r["valid"]
        color    = C_VALID if is_valid else C_INVALID
        status   = "✅ Đạt" if is_valid else "❌ Không đạt"
        row_data = [ri - 1, r["account"], r["password"], status,
                    r["reasons"] if r["reasons"] else "Đạt chuẩn"]

        for ci, val in enumerate(row_data, 1):
            c = ws.cell(row=ri, column=ci, value=val)
            c.font      = _dfont()
            c.fill      = _fill(color)
            c.alignment = _align("center" if ci in (1, 4) else "left")
            c.border    = _border()
        ws.row_dimensions[ri].height = 18

        if is_valid:
            valid_count += 1
        else:
            invalid_count += 1

    ws.freeze_panes = "A2"   # Cố định header khi cuộn

    # ── Sheet 2: Tóm tắt ─────────────────────────────────────────────────
    ws2 = wb.create_sheet("Tóm tắt")
    total = len(results)

    sum_headers = ["Chỉ số", "Số lượng", "Tỷ lệ (%)"]
    for ci, h in enumerate(sum_headers, 1):
        c = ws2.cell(row=1, column=ci, value=h)
        c.font      = _hfont()
        c.fill      = _fill(C_HEADER)
        c.alignment = _align("center")
        c.border    = _border()

    rows = [
        ("Tổng số tài khoản",        total,         "100%"),
        ("✅ Số tài khoản ĐẠT",       valid_count,   f"{valid_count/total*100:.1f}%"),
        ("❌ Số tài khoản KHÔNG ĐẠT", invalid_count, f"{invalid_count/total*100:.1f}%"),
    ]
    row_colors = ["FFFFFF", C_VALID, C_INVALID]

    for ri, (label, cnt, pct) in enumerate(rows, 2):
        for ci, val in enumerate([label, cnt, pct], 1):
            c = ws2.cell(row=ri, column=ci, value=val)
            c.font      = _dfont(bold=(ci == 1))
            c.fill      = _fill(row_colors[ri - 2])
            c.alignment = _align("center")
            c.border    = _border()

    for col, w in zip(["A", "B", "C"], [35, 18, 16]):
        ws2.column_dimensions[col].width = w

    wb.save(output_path)
    abs_path = str(Path(output_path).resolve())
    log.info(f"Xuất thành công: {abs_path}")
    return abs_path


# ════════════════════════════════════════════════════════════════════════════
# 4. MAIN
# ════════════════════════════════════════════════════════════════════════════

def main():
    print("=" * 55)
    print("  🔐 PASSWORD STRENGTH CHECKER")
    print("=" * 55)

    # ── Lấy đường dẫn file đầu vào ──────────────────────────────────────
    if len(sys.argv) >= 2:
        input_path = sys.argv[1]
    else:
        print("\nNhập đường dẫn file mật khẩu (vd: passwords.txt):")
        input_path = input("  > ").strip()
        if not input_path:
            log.error("Người dùng không nhập đường dẫn.")
            print("❌ Lỗi: chưa nhập đường dẫn file.")
            sys.exit(1)

    # ── Lấy đường dẫn file Excel đầu ra ─────────────────────────────────
    if len(sys.argv) >= 3:
        output_path = sys.argv[2]
    else:
        ts = datetime.now().strftime("%Y%m%d_%H%M%S")
        output_path = f"password_report_{ts}.xlsx"

    log.info(f"input='{input_path}' | output='{output_path}'")

    # ── Đọc & kiểm tra ───────────────────────────────────────────────────
    try:
        print(f"\n🔍 Đọc và kiểm tra: {input_path}")
        results = read_and_check(input_path)
    except FileNotFoundError:
        print(f"❌ Không tìm thấy file '{input_path}'. Xem debug.txt để biết thêm.")
        sys.exit(1)
    except ValueError as e:
        print(f"❌ Dữ liệu không hợp lệ: {e}")
        sys.exit(1)
    except Exception as e:
        log.critical(f"Lỗi không mong đợi:\n{traceback.format_exc()}")
        print(f"❌ Lỗi nghiêm trọng: {e}. Xem debug.txt.")
        sys.exit(1)

    # ── In tóm tắt console ───────────────────────────────────────────────
    total         = len(results)
    valid_count   = sum(1 for r in results if r["valid"])
    invalid_count = total - valid_count

    print("\n" + "─" * 45)
    print(f"  Tổng tài khoản    : {total}")
    print(f"  ✅ Đạt chuẩn      : {valid_count}  ({valid_count/total*100:.1f}%)")
    print(f"  ❌ Không đạt      : {invalid_count}  ({invalid_count/total*100:.1f}%)")
    print("─" * 45)

    # ── Xuất Excel ───────────────────────────────────────────────────────
    try:
        print(f"\n📤 Xuất Excel...")
        abs_path = export_excel(results, output_path)
        print(f"✅ Đã lưu: {abs_path}")
    except PermissionError:
        print(f"❌ Không ghi được '{output_path}' (file đang mở?). Xem debug.txt.")
        sys.exit(1)
    except Exception as e:
        log.critical(f"Lỗi xuất Excel:\n{traceback.format_exc()}")
        print(f"❌ Lỗi xuất Excel: {e}. Xem debug.txt.")
        sys.exit(1)

    print(f"\n📋 Log chi tiết: debug.txt\n")


if __name__ == "__main__":
    main()
```

---

### `requirements.txt`

```text name=requirements.txt
openpyxl>=3.1.2
```

---

### `sample_passwords.txt`

```text name=sample_passwords.txt
# Định dạng mỗi dòng: account:password
alice:Abc@1234
bob:password123
charlie:SHORT1!
david:alllowercase1!
eve:ALLUPPERCASE1!
frank:NoNumber@Here
grace:N0Special
henry:Abcdefg1!
ivan:Tr0ub4dor&3
judy:weak
```

---

### `README.md`

````markdown name=README.md
# 🔐 Password Strength Checker

Tự động kiểm tra độ mạnh mật khẩu hàng loạt từ file `.txt` và xuất báo cáo Excel.

---

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

## Kiến trúc & Tech stack

**Vấn đề giải quyết:** Kiểm tra thủ công hàng loạt mật khẩu tốn thời gian, dễ bỏ sót — công cụ này tự động hóa hoàn toàn.

| Thành phần | Lý do chọn |
|------------|-----------|
| **Python** | Đa nền tảng, dễ đọc, stdlib mạnh |
| **openpyxl** | Xuất Excel không cần cài Office, hỗ trợ màu sắc/định dạng |
| **re + logging** | Có sẵn trong stdlib, không phụ thuộc thêm |

> Toàn bộ logic gói gọn trong **1 file `check.py`** — dễ copy, dễ deploy.
````

---

## Tóm tắt thay đổi

| Trước | Sau |
|-------|-----|
| 5 file `.py` | **1 file `check.py`** duy nhất |
| `debug.log` | **`debug.txt`** |
| Module riêng lẻ | 4 section rõ ràng trong 1 file: **Config logger → Check → Export → Main** |

Chạy ngay:
```bash
pip install openpyxl
python check.py sample_passwords.txt
```
