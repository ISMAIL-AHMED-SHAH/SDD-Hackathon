

# 🗓️ Phase 1: The Setup (Do this TODAY)

Before writing code, we need the tools installed.

### 1. Install Basic Requirements

Node.js: Download and install the LTS version (v20+) from nodejs.org.

Python: Install Python 3.11 or 3.12.

VS Code: Your code editor.

Git: Make sure Git is installed and connected to your GitHub account.

### 2. Get Your API Keys (Save these in a notepad)

Gemini API Key (Free): Go to aistudio.google.com and get a free key. This allows you to use Claude Code for free.

OpenAI API Key: You will need this for the Chatbot (RAG) part.

Qdrant API Key & URL: Sign up for Qdrant Cloud (Free Tier) to store your book's data for the chatbot.

## 🛠️ Phase 2: Setup "Claude Code" with "Router" (The Free Method)

The hackathon rules allow using the Claude Code CLI but routed through a free model (Gemini) so you don't pay Anthropic fees.

#### 1. Install Claude Code Router
Open your terminal (Command Prompt/PowerShell) and run:

code
Bash
download
content_copy
expand_less
npm install -g @musistudio/claude-code-router

#### 2. Configure it for Gemini
Run this command to set your provider to Google Gemini:

code
Bash
download
content_copy
expand_less
claude-router config set provider google

Then set your API key:

code
Bash
download
content_copy
expand_less
claude-router config set google.apiKey "YOUR_GEMINI_API_KEY_HERE"

3. Install the Official Claude Code CLI
Now install the main tool:

code
Bash
download
content_copy
expand_less
npm install -g @anthropic-ai/claude-code

#### 4. Test it
Type claude in your terminal. It should start. Ask it "Hello, who are you?". If it responds using the Gemini model via the router, you are ready!

## 📚 Phase 3: Create the Book Framework (Docusaurus)

We need a blank book ready to go.

1. Create the Project
In your terminal, navigate to your desktop and run:

code
Bash
download
content_copy
expand_less
npx create-docusaurus@latest my-hackathon-book classic

(Select JavaScript if asked).

2. Clean it up

Open the folder my-hackathon-book in VS Code.

Go to docusaurus.config.js. Change the title to "My AI Book" (you will update this on Sunday).

Delete the contents of the docs folder (keep the folder, just remove the tutorial files inside).

3. Verify it runs
Run:

code
Bash
download
content_copy
expand_less
npm start

This opens a local website. If you see it, you are good.

## 🤖 Phase 4: The RAG Chatbot Skeleton (The Hard Part)

This is where you need to integrate OpenAI ChatKit and FastAPI. Do not wait for Sunday to build this. Build the "empty" version now.

1. Create a Backend Folder
Inside your main folder, create a folder named backend.

2. Setup Python Environment
Inside backend, open a terminal and run:

code
Bash
download
content_copy
expand_less
python -m venv venv
### Activate it:
### Windows: venv\Scripts\activate
### Mac: source venv/bin/activate

3. Install Libraries
Create a requirements.txt file in backend with:

code
Text
download
content_copy
expand_less
fastapi
uvicorn
openai
qdrant-client
pydantic
python-dotenv

Run pip install -r requirements.txt.

4. Create the API Skeleton (main.py)
Create a file backend/main.py. This is where the logic goes to:

Receive a question from the user.

Search Qdrant for relevant book text.

Send text + question to OpenAI.

Return answer.

(Note: You can use Claude Code to write this script for you! Command: claude "Write a FastAPI app that connects to Qdrant and OpenAI ChatKit for RAG". Save this code.)

## 🧠 Phase 5: The "Spec-Kit" Strategy (Your Secret Weapon)

The hackathon is "Spec-Driven." This means you shouldn't write the book manually. You should write a Specification File that tells the AI to write the book.

Create a file named book-spec.md in your project root.
Copy this template into it:

code
Markdown
download
content_copy
expand_less
# Book Generation Specification

## Project Overview
**Book Title:** [TO BE UPDATED SUNDAY]
**Target Audience:** Beginners to Intermediate learners.
**Tone:** Professional, engaging, and easy to understand.

## Content Requirements
1.  **Structure:** The book must have 5 chapters.
2.  **Format:** All files must be written in Markdown (.md) inside the /docs folder.
3.  **Frontmatter:** Each file must include Docusaurus frontmatter (id, title, sidebar_label).

## Sub-Agent Instructions (Bonus)
*   **Researcher Agent:** Search the web for the latest facts about the [Title].
*   **Writer Agent:** Draft the content based on research.
*   **Editor Agent:** Review for grammar and Docusaurus compatibility.

## Execution Steps
1.  Generate the Table of Contents in `sidebars.js`.
2.  Write Chapter 1.
3.  Write Chapter 2...

Why do this?
On Sunday at 9:00 AM, when they announce the title (e.g., "The History of AI"), you simply open this file, paste the title in [TO BE UPDATED SUNDAY], and tell Claude Code:

"Read book-spec.md and execute the instructions to generate the book."

## 🚀 Phase 6: Sunday Game Plan (Nov 30th)

9:00 AM - The Announcement

Receive the Title on WhatsApp.

Update docusaurus.config.js with the new Title.

Update book-spec.md with the new Title.

9:15 AM - Generation

Open terminal. Run claude.

Command: "Read book-spec.md. Act as a technical author. Create the Table of Contents in sidebars.js and generate the markdown files for the docs folder based on the spec."

Watch it write the book for you.

11:00 AM - RAG Integration

Once the book text is generated, you need to "feed" it to Qdrant.

Ask Claude: "Write a script to read all .md files in /docs and upload them to my Qdrant vector database." Run that script.

2:00 PM - Final Polish & Deploy

Test the Chatbot on your local site.

Deploy to GitHub:

Create a repo hackathon-project.

git init, git add ., git commit, git push.

Deploy to GitHub Pages:

Run: npm run deploy (Ensure your docusaurus.config.js has the correct GitHub settings).

5:00 PM - Submit
Fill out the Google Form link provided in your instructions.

💡 Tips for Bonus Marks (Matrix/Subagents)

The instructions mention "Reusable Intelligence."
Create a folder called /skills. Inside, create text files like write_quiz.md:

"Skill: Whenever asked to write a quiz, create 5 multiple choice questions with answers at the bottom."

In your main prompt, say: "Load the skill from /skills/write_quiz.md and add a quiz to the end of every chapter." This shows you are using modular sub-agent skills!

Summary Checklist for NOW:

Install Node, Python, Git.

Get claude-code working with the free Gemini Router.

Create the Docusaurus folder.

Write the book-spec.md template.

Relax and wait for Sunday!
