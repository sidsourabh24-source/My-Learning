# 🤖 AI Day 2 — Building Conversational Chatbots with Memory & Autonomous AI Agents with Tools
> **Combined Lecture Notes**:
> - 💬 **Video #3**: [I Build ChatBot from Scratch | GenAI Full Course #3](https://youtu.be/tNTlcvEDFzE) (~40% Architecture & Memory)
> - 🛠️ **Video #4**: [Create Your FIRST AI Agent Today ! | GenAI Full Course #4](https://youtu.be/SaBD-bBLb3s) (~60% Agents & Tool Calling)

---

## 🗂️ Table of Contents

```
                   ┌─────────────────────────────────────────────────────────┐
                   │               THE EVOLUTION OF AI INTERACTION           │
                   ├────────────────────────────┬────────────────────────────┤
                   │  PART 1: CHATBOTS (40%)    │  PART 2: AI AGENTS (60%)   │
                   │  • Multi-turn Memory       │  • Autonomous Reasoning    │
                   │  • Context Window Mgmt     │  • Function Calling/Tools  │
                   │  • Buffer & Window Memory  │  • ReAct Execution Loop    │
                   └────────────────────────────┴────────────────────────────┘
```

| # | Section | Key Concepts |
|---|---------|--------------|
| 1 | **Chatbot vs AI Agent: The Fundamental Shift** | Passive conversationalist vs Autonomous decision maker |
| 2 | **Part 1: Stateless LLMs & Multi-Turn Memory** | Why LLMs forget, Conversation History array, Role schema |
| 3 | **Production Memory Strategies** | Buffer Memory, Sliding Window ($K$-turns), Summary Compression |
| 4 | **Hands-On: Production Chatbot in Python** | Interactive CLI / Web Chatbot with dynamic memory |
| 5 | **Part 2: What is an AI Agent?** | Agent = LLM (Brain) + Memory + Planning + Tools (Hands & Feet) |
| 6 | **The ReAct (Reason + Act) Loop** | `Thought ➔ Action ➔ Observation ➔ Answer` cycle |
| 7 | **Function Calling / Tool Calling Deep-Dive** | Tool Schema declaration, Function dispatch, Return payloads |
| 8 | **Hands-On: Building a Multi-Tool Agent from Scratch** | Calculator + Weather API + Python REPL with Google GenAI SDK |
| 9 | **Failure Modes & Production Guardrails** | Infinite action loops, Argument hallucinations, Sandboxing |
| 10| **Mental Model Cheatsheet & Architecture Comparison** | Quick reference for interviews & system design |

---

## 1️⃣ Chatbot vs AI Agent: The Fundamental Shift

```
     ┌─────────────────────────────────────────────────────────────────────────────┐
     │                      CHATBOT vs AI AGENT COMPARISON                         │
     ├──────────────────────────┬──────────────────────┬───────────────────────────┤
     │ Feature                  │ 💬 Chatbot (Passive) │ 🤖 AI Agent (Active)      │
     ├──────────────────────────┼──────────────────────┼───────────────────────────┤
     │ **Primary Job**          │ Text In ➔ Text Out   │ Goal In ➔ Actions ➔ Output│
     │ **Capability**           │ Knowledge retrieval  │ Solves real-world tasks   │
     │ **External World Access**│ ❌ No tools/actions  │ ✅ APIs, DBs, Python, Web │
     │ **Decision Autonomy**    │ Single response      │ Multi-step self-looping   │
     │ **Analogy**              │ Encyclopedia Guide   │ Junior Software Engineer  │
     └──────────────────────────┴──────────────────────┴───────────────────────────┘
```

```
                      CHATBOT (Single Turn Conversation):
  User: "What's 38492 * 49281?" ──► [LLM] ──► Hallucinated guess: "1896883252" ❌

                      AI AGENT (Reasoning + Tool Execution):
  User: "What's 38492 * 49281?"
         │
         ▼
  [Thought]: "I need an exact mathematical calculation. I will use the calculator tool."
         │
         ▼
  [Action]: execute_calculator(a=38492, b=49281, op="multiply")
         │
         ▼
  [Observation]: 1896924252 (Calculated by Python CPU)
         │
         ▼
  [Final Answer]: "The exact product of 38492 and 49281 is 1,896,924,252." ✅
```

---

# 💬 PART 1: Building Chatbots & Memory Architecture (~40%)

---

## 2️⃣ Stateless LLM APIs & Multi-Turn Conversation History

> ⚠️ **The Core Problem**: LLM APIs (OpenAI, Gemini, Anthropic) are **100% Stateless**.  
> Jab tum naya API request bhejte ho, model ko purani baatein yaad nahi rehti!

```
                   WHY LLM FORGETS WITHOUT HISTORY:
  Turn 1:
    User: "My name is Sourabh and I live in Delhi."
    Model: "Hello Sourabh! Nice to meet you."
    
  Turn 2 (If sent without history):
    User: "What is my name?"
    Model: "I'm sorry, I don't know your name because you haven't mentioned it yet." ❌

                   HOW MULTI-TURN CHATBOT SOLVES THIS:
  Turn 2 (Sent with Full History Array):
    [
      { "role": "user",  "parts": ["My name is Sourabh and I live in Delhi."] },
      { "role": "model", "parts": ["Hello Sourabh! Nice to meet you."] },
      { "role": "user",  "parts": ["What is my name?"] }
    ]
    Model: "Your name is Sourabh and you live in Delhi!" ✅
```

---

## 3️⃣ Production Memory Strategies

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                       MEMORY MANAGEMENT STRATEGIES                          │
├──────────────────────────┬──────────────────────────────────────────────────┤
│ 1. Buffer Memory         │ Saari purani baatein as-it-is append karte jao.  │
│                          │ ❌ Danger: Context window bhar jayegi & $ costs! │
├──────────────────────────┼──────────────────────────────────────────────────┤
│ 2. Sliding Window ($K$)  │ Sirf last $K$ messages (e.g. last 6 turns) rakho.│
│                          │ ⚡ Low cost, but purani baatein bhool jaata hai. │
├──────────────────────────┼──────────────────────────────────────────────────┤
│ 3. Summary Memory        │ Jab history lambi ho jaye, LLM se summary        │
│                          │ generate kara ke compact system prompt me daalo. │
└──────────────────────────┴──────────────────────────────────────────────────┘
```

```
            Sliding Window Buffer (Keeping Last 4 Turns):
  ┌────────────────────────────────────────────────────────────────┐
  │ Turn 1 (User: "I like python")       ◄── Discarded / Pruned 🗑️ │
  │ Turn 2 (Model: "Great choice")       ◄── Discarded / Pruned 🗑️ │
  ├────────────────────────────────────────────────────────────────┤
  │ Turn 3 (User: "Recommend a project") ◄── Kept in Window        │
  │ Turn 4 (Model: "Build a chatbot")    ◄── Kept in Window        │
  │ Turn 5 (User: "How to add memory?")  ◄── Kept in Window        │
  │ Turn 6 (Current User Query)          ◄── Active Input          │
  └────────────────────────────────────────────────────────────────┘
```

---

## 4️⃣ Hands-On: Production Chatbot in Python (Google GenAI SDK)

```python
import os
from google import genai
from google.genai import types
from dotenv import load_dotenv

load_dotenv()

client = genai.Client(api_key=os.getenv("GEMINI_API_KEY"))

# Step 1: Initialize Chat Session (Manages History automatically!)
chat = client.chats.create(
    model="gemini-2.5-flash",
    config=types.GenerateContentConfig(
        system_instruction="You are an empathetic, concise software engineering mentor. Always respond in friendly Hinglish.",
        temperature=0.7
    )
)

print("🤖 AI Chatbot Initialized! Type 'exit' or 'quit' to stop.\n" + "-"*50)

# Step 2: Interactive Conversation Loop
while True:
    user_input = input("\n👤 You: ")
    if user_input.lower() in ["exit", "quit", "q"]:
        print("🤖 AI: Bye! Happy coding! 🚀")
        break
    
    if not user_input.strip():
        continue

    # Send message in conversational session
    response = chat.send_message(user_input)
    print(f"🤖 AI: {response.text}")

# Step 3: Inspecting Chat History Array
print("\n📜 Conversation History Stored:")
for message in chat.get_history():
    role = message.role
    content = message.parts[0].text if message.parts else ""
    print(f"[{role.upper()}]: {content[:60]}...")
```

---

# 🛠️ PART 2: Building Autonomous AI Agents with Tools (~60%)

---

## 5️⃣ What is an AI Agent? (The 4 Core Pillars)

> 💡 **Core Formula**:  
> $$\text{AI Agent} = \text{LLM (Brain)} + \text{Memory (Context)} + \text{Planning (Reasoning)} + \text{Tools (Hands \& Feet)}$$

```
                            THE ANATOMY OF AN AI AGENT:
  
                                ┌─────────────────┐
                                │     PLANNING    │
                                │ • Goal Breakup  │
                                │ • Self-Critique │
                                └────────┬────────┘
                                         │
                                         ▼
   ┌─────────────────┐          ┌─────────────────┐          ┌─────────────────┐
   │     MEMORY      │ ◄──────► │    LLM BRAIN    │ ◄──────► │      TOOLS      │
   │ • Short-term    │          │ (Decision Maker)│          │ • Calculator    │
   │ • Long-term/RAG │          └─────────────────┘          │ • Web Search    │
   └─────────────────┘                                       │ • SQL Database  │
                                                             │ • Weather API   │
                                                             └─────────────────┘
```

---

## 6️⃣ The ReAct (Reason + Act) Loop

> 🔄 **ReAct Framework**: Model har step par pehle **Reason (Sochta)** hai, phir **Act (Tool chalta)** hai, output **Observe (Dekhta)** hai, aur jab tak goal complete na ho yeh loop repeat karta hai!

```
                          THE ReAct EXECUTION CYCLE:

                  User Goal: "What is the weather in Delhi 
                              and what clothes should I wear?"
                                     │
                 ┌───────────────────┴───────────────────┐
                 │                                       │
                 ▼                                       ▼
           [ 1. THOUGHT ]                          [ 4. THOUGHT ]
     "I need live weather of Delhi.          "Temperature is 38°C and clear.
      I should call getWeather tool."         I should recommend light clothes."
                 │                                       │
                 ▼                                       ▼
           [ 2. ACTION ]                           [ 5. FINAL ANSWER ]
     Call: getWeather(city="Delhi")          "Delhi is currently 38°C (Hot).
                 │                            Wear light cotton clothes and stay
                 ▼                            hydrated!" ✅
          [ 3. OBSERVATION ]
     Returns: { "temp": 38, "sky": "Clear" }
                 │
                 └───────────────────► Repeats Loop
```

---

## 7️⃣ Function Calling / Tool Calling Architecture

```
                    HOW TOOL CALLING WORKS INTERNALLY:

 1. Developer registers Python functions with schema definition:
    def get_stock_price(symbol: str) -> float: ...

 2. User asks: "What is the price of NVDA and AAPL?"
    Client sends Prompt + Tool Definitions to LLM.

 3. LLM returns FunctionCall Object (NOT plain text):
    FunctionCall(name="get_stock_price", args={"symbol": "NVDA"})

 4. Python executes local function get_stock_price("NVDA") ➔ Returns 125.50

 5. Client sends function output back to LLM:
    FunctionResponse(name="get_stock_price", response={"price": 125.50})

 6. LLM analyzes response and generates final human answer! 🎯
```

---

## 8️⃣ Hands-On: Building a Multi-Tool Agent from Scratch in Python

> 🚀 Ab ek complete AI Agent banate hain jiske paas **2 Real Tools** honge:
> 1. `calculate_expression` (Exact Math Calculator via Python `eval`)
> 2. `get_live_weather` (Live City Weather Simulator)

```python
import os
import json
from google import genai
from google.genai import types
from dotenv import load_dotenv

load_dotenv()
client = genai.Client(api_key=os.getenv("GEMINI_API_KEY"))

# ==========================================
# 🛠️ STEP 1: DEFINE PYTHON FUNCTIONS (TOOLS)
# ==========================================

def calculate_expression(expression: str) -> str:
    """Calculates mathematical expressions with 100% precision.
    Args:
        expression: A valid math string like '38492 * 49281' or 'sqrt(144)'.
    """
    try:
        # Safe mathematical evaluation
        allowed_names = {"__builtins__": None}
        result = eval(expression, allowed_names)
        return json.dumps({"expression": expression, "result": result, "status": "success"})
    except Exception as e:
        return json.dumps({"error": str(e), "status": "failed"})


def get_live_weather(city: str) -> str:
    """Fetches real-time temperature and weather conditions for a given city.
    Args:
        city: Name of the city (e.g., 'Delhi', 'Mumbai', 'Bangalore').
    """
    city_clean = city.lower().strip()
    mock_weather_db = {
        "delhi":     {"temp_c": 36, "condition": "Sunny & Hot", "humidity": "45%"},
        "mumbai":    {"temp_c": 31, "condition": "Humid & Breezy", "humidity": "80%"},
        "bangalore": {"temp_c": 24, "condition": "Pleasant & Cloudy", "humidity": "60%"},
    }
    
    data = mock_weather_db.get(city_clean, {"temp_c": 28, "condition": "Partly Cloudy", "humidity": "50%"})
    return json.dumps({"city": city, "data": data})


# ==========================================
# 🤖 STEP 2: CONFIGURE AGENT WITH TOOLS
# ==========================================

# Pass tool functions directly into Gemini Client configuration!
tools_list = [calculate_expression, get_live_weather]

config = types.GenerateContentConfig(
    system_instruction=(
        "You are an autonomous AI Agent. Whenever user asks a mathematical calculation or weather query, "
        "DO NOT guess. ALWAYS use the provided tools to calculate and fetch live information."
    ),
    temperature=0.0, # 0.0 temperature ensures reliable deterministic tool selection
    tools=tools_list
)

# ==========================================
# 🚀 STEP 3: RUN THE AGENTIC QUERY
# ==========================================

user_prompt = "What is 78923 * 412? Also tell me what the weather is in Bangalore."
print(f"👤 User Query: {user_prompt}\n" + "="*60)

# Model call with tools enabled
response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents=user_prompt,
    config=config
)

# In Google GenAI SDK v1+, Automatic Function Calling handles tool execution under the hood!
print("\n🎯 Agent's Grounded Final Response:")
print(response.text)
```

```
Output:
============================================================
👤 User Query: What is 78923 * 412? Also tell me what the weather is in Bangalore.
============================================================

🎯 Agent's Grounded Final Response:
1. **Mathematical Calculation**: 
   78,923 * 412 = **32,516,276**

2. **Weather in Bangalore**: 
   Current temperature is **24°C** with **Pleasant & Cloudy** conditions and 60% humidity.
```

---

## 9️⃣ Agent Failure Modes & Production Guardrails

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    AGENT FAILURE MODES & DEFENSE STRATEGIES                 │
├─────────────────────────┬───────────────────────────────────────────────────┤
│ Failure Mode            │ Production Guardrail / Solution                   │
├─────────────────────────┼───────────────────────────────────────────────────┤
│ 🔄 Infinite Action Loop │ Set `max_iterations = 5` hard cap in agent loop. │
│ 💊 Hallucinated Args    │ Strict Pydantic / JSON Schema validation on args. │
│ 💣 Dangerous Code Exec  │ Sandboxed Docker / gVisor container for Python.   │
│ 💸 Runaway Token Cost   │ Enforce max token budget per agent task.          │
└─────────────────────────┴───────────────────────────────────────────────────┘
```

---

## 🧠 Mental Model Cheatsheet

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         CHATBOT & AGENT CHEATSHEET                          │
├─────────────────────────────────────────────────────────────────────────────┤
│  CHATBOT MEMORY:                                                            │
│    • LLM is stateless → Client must store & send `history = [...]` array.   │
│    • Production: Buffer Window ($K$ turns) or Summary compression.          │
├─────────────────────────────────────────────────────────────────────────────┤
│  AI AGENT ARCHITECTURE:                                                     │
│    • Agent = LLM Brain + Tools + Memory + ReAct loop.                       │
│    • ReAct Cycle: Thought ➔ Action ➔ Observation ➔ Final Answer.            │
│    • Tool Temperature = Always `0.0` for deterministic schema invocation.   │
├─────────────────────────────────────────────────────────────────────────────┤
│  TOOL CALLING FLOW:                                                         │
│    1. Define Python Function with type hints & docstrings.                  │
│    2. Pass `tools=[func1, func2]` in LLM Config.                            │
│    3. LLM returns FunctionCall with JSON arguments.                         │
│    4. Python runs tool and passes FunctionResponse back to LLM.             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

> **Next**: AI Day 3 — Vector Embeddings, Semantic Search & Building Custom RAG Pipelines from Scratch 🚀
