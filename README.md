# Hospital AI Assistant — Complete Run Guide (Step by Step)

This guide will walk you through everything from scratch — installing Python, setting up the environment, installing dependencies, and running all three services. No technical knowledge required.

---

## Table of Contents

1. [Install Python](#1-install-python)
2. [Create a Python Virtual Environment](#2-create-a-python-virtual-environment)
3. [Activate the Virtual Environment](#3-activate-the-virtual-environment)
4. [Install Dependencies for All Services](#4-install-dependencies-for-all-services)
5. [Configure Environment Files (.env)](#5-configure-environment-files-env)
6. [Ingest Documents (One-Time Setup)](#6-ingest-documents-one-time-setup)
7. [Run the Application](#7-run-the-application)
8. [Verify Everything is Working](#8-verify-everything-is-working)
9. [Troubleshooting](#9-troubleshooting)

---

## 1. Install Python

This application requires Python 3.10 or newer.

### Step 1.1 — Check if Python is already installed

Open **Command Prompt** (press `Win + R`, type `cmd`, press Enter) and run:

```
python --version
```

If you see something like `Python 3.10.x` or `Python 3.11.x` or `Python 3.12.x`, you're good — skip to [Section 2](#2-create-a-python-virtual-environment).

If you see an error or a version below 3.10, follow Step 1.2 below.

### Step 1.2 — Install Python (if needed)

1. Go to **[python.org/downloads](https://python.org/downloads)**
2. Click the big yellow **"Download Python"** button (it will download the latest version)
3. Once downloaded, double-click the installer file
4. **IMPORTANT:** At the bottom of the installer window, check the box that says **"Add Python to PATH"**
5. Click **"Install Now"**
6. Wait for the installation to finish, then click **"Close"**

### Step 1.3 — Verify the installation

Close and reopen **Command Prompt**, then run:

```
python --version
```

You should now see the Python version number. If you do, great! 

---

## 2. Create a Python Virtual Environment

A virtual environment is like a clean, isolated workspace for your project. It keeps all the required libraries in one place without affecting anything else on your computer.

### Step 2.1 — Open Command Prompt in the project folder

1. Open **File Explorer** and navigate to your project folder: `C:\Users\MUNEER-IQBAL\Desktop\Hospital-Ai-Assistant`
2. In the address bar at the top, type `cmd` and press **Enter**
3. A black Command Prompt window will open, already pointing to your project folder

### Step 2.2 — Create the virtual environment

In the Command Prompt, type this command and press Enter:

```
python -m venv venv
```

This will take about 10–20 seconds. You'll see a new folder called `venv` appear in your project folder. This is your virtual environment.

**What just happened:** Python created a fresh, empty environment named `venv` inside your project folder. Think of it as a clean room where you'll install all the tools this project needs.

---

## 3. Activate the Virtual Environment

The virtual environment must be **activated** before you install anything or run the application. You need to do this every time you open a new Command Prompt to work on this project.

### Step 3.1 — Activate it

In the same Command Prompt (still in your project folder), run:

```
venv\Scripts\activate
```

If it works, you'll see `(venv)` appear at the beginning of your command line, like this:

```
(venv) C:\Users\MUNEER-IQBAL\Desktop\Hospital-Ai-Assistant>
```

That `(venv)` prefix means the environment is active and you're ready to go.

> **Every time you reopen Command Prompt**, you must:
> 1. Navigate to the project folder: `cd C:\Users\MUNEER-IQBAL\Desktop\Hospital-Ai-Assistant`
> 2. Activate the environment: `venv\Scripts\activate`

---

## 4. Install Dependencies for All Services

The project has three independent services, each with its own `requirements.txt` file listing the libraries it needs. You need to install the dependencies for each one.

Make sure your virtual environment is **active** (you see `(venv)` in the command line) before proceeding.

### Step 4.1 — Install MCP Server dependencies

```
cd mcp
pip install -r requirements.txt
cd ..
```

This installs everything the MCP (database) server needs. Wait for it to finish — you'll see progress bars and a "Successfully installed..." message at the end.

### Step 4.2 — Install Backend dependencies

```
cd backend
pip install -r requirements.txt
cd ..
```

This installs everything the Backend API and AI Agent need. This is the largest install and may take 1–2 minutes.

### Step 4.3 — Install Frontend dependencies

```
cd frontend
pip install -r requirements.txt
cd ..
```

This installs everything the Chat UI (frontend) needs.

### Step 4.4 — Verify all installations

You can check that everything installed correctly by running:

```
pip list
```

This shows you all the packages now available in your virtual environment. You should see packages like `fastapi`, `chainlit`, `openai`, `pinecone`, etc.

---

## 5. Configure Environment Files (.env)

Each service needs a `.env` file containing your API keys and configuration. These files tell each service how to connect to external services (OpenAI, Pinecone, MongoDB).

### Step 5.1 — Backend .env file

If `backend\.env` doesn't already exist, create it:

1. Open **Notepad**
2. Copy and paste the following, replacing the placeholder values with your actual keys:

```ini
OPENAI_API_KEY=sk-your-openai-api-key-here
MODEL=gpt-4.1
EMBEDDING_MODEL=text-embedding-3-large

PINECONE_API_KEY=pcsk-your-pinecone-api-key-here
PINECONE_ENVIRONMENT=us-east-1
PINECONE_INDEX_NAME=hospital-ai-index

MCP_SERVER_URL=http://localhost:8002

ENVIRONMENT=development
JWT_SECRET_KEY=your-secret-key-change-in-production
CORS_ORIGINS=["http://localhost:8000", "http://localhost:3000"]
MCP_API_KEY=mcp-shared-secret-change-in-production
```

3. Save as: Go to **File → Save As**
   - Navigate to `C:\Users\MUNEER-IQBAL\Desktop\Hospital-Ai-Assistant\backend\`
   - Set **Save as type** to **All Files (*.*)**
   - File name: `.env`
   - Click **Save**

### Step 5.2 — MCP Server .env file

Create `mcp\.env` with:

```ini
MONGO_URI=mongodb://your-mongodb-connection-string-here/
DEFAULT_DB_NAME=tenant_hospital
PORT=8002
MCP_API_KEY=mcp-shared-secret-change-in-production
```

> **Important:** The `MCP_API_KEY` value must be the **same** in both `backend\.env` and `mcp\.env`.

Save this file in `C:\Users\MUNEER-IQBAL\Desktop\Hospital-Ai-Assistant\mcp\`.

### Step 5.3 — Frontend .env file

Create `frontend\.env` with:

```ini
BACKEND_API_URL=http://localhost:8001
AUTH_USERNAME=doctor_001
AUTH_PASSWORD=doctor123
```

Save this file in `C:\Users\MUNEER-IQBAL\Desktop\Hospital-Ai-Assistant\frontend\`.

---

## 6. Ingest Documents (One-Time Setup)

Before the AI can answer questions about your hospital documents (SOPs, guidelines, etc.), you need to upload them to the vector database. This is a one-time process.

### Step 6.1 — Place your documents

1. Open File Explorer and go to `C:\Users\MUNEER-IQBAL\Desktop\Hospital-Ai-Assistant\backend\ingestion\data\`
2. Copy your PDF and/or DOCX files into this folder
3. The agent will be able to search and answer questions based on these documents

### Step 6.2 — Run the ingestion

Make sure your virtual environment is active, then:


## 7. Run the Application

Now you're ready to run the application. You need **three separate Command Prompt windows** — one for each service. They must be started in a specific order.

> **Before starting:** In each Command Prompt window, you must:
> 1. Navigate to the project folder: `cd C:\Users\MUNEER-IQBAL\Desktop\Hospital-Ai-Assistant`
> 2. Activate the virtual environment: `venv\Scripts\activate`

---

### Terminal 1 — Start the MCP Server (Database Layer)

The MCP server connects to your MongoDB database and provides 47 tools for querying live hospital data. **This must be started first** because the backend depends on it.

Open a **new Command Prompt** window, then:

```
cd C:\Users\MUNEER-IQBAL\Desktop\Hospital-Ai-Assistant
venv\Scripts\activate
cd mcp
python main.py
```

**What you should see:** The server will start and display something like:

```
INFO:     Started server process
INFO:     Uvicorn running on http://0.0.0.0:8002
MCP Server 'Hospital MCP Server' running on port 8002
```

**Leave this window open.** Do not close it — the server must keep running.

---

### Terminal 2 — Start the Backend API (AI & Business Logic)

The backend is the brain of the application. It connects to the MCP server, Pinecone (vector database), and OpenAI, and serves the API that the frontend talks to.

Open a **second new Command Prompt** window, then:

```
cd C:\Users\MUNEER-IQBAL\Desktop\Hospital-Ai-Assistant
venv\Scripts\activate
cd backend
python -m app.main
```

**What you should see:** The backend will start, connect to the MCP server and Pinecone, and show:

```
MCP Server connected: 47 tools available
RAG Engine ready. Index has N vectors.
Medical Agent (OpenAI Agents SDK) initialized with model: gpt-4.1
INFO:     Uvicorn running on http://0.0.0.0:8001
```

**Leave this window open.** Do not close it.

---

### Terminal 3 — Start the Frontend (Chat UI)

The frontend is the chat interface where users type questions and see AI responses.

Open a **third new Command Prompt** window, then:

```
cd C:\Users\MUNEER-IQBAL\Desktop\Hospital-Ai-Assistant
venv\Scripts\activate
cd frontend
chainlit run app.py --port 8000
```

**What you should see:**

```
Your app is available at http://localhost:8000
```

---

### You're Running!

Open your web browser and go to **http://localhost:8000**. You should see the Hospital AI Assistant chat interface. Type a question like "List all processes" and the AI will respond using live data from your database.

---

## 8. Verify Everything is Working

### Quick Verification Checklist

| Check | How to Verify |
|-------|---------------|
| MCP Server is running | Open http://localhost:8002/health in your browser — you should see a JSON response |
| Backend is running | Open http://localhost:8001/health in your browser — you should see a JSON health status |
| Backend API docs | Open http://localhost:8001/docs — you should see the Swagger API documentation |
| Frontend is running | Open http://localhost:8000 — you should see the chat interface |

### Test a Simple Chat

In the chat interface at http://localhost:8000:

1. Log in with the default credentials (if prompted):
   - **Username:** `doctor_001`
   - **Password:** `doctor123`
2. Type: `List all processes`
3. The AI should respond with a list of processes from your database

---

## 9. Troubleshooting

### "python is not recognized"
- Python is not installed or not added to PATH. Go back to [Section 1.2](#step-12--install-python-if-needed) and make sure you checked "Add Python to PATH" during installation.

### "pip is not recognized"
- This usually means Python was installed without pip. Reinstall Python and make sure the "pip" option is checked in the installer.

### "(venv) is not showing in the command line"
- The virtual environment is not activated. Run `venv\Scripts\activate` again from the project folder.

### "ModuleNotFoundError: No module named 'xyz'"
- A dependency is missing. Make sure you ran all three `pip install -r requirements.txt` commands in [Section 4](#4-install-dependencies-for-all-services).

### Backend fails to start with "Connection refused" to MCP
- The MCP Server (Terminal 1) must be started **before** the Backend (Terminal 2). If you see this error, make sure Terminal 1 is running and shows the MCP server is ready.

### Frontend shows "Failed to connect to backend"
- The Backend (Terminal 2) must be started **before** the Frontend (Terminal 3). Make sure Terminal 2 shows the backend is running on port 8001.

### Port already in use error
- Another program is using the port. You can either:
  - Close the program using that port
  - Change the port in the `.env` file and restart

### "MCP Server connected: 0 tools available"
- The backend connected to the MCP server but no tools were found. Check that:
  - The MCP server is fully started (wait a few seconds after starting Terminal 1)
  - The `MCP_API_KEY` in `backend\.env` and `mcp\.env` are **exactly the same**
  - MongoDB is accessible with the `MONGO_URI` in `mcp\.env`

### Pinecone connection fails
- Check that your `PINECONE_API_KEY` in `backend\.env` is correct
- Make sure the index name (`PINECONE_INDEX_NAME`) exists in your Pinecone account
- If you haven't ingested any documents yet, the RAG engine will still start but with 0 vectors — this is fine

---

## Quick Reference: Commands Cheat Sheet

Here are all the commands you need, summarized:

```bash
# ---- CREATE ENVIRONMENT (one-time) ----
cd C:\Users\MUNEER-IQBAL\Desktop\Hospital-Ai-Assistant
python -m venv venv

# ---- ACTIVATE ENVIRONMENT (every time) ----
venv\Scripts\activate

# ---- INSTALL DEPENDENCIES (one-time) ----
cd mcp
pip install -r requirements.txt
cd ..
cd backend
pip install -r requirements.txt
cd ..
cd frontend
pip install -r requirements.txt
cd ..

# ---- INGEST DOCUMENTS (one-time) ----
cd backend
python ingestion/ingest.py
cd ..

# ---- TERMINAL 1: MCP Server ----
cd C:\Users\MUNEER-IQBAL\Desktop\Hospital-Ai-Assistant
venv\Scripts\activate
cd mcp
python main.py

# ---- TERMINAL 2: Backend ----
cd C:\Users\MUNEER-IQBAL\Desktop\Hospital-Ai-Assistant
venv\Scripts\activate
cd backend
python -m app.main

# ---- TERMINAL 3: Frontend ----
cd C:\Users\MUNEER-IQBAL\Desktop\Hospital-Ai-Assistant
venv\Scripts\activate
cd frontend
chainlit run app.py -p 8000
```

---

## Stopping the Application

To stop any of the services, go to its Command Prompt window and press **Ctrl + C**. This safely shuts down the service.

To stop everything, press **Ctrl + C** in all three terminal windows.

To deactivate the virtual environment (optional), type:

```
deactivate
```

---

## Need Help?

contact us 
