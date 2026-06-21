# CHANGELOG.md

> Lịch sử các phiên bản của skill `luu`, cái skill lo việc đóng phiên (session close) cho vault.
> Version history for the `luu` skill, which handles session close for your vault.

---

## [2.1] - 2026-06-21

### TIẾNG VIỆT

Bản này gia cố chuyện an toàn dữ liệu. Sau một vòng soi kỹ (22 góc nhìn chuyên gia, cộng 6 lần dựng cảnh lỗi để thử), tôi vá đúng 6 lỗ thật. Không vẽ vời thêm: không khóa file, không transaction, không cơ sở dữ liệu, chỉ thêm vài dòng bash lẻ.

Sáu lỗ, kể từng cái:

**Lỗ 1 (nặng, mất trắng).** Bước 3 trước đây ghi đè thẳng lên `active.md`, ghi mù không nhìn lại. Vault này không phải git, nên mất là mất luôn, không lấy lại được. Cách sửa: đọc trước rồi mới ghi, giữ một bản `.bak` phòng thân (chép bằng `cp`), và ghi theo kiểu an toàn qua một file tạm rồi mới `mv` đè vào.

**Lỗ 1b (nặng, hai phiên đè nhau).** Hai phiên mở lệch giờ mà cùng chạy `/luu` thì hóa ra ai ghi sau là thắng, người ghi trước mất công. Cách sửa: đọc dấu thời gian của file ngay đầu phiên (`stat -f %m`, cú pháp của macOS), rồi so lại đúng lúc sắp ghi. Nếu dấu đó đã đổi, tức là có người vừa động vào, thì dừng lại hỏi, không ghi đè mù.

**Lỗ 2 (nhẹ, làm bẩn log).** Bước 2 cứ nối thêm vào `decisions.md` mà không kiểm trùng, nên đã từng đẻ ra 3 khối cùng một ngày nằm chồng lên nhau. Cách sửa: trước khi ghi thì soi xem hôm nay đã có khối chưa (`grep -q "^## $(date +%F)"`); có rồi thì thêm một dòng vào khối cũ, bỏ hẳn cái mẹo cũ phải gõ tay "(phần 2)/(phần 3)".

**Lỗ 3 (chạy lại giữa chừng thì dở dang).** Một bước hỏng giữa đường để lại trạng thái nửa nạc nửa mỡ. Cách sửa: giờ các bước ghi đều chạy lại được mà không hại gì, nên gọi `/luu` lần nữa là an toàn. Còn hai bước đụng vào dữ liệu (2 và 3) mà hỏng thì dừng lại, báo rõ ràng, không lặng lẽ đi tiếp.

**Lỗ 4 (đứt mạch vì cái hook chặn em-dash).** Cái hook `emdash-block.py` thấy ký tự gạch dài là chặn cứng (thoát mã 2), làm đứt chuỗi 5 bước ngay giữa đường. Sửa tận gốc: dọn sạch em-dash ngay từ Bước 1 lúc trích nội dung, trước khi đụng tới bất kỳ lệnh ghi nào.

**Lỗ preflight (nặng, hỏng mà vẫn báo xanh).** Nếu cấu trúc vault đổi tên một thư mục, lệnh ghi sẽ tạo file ở một đường dẫn chết, nhưng phần Output vẫn in dấu tích xanh như chưa có chuyện gì. Lừa người dùng. Cách sửa: thêm Bước 0 kiểm tra ba thư mục cha có thật không (`test -d`); thiếu cái nào thì dừng, hỏi, tuyệt đối không tự ý đẻ đường dẫn mới. Và Output cuối cùng đọc lại từ đĩa rồi in con số thật (số byte, số khối đếm được), không in tích xanh khống nữa.

Còn lại giữ y nguyên, không đổi một byte, dùng ngược về sau vẫn chạy: thứ tự 5 bước, định dạng của `active.md` và `decisions.md`, một slot `.bak` ghi đè. Tổng cộng chỉ thêm chừng 6 đến 10 dòng bash lẻ.

### ENGLISH

This release hardens data safety. After one audit round (22 expert perspectives plus 6 simulated failure scenarios), exactly 6 real holes were patched, with no over-engineering: no file lock, no transaction, no database, just a few scattered bash lines.

The six holes, one by one:

- **Hole 1 (high, total loss):** Step 3 used to overwrite `active.md` blindly. This vault is not a git repository, so a loss is permanent. Fix: read before writing, keep one `.bak` backup slot (`cp`), and write atomically via a temp file followed by `mv`.
- **Hole 1b (high, concurrent overwrite):** Two sessions opened at different times, both running `/luu`, degenerated into last-writer-wins. Fix: read the file's modification time at session start (`stat -f %m`, BSD syntax), then re-check it just before writing. If the time has changed, someone else touched the file, so stop and ask rather than overwriting blindly.
- **Hole 2 (low, log pollution):** Step 2 appended to `decisions.md` without checking for duplicates, and had already produced 3 same-day blocks stacked on top of each other. Fix: before writing, check whether today's block already exists (`grep -q "^## $(date +%F)"`); if it does, add a bullet under the existing block, and drop the old manual "(part 2)/(part 3)" convention entirely.
- **Hole 3 (half-done state on retry):** A step failing mid-chain left the data in a half-finished state. Fix: the write steps are now idempotent, so re-running `/luu` is safe. The two steps that touch data (2 and 3) stop and report clearly on failure instead of quietly moving on.
- **Hole 4 (chain break from the em-dash hook):** The `emdash-block.py` hook hard-blocks any write containing U+2014 (exit code 2), snapping the 5-step chain mid-way. Fix at the source: sanitize em-dashes right at Step 1 (extraction), before touching any write command.
- **Preflight hole (high, silent failure with a green check):** If the vault structure renames a directory, write commands create files at a dead path while Output still prints a false green check, misleading the user. Fix: add a Step 0 that checks the three parent directories actually exist (`test -d`); if any is missing, stop and ask, never self-create a new path. The final Output reads back from disk and prints real numbers (bytes, block count) instead of a false green check.

Everything else is kept as-is, zero byte change, backward-compatible: the ordered 5-step structure, the `active.md` and `decisions.md` formats, the single overwrite `.bak` slot. About 6 to 10 scattered bash lines were added in total.

---

## [2.0] - 2026-06-19

### TIẾNG VIỆT

Bản này dọn nhà sau khi đổi vault. Bản v1 còn trỏ về vault cũ và mang theo mấy bước đã chết trong đợt xóa dọn trước đó, nên giờ gỡ ra cho gọn.

Đổi những gì:

- Trỏ sang vault mới.
- Bỏ các bước đã chết: hàng đợi JSONL (collector cũ, đã dẹp cùng cái cron chạy lén), bước kiểm hàng tuần (chưa cần tới), script `check-brain.sh` (đã thất lạc), và bước save-session bên hệ thống Code.
- Thêm Bước 5: chốt lại phần AI-OS theo đúng file hướng dẫn của vault (doctrine, các generator, skill, MCP, membrane).

### ENGLISH

This release cleans house after a vault change. Version 1 still pointed at the old vault and carried several steps that died in an earlier cleanup, so those were removed.

What changed:

- Repointed to the new vault.
- Removed dead steps: the JSONL backlog (old collector, killed along with a stray cron job), the weekly check (not needed yet), the `check-brain.sh` script (lost), and the Code-side save-session step.
- Added Step 5: capture the AI-OS section per the vault's instructions file (doctrine, generators, skills, MCP, membrane).

---

## [1.0] - 2026-05-03

### TIẾNG VIỆT

Bản đầu tiên. Lo việc đóng phiên cho vault cũ: ghi lại các quyết định, cập nhật trạng thái đang chạy, ghi daily note, cộng vài bước pipeline mà giờ đã bỏ.

### ENGLISH

First release. Handles session close for the old vault: record decisions, update the active state, write a daily note, plus a few pipeline steps that have since been removed.
