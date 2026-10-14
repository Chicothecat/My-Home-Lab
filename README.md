# My-Home-Lab
Home Lab Detail AI for learning
# RTK + Codex CLI + Claude Code + Antigravity — Windows Cheat Sheet

## 1. Phân biệt các công cụ
- **RTK**: CLI lọc/nén output của lệnh shell trước khi AI đọc; không tự viết code.
- **Codex CLI**: AI coding agent chạy trong terminal.
- **Claude Code CLI**: AI coding agent chạy trong terminal.
- **Antigravity**: IDE tích hợp agent, quản lý file và notebook trực quan.
- RTK chỉ tối ưu các lệnh/đường thực thi được hook hỗ trợ, không tự động giảm mọi token. Thống kê token RTK là ước lượng.

## 2. Trạng thái thiết lập trên máy
- RTK v0.50.0: đã cài bằng `winget install rtk-ai.rtk`.
- Node.js v24.20.0, npm 11.19.0: đã hoạt động.
- Claude Code: `rtk init -g` đã tạo hook, RTK.md, CLAUDE.md, settings.json. Chưa xác nhận đã chạy Claude Code CLI.
- Codex: `rtk init -g --codex` đã đăng ký hook và AGENTS.md; Codex CLI đã mở được.
- Antigravity: chưa xác nhận hook RTK đã được cài.

## 3. Mở project và Codex
PowerShell:
```powershell
cd C:\Users\ADMIN\Paper_1\git_data
codex.cmd
```
Trong Codex, gõ yêu cầu bằng ngôn ngữ tự nhiên. Ví dụ:
> Phân tích kiến trúc mô hình trong các notebook, tóm tắt pipeline huấn luyện. Chỉ đọc, không sửa file và không chạy GPU.

Để thoát Codex: `/exit` (hoặc Ctrl+C khi phù hợp).

Mở project khác: thoát Codex, `cd` vào thư mục mới, chạy `codex.cmd` lại.

## 4. Cài và cấu hình RTK
```powershell
winget install rtk-ai.rtk
rtk --version
rtk --help
rtk init -g                   # Claude Code
rtk init -g --codex           # Codex CLI
rtk init --show               # Kiểm tra cấu hình
```

**Antigravity:** README hiện tại của RTK nêu:
```powershell
rtk init --agent antigravity       # Cấu hình theo project
rtk init -g --agent antigravity    # Cấu hình toàn cục
```
Trước khi chạy trên RTK v0.50.0, kiểm tra `rtk init --help` xem phiên bản đã hỗ trợ `antigravity` hay chưa; nếu chưa, cập nhật và kiểm tra lại. Khởi động lại agent sau khi cài hook.

## 5. Lệnh RTK thường dùng
| Việc | Lệnh |
|---|---|
| Phiên bản | `rtk --version` |
| Hướng dẫn | `rtk --help` |
| Trạng thái Git | `rtk git status` |
| Thay đổi Git | `rtk git diff` |
| Lịch sử Git | `rtk git log -n 10` |
| Đọc file | `rtk read main.py` |
| Tìm trong file | `rtk grep "TODO" .` (cần công cụ shell tương ứng) |
| Chạy pytest | `rtk pytest` (cần pytest trong môi trường) |
| Bỏ lọc | `rtk bypass git status` |
| Thống kê | `rtk gain` |
| Lịch sử thống kê | `rtk gain --history` |
| Biểu đồ | `rtk gain --graph` |
| Phát hiện cơ hội tối ưu | `rtk discover` |
| Kiểm tra session | `rtk session` |
| Tắt telemetry | `rtk telemetry disable` |

Lưu ý: PowerShell `ls` là alias, không phải chương trình `ls.exe`; `rtk ls .` có thể lỗi trên Windows native. Dùng Git Bash/WSL khi cần lệnh Unix. `rtk git status` cần đứng trong Git repository.

## 6. Giao việc cho Codex
- **Đọc hiểu**: `Hãy đọc các notebook trong project, giải thích input, preprocessing, mô hình và metrics; không sửa file.`
- **Sửa bug**: `Tìm nguyên nhân lỗi trong train.py, đề xuất bản vá tối thiểu; trước khi sửa hãy nêu file sẽ thay đổi.`
- **Review code**: `Review pipeline theo mức độ nghiêm trọng, tập trung data leakage, train/test split, random seed và metric.`
- **Tối ưu**: `Đề xuất tối ưu RAM/GPU cho DataLoader, mixed precision và batch size; chưa chạy huấn luyện.`
- **Viết test**: `Tạo unit test nhỏ cho preprocessing, không tải dataset lớn.`

## 7. Codex và Antigravity: dùng khi nào?
- **Codex CLI**: hợp với thao tác hàng loạt trên repo, refactor, test, git, chạy command, review bằng prompt.
- **Antigravity**: hợp khi cần trực tiếp quan sát/sửa notebook, xem cell, kernel, kết quả thí nghiệm, hoặc thao tác IDE.
- Cả hai đều có thể đọc/sửa file khi được cấp quyền. RTK không quyết định agent nào mạnh hơn.

## 8. Kiểm tra hook thực sự chạy
1. Mở lại Codex/Claude Code/Antigravity sau khi cấu hình.
2. Yêu cầu agent kiểm tra Git status qua terminal (không sửa file).
3. Xem lệnh agent thực thi và chạy `rtk gain --history`.
4. Nếu không có lịch sử, chưa đủ cơ sở kết luận hook đã hoạt động; xem cấu hình và log.
5. Nếu RTK lược mất thông tin debug, chạy lệnh gốc hoặc `rtk bypass`.

## 9. Lỗi phổ biến
- `rtk not recognized`: khởi động lại terminal/IDE để cập nhật PATH.
- `node not recognized`: khởi động lại IDE sau cài Node.js.
- `codex not recognized`: cài `npm.cmd install -g @openai/codex`, rồi thử `codex.cmd`.
- `Not a git repository`: `cd` vào thư mục chứa `.git`.
- `rtk ls .` báo không thấy `ls`: dùng Git Bash/WSL hoặc lệnh tương thích native Windows.
- `SQLITE_CANTOPEN`: chỉ cân nhắc sửa quyền writable_roots của Codex khi thực sự gặp lỗi; giữ cấu hình sandbox hiện có.

## 10. Tham khảo
- https://github.com/rtk-ai/rtk
- https://www.rtk-ai.app/guide
- https://github.com/rtk-ai/rtk/blob/develop/INSTALL.md
