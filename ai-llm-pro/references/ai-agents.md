# AI Agents Guide

## What is an AI Agent?

A **regular LLM** just takes input → gives output. Done.

An **AI Agent** can:
- 🔧 Use **tools** (search web, run code, call APIs)
- 🧠 Have **memory** (remember past conversations)
- 📋 **Plan** multi-step tasks
- 🔄 **Loop** — check its own output and retry if wrong

```
User Request
     ↓
  [Agent]
     ↓
  Think → Plan → Use Tool → Check Result → Think again → ...
     ↓
  Final Answer
```

---

## Core Components of an Agent

### 1. Brain (LLM)
The model that reasons and decides what to do next.
- GPT-4o, Claude 3.5 Sonnet, Gemini 1.5 Pro — all work well

### 2. Tools
Functions the agent can call.
```python
tools = [
    {
        "name": "search_web",
        "description": "Search the internet for current information",
        "parameters": {
            "query": {"type": "string", "description": "Search query"}
        }
    },
    {
        "name": "run_python",
        "description": "Execute Python code and return the result",
        "parameters": {
            "code": {"type": "string", "description": "Python code to run"}
        }
    }
]
```

### 3. Memory

| Type | What it is | Example |
|------|-----------|---------|
| **Short-term** | Current conversation context | Chat history |
| **Long-term** | Stored knowledge (vector DB) | User preferences |
| **Episodic** | Past task records | "Last time user asked about X" |
| **Semantic** | Domain knowledge | Company docs in RAG |

### 4. Planner
How the agent decides what steps to take.

**ReAct Pattern** (most common):
```
Thought: I need to find current weather in Mumbai
Action: search_web("Mumbai weather today")
Observation: Mumbai is 32°C, humid
Thought: I have the answer now
Action: respond_to_user("Mumbai is 32°C and humid today")
```

---

## Agent Architectures

### Single Agent
```
User → [Agent + Tools] → Response
```
Good for: Simple tasks, single domain

### Multi-Agent System
```
User → [Orchestrator Agent]
              ↓
    ┌─────────┬─────────┐
[Research  [Code    [Writer
 Agent]    Agent]    Agent]
    └─────────┴─────────┘
              ↓
         Final Output
```
Good for: Complex tasks, parallel work, specialized domains

### RAG Agent (Retrieval Augmented Generation)
```
User Query → [Search Vector DB] → [Inject relevant docs into prompt] → LLM → Answer
```
Good for: Company knowledge base, documentation Q&A

---

## Building an Agent — Code Examples

### Simple Agent with OpenAI
```python
from openai import OpenAI
import json

client = OpenAI()

def search_web(query: str) -> str:
    # Your search implementation
    return f"Search results for: {query}"

def run_agent(user_message: str):
    messages = [
        {"role": "system", "content": "You are a helpful research assistant. Use tools when needed."},
        {"role": "user", "content": user_message}
    ]
    
    tools = [{
        "type": "function",
        "function": {
            "name": "search_web",
            "description": "Search the web for information",
            "parameters": {
                "type": "object",
                "properties": {
                    "query": {"type": "string", "description": "Search query"}
                },
                "required": ["query"]
            }
        }
    }]
    
    while True:
        response = client.chat.completions.create(
            model="gpt-4o",
            messages=messages,
            tools=tools
        )
        
        message = response.choices[0].message
        
        # No tool call → agent is done
        if not message.tool_calls:
            return message.content
        
        # Process tool calls
        messages.append(message)
        for tool_call in message.tool_calls:
            result = search_web(json.loads(tool_call.function.arguments)["query"])
            messages.append({
                "role": "tool",
                "tool_call_id": tool_call.id,
                "content": result
            })
```

### Agent with Memory (LangChain)
```python
from langchain_openai import ChatOpenAI
from langchain.memory import ConversationBufferMemory
from langchain.agents import AgentExecutor, create_openai_tools_agent
from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder

llm = ChatOpenAI(model="gpt-4o")
memory = ConversationBufferMemory(memory_key="chat_history", return_messages=True)

prompt = ChatPromptTemplate.from_messages([
    ("system", "You are a helpful assistant."),
    MessagesPlaceholder(variable_name="chat_history"),
    ("human", "{input}"),
    MessagesPlaceholder(variable_name="agent_scratchpad"),
])

agent = create_openai_tools_agent(llm, tools, prompt)
agent_executor = AgentExecutor(agent=agent, tools=tools, memory=memory)
```

---

## Popular Agent Frameworks

| Framework | Best For | Difficulty |
|-----------|---------|------------|
| **LangChain** | General agents, RAG | Intermediate |
| **LangGraph** | Complex multi-agent flows | Advanced |
| **AutoGen** | Multi-agent conversations | Intermediate |
| **CrewAI** | Role-based agent teams | Beginner-friendly |
| **Agno** | Fast, lightweight agents | Intermediate |
| **Antigravity** | AI coding agents | Beginner-friendly |

---

## Agent Design Best Practices

### DO ✅
- Give each agent a **clear, single responsibility**
- Add **retry logic** with max iterations (avoid infinite loops)
- Log every tool call for debugging
- Set **timeouts** on all tool calls
- Use **structured output** (JSON) for agent-to-agent communication
- Test each tool independently before connecting to agent

### DON'T ❌
- Don't give one agent too many tools (5-7 max)
- Don't skip error handling in tools
- Don't use agents for simple tasks (overkill)
- Don't let agents run without a max iteration limit
- Don't hardcode API keys in agent code

---

## When to Use an Agent vs Simple LLM Call?

| Use Simple LLM | Use Agent |
|----------------|-----------|
| Single question → answer | Multi-step research |
| Text generation/summarization | Needs real-time data |
| Code completion | Needs to run/test code |
| Classification | Needs external APIs |
| Translation | Complex decision making |

---

## Debugging Agents

**Agent stuck in loop?**
→ Add `max_iterations=10` limit
→ Check if tool outputs are being parsed correctly

**Agent ignores tools?**
→ Improve tool descriptions — be very specific
→ Add examples of when to use each tool in system prompt

**Agent gives wrong answers?**
→ Add "Always verify your answer by checking with [tool]" in system prompt
→ Use stronger model (GPT-4o over GPT-3.5)

**Agent too slow?**
→ Reduce tool count
→ Use faster model for simple steps
→ Parallelize independent tool calls
