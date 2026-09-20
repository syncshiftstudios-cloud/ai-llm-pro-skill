# Agent Examples — Real World Workflows

## 1. Research Agent
**Task:** "Research the latest AI news and write a summary"

```
[User] → Research the latest AI news from this week

[Agent Thought] I need to search for recent AI news, then summarize it

[Tool Call] search_web("AI news September 2025")
[Observation] Found: OpenAI released GPT-5, Google launched Gemini Ultra 2...

[Tool Call] search_web("AI news September 2025 site:techcrunch.com")
[Observation] More detailed articles found...

[Agent Thought] I have enough information. Let me organize and summarize.

[Response] Here's this week's AI news summary:
1. OpenAI released GPT-5...
2. Google launched...
```

**Key design:**
- Multiple searches to get comprehensive coverage
- Separate tool call for deeper sources
- Synthesizes before responding

---

## 2. Code Agent (Debugging)
**Task:** "Fix the bug in my Python script"

```
[User] → My script crashes with: IndexError: list index out of range

[Agent Thought] I need to see the code and understand the error

[Tool Call] read_file("script.py")
[Observation] Code loaded: 45 lines, error on line 23

[Agent Thought] Line 23: data[index] — need to check if index is within bounds
              The for loop on line 20 goes beyond list length

[Tool Call] run_python("
# Test the logic
data = [1, 2, 3]
for i in range(10):  # Bug: should be range(len(data))
    print(data[i])
")
[Observation] IndexError confirmed at i=3

[Tool Call] edit_file("script.py", line=20, 
    old="for i in range(10):", 
    new="for i in range(len(data)):")
[Observation] File edited

[Tool Call] run_python("exec(open('script.py').read())")
[Observation] Script runs successfully, output: [1, 2, 3]

[Response] Fixed! The bug was on line 20...
```

---

## 3. Customer Support Agent
**Task:** Handle customer queries automatically

```python
# System Prompt
SYSTEM = """
You are a customer support agent for ShopEasy.
You have access to these tools:
- lookup_order(order_id): Get order status and details
- process_refund(order_id, reason): Process a refund
- update_shipping(order_id, address): Update shipping address
- escalate_to_human(issue_summary): Escalate complex issues

Rules:
- Always look up the order before taking any action
- Confirm with customer before processing refunds
- Escalate if: fraud suspected, order > $500, customer requests supervisor
- Be empathetic and professional
"""

# Example conversation flow:
# User: "My order #12345 never arrived, I want a refund"
# → lookup_order("12345") → Status: Delivered 5 days ago
# → Ask customer to check with neighbors / building reception
# → If still not found → process_refund("12345", "item_not_received")
```

---

## 4. Data Analysis Agent
**Task:** "Analyze my sales CSV and give insights"

```
[User] → Here's my sales data (uploads CSV), what are the key trends?

[Agent Thought] I'll analyze this data step by step

[Tool Call] read_file("sales.csv")
[Observation] 1000 rows, columns: date, product, region, revenue, units

[Tool Call] run_python("
import pandas as pd
df = pd.read_csv('sales.csv')
print(df.describe())
print(df.groupby('region')['revenue'].sum().sort_values(ascending=False))
")
[Observation] Stats and regional breakdown returned

[Tool Call] run_python("
# Find top products
top_products = df.groupby('product')['revenue'].sum().nlargest(5)
# Find trend
df['date'] = pd.to_datetime(df['date'])
monthly = df.groupby(df['date'].dt.month)['revenue'].sum()
print(top_products, monthly)
")
[Observation] Top 5 products and monthly trend

[Tool Call] create_chart(data=monthly, type="line", title="Monthly Revenue Trend")
[Observation] Chart created at chart.png

[Response] 
Key Insights:
1. 📈 Revenue grew 23% from Q1 to Q3
2. 🏆 Top product: "Widget Pro" (34% of total revenue)
3. 🌍 Best region: North (42% of revenue)
4. ⚠️ Alert: December showed 15% dip — investigate
[Attaches chart.png]
```

---

## 5. Multi-Agent Content Pipeline
**Task:** "Write a blog post about AI trends"

```
[Orchestrator Agent]
  ↓ Assign tasks
  
[Research Agent]
  → search_web("AI trends 2025")
  → search_web("AI statistics 2025")
  → Returns: Research notes (500 words)
  
[Outline Agent]  
  → Input: Research notes
  → Returns: Blog outline (5 sections, key points per section)
  
[Writer Agent]
  → Input: Outline + Research
  → Returns: Full draft blog post (1200 words)
  
[Editor Agent]
  → Input: Draft
  → Check: Grammar, flow, factual accuracy, SEO keywords
  → Returns: Edited final post
  
[Orchestrator Agent]
  → Combines all outputs
  → Returns final blog post to user
```

**Implementation with CrewAI:**
```python
from crewai import Agent, Task, Crew

researcher = Agent(
    role="Research Specialist",
    goal="Find accurate, current information about AI trends",
    backstory="Expert researcher with 10 years in tech journalism",
    tools=[search_tool],
    llm="gpt-4o"
)

writer = Agent(
    role="Content Writer",
    goal="Write engaging, SEO-optimized blog posts",
    backstory="Professional tech writer, 500+ articles published",
    llm="gpt-4o"
)

editor = Agent(
    role="Senior Editor",
    goal="Ensure content quality, accuracy, and readability",
    backstory="Editor with expertise in technical content",
    llm="gpt-4o"
)

research_task = Task(
    description="Research top 5 AI trends of 2025 with statistics",
    agent=researcher,
    expected_output="Structured research notes with sources"
)

write_task = Task(
    description="Write a 1200-word blog post based on research",
    agent=writer,
    expected_output="Complete blog post in markdown format",
    context=[research_task]
)

edit_task = Task(
    description="Review and improve the blog post",
    agent=editor,
    expected_output="Polished final blog post",
    context=[write_task]
)

crew = Crew(
    agents=[researcher, writer, editor],
    tasks=[research_task, write_task, edit_task],
    verbose=True
)

result = crew.kickoff()
```
