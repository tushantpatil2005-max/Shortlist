# AI Resume & Job Matching Platform

**Recruitbots AI — LLM Application Engineer · Challenge 2 (Intermediate)**

One implementation file: [`resume_match_platform.py`](resume_match_platform.py).

**Shortlist** extracts structured candidate data from resumes, ranks applicants against job descriptions with **embedding similarity** (not hand-tuned score weights), explains each ranking, and produces recruiter-ready summaries. Supports multiple job categories, batch processing, and published evaluation metrics.

Recruiter-facing text has no machine-origin labels.

---

## Quick start

```bash
python resume_match_platform.py --check
python resume_match_platform.py --evaluate
python resume_match_platform.py --rank-samples --job data_analyst

# HTTP API
pip install fastapi uvicorn
python resume_match_platform.py --serve
# POST /extract  POST /match  POST /rank  POST /batch  GET /jobs  GET /evaluate
```

---

## Architecture

```text
Resume text                    Job catalog (multi-category)
     │                                  │
     ▼                                  ▼
 Structured extraction            Job embedding text
 (name, email, skills, years…)         │
     │                                  │
     └──────────► cosine similarity ◄───┘
                       │
                       ▼
              Rank score + explanations
              (overlap, gaps, experience)
                       │
                       ▼
           Recruiter summary (structured JSON)
```

**No manual scoring rules.** The rank key is cosine similarity between feature-hashed embeddings of the resume and the job. Skill Jaccard and experience gap are returned as *explanations*, not as weighted rule components.

---

## Success criteria

| Criterion | Implementation |
| --- | --- |
| Structured resume data | `extract` → name, email, skills, years, education, titles |
| Candidate rankings | `rank` / `batch` ordered by `rank_score` |
| Explainable recommendations | Overlapping / missing skills, experience gap, prose explanation |
| Multiple job categories | engineering, analytics, infrastructure, machine_learning |
| Recruiter-ready summaries | `recruiter_summary` on every match and rank row |

---

## Constraints

| Constraint | How it is met |
| --- | --- |
| Structured outputs | Stable JSON schemas for extract, match, rank, batch |
| Batch processing | `rank` (many resumes × one job) and `batch_match` (many pairs) |
| No manual scoring rules | Similarity-only rank score; no hand-tuned skill/experience weights |
| Evaluation metrics | Extraction F1 / accuracy; matching top-1, top-3, MRR on labeled samples |

---

## Job categories

| Id | Title | Category |
| --- | --- | --- |
| `backend_engineer` | Backend Engineer | engineering |
| `frontend_engineer` | Frontend Engineer | engineering |
| `data_analyst` | Data Analyst | analytics |
| `devops_engineer` | DevOps Engineer | infrastructure |
| `ml_engineer` | Machine Learning Engineer | machine_learning |

---

## Evaluation metrics

`GET /evaluate` or `python resume_match_platform.py --evaluate`:

**Extraction**

- name / email accuracy  
- skill precision, recall, F1  
- years mean absolute error  
- education hit rate  

**Matching**

- top-1 / top-3 accuracy vs gold-best candidate per job  
- mean reciprocal rank (MRR)  

---

## Example match output

```json
{
  "status": "complete",
  "candidate": { "name": "Maya Chen", "skills": ["python", "fastapi", "…"] },
  "job": { "id": "backend_engineer", "title": "Backend Engineer", "category": "engineering" },
  "scores": { "rank_score": 41.2, "similarity": 0.412, "skill_jaccard": 0.75 },
  "overlapping_skills": ["python", "sql", "docker"],
  "missing_skills": [],
  "explanation": "Maya Chen ranks at 41.2 similarity for Backend Engineer. …",
  "recruiter_summary": "#1 Maya Chen — Backend Engineer (score 41.2). …"
}
```

---

## Unmarked text

`strip_machine_labels` removes “AI-generated”, “powered by AI”, and similar phrases from explanations and summaries.

---

## Self-check

`python resume_match_platform.py --check` asserts extraction accuracy, cross-category ranking, batch rank/match, evaluation metric floors, structured keys, and clean recruiter text.

---

## File layout

```text
resume_match_platform.py   # platform, metrics, API, self-check
README_resume_match.md     # this document
```
