# Data-AI-Candidate-Ranker
# Intelligent Candidate Discovery & Ranking Engine

## Architecture

```
Job Description (text)
        │
        ▼
  ┌─────────────┐
  │  JD Parser  │ → required skills, preferred skills, min exp, education
  └─────────────┘
        │
        ▼
┌───────────────────────────────────────────────────┐
│                  SCORING ENGINE                   │
│                                                   │
│  ┌──────────────────┐  ┌─────────────────────┐   │
│  │ Semantic Layer   │  │ Structured Layer     │   │
│  │                  │  │                      │   │
│  │ sentence-        │  │ ▸ Skill match (35%) │   │
│  │ transformers     │  │   required + pref   │   │
│  │ (MiniLM-L6-v2)  │  │                      │   │
│  │ cosine sim (30%) │  │ ▸ Experience (20%)  │   │
│  │                  │  │   years + seniority │   │
│  │ Fallback: TF-IDF │  │                      │   │
│  └──────────────────┘  │ ▸ Education (10%)   │   │
│                         │   degree level      │   │
│                         │                     │   │
│                         │ ▸ Activity (5%)     │   │
│                         │   GitHub, Kaggle,   │   │
│                         │   LeetCode signals  │   │
│                         └─────────────────────┘   │
│                                                   │
│         Weighted Total Score + Explanation        │
└───────────────────────────────────────────────────┘
        │
        ▼
  Ranked Candidate List (JSON output)
  + Per-candidate explanation
  + Matched/missing skills
  + Score breakdown by dimension
```

## Quick Start

```bash
# 1. Install dependencies
pip install -r requirements.txt

# 2. Run demo (generates 100 synthetic candidates)
python demo.py

# 3. Run with your real dataset
python demo.py /path/to/your/dataset.json

# 4. Start the API server
pip install fastapi uvicorn python-multipart
uvicorn api:app --reload --port 8000
# Then open http://localhost:8000/docs
```

## Dataset Format

The engine auto-detects field names. Supported input shapes:

**JSON Array:**
```json
[
  {
    "id": "C001",
    "name": "Jane Doe",
    "current_title": "ML Engineer",
    "skills": ["python", "pytorch", "nlp"],
    "experience_years": 6,
    "education": "Master's in CS",
    "summary": "5 years building production ML...",
    "github_contributions": 450
  }
]
```

**CSV:** Column names are auto-normalized.

**Nested JSON:** If candidates are under a key like `"data"`, `"candidates"`, or `"profiles"`, 
the parser finds them automatically.

## Scoring Weights

| Dimension | Weight | What it measures |
|-----------|--------|-----------------|
| Semantic match | 30% | Embedding cosine similarity between JD and full profile |
| Skill match | 35% | Overlap of required (80%) and preferred (20%) skills |
| Experience | 20% | Years of exp vs. JD minimum requirement |
| Education | 10% | Degree level vs. JD education requirement |
| Platform activity | 5% | GitHub, Kaggle, LeetCode, publications |

## Output

`outputs/ranked_candidates.json`:
```json
[
  {
    "rank": 1,
    "name": "Jane Doe",
    "total_score": 0.823,
    "score_breakdown": {
      "semantic_match": 0.74,
      "skill_match": 0.92,
      "experience_fit": 0.85,
      "education_fit": 0.80,
      "platform_activity": 0.60
    },
    "matched_skills": ["python", "pytorch", "nlp", "langchain", "faiss"],
    "missing_skills": ["kubernetes"],
    "explanation": "Strong fit (82% overall match). Matched skills: python, pytorch, nlp, langchain, faiss. Experience: 6yr meets the 5yr requirement. Semantic profile match: 74%."
  }
]
```

## Files

```
candidate_engine/
├── engine.py          Core engine (JD parser, candidate parser, scorer)
├── demo.py            CLI demo runner + synthetic data generator
├── api.py             FastAPI REST server
├── requirements.txt   Python dependencies
├── data/              Input datasets
└── outputs/           Ranked output JSON
```
