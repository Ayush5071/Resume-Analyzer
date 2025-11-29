# AI-Based Resume–Job Fit & Skill Gap Intelligence System

A production-grade AI platform that evaluates how well a candidate’s resume matches a given job description (JD). The system computes a **job–resume fit score**, identifies **skill gaps**, highlights **experience alignment**, and generates **personalized learning paths** using LLM reasoning.

---

## 1. Problem Statement

Most Applicant Tracking Systems (ATS) rely on:

* Simple keyword matching
* Rigid rule-based filters
* Static CV templates

These approaches:

* Miss candidates with non-traditional backgrounds
* Ignore transferable skills and project depth
* Fail to explain *why* a candidate is a good or bad fit

**Goal of this project:**

> Build an AI system that understands resumes and job descriptions semantically, computes a meaningful fit score, surfaces skill gaps, and recommends next steps for both candidates and hiring teams.

---

## 2. Key Features

* **Semantic Resume–JD Matching**
  Uses text embeddings + vector similarity to measure alignment beyond keywords.

* **Automatic Skill Extraction**
  Extracts technical, soft, and domain skills from resumes and JDs using NER + custom dictionaries.

* **Experience & Seniority Alignment**
  Analyzes years of experience, role titles, and project responsibilities.

* **Skill Gap Analysis**
  Identifies missing or weak skills compared to job requirements.

* **LLM-Powered Learning Path Generator**
  Uses Gemini (or any LLM) to propose tailored upskilling paths (courses, topics, project ideas).

* **HR Insights Dashboard**
  Visual matrix of candidates vs roles with scores, gaps, and explanations.

* **Privacy by Design**
  PII masking on resumes using Presidio-style entities (name, email, phone, address).

---

## 3. High-Level Architecture

```text
                        ┌────────────────────────┐
                        │        Frontend        │
                        │  (Next.js / React)     │
                        │                        │
                        │ ▸ Resume upload        │
                        │ ▸ JD creation          │
                        │ ▸ Fit score dashboard  │
                        └───────────┬────────────┘
                                    │  HTTPS (REST)
                                    ▼
                        ┌────────────────────────┐
                        │     Backend API        │
                        │      (FastAPI)         │
                        │                        │
                        │ ▸ Auth (optional)      │
                        │ ▸ Resume/JD APIs       │
                        │ ▸ Scoring endpoint     │
                        └───────────┬────────────┘
                                    │
              ┌─────────────────────┼─────────────────────────┐
              ▼                     ▼                         ▼
   ┌────────────────┐    ┌─────────────────────┐    ┌─────────────────────┐
   │  NLP Pipeline  │    │ Embedding + Vector  │    │   LLM Orchestration  │
   │ (spaCy/Custom) │    │  Store (FAISS/etc.)│    │  (Gemini / OpenAI)   │
   └──────┬─────────┘    └──────────┬──────────┘    └──────────┬──────────┘
          │                         │                          │
          ▼                         ▼                          ▼
   ┌───────────────┐       ┌────────────────┐          ┌───────────────────┐
   │ Parsed Resume │       │  Similarity    │          │  Learning Path /   │
   │ + Skills JSON │       │  & Fit Score   │          │  Justifications    │
   └───────────────┘       └────────────────┘          └───────────────────┘
                                    │
                                    ▼
                          ┌────────────────────┐
                          │ Database (MongoDB) │
                          │ ▸ Users            │
                          │ ▸ Resumes          │
                          │ ▸ JDs              │
                          │ ▸ Scores & logs    │
                          └────────────────────┘
```

---

## 4. Data Flow

1. **User uploads resume (PDF/DOCX)** via frontend.
2. Frontend sends file → Backend `/api/resume/upload`.
3. Backend parses text using **pdfplumber / docx2txt** + optional OCR.
4. NLP pipeline:

   * Cleans and normalizes text
   * Extracts skills, tools, frameworks
   * Extracts education, experience, roles
5. User or HR selects / creates a **Job Description (JD)**.
6. Backend converts **resume + JD** into embeddings using a sentence transformer or Gemini embedding API.
7. Embeddings stored / queried in **FAISS / Pinecone** for similarity.
8. System computes:

   * Similarity score
   * Missing skills (JD skills – Resume skills)
   * Strengths (intersection of JD and resume skills)
9. LLM (Gemini) is prompted with structured JSON context to:

   * Explain the score in natural language
   * Suggest a learning path (topics, order, project ideas)
10. Results (scores + explanation + path) are returned to frontend and optionally persisted in MongoDB.

---

## 5. Tech Stack (Detailed)

### 5.1 Backend

* **Language:** Python 3.10+
* **Framework:** FastAPI
* **Auth (optional):** JWT / OAuth2
* **Task Queue (optional for async):** Celery / RQ + Redis

### 5.2 NLP & Embeddings

* **Text Parsing:** pdfplumber, python-docx, PyMuPDF
* **Preprocessing:** spaCy / NLTK (tokenization, lemmatization, stopwords)
* **Skill Extraction:**

  * Custom skill dictionary (JSON of skills → categories)
  * spaCy NER / PhraseMatcher
* **Embeddings:**

  * SentenceTransformers (`all-MiniLM-L6-v2` or better), or
  * Gemini / OpenAI Embeddings (for cloud-based semantic search)

### 5.3 Vector Similarity

* **Local:** FAISS / ChromaDB
* **Cloud (optional):** Pinecone

### 5.4 LLM Orchestration

* **Primary LLM:** Gemini (or any chat-optimised LLM)
* **Use Cases:**

  * Explanation of fit score
  * Learning roadmap generation
  * Role suggestions based on profile

### 5.5 Database

* **MongoDB** (document store):

  * `users`: profile + auth
  * `resumes`: parsed resumes + embeddings
  * `job_descriptions`: JD text + extracted skills
  * `matches`: scores, explanations, timestamps

### 5.6 Frontend

* **Framework:** Next.js / React
* **UI:** Tailwind CSS / Chakra UI
* **Features:**

  * Resume upload, JD management
  * Table of candidates vs roles
  * Detail view per candidate (skills, gaps, explanation)

### 5.7 DevOps / Infra (Optional)

* **Containerization:** Docker
* **Orchestration:** Docker Compose / Kubernetes (for scaling)
* **Hosting:** Render / Railway / Azure App Service / Vercel (frontend)

---

## 6. Suggested Project Structure

```text
skill-fit-ai/
├── backend/
│   ├── app/
│   │   ├── main.py              # FastAPI entrypoint
│   │   ├── config.py            # Settings, keys, env vars
│   │   ├── models/              # Pydantic models, DB schemas
│   │   ├── routers/             # API routes
│   │   │   ├── resumes.py
│   │   │   ├── jobs.py
│   │   │   ├── matches.py
│   │   ├── services/            # Business logic
│   │   │   ├── parsing_service.py
│   │   │   ├── embedding_service.py
│   │   │   ├── scoring_service.py
│   │   │   ├── learning_path_service.py
│   │   ├── nlp/                 # NLP/Skill extraction
│   │   │   ├── skill_dict.json
│   │   │   ├── nlp_pipeline.py
│   │   ├── db/                  # DB connection, queries
│   │   │   ├── mongo.py
│   │   └── utils/
│   │       ├── logger.py
│   │       ├── security.py
│   ├── requirements.txt
│   └── Dockerfile
│
├── frontend/
│   ├── app/ or src/
│   │   ├── pages/
│   │   ├── components/
│   │   │   ├── ResumeUpload.tsx
│   │   │   ├── JobForm.tsx
│   │   │   ├── MatchTable.tsx
│   │   │   └── MatchDetailDrawer.tsx
│   ├── package.json
│   └── Dockerfile
│
├── docs/
│   ├── architecture-diagram.png
│   └── api-spec.md
│
├── README.md
└── docker-compose.yml
```

---

## 7. API Design (Example)

### 7.1 Upload Resume

`POST /api/resumes`

**Body (multipart/form-data):**

* `file`: PDF/DOCX resume

**Response:**

```json
{
  "resume_id": "64f1b1...",
  "extracted_skills": ["Python", "FastAPI", "React"],
  "years_experience": 2.5
}
```

### 7.2 Create / Submit Job Description

`POST /api/jobs`

**Body (JSON):**

```json
{
  "title": "Data Science Intern",
  "description": "We are looking for...",
  "location": "Bangalore, India"
}
```

**Response:**

```json
{
  "job_id": "job_123",
  "required_skills": ["Python", "Statistics", "SQL", "Machine Learning"]
}
```

### 7.3 Compute Match Score

`POST /api/matches`

**Body (JSON):**

```json
{
  "resume_id": "64f1b1...",
  "job_id": "job_123"
}
```

**Response:**

```json
{
  "match_id": "match_001",
  "fit_score": 0.82,
  "fit_band": "Strong",
  "matched_skills": ["Python", "SQL"],
  "missing_skills": ["Statistics", "Machine Learning"],
  "llm_explanation": "The candidate is a strong fit because...",
  "learning_path": [
    {
      "topic": "Statistics for Data Science",
      "resources": ["Khan Academy", "StatQuest"]
    },
    {
      "topic": "Supervised ML Basics",
      "resources": ["Andrew Ng Coursera"]
    }
  ]
}
```

---

## 8. LLM Prompting Strategy (Gemini / OpenAI)

### 8.1 Example Prompt for Explanation

> You are an expert technical recruiter.
> You are given:
>
> * A structured JSON for a candidate resume
>
> * A structured JSON for a job description
>
> * A list of matched skills and missing skills
>
> 1. Explain in 4–6 sentences how well this candidate fits the role.
> 2. Highlight 3 key strengths.
> 3. Highlight 3 key improvement areas.
> 4. Keep the tone professional and concise.

### 8.2 Example Prompt for Learning Path

> You are an AI career coach.
> Based on this candidate's skills and missing skills, design a 6–8 week learning roadmap.
> Include:
>
> * Weekly topics
> * 1–2 suggested resources per week
> * 1 project idea at the end.

---

## 9. Setup & Installation (Backend)

```bash
cd backend
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate
pip install -r requirements.txt

# Set environment variables (example)
export MONGO_URI="mongodb://localhost:27017/skillfit"
export EMBEDDING_MODEL="sentence-transformers/all-MiniLM-L6-v2"
export GEMINI_API_KEY="your_key_here"

uvicorn app.main:app --reload
```

## 10. Setup & Installation (Frontend)

```bash
cd frontend
npm install
npm run dev
```

Then open `http://localhost:3000` in your browser.

---

## 11. Future Enhancements

* Add **multi-role recommendation** (suggest better roles based on profile).
* Support **multiple languages** for resumes and JDs.
* Add **interview question generation** based on candidate gaps.
* Log and visualize LLM explanations for fairness and bias checks.
* Integrate with real ATS via webhooks / API.

---

## 12. How to Present This Project (Resume-Friendly)

> Built an AI-based system that ranks resumes against job descriptions using semantic embeddings and LLM reasoning, performs skill-gap analysis, and generates personalized learning paths; designed a full-stack architecture with FastAPI, MongoDB, FAISS, and a Next.js dashboard for HR insights.

You can adapt this README and simplify or extend sections based on the actual scope you implement. This structure is strong enough for internships at Microsoft, other big tech, or research-focused roles.
