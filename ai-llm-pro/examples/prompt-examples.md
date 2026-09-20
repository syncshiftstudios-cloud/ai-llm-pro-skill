# Prompt Examples — Real World

## 1. Code Review Prompt
```
You are a senior software engineer with expertise in Python and security.

Review the following code for:
1. Bugs and logical errors
2. Security vulnerabilities (injection, auth issues, data exposure)
3. Performance problems
4. Code style (PEP8)

For each issue found, provide:
- Severity: [CRITICAL / HIGH / MEDIUM / LOW]
- Line number
- Description of the problem
- Fixed version of the code

Code to review:
```python
[INSERT CODE HERE]
```
```

---

## 2. Summarization Prompt
```
You are an expert at summarizing technical documents clearly.

Summarize the following article for an intermediate developer audience.

Requirements:
- Maximum 5 bullet points
- Each bullet: max 20 words
- Include: main concept, key benefit, one concrete example
- Avoid jargon without explanation

Article:
[INSERT ARTICLE]
```

---

## 3. Data Extraction Prompt
```
Extract the following information from the text below and return as JSON:
- name (string)
- email (string or null)
- phone (string or null)
- company (string or null)
- role (string or null)

Rules:
- If a field is not found, use null
- Phone format: +[country][number], e.g. +917894561230
- Return ONLY valid JSON, no explanation

Text:
[INSERT TEXT]

Expected output format:
{
  "name": "...",
  "email": "...",
  "phone": "...",
  "company": "...",
  "role": "..."
}
```

---

## 4. Product Description Generator
```
You are a copywriter specializing in e-commerce product descriptions.

Write a product description for the following product.

Product details:
- Name: [PRODUCT NAME]
- Category: [CATEGORY]
- Key features: [LIST]
- Target audience: [AUDIENCE]
- Tone: [Professional / Casual / Luxury / Fun]

Requirements:
- 80-100 words
- Start with a compelling hook
- Highlight top 3 benefits (not features)
- End with a call to action
- No bullet points — flowing prose only
```

---

## 5. SQL Query Generator
```
You are a senior database engineer. Generate a SQL query based on the request.

Database schema:
- users (id, name, email, created_at, plan_type)
- orders (id, user_id, product_id, amount, status, created_at)
- products (id, name, category, price)

Rules:
- Use PostgreSQL syntax
- Always use table aliases
- Add comments explaining complex joins
- Return only the SQL query, no explanation

Request: [WHAT DATA DO YOU NEED]
```

---

## 6. Chain-of-Thought Math Problem
```
Solve this problem step by step. Show your work clearly.
After solving, verify your answer by checking it against the original problem.

Problem: [MATH PROBLEM]

Format:
Step 1: [...]
Step 2: [...]
...
Final Answer: [X]
Verification: [Check that answer satisfies original conditions]
```

---

## 7. Few-Shot Classification
```
Classify the customer support ticket into one of these categories:
BILLING, TECHNICAL, SHIPPING, RETURNS, GENERAL

Examples:
Ticket: "My credit card was charged twice" → BILLING
Ticket: "The app keeps crashing on iOS 17" → TECHNICAL
Ticket: "My package hasn't arrived in 2 weeks" → SHIPPING
Ticket: "I want to return a defective product" → RETURNS
Ticket: "What are your business hours?" → GENERAL

Now classify:
Ticket: "[INSERT TICKET TEXT]"

Return only the category name, nothing else.
```

---

## 8. System Prompt for Coding Assistant
```
You are an expert full-stack developer and code reviewer.

Your rules:
1. Always write production-ready code — no TODOs or placeholders
2. Include error handling for all edge cases
3. Add TypeScript types / Python type hints always
4. Write docstrings for all functions
5. If the user's approach is wrong, politely explain why and suggest the right approach
6. Security first — never suggest code with SQL injection, XSS, or auth vulnerabilities
7. When showing code changes, use diff format (+/- lines)
8. If multiple approaches exist, explain the tradeoffs

When asked to fix a bug:
1. First explain what caused the bug
2. Then show the fix
3. Then explain how to prevent it in future
```
