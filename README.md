# luu, Session Close cho vault

[![License: Apache-2.0](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](./LICENSE) [![Version](https://img.shields.io/badge/version-2.1-green.svg)](#ba-thời-kỳ) [![Claude Code Skill](https://img.shields.io/badge/Claude_Code-Skill-8A2BE2.svg)](#) [![No external deps](https://img.shields.io/badge/deps-none-brightgreen.svg)](#cài-đặt-và-cấu-hình)

*Demo: phiên `/luu` chạy trong 15 giây, GIF sắp bổ sung. A 15s `/luu` run, demo GIF coming soon.*
<!-- Khi có GIF: lưu vào assets/demo.gif rồi thay dòng trên bằng: ![demo](assets/demo.gif) -->

> Tài liệu này song ngữ. Bản tiếng Việt trước, bản English sau. Không dùng dấu gạch dài em-dash ở bất kỳ đâu.

---

## Tiếng Việt

### luu là gì, và nó gỡ cái gì cho bạn

`luu` là một skill đóng phiên. Nói nôm na: khi bạn làm xong một buổi với Claude, nó là cái lệnh dọn dẹp và cất lại những gì đáng giữ vào quyển sổ nhớ của bạn. Gọi bằng `/luu`, hoặc nói "lưu session", "đóng session", "save session".

Vấn đề nó gỡ thì ai làm việc dài hơi với AI cũng gặp. Cuối mỗi buổi, có những thứ phải đọng lại: quyết định vừa chốt, project đang dở tới đâu, ghi chú trong ngày, và dấu vết những lần bạn chạm vào hệ sinh thái AI của mình. Không có một lệnh đóng phiên đáng tin, hai chuyện hay xảy ra. Một, bạn quên ghi, buổi sau mở ra lại bắt đầu từ con số không. Hai, bạn ghi vội, ghi đè lên dữ liệu cũ, và vì quyển sổ này không phải kho git, mất là mất trắng.

`luu` chạy 5 bước theo đúng một thứ tự, ghi vào bốn chỗ:
- `00_SYSTEM/decisions.md` (quyết định)
- `00_SYSTEM/active.md` (trạng thái hiện tại)
- `40-archive/daily-notes/YYYY-MM-DD.md` (ghi chú ngày)
- `20-areas/AI-OS/` (lưu lại dấu vết hệ AI, chỉ khi buổi đó có chạm vào AI)

Cái lõi của nó gói trong một câu: mỗi bước ghi đè phải sao lưu (.bak) trước đã, mỗi bước thêm dòng phải chạy lại được mà không nhân đôi. Đó là cái giá rẻ nhất để một lệnh đóng phiên không tự tay phá quyển sổ của bạn.

### Ba thời kỳ

**v1, thời trước clean-room.** Bản này trỏ vào quyển sổ cũ. Nhiều bước trong nó đã chết theo một đợt dọn lớn: một đường ống thu gom dữ liệu kiểu cũ đã bị diệt cùng một cron zombie, một bước kiểm theo tuần chưa bao giờ cần tới, một script đã thất lạc, một bước save vốn thuộc phía công cụ khác. Nói gọn, v1 lỗi thời, và trỏ vào một chỗ không còn tồn tại nữa.

**v2.0, sau clean-room.** Bỏ hết bước chết, trỏ sang quyển sổ mới, thêm Bước 5 lưu dấu vết hệ AI. Đây là bản di trú, làm cho skill sống lại đúng địa chỉ.

**v2.1, sau một vòng audit của hội đồng 22 chuyên gia.** Đây là bản gia cố. Vòng soi đó tìm ra 6 lỗ hổng an toàn dữ liệu vẫn còn sống trong v2.0. v2.1 vá cả 6 bằng vài dòng bash, và cố tình không làm quá tay: không khóa file, không transaction, không database. Vẫn tương thích ngược, không đụng tới cấu trúc file.

### Hôm nay luu làm được gì (năng lực v2.1)

- **Bước 0, Preflight.** Trước khi ghi bất cứ thứ gì, nó kiểm ba thư mục cha có thật không. Thiếu cái nào thì dừng lại, hỏi bạn, tuyệt đối không tự đẻ ra đường dẫn mới. Hết cảnh thất bại trong im lặng mà màn hình vẫn báo xanh.
- **Bước 1, Bóc và làm sạch.** Đọc lại buổi vừa rồi, rút ra quyết định, trạng thái, dữ kiện, dấu vết AI. Ngay tại đây khử sạch em-dash trong phần nội dung sắp ghi, đổi sang dấu phẩy, dấu hai chấm, hoặc tách câu. Cái hook chặn em-dash từ đó chỉ còn là tấm lưới an toàn cuối cùng, không còn là chỗ làm đứt mạch giữa chuỗi 5 bước.
- **Bước 2, decisions.md, không nhân đôi.** Chỉ ghi khi có quyết định thật. Tìm khối ngày hôm nay trước khi thêm: chưa có thì tạo khối mới, có rồi thì thêm một gạch đầu dòng vào dưới khối đó, không bao giờ đẻ header trùng. Bỏ hẳn cái lệ ghi tay "(phần 2), (phần 3)".
- **Bước 3, active.md, đọc trước khi ghi.** Đọc bản cũ để giữ lại dấu tay của bạn, kiểm thời điểm sửa cuối để phòng hai phiên ghi đè nhau, chép ra một bản .bak, rồi ghi gọn qua một file tạm và đổi tên. Hết cảnh active.md bị xóa sạch còn trơ giấy trắng.
- **Bước 4, ghi chú ngày.** Có file rồi thì thêm vào mục dữ kiện mới, chưa có thì tạo mới. Không ghi đè, không lặp header.
- **Bước 5, lưu dấu vết hệ AI.** Chỉ khi buổi đó chạm vào hệ AI: ghi doctrine vào đúng file của nó, hoặc một ghi chú để buổi sau nối lại nếu phiên sắp bị nén.
- **Khi có sự cố.** Hai bước ghi dữ liệu nặng (2 và 3) mà hỏng thì dừng và báo rõ bước nào đã xong; vì các bước đều chạy lại được không nhân đôi, nên gọi lại `/luu` là an toàn. Hai bước nhẹ hơn (4 và 5) thì ghi một lời cảnh báo rồi đi tiếp.
- **Output là bằng chứng, không phải lời tuyên bố.** Cuối cùng nó đọc lại từ ổ đĩa rồi in ra con số thật: active.md nặng bao nhiêu byte, bản .bak có tồn tại không, đếm được mấy khối ngày trong decisions.md, ghi chú ngày có chứa không. Bước nào hỏng thì in dấu cảnh báo đỏ, không bao giờ in một dấu tick xanh giả.

### Đã tính trước bao nhiêu trường hợp

`luu` v2.1 không viết ra từ cảm tính. Nó đi qua mấy lớp kiểm chứng.

- **6 kịch bản mô phỏng lỗi** riêng cho `luu`: hai phiên song song cùng ghi active.md (kẻ ghi sau thắng, mất trắng), chuỗi 5 bước đứt giữa chừng vì một em-dash, thao tác thêm vào decisions.md không idempotent làm nhiễm log, cấu trúc quyển sổ trôi đi (thư mục bị đổi tên) gây thất bại trong im lặng mà vẫn báo xanh, cùng vài biến thể khác. Mỗi kịch bản đều gắn với một thất bại thật và một tấm chắn rẻ nhất cho nó.
- **22 lăng kính chuyên gia** soi từng bước, lặp tới khi đồng thuận, kết luận: thông qua.
- **Các issue cộng đồng đã kiểm qua GitHub API** (khớp tiêu đề thật, không đoán từ trí nhớ): #27311 plan file bị ghi đè giữa các phiên song song, #13744 PreToolUse exit 2 không chặn được Write/Edit (đã fix ở CHANGELOG 2.1.90), #24846 lệnh chặn đọc không có hiệu lực với .env, #40226 ghi song song làm hỏng file cấu hình trên macOS.

Chi tiết 6 mô phỏng cộng 22 lăng kính: xem [TESTS.md](./TESTS.md).

### Vì sao tin được (phương pháp luận)

Đây không phải một prompt viết phóng tay. `luu` v2.1 là sản phẩm cuối của một hành trình 8 chặng, qua nhiều vòng kiểm rồi sai rồi sửa, tốn không ít credit và token (8 workflow, hơn 100 sub-agent, hàng triệu token):

1. **Radar GitHub.** Cào 1307 repo, kiểm chứng qua GitHub API, phân tầng 332 cái (dùng ngay, theo dõi, hoặc bỏ).
2. **Hóa thân.** Chưng cất trí tuệ của 22 chuyên gia cộng với chính cái harness của Claude Code, rồi xác tín chéo song song.
3. **Bản thiết kế.** Soi hệ Claude Code trên 9 chiều, thiết kế một harness tối giản, kèm một lớp soi chéo để chặn cái tật làm quá tay.
4. **Giả lập.** Mô phỏng 70 điểm vỡ, đối chiếu cộng đồng, xác nhận được 18.
5. **Hội đồng.** 22 chuyên gia lặp tới khi đồng thuận.
6. **Audit vòng hai, kiểm lại chính cái xác tín.** Xác nhận 11 issue GitHub là có thật qua API, lật ngược 2 trụ sai (cái hook exit-2 hóa ra đã được fix ở 2.1.90, và stdout lúc khởi động phiên không hề làm chết chương trình).
7. **Dọn dẹp.** Trả tool, skill, plugin về đúng bộ của chúng, tách bạch mọi thứ cho gọn.
8. **Audit riêng cho luu (chính chặng này).** 22 chuyên gia soi riêng `luu`, 6 mô phỏng lỗi, viết lại thành v2.1, soi chéo một lần cuối.

Điểm cốt lõi nằm ở đây: mọi điều khẳng định đều được kiểm lại, cộng đồng trước rồi mới tới web hay nguồn chính thức, và chính cái việc kiểm đó cũng bị đem ra kiểm. Khi bằng chứng lật một giả định (ví dụ cái hook exit-2 hóa ra đã fix rồi), skill nhận và sửa, không bảo thủ.

### Cài đặt và cấu hình

Clone repo về thư mục skill của Claude Code:

```bash
git clone https://github.com/hoangCreative/claude-session-save.git ~/.claude/skills/luu
```

Skill sống ở `<CLAUDE_HOME>/skills/luu/SKILL.md`.

Cấu hình duy nhất là đường dẫn gốc của quyển sổ, đặt trong frontmatter của SKILL.md (`vault:`) và dùng lại trong mọi bước:

```
<VAULT_ROOT> = đường dẫn tới quyển sổ của bạn
```

Nếu dùng trên máy khác, đổi `<VAULT_ROOT>` cho khớp và bảo đảm ba thư mục cha tồn tại:
- `<VAULT_ROOT>/00_SYSTEM`
- `<VAULT_ROOT>/40-archive/daily-notes`
- `<VAULT_ROOT>/20-areas/AI-OS`

Bước 0 Preflight sẽ tự kiểm ba thư mục này. Thiếu thì skill dừng và báo, không tự đẻ đường dẫn mới.

Không cần cài thêm gì. Không database, không daemon, không cron. Mọi tấm chắn chỉ là 1 đến 3 dòng bash (`cp`, `mv`, `stat`, `grep`, `test`).

### License

Xem [LICENSE](./LICENSE).

---

## English

### What luu is, and the failure it removes

`luu` (Vietnamese for "save") is a session-close skill for a personal memory vault. Invoke it with `/luu`, or say "luu session", "dong session", or "save session".

The problem it solves is one anyone running long-form work with an AI will recognize. At the end of every working session, some things need to persist: decisions just made, how far the current project got, notes for the day, and any trace of touching your AI ecosystem. Without a trustworthy close command, two failures recur. One, you forget to write, so the next session starts from zero. Two, you write in a hurry and overwrite older data, and because this vault is not a git repository, loss is total.

`luu` runs 5 ordered steps, writing to four places:
- `00_SYSTEM/decisions.md` (decisions)
- `00_SYSTEM/active.md` (current state)
- `40-archive/daily-notes/YYYY-MM-DD.md` (daily note)
- `20-areas/AI-OS/` (AI-ecosystem capture, only when the session touched AI)

The core fits in one sentence: every overwrite step must take a `.bak` first, and every append step must be re-runnable without duplicating. That is the cheapest price for a close command that does not destroy your vault by its own hand.

### Three eras

**v1, pre clean-room.** This version pointed at the old vault. Several of its steps had died in a large cleanup: an old data-collection pipeline killed alongside a zombie cron, a weekly check that was never actually needed, a lost script, and a save step that belonged to a different tool entirely. In short, v1 was stale and pointed at a place that no longer existed.

**v2.0, post clean-room.** Removed every dead step, repointed to the new vault, added Step 5 to capture AI-ecosystem traces. This was the migration release, bringing the skill back to life at the correct address.

**v2.1, post 22-expert council audit.** This is the hardening release. That review found 6 data-safety holes still alive in v2.0. v2.1 patched all 6 with a few lines of bash, deliberately without overbuilding: no file lock, no transaction, no database. Still backward-compatible, with no change to file structure.

### What luu does today (v2.1 capabilities)

- **Step 0, Preflight.** Before writing anything, it checks that the three parent directories actually exist. If any is missing, it stops, asks you, and never auto-creates a new path. This ends the case of a silent failure behind a green checkmark.
- **Step 1, Extract and sanitize.** Re-read the session, pull out decisions, state, facts, and AI traces. Right here it strips every em-dash from the content about to be written, swapping in commas, colons, or sentence breaks. The em-dash-blocking hook then serves only as a final safety net, no longer a break point in the middle of the 5-step chain.
- **Step 2, decisions.md, no duplication.** Write only when there is a real decision. Look for today's date block before appending: if absent, create a new block; if present, add a bullet under it, never a duplicate header. The manual "(part 2), (part 3)" convention is gone for good.
- **Step 3, active.md, read before write.** Read the old version to preserve your fingerprints, check the last-modified time to guard against two sessions overwriting each other, copy out a `.bak`, then write cleanly through a temp file and a rename. This ends the case of active.md wiped down to a blank page.
- **Step 4, daily note.** If the file exists, append to the new-facts section; if not, create it. No overwrite, no repeated header.
- **Step 5, AI-ecosystem capture.** Only when the session touched the AI ecosystem: write doctrine into its proper file, or a resume note so the next session can pick up the thread if the current one is near compaction.
- **When something fails.** The two heavy data-write steps (2 and 3) that fail will stop and report exactly which step finished; because the steps are re-runnable without duplication, calling `/luu` again is safe. The two lighter steps (4 and 5) log a warning and move on.
- **Output is evidence, not a claim.** At the end it re-reads from disk and prints real numbers: how many bytes active.md weighs, whether the `.bak` exists, how many date blocks are in decisions.md, whether the daily note contains the entry. Any failed step prints a red warning, never a fake green tick.

### How many cases were anticipated

`luu` v2.1 was not written from feel. It passed through several verification layers.

- **6 failure simulations** specific to `luu`: two concurrent sessions both writing active.md (last writer wins, total loss), the 5-step chain breaking mid-way on a single em-dash, a non-idempotent append to decisions.md polluting the log, vault structure drift (a renamed directory) causing a silent failure that still reports green, plus other variants. Each scenario is tied to a real failure and the cheapest guard against it.
- **22 expert lenses** reviewing each step, looping to consensus, verdict: approved.
- **Community issues verified via the GitHub API** (real titles matched, not recalled from memory): #27311 plan files overwritten across concurrent sessions, #13744 PreToolUse exit 2 failing to block Write/Edit (fixed in CHANGELOG 2.1.90), #24846 a read-deny rule not enforced for .env, #40226 concurrent writes corrupting a config file on macOS.

Details of the 6 simulations plus 22 lenses: see [TESTS.md](./TESTS.md).

### Why it is trustworthy (methodology)

This is not a freehand prompt. `luu` v2.1 is the end product of an 8-stage journey, through many check-fail-fix rounds, at real credit and token cost (8 workflows, over 100 sub-agents, millions of tokens):

1. **GitHub Radar.** Scraped 1307 repos, verified via the GitHub API, tiered 332 of them (use now, watch, or drop).
2. **Embodiment.** Distilled the wisdom of 22 experts plus the Claude Code harness itself, then cross-verified in parallel.
3. **Blueprint.** Audited the Claude Code system across 9 dimensions, designed a minimal harness, with a cross-check layer to stop the overbuilding habit.
4. **Simulation.** Modeled 70 break points, checked against community evidence, confirmed 18.
5. **Council.** 22 experts looping to consensus.
6. **Verification audit, round 2, checking the verification itself.** Confirmed 11 GitHub issues real via the API, overturned 2 false pillars (the exit-2 hook turned out to be already fixed in 2.1.90, and session-start stdout does not in fact crash the program).
7. **Cleanup.** Returned tools, skills, and plugins to their proper sets, separating concerns cleanly.
8. **luu audit (this stage).** 22 experts reviewing `luu` specifically, 6 failure simulations, the v2.1 rewrite, one final cross-check.

The crux is here: every assertion was re-verified, community first and only then web or official sources, and the act of verifying was itself audited. When evidence overturned an assumption (for example, the exit-2 hook turning out to be already fixed), the skill accepted it and corrected course, with no defensiveness.

### Installation and configuration

Clone the repo into your Claude Code skills directory:

```bash
git clone https://github.com/hoangCreative/claude-session-save.git ~/.claude/skills/luu
```

The skill lives at `<CLAUDE_HOME>/skills/luu/SKILL.md`.

The only configuration is the vault root path, set in the SKILL.md frontmatter (`vault:`) and reused in every step:

```
<VAULT_ROOT> = path to your vault
```

On a different machine, change `<VAULT_ROOT>` to match and ensure the three parent directories exist:
- `<VAULT_ROOT>/00_SYSTEM`
- `<VAULT_ROOT>/40-archive/daily-notes`
- `<VAULT_ROOT>/20-areas/AI-OS`

Step 0 Preflight checks these three automatically. If any is missing, the skill stops and reports, never auto-creating a new path.

Nothing else to install. No database, no daemon, no cron. Every guard is 1 to 3 lines of bash (`cp`, `mv`, `stat`, `grep`, `test`).

### Credits

Author: Le Cong Hoang, a Vietnamese Reflector and strategic advisor. Contact: leconghoangstudio@gmail.com

### License

See [LICENSE](./LICENSE). Apache-2.0.

---

## Star this repo if it helps / Cho một sao nếu hữu ích

Nếu `luu` giúp được buổi làm việc của bạn, một ngôi sao trên GitHub là cách rẻ nhất để người sau tìm thấy nó. / If `luu` saved you a session, a star is the cheapest way to help the next person find it.

**Cite / Trích dẫn.** Nếu bạn dùng `luu` trong nghiên cứu hay bài viết, xem [CITATION.cff](./CITATION.cff) để lấy đúng định dạng trích dẫn. / If you use `luu` in research or writing, see [CITATION.cff](./CITATION.cff) for the citation format.
