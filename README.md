# My-Home-Lab
Home Lab Detail AI for learning
# Quy trình xây dựng project với Antigravity + Matt Pocock Skills + Tokenometer + RTK

> **Mục tiêu:** Ý tưởng → thiết kế Skills & Rules → review → sửa → dự trù chi phí → người dùng duyệt → tối ưu prompt & terminal → review lần hai → người dùng duyệt → khởi tạo.
>
> **Môi trường:** Windows 10/11, PowerShell, Antigravity, Codex CLI. Lệnh minh họa sử dụng project `C:\Users\ADMIN\Paper_1\git_data`.

## 0. Phân công công cụ

| Thành phần | Vai trò | Không làm gì |
|---|---|---|
| **Antigravity** | Nơi người dùng làm việc với Agent, xây dựng ý tưởng, rules, skills và code | Không mặc định dùng chung plugin/hook với Codex CLI |
| **Matt Pocock Skills** | Hướng dẫn Agent khảo sát yêu cầu, review, kiểm thử, cải thiện kiến trúc | Không phải một AI/model hay hệ thống kiểm thử tự động độc lập |
| **Tokenometer** | Đếm token đầu vào đã biết, ước tính USD theo model/bảng giá | Không biết trước toàn bộ token của vòng lặp Agent |
| **RTK** | Rút gọn output của một số lệnh terminal qua hook/CLI, giảm token log | **Không** tự viết lại prompt hoặc tự giảm giá API/model |
| **Codex/Claude/Gemini Agent** | Thực thi các prompt và skill, chỉnh sửa file theo quyền được cấp | Không nên được phép triển khai khi chưa có phê duyệt |

**Nguyên tắc:** Matt review ở hai điểm, người dùng duyệt ở hai điểm. Không chạy training, xóa file, push Git hoặc triển khai nếu chưa được cho phép.

## 1. Chuẩn bị môi trường (PowerShell)

Mở terminal **PowerShell** trong Antigravity:

```powershell
cd C:\Users\ADMIN\Paper_1\git_data
node -v
npm.cmd -v
codex.cmd --version
rtk --version
```

Cài Tokenometer (nếu chưa cài):

```powershell
npm.cmd install -g tokenometer
tokenometer.cmd --help
```

Cài Matt Pocock Skills cho Codex bằng trình cài đặt đa Agent:

```powershell
npx.cmd skills@latest add mattpocock/skills
```

Chọn **Codex**, tối thiểu các skill `setup-matt-pocock-skills`, `code-review`, `grill-with-docs`, `diagnosing-bugs`, `tdd`, `writing-for-agents` (nếu có). Có thể chọn project hoặc global. Nếu đã cài thì không cần cài lại.

```powershell
Test-Path "$HOME\.agents\skills\code-review\SKILL.md"
Test-Path "$HOME\.agents\skills\setup-matt-pocock-skills\SKILL.md"
```

`True` cho biết file có mặt tại **đường dẫn global minh họa**; nếu cài local, hãy kiểm tra thư mục skill trong project. Khởi động lại Agent sau khi cài. Đọc nội dung skill trước khi chạy vì skill có thể hướng dẫn Agent dùng quyền chỉnh sửa/chạy lệnh.

> **Antigravity:** việc Codex nhận skill không chứng minh Antigravity Agent cũng đã nhận. Nếu Antigravity hỗ trợ thư mục skills tương thích, hãy cài/chọn Agent tương ứng bằng `npx.cmd skills@latest add mattpocock/skills` hoặc đặt skill theo tài liệu Antigravity đang dùng. Nếu không, thực hiện bước review bằng **Codex CLI trong terminal Antigravity**, trên cùng thư mục project.

## 2. Cấu trúc project đề xuất

```text
project/
├── .agents/
│   └── skills/
│       └── my-project-planner/
│           └── SKILL.md
├── docs/
│   ├── idea.md
│   ├── requirements.md
│   ├── skills-and-rules.md
│   ├── review-01.md
│   ├── cost-estimate.md
│   ├── optimized-prompt.md
│   ├── review-02.md
│   └── approvals.md
├── prompts/
│   └── implementation.md
├── AGENTS.md
└── src/
```

Đây là **cấu trúc đề xuất**, không phải quy định bắt buộc của Antigravity. Nếu IDE có vị trí Rules/Workflows/Skills riêng, hãy tạo tại vị trí mà IDE hỗ trợ; đồng thời lưu bản mô tả vào `docs/` để dễ review bằng Codex.

Tạo thư mục:

```powershell
New-Item -ItemType Directory -Force docs, prompts, src, .agents\skills\my-project-planner
```

Tạo file skill cơ bản bằng PowerShell:

```powershell
@'
---
name: my-project-planner
description: Design project requirements, agent skills, rules, and acceptance criteria before coding.
---

# Project planner
1. Read docs/idea.md.
2. Ask for missing requirements before implementation.
3. Produce docs/requirements.md and docs/skills-and-rules.md.
4. Define acceptance criteria and a test strategy.
5. Never implement, install dependencies, train models, or deploy without user approval.
'@ | Set-Content -Encoding utf8 .agents\skills\my-project-planner\SKILL.md
```

Tạo quy tắc repository chung (được Codex đọc khi thích hợp):

```powershell
@'
# Project Rules
- Plan before implementing.
- No destructive commands, package installation, GPU training, git push, or deployment without explicit approval.
- Record requirements and decisions under docs/.
- Review changes before finalizing.
- Report changed files and test results.
'@ | Set-Content -Encoding utf8 AGENTS.md
```

**Lưu ý:** `AGENTS.md` là cơ chế hướng dẫn phổ biến cho Codex; để áp dụng trong Antigravity, cấu hình Rules tương ứng trong Antigravity theo giao diện/phiên bản đang sử dụng. Không giả định Antigravity tự đọc `AGENTS.md`.

## 3. Workflow 7 giai đoạn — câu lệnh và prompt cụ thể

### Giai đoạn 1 — Người dùng đưa ý tưởng; Antigravity lập skills & rules

Viết ý tưởng vào `docs/idea.md` (hoặc trao đổi trực tiếp với Antigravity Agent):

```powershell
@'
# Ý tưởng
Xây dựng pipeline phát hiện xâm nhập mạng bằng PyTorch.
Dữ liệu: NF-ToN-IoT-v2 và NF-BoT-IoT-v2.
Yêu cầu: tách preprocessing, model, training, evaluation; báo cáo macro-F1.
Giới hạn: không chạy training GPU khi chưa được duyệt.
'@ | Set-Content -Encoding utf8 docs\idea.md
```

**Prompt cho Antigravity Agent:**

```text
Đọc docs/idea.md. Hãy thiết kế bộ Skills và Rules cho project này.
1. Phân tích mục tiêu, phạm vi, rủi ro, tiêu chí nghiệm thu.
2. Đề xuất từng skill với name, description, input, output, giới hạn quyền.
3. Soạn nội dung SKILL.md cho các skill cần thiết và rules tương ứng.
4. Lưu bản thảo vào docs/requirements.md và docs/skills-and-rules.md.
5. Chỉ thiết kế, KHÔNG viết code triển khai, không chạy training.
6. Nếu thiếu thông tin, hỏi tôi trước.
```

**Đầu ra cần có:** `requirements.md`, `skills-and-rules.md`, bản nháp skill/rules, acceptance criteria.

### Giai đoạn 2 — Matt Pocock review bản thiết kế

Mở Codex CLI trong cùng thư mục project:

```powershell
codex.cmd
```

**Prompt trong Codex** (không bắt buộc dùng slash command):

```text
Review docs/requirements.md, docs/skills-and-rules.md, AGENTS.md
and .agents/skills/my-project-planner/SKILL.md.
Use Matt Pocock's writing-for-agents and relevant review/design skills
if available. Identify contradictions, missing constraints, unclear
skill triggers, unsafe permissions, unverifiable acceptance criteria,
and conflicts between rules. Report Critical/High/Medium/Low issues
with file locations. Do not modify files.
```

> `code-review` của Matt chủ yếu review **diff code** so với standards/spec. Đối với **skill/rules dạng văn bản**, `writing-for-agents` và `grill-with-docs` thường phù hợp hơn. Có thể yêu cầu `code-review` khi đã có diff và tiêu chí rõ.

Lưu nhận xét vào `docs/review-01.md` (nhờ Agent lưu hoặc copy từ kết quả). Nếu Codex chưa nhận skill, kiểm tra `SKILL.md` và khởi động lại Codex.

### Giai đoạn 3 — Sửa theo review, kiểm tra lại

**Prompt:**

```text
Read docs/review-01.md. Fix the confirmed High/Critical issues in
our draft skills, rules and requirements. Keep the original project
scope. For each fix, explain what changed and why. Do not implement
application code. Re-review the updated documents and list any
remaining issues.
```

**Gate:** Không sang bước chi phí nếu còn lỗi nghiêm trọng hoặc chưa xác định được tiêu chí nghiệm thu.

### Giai đoạn 4 — Tokenometer dự trù chi phí

Tạo prompt triển khai từ bản đã sửa, ví dụ:

```powershell
@'
Implement the approved PyTorch NIDS project according to docs/requirements.md.
Follow docs/skills-and-rules.md and AGENTS.md.
Do not run GPU training, install packages, push commits or deploy without approval.
Start with a file-by-file implementation plan and tests.
'@ | Set-Content -Encoding utf8 prompts\implementation.md
```

Ước tính token và **chi phí input** của prompt đã biết:

```powershell
tokenometer.cmd .\prompts\implementation.md --model gpt-4o,claude-sonnet-4-6
```

Xem model được hỗ trợ:

```powershell
tokenometer.cmd --help
```

**Cách tính dự trù toàn nhiệm vụ** (thủ công, vì Agent sẽ đọc file, suy luận, gọi tool nhiều lần):

```text
Tổng dự trù ≈ (input_tokens_dự_kiến × giá_input
              + output_tokens_dự_kiến × giá_output) / 1,000,000
              × số_lượt_dự_kiến
```

Ước lượng 3 kịch bản **Low / Expected / High**, tính cả file context, output, vòng sửa và cache nếu có. Kiểm tra đơn giá model tại thời điểm sử dụng. `tokenometer` không thể dự đoán chính xác tổng chi phí một Agent chưa chạy, và USD quy đổi API **không đồng nghĩa** khoản tiền bị trừ từ gói ChatGPT/Claude/Gemini.

**Prompt lập ngân sách:**

```text
Use the Tokenometer result for prompts/implementation.md as the known
input baseline. Estimate LOW / EXPECTED / HIGH total token usage for
reading project files, implementation, tests and 2 review loops.
State assumptions, model prices, cache assumptions and uncertainty.
Write docs/cost-estimate.md. Do not run implementation.
```

### Giai đoạn 5 — Người dùng duyệt, tối ưu prompt + RTK

**Điểm duyệt 1:** Người dùng xác nhận bản thiết kế và ngân sách. Ghi vào `docs/approvals.md`:

```markdown
## Approval 1 — Design & budget
- Status: APPROVED / CHANGES REQUESTED
- Approved scope:
- Budget ceiling:
- Approved by:
- Date:
```

**Prompt cho Agent tối ưu prompt** (RTK **không** viết prompt):

```text
Rewrite prompts/implementation.md into docs/optimized-prompt.md.
Keep every functional requirement, constraint, acceptance criterion,
security boundary and approval gate. Remove repetition and ambiguity.
Provide before/after token counts using Tokenometer, and a requirement
coverage matrix proving nothing important was lost. Do not implement.
```

So sánh input token trước/sau:

```powershell
tokenometer.cmd .\prompts\implementation.md --model gpt-4o
tokenometer.cmd .\docs\optimized-prompt.md --model gpt-4o
```

**Bật RTK cho terminal của Codex (nếu chưa cấu hình):**

```powershell
rtk init -g --codex
rtk init --show
```

Khởi động lại Codex sau khi cài hook. RTK chỉ có tác dụng khi các lệnh tương thích đi qua RTK/hook. Có thể tự thử:

```powershell
rtk git status
rtk git diff
rtk gain
rtk gain --history
```

`rtk gain` đo tiết kiệm token từ các lệnh RTK ghi nhận, **không** đo mức giảm chi phí của prompt do Agent viết lại. Trên PowerShell, một số lệnh Unix như `rtk ls` có thể không hoạt động nếu thiếu executable tương ứng.

### Giai đoạn 6 — Matt review bản tối ưu + người dùng duyệt

**Prompt:**

```text
Review docs/optimized-prompt.md against docs/requirements.md,
docs/skills-and-rules.md, docs/cost-estimate.md and the original
prompts/implementation.md. Use Matt Pocock's writing-for-agents and
code-review skills where applicable. Verify requirement coverage,
security constraints, correct skill triggers, testability, and no
loss of meaning. Identify regressions by severity. Do not modify.
```

Lưu `docs/review-02.md`. Nếu phát hiện lỗi, quay lại giai đoạn 5, sửa prompt và đo lại token. **Người dùng duyệt lần 2** sau khi xem review và dự toán cập nhật:

```markdown
## Approval 2 — Optimized execution plan
- Status: APPROVED / CHANGES REQUESTED
- Final prompt: docs/optimized-prompt.md
- Final budget ceiling:
- Allowed actions: [ ] edit code [ ] run tests [ ] install deps [ ] GPU train [ ] push/deploy
- Approved by:
- Date:
```

### Giai đoạn 7 — Khởi tạo và triển khai khi đã duyệt

Chỉ sau khi `Approval 2 = APPROVED`:

```powershell
cd C:\Users\ADMIN\Paper_1\git_data
codex.cmd
```

**Prompt triển khai:**

```text
Read docs/approvals.md. Proceed ONLY if Approval 1 and Approval 2
are APPROVED. Follow docs/optimized-prompt.md and the approved skills
and rules. Implement incrementally, run only explicitly allowed tests,
show git diff, and use Matt Pocock's code-review skill on the resulting
diff before final delivery. Stop and ask permission if the requested
action exceeds the approved scope or budget.
```

**Kiểm tra kết quả trong PowerShell:**

```powershell
rtk git status
rtk git diff
rtk gain
```

**Quan trọng:** Các bước phê duyệt trong tài liệu là **quy trình vận hành**, không phải cơ chế bảo mật tự động. Muốn chặn hành động thật sự, hãy dùng chế độ quyền/phê duyệt của Agent, Git protections và giới hạn quyền môi trường.

## 4. Few-shot examples — prompt mẫu theo tình huống

### Ví dụ A — Project NIDS Transformer/Mamba (nghiên cứu AI)

**Người dùng:** “Tôi muốn xây dựng IDS đa lớp với Transformer + Mamba, đo macro-F1, không chạy GPU cho đến khi duyệt.”

**Prompt tạo skill/rules:**

```text
Draft a research-engineer skill for a PyTorch NIDS project.
Inputs: NF-ToN-IoT-v2, NF-BoT-IoT-v2.
Outputs: preprocessing, model, training, evaluation, reproducibility docs.
Rules: prevent data leakage; stratified split; report macro-F1 and per-class
recall; never train GPU without approval. Save design docs only.
```

**Prompt Matt review:**

```text
Review the drafted NIDS skills/rules for data leakage, incorrect split
protocols, missing ablations, reproducibility gaps, and unsafe execution
permissions. Give file:line and severity. Do not change files.
```

**Prompt tối ưu sau duyệt:**

```text
Compress the implementation prompt without removing evaluation protocol,
seed control, no-leakage rule, no-GPU rule, and review checkpoints.
Compare token counts and show requirement coverage.
```

### Ví dụ B — Web dashboard thống kê token

**Người dùng:** “Tôi muốn dashboard hiển thị token/chi phí của Codex, Claude, Gemini từ các log local.”

**Prompt tạo skill/rules:**

```text
Design a read-only local usage dashboard. Draft a data-ingestion skill,
privacy rules, model-pricing assumptions, and acceptance tests.
Never upload logs or API keys. Do not write implementation code yet.
```

**Prompt Matt review:**

```text
Review for unsafe log handling, leaked secrets, wrong assumptions about
subscription billing vs API estimates, missing error cases, and testability.
```

**Prompt dự toán và tối ưu:**

```text
Prepare a bounded implementation prompt, count its known input tokens
with Tokenometer, estimate 3 total-cost scenarios, then propose a shorter
prompt preserving privacy and accuracy requirements.
```

### Ví dụ C — Sửa bug trong ứng dụng Python có sẵn

**Người dùng:** “Training bị hết RAM; muốn AI tìm nguyên nhân nhưng không tự thay kiến trúc.”

**Prompt:**

```text
Create a debugging skill with read-only investigation first.
Require evidence before each hypothesis. Use Matt Pocock's diagnosing-bugs
skill to review the plan. Estimate token budget before reading the whole
repository. After approval, optimize terminal output with RTK and implement
only the minimal approved fix, then review the diff.
```

## 5. Checklist trước khi khởi tạo

- [ ] Ý tưởng, yêu cầu và tiêu chí nghiệm thu đã rõ.
- [ ] Skills/Rules đã được tạo và Agent nhận diện đúng.
- [ ] Matt review lần 1; lỗi Critical/High đã được xử lý.
- [ ] Tokenometer đã đo input prompt; ngân sách tổng có giả định Low/Expected/High.
- [ ] Người dùng đã phê duyệt thiết kế và ngân sách.
- [ ] Prompt được tối ưu nhưng không mất yêu cầu.
- [ ] RTK hook đã cấu hình cho Agent tương thích và kiểm tra `rtk gain`.
- [ ] Matt review lần 2; người dùng duyệt bản cuối.
- [ ] Chỉ các quyền được phê duyệt mới được sử dụng khi khởi tạo.

## 6. Lệnh nhanh

```powershell
# Mở project
cd C:\Users\ADMIN\Paper_1\git_data

# Cài Matt skills (nếu cần)
npx.cmd skills@latest add mattpocock/skills

# Kiểm tra skill đã cài global
Test-Path "$HOME\.agents\skills\code-review\SKILL.md"

# Dự trù input token
 tokenometer.cmd .\prompts\implementation.md --model gpt-4o,claude-sonnet-4-6

# RTK cho Codex
rtk init -g --codex
rtk init --show
rtk gain

# Khởi động Agent
codex.cmd
```

## 7. Tài liệu nguồn

- [Matt Pocock Skills](https://github.com/mattpocock/skills)
- [Matt Pocock — code-review](https://github.com/mattpocock/skills/blob/main/skills/engineering/code-review/SKILL.md)
- [Matt Pocock — writing-for-agents](https://github.com/mattpocock/skills/blob/main/skills/productivity/writing-for-agents/SKILL.md)
- [Skills installer](https://github.com/vercel-labs/skills)
- [Tokenometer](https://github.com/faraa2m/tokenometer)
- [RTK](https://github.com/rtk-ai/rtk)

> Các model ID, giá và khả năng tích hợp Agent có thể thay đổi theo phiên bản. Kiểm tra `--help` và README hiện hành trước khi chạy.
