# Quy trình làm việc của dự án

## Git và commit
- Dự án phải được quản lý bằng Git. Trước mỗi nhiệm vụ, kiểm tra `git status` để nhận diện thay đổi sẵn có.
- Sau mỗi nhiệm vụ hoặc nhóm thay đổi hoàn chỉnh, tạo commit gồm mã nguồn, test và tài liệu liên quan; nhiệm vụ chỉ khảo sát, phân tích hoặc kiểm tra vẫn phải ghi kết quả trong `docs/` và commit.
- Chỉ stage thay đổi thuộc nhiệm vụ hiện tại. Không ghi đè, hoàn tác hoặc commit thay đổi không liên quan của người dùng.
- Commit mô tả rõ mục đích, ưu tiên `feat:`, `fix:`, `test:`, `docs:` hoặc `chore:`.
- Không tự ý push, force-push, sửa lịch sử hoặc dùng lệnh Git có thể làm mất dữ liệu.
- Nếu không thể commit, báo nguyên nhân và yêu cầu thông tin/quyền cần thiết; không tự đặt danh tính hoặc sửa cấu hình Git toàn cục.

## Kiểm thử và xác minh
- Mỗi thay đổi mã nguồn hoặc hành vi phải có test mới hoặc cập nhật test liên quan.
- Thay đổi chỉ tài liệu hoặc cấu hình không cần test tự động phải giải thích lý do và xác minh phù hợp.
- Nếu có mã nguồn cần kiểm thử nhưng chưa có hệ thống test, thiết lập bộ test tối thiểu phù hợp; không tạo test giả.
- Trước commit và bàn giao, chạy test liên quan, toàn bộ test hiện có và lint, typecheck hoặc build phù hợp.
- Lệnh test dự án: `python -m unittest discover -s tests -v`, sau khi cài `python -m pip install -r requirements.txt`; nên dùng `.venv`.
- Chỉ xác nhận hoàn tất khi kiểm tra bắt buộc pass. Nếu không chạy được hoặc thất bại, ghi lệnh, kết quả và trở ngại trong `docs/`; không tuyên bố pass thiếu bằng chứng.
- Không vô hiệu hóa test, hạ tiêu chuẩn hoặc sửa phần không liên quan để có kết quả pass.
- Khi bị chặn, chỉ commit tiến độ chưa hoàn tất nếu người dùng cho phép; ghi rõ trạng thái trong nhật ký và thông điệp commit.

## Tài liệu trong docs/
- Luôn duy trì `docs/` tại gốc dự án. Mọi nhiệm vụ, kể cả thiết lập, khảo sát, phân tích, test và cấu hình, phải có nhật ký Markdown.
- Dùng tên `docs/YYYY-MM-DD-HHMMSS-mo-ta-ngan.md`; chỉ cập nhật nhật ký của chính nhiệm vụ đang tiếp tục.
- Ghi mục tiêu, việc đã làm, quyết định, tệp thay đổi, lệnh xác minh/test và kết quả thực tế, hạn chế và bước tiếp theo.
- Duy trì `docs/README.md` liên kết tới mọi nhật ký; không ghi bí mật hoặc dữ liệu nhạy cảm.
- Cập nhật nhật ký trong quá trình làm việc và trước commit; tổng hợp thao tác nhỏ theo nhiệm vụ. Tài liệu nằm cùng commit với thay đổi liên quan.
- Không ghi hash của chính commit vào tài liệu; báo hash khi bàn giao.

## Thiết lập và bàn giao
- Kiểm tra Git và gốc repository; chỉ `git init` khi chưa có repository. Hỏi trước khi tạo repository lồng trong repository cha.
- Kiểm tra và tái sử dụng `.gitignore`, `AGENTS.md`, mục lục và nhật ký đã có; chỉ bổ sung phần thiếu, không tạo nhật ký hoặc commit khởi tạo trùng lặp.
- Đọc và tuân thủ AGENTS.md hiện có; chỉ hợp nhất hướng dẫn khi không mâu thuẫn, hỏi người dùng nếu có mâu thuẫn.
- Ignore bí mật, token, API key, `.env`, dependencies, cache và build artifacts; `.env.example` chỉ được theo dõi nếu không chứa bí mật.
- Trước commit khởi tạo, kiểm tra AGENTS.md, mẫu ignore, liên kết tài liệu, diff và danh sách stage; chỉ stage tệp khởi tạo và dùng thông điệp `chore: initialize project workflow and documentation`.
- Nếu chưa có mã nguồn/test, ghi rõ và xác minh tệp thiết lập; không tạo test giả. Nếu quyền ghi/commit bị chặn, xin quyền qua môi trường và báo thao tác chưa thực hiện.
- Chỉ xác nhận thiết lập khi đã xác minh và commit thành công.
- Bàn giao tóm tắt thay đổi, trạng thái test/xác minh thực tế, liên kết nhật ký, hash và thông điệp commit, trở ngại hoặc phần chưa hoàn tất.
