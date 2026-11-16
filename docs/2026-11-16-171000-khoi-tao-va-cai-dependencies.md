# Khởi tạo quy trình và cài dependencies

## Mục tiêu
Cài requirements.txt theo README và bổ sung phần thiết lập còn thiếu theo hướng dẫn người dùng.

## Thực hiện và quyết định
- Git đã tồn tại tại đúng gốc dự án, trạng thái ban đầu sạch; không tạo repository mới hoặc sửa lịch sử.
- Có docs/ nhưng chưa có AGENTS.md, docs/README.md và nhật ký khởi tạo. Bổ sung các tệp này cùng mẫu ignore cho bí mật, dependencies, cache và build.
- Python hệ thống 3.14.8 chưa có pip. Tạo môi trường riêng bằng `python -m venv .venv`.
- Lần cài trong sandbox thất bại vì không phân giải được pypi.org. Chạy lại với quyền mạng được phê duyệt: `.venv/bin/python -m pip install --no-cache-dir -r requirements.txt`, thành công.
- Đã cài Pillow 13.0.0, jsonschema 4.26.0 và các phụ thuộc attrs 26.1.0, jsonschema-specifications 2025.9.1, referencing 0.37.0, rpds-py 2026.9.1.

## Tệp thay đổi
- AGENTS.md: quy trình Git, test, nhật ký và bàn giao.
- .gitignore: bổ sung mẫu loại trừ.
- docs/README.md và nhật ký này.
- .venv/ là môi trường cục bộ, được ignore; requirements.txt và mã nguồn không đổi.

## Xác minh
- `.venv/bin/python -m pip check`: No broken requirements found.
- `.venv/bin/python -m unittest discover -s tests -v`: 20 tests, 16.869 giây, OK.
- Không thêm test mới vì không thay đổi mã nguồn hoặc hành vi; đã chạy toàn bộ test hiện có theo CI. Dự án không cấu hình lint/typecheck/build riêng.
- `git diff --check`: thành công. Kiểm tra liên kết mục lục, mẫu ignore và danh sách stage trước commit để đảm bảo chỉ có tệp thiết lập thuộc nhiệm vụ.

## Hạn chế và bước tiếp theo
- Môi trường đã kiểm thử là Python 3.14.8; CI cấu hình Python 3.9 và 3.13, chưa chạy hai phiên bản đó tại máy này.
- Chạy `source .venv/bin/activate` trước các lệnh Python trong tài liệu.
- Không push; không đưa môi trường ảo vào Git.
