---
name: luu
version: "2.1"
updated: "2026-06-21"
description: >
  Session close protocol cho vault Obsidian/PARA chay trong Claude Code. Invoke: /luu, "luu session", "dong session", "save session".
  5 buoc theo thu tu: extract+sanitize -> decisions.md (idempotent) -> active.md (doc-truoc-ghi, .bak, mtime-check) -> daily note -> capture AI-OS neu cham AI.
metadata:
  author: Le Cong Hoang
  copyright: © 2026 Le Cong Hoang. Licensed under Apache-2.0.
  license: Apache-2.0
  created: "2026-05-03"
---

# luu - Session Close (Vault PARA)

Đóng phiên cho gọn ghẽ: gom hết trí nhớ của phiên vào vault, trong một lệnh.
Gọi bằng: `/luu`, hoặc "lưu session", hoặc "đóng session".

**Khung phiên này:** bạn đang ở cuối phiên. Việc cần làm: cất 5 mảng trí nhớ vào vault. Cần bạn can thiệp không? Thường là không, trừ khi giờ sửa file lệch nhau hoặc thiếu thư mục, lúc đó skill dừng lại hỏi. Thế nào là xong? active.md có bản `.bak` và lớn hơn 0 byte, decisions.md không trùng khối ngày, daily note đã nằm đó, và phần Output in ra số thật chứ không phải lời hứa suông.

---

## CONFIG (đọc trước, đặt một lần)

Cả skill chạy quanh một biến gốc: `VAULT_ROOT`. Mọi đường dẫn đều bắt từ đó. Trước khi dùng, thay placeholder `<VAULT_ROOT>` bằng đường dẫn tuyệt đối tới vault của bạn, tức thư mục PARA chứa `00_SYSTEM`, `40-archive`, `20-areas`.

Cách làm gọn nhất: đầu mỗi lệnh bash, đặt biến `V` trỏ về vault của bạn.
```bash
V="<VAULT_ROOT>"          # vi du: V="/Users/ten-ban/Documents/vault"
```
Đường dẫn có dấu cách thì luôn nhớ bọc trong ngoặc kép (`"$V/..."`). Mọi path bên dưới đều tính từ `$V`.

Cấu trúc vault mà skill này giả định (PARA tối giản):
```
<VAULT_ROOT>/
├── 00_SYSTEM/                ← active.md, decisions.md, principles.md, context.md
├── 20-areas/AI-OS/           ← doctrine, generator registry, skill/MCP notes
└── 40-archive/daily-notes/   ← YYYY-MM-DD.md
```
Vault của bạn đặt tên thư mục khác thì sửa ba path trong skill cho khớp: `00_SYSTEM`, `40-archive/daily-notes`, `20-areas/AI-OS`. Đừng để skill tự dựng path mới khi không khớp, lý do nằm ở Bước 0.

---

## Vì sao có hàng rào (đọc trước khi định cắt cho gọn)

Skill này giả định vault của bạn KHÔNG nằm trong git. Mất file là mất trắng, không có chỗ nào kéo lại. Vì thế mọi bước ghi đè đều phải có `.bak` trước, mọi bước nối thêm đều phải idempotent, tức chạy lại nhiều lần cũng không sinh trùng. Đây là cái giá tối thiểu để một lệnh đóng phiên không tự tay phá vault, không phải làm cho phức tạp. Đừng tháo `.bak`, đừng tháo bước so giờ sửa file, đừng tháo `grep` vì tưởng nó thừa.

---

## Khi nào dùng
- Cuối mỗi phiên.
- Trước khi chạm trần context hoặc trước khi compact.
- Sau khi vừa có một quyết định quan trọng cần giữ lại.

---

## Thứ tự, KHÔNG đảo bước

### Bước 0: Preflight (chỉ đọc, rẻ, không ghi gì)

Trước khi đụng vào bất cứ lệnh ghi nào, kiểm xem ba thư mục cha có còn đó không:
```bash
V="<VAULT_ROOT>"
test -d "$V/00_SYSTEM" && test -d "$V/40-archive/daily-notes" && test -d "$V/20-areas/AI-OS" && echo "preflight OK" || echo "preflight FAIL"
```
Nếu FAIL thì DỪNG, báo bạn biết thiếu thư mục nào, và KHÔNG tự tạo path mới. Cấu trúc vault có thể đã đổi tên, tự tạo là ghi vào một chỗ chết.

Ghi nhớ giờ sửa cuối của active.md để Bước 3 còn so lại:
```bash
stat -f %m "$V/00_SYSTEM/active.md" 2>/dev/null
```
Cất số này lại, gọi là `MTIME_DAU`. File chưa có thì để trống, Bước 3 hiểu đó là tạo mới.

Lưu ý hệ điều hành: `stat -f %m` là cú pháp BSD, dùng cho macOS. Trên Linux thì đổi sang `stat -c %Y`.

---

### Bước 1: Rút từ phiên ra, kèm SANITIZE em-dash (TRƯỚC khi viết bất cứ thứ gì)

Đọc lại cả phiên. Tìm bốn thứ:

**Decisions.** Những gì bạn quyết dứt khoát: đồng ý hay không, chọn A bỏ B, đặt một luật mới, khai tử một hướng. Cái không tính: bàn luận, các lựa chọn đang cân, brainstorm chưa chốt.

**Active state.** Project nào đang mở? Có vướng gì thật không? Tâm điểm lúc này là gì?

**Daily summary.** Hôm nay làm gì? Sự thật mới, quyết định, câu hỏi còn để ngỏ?

**AI-OS touch.** Phiên này có chạm vào hệ AI không, tức doctrine, generator, skill, MCP, memory? Có thì giữ lại, Bước 5 sẽ capture.

**SANITIZE (bắt buộc, làm ngay tại đây).** Trước khi đụng bất kỳ lệnh ghi nào, quét hết nội dung sắp ghi và thay mọi dấu gạch dài (em-dash, U+2014) bằng dấu phẩy, dấu hai chấm, hoặc tách thành câu mới. Lý do: nếu vault của bạn có hook chặn em-dash (exit 2), nó sẽ cắt đứt chuỗi 5 bước ngay giữa chừng. Làm sạch ngay tại nguồn thì hook chỉ còn là tấm lưới an toàn cuối cùng, không còn là chỗ đứt mạch.

Ghi kết quả rút ra (đã sạch em-dash), nhưng chưa viết file nào.

---

### Bước 2: decisions.md, IDEMPOTENT, chỉ khi có quyết định thật

File: `00_SYSTEM/decisions.md`

**Đừng nối thêm nếu** không có quyết định nào rõ ràng. Không bịa, không gói một cuộc bàn luận thành "quyết định".

Có quyết định thật thì kiểm xem khối ngày hôm nay đã có chưa, TRƯỚC khi ghi:
```bash
V="<VAULT_ROOT>"
grep -q "^## $(date +%F)" "$V/00_SYSTEM/decisions.md" && echo "co khoi ngay" || echo "chua co"
```
- **Chưa có:** dựng khối mới.
- **Có rồi:** thêm một bullet MỚI vào dưới khối ngày đó, đừng tạo header trùng. Bỏ hẳn lối đặt tay "(phần 2)/(phần 3)", đó là né lỗi chứ không phải sửa lỗi.

Format khối ngày (tiền tố `## YYYY-MM-DD` vẫn khớp cả với tiêu đề dạng `## YYYY-MM-DD - tiêu đề`):
```markdown
## YYYY-MM-DD
*Daily: [[YYYY-MM-DD]]*
- **Tên quyết định:** mô tả ngắn, kèm lý do nếu có
```

Bước này mà ghi lỗi thì DỪNG, báo rõ "Bước 2 fail, decisions.md chưa ghi". Đừng lặng lẽ đi tiếp.

---

### Bước 3: active.md, ĐỌC-TRƯỚC-GHI, .bak, kiểm giờ sửa, ghi atomic

File: `00_SYSTEM/active.md`. Chạy mỗi phiên, nhưng không bao giờ ghi mù.

1. **Đọc bản cũ trước.** Mở active.md hiện tại để giữ lại những mục bạn tự thêm vào (ví dụ "Sản phẩm phiên", "Mood / energy" bạn đã điền). Dựng bản mới TỪ bản cũ cộng phần cập nhật, đừng dựng lại từ template trắng rồi làm mất dấu tay của bạn.

2. **Kiểm giờ sửa, chống hai phiên ghi đè nhau.** Đọc lại giờ sửa ngay trước khi ghi, rồi so với `MTIME_DAU` ở Bước 0:
```bash
V="<VAULT_ROOT>"
stat -f %m "$V/00_SYSTEM/active.md" 2>/dev/null
```
Nếu giờ sửa đã đổi, tức một phiên khác vừa ghi vào giữa chừng: DỪNG. Cứ tạo `.bak` trước cho chắc, rồi hỏi bạn: "active.md vừa bị một phiên khác sửa, .bak đã lưu, ghi đè hay gộp?". KHÔNG ghi đè mù.

3. **Sao lưu một slot trước khi ghi:**
```bash
cp "$V/00_SYSTEM/active.md" "$V/00_SYSTEM/active.md.bak"
```
Một bản `.bak` gần nhất, mỗi lần ghi đè lên, không giữ nhiều bản.

4. **Ghi atomic qua file tạm rồi mv:** ghi nội dung ra `active.md.tmp` trước, rồi
```bash
mv "$V/00_SYSTEM/active.md.tmp" "$V/00_SYSTEM/active.md"
```
`mv` là một thao tác nguyên khối: có đứt giữa chừng cũng không để lại một file rách.

Bước này mà fail thì DỪNG, báo rõ "Bước 3 fail" và cho biết `.bak` vẫn còn nguyên. Đừng lặng lẽ đi tiếp.

Format active.md:
```markdown
# active.md - Trạng thái hiện tại
*Đọc bản cũ trước khi ghi, backup .bak, ghi atomic. Không archive.*
*Xem thêm: [[decisions]] · [[principles]] · [[context]]*

## Cập nhật: YYYY-MM-DD

## Đang tập trung vào
[1-2 việc thật sự đang làm]

## Projects đang open
[chỉ project có hoạt động gần đây, kèm wikilinks]

## Blockers / pending
[chỉ vướng mắc thật, không placeholder]

## Quyết định vừa đưa ra
Xem [[decisions]] khối YYYY-MM-DD.

## Resume sau compact
[link resume note nếu sắp compact]

## Mood / energy
*[bạn tự điền nếu muốn, giữ nguyên nếu bản cũ đã có]*
```

---

### Bước 4: Daily note

File: `40-archive/daily-notes/YYYY-MM-DD.md`

**Kiểm trước:**
```bash
V="<VAULT_ROOT>"
ls "$V/40-archive/daily-notes/$(date +%F).md" 2>/dev/null
```
- Chưa có: tạo mới từ ngữ cảnh phiên.
- Đã có: nối nội dung mới vào dưới mục "Facts mới" (đánh dấu giờ nếu muốn), KHÔNG tạo file mới, KHÔNG lặp lại header.

Format:
```markdown
# Daily Note - YYYY-MM-DD

## Facts mới
[tối đa 5 bullets, sự thật thật, không lặp thứ đã biết]

## Decisions
Xem [[decisions]] khối YYYY-MM-DD.

## Questions / Follow-up
[câu hỏi còn để ngỏ]

## Sessions
*[N phiên hôm nay, ghi rõ mặt: Code/Cowork]*

---
*Links: [[active]] · [[decisions]]*
*Synthesized: YYYY-MM-DD | Manual*
```

---

### Bước 5: Capture AI-OS (chỉ khi Bước 1 thấy phiên có chạm AI)

Phiên có động vào hệ AI thì cập nhật nhà chính ở `20-areas/AI-OS/`:
- Doctrine hoặc generator mới, hoặc vừa sửa: ghi vào file registry doctrine (file tiêu thụ chỉ giữ trigger cộng link).
- Skill, MCP, membrane có đổi: ghi vào file tương ứng trong `20-areas/AI-OS/`.
- Sắp compact: viết một resume note `20-areas/AI-OS/NNN-resume-after-compact-YYYY-MM-DD.md`, nối tiếp note trước, đủ để sau compact đọc vào là bắt lại mạch.

Không có gì chạm AI thì bỏ qua bước này.

---

## Xử lý khi fail

Vì các bước ghi giờ đã idempotent (`.bak` cộng kiểm giờ sửa cho active, `grep` cho decisions), chạy lại `/luu` sau khi đứt giữa chừng là AN TOÀN: không trùng, không mất.

- **Bước ghi dữ liệu (2 và 3):** fail thì DỪNG và báo rõ bước nào đã xong, bước nào chưa. Đừng lặng lẽ đi tiếp, để lại trạng thái dở dang là khó đọc về sau.
- **Bước 4 và 5 (daily, AI-OS, ít rủi ro hơn):** fail thì log `⚠️ Bước N: [lỗi ngắn]` rồi tiếp bước sau, không để crash.

Cuối cùng báo một summary các lỗi.

---

## Output cuối cùng (bằng chứng, không phải lời tuyên bố)

Ghi xong thì đọc lại từ đĩa rồi in số thật:
```bash
V="<VAULT_ROOT>"
wc -c < "$V/00_SYSTEM/active.md"                          # so byte active.md, phai > 0
test -f "$V/00_SYSTEM/active.md.bak" && echo "bak OK"      # xac nhan .bak ton tai
grep -c "^## $(date +%F)" "$V/00_SYSTEM/decisions.md"      # dem khoi ngay hom nay; ngay cu co the nhieu khoi tu quy uoc cu, binh thuong
test -f "$V/40-archive/daily-notes/$(date +%F).md" && echo "daily OK"
```

Mẫu output:
```
Vault updated (đã verify từ đĩa):
  - decisions.md: [N bullets thêm vào khối ngày / skipped] | khoi ngay hom nay ton tai (grep -q)
  - active.md: [số byte] byte | .bak: [OK / thiếu]
  - daily note YYYY-MM-DD: [created / updated / OK]
  - AI-OS capture: [files / skipped]
  - resume note: [created / N/A]
```
Bước nào fail thì in dấu cảnh báo đỏ `⚠️` cho bước đó, KHÔNG in tick xanh khống.

---

## Anti-patterns
- KHÔNG bịa decisions, không chắc thì bỏ qua.
- KHÔNG ghi đè active.md mù: luôn `.bak` trước, luôn kiểm giờ sửa.
- KHÔNG tạo header ngày trùng trong decisions.md: grep trước, gộp vào khối có sẵn.
- KHÔNG để em-dash lọt vào nội dung sắp ghi: sanitize ở Bước 1.
- KHÔNG đè daily note đã có nếu chỉ định nối thêm.
- KHÔNG tự tạo path mới khi preflight fail.
- KHÔNG để file tiêu thụ (CLAUDE.md) chép định nghĩa dài, đẩy về file registry doctrine.

---

## Ghi chú gia cố (2026-06-21, v2.1)
v2.1 vá 6 lỗ data-safety mà KHÔNG làm phức tạp lên (không file-lock, không transaction, không DB, chỉ thêm vài dòng bash):
- Bước 3 từ ghi-đè-mù đổi sang đọc-trước-ghi cộng `.bak` cộng kiểm giờ sửa cộng ghi atomic (`mv`). Vá lỗ mất trắng active.md (vault không git).
- Bước 2 idempotent bằng grep khối ngày, bỏ lối đặt tay "(phần 2)". Vá lỗ nối trùng khi chạy lại trong cùng ngày.
- Bước 0 preflight `test -d` ba thư mục cha: thiếu thì dừng, không ghi vào path chết (vá lỗ fail âm thầm mà vẫn báo xanh).
- Sanitize em-dash ngay ở Bước 1 (tại nguồn) để hook không cắt đứt chuỗi 5 bước.
- Xử lý fail: bước ghi (2, 3) fail thì dừng và báo; chạy lại an toàn nhờ idempotent.
- Output đọc lại từ đĩa, in số byte cộng đếm khối, không in tick xanh khống.

## Ghi chú di trú (2026-06-19, v2.0)
v1 trỏ về vault cũ và có vài bước đã chết theo đợt wipe: JSONL backlog (pipeline collector cũ), weekly check (chưa cần tới), check-brain.sh (script đã mất), ECC save-session (đồ phía Code). v2 bỏ hết, trỏ sang vault mới, thêm Bước 5 capture AI-OS theo CLAUDE.md.
