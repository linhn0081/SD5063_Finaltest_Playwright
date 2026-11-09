# AI-assisted Application Security Testing for Tester — Tổng hợp nội dung Training

> **Nguồn:** transcript buổi training (phần 1 + phần 2) · **Trainer:** Hien Trinh Minh · Company Training (Adhoc)
>
> **Lưu ý khi đọc:** Transcript được auto-generate nên nhiều thuật ngữ bị sai chính tả. Tài liệu này đã chuẩn hoá lại: "ô áp" = **OWASP**, "ESPS/ASPS" = **ASVS**, "MTP/MTV server" = **MCP Server**, "intech" = **intake**, "eviden" = **evidence**, "fidin" = **finding**, "serverity" = **severity**.
>
> 📄 Bản Word đầy đủ (có bảng + box highlight màu): [`AI-assisted-Security-Testing-Training-Summary.docx`](./AI-assisted-Security-Testing-Training-Summary.docx)

---

## 1. Mục tiêu & phạm vi khoá học

- Chủ đề: **dùng AI (GitHub Copilot Agent) hỗ trợ Security Testing cho ứng dụng Web**.
- Đối tượng chính là **Tester**, nhưng mở public cho cả Dev / PM trong công ty.
- Đây là khoá **cơ bản**, không chuyên sâu: chỉ làm **OWASP ASVS Level 1** — chạy được full checklist Level 1 và biết viết report ở mức căn bản (chưa phải report pentest chuyên nghiệp).

**Giá trị nhận được sau khoá học:**

- Biết dùng tool/agent (Copilot Agent) để chạy security test.
- Cải thiện giao tiếp với team Dev khi raise bug bảo mật.
- Tăng giá trị profile: security đang được gộp chung vào testing → tester chỉ biết manual/automation sẽ bị **giới hạn**.

> **Lưu ý nghề nghiệp:** Bug security **ít nhưng nặng** — khác hẳn bug manual/automation. Một app có thể chỉ mở ra 1–3 bug, nhưng client rất quan tâm vì nó đánh giá ứng dụng **mạnh hay yếu và có đủ khả năng chạy trên Production hay không**.

---

## 2. Điều kiện bắt buộc (phải tự check TRƯỚC buổi practice)

| Nhóm | Nội dung cần chuẩn bị |
|---|---|
| Account | GitHub Copilot license (team Training đã request sẵn, ~$19 subscription cho người tham gia) |
| Local tools | VS Code, Git, Node.js, Python… (4 tool theo tài liệu) — tự kiểm tra version bằng command trong doc |
| Workspace | **Clone repo** mà trainer cung cấp (đã có sẵn skill, template, MCP config) |
| MCP Server | Chứa bộ checklist OWASP ASVS — cần Node để chạy |

- Về version: **không có "mốc trần"** — bản nào cũng chạy. Nhưng version quá thấp sẽ dính lỗ hổng bảo mật do IT scan ra và bắt update. Mức sàn tham khảo: **Node 18+, npm 9+**.
- Permission / lỗi cài đặt → **raise sớm nhất có thể** trên channel, đừng đợi tới buổi practice. Trainer nhấn mạnh: máy của trainer chạy trơn tru vì đã qua debug rồi, máy mỗi người mỗi khác.

---

## 3. Kiến trúc: 3 thành phần chính

*(Đây là ý được trainer nhắc đi nhắc lại nhiều lần trong buổi học)*

```
Tester   →   Copilot Agent   →   MCP Server (OWASP ASVS checklist)
```

1. **Tester (bạn):** cung cấp target/scope, authorize, chọn loại test, khai báo tiêu chuẩn, cung cấp account (nếu Grey Box), định nghĩa nơi xuất report, và **review & quyết định Yes/No**.
2. **Copilot Agent:** đọc file intake → lập plan → tự gửi request, bắt response, so sánh → map kết quả Pass / Fail / Not Applicable → viết finding → ghi report theo template.
3. **MCP Server:** thực chất chỉ là **một server + một file JSON liệt kê toàn bộ test case ASVS Level 1** — "không có gì ghê gớm". Chỉ cần clone repo về là đã có.

---

## 4. Hai file quan trọng nhất

### 4.1. `security-test-intake.md` — file INPUT

**Đây là file duy nhất bạn làm việc trên đó.** Nó vừa là input, vừa là công cụ "lái" AI chạy. **Sửa trực tiếp trong file, không sửa rời rạc ở chỗ khác.**

| # | Mục khai báo | Ghi chú quan trọng |
|---|---|---|
| 0 | **Authorization statement** | Câu "tuyên bố cho phép", để AI chịu chạy. AI sẽ từ chối tấn công target mà nó không thấy bằng chứng được phép |
| 1 | **Target** (URL / domain / IP) | **BẮT BUỘC.** Không có target thì không ai test được, kể cả security tester thật |
| 2 | In-scope / Out-of-scope | Khai báo subdomain, link API. Out-of-scope ví dụ: Identity Server dùng chung, hệ thống ngân hàng → không cho AI đụng vào |
| 3 | **Testing type** | Black Box / Grey Box / White Box |
| 4 | Environment | local / testing / staging |
| 5 | **Standard + Level** | OWASP ASVS Level 1 |
| 6 | **Test accounts** | Bắt buộc với Grey Box. Nên khai ≥ 2 account khác role → AI test luôn **role-based / privilege escalation** (user A ↔ user B ↔ admin) |
| 7 | Rate limit | Ví dụ set **5 request/giây** để không làm sập hệ thống |
| 8 | Output path | Folder chứa file report |
| 9 | Escalation / Pre-scan | Gặp file độc hại thì dừng, đừng bypass — dễ bị **block account** |

> ### ⚠️ Lưu ý pháp lý (ý rất quan trọng)
>
> Gửi một chuỗi SQL Injection tới target chưa được phép = **bạn đang hack hệ thống của họ**. Ở Việt Nam có luật xử lý việc này và **không thể biện hộ "tôi vô tình"** (vô tình thì không thể gửi được một chuỗi SQL Injection). Ngoài thực tế, công ty pentest luôn phải có **NDA / giấy uỷ quyền ký trước**. AI cũng tuân theo nguyên tắc đó — vì vậy mới cần mục Authorization.

**Hai bẫy khi khai báo mục Authorization:**

- Mục **Engagement / Pen test signed**: nếu tick "đã ký" thì khi chạy **AI sẽ hỏi xin giấy tờ** rồi mới chịu chạy → tốt nhất khai là **internal**, đừng tick.
- Mục **Bug Bounty**: khai **No**. Nếu khai Yes, AI sẽ hỏi lòng vòng về chương trình bounty.

> ### 💡 MẸO HAY NHẤT BUỔI HỌC
>
> Nếu bạn chỉ biết điền mục 1 (Target) → **đưa target cho AI và prompt bắt AI tự explore app rồi fill ngược lại toàn bộ file intake**.
>
> Ví dụ prompt: *"Hãy nghiên cứu ứng dụng tại `<URL>` này, rồi điền ngược lại đầy đủ cho file security-test-intake.md"*
>
> AI sẽ tự dò công nghệ, liệt kê scope, và **hỏi lại bạn** những chỗ cần chọn ("bạn muốn test Black Box hay Grey Box?"). Không biết Black Box / Grey Box là gì → hỏi tiếp AI. Sau đó bạn chỉ việc review lại.

### 4.2. `security-test-report-template.md` — file TEMPLATE report

- Đã có sẵn trong repo, đúc kết từ kinh nghiệm test dự án thật của trainer.
- Chỉ là file `.md` hướng dẫn, **không phải code** → bạn tự sửa được, hoặc nhờ AI sửa giúp.
- AI sẽ tự điền các biến từ file intake vào template này khi viết report.

---

## 5. Luồng end-to-end

1. Bạn điền `security-test-intake.md` (hoặc nhờ AI điền giúp).
2. **Dry-run / "chạy khô"**: prompt kêu AI check lại file input đã đúng chưa, **trước khi scan thật**.
3. Prompt kêu agent đọc file intake và chạy — có thể chạy **full** hoặc chỉ **1 test case / 1 item**.
4. Agent lập plan → gửi request → bắt response → so với testing guide → chấm **Pass / Fail / Not Applicable**.
5. Agent **hỏi lại bạn** khi cần permission hoặc khi thấy vượt rào → bạn quyết định Yes/No.
6. **Memory**: agent lưu lại cấu hình, lịch sử đã chạy, session/ngày chạy → lần sau chạy lại nó biết đã chạy gì rồi.
7. Mỗi Fail → mở **1 finding kèm evidence**.
8. Trước khi viết report, agent gọi **MCP Server** để map item ↔ OWASP ASVS.
9. Report xuất ra file theo template. Muốn chạy lại / re-test item nào thì khai trong intake hoặc prompt → nó **update vào report hiện tại**.

---

## 6. Nguyên tắc vàng của Security Testing

> ### 🥇 NGUYÊN TẮC 1 — Không có Evidence thì không phải là Bug
>
> *"No evidence → Not a finding"*. Evidence = screenshot, request, response, chỉ rõ chính xác chỗ phát hiện lỗi.
>
> - Bug phải **reproduce được**. Không reproduce được thì dev sẽ nói "tôi không review được, không fix".
> - Security report **chỉ report case FAIL**, không viết case PASS (quá nhiều case, tốn token; commitment với khách hàng cũng chỉ cam kết cung cấp fail case).

> ### 🥇 NGUYÊN TẮC 2 — AI execute, Con người chịu trách nhiệm
>
> Agent chạy và tự review, nhưng **bạn là người debug, verify và chịu trách nhiệm cuối cùng** cho kết quả. Không được đổ thừa "AI chạy ra vậy".
>
> - Cách verify nhanh: copy URL/payload trong finding, paste lên browser xem có reproduce được không.

> ### 🥇 NGUYÊN TẮC 3 — Test theo TIÊU CHUẨN, không theo ý kiến cá nhân hay business của khách hàng
>
> Ví dụ thảo luận ngay trong buổi: khách hàng nói *"tôi cho phép password bất kỳ"*, AI vẫn mở bug *"password policy quá yếu (5 ký tự, không yêu cầu hoa/thường/ký tự đặc biệt)"*.
>
> → **Bug vẫn đúng, vẫn ở trạng thái Fail**, vì test căn theo OWASP ASVS. Việc fix hay không, accept hay không là quyết định của Dev/khách hàng (họ accept vào list), nhưng **bug vẫn phải được mở**.

---

## 7. OWASP ASVS — nền tảng tiêu chuẩn

- OWASP = tiêu chuẩn cộng đồng; ASVS = **Application Security Verification Standard**; version hiện tại **5.0**.

| Level | Áp dụng cho | Số item |
|---|---|---|
| **L1** | Mức tối thiểu — **mọi app đều phải đạt** | vài chục (~70 mấy) |
| **L2** | App có sensitive data | — |
| **L3** | Finance, Banking, Healthcare, Insurance | Tổng cả 3 level ~300+ |

- Cấu trúc: **V1 → V12** (gần đây có thêm chương về AI, không nằm trong scope khoá này). Trainer có chia nhỏ list thành 16/18/19 nhóm trong file config.
- Mỗi item có **Testing Guide** hướng dẫn chi tiết cách test. Ví dụ **V3.4.1 – Strict-Transport-Security header**: test header gì, timeout, parameter nào cần có.
- Khi report finding chuyên nghiệp **luôn phải ghi rõ: thuộc OWASP ASVS item nào, Level mấy, ID bao nhiêu**.
- Định hướng dài hạn: giai đoạn đầu bạn là người "chạy AI Agent" (AI-assisted). Muốn lấn sân sang security tester thật thì **phải tự học từng item, tự hiểu kỹ thuật test**.

---

## 8. Phân loại khả năng test (3 loại)

| Loại | Mô tả | AI test được? |
|---|---|---|
| **1** | Gửi request → nhìn response là xác định Pass/Fail ngay | ✅ Được |
| **2** | Cần qua nhiều bước trung gian mới kết luận được. VD: test **phân quyền** — phải đăng ký user, login user A, login user B, thử bypass / escalate từ A sang B | ✅ Được, nhưng phải qua "một cổng nữa" |
| **3** | Các item phải kiểm tra qua **documentation** (policy, tài liệu dự án, quy trình) | ❌ **KHÔNG test được** |

> **Giải thích thêm từ phần Q&A:** Scope của Copilot Agent này chỉ đơn giản là **gửi request / bắt response**. Nó **không biết tài liệu dự án**, cũng không có cơ sở tiêu chuẩn nào để đánh giá document của client. Đây là **giới hạn của scope agent**, không phải AI không làm được — trainer có một agent riêng khác cho việc đó, nhưng không nằm trong khoá này.

---

## 9. So sánh: Tester thủ công vs Copilot Agent (ví dụ test Cookie)

| Tiêu chí | Tester truyền thống | Copilot Agent |
|---|---|---|
| Công cụ | Browser + **proxy tool** (phải cấu hình browser đi qua proxy) | Tự tạo request bằng giao thức riêng, **không cần proxy tool** |
| Việc phải làm | Phải **hiểu item**, biết dùng tool gì, cấu hình browser ra sao, soi từng request/response | Tự đọc scope + level trong intake, tự map item (vd V3.3.1), tự gửi & check response theo testing guide |
| Vai trò của bạn | Làm toàn bộ | **Chỉ review output** |

---

## 10. Giới hạn của AI & cách khắc phục (rất thực tế)

- **Grey Box mà không cung cấp account (hoặc account sai)** → AI chỉ chạy như Black Box, nhìn từ ngoài, **không vào được bên trong app** → scan bao nhiêu lần cũng không ra bug. Phải xác nhận có "tín hiệu AI đã login được vào app".
- **App có nhiều business logic phức tạp** → để AI tự explore sẽ bị sót. Bạn phải **chỉ định cụ thể từng menu / chức năng** cho nó.
- Ví dụ luồng workflow: user tạo ticket → cần manager/admin approve → **AI không có cửa đi tiếp**. Bạn phải giả lập / tạo sẵn dữ liệu, request đó giúp nó.
- Scan ra ít bug / thiếu bug quen thuộc → prompt chỉ định: *"sao chưa thấy lỗi X?"* → nó sẽ chạy lại và đào sâu đúng chỗ đó.
- Không explore đủ app → chắc chắn coverage thiếu, **dù checklist đã đầy đủ**.

---

## 11. Cảnh báo an toàn khi bấm Allow / Approve

> ### 🚨 ĐỪNG luôn luôn bấm Yes/Allow, và ĐỪNG bật auto-approve
>
> - **Ví dụ 1:** agent hỏi xin **upload file** → upload file có thể là **malware** ảnh hưởng hệ thống.
> - **Ví dụ 2:** agent hỏi tải / thực thi **script từ link bên ngoài** → "rước hoạ vào nhà".
> - Antivirus công ty có thể bắt được virus thường, **nhưng code inject trực tiếp, mở port, start service lạ thì chưa chắc phát hiện**.
> - Nghi ngờ gì → **dừng lại**. Trách nhiệm cuối cùng vẫn là của bạn, không đổ thừa được cho AI.

---

## 12. Ví dụ finding thật được demo trong buổi (Path Traversal / LFI)

**Tình huống:** app được host trên **Linux**, trên URL có một tham số dùng để load page (ví dụ `?page=index`). Tham số này chính là điểm inject.

```
Bình thường:  https://<target>/?page=index
Khai thác:    https://<target>/?page=../../../../etc/passwd
```

- AI tự phát hiện điểm inject, thử rất nhiều payload, và khai thác thành công bằng cách truyền path kiểu `../../../../etc/passwd`.
- Lỗi này gọi là **Path Traversal / Local File Inclusion (LFI)**.

**Vì sao rất nặng** — AI chỉ lấy 1 evidence (file `/etc/passwd`), nhưng viễn cảnh thực tế là:

- Đọc được **bất kỳ file nào trên server**: file cấu hình, source code, **log file**.
- `/etc/passwd` chứa password đã hash → có thể decrypt ra.
- Log file của ứng dụng kiểu cũ có khi log cả **username / password** cho cả case login pass lẫn fail.
- Hacker chỉ cần đọc được 1 file là sẽ tìm cách **lấy hết toàn bộ file** của hệ thống.
- *(Ghi chú thêm: app hiện đại mà log chứa **sensitive data** cũng tự động bị Fail một test case khác.)*

---

## 13. Cấu trúc một Finding

### 13.1. Report chuyên nghiệp (mức pentester thật)

1. **Title**
2. **Severity** (Critical / High / Medium / Low) — chấm điểm theo **CVSS Score**
3. Mapping: OWASP ASVS item + ID, OWASP Testing Guide
4. **Description** (mô tả) → sau đó mới nêu vấn đề
5. **Impact** — rất quan trọng: *bug không có impact thì severity rất thấp*
6. **Steps to Reproduce**
7. **Remediation / Recommendation** (gợi ý cho dev fix)
8. **References**

### 13.2. Mức của khoá này (đã lược bớt)

Target / Scope + Summary + thống kê Pass/Fail + Findings gồm: **ASVS Level 1 item → Description → Steps to reproduce → Impact → Recommendation**.

→ **Bỏ phần Severity / CVSS** vì đòi hỏi hiểu sâu hơn.

> **Góc nhìn của Dev:** Trainer nhấn mạnh: dev thực ra chỉ cần đọc **Steps to reproduce + evidence** (API nào, input gì, output gì) là đã fix được. Phần Impact / Recommendation là để diễn giải thêm. Recommendation do AI generate thường khá **chung chung**; report chuyên nghiệp sẽ chỉ định cụ thể hơn.

---

## 14. Agent "Reviewer" (bonus, ngoài scope khoá học)

- Trainer có viết thêm **1 agent tên "Reviewer"**, sẽ chia sẻ qua document (không training).
- Cách dùng: **đưa cho nó cặp Request + Response** (viết prompt bằng tiếng Việt hay tiếng Anh đều được).
- Nó trả về **một finding đầy đủ, chuẩn pentester**: severity, impact, recommendation, reference → bạn chỉ việc copy vào report.
- Dùng khi: bạn không chắc đó có phải bug không, hoặc không biết viết finding sao cho chuyên nghiệp.

---

## 15. Practice & Assessment

- **Practice:** **không chấm điểm**, không cần nộp. Chỉ hỏi khi làm không được.
- **Assessment:** sau practice, trainer đưa **một site khác + URL target**; bạn tự chọn Black Box hay Grey Box; chạy sao cho **tìm ra hết các finding mà trainer đã scan sẵn để chấm**; nộp **file report** qua link được share.
- Chạy lại / re-test cho tới khi hài lòng với report rồi mới nộp. Nên thao tác trên **cùng file intake** (nó record session/ngày chạy), đừng tách rời.
- Repo có sẵn **file chứa các prompt mẫu**: prompt validate file intake, prompt chạy full, prompt chạy 1 test case/item, prompt re-test. Nhập tiếng Việt nó cũng tự hiểu.

---

## ✅ CHECKLIST — 10 điều cần nhớ nhất

| # | Điều cần nhớ |
|---|---|
| 1 | Scope khoá = **OWASP ASVS Level 1**, web app, AI-assisted. |
| 2 | 3 thành phần: **Tester → Copilot Agent → MCP Server (checklist ASVS)**. |
| 3 | Chỉ làm việc trên **1 file: `security-test-intake.md`**. |
| 4 | **Target là bắt buộc**; Grey Box **bắt buộc có account** (nên ≥ 2 role khác nhau). |
| 5 | Phải có **Authorization** — test khi chưa được phép = **phạm pháp**. |
| 6 | Không biết điền intake → **bắt AI tự explore target và fill ngược lại**. |
| 7 | **Chạy khô (validate input) trước**, rồi mới scan thật. |
| 8 | **No Evidence = No Bug**; chỉ report Fail case; phải **reproduce được**. |
| 9 | Chấm theo **tiêu chuẩn**, không theo business của khách hàng. |
| 10 | **Đừng auto-Allow** mọi thứ; **bạn chịu trách nhiệm cuối cùng**, không đổ thừa AI. |

---

*— Hết —*
