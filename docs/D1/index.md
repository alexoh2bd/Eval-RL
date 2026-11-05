### Day 1 — What makes a benchmark valid?

**Read:**

- [Evaluation and Benchmarking of LLM Agents: A Survey (KDD 2025)](https://doi.org/10.1145/3711896.3736570)
- [Introducing Harbor-Index](https://harbor-index.org/)

**Learn:** Construct validity, task distributions, metric selection, benchmark difficulty versus broken tasks.

**Exercises:**

- Analyze three existing agent benchmarks. Identify their target capabilities, task distributions, success criteria, and possible shortcuts.
  - [Legal Agent Benchmark](https://www.harvey.ai/blog/introducing-harveys-legal-agent-benchmark) (LAB) (HARVEY): Long-horizon legal tasks 
    - Instructions: Partner to associate requests for work, 
    - Environment: Closed Universe of documents and materials for files, templates, email comms, and other info to discover 
    - Output: Reviewable legal work product
    - Verified by expert rubrics (format, facts, correct analysis)
  - [Physical-Engineering's Last Exam](https://arxiv.org/pdf/2610.30537)
    - 
- Define five failure categories for document-investigation agents: 
  - missing findings, wrong evidence, incorrect citations, conflicting statements, and hallucinated claims.
- Write a short benchmark specification identifying exactly what success means.
- 

**Build:** Create the VerifierLab repository, initialize Harbor, implement one task with an oracle reference solution and a no-op baseline.

**Deliverable:** `benchmark_spec.md` and one validated Harbor task.