# Hướng dẫn Matt Pocock Skills — Codex, Claude Code, Gemini CLI (Windows)

> Tài liệu dành cho Windows 10/11, PowerShell và project `C:\Users\ADMIN\Paper_1\git_data`. Nguồn: https://github.com/mattpocock/skills (README, tháng 10/2026).

## 1. Skills là gì?

**Skill** là tập hướng dẫn cho AI coding agent. Skill **không tự chạy code**, không phải model AI riêng. Codex/Claude/Gemini đọc `SKILL.md` rồi thực hiện công việc bằng các công cụ và quyền đang có. Có thể phối hợp **skill tự tạo → triển khai → Matt Pocock `code-review` → sửa lỗi**.

Matt phân biệt:
- **User-invoked:** chủ động gọi khi muốn điều phối công việc, ví dụ `setup-matt-pocock-skills`, `implement`, `grill-with-docs`.
- **Model-invoked:** agent có thể chủ động áp dụng khi phù hợp, ví dụ `code-review`, `diagnosing-bugs`, `tdd`.

> **Chú ý Codex CLI:** `/setup-matt-pocock-skills` có thể báo `Unrecognized command`. Thay bằng prompt tự nhiên: `Use the setup-matt-pocock-skills skill to configure this repository.`

## 2. Cài đặt trên Windows

### Điều kiện

```powershell
node -v
npm.cmd -v
codex.cmd --version
```

### Cách A — Codex plugin (README chính thức)

Chạy trong **PowerShell**, không phải trong giao diện chat Codex:

```powershell
codex.cmd plugin marketplace add mattpocock/skills
codex.cmd plugin add mattpocock-skills@mattpocock
```

Nếu bản Codex không hỗ trợ lệnh plugin, dùng **Cách B**. **Chỉ chọn một cách cài cho mỗi agent** để tránh trùng skill.

### Cách B — Skills CLI (đã dùng trên máy bạn)

```powershell
cd C:\Users\ADMIN\Paper_1\git_data
npx.cmd skills@latest add mattpocock/skills
```

Chọn **Codex** và những skill cần cài. Nếu muốn dùng mọi project, chọn cài global. Kết quả thực tế có thể nằm tại:

```text
C:\Users\ADMIN\.agents\skills\
```

Kiểm tra:

```powershell
Test-Path "$HOME\.agents\skills\setup-matt-pocock-skills\SKILL.md"
Test-Path "$HOME\.agents\skills\code-review\SKILL.md"
Get-ChildItem "$HOME\.agents\skills" -Directory | Select-Object Name
```

`True` nghĩa là file tồn tại, **chưa đủ để kết luận agent đã tải skill**. Cần khởi động lại Codex và xác nhận phản hồi của agent.

### Claude Code

```powershell
claude plugin install mattpocock-skills@claude-plugins-official
```

### Gemini CLI

```powershell
gemini skills install https://github.com/mattpocock/skills.git --path skills/engineering
gemini skills install https://github.com/mattpocock/skills.git --path skills/productivity
```

> Đừng chạy đồng thời các cách cài khác nhau trên cùng agent nếu không cần. Với Antigravity, hỗ trợ skill phụ thuộc cơ chế agent/phiên bản; không mặc định suy ra cài Codex là Antigravity Agent đã nhận.

## 3. Thiết lập lần đầu trong Codex

Mở project rồi chạy Codex:

```powershell
cd C:\Users\ADMIN\Paper_1\git_data
codex.cmd
```

Trong ô **Ask Codex to do anything**, nhập:

```text
Use the setup-matt-pocock-skills skill to configure this repository.
```

Skill sẽ hỏi về **issue tracker**, **triage labels** và **vị trí lưu tài liệu**. Gợi ý cho project nghiên cứu:

| Mục | Gợi ý |
|---|---|
| Issue tracker | GitHub nếu đang dùng Issues; nếu chưa dùng thì local files |
| Triage labels | Giữ mặc định hoặc định nghĩa theo quy trình hiện có |
| Documentation | `docs/` |

Trong phiên của bạn, Codex đã báo `I found and am using setup-matt-pocock-skills` — nghĩa là **đã tìm thấy skill**. Tuy nhiên, ảnh cũng có lỗi `Get-Content` exit 1, nên phải chờ kết thúc và kiểm tra báo cáo cuối trước khi kết luận thiết lập hoàn tất.

## 4. Danh sách skill hữu ích

| Skill | Công dụng | Cách yêu cầu |
|---|---|---|
| `ask-matt` | Gợi ý skill/quy trình phù hợp | `Use ask-matt to recommend a workflow for this task.` |
| `grill-with-docs` | Làm rõ yêu cầu và tài liệu domain | `Use grill-with-docs before implementation.` |
| `to-spec` | Chuyển yêu cầu thành đặc tả | `Use to-spec to draft a spec from our discussion.` |
| `to-tickets` | Chia đặc tả thành đầu việc | `Use to-tickets to break down this spec.` |
| `implement` | Triển khai theo spec/tickets | `Use implement for this approved spec.` |
| `code-review` | Review diff theo chuẩn code và spec | `Use code-review on uncommitted changes.` |
| `diagnosing-bugs` | Điều tra lỗi theo từng bước | `Use diagnosing-bugs to identify the root cause.` |
| `tdd` | Test trước, sửa code, refactor | `Use tdd for this bug fix.` |
| `improve-codebase-architecture` | Khảo sát cơ hội cải thiện kiến trúc | `Use improve-codebase-architecture to survey the repo.` |
| `retro` | Đánh giá quy trình sau phiên làm việc | `Use retro to suggest workflow improvements.` |
| `handoff` | Bàn giao ngữ cảnh cho agent khác | `Use handoff to prepare a continuation document.` |

Các skill thuộc nhiều nhóm khác nhau; không phải skill nào cũng tự chạy. Xem README và `SKILL.md` của từng skill để hiểu phạm vi.

## 5. Prompt mẫu: skill của bạn → Matt review

### A. Skill riêng triển khai

```text
Use my custom skill <TEN_SKILL_CUA_TOI> to implement the requested changes.
Project: C:\Users\ADMIN\Paper_1\git_data
Requirements:
- Keep preprocessing and dataset splits unchanged.
- Do not run GPU training.
- Explain which files you modified.
- Do not commit or push without my approval.
Confirm the skill file you loaded before starting.
```

### B. Review độc lập bằng Matt

```text
Use Matt Pocock's code-review skill to review the changes from the previous task.
Review the diff against the original requirements and project standards.
Check correctness, data leakage, training/evaluation logic, performance,
and maintainability.
For every issue, report: severity, file:line, evidence, impact, suggested fix.
Do not modify files, run GPU training, commit, or push.
Confirm which SKILL.md was loaded.
```

### C. Sửa sau khi review

```text
Fix only the High and Critical findings in the previous review.
Keep the changes minimal, run only lightweight checks, and show the diff.
Do not run full training, commit, or push.
Then perform another code-review pass.
```

### D. Một prompt cho cả pipeline

```text
1. Use my custom skill <TEN_SKILL_CUA_TOI> to implement the task.
2. Show the changed files and a concise diff summary.
3. Use Matt Pocock's code-review skill to review the changes.
4. Report findings by severity with file and line references.
5. Stop and wait for my approval before applying review fixes.
Never run GPU training or commit/push automatically.
```

**Giới hạn:** Dùng hai skill không mặc nhiên tạo hai reviewer độc lập; cùng một agent có thể thực hiện cả hai bước. Nếu cần review độc lập thực sự, dùng một phiên agent khác và chỉ cung cấp diff/spec cho reviewer.

## 6. Lệnh kiểm tra và quy trình Git

Trong **PowerShell**:

```powershell
# Xem các file đã thay đổi
git status

# Xem diff chưa stage
git diff

# Xem diff đã stage
git diff --cached

# Xem thư mục skill
Get-ChildItem "$HOME\.agents\skills" -Directory

# Kiểm tra nội dung skill trước khi dùng
Get-Content "$HOME\.agents\skills\code-review\SKILL.md" -TotalCount 60
```

Với RTK đã cài, có thể thay một số lệnh Git bằng `rtk git status` hoặc `rtk git diff` để rút gọn output. **RTK không phải bộ review code**.

## 7. Lỗi thường gặp

**`Unrecognized command '/setup-matt-pocock-skills'`**  
Dùng prompt ngôn ngữ tự nhiên thay cho slash command; skill có thể đã cài nhưng không đăng ký dưới dạng slash command.

**Codex không thấy skill**  
Kiểm tra `SKILL.md` tồn tại, khởi động lại Codex, yêu cầu agent nêu chính xác đường dẫn skill đã tải. Kiểm tra đã chọn Codex trong bước cài.

**`Get-Content` exit 1**  
Mở rộng log lỗi, xem đường dẫn có đúng không, rồi thử trong PowerShell:

```powershell
Get-Content -LiteralPath "$HOME\.agents\skills\setup-matt-pocock-skills\SKILL.md" -TotalCount 30
```

Không kết luận cấu hình hoàn tất chỉ từ câu `I found and am using...`.

**Skill cài global nhưng project không nhận**  
Khởi động lại agent; kiểm tra danh sách skill và quy tắc discovery của agent đang sử dụng.

**Cài quá nhiều skill**  
Chỉ giữ các skill cần thiết. Đọc `SKILL.md` trước khi cấp quyền; skill có thể hướng dẫn agent chạy lệnh và sửa file với quyền hiện tại.

## 8. Tài liệu chính thức

- Repository: https://github.com/mattpocock/skills
- README: https://github.com/mattpocock/skills/blob/main/README.md
- Code review: https://github.com/mattpocock/skills/blob/main/skills/engineering/code-review/SKILL.md
- Setup: https://github.com/mattpocock/skills/blob/main/skills/engineering/setup-matt-pocock-skills/SKILL.md
- Diagnosing bugs: https://github.com/mattpocock/skills/blob/main/skills/engineering/diagnosing-bugs/SKILL.md
- Skills installer: https://github.com/vercel-labs/skills

## 9. Đưa tài liệu này lên GitHub

Đặt file vào repository, sau đó chạy:

```powershell
git add Matt_Pocock_Skills_Guide.md
git commit -m "docs: add Matt Pocock skills guide"
git push
```

> Kiểm tra `git status` và nội dung diff trước khi commit; không đưa API keys hoặc file chứa dữ liệu nhạy cảm lên GitHub.
