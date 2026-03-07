# Qriocity – AI-Powered Adaptive Learning Platform

An intelligent educational platform that leverages AI to deliver personalized learning experiences. Students upload course materials, which are processed through retrieval-augmented generation (RAG) to generate adaptive quizzes, teaching content, and career recommendations based on learning mastery.

## Features

- **AI-Powered Curriculum Processing**: Parse course PDFs and generate structured learning materials
- **Adaptive Quizzes**: Generate contextual quizzes from course materials using Groq LLM
- **Mastery Tracking**: Monitor student performance and knowledge state across topics
- **RAG-Based Learning**: Retrieve relevant course content for personalized teaching
- **Career Recommendations**: AI-driven career path suggestions based on student mastery levels
- **JWT Authentication**: Secure student and instructor authentication
- **Full-Stack Integration**: FastAPI backend with Next.js frontend

## Tech Stack

### Backend

- **Framework**: FastAPI (Python 3.10+)
- **LLM**: Groq (via LangChain)
- **Vector Store**: ChromaDB with HuggingFace embeddings
- **Database**: SQLite
- **Auth**: JWT with bcrypt password hashing
- **PDF Processing**: PyPDF2 + LangChain text splitters

### Frontend

- **Framework**: Next.js 16+ (React 19)
- **Styling**: Tailwind CSS
- **HTTP Client**: Axios
- **Icons**: Lucide React

## Prerequisites

- **Python 3.10+** (3.11 recommended)
- **Node.js 18+** (Node 20 LTS recommended)
- **npm** (ships with Node)
- **Groq API Key** (get one at [console.groq.com](https://console.groq.com))

## Setup & Installation

### 1) Backend Setup (Windows)

Open a terminal at the project root.

```powershell
# Create virtual environment
cd backend
python -m venv venv
.\venv\Scripts\Activate.ps1
cd ..

# Install Python dependencies
pip install -r requirements.txt

# Create .env file
# backend/.env
cat > backend\.env << 'EOF'
JWT_SECRET="your-very-long-random-secret-key-here"
GROQ_API_KEY="your-groq-api-key-here"
EOF
```

**Notes:**

- `JWT_SECRET`: Use a long random string (at least 32 characters) for security
- `GROQ_API_KEY`: Required for LLM features (curriculum parsing, quiz generation, recommendations)

### 2) Backend Startup

```powershell
cd backend
uvicorn main:app --reload
```

Backend API available at: **http://127.0.0.1:8000**

- API Docs: http://127.0.0.1:8000/docs
- ReDoc: http://127.0.0.1:8000/redoc

### 3) Frontend Setup

Open a new terminal at the project root.

```bash
cd frontend
npm install
```

### 4) Frontend Startup

```bash
npm run dev
```

Frontend available at: **http://localhost:3000**

## Project Structure

```
.
├── backend/
│   ├── main.py                    # FastAPI app; auth, course, quiz endpoints
│   ├── ingest.py                  # PDF ingestion + RAG vector store logic
│   ├── database.py                # SQLite schema + helper functions
│   ├── curriculum_parser.py       # Curriculum parsing logic (LangChain prompts)
│   ├── test_setup.py              # Basic setup verification
│   ├── .env                       # Environment variables (create this)
│   ├── data/                      # Sample course PDFs for testing
│   ├── uploads/                   # User uploaded PDFs (auto-generated)
│   ├── db_chroma/                 # Vector store (auto-generated)
│   ├── student_data.db            # SQLite DB (auto-generated)
│   └── venv/                      # Python virtual environment (create this)
├── frontend/
│   ├── app/
│   │   ├── layout.tsx             # Root layout
│   │   ├── page.tsx               # Home page
│   │   ├── components/            # Reusable React components
│   │   ├── course/                # Course detail pages
│   │   ├── dashboard/             # Student dashboard
│   │   ├── lobby/                 # Course lobby
│   │   ├── login/ & register/     # Auth pages
│   │   └── lib/api.ts             # API client
│   ├── public/                    # Static assets
│   ├── package.json               # Node dependencies
│   ├── tsconfig.json              # TypeScript config
│   ├── next.config.ts             # Next.js config
│   └── node_modules/              # Dependencies (auto-generated)
├── requirements.txt               # Python dependencies
└── setup.md                       # Setup guide (this file's source)
```

## Key Endpoints

| Method | Endpoint                     | Description                           |
| ------ | ---------------------------- | ------------------------------------- |
| POST   | `/register`                  | Register new student                  |
| POST   | `/login`                     | Login and get JWT token               |
| POST   | `/course/create`             | Create a new course                   |
| POST   | `/course/{course_id}/upload` | Upload & ingest course PDFs           |
| POST   | `/course/{course_id}/quiz`   | Generate adaptive quiz                |
| GET    | `/course/{course_id}/topics` | Get course topics                     |
| GET    | `/student/mastery`           | Get student mastery levels            |
| POST   | `/chat`                      | Chat with AI tutor for course content |

## Environment Variables

Create `backend/.env`:

```env
# Required
JWT_SECRET="your-secret-key-here"
GROQ_API_KEY="your-groq-api-key-here"

# Optional (defaults shown)
# GROQ_MODEL="llama-3.1-8b-instant"
```

## Database Schema

### students

- `id` (PK)
- `username` (unique)
- `password_hash`
- `learning_style` (visual, auditory, kinesthetic)

### courses

- `id` (PK)
- `owner_id` (FK students)
- `name`
- `created_at`

### quiz_results

- `id` (PK)
- `student_id` (FK students)
- `topic`
- `score`
- `feedback`
- `timestamp`

### knowledge_state

- `id` (PK)
- `student_id` (FK students)
- `course_id` (FK courses)
- `topic`
- `mastery_level` (0.0–1.0)
- `last_updated`

### career_recommendations

- `id` (PK)
- `student_id` (FK students)
- `recommended_career`
- `confidence`
- `generated_at`

## Testing

### Backend Test

```bash
cd backend
python test_setup.py
```

Verifies dependencies, environment variables, and basic database connectivity.

## Build for Production

### Backend

```bash
# No build step; run directly with Uvicorn
uvicorn backend.main:app --host 0.0.0.0 --port 8000
```

### Frontend

```bash
cd frontend
npm run build
npm start
```

## Known Limitations & Considerations

- **Vector Store**: ChromaDB is persisted locally to `db_chroma/`. For multi-instance deployments, use a managed vector DB (e.g., Pinecone, Weaviate).
- **SQLite**: Suitable for development and small-scale deployments. Use PostgreSQL for production.
- **CORS**: Currently configured for localhost. Update `CORS_ORIGINS` in `main.py` for production domains.
- **Rate Limiting**: Not implemented; add rate limiting middleware for production.
- **File Upload Size**: Default FastAPI limit is 100MB. Adjust in `main.py` if needed.

## Troubleshooting

### Issue: `GROQ_API_KEY` not found

**Solution**: Ensure `backend/.env` exists and contains a valid Groq API key from https://console.groq.com

### Issue: ChromaDB persistence fails

**Solution**: Ensure `db_chroma/` folder has write permissions and adequate disk space.

### Issue: JWT token errors

**Solution**: Verify `JWT_SECRET` is set in `backend/.env` and is at least 32 characters long.

### Issue: CORS errors on frontend

**Solution**: Check that backend CORS middleware includes your frontend URL (default: `http://localhost:3000`).

## License

MIT

## Contact & Support

For questions or issues, please open a GitHub issue or reach out to the development team.
