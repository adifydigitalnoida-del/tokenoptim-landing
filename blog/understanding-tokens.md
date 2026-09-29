# Understanding AI Tokens: The Hidden Cost of Your Prompts

**Meta:** Keywords: token usage, Claude tokens, ChatGPT tokens, AI API costs, prompt compression
**Slug:** `/blog/understanding-tokens`
**Read time:** 6 min

## Why tokens matter

Every API call to Claude, ChatGPT, Gemini, or any LLM costs money. But you're not charged per *word*—you're charged per **token**.

One token ≈ 4 characters. But it's not that simple.

A single "hello" = 1 token. But "Hello!" = 2 tokens. A newline? Maybe 1. A code block? Potentially 10+ tokens per line.

**The problem:** Without seeing token counts in real-time, you're flying blind. You might write a 2,000-word prompt thinking it costs $0.02, only to get billed $0.15.

And if you're a team running 100 prompts/day? That's easily **$5,000–15,000/month** in wasted tokens.

## How tokens work

### Text tokenization
Each LLM breaks text into chunks (tokens) using its own tokenizer:
- **Claude:** Uses a custom tokenizer (slightly fewer tokens than GPT-4)
- **ChatGPT (GPT-4):** Uses the cl100k_base tokenizer
- **Gemini:** Uses SentencePiece tokenization

The same sentence in Claude costs 12 tokens but 14 in ChatGPT.

### Code is token-heavy
Code tokens way more than prose. A function signature can cost 15–20 tokens, and a loop adds another 10. That's why pasting entire codebases into AI is expensive.

### System prompts add up
That 500-word system prompt? **200–250 tokens per request.** If you run 100 prompts, that's 20,000+ tokens just on instructions.

## Real-world example

**The prompt:**
```
You are an expert Ruby developer. Optimize this code for performance.

def calculate_total(items)
  total = 0
  items.each do |item|
    total += item[:price] * item[:quantity]
  end
  total
end

Your response should include:
1. The optimized code
2. 3 specific optimizations
3. Benchmarks if possible
```

**Token breakdown (Claude):**
- System prompt: 45 tokens
- User prompt: 89 tokens
- **Total input: 134 tokens**

At Claude's Opus rate ($0.015 per 1K input tokens):
- **Cost: $0.002 per request**

But run this 50 times/day:
- **$0.10/day = $3/month per person**
- Scale to 10 people: **$30/month**
- 100 people: **$300/month** (and you never tracked it)

## How to reduce token usage

### 1. Compress your prompts
Instead of:
```
I am a Python expert. I specialize in data science and machine learning.
I have 10 years of experience writing production code. Please help me write
a function that reads a CSV file and returns the top 10 rows sorted by 
the "age" column in descending order.
```

Write:
```
Expert Python dev. Write a function to read CSV, return top 10 rows sorted by age (desc).
```

**Saves:** ~40 tokens per prompt

### 2. Reuse system prompts
Create 3–5 core system prompts (writer, coder, analyst) instead of 20 unique ones. Store them server-side; don't re-send every request.

### 3. Batch requests
If you can combine 3 small prompts into 1 larger one, do it. The overhead is lower.

### 4. Use semantic caching
If you ask the same question twice (e.g., "Summarize this doc"), the LLM remembers and charges 10% for the cached portion.

**Claude's semantic caching saves 90% on cached tokens.** ChatGPT doesn't support it yet.

## Token counting tools

Manually counting is slow. Use automated tools:
- **TokenOptim:** Real-time counter + compression suggestions (free Chrome extension)
- **OpenAI tokenizer:** CLI tool (ChatGPT only)
- **Anthropic tokenizer:** Python library (Claude only)
- **Gemini tokenizer:** Web dashboard (Gemini only)

## Key takeaway

Token usage is the **hidden variable** in your AI budget. A 10% reduction in tokens = **$5,000–10,000/year savings** per person.

Track it. Compress relentlessly. Use caching. Watch your AI costs plummet.

---

**Next read:** [The Ultimate Token Compression Guide](/blog/token-compression)
