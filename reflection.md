
Báo Cáo Cá Nhân — Trong Minh
Dự án: V-Triage AI — Trợ lý phân tuyến chuyên khoa Vinmec
Nhóm: 11 | Ngày: 09/04/2026

1. Role cụ thể trong nhóm
AI Agent & Prompt Engineering Lead
Tôi chịu trách nhiệm xây dựng "bộ não" cho AI agent — tức là thiết kế logic suy luận (reasoning), cấu trúc prompt, decision trees và JSON schema để xác định luồng phân tuyến. Công việc chính là chuyển các yêu cầu nghiệp vụ (phân tuyến bệnh nhân chính xác) thành một hệ thống prompt-based mạnh mẽ, với khả năng self-correct và learn từ feedback.

2. Phần phụ trách cụ thể (output rõ ràng)
a) Xây dựng Prompt Architecture cho 4 luồng (Paths):
Thiết kế system prompt và chain-of-thought prompting để AI xác định:

Happy Path: Triệu chứng rõ ràng → phân tuyến ngay lập tức (confidence > 80%)
Uncertain Path: Thông tin chưa đủ → tự động sinh danh sách câu hỏi mà Frontend render thành form (multiselect)
Emergency Path: Dấu hiệu nguy hiểm → cảnh báo và gợi ý gọi 115
Multiple Departments: Các triệu chứng phức hợp → ranking nhiều khoa có khả năng cao

b) Xây dựng JSON Output Schema:
Định nghĩa structure JSON mà Frontend có thể render:
json{
  "path": "happy|uncertain|emergency",
  "confidence": 0.92,
  "department_primary": "Khoa Nội Tiêu hóa",
  "reasoning": "Đau bụng vùng quanh rốn + buồn nôn → nghi ngờ viêm ruột thừa",
  "follow_up_questions": ["Đau bao lâu?", "Có sốt không?"],
  "symptoms_extracted": ["đau bụng", "buồn nôn"],
  "risk_flags": []
}
c) Agent Decision Logic & Routing:
Xây dựng heuristic để agent quyết định:

Có đủ thông tin để phân tuyến không?
Nếu chưa, hỏi cái gì tiếp theo (ưu tiên theo tầm quan trọng)?
Các triệu chứng có xung đột không (vd: cơn đau đột ngột + ngang ngửa)?
Cần escalate lên emergency hay không?

d) Eval Metrics cho AI Output:
Xác định các chỉ số đánh giá:

Accuracy: % phân tuyến đúng so với chẩn đoán thực tế từ bác sĩ
Confidence calibration: Khi AI nói 90%, có đúng 90% cases là correct không?
Completeness: % cases mà AI tự tin phân tuyến vs. cần hỏi thêm
Recall (Safety): Có bỏ sót các dấu hiệu nguy hiểm không?


3. SPEC phần nào mạnh nhất, yếu nhất?
Mạnh nhất — Uncertain Path Logic (Phân luồng thông tin thiếu):
Hệ thống tự động sinh danh sách câu hỏi follow-up thông minh thay vì bắt người bệnh gõ text. Ví dụ:

User nói: "Tôi bị đau đầu"
Agent tự sinh: ["Đau bao lâu?", "Có sốt?", "Có buồn nôn?", "Có nhạy cảm với ánh sáng?"]
Frontend render thành checkbox → user tick nhanh → agent update confidence score
Luồng này giảm friction tương tác trong khi tối ưu hoá việc thu thập thông tin.

Yếu nhất — Eval Metrics & ROI phần AI:
Các con số accuracy hiện tại dựa trên kiểm thử thủ công trên 20-30 test cases. Chúng ta chưa có dữ liệu production thực tế từ Vinmec để validate:

Có bao nhiêu % AI phân tuyến sai so với bác sĩ?
Confidence calibration có tốt không (khi AI nói 85% confident, reality là bao nhiêu)?
Có bị miss bao nhiêu emergency cases (false negatives)?


4. Đóng góp cụ thể khác (test, debug, support)
Xử lý Edge Case "Triệu chứng xung đột":
Khi user nói vừa có triệu chứng của COVID (ho, sốt) vừa đau bụng dữ dội (có thể chuyên khoa khác), tôi thiết kế logic để AI:

Phát hiện xung đột
Tạo hypothesis từng khoa riêng
Rank theo severity (ưu tiên triệu chứng nguy hiểm trước)
Gợi ý "Có thể cần khám nhiều khoa, bắt đầu từ Khoa A trước"

Debug Chain-of-Thought (CoT) Output:
Khi AI trả về quyết định phân tuyến sai, tôi có thể trace được logic nó suy luận thế nào bằng cách yêu cầu nó output "reasoning" field. Điều này giúp:

Tìm được bug ở tầng prompt (vd: hệ thống prompt thiếu context)
Không phải vò đầu bứt tóc debug black box
Có thể iterate prompt nhanh

Xử lý Hallucination từ OpenAI:
Bọc toàn bộ lệnh gọi GPT-4o-mini trong validation logic:

Kiểm tra JSON có valid không
Verify confidence score nằm trong [0, 1]
Verify department được phân tuyến có trong whitelist Vinmec không
Nếu lỗi, retry với refined prompt hoặc fallback safety


5. Một điều học được trong hackathon mà trước đó chưa biết
Chain-of-Thought prompting vs. Direct prompting — và khi nào nên dùng cái nào.
Ban đầu, tôi viết prompt đơn giản:
Người bệnh nói: {user_input}
Hãy phân tuyến đến khoa nào? JSON: {...}
Kết quả: AI trả lời nhanh nhưng hay "ảo giác" — gợi ý khoa chuyên không phù hợp hoặc miss dấu hiệu nguy hiểm.
Sau đó, tôi thêm chain-of-thought:
Hãy phân tích từng bước:
1. Triệu chứng chính là gì?
2. Các triệu chứng phụ?
3. Có dấu hiệu nguy hiểm không?
4. Đâu là khoa chuyên phù hợp nhất?
JSON: {...}
Kết quả: Accuracy tăng từ ~75% lên 88% vì AI "suy luận" thay vì "guessing". Nhưng latency cũng tăng (~2s thay vì 0.5s).
Lesson: CoT tốt cho accuracy nhưng chậm. Cần trade-off dựa trên use case (y tế = accuracy quan trọng > latency).

6. Nếu làm lại, đổi gì?
Nếu làm lại dự án này ở quy mô Production, tôi sẽ:
1. Structured Few-Shot Prompting từ đầu:
Thay vì generic prompt, tôi sẽ xây dựng 5-10 exemplars (examples) thực tế từ y bác sĩ:
Example 1:
Input: "Đau bụng quanh rốn, buồn nôn"
Reasoning: [Chi tiết suy luận]
Output: {"path": "happy", "department": "Nội tiêu hóa", "confidence": 0.92}

Example 2:
...
Few-shot này làm AI "hiểu ngữ cảnh y tế" hơn là prompt chung chung.
2. Separate Logic cho Triage vs. Retrieval:
Hiện tại tôi dùng 1 prompt cho tất cả. Nếu làm lại:

Phase 1 (Triage): GPT-4o mini phân tuyến nhanh
Phase 2 (Enrichment): Nếu uncertain, dùng RAG để lấy clinical guidelines từ database Vinmec
Phase 3 (Confidence Calibration): Cross-check với decision tree dựa trên điều trị

3. Implement A/B Testing Framework:
So sánh prompt version A vs. B trên test set thực tế (không chỉ 30 test cases) để see which one performs better.

7. AI giúp gì và AI sai/mislead ở đâu?
AI giúp:

Sinh prompt templates rất nhanh: Tôi describe use case, AI viết ra prompt initial structure (system + user + examples). Tiết kiệm ~1-2 tiếng.
Tạo test cases phức tạp: Nhờ AI generate 100+ medical scenarios (triệu chứng combinations) để test agent. Mà không cần ask doctor.
Debug logic: Nhờ AI explain lại JSON output của nó theo "reasoning" field, giúp trace bug nhanh.

AI sai/mislead:

Hallucinate về medical accuracy: AI tự tin nói "Accuracy này đạt 95%+" mà thực tế chỉ test trên 20 cases. Tôi mất thời gian verify rồi discover nó mislead.
Gợi ý approach không phù hợp domain: AI hay suggest "dùng reinforcement learning" hoặc "fine-tune GPT" mà không hiểu constraint của hackathon (thời gian, budget, data). Phải manually reject và explain lại scope.
Confidence score bias: AI generate prompt mà model luôn output confidence cao (vd: 0.85 trung bình) nhưng không phản ánh thực tế. Phải explicitly add instruction "Be conservative with confidence".


Tóm lại:
Vai trò "bộ não AI agent" yêu cầu kết hợp:

Prompt engineering (chain-of-thought, few-shot)
Decision logic (routing, heuristic)
Evaluation (accuracy, calibration, safety)
Debugging (tracing reasoning, fixing hallucination)
