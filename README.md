# VietMCQ-AI v3.0 — HackAIthon 2026 · Bảng C INNOVATOR

> RAG + Knowledge Base + Self-Consistency Ensemble · 100% offline · Docker

## Chạy 1 lệnh

```bash
docker run --rm \
  -v $(pwd)/data:/data \
  -v $(pwd)/output:/output \
  --gpus all \
  phamvantuyenuit2006-oss/vietmcq-ai:latest
```

Kết quả: `/output/pred.csv` với 2 cột `qid, answer (A/B/C/D)`

## Kiến trúc

```
public_test.csv
    │
    ▼
[Subject Detection]     → Toán / Lý / Hóa / Sinh / Văn / Sử / Địa / CNTT...
    │
    ▼
[Hybrid RAG Retrieval]
  ├─ Knowledge Base (kiến thức nhúng sẵn, đa môn)   → top-10
  └─ Self-Retrieval (câu hỏi tương tự trong đề)      → top-10
    │
    ▼
[Qwen3-Reranker]        → top-6 context chất lượng nhất
    │
    ▼
[Qwen3.5-7B-Instruct]
  × 3 lần generate      → Self-consistency ensemble
    │
    ▼
[Majority Vote + Confidence Scoring]
  └─ conf < 60%         → Re-generate với few-shot mạnh hơn
    │
    ▼
/output/pred.csv
```

## Models (đúng quy định BTC)

| Vai trò | Model |
|---------|-------|
| LLM | `Qwen/Qwen3.5-7B-Instruct` (≤9B) |
| Embedding | `BAAI/bge-m3` |
| Reranker | `Qwen/Qwen3-Reranker-0.6B` |

## Build từ source

```bash
git clone https://github.com/phamvantuyenuit2006-oss/vietmcq-ai.git
cd vietmcq-ai
docker build -t vietmcq-ai:latest .
cp public_test.csv data/
docker run --rm -v $(pwd)/data:/data -v $(pwd)/output:/output --gpus all vietmcq-ai:latest
```

## Unit Tests

```bash
python3 tests/test_pipeline.py
# → 28 tests, ALL PASSED (3/3 lần ổn định)
```

## Tùy chọn

| Flag | Mặc định | Mô tả |
|------|----------|-------|
| `--n-samples` | 3 | Số lần generate ensemble |
| `--no-ensemble` | false | Tắt ensemble (nhanh hơn) |
| `--no-rag` | false | Tắt RAG |
| `--no-kb` | false | Tắt Knowledge Base |
| `--batch-size` | auto | Tự động theo VRAM |
| `--debug` | false | Chỉ 20 câu đầu |
