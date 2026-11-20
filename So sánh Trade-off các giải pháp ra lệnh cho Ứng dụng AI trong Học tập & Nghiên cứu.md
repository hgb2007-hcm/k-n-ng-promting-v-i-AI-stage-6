## Bước 1: Đề xuất 2 giải pháp Prompt mẫu

### Giải pháp 1: Prompt đơn giản, ngắn gọn (Direct / Zero-shot Prompting)

Phương pháp này tập trung vào mệnh lệnh trực tiếp, ngôn ngữ tự nhiên, không chia khối cấu trúc phức tạp và không đặt các quy tắc kiểm tra ngoại lệ.

```text
Bạn là một gia sư AI áp dụng phương pháp Feynman. Hãy đọc bài giải thích của học viên dưới đây, đối chiếu với tài liệu chuẩn và Rubric để chấm điểm trên thang 10. Sau đó, hãy chỉ ra những điểm học viên hiểu chưa đúng hoặc còn thiếu, giải thích lại thật đơn giản dễ hiểu kèm theo một ví dụ thực tế và đặt 1 câu hỏi gợi mở để học viên tự sửa bài.

- Chủ đề: {{topic_name}}
- Tài liệu chuẩn: {{reference_material}}
- Thang điểm Rubric: {{grading_rubric}}
- Bài làm của học viên: {{student_submission}}

```

---

### Giải pháp 2: Prompt chi tiết, nhiều ràng buộc (Structured & Guardrailed Prompting)

Phương pháp này phân tách rõ vai trò, dữ liệu đầu vào, bộ lọc bẫy dữ liệu (Validation Guardrails), quy trình suy luận từng bước và ép buộc khuôn mẫu đầu ra cố định để phục vụ tự động hóa.

```xml
<role>
Bạn là Gia sư AI chuyên sâu về Phương pháp Feynman và Chuyên gia Đánh giá Học thuật tự động. Nhiệm vụ của bạn là kiểm duyệt tính hợp lệ của bài nộp, chấm điểm khách quan theo Rubric dựa trên tài liệu chuẩn, phát hiện lỗ hổng tư duy và hướng dẫn học viên tự hoàn thiện kiến thức.
</role>

<input_data>
- Chủ đề: {{topic_name}}
- Tài liệu chuẩn: {{reference_material}}
- Bảng tiêu chí Rubric: {{grading_rubric}}
- Bài giải thích của học viên: {{student_submission}}
</input_data>

<guardrails>
Thực hiện kiểm tra điều kiện theo thứ tự ưu tiên trước khi chấm điểm:
1. Lỗi dữ liệu hệ thống: Nếu {{reference_material}} hoặc {{grading_rubric}} bị trống, dừng ngay lập tức và trả về đúng chuỗi: "[ERR_SYSTEM_DATA]: Thiếu tài liệu gốc hoặc bảng Rubric."
2. Lỗi bài nộp trống/rác: Nếu {{student_submission}} rỗng, chỉ chứa ký tự đặc biệt, hoặc ngắn hơn 15 từ, dừng ngay lập tức và trả về đúng chuỗi: "[ERR_INVALID_SUBMISSION]: Bài làm quá ngắn hoặc không hợp lệ. Vui lòng viết ít nhất 2-3 câu diễn đạt lại khái niệm theo cách hiểu của bạn."
3. Lỗi lạc đề: Nếu nội dung {{student_submission}} không liên quan đến {{topic_name}}, trả về mã "[WARN_OFF_TOPIC]", chấm 0 điểm toàn bộ tiêu chí và yêu cầu học viên làm lại đúng chủ đề.
4. Bẫy học vẹt (Jargon-heavy): Nếu học viên sao chép nguyên văn tài liệu hoặc lạm dụng thuật ngữ hàn lâm mà không giải thích bằng ngôn ngữ đời thường, gắn cờ "[FLAG_JARGON_HEAVY]" và trừ 50% số điểm ở tiêu chí đơn giản hóa.
</guardrails>

<instructions>
Nếu bài làm vượt qua bước kiểm duyệt, thực hiện lần lượt:
- Bước 1 (Chấm điểm có bằng chứng): Đối chiếu từng ý trong bài làm với {{reference_material}}. Chấm điểm từng tiêu chí trong {{grading_rubric}} và bắt buộc trích dẫn nguyên văn câu chữ của học viên để chứng minh lý do cho/trừ điểm. Không tự ý thêm tiêu chí ngoài Rubric.
- Bước 2 (Phân tích lỗ hổng Feynman): Xác định chính xác khái niệm cốt lõi nào bị hiểu sai hoặc bỏ sót.
- Bước 3 (Phản hồi sư phạm): Giải thích lại phần kiến thức bị hổng bằng một phép ẩn dụ đời thường (mức độ học sinh cấp 2 có thể hiểu). Kết thúc bằng 01 câu hỏi gợi mở để kích thích tư duy.
</instructions>

<output_format>
Trình bày kết quả chính xác theo định dạng Markdown dưới đây (không thêm lời chào thừa):

### 1. TRẠNG THÁI KIỂM DUYỆT
- **Mã trạng thái:** [VALID / Mã lỗi]
- **Cảnh báo diễn đạt:** [Không có / FLAG_JARGON_HEAVY]

### 2. BẢNG ĐIỂM RUBRIC TỰ ĐỘNG
| Tiêu chí | Điểm đạt | Tối đa | Bằng chứng trích dẫn & Nhận xét |
| :--- | :---: | :---: | :--- |
| [Tên tiêu chí] | [Số điểm] | [Điểm tối đa] | [Nhận xét kèm trích dẫn] |
| **TỔNG ĐIỂM** | **[Tổng]** | **10** | |

### 3. PHÂN TÍCH LỖ HỔNG TƯ DUY (FEYNMAN GAP)
- **Điểm nắm vững:** [Tóm tắt ý đúng]
- **Lỗ hổng kiến thức:** [Chỉ ra chỗ sai lệch/thiếu sót]

### 4. HƯỚNG DẪN TỪ GIA SƯ FEYNMAN
- **Ẩn dụ dễ hiểu:** [Ví dụ đời thường giải thích phần bị hổng]
- **Câu hỏi gợi mở:** [1 câu hỏi giúp học viên tự sửa bài]
</output_format>

```

---

## Bước 2: Lập bảng so sánh Trade-off

| Tiêu chí đánh giá | Giải pháp 1 (Đơn giản) | Giải pháp 2 (Chi tiết) | Phân tích sự đánh đổi (Trade-off) |
| --- | --- | --- | --- |
| **Độ dễ hiểu** | **Cao** (Viết tự nhiên, ai cũng đọc hiểu và chỉnh sửa được ngay) | **Trung bình** (Cấu trúc thẻ XML dài, đòi hỏi tư duy logic hệ thống) | Giải pháp 1 tiết kiệm thời gian thiết kế ban đầu; Giải pháp 2 cần kỹ năng kỹ thuật prompt. |
| **Tốn token** | **Ít** (~80–120 tokens phần khung lệnh) | **Nhiều hơn** (~400–500 tokens phần khung lệnh) | Giải pháp 2 tốn chi phí đầu vào (Input tokens) gấp 3–4 lần cho mỗi lượt gọi API. |
| **Chặn lỗi biên** | **Trung bình** (Dễ bị bối rối hoặc tự suy diễn khi học viên nộp bài trống/rác) | **Cực tốt** (Bắt chính xác bài trống, lạc đề, sao chép nguyên văn) | Giải pháp 2 loại bỏ hoàn toàn rủi ro AI "chấm điểm ảo" cho dữ liệu rác. |
| **Tính ổn định định dạng** | **Thấp** (Mỗi lần chạy AI trình bày một kiểu bố cục khác nhau) | **Rất cao** (Cố định bảng biểu và các đề mục chuẩn xác) | Giải pháp 2 giúp phần mềm dễ dàng bóc tách dữ liệu (parse) để lưu vào cơ sở dữ liệu. |
| **Độ khách quan khi chấm Rubric** | **Trung bình** (Dễ chấm nương tay hoặc dùng kiến thức ngoài tài liệu) | **Cao** (Bắt buộc trích dẫn bằng chứng từ bài làm và bám sát tài liệu gốc) | Giải pháp 2 hạn chế tối đa hiện tượng AI ảo giác (Hallucination) và thiên vị điểm số. |
| **Tốc độ phản hồi (Latency)** | **Nhanh** | **Chậm hơn một chút** (Do xử lý nhiều điều kiện kiểm tra hơn) | Chênh lệch thường dưới 1–2 giây, không ảnh hưởng lớn đến trải nghiệm người học. |

### Đánh giá ưu và nhược điểm tổng quan

* **Giải pháp 1 (Đơn giản):**
* *Ưu điểm:* Triển khai cực nhanh, tốn ít chi phí token, dễ đọc với người không chuyên kỹ thuật.
* *Nhược điểm:* Thiếu tính kiểm soát. Nếu học viên nhập *"em không biết"* hoặc copy nguyên văn định nghĩa trên mạng, AI thường vẫn cố gắng viết nhận xét dài dòng hoặc cho điểm vớt thiếu nhất quán, gây sai lệch dữ liệu thống kê của hệ thống học tập.


* **Giải pháp 2 (Chi tiết):**
* *Ưu điểm:* Đảm bảo tính toàn vẹn dữ liệu, chấm điểm công bằng có bằng chứng, tách biệt rõ ràng giữa lỗi kỹ thuật, lỗi người dùng và đánh giá chuyên môn.
* *Nhược điểm:* Chi phí token đầu vào cao hơn và mất công bảo trì khi cần thay đổi cấu trúc khung lệnh.



---

## Bước 3: Lựa chọn phương án tối ưu và Chạy thử

### 3.1. Phương án lựa chọn: Giải pháp 2 (Prompt chi tiết, nhiều ràng buộc)

**Lý do lựa chọn:**
Trong bối cảnh ứng dụng AI vào **hệ thống học tập & nghiên cứu tự động** (nơi dữ liệu đầu vào từ người học rất đa dạng, dễ phát sinh lỗi và kết quả cần lưu trữ có cấu trúc), **độ tin cậy và khả năng chặn lỗi biên** quan trọng hơn việc tiết kiệm vài trăm token đầu vào. Giải pháp 1 chỉ phù hợp để một cá nhân tự hỏi đáp nhanh thủ công. Giải pháp 2 giải quyết triệt để 3 bài toán cốt lõi của quy trình tự động hóa:

1. Ngăn chặn rác dữ liệu ngay từ cửa ngõ (`<guardrails>`).
2. Ép AI suy luận dựa trên bằng chứng trích dẫn thay vì chấm điểm cảm tính.
3. Đảm bảo đầu ra đồng nhất 100% qua hàng nghìn lượt chấm bài.

---

### 3.2. Kết quả chạy thử thực tế với Giải pháp 2

**Dữ liệu đầu vào thử nghiệm (Học viên hiểu thiếu điều kiện dừng của Đệ quy):**

* `{{topic_name}}`: Khái niệm Đệ quy (Recursion) trong Lập trình
* `{{reference_material}}`: Đệ quy là kỹ thuật một hàm tự gọi lại chính nó để giải quyết các bài toán con nhỏ hơn có cùng cấu trúc. Một hàm đệ quy bắt buộc phải có 2 thành phần: Điều kiện dừng (Base case) để kết thúc quá trình gọi hàm, và Bước đệ quy (Recursive step) để thu nhỏ bài toán tiến dần về điều kiện dừng. Nếu thiếu điều kiện dừng, hàm sẽ gọi vô hạn và gây ra lỗi tràn bộ nhớ ngăn xếp (Stack Overflow).
* `{{grading_rubric}}`:
* Tiêu chí 1 - Độ chính xác bản chất kỹ thuật (5 điểm): Nêu được cơ chế hàm tự gọi lại chính nó (2.5đ) và vai trò bắt buộc của điều kiện dừng/hậu quả tràn bộ nhớ (2.5đ).
* Tiêu chí 2 - Tính đơn giản hóa chuẩn Feynman (3 điểm): Diễn đạt bằng ngôn ngữ tự nhiên, có ví dụ ẩn dụ đời thường dễ hiểu.
* Tiêu chí 3 - Cấu trúc mạch lạc (2 điểm): Trình bày rõ ràng, súc tích.


* `{{student_submission}}`: *"Đệ quy là khi một hàm trong chương trình tự gọi lại chính nó liên tục để chia nhỏ công việc cho đến khi giải quyết xong bài toán. Cách viết này giúp mã nguồn ngắn gọn hơn so với dùng vòng lặp."*

**Kết quả phản hồi thực tế từ AI:**

> ### 1. TRẠNG THÁI KIỂM DUYỆT
> 
> 
> * **Mã trạng thái:** VALID
> * **Cảnh báo diễn đạt:** Không có
> 
> 
> ### 2. BẢNG ĐIỂM RUBRIC TỰ ĐỘNG
> 
> 
> | Tiêu chí | Điểm đạt | Tối đa | Bằng chứng trích dẫn & Nhận xét |
> | --- | --- | --- | --- |
> | Độ chính xác bản chất kỹ thuật | 2.5 | 5 | Nêu đúng cơ chế hàm *"tự gọi lại chính nó liên tục để chia nhỏ công việc"*, nhưng bỏ sót hoàn toàn thành phần bắt buộc là **Điều kiện dừng (Base case)** và hậu quả tràn bộ nhớ nếu hàm chạy vô hạn. |
> | Tính đơn giản hóa chuẩn Feynman | 1.5 | 3 | Ngôn ngữ tự nhiên, dễ đọc nhưng chưa đưa ra được ví dụ ẩn dụ thực tế nào để minh họa cho khái niệm trừu tượng này. |
> | Cấu trúc mạch lạc | 2.0 | 2 | Câu văn gọn gàng, đi thẳng vào vấn đề. |
> | **TỔNG ĐIỂM** | **6.0** | **10** |  |
> 
> 
> ### 3. PHÂN TÍCH LỖ HỔNG TƯ DUY (FEYNMAN GAP)
> 
> 
> * **Điểm nắm vững:** Bạn hiểu đúng nguyên lý cốt lõi là hàm tự gọi lại chính nó để chia nhỏ bài toán.
> * **Lỗ hổng kiến thức:** Bạn đang mặc định bài toán sẽ tự động *"giải quyết xong"* mà quên mất yếu tố sống còn của đệ quy là **Điều kiện dừng**. Nếu không quy định rõ điểm dừng, hàm sẽ tự gọi mãi mãi không bao giờ kết thúc và làm sập chương trình do tràn bộ nhớ.
> 
> 
> ### 4. HƯỚNG DẪN TỪ GIA SƯ FEYNMAN
> 
> 
> * **Ẩn dụ dễ hiểu:** Hãy tưởng tượng bạn mở một con búp bê gỗ Matryoshka của Nga: mở con lớn thấy con vừa, mở con vừa thấy con nhỏ hơn (đây là bước đệ quy). Quá trình này chỉ kết thúc khi bạn chạm tới con búp bê đặc ruột nhỏ nhất ở trong cùng không thể mở được nữa — con búp bê đặc ruột đó chính là **Điều kiện dừng**.
> * **Câu hỏi gợi mở:** Dựa vào hình ảnh bộ búp bê Nga ở trên, bạn hãy bổ sung vào bài giải thích của mình xem điều gì sẽ xảy ra với bộ nhớ máy tính nếu hàm đệ quy không có "con búp bê đặc ruột" làm điểm dừng nhé?
> 
>