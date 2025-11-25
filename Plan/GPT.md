
# Plan (what you must deliver)

1. A **book** (static site) built with **Docusaurus** and published to **GitHub Pages**. (Required) ([docusaurus.io][2])
2. An **embedded RAG chatbot** inside the published book site that answers questions about the book (and can answer using text the user selects). Use a small backend (FastAPI) + vector DB (Qdrant Cloud free tier) + embeddings + Chat/Agent SDK (OpenAI ChatKit or any compatible SDK). (Required) ([Qdrant][3])
3. **Bonus**: Claude Code Subagents + Agent Skills (reusable intelligence modules) — include at least one Skill/subagent that performs a reusable task in the book or chatbot (e.g., summarizer, citation extractor, chapter generator). Use Spec-Kit Plus to manage specs & agents. ([GitHub][1])

---

# Immediate tasks (do these RIGHT NOW — essential prep)

1. **Accounts & basic tools**

   * Create a **GitHub** account / repo for your project.
   * Create a **Qdrant Cloud** account and create a free cluster (no CC). Get API key & cluster URL. (Qdrant free clusters available: 1GB free tier). ([Qdrant][3])
   * Create/open an **OpenAI** account or the provider you’ll use for embeddings / ChatKit (or plan to use free models via Claude Code Router). If you use OpenAI ChatKit you’ll need API keys. ([OpenAI Platform][4])
   * Install Git, Node (18+), pnpm (or npm/yarn), Python 3.10+.

2. **Clone / star these repos (locally)**

   * `musistudio/claude-code-router` (routes Claude Code to free/other LLMs). ([GitHub][1])
   * `panaversity/spec-kit-plus` (Spec-Kit Plus templates & CLI). ([GitHub][5])

3. **Create a GitHub repo and a branch for the Hackathon** (e.g., `book-rag-hackathon`).

---

# Step-by-step implementation (copy + paste ready)

## Phase A — Scaffold the book (Docusaurus) — Local step

1. Create a new Docusaurus site:

```bash
# using pnpm recommended by Docusaurus
pnpm create docusaurus@latest my-book classic
cd my-book
pnpm install
```

2. Edit `docusaurus.config.js` — set `title`, `tagline`, `url`, `baseUrl` etc. Add an obvious place in the navbar for the chatbot (e.g., `"/chat"`). See Docusaurus docs for deployment configuration. ([docusaurus.io][2])

3. Add your book content under `docs/` (use headings for chapters). Keep chapters short (RAG works best on smaller chunks).

4. Run locally:

```bash
pnpm start
```

5. When ready to publish to GitHub Pages (deployment):

```bash
# configure homepage in package.json or docusaurus.config
pnpm build
pnpm run deploy
```

Docusaurus provides a `docusaurus deploy` helper; you can also use GitHub Actions. ([docusaurus.io][6])

---

## Phase B — Prepare the spec + Spec-Kit Plus (spec-driven approach)

1. Scaffold Spec-Kit Plus template (pick a template that fits “book + rag”):

   * Clone `panaversity/spec-kit-plus`, pick the template zip for your agent (Claude/OpenAI).
2. Write a short *spec* describing:

   * Book title, chapters, chapter length, style.
   * Chatbot features: answer from whole book; answer from user-selected text only; cite paragraph/chapter; fallback: “I don’t know”.
   * Non-functional: responses must reference the chapter and exact paragraph used.
3. Put the spec in the repo root (`spec/`), so your Claude Code subagents can read it and generate code to that spec. This structure will help judges see you used Spec-Kit Plus. ([AI Native Software Development][7])

---

## Phase C — Ingest the book to the vector DB (pipeline)

Goal: split your docs into chunks, embed them, upsert into Qdrant.

**Dependencies** (Python):

```bash
python -m venv .venv
source .venv/bin/activate
pip install qdrant-client sentence-transformers fastapi uvicorn python-multipart
# If using OpenAI embeddings:
pip install openai
```

**Minimal ingestion script**

```python
# ingest.py
from qdrant_client import QdrantClient
from sentence_transformers import SentenceTransformer
import glob, os, json

# config
QDRANT_URL = "https://<your-cluster>.qdrant.cloud"
QDRANT_API_KEY = "YOUR_QDRANT_API_KEY"
COLLECTION_NAME = "book_collection"

client = QdrantClient(url=QDRANT_URL, api_key=QDRANT_API_KEY)
embed_model = SentenceTransformer('all-MiniLM-L6-v2')  # small, fast

# read docs/ (md files)
chunks = []
for md in glob.glob("docs/**/*.md", recursive=True):
    text = open(md, encoding="utf-8").read()
    # naive splitter by paragraphs — you can use tiktoken / token-based split if needed
    for i, para in enumerate([p.strip() for p in text.split("\n\n") if p.strip()]):
        chunks.append({
            "id": f"{os.path.basename(md)}_p{i}",
            "text": para,
            "meta": {"source": md, "para_index": i}
        })

# compute embeddings
texts = [c["text"] for c in chunks]
embeddings = embed_model.encode(texts, show_progress_bar=True).tolist()

# create collection if not exists
client.recreate_collection(
    collection_name=COLLECTION_NAME,
    vector_size=len(embeddings[0]),
    distance="Cosine"
)

# upsert
points = []
for c, emb in zip(chunks, embeddings):
    points.append({"id": c["id"], "vector": emb, "payload": c["meta"]})
client.upsert(collection_name=COLLECTION_NAME, points=points)
print("Ingested", len(points))
```

Notes:

* You can replace `sentence-transformers` with cloud embeddings (OpenAI/Gemini) to match ChatKit usage; pick an approach that fits your API limits. Many tutorials use `sentence-transformers` locally for free. ([Qdrant][8])

---

## Phase D — Simple FastAPI RAG backend (serve retrieval & QA)

This backend will:

* accept user query,
* retrieve top-k vectors from Qdrant,
* assemble context,
* call Chat/LLM (or ChatKit/Agent) to generate answer referencing retrieved text.

**Minimal FastAPI example**

```python
# app.py
from fastapi import FastAPI
from pydantic import BaseModel
from qdrant_client import QdrantClient
from sentence_transformers import SentenceTransformer
import os
import openai  # if using OpenAI as the LLM

app = FastAPI()
q = QdrantClient(url=os.getenv("QDRANT_URL"), api_key=os.getenv("QDRANT_API_KEY"))
embed = SentenceTransformer('all-MiniLM-L6-v2')

COL = "book_collection"

class Query(BaseModel):
    question: str
    use_selected_text: bool = False
    selected_text: str | None = None

@app.post("/query")
async def query(qry: Query):
    # if user selected text, prefer that as context
    if qry.use_selected_text and qry.selected_text:
        context = qry.selected_text
    else:
        q_emb = embed.encode(qry.question).tolist()
        hits = q.search(collection_name=COL, query_vector=q_emb, limit=5)
        context = "\n\n".join([h.payload.get("text", "") for h in hits])
    # Call LLM / ChatKit to answer — below is a placeholder using OpenAI
    prompt = f"Use only the context below to answer the question. If the answer is not found, say 'I don't know'.\n\nCONTEXT:\n{context}\n\nQUESTION:\n{qry.question}"
    resp = openai.ChatCompletion.create(
        model="gpt-4o-mini", messages=[{"role":"user","content":prompt}]
    )
    return {"answer": resp.choices[0].message.content}
```

Important: for production or judged demo, ensure the LLM is constrained to only use the retrieved context (explicit system prompt + instruction to refuse if not found).

For better reliability and to satisfy “answer based only on user-selected text” requirement, design an API endpoint that, when `use_selected_text` is true, calls LLM with only that text as context. The site frontend can send the selected text to the backend.

(There are many example RAG tutorials integrating FastAPI + Qdrant; see Qdrant docs and examples.) ([Qdrant][9])

---

## Phase E — Embedding the chatbot into the Docusaurus site

Options:

1. **Simple Web Chat UI** (JS frontend that calls your FastAPI `/query` endpoint). Add a `/chat` page in Docusaurus that hosts a React component that:

   * sends user queries to the backend,
   * displays answers and the source paragraphs,
   * includes a “Use selected text” button: when the user highlights text on a doc page, the selection is sent as `selected_text` to the backend.

2. **OpenAI ChatKit (recommended if using OpenAI)** — OpenAI ChatKit helps build web chat experiences and can integrate with your RAG backend as the tool/knowledge retriever. See ChatKit docs for embedding approaches. ([OpenAI Platform][4])

**Basic front-end flow (pseudocode)**:

* On Docusaurus doc page, add a small floating action button “Ask about this page”.
* When clicked: capture `window.getSelection()` → send to `/query` with `use_selected_text=true`.
* Show answer in the chat drawer with the paragraph ID link (so user can open that exact paragraph).

---

## Phase F — Claude Code Router + Subagents + Spec-Kit Plus (bonus marks)

1. Install and configure `claude-code-router` locally. Use it to route parts of your agent pipeline to free models or to glue logic between subagents. The repo has examples and an install README. ([GitHub][1])
2. **Create at least one Subagent / Skill**:

   * Example Skill: `SummarizerSkill` — given a selected paragraph, returns a 2-line summary + keywords.
   * Example Subagent flow: main agent receives user query → if query requires a “rewrite” or “summarize” it delegates to SummarizerSkill; otherwise it goes to Retriever → QA.
3. Implement Agent Skills in Claude Code (or in your own orchestration layer) and show in your spec that they are reusable for other books.

Docs for subagents & skills: Claude Code docs (Subagents, Skills). ([Claude Code][10])

---

## Phase G — Testing, demo checklist & packaging

Before submitting:

* [ ] `pnpm build` → ensure site builds.
* [ ] `docusaurus deploy` or GH Actions → check GitHub Pages live URL.
* [ ] Run FastAPI locally and test `/query` for several questions (edge cases like “not in book”).
* [ ] Test “highlight text → ask” flow and verify the backend uses only that text.
* [ ] Record a short demo screencast (30–90s) showing: open book → highlight a paragraph → ask a question → chatbot answers and shows source.
* [ ] Push code to GitHub and submit the form: `https://forms.gle/CQsSEGM3GeCrL43c8` (the form in your brief).
* [ ] Be in Zoom at **Nov 30, 2025 06:00 PM** with your repo link ready. (You already have the meeting link in the brief.)

---

# Minimal file structure I recommend

```
book-rag-hackathon/
├─ docusaurus/ (Docusaurus site)
│  ├─ docs/
│  ├─ src/
│  └─ docusaurus.config.js
├─ backend/
│  ├─ ingest.py
│  ├─ app.py   # FastAPI
│  └─ requirements.txt
├─ spec/      # Spec-Kit Plus + your spec files
├─ agents/    # claude code router configs, subagents, skills
└─ README.md  # instructions + demo link + live site URL
```

---

# What I’d do if I were on the clock (recommended priority)

1. **Today**: create GitHub repo, Docusaurus scaffold, Qdrant account & collection, basic docs content (empty chapters ready). (Prep work)
2. **Next**: implement ingestion script & run to populate Qdrant.
3. **Then**: implement FastAPI `/query` with simple embed + search + LLM call. Test.
4. **Next**: integrate chat UI in Docusaurus (simple Fetch to backend).
5. **Polish**: Claude subagent / Skill demonstration and include spec-driven artifacts in repo.
6. **Deploy**: Docusaurus to GitHub Pages and verify Live URL.
7. **Submit**: Google form + Zoom demo.

---

# Helpful links (quick)

* Claude Code router repo (install & examples). ([GitHub][1])
* Spec-Kit Plus repo / templates. ([GitHub][5])
* Docusaurus (getting started + deployment). ([docusaurus.io][2])
* Qdrant Cloud (free tier + create cluster). ([Qdrant][3])
* OpenAI ChatKit docs & chatkit-python. ([OpenAI Platform][4])

---

# Risks & judgeable points (what they’ll look at)

* Does the chatbot **actually** answer based on selected text and cite the source paragraph? (very important)
* Is the site live and accessible on GitHub Pages? (required)
* Are Spec artifacts present (spec + reasoning + test cases)? (Spec-Kit Plus usage)
* Bonus: are Subagents/Skills implemented and reusable? (Claude Code subagents)
* UX polish: small features like “show source paragraph link”, “confidence/metadata”, and “fallback ‘I don’t know’” earn points.

---


[1]: https://github.com/musistudio/claude-code-router "GitHub - musistudio/claude-code-router: Use Claude Code as the foundation for coding infrastructure, allowing you to decide how to interact with the model while enjoying updates from Anthropic."
[2]: https://docusaurus.io/ "Build optimized websites quickly, focus on your content | Docusaurus"
[3]: https://qdrant.tech/documentation/cloud/create-cluster/?utm_source=chatgpt.com "Creating a Qdrant Cloud Cluster"
[4]: https://platform.openai.com/docs/guides/custom-chatkit "OpenAI Platform"
[5]: https://github.com/panaversity/spec-kit-plus?utm_source=chatgpt.com "panaversity/spec-kit-plus"
[6]: https://docusaurus.io/docs/deployment?utm_source=chatgpt.com "Deployment"
[7]: https://ai-native.panaversity.org/docs/Spec-Driven-Development/spec-kit-plus-hands-on/spec-kit-plus-foundation?utm_source=chatgpt.com "Spec-Kit Plus Foundation: What You're About to Build With"
[8]: https://qdrant.tech/documentation/beginner-tutorials/neural-search/?utm_source=chatgpt.com "Build a Neural Search Service"
[9]: https://qdrant.tech/documentation/examples/?utm_source=chatgpt.com "Examples"
[10]: https://code.claude.com/docs/en/overview "Claude Code overview - Claude Code Docs"
