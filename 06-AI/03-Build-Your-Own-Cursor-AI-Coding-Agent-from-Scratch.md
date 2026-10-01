# ⚡ AI Day 3 — Build Your Own Cursor: Autonomous AI Coding Agent & Terminal Engine from Scratch
> **Video Resource**: [Build Your Own Cursor | GenAI Full Course #5](https://youtu.be/asJseAWRl0M) | **Instructor**: Rohit Negi | **Series**: AI & GenAI Mastery

---

## 🗂️ Executive Architecture Overview

```
                      ┌────────────────────────────────────────────────────────┐
                      │              THE ANATOMY OF CURSOR / DEVIN             │
                      ├────────────────────────────────────────────────────────┤
                      │  Traditional Chatbot:  User Prompt ➔ Code in Markdown  │
                      │  Autonomous Cursor:    User Prompt ➔ Creates Files ➔   │
                      │                        Runs Tests ➔ Fixes Errors ➔     │
                      │                        Delivers Working Software 🚀    │
                      └────────────────────────────────────────────────────────┘
```

```
                                  THE CURSOR AGENTIC LOOP:

                                ┌───────────────────────────┐
                                │      Developer Goal       │
                                │ "Build a FastAPI server & │
                                │  verify /health endpoint" │
                                └─────────────┬─────────────┘
                                              │
                                              ▼
                                ┌───────────────────────────┐
                                │     LLM REASONING CORE    │
                                │ (Selects next tool action)│
                                └──────┬─────────────▲──────┘
                                       │             │
                    Tool Calls (JSON)  │             │ Observation (stdout / stderr)
                                       ▼             │
    ┌────────────────────────────────────────────────┴─────────────────────────────────┐
    │                              LOCAL OPERATING SYSTEM / TOOLS                      │
    │  ┌───────────────────────┬────────────────────────┬────────────────────────────┐ │
    │  │ write_file("main.py") │ execute_command("uv..")│ read_file("requirements") │ │
    │  └───────────────────────┴────────────────────────┴────────────────────────────┘ │
    └──────────────────────────────────────────────────────────────────────────────────┘
```

---

## 🗂️ Topics Covered

| # | Topic | Key Concepts |
|---|-------|--------------|
| 1 | **What Makes Cursor Magic? (Chat vs Agent)** | Passive snippet generation vs Active OS & Terminal execution |
| 2 | **The 4 Essential Agent Tools** | `execute_command`, `write_file`, `read_file`, `list_directory` |
| 3 | **The Self-Healing Autonomous Loop** | Capturing `stdout` / `stderr` and enabling model self-correction |
| 4 | **Production Python Code Implementation** | `subprocess.run`, streaming logs, and Google GenAI SDK integration |
| 5 | **Security & Sandboxing Architecture** | Safe command whitelisting, preventing accidental destructive commands |
| 6 | **Live Walkthrough: Building a Real Web App** | Agent builds, tests, detects missing deps, installs, and runs |
| 7 | **Production System Design Cheatsheet** | High-level architecture patterns for coding agents |

---

## 1️⃣ What Makes Cursor Magic? (Chat vs Autonomous Coding Agent)

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    TRADITIONAL CHATBOT vs AI CODING AGENT                   │
├──────────────────────────┬──────────────────────┬───────────────────────────┤
│ Metric                   │ 💬 Regular ChatGPT   │ ⚡ Cursor / Coding Agent  │
├──────────────────────────┼──────────────────────┼───────────────────────────┤
│ **Input**                │ Human prompt         │ Human prompt + Project env│
│ **Output**               │ Markdown text snippet│ Actual files created on OS│
│ **Testing Capability**   │ ❌ Cannot run code   │ ✅ Executes in terminal   │
│ **Error Handling**       │ User must copy-paste │ ✅ Auto-reads stderr & fix│
│ **Filesystem Access**    │ ❌ Blind to files    │ ✅ Full workspace vision  │
└──────────────────────────┴──────────────────────┴───────────────────────────┘
```

> 💡 **Analogy**:
> - **Regular LLM**: Ek software architect jo sirf paper pe drawing bana kar deta hai, par cement ya bricks ko haath nahi lagata.
> - **Cursor AI Agent**: Ek full-stack developer jo code likhta bhi hai, terminal me run karta hai, error aane par debug karke fix bhi karta hai!

---

## 2️⃣ The 4 Essential Agent Tools

Ek AI model computer ko tabhi operate kar sakta hai jab use **OS System Calls** ke tools diye jayein:

```
                          AGENT TOOL SUITE:
                          
  1. execute_command(cmd) ──► Runs shell command, captures stdout/stderr & exit code.
  2. write_file(path, content) ──► Creates new files or updates existing code.
  3. read_file(path) ──► Reads codebase files to understand context & imports.
  4. list_directory(path) ──► Explores the folder tree to discover project layout.
```

---

## 3️⃣ The Self-Healing Autonomous Loop (How Cursor Fixes Its Own Bugs)

```
  Step 1: User says "Create a Node.js server that listens on port 3000."
  Step 2: Agent calls write_file("server.js", "const express = require('express')...")
  Step 3: Agent calls execute_command("node server.js")
  
  💥 RUNTIME CRASH:
     stderr: "Error: Cannot find module 'express'"
     exit_code: 1
     
  Step 4: Observation fed back to LLM!
  Step 5: LLM Reason: "Express is not installed in the workspace. I must run npm install."
  Step 6: Agent calls execute_command("npm install express")
  Step 7: Agent calls execute_command("node server.js") ──► Server running on 3000! ✅
```

---

## 4️⃣ Production Python Implementation — Build Your Own Cursor

> 🚀 Ab hum ek complete production-grade **CLI Cursor Agent** banayenge jo tumhare computer ke terminal pe commands run karega aur files create karega!

### 📦 Step 1: Install Dependencies & Setup `.env`
```bash
pip install google-genai python-dotenv
```

```env
GEMINI_API_KEY=your_gemini_api_key_here
```

---

### 🛠️ Step 2: Define System Tools (`subprocess` & Filesystem)

```python
import os
import subprocess
import json
from google import genai
from google.genai import types
from dotenv import load_dotenv

load_dotenv()
client = genai.Client(api_key=os.getenv("GEMINI_API_KEY"))

# =========================================================
# 🛠️ TOOL 1: EXECUTE TERMINAL COMMAND (The Engine of Cursor)
# =========================================================
def execute_command(command: str) -> str:
    """Executes any shell command on the host operating system.
    Captures stdout, stderr, and returncode.
    
    Args:
        command: The terminal command to execute (e.g. 'npm init -y', 'python app.py', 'ls -la', 'dir').
    """
    print(f"\n⚙️ [TERMINAL EXEC] > {command}")
    
    # Block dangerous destructive commands
    dangerous_patterns = ["rm -rf /", "mkfs", ":(){ :|:& };:", "format c:"]
    for pattern in dangerous_patterns:
        if pattern in command.lower():
            return json.dumps({"error": f"Command '{command}' blocked for safety reasons.", "exit_code": 1})
            
    try:
        # Run command in current workspace directory with timeout
        result = subprocess.run(
            command,
            shell=True,
            capture_output=True,
            text=True,
            timeout=30 # 30 seconds timeout to prevent hanging servers
        )
        
        output = {
            "stdout": result.stdout.strip(),
            "stderr": result.stderr.strip(),
            "exit_code": result.returncode,
            "status": "success" if result.returncode == 0 else "error"
        }
        
        if result.stdout:
            print(f"📄 [STDOUT]:\n{result.stdout.strip()[:300]}")
        if result.stderr:
            print(f"⚠️ [STDERR]:\n{result.stderr.strip()[:300]}")
            
        return json.dumps(output)
        
    except subprocess.TimeoutExpired:
        return json.dumps({"error": "Command timed out after 30 seconds", "exit_code": 124})
    except Exception as e:
        return json.dumps({"error": str(e), "exit_code": 1})


# =========================================================
# 🛠️ TOOL 2: WRITE / CREATE FILE
# =========================================================
def write_file(filepath: str, content: str) -> str:
    """Creates a new file or overwrites an existing file with the provided code content.
    
    Args:
        filepath: Relative or absolute path to the file (e.g. 'app.py', 'src/index.js').
        content: The complete raw code/text content to write.
    """
    print(f"\n📝 [FILE WRITE] > {filepath}")
    try:
        # Ensure parent directories exist
        os.makedirs(os.path.dirname(filepath), exist_ok=True) if os.path.dirname(filepath) else None
        
        with open(filepath, "w", encoding="utf-8") as f:
            f.write(content)
            
        return json.dumps({"status": "success", "message": f"File '{filepath}' written successfully ({len(content)} bytes)."})
    except Exception as e:
        return json.dumps({"status": "error", "error": str(e)})


# =========================================================
# 🛠️ TOOL 3: READ FILE
# =========================================================
def read_file(filepath: str) -> str:
    """Reads and returns the contents of a file from disk.
    
    Args:
        filepath: Path to the file to read.
    """
    print(f"\n🔍 [FILE READ] > {filepath}")
    try:
        if not os.path.exists(filepath):
            return json.dumps({"status": "error", "error": f"File '{filepath}' does not exist."})
            
        with open(filepath, "r", encoding="utf-8") as f:
            content = f.read()
            
        return json.dumps({"status": "success", "content": content})
    except Exception as e:
        return json.dumps({"status": "error", "error": str(e)})


# =========================================================
# 🛠️ TOOL 4: LIST DIRECTORY
# =========================================================
def list_directory(directory_path: str = ".") -> str:
    """Lists files and folders in the specified directory.
    
    Args:
        directory_path: Directory to inspect (default: current directory '.').
    """
    try:
        items = os.listdir(directory_path)
        return json.dumps({"status": "success", "items": items})
    except Exception as e:
        return json.dumps({"status": "error", "error": str(e)})
```

---

### 🤖 Step 3: Configure the Cursor AI Agent Brain

```python
# System prompt sets the persona and operational guidelines of Cursor
CURSOR_SYSTEM_INSTRUCTION = """
You are an expert Autonomous AI Software Engineer (like Cursor / Claude Code).
You have direct access to the user's terminal and filesystem through tools.

OPERATIONAL RULES:
1. Always solve tasks completely end-to-end.
2. If asked to create a project or server:
   - Step A: Create the code files using write_file.
   - Step B: Install required dependencies or run the tests using execute_command.
   - Step C: If any command fails with an error in stderr, ANALYZE the error and FIX it autonomously.
   - Step D: Verify the solution works before declaring completion.
3. Keep your reasoning sharp, concise, and focused on working code.
"""

cursor_tools = [execute_command, write_file, read_file, list_directory]

cursor_config = types.GenerateContentConfig(
    system_instruction=CURSOR_SYSTEM_INSTRUCTION,
    temperature=0.0, # Zero temperature is strictly required for accurate tool calling
    tools=cursor_tools
)
```

---

### 🎮 Step 4: Interactive Main CLI Loop

```python
def start_cursor_cli():
    print("=" * 70)
    print("⚡ MINI-CURSOR AI AGENT INITIALIZED (Terminal & Filesystem Enabled)")
    print("=" * 70)
    print("Give any programming or system task (Type 'exit' to quit).\n")
    
    # Create persistent multi-turn chat session
    chat_session = client.chats.create(
        model="gemini-2.5-flash",
        config=cursor_config
    )
    
    while True:
        try:
            user_input = input("\n👤 Developer > ")
            if user_input.strip().lower() in ["exit", "quit", "q"]:
                print("👋 Exiting Mini-Cursor. Happy Coding!")
                break
                
            if not user_input.strip():
                continue
                
            print("\n🤖 Cursor Agent Thinking & Working...")
            response = chat_session.send_message(user_input)
            
            print("\n" + "-" * 60)
            print(f"🎯 [AGENT SUMMARY]:\n{response.text}")
            print("-" * 60)
            
        except KeyboardInterrupt:
            print("\nOperation cancelled by user.")
            break
        except Exception as e:
            print(f"\n❌ Error occurred: {e}")

if __name__ == "__main__":
    start_cursor_cli()
```

---

## 5️⃣ Real-World Test Run Simulation

### 💻 User Task:
> *"Create a Python script `math_utils.py` with a function to check if a number is prime. Then create a test script `test_math.py`, run it with python, and confirm it passes."*

```
══════════════════════════════════════════════════════════════════════
👤 Developer > Create a Python script math_utils.py with is_prime(n), 
               create test_math.py, run it, and confirm it passes.

🤖 Cursor Agent Thinking & Working...

📝 [FILE WRITE] > math_utils.py
(Created is_prime function)

📝 [FILE WRITE] > test_math.py
(Created test assertions for primes 2, 3, 17, 97 and non-primes 4, 9, 100)

⚙️ [TERMINAL EXEC] > python test_math.py
📄 [STDOUT]:
All 8 test assertions passed successfully! ✅

------------------------------------------------------------
🎯 [AGENT SUMMARY]:
1. Created `math_utils.py` with an optimized $O(\sqrt{n})$ `is_prime` implementation.
2. Created `test_math.py` covering edge cases (0, 1, negatives, large primes).
3. Executed `python test_math.py` in the terminal — all tests passed with exit code 0.
------------------------------------------------------------
```

---

## 6️⃣ Security, Guardrails & Production Sandboxing

```
                     PRODUCTION CODING AGENT SECURITY:
                     
  Level 1: Blacklist & Regex Filtering
  • Block dangerous shell commands (`rm -rf`, `sudo`, `dd`, `curl | bash`).

  Level 2: User-in-the-Loop Confirmation
  • Ask human: "Agent wants to run `npm install express`. Allow? (y/n)"

  Level 3: Isolated Sandboxes (Docker / gVisor)
  • Agent runs inside a disposable Docker container with resource CPU/RAM limits.
```

---

## 🧠 Mental Model & Quick Revision Cheatsheet

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    BUILD YOUR OWN CURSOR AGENT CHEATSHEET                   │
├─────────────────────────────────────────────────────────────────────────────┤
│  1. THE 4 PILLARS OF CURSOR:                                                │
│     • LLM Brain (Gemini 2.5 Flash / Claude 3.5 Sonnet / GPT-4o)             │
│     • Terminal Tool (`subprocess.run` with stdout/stderr capture)           │
│     • Filesystem Tools (`read_file`, `write_file`, `list_dir`)              │
│     • ReAct Self-Healing Feedback Loop                                      │
├─────────────────────────────────────────────────────────────────────────────┤
│  2. SELF-CORRECTION MECHANISM:                                              │
│     If `exit_code != 0` ➔ Return `stderr` to LLM ➔ Model modifies code     │
│     ➔ Re-executes terminal command ➔ Verifies success.                     │
├─────────────────────────────────────────────────────────────────────────────┤
│  3. KEY CONFIGURATION:                                                      │
│     • Temperature = 0.0 (Strictly deterministic function calling)           │
│     • Subprocess Timeout = 30s (Avoids hanging infinite processes)          │
│     • Persistent Chat Session = Keeps memory of file paths & execution logs │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

> **Next**: AI Day 4 — Vector Embeddings, Semantic Similarity & Building Custom RAG Pipelines from Scratch 🚀
