# METHODOLOGY - Cách dựng skill `luu` v2.1 / How `luu` v2.1 was built

> Author: Le Cong Hoang, Vietnamese Reflector, strategic advisor
> Contact: via GitHub issues at https://github.com/hoangCreative/claude-session-save
> Subject: the methodology that produced skill `luu` (claude-session-save) v2.1
> Companion file: see TESTS.md for the counted results this methodology produced.

---

## TIẾNG VIỆT

### Đang ở đâu, file này làm gì

File này không kể `luu` làm được gì. Nó kể `luu` được dựng ra THẾ NÀO. Phân biệt này quan trọng: một prompt hay có thể viết trong mười phút, còn một skill mà bạn dám để nó ghi đè trí nhớ phiên của mình thì phải đi qua thử nghiệm. File này là bản ghi cách thử nghiệm đó chạy, đủ chi tiết để người khác lặp lại, soi lại, hoặc bác lại.

Cần ở bạn: nếu bạn định trích dẫn `luu` như một tài liệu kỹ thuật, đây là phần "phương pháp" của nó. TESTS.md là phần "kết quả". Hai file đọc cùng nhau.

Xong là gì: đọc hết file này, bạn biết chính xác mỗi khẳng định trong `luu` đến từ đâu, được kiểm bằng cách nào, và chỗ nào còn là giả định đã biết chứ không phải sự thật đã chốt.

### Tinh thần nền, một câu

Mọi khẳng định đều bị kiểm lại; cộng đồng kiểm trước, official sau; chính việc kiểm cũng bị kiểm; khi bằng chứng lật một giả định thì nhận và sửa, không bảo vệ bản nháp cũ.

Đây không phải khẩu hiệu. Nó là lý do file TESTS.md có hai cột trụ từng tin chắc rồi bị lật ngược: hook ở mức exit 2 và dòng ra đầu phiên. Một quy trình lương thiện phải để lại dấu vết của những lần mình sai, không chỉ những lần mình đúng.

### Đây là quy trình thực nghiệm, không phải prompt phóng tay

Nói thẳng để khỏi hiểu lầm. `luu` v2.1 không ra đời từ một người ngồi viết một file rồi gọi nó là xong. Nó ra từ khoảng tám luồng làm việc nối nhau, hơn một trăm tác tử phụ chạy song song, và hàng triệu token. Đặc tính của một quy trình thực nghiệm là vòng lặp: thử, tìm chỗ hỏng, sửa, làm lại, rồi kiểm cả cái sửa. File này mô tả tám chặng của vòng lặp đó.

Một lưu ý về quy mô. Con số "hơn một trăm tác tử phụ" và "hàng triệu token" là mô tả công sức bỏ ra, không phải bằng chứng về chất lượng. Bằng chứng về chất lượng nằm ở TESTS.md, nơi mỗi con số đếm được. Tôi tách rạch hai thứ này để không bán quy mô như thể nó là sự đúng.

### Tám chặng dựng nên v2.1

#### Chặng 1, Radar GitHub

Quét rộng để biết bối cảnh thật, không dựa trí nhớ. Cào 1307 kho mã, rồi phân tầng xuống 332 kho đáng đọc kỹ. Mục tiêu chặng này không phải copy giải pháp của ai, mà là biết người ta đang vấp ở đâu khi cho tác tử tự ghi file, để `luu` không vá tưởng tượng.

#### Chặng 2, Hóa thân

Chưng cất cách hơn hai mươi chuyên gia harness Claude Code làm việc. Không phải hỏi một người rồi tin, mà dựng nhiều lăng kính chạy song song để mỗi khẳng định được soi từ nhiều phía cùng lúc. Đây là bước biến kiến thức rải rác thành góc nhìn dùng được.

#### Chặng 3, Blueprint

Audit chín chiều lên bản thiết kế. Soi chéo để bắt chính cái bệnh hay gặp: làm quá tay. Một skill ghi file mà thêm bảy lớp bảo vệ cho một máy một người dùng thì không an toàn hơn, chỉ phức tạp hơn. Chặng này hỏi với mỗi lớp: cái này đỡ một lỗi THẬT, hay chỉ làm tôi thấy yên tâm.

#### Chặng 4, Gia lập

Vỡ ra bảy mươi điểm có thể hỏng, rồi xác tín từng cái qua nguồn thật, cộng đồng trước. Sau sàng lọc, mười tám điểm được xác nhận là lỗ thật. Bảy mươi là số nghĩ ra; mười tám là số đứng được sau kiểm. Khoảng cách giữa hai con số chính là phần lương thiện của chặng này: phần lớn nỗi lo ban đầu không sống sót qua bằng chứng.

#### Chặng 5, Hội đồng

Hai mươi hai lăng kính chuyên gia soi toàn bộ skill, lặp vòng tới khi đồng thuận. Không dừng ở "nghe có vẻ ổn", mà chạy lại tới khi các lăng kính hết phản đối hoặc phản đối còn lại được ghi ra minh bạch. Kết quả chặng này thành Lớp 2 trong TESTS.md.

#### Chặng 6, Audit xác tín vòng hai

Kiểm lại chính cái việc kiểm. Đây là chặng dễ bị bỏ nhất và quan trọng nhất. Gọi GitHub API bằng curl để xác nhận mười một issue có thật, tiêu đề khớp, không phải nhớ mang máng. Quan trọng hơn: chặng này lật hai trụ từng tin sai. Một, hook ở mức exit 2 hóa ra đã sửa ở thượng nguồn, nên `luu` không cần vá nó. Hai, dòng ra đầu phiên không gây chết chương trình như từng lo. Hai lần lật này là bằng chứng quy trình tự sửa được, không chỉ tự khen.

#### Chặng 7, Dọn dẹp

Trả mỗi tool và skill về đúng bộ của nó. Một quy trình tốt không để lại rác: thứ mượn để thử thì trả lại, ranh giới rõ ràng, file sau đọc không phải đoán.

#### Chặng 8, Audit riêng cho `luu`

Soi riêng skill cuối cùng qua hai mươi hai lăng kính cộng sáu mô phỏng, rồi viết lại thành v2.1, rồi soi chéo lần nữa. Chặng này hợp lưu mọi thứ trước đó vào đúng một file skill, và sinh ra hai tài liệu kết quả: sáu kịch bản mô phỏng và bảng issue trong TESTS.md.

### Ba lớp kiểm, vì sao đủ

Quy trình này không dựa vào một loại bằng chứng. Nó chồng ba lớp khác bản chất lên nhau, để chỗ yếu của lớp này được lớp khác đỡ.

1. Sáu mô phỏng lỗi. Đây là lý lẽ: nghĩ trước từng cách hỏng, đặt một bản án.
2. Hai mươi hai lăng kính chuyên gia. Đây là phản biện: nhiều góc nhìn soi cùng một thiết kế tới đồng thuận.
3. Bốn issue cộng đồng kiểm qua API. Đây là thực địa: lỗi THẬT ngoài đời, xác nhận bằng curl, không phải `luu` tự nghĩ.

Lý lẽ một mình có thể tự huyễn. Phản biện một mình có thể đồng thuận sai. Thực địa một mình thiếu khung. Ba lớp cùng đứng thì khó sai cùng kiểu. Chi tiết kết quả và con số đếm được nằm trong TESTS.md.

### Giới hạn đã biết, ghi ra cho minh bạch

Quy mô tác tử và token đo công, không đo đúng. "Hơn hai mươi chuyên gia" là lăng kính chưng cất, không phải người thật ký tên. So mtime không phải khóa file tuyệt đối, chỉ đủ cho giả định một người ghi trên máy một người dùng. Những giới hạn này đã nằm trong TESTS.md và tôi không giấu chúng ở đây.

### Xong là gì

Đọc xong file này cộng TESTS.md, bạn có đủ để tự lặp lại quy trình, tự soi lại kết luận, hoặc tự bác nếu bằng chứng của bạn mạnh hơn. Đó là chuẩn của một tài liệu có thể trích dẫn: không xin bạn tin, mời bạn kiểm.

---
---

## ENGLISH

### Where this fits, what this file does

This file does not say what `luu` does. It says HOW `luu` was built. The distinction matters: a good prompt can be written in ten minutes, but a skill you would trust to overwrite your own session memory has to go through experiment. This file is the record of how that experiment ran, detailed enough for someone else to reproduce, audit, or refute.

What is asked of you: if you intend to cite `luu` as a technical artifact, this is its "methods" section. TESTS.md is the "results" section. Read the two together.

Done means: after this file, you know exactly where each claim in `luu` came from, how it was verified, and which parts are still known assumptions rather than settled fact.

### The underlying spirit, one line

Every claim is re-checked; community first, official second; the checking itself is checked; when evidence overturns an assumption, it is accepted and fixed, not defended.

This is not a slogan. It is why TESTS.md carries two pillars once believed and then overturned: the exit-2 hook and the session-start stdout. An honest process must leave traces of where it was wrong, not only where it was right.

### This is an empirical process, not a freehand prompt

Said plainly to avoid confusion. `luu` v2.1 was not produced by one person writing a file and calling it done. It came from roughly eight chained workflows, over one hundred parallel sub-agents, and millions of tokens. The signature of an empirical process is the loop: try, find what breaks, fix, redo, then check the fix too. This file describes the eight stages of that loop.

A note on scale. The figures "over one hundred sub-agents" and "millions of tokens" describe effort spent, not proof of quality. Proof of quality lives in TESTS.md, where every number is counted. I keep these separate so scale is not sold as if it were correctness.

### The eight stages that built v2.1

#### Stage 1, GitHub radar

Scan wide to learn the real landscape, not from memory. Crawled 1307 repositories, then tiered down to 332 worth a close read. The goal here was not to copy anyone's solution, but to learn where people actually trip when they let an agent write files, so `luu` patches reality rather than imagination.

#### Stage 2, Embodiment

Distill how more than twenty Claude Code harness experts work. Not asking one person and trusting them, but standing up multiple lenses in parallel so each claim is examined from several sides at once. This is the step that turns scattered knowledge into usable perspective.

#### Stage 3, Blueprint

A nine-dimension audit of the design. Cross-examined to catch the common disease: over-engineering. A file-writing skill that adds seven layers of protection for a single-user machine is not safer, only more complex. This stage asks of each layer: does this defend against a REAL failure, or does it only make me feel reassured.

#### Stage 4, Simulation

Broke out seventy points that could fail, then verified each against real sources, community first. After filtering, eighteen were confirmed as real holes. Seventy is what was imagined; eighteen is what survived checking. The gap between the two numbers is the honest part of this stage: most of the initial worry did not survive contact with evidence.

#### Stage 5, Council

Twenty-two expert lenses reviewed the whole skill, looping to consensus. Not stopping at "sounds fine", but rerunning until the lenses ran out of objections or the remaining objections were written down in the open. The result of this stage became Layer 2 in TESTS.md.

#### Stage 6, Verification audit, round two

Check the checking itself. This is the easiest stage to skip and the most important. Called the GitHub API with curl to confirm eleven issues are real, titles matched, not vaguely remembered. More importantly: this stage overturned two pillars once believed wrong. One, the exit-2 hook turned out to be fixed upstream, so `luu` need not patch it. Two, the session-start stdout does not crash the program as once feared. These two reversals are the evidence that the process can correct itself, not only congratulate itself.

#### Stage 7, Cleanup

Return each tool and skill to its proper set. A good process leaves no litter: what was borrowed for a test is given back, boundaries are clear, and the next reader does not have to guess.

#### Stage 8, The `luu`-specific audit

Review the final skill on its own through twenty-two lenses plus six simulations, rewrite it into v2.1, then cross-examine once more. This stage converges everything before it into a single skill file, and produces the two results documents: the six simulation scenarios and the issue table in TESTS.md.

### The three verification layers, why they suffice

This process does not rest on one kind of evidence. It stacks three layers of different nature so the weakness of one is covered by another.

1. Six failure simulations. This is reasoning: think through each failure mode in advance, set a verdict.
2. Twenty-two expert lenses. This is critique: many viewpoints on one design, to consensus.
3. Four community issues verified via the API. This is the field: REAL bugs in the wild, confirmed by curl, not invented by `luu`.

Reasoning alone can fool itself. Critique alone can reach false consensus. The field alone lacks a frame. The three standing together are hard to fail in the same way. Detailed results and counted totals live in TESTS.md.

### Known limits, stated for transparency

Agent and token scale measure effort, not correctness. "More than twenty experts" are distilled lenses, not real people who signed. The mtime check is not an absolute file lock, only enough for the single-writer assumption on a single-user machine. These limits already appear in TESTS.md and I do not hide them here.

### Done means

After this file plus TESTS.md, you have enough to reproduce the process, re-examine the conclusions, or refute them if your evidence is stronger. That is the standard of a citable artifact: it does not ask you to believe, it invites you to check.
