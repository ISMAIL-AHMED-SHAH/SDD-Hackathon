

## 📂 Step 1: Directory Setup

Create a folder named .spec-kit in the root of your project folder. We will place all your "brains" here.

code
Text
download
content_copy
expand_less
/my-hackathon-project
```

  ├── .spec-kit/
  │   ├── 01_constitution.md
  │   ├── 02_specification.md
  │   ├── 03_skills_matrix.md
  │   └── 04_plan_template.md
  └── (Project files will be created here later)
```


## 📝 Step 2: Create the Files (Copy & Paste)

Create these files inside the .spec-kit folder.

File 1: .spec-kit/01_constitution.md

Purpose: Sets the non-negotiable laws for the AI.

code
Markdown
download
content_copy
expand_less
# 📜 PROJECT CONSTITUTION (NON-NEGOTIABLE RULES)

## 1. Zero-Placeholder Policy
*   NEVER leave comments like `# logic goes here` or `<!-- content to be added -->`.
*   ALL code must be fully implemented, functional, and production-ready.
*   If a text section is required, write full, high-quality English paragraphs.

## 2. Tech Stack Mandates
*   **Frontend:** Docusaurus 3.x (React/TypeScript/Markdown).
*   **Backend:** Python 3.12, FastAPI, Uvicorn.
*   **AI/RAG:** OpenAI ChatKit (or SDK), Qdrant Cloud (Vector DB).
*   **Styling:** Custom CSS inside Docusaurus `custom.css`.

## 3. Operational Integrity
*   **Security:** NEVER hardcode API keys. Use `.env` files and `python-dotenv`.
*   **Error Handling:** All Python endpoints must have `try/except` blocks.
*   **Git:** Commit messages must be descriptive (e.g., "feat: added RAG endpoint").

## 4. The "Agentic" Behavior
*   You are not just a coder; you are the **Lead Engineer**.
*   If a file is missing, create it.
*   If a package is missing, install it (or update `requirements.txt`).
*   Do not ask for permission for minor fixes; just fix them and report.
File 2: .spec-kit/02_specification.md

Purpose: Describes what we are building. (Note the Placeholder for the Title).

code
Markdown
download
content_copy
expand_less
# 🏗️ PROJECT SPECIFICATION (THE BLUEPRINT)

## 1. Project Identity
**Project Name:** AI-Generated Book & Intelligent RAG Agent
**Book Title:** [INSERT_TITLE_HERE_ON_SUNDAY]
**Goal:** A static documentation site hosting a full book, featuring an embedded AI chatbot that answers questions based *only* on the book's content.

## 2. Frontend Requirements (Docusaurus)
*   **Theme:** "Classic" preset. Clean, minimalist, professional.
*   **Navigation:** Top bar must have "Book", "Chat with AI", "GitHub".
*   **Chat Interface:** A floating chat widget (bottom-right) or a dedicated "Chat" page.
    *   User inputs text.
    *   App sends request to Backend API.
    *   App displays streaming response.

## 3. Backend Requirements (FastAPI)
*   **Endpoint:** `POST /api/chat`
*   **Input:** `{ query: string, context_selection: string (optional) }`
*   **Logic (RAG Pipeline):**
    1.  Receive query.
    2.  Embed query using OpenAI `text-embedding-3-small`.
    3.  Search Qdrant Vector DB for top 3 matching book chunks.
    4.  Construct Prompt: "Answer using ONLY this context: {chunks}..."
    5.  Return answer.
*   **Ingestion Script:** A standalone script (`ingest.py`) that reads all Markdown files in `/docs`, chunks them, and uploads to Qdrant.

## 4. Content Strategy
*   **Book Structure:** 5 Chapters minimum.
*   **Style:** Educational, witty, using analogies (Matrix style).
*   **Bonus:** Each chapter must end with a "Key Takeaways" section.
File 3: .spec-kit/03_skills_matrix.md

Purpose: The "Reusable Intelligence" (Bonus Marks).

code
Markdown
download
content_copy
expand_less
# 🧠 SKILLS MATRIX (SUB-AGENTS)

## Skill: `load_helicopter` (Concept)
**Trigger:** When asked to "Explain complex topic".
**Action:** Use an analogy involving simulation, training, or Matrix concepts.

## Skill: `polisher_agent`
**Trigger:** Before saving any Markdown file.
**Action:**
1.  Check for spelling.
2.  Ensure Docusaurus Frontmatter (`id`, `title`) exists.
3.  Add emojis to headers to make it engaging.

## Skill: `debug_protocol`
**Trigger:** If a command fails.
**Action:**
1.  Read the error log.
2.  Search for the error pattern.
3.  Apply fix.
4.  Retry MAX 3 times before asking human.
🎮 Step 3: The Command Structure (How to run it)

On Sunday, you won't be typing random prompts. You will use Claude Code to "boot up" your Spec-Kit.

Here is the exact workflow to execute in your terminal:

1. The Initialization Command (Run this FIRST)

Copy-paste this entire block into Claude Code:

code
Text
download
content_copy
expand_less
/init
I am initializing the Spec-Kit Plus protocol. 
1. Read `.spec-kit/01_constitution.md`. These are your LAWS.
2. Read `.spec-kit/02_specification.md`. This is your MISSION.
3. Read `.spec-kit/03_skills_matrix.md`. These are your CAPABILITIES.

Confirm you have read and understood these files by saying: "System Online. Awaiting Book Title."
2. The /specify Command (Run at 9:00 AM Sunday)

Once you get the title (e.g., "The Future of Mars"), run this:

code
Text
download
content_copy
expand_less
/specify
The official Book Title is: "The Future of Mars".
Update the Specification file with this title.
Now, generate a file named `.spec-kit/04_plan.md`.
The plan must break down the project into 4 Phases:
1. Environment Setup (Docusaurus + Python venv)
2. Content Generation (Writing the book files)
3. Backend RAG Implementation (FastAPI + Qdrant)
4. Integration & UI (Connecting Chatbot to Frontend)
Display the plan for my approval.
3. The /implement Command (The Loop)

Once you say "Yes" to the plan, you run this repeatedly:

code
Text
download
content_copy
expand_less
/implement Phase 1
Execute Phase 1 of the plan. 
- Install dependencies.
- Create the folder structure.
- Verify the server starts.
Run the code, don't just write it.

(When Phase 1 is done)

code
Text
download
content_copy
expand_less
/implement Phase 2
Execute Phase 2. 
- Use your "Writer Agent" capability.
- Generate the Table of Contents.
- Write Chapter 1 through 5 in Markdown.
- Ensure Frontmatter is correct.

(When Phase 2 is done)

code
Text
download
content_copy
expand_less
/implement Phase 3
Execute Phase 3.
- Create `backend/main.py`.
- Create `backend/ingest.py`.
- Ask me for my OpenAI and Qdrant Keys (I will paste them in .env).
## 🛡️ Pre-Hackathon Checklist (Do this NOW)

To make sure this works on Sunday, do this tonight:

Create the .spec-kit folder and the 3 files above.

Test the flow:

Open claude.

Run the Initialization Command (Step 3.1).

Run a fake /specify with a fake title (e.g., "Pizza Making 101").

See if Claude creates a good Plan.

Prepare your keys: Have OPENAI_API_KEY and QDRANT_URL/QDRANT_API_KEY ready in a text file.

This setup forces the AI to follow your Constitution, ensuring you don't get half-baked code or lazy placeholders. You are effectively the "Architect" and Claude is the "Builder."
