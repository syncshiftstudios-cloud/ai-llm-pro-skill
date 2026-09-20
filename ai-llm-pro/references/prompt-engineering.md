# Prompt Engineering Guide

## Core Concepts

### What is a Prompt?
A prompt is the instruction you give to an LLM. The quality of your prompt
directly determines the quality of the output.

**Bad prompt:** `Write code`
**Good prompt:** `Write a Python function that takes a list of integers and
returns the top 3 largest values. Include type hints, docstring, and edge case
handling for empty lists.`

---

## The 6 Elements of a Perfect Prompt

```
[ROLE]       → Who is the AI?
[CONTEXT]    → Background information
[TASK]       → What exactly to do
[FORMAT]     → How the output should look
[EXAMPLES]   → Show, don't just tell
[CONSTRAINTS]→ Rules and limitations
```

### Template:
```
You are a [ROLE] with expertise in [DOMAIN].

Context: [BACKGROUND INFO]

Task: [SPECIFIC TASK]

Rules:
- [CONSTRAINT 1]
- [CONSTRAINT 2]

Output format: [FORMAT — JSON, markdown, bullet points, etc.]

Example:
Input: [EXAMPLE INPUT]
Output: [EXAMPLE OUTPUT]
```

---

## Key Techniques

### 1. Chain of Thought (CoT)
Make the model think step by step before answering.

```
❌ Bad:  "What is 17 × 24?"
✅ Good: "What is 17 × 24? Think step by step before giving the answer."
```

**When to use:** Math, logic, reasoning, complex analysis

---

### 2. Few-Shot Prompting
Give 2-5 examples of input → output pairs.

```
Classify the sentiment:

Text: "I loved the movie!" → Positive
Text: "The food was awful" → Negative
Text: "The service was okay" → Neutral

Now classify: "The app crashes sometimes but the UI is beautiful"
```

**When to use:** Classification, formatting, style matching

---

### 3. System Prompts
Set the AI's persona and rules at the start of a conversation.

```python
# OpenAI
client.chat.completions.create(
    model="gpt-4o",
    messages=[
        {"role": "system", "content": "You are a senior Python engineer. 
         Always write production-ready code with error handling. 
         Never use deprecated libraries."},
        {"role": "user", "content": "Write a file upload function"}
    ]
)
```

**When to use:** Any multi-turn conversation, chatbots, agents

---

### 4. Structured Output (JSON Mode)
Force the model to return valid JSON.

```python
# OpenAI JSON mode
response = client.chat.completions.create(
    model="gpt-4o",
    response_format={"type": "json_object"},
    messages=[{
        "role": "user",
        "content": "Extract name, age, and email from: 'Hi I am Yash, 25 years old, yash@email.com'"
    }]
)
# Output: {"name": "Yash", "age": 25, "email": "yash@email.com"}
```

**When to use:** When output will be parsed by code

---

### 5. Role Prompting
Assign a specific expert persona.

```
You are a senior security engineer with 10 years of experience in
web application security. Review this code for vulnerabilities:
[CODE]
```

**Powerful roles to use:**
- "You are a senior [ENGINEER TYPE] at [COMPANY TYPE]"
- "You are a [DOMAIN] expert who is also a great teacher"
- "You are a critical code reviewer who finds edge cases"

---

### 6. Prompt Chaining
Break complex tasks into smaller prompts.

```
Step 1 Prompt: "Analyze this business idea and list 5 key risks"
Step 2 Prompt: "For each risk from the previous response, suggest a mitigation strategy"
Step 3 Prompt: "Now create a 30-day action plan to launch despite these risks"
```

**When to use:** Complex multi-step tasks, long documents

---

## Platform-Specific Tips

### OpenAI (GPT-4o, o1)
- Use `response_format` for JSON
- `o1` models: don't add "think step by step" — they do it automatically
- Temperature: 0 for factual, 0.7-1.0 for creative

### Anthropic (Claude)
- Put instructions in `<instructions>` XML tags for clarity
- Use `<example>` tags for few-shot examples
- Claude follows system prompts very strictly

### Google Gemini
- Works well with markdown-formatted prompts
- Supports multimodal — combine text + image prompts

### Meta Llama (local/Ollama)
- Be very explicit — less instruction-following than GPT-4
- System prompts are critical
- Repeat important constraints in user message too

---

## Common Mistakes to Avoid

| ❌ Mistake | ✅ Fix |
|-----------|--------|
| Vague task | Be specific about expected output |
| No format specified | Always specify JSON/markdown/plain text |
| Too long context | Put most important info at start AND end |
| No examples | Add at least 1-2 examples |
| Asking multiple things | One prompt = one task |
| "Don't do X" | Say "Do Y instead" (positive instructions work better) |

---

## Prompt Debugging Checklist

When a prompt gives bad results:
- [ ] Is the role/persona clear?
- [ ] Is the task specific enough?
- [ ] Did you specify output format?
- [ ] Did you add examples?
- [ ] Is the context too long? (trim it)
- [ ] Are you using the right model for this task?
- [ ] Try temperature = 0 to test determinism
