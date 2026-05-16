# Daily Motivation 5AM — Hướng dẫn cài đặt Claude Code Routine

> Tạo một routine tự động chạy lúc 5h sáng mỗi ngày, vừa thêm câu motivation song ngữ vào repo GitHub, vừa pre-warm window 5h của Claude Code để tối ưu thời gian làm việc trong ngày.

---

## Mục lục

1. [Bối cảnh và mục tiêu](#bối-cảnh-và-mục-tiêu)
2. [Cơ chế 5-hour window — tại sao 5h sáng?](#cơ-chế-5-hour-window--tại-sao-5h-sáng)
3. [Chuẩn bị](#chuẩn-bị)
4. [Bước 1 — Tạo GitHub repository](#bước-1--tạo-github-repository)
5. [Bước 2 — Tạo routine trên Claude Code](#bước-2--tạo-routine-trên-claude-code)
6. [Bước 3 — Cấu hình Connectors, Behavior, Permissions](#bước-3--cấu-hình-connectors-behavior-permissions)
7. [Bước 4 — Schedule và Create](#bước-4--schedule-và-create)
8. [Bước 5 — Test với Run now](#bước-5--test-với-run-now)
9. [Troubleshooting](#troubleshooting)
10. [Tinh chỉnh nâng cao](#tinh-chỉnh-nâng-cao)
11. [Tham khảo](#tham-khảo)

---

## Bối cảnh và mục tiêu

Routine này phục vụ **hai mục đích cùng lúc**:

### Mục đích 1 — Pre-warm window 5h Claude Code

Cửa sổ 5h của Claude Code **không reset theo đồng hồ** (không phải nửa đêm hay giờ tròn). Nó **bắt đầu tính từ prompt đầu tiên** trong session và reset đúng 5 tiếng sau đó. Đây là *rolling window* gắn với timing của bạn, không phải clock time.

**Hệ quả thực tế:** Nếu prompt đầu tiên của ngày là lúc 9h sáng, window sẽ reset lúc 14h chiều — đúng lúc bạn đang focus buổi chiều. Bạn bị khoá session đúng vào lúc cần làm việc nhất.

**Giải pháp:** Cho một prompt nhỏ chạy lúc 5h sáng (khi bạn còn ngủ) để bắt đầu window sớm. Khi 9h bạn ngồi vào làm việc, đã có 4 tiếng trôi qua, và window reset lúc 10h — giải phóng cả ngày còn lại.

### Mục đích 2 — Tạo evidence và động lực có nghĩa

Thay vì chỉ ping API (không để lại dấu vết hữu ích), routine sẽ:

- Mở 1 repo GitHub của bạn
- Sinh 1 câu motivation song ngữ Việt-Anh (không trùng các câu cũ)
- Append vào README với timestamp
- Commit và push lên `main`

Kết quả: mỗi sáng bạn mở GitHub thấy có commit mới + câu motivation mới. Vừa verify được routine chạy, vừa có cảm hứng bắt đầu ngày.

### Lợi ích phụ

- **GitHub contribution graph xanh đều** — như Duolingo streak cho dev
- **Audit log tự nhiên** — không cần monitoring riêng, GitHub commit history chính là log
- **Tiêu rất ít quota** — Haiku model + 1 ngắn, không ảnh hưởng weekly cap

---

## Cơ chế 5-hour window — tại sao 5h sáng?

### So sánh trước và sau khi có routine

**Không có routine (prompt đầu lúc 9h):**

| Time | Trạng thái |
|------|-----------|
| 09:00 | Window 1 mở (bắt đầu làm việc) |
| 14:00 | Window 1 reset → Window 2 mở |
| 19:00 | Window 2 reset → Window 3 mở (đã hết giờ làm) |

→ Chỉ có **2 window hữu ích** trong giờ làm việc (9-14h và 14-19h).

**Có routine 5h sáng:**

| Time | Trạng thái |
|------|-----------|
| 05:00 | Window 1 mở (routine chạy, bạn còn ngủ) |
| 10:00 | Window 1 reset → Window 2 mở (đang làm việc) |
| 15:00 | Window 2 reset → Window 3 mở |
| 20:00 | Window 3 reset → Window 4 (tối, optional) |

→ Có **3 window hữu ích** rơi đúng vào giờ làm việc (10-15h, 15-20h, và buffer 9-10h từ window 1).

### Giới hạn cần biết

- Routine **không cho thêm message** — quota mỗi window vẫn nguyên
- Weekly cap **không đổi** — chỉ phân bổ đều capacity trong ngày
- Pro: ~45 msg/window, Max 5x: ~225, Max 20x: ~900
- Routine daily limit: Pro 5/ngày, Max 15/ngày, Team/Enterprise 25/ngày

### Lưu ý quan trọng về pre-warm

Routine chạy trên **Anthropic-managed cloud infrastructure**, không phải account local của bạn. Việc nó có thực sự "đụng vào" window 5h của subscription cá nhân hay không cần test thực tế:

1. Chạy routine lúc 5h sáng (hoặc Run now để test)
2. Sau khi routine xong, vào Claude Code CLI gõ `/usage`
3. Xem `Current session` — nếu thấy session đã started → pre-warm có hiệu lực
4. Nếu không thấy → routine không pre-warm, nhưng task vẫn có giá trị tự thân (commit motivation)

---

## Chuẩn bị

### Yêu cầu

- Tài khoản Claude với plan **Pro / Max / Team / Enterprise** (có Claude Code on web)
- Tài khoản GitHub (free plan đủ dùng)
- 10 phút setup

### Plan limit reminder

| Plan | Routine/ngày |
|------|--------------|
| Pro | 5 |
| Max | 15 |
| Team / Enterprise | 25 |

Routine này chỉ tiêu **1 slot/ngày** nên không lo vượt giới hạn.

---

## Bước 1 — Tạo GitHub repository

### 1.1 Tạo repo mới

Vào [github.com/new](https://github.com/new):

| Trường | Giá trị |
|--------|---------|
| Repository name | `daily-motivation` |
| Description | `Daily bilingual motivation streak — automated by Claude Code` |
| Visibility | **Private** (recommended) hoặc **Public** nếu muốn khoe streak |
| Add a README file | ✅ **Tick** |
| Add .gitignore | Bỏ qua |
| Add a license | Bỏ qua |

Click **Create repository**.

### 1.2 Khởi tạo README

Mở `README.md` vừa tạo trên GitHub → click nút edit (✏️) → thay nội dung thành:

```markdown
# Daily Motivation Streak

Mỗi 5h sáng, một câu động lực mới được tự động thêm bởi Claude Code routine.

## Lịch sử
```

Commit thay đổi (commit message: `chore: initialize motivation log`).

**Quan trọng:** Dòng `## Lịch sử` ở cuối là điểm anchor mà routine sẽ chèn entry mới ngay sau đó.

---

## Bước 2 — Tạo routine trên Claude Code

### 2.1 Mở Routines

Truy cập [claude.ai/code/routines](https://claude.ai/code/routines) → click **New routine**.

### 2.2 Điền thông tin cơ bản

**Name:**
```
Daily Motivation 5AM
```

**Model:** Chọn `claude-haiku-4-5` trong dropdown.

> Lý do dùng Haiku: task rất ngắn, không cần reasoning sâu, tiết kiệm chi phí và quota.

**Prompt:** Paste nguyên block dưới đây:

```text
You are running as a scheduled routine at 5 AM Vietnam time. Your task is to add one bilingual motivational quote to the README.md of this repository.

STEPS:

1. Read the current README.md to understand its structure.

2. Read the last 100 lines of README.md to see recent motivational quotes (if any). You must AVOID:
   - Reusing any quote already in the file
   - Repeating the same opening phrase
   - Repeating the same metaphor or theme as the last 7 entries
   - Generic clichés like "believe in yourself" or "never give up" — be specific and concrete

3. Generate ONE original motivational quote in BOTH Vietnamese and English. The quote should be:
   - 1-2 sentences max
   - Concrete and grounded, not abstract platitudes
   - Suitable for a developer/knowledge worker starting their day
   - Rotate themes across days: discipline, craft, patience, focus, learning from failure, small wins, deep work, curiosity, debugging mindset, etc.

4. Get the current timestamp in Vietnam time (UTC+7), format: YYYY-MM-DD HH:MM

5. PREPEND (insert at top, not append) the new entry to README.md.

   Steps:
   a. Read the entire README.md file.
   b. Find the line "## Lịch sử".
   c. Insert the new entry block IMMEDIATELY AFTER that line, with one blank line before and after the block.
   d. All existing entries stay below in their current order.
   e. Write the entire updated content back using the Edit or Write tool.

   The new entry format (EXACT formatting):

---

**[YYYY-MM-DD HH:MM ICT]**

* **VI**: <Vietnamese quote>
* **EN**: <English quote>

   IMPORTANT:
   - The bullets must use "* " (asterisk + space), not "- " (dash).
   - There must be a blank line between the timestamp and the first bullet.
   - Do NOT use bash >> redirection. Use Edit or Write tool to insert at correct position.

6. Commit the change directly on the main branch with message:
   "chore: daily motivation YYYY-MM-DD"

7. Push directly to origin/main. DO NOT create a new branch.
   DO NOT use any branch with "claude/" prefix.
   The repository has "Allow unrestricted git push" enabled,
   so pushing directly to main is permitted and expected.

   Specifically run:
   - git checkout main (ensure you are on main)
   - git pull --rebase origin main
   - git add README.md
   - git commit -m "chore: daily motivation YYYY-MM-DD"
   - git push origin main

8. Output a single confirmation line:
   - On success: "[YYYY-MM-DD HH:MM] Motivation added: <first 5 words of EN quote>..."
   - On failure: "[YYYY-MM-DD HH:MM] FAILED: <reason>"

CONSTRAINTS:
- Do NOT modify any existing content in README.md, only insert new content.
- Do NOT create new files.
- If git push fails due to conflicts, pull with rebase first, then retry once. If it still fails, output FAILED with the reason.
- Keep the quote tasteful — no toxic positivity, no hustle culture cringe.
```

### 2.3 Chọn Repository

Chọn repo `daily-motivation` vừa tạo từ dropdown.

> **Nếu chưa thấy repo:** Click "Manage GitHub access" → cấp quyền cho Claude Code truy cập repo này → quay lại refresh.

---

## Bước 3 — Cấu hình Connectors, Behavior, Permissions

Đây là phần **quan trọng nhất** mà nhiều hướng dẫn bỏ qua. Vì routine không có cơ hội hỏi xin phép giữa chừng, mọi thứ phải pre-approve từ đây.

### 3.1 Tab Connectors

→ **Để trống.** Routine không cần Slack, Linear, Notion, hay bất kỳ service nào ngoài GitHub (đã handle qua phần Repository).

> Best practice: least privilege. Càng ít connector càng ít rủi ro.

### 3.2 Tab Behavior

| Setting | Giá trị |
|---------|---------|
| Auto-fix pull requests | **OFF** |
| Setup script | Để trống |
| Environment variables | Để trống |

> "Auto-fix PRs" dành cho routine code generation. Routine motivation không tạo PR, không có CI, nên OFF.

### 3.3 Tab Permissions

**Quan trọng nhất ở đây.**

Mặc định Claude Code chỉ cho phép push lên branch có prefix `claude/`. Nếu không bật permission này, mỗi sáng bạn sẽ có 1 PR mới chờ merge thay vì commit thẳng vào main.

| Setting cho repo `daily-motivation` | Giá trị |
|--------------------------------------|---------|
| Allow unrestricted git push | ✅ **ON** |

Sẽ thấy cảnh báo vàng: `⚠️ Elevated permissions may allow unintended changes to protected branches.`

→ **Bỏ qua cảnh báo này.** Với repo cá nhân chỉ chứa motivation log, rủi ro = 0. Cảnh báo chỉ áp dụng nếu bạn dùng cho production repo.

---

## Bước 4 — Schedule và Create

### 4.1 Configure trigger

| Setting | Giá trị |
|---------|---------|
| Trigger type | **Schedule** |
| Preset | **Daily** |
| Time | **5:00 AM** |
| Timezone | Tự động lấy từ trình duyệt (Asia/Ho_Chi_Minh) |

> Routine có thể delay vài phút do stagger của Anthropic — không phải lỗi.

### 4.2 Create

Click **Create**. Routine xuất hiện trong danh sách.

---

## Bước 5 — Test với Run now

**Không cần chờ 5h sáng để biết routine có chạy đúng không.**

### 5.1 Run now

Vào trang detail của routine vừa tạo → click **Run now** (góc phải trên).

Routine sẽ chạy trong **30 giây - 2 phút**.

### 5.2 Verify kết quả

Sau khi routine xong, check 3 chỗ:

**1. GitHub repo → Commits:**
- Phải có commit mới: `chore: daily motivation YYYY-MM-DD`
- Author: `claude` (hoặc tương tự)
- Commit nằm trên branch `main` (không phải `claude/xxx`)

**2. README.md:**
- Entry mới nằm ngay sau `## Lịch sử`, KHÔNG ở cuối file
- Format đúng: timestamp đậm, 2 bullet `* **VI**:` và `* **EN**:`
- Câu motivation hợp lý, không nhảm

**3. Branch list:**
- Chỉ có `1 Branch` (main)
- KHÔNG có branch `claude/xxx` còn sót lại

### 5.3 Render preview

Format đúng sẽ render thế này trên GitHub:

```markdown
# Daily Motivation Streak

Mỗi 5h sáng, một câu động lực mới được tự động thêm bởi Claude Code routine.

## Lịch sử

---

**[2026-05-16 05:00 ICT]**

* **VI**: Hôm nay viết một test trước khi viết code. Test giúp bạn biết bạn đang làm gì.
* **EN**: Write one test before you write the code today. Tests tell you what you're actually doing.
```

---

## Troubleshooting

### Lỗi: "Permission denied" khi push

**Nguyên nhân:** Claude Code chưa có quyền write vào repo.

**Fix:**
1. GitHub → Settings → Applications → Authorized OAuth Apps
2. Tìm "Claude Code"
3. Đảm bảo có quyền read + write cho repo `daily-motivation`

### Lỗi: Routine tạo branch `claude/xxx` thay vì push vào main

**Nguyên nhân:** Permission "Allow unrestricted git push" chưa bật, hoặc prompt chưa đủ rõ.

**Fix:**
1. Vào routine → Edit → tab Permissions → đảm bảo "Allow unrestricted git push" ON
2. Verify prompt có chứa: `DO NOT create a new branch. DO NOT use any branch with "claude/" prefix.`
3. Xoá branch cũ: GitHub repo → Branches → delete `claude/xxx`
4. Run now lại

### Lỗi: Câu motivation bị merged thành 1 dòng

**Nguyên nhân:** Markdown không có blank line giữa các dòng.

**Fix:** Verify prompt có chứa rõ:
- Blank line giữa timestamp và bullet đầu tiên
- Dùng `* ` (asterisk + space) không phải `- ` (dash)

### Câu motivation bị lặp sau vài ngày

**Fix:** Trong prompt, tăng `last 100 lines` → `last 200 lines`, hoặc thêm explicit list themes:

```text
Rotate themes in this exact order across days of the week:
- Monday: discipline & habits
- Tuesday: deep focus
- Wednesday: craft & quality
- Thursday: learning from failure
- Friday: small wins & progress
- Saturday: curiosity & exploration
- Sunday: rest & reflection
```

### Routine không chạy đúng 5h

- Delay 5-15 phút: **bình thường**, do stagger
- Delay > 30 phút: check trang status của routine xem có error log không
- Không chạy hoàn toàn: check daily limit (Pro 5 routines/ngày)

---

## Tinh chỉnh nâng cao

### Đổi giờ chạy

Tuỳ giờ bắt đầu làm việc của bạn:

| Bắt đầu làm việc | Schedule routine | Window 2 reset |
|------------------|-------------------|----------------|
| 9:00 | 5:00 | 10:00 |
| 10:00 | 6:00 | 11:00 |
| 8:00 | 4:00 | 9:00 |

Quy tắc: **routine_time = work_start − 4 giờ** (để window 1 cover bao phủ 1 giờ đầu, window 2 reset đúng lúc bạn đang nóng máy).

### Thêm hybrid section "Hôm nay"

Nếu muốn câu mới nhất hiển thị ở top dưới dạng banner riêng:

```markdown
# Daily Motivation Streak

## Hôm nay
> [Câu motivation mới nhất - overwrite mỗi sáng]

## Lịch sử
[Toàn bộ entry - prepend như cũ]
```

Update prompt: step 5 cần update 2 chỗ thay vì 1.

### Thêm Slack notification

Nếu muốn nhận thông báo qua Slack mỗi sáng:

1. Vào routine → Connectors → thêm Slack connector
2. Update prompt step 8: thay vì chỉ output, gọi Slack tool để post câu motivation vào channel #morning

### Track streak count

Thêm vào prompt step 5: đếm số entry hiện có, update dòng "🔥 Streak: N days" ở đầu README.

---

## Tham khảo

- [Claude Code Routines Docs](https://docs.claude.com/en/docs/claude-code/routines)
- [Anthropic Routines Announcement](https://claude.com/blog/introducing-routines-in-claude-code)
- [Claude Code Usage Limits](https://support.claude.com/en/articles/9797557-usage-limit-best-practices)
- [Keep a Changelog format](https://keepachangelog.com/) — pattern reverse chronological log

---

## Checklist tổng

Trước khi đi ngủ ngày đầu tiên, check lại:

- [ ] Repo `daily-motivation` đã tạo với README có dòng `## Lịch sử`
- [ ] Routine name: `Daily Motivation 5AM`
- [ ] Model: `claude-haiku-4-5`
- [ ] Prompt: đã paste đầy đủ
- [ ] Repository: `daily-motivation` đã chọn
- [ ] Connectors: trống
- [ ] Behavior: Auto-fix PRs OFF
- [ ] Permissions: Allow unrestricted git push ON
- [ ] Schedule: Daily 5:00 AM
- [ ] **Run now**: đã test thành công, có commit trên main, README render đúng

Nếu tất cả ✅, ngủ ngon. Mai mở GitHub sẽ có câu motivation mới đang chờ. 🌅
