# Hướng dẫn đo token và dự trù chi phí AI Coding trên Windows

> Dành cho Codex CLI, Claude Code, Gemini CLI và người dùng Antigravity / VS Code.  
> Công cụ chính: [Tokenometer](https://github.com/faraa2m/tokenometer). Công cụ bổ trợ: [RTK](https://github.com/rtk-ai/rtk) và [ccusage](https://github.com/ccusage/ccusage).

## 1. Phân biệt các công cụ

| Công cụ | Vai trò | Có tự sửa code? |
|---|---|---|
| Codex CLI / Claude Code / Gemini CLI | AI Agent đọc, phân tích, sửa file, chạy lệnh | Có, tùy quyền |
| Antigravity | IDE có AI Agent và terminal | Có, qua Agent |
| RTK | Rút gọn output terminal cho Agent để giảm token ngữ cảnh | Không |
| Tokenometer | Đếm token prompt và ước tính chi phí theo đơn giá API | Không |
| ccusage | Thống kê usage/chi phí từ log Agent mà nó hỗ trợ | Không |

**Lưu ý:** Tokenometer **không** biết trước toàn bộ số token của một tác vụ Agent nhiều bước. Nó chủ yếu đo **nội dung prompt/file được đưa vào**; output, reasoning, tool calls, file Agent đọc thêm và số vòng sửa code đều cần dự trù riêng. Chi phí API quy đổi không đồng nghĩa với tiền bị trừ từ gói thuê bao ChatGPT/Claude/Gemini.

## 2. Yêu cầu hệ thống

- Windows 10/11, PowerShell.
- Node.js và npm đã cài. Kiểm tra:

```powershell
node -v
npm.cmd -v
```

Nếu chưa cài Node.js:

```powershell
winget install OpenJS.NodeJS.LTS
```

Đóng/mở lại terminal sau khi cài.

## 3. Cài Tokenometer

```powershell
npm.cmd install -g tokenometer
```

Kiểm tra:

```powershell
tokenometer.cmd --help
```

Nếu lệnh không được nhận diện, hãy đóng/mở lại PowerShell và kiểm tra npm global path:

```powershell
npm.cmd config get prefix
```

Không cần API key để dùng chế độ **offline** (ước lượng). Tên model được hỗ trợ có thể thay đổi theo phiên bản; luôn kiểm tra `--help`.

## 4. Tạo prompt mẫu và tính chi phí trước khi chạy

Đi đến project:

```powershell
cd C:\Users\ADMIN\Paper_1\git_data
```

Tạo file prompt:

```powershell
@"
Analyze my PyTorch Transformer architecture.
Identify potential bugs and suggest improvements.
Do not modify files or run GPU training.
"@ | Set-Content -Encoding utf8 prompt.md
```

Ước tính token và chi phí đầu vào của prompt:

```powershell
tokenometer.cmd .\prompt.md --model gpt-4o
```

So sánh GPT và Claude (model ID minh họa theo README):

```powershell
tokenometer.cmd .\prompt.md --model gpt-4o,claude-sonnet-4-6
```

Chỉ định một định dạng để kết quả dễ đọc:

```powershell
tokenometer.cmd .\prompt.md --model gpt-4o --format text
```

Xem danh sách model ID / tùy chọn hiện có:

```powershell
tokenometer.cmd --help
```

**Gemini:** chọn đúng model ID Google được liệt kê trong phiên bản Tokenometer đang cài. Không nên tự đoán model ID vì danh mục thay đổi theo bản phát hành.

### Chế độ đo chính xác hơn (tùy chọn)

Theo tài liệu dự án, Tokenometer có `--empirical`: Anthropic và Google có thể dùng API đếm token chính thức khi có API key; OpenAI dùng tokenizer offline. Đọc hướng dẫn của dự án trước khi cấu hình khóa. **Không đưa API key vào Git, prompt.md hoặc ảnh chụp terminal.**

```powershell
tokenometer.cmd .\prompt.md --model claude-sonnet-4-6 --empirical --max-spend 0.05
```

Lệnh này chỉ có ý nghĩa khi môi trường đã cấu hình API key phù hợp. Không cần thực hiện nếu chỉ muốn dự trù offline.

## 5. Dự trù tổng chi phí của một tác vụ Agent

Công thức API đơn giản (chưa tính cache, phụ phí và mức giá đặc biệt):

```text
Chi phí USD ≈ (input_tokens × giá_input_mỗi_1M
             + output_tokens × giá_output_mỗi_1M) / 1,000,000
```

Với Agent nhiều lượt:

```text
Tổng chi phí ≈ tổng chi phí của tất cả lượt model gọi
```

**Ví dụ giả định:** Một tác vụ gồm 3 lượt, mỗi lượt 100.000 input token và 10.000 output token. Nếu giá giả định là $2/1M input và $10/1M output:

```text
Mỗi lượt = (100.000 × 2 + 10.000 × 10) / 1.000.000 = $0,30
Ba lượt = $0,90
```

Đây chỉ là **ví dụ số học**, không phải báo giá cho một model cụ thể. Đối chiếu bảng giá chính thức của nhà cung cấp trước khi lập ngân sách.

### Tại sao chi phí thực tế có thể khác?

1. Agent tự đọc thêm nhiều file và log.
2. Có nhiều lượt suy luận, gọi công cụ, kiểm thử, sửa lại.
3. Cache token được tính giá khác.
4. Các gói thuê bao và hạn mức Agent không quy đổi trực tiếp sang hóa đơn API.
5. RTK chỉ giảm **một phần output terminal** đưa vào ngữ cảnh, không đảm bảo giảm toàn bộ token.

## 6. Tích hợp Tokenometer vào VS Code / Antigravity

- Extension: https://marketplace.visualstudio.com/items?itemName=faraa2m.tokenometer-vscode
- Trong VS Code: `Ctrl + Shift + X` → tìm **Tokenometer** → Install.
- Extension có thể hiển thị model, token và USD cho **file prompt đang mở**.
- Antigravity có thể không hỗ trợ mọi extension VS Code; nếu không cài được, dùng CLI Tokenometer ngay trong terminal của Antigravity.

**Quan trọng:** Extension không tự đoán được mọi file Agent sẽ đọc trong tương lai.

## 7. Dùng cùng Codex, Claude và Gemini

### Codex CLI

```powershell
cd C:\Users\ADMIN\Paper_1\git_data
codex.cmd
```

Nhập yêu cầu bằng ngôn ngữ tự nhiên, ví dụ:

```text
Review the PyTorch training pipeline and suggest fixes.
Do not modify files until I approve the proposed changes.
Do not start GPU training.
```

Thoát Codex bằng `/exit` hoặc `Ctrl+C` (tùy phiên bản). Tokenometer được chạy **trước** trong PowerShell; nó không tự chặn Codex nếu dự toán vượt ngân sách.

### Claude Code và Gemini CLI

Tương tự: chuẩn bị prompt trong `.md` → dùng Tokenometer đo → chọn Agent và giao nhiệm vụ. Lệnh khởi động Agent phụ thuộc CLI đã cài. **Không nhầm việc dùng model Claude/Gemini trong Antigravity với việc đã cài Claude Code/Gemini CLI độc lập.**

## 8. RTK và ccusage bổ trợ ra sao?

RTK đã cài có thể kiểm tra:

```powershell
rtk --version
rtk init --show
rtk gain
rtk gain --history
```

RTK **không** dự trù giá prompt; nó tối ưu output terminal khi hook hoạt động và có lệnh tương thích.

Để kiểm tra công cụ thống kê usage sau khi Agent chạy, tham khảo tài liệu ccusage: https://github.com/ccusage/ccusage. Khả năng đọc log phụ thuộc Agent và phiên bản; không mặc định mọi nguồn (đặc biệt Antigravity) đều được hỗ trợ đầy đủ.

## 9. Checklist làm việc hằng ngày

1. Soạn prompt vào `prompt.md`.
2. Chạy `tokenometer.cmd .\prompt.md --model <model-id>`.
3. Tính thêm dự phòng cho output và các lượt gọi model của Agent.
4. Mở Codex/Claude/Gemini và giao việc; giới hạn phạm vi file, quyền sửa, và training GPU.
5. Review diff, test và log sau khi Agent chạy.
6. Dùng RTK/ccusage (nếu tương thích) để theo dõi usage thực tế.

## 10. Xử lý lỗi thường gặp

| Lỗi | Cách xử lý |
|---|---|
| `npm` bị PowerShell chặn script | Dùng `npm.cmd` |
| `tokenometer` không được nhận diện | Dùng `tokenometer.cmd`, mở terminal mới, kiểm tra npm global prefix |
| `Unknown model` | Chạy `tokenometer.cmd --help`, chọn model ID được hỗ trợ |
| Chi phí chỉ vài xu dù Agent làm việc lâu | Tokenometer chỉ đang đo prompt/file đầu vào, chưa đo toàn bộ vòng Agent |
| Không có số liệu RTK | Hook chưa chạy, Agent không dùng lệnh tương thích, hoặc chưa có lịch sử |
| Gemini/Claude offline token không khớp | Đếm offline có thể là xấp xỉ; cân nhắc `--empirical` khi có API key |

## Nguồn chính thức

- Tokenometer README: https://github.com/faraa2m/tokenometer
- Tokenometer CLI: https://github.com/faraa2m/tokenometer/tree/main/packages/cli
- Tokenometer VS Code extension: https://marketplace.visualstudio.com/items?itemName=faraa2m.tokenometer-vscode
- RTK: https://github.com/rtk-ai/rtk
- ccusage: https://github.com/ccusage/ccusage

---

*Tài liệu dành cho môi trường Windows PowerShell; các ví dụ về model và giá có thể thay đổi theo thời gian.*
