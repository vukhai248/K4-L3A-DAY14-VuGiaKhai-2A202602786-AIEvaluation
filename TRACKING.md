# 📋 BẢNG THEO DÕI TIẾN ĐỘ LAB (TRACKING DASHBOARD)
## Day 14 — AI Evaluation & Benchmarking Pipeline (K4 - Level 3A)

- **Học viên:** Vũ Gia Khải
- **Mã số sinh viên (MSSV):** `2A202602786`
- **Tên Repository:** `K4-L3A-DAY14-VuGiaKhai-2A202602786-AIEvaluation` (Đúng chuẩn quy định, không bị trừ 5 điểm)
- **Hạn nộp:** 23h59 ngày làm lab (LMS / Codelab)

---

## 🎯 1. BẢNG TỔNG QUAN TIÊU CHÍ CHẤM ĐIỂM (RUBRIC - 100 ĐIỂM BẮT BUỘC + 10 BONUS)

| STT | Tiêu chí | Điểm tối đa | Trạng thái | Ghi chú / Yêu cầu chính |
|:---:|---|:---:|:---:|---|
| 1 | **Core Coding & Tests Pass** | 50 | ⏳ Chưa xong | Hoàn thành Task 1–5 trong `template.py` / `solution/solution.py`, pass 41/42 tests |
| 2 | **Golden Dataset (20 QA)** | 15 | ⏳ Chưa xong | 20 QA (5 Easy, 7 Medium, 5 Hard, 3 Adversarial), phủ đủ 10 docs, validate PASS |
| 3 | **LLM-as-a-Judge Rubric Design** | 10 | ⏳ Chưa xong | Exercise 3.3 trong `exercises.md`: rubric 1–5 domain OrbitTech, kiểm soát 3 bias |
| 4 | **Benchmark, 5 Whys & Failure Analysis** | 15 | ⏳ Chưa xong | Exercise 3.2, 3 cases 5 Whys trong `reflection.md`, failure taxonomy & improvement log |
| 5 | **Code Quality & Regression Strategy** | 10 | ⏳ Chưa xong | Clean code, type hints, chiến lược CI/CD quality gate chặn drop > 0.05 trong `reflection.md` |
| **TỔNG** | **Bắt buộc** | **100** | | |
| *Bonus* | Exercise 3.4 (So sánh 2 frameworks) | +5 | ⚪ Tùy chọn | So sánh RAGAS vs DeepEval/TruLens trên cùng dataset trong `exercises.md` |
| *Bonus* | Exercise 3.5 (Retrieval Reranking) | +5 | ⚪ Tùy chọn | Implement `rerank_by_overlap()`, chạy test thứ 42 và đo delta Context Precision |

---

## 📦 2. DANH SÁCH SẢN PHẨM NỘP BÀI (DELIVERABLES)

- [ ] `solution/solution.py`: Bản sao hoàn thiện từ `template.py` (Tất cả 5 Tasks bắt buộc).
- [ ] `golden_dataset.json`: File dataset 20 QA đã điền đầy đủ và pass `validate_golden_dataset.py`.
- [ ] `exercises.md`: Đã hoàn thiện Part 1 (1.1, 1.2, 1.3), Part 3 (3.1, 3.2, 3.3, và 3.4/3.5 nếu làm bonus).
- [ ] `reflection.md`: Đã hoàn thiện toàn bộ 7 mục (summary, 3 case 5 Whys, clustering, improvement log, regression, loop, reflection).
- [ ] *(Tạo tự động trong quá trình chạy)* `artifacts/actual_answers.json` & `artifacts/benchmark_results.json`.

> ⚠️ **CẢNH BÁO BẢO MẬT & TRỪ ĐIỂM:**
> - Tuyệt đối **KHÔNG commit** `.env` chứa `OPENAI_API_KEY` lên Git (vi phạm trừ **10 điểm**).
> - Không can thiệp sửa đổi các file test trong thư mục `tests/` (vi phạm hủy toàn bộ 50 điểm code).

---

## 🚀 3. LỘ TRÌNH THỰC HIỆN CHI TIẾT THEO CHECKPOINTS (CP0 -> CP5)

### 🔹 CHECKPOINT 0: Setup & Baseline Môi trường
- [x] **CP0.1** Kiểm tra phiên bản Python (yêu cầu Python 3.11+): `Python 3.13.13` (Conda `DL`).
- [x] **CP0.2** Cài đặt/xác nhận dependencies: `openai`, `python-dotenv`, `pytest` đã sẵn sàng.
- [ ] **CP0.3** Tạo file `.env` từ `.env.example` và cấu hình:
  ```powershell
  Copy-Item .env.example .env
  ```
  *(Cần điền `OPENAI_API_KEY` trước khi chạy RAG ở CP4).*
- [x] **CP0.4** Chạy baseline tests:
  ```powershell
  pytest tests/ -v
  ```
  *Kết quả ban đầu: 42 collected, 42 failed (chuẩn baseline).*

---

### 🔹 CHECKPOINT 1: Hoàn thành Task 1 — Data Models (CP1)
- [x] **CP1.1** Khai báo dataclass `QAPair` trong `template.py`:
  - `question: str`
  - `expected_answer: str`
  - `context: str | None = ""`
  - `metadata: dict = field(default_factory=dict)`
  - `retrieved_contexts: list = field(default_factory=list)`
- [x] **CP1.2** Khai báo dataclass `EvalResult` trong `template.py`:
  - Các answer metrics: `faithfulness: float`, `relevance: float`, `completeness: float`
  - Retrieval metrics (optional): `context_recall: float | None = None`, `context_precision: float | None = None`
  - Kết quả: `passed: bool = True`, `failure_type: str | None = None`
  - Method `overall_score() -> float`: trả về trung bình cộng `(faithfulness + relevance + completeness) / 3.0`
- [x] **CP1.3** Kiểm tra Targeted Test Task 1:
  ```powershell
  pytest tests/test_solution.py::TestEvalResultOverallScore -v
  ```
  *Kết quả:* **3 passed in 0.07s**.

---

### 🔹 CHECKPOINT 2: Task 2 (RAGAS Metrics) & Task 3 (LLM Judge) (CP2)
- [x] **CP2.1** Hoàn thiện 3 Answer-side Metrics trong `RAGASEvaluator`:
  - `evaluate_faithfulness(answer, context)` (đo hallucination: token answer có xuất hiện trong context không).
  - `evaluate_relevance(answer, question)` (đo độ liên quan của answer với query).
  - `evaluate_completeness(answer, expected)` (đo độ đầy đủ so với ground-truth).
- [x] **CP2.2** Hoàn thiện 2 Retrieval-side Metrics trong `RAGASEvaluator`:
  - `evaluate_context_recall(contexts, expected)` (đo độ bao phủ ground-truth của union các retrieved contexts).
  - `evaluate_context_precision(contexts, expected)` (đo rank-aware Average Precision@K, chunk relevant ở top được điểm cao).
- [x] **CP2.3** Hoàn thiện `run_full_eval()`:
  - Luôn tính 3 answer metrics.
  - Nếu có `contexts`: gọi 2 retrieval metrics; nếu `contexts is None`: gán 2 retrieval metrics là `None`.
  - Phân loại `failure_type`: "hallucination", "irrelevant", "incomplete", "off_topic".
- [x] **CP2.4** Hoàn thiện `LLMJudge`:
  - `score_response(question, answer, rubric)`: gọi LLM mock/real, parse điểm JSON (fallback 0.5 nếu lỗi).
  - `detect_bias(scores_batch)`: phát hiện positional bias, leniency bias (> 0.8), severity bias (< 0.3).
- [x] **CP2.5** Kiểm tra Targeted Test Task 2 & Task 3:
  ```powershell
  pytest tests/test_solution.py::TestRAGASEvaluator tests/test_solution.py::TestContextMetrics tests/test_solution.py::TestRetrievalMetricWiring::test_run_full_eval_connects_optional_retrieval_metrics tests/test_solution.py::TestLLMJudge -v
  ```
  *Kết quả:* **19/19 passed in 0.06s** (bao gồm cả test bonus reranking). Toàn suite đạt **22 passed, 20 failed**.

---

### 🔹 CHECKPOINT 3: Task 4 (Benchmark Runner) & Task 5 (Failure Analyzer) (CP3)
- [ ] **CP3.1** Hoàn thiện `BenchmarkRunner`:
  - `run(qa_pairs, agent_fn, evaluator)`: gọi agent, forward `retrieved_contexts` vào `run_full_eval()`.
  - `generate_report(results)`: tính pass rate, average answer metrics và average retrieval metrics (bỏ qua `None`).
  - `run_regression(new_results, baseline_results)`: phát hiện metric tụt > 0.05 so với baseline.
  - `identify_failures(results, threshold)`: lọc danh sách kết quả có `overall_score < threshold` hoặc `passed == False`.
- [ ] **CP3.2** Hoàn thiện `FailureAnalyzer`:
  - `categorize_failures(failures)`: đếm số lượng lỗi theo từng `failure_type`.
  - `find_root_cause(failure)`: suy luận nguyên nhân gốc dựa vào metric thấp nhất.
  - `generate_improvement_suggestions(failures)`: đưa ra ít nhất 3 hành động cụ thể.
  - `generate_improvement_log(failures, suggestions)`: sinh bảng Markdown tracking lỗi và đề xuất xử lý.
- [ ] **CP3.3** Đồng bộ `template.py` sang `solution/solution.py`:
  ```powershell
  Copy-Item template.py solution/solution.py
  ```
- [ ] **CP3.4** Chạy toàn bộ Test Suite bắt buộc:
  ```powershell
  pytest tests/ -v
  ```
  *Kỳ vọng:* **41 passed, 1 skipped** (test reranking skipped nếu chưa làm bonus).

---

### 🔹 CHECKPOINT 4: Golden Dataset (20 QA) & Real Benchmark Run (CP4)
- [ ] **CP4.1** Đọc corpus trong `data/technology_store/` (10 documents từ `00_` đến `09_`).
- [ ] **CP4.2** Điền `golden_dataset.json` với đúng 20 QA:
  - 5 Easy (`E01` - `E05`): Factual lookup, 1 doc.
  - 7 Medium (`M01` - `M07`): Multi-step / multi-doc (2-3 docs).
  - 5 Hard (`H01` - `H05`): Điều kiện, ngoại lệ, chính sách ngày tháng, xung đột phiên bản.
  - 3 Adversarial:
    - `A01` (`out_of_scope`): Trả lời ngoài phạm vi hỗ trợ OrbitTech.
    - `A02` (`prompt_injection`): Cố tình bypass system rules.
    - `A03` (`false_premise_or_ambiguous_trap`): Giả định sai sự thật.
  - *Lưu ý sống còn:* `text` trong contexts phải là **verbatim substring** (trích dẫn nguyên văn) từ tài liệu nguồn; toàn bộ 20 QA phải bao phủ đủ 10 file tài liệu nguồn ít nhất 1 lần.
- [ ] **CP4.3** Chạy validator xác nhận dataset hợp lệ:
  ```powershell
  python validate_golden_dataset.py
  ```
  *Kỳ vọng:* **`PASS: dataset structure and evidence provenance are valid.`**
- [ ] **CP4.4** Điền Exercise 1.1, 1.2, 1.3 và Exercise 3.1 trong `exercises.md`.
- [ ] **CP4.5** Sinh 20 câu trả lời thật từ RAG assistant (cần OpenAI API key):
  ```powershell
  python domain_assistant.py
  ```
  *Kiểm tra file output được tạo ra:* `artifacts/actual_answers.json`.
- [ ] **CP4.6** Chạy pipeline chấm benchmark thật:
  ```powershell
  python evaluate_answers.py
  ```
  *Kiểm tra file output được tạo ra:* `artifacts/benchmark_results.json`.
- [ ] **CP4.7** Điền kết quả thật vào Exercise 3.2 và viết Rubric vào Exercise 3.3 trong `exercises.md`.

---

### 🔹 CHECKPOINT 5: Reflection, Báo cáo & Kiểm tra Cuối (CP5)
- [ ] **CP5.1** Điền đầy đủ file `reflection.md`:
  - [ ] Mục 1: Benchmark Results Summary (Overall pass rate, bảng 5 metrics, phân bố failure).
  - [ ] Mục 2: Top 3 Worst Failures với phương pháp **5 Whys** (đi sâu từ Symptom -> Root cause).
  - [ ] Mục 3: Failure Clustering (gom nhóm theo root cause, chọn cluster ưu tiên).
  - [ ] Mục 4: Improvement Log (paste bảng Markdown từ `generate_improvement_log()`, 3 đề xuất).
  - [ ] Mục 5: Regression Testing Strategy (ngưỡng 0.05, block vs alert, CI/CD stages).
  - [ ] Mục 6: Continuous Improvement Loop & 2-3 ca kiểm thử bổ sung.
  - [ ] Mục 7: Final Reflection (điều bất ngờ & giới hạn của word-overlap heuristic).
- [ ] **CP5.2** Đồng bộ code lần cuối sang `solution/solution.py`:
  ```powershell
  Copy-Item template.py solution/solution.py
  ```
- [ ] **CP5.3** Chạy kiểm tra toàn diện:
  ```powershell
  pytest tests/ -v
  python validate_golden_dataset.py
  ```
- [ ] **CP5.4** Rà soát git status, đảm bảo KHÔNG commit `.env`:
  ```powershell
  git status
  ```

---

## 🌟 4. BONUS TRACK (TÙY CHỌN - TỐI ĐA +10 ĐIỂM)

- [ ] **Bonus 1 (Exercise 3.4 - +5 điểm):** So sánh 2 frameworks đánh giá AI (RAGAS vs DeepEval / TruLens). Hoàn thiện phân tích trong `exercises.md`.
- [ ] **Bonus 2 (Exercise 3.5 - +5 điểm):** Implement `rerank_by_overlap()` trong `template.py` / `solution/solution.py`, chạy full test suite đạt **42/42 passed**, đo đạc kết quả trước/sau rerank trong `exercises.md`.

---

## 📝 5. NHẬT KÝ TIẾN ĐỘ THỰC HIỆN

| Thời điểm | Mục tiêu / Checkpoint | Kết quả thực tế | Người thực hiện / Ghi chú |
|---|---|---|---|
| Khởi tạo | Đọc hiểu repo & tạo file tracking | Đã hoàn thành phân tích toàn bộ repo, tạo TRACKING.md | AI Pair Programmer |
| CP0 | Xác nhận môi trường & baseline test | Python 3.13.13, 42 tests failed đúng baseline | AI Pair Programmer |
| CP1 | Hoàn thành Task 1 (Data Models) | `TestEvalResultOverallScore` PASS 3/3 tests | AI Pair Programmer |
| CP2 | Hoàn thành Task 2 & 3 (RAGAS & LLMJudge) | Targeted tests PASS 19/19 tests (cộng dồn 22 passed, 20 failed) | AI Pair Programmer |
| | | | |
