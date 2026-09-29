# Advanced Token Optimization: Semantic Caching, Batching, & Prompt Engineering

**Meta:** Keywords: semantic caching, prompt batching, token efficiency, advanced optimization, batch API, cost reduction
**Slug:** `/blog/advanced-token-optimization`
**Read time:** 10 min

## Beyond the basics

Compression is table stakes. Here's how teams that are *serious* about costs cut 50-70% more.

## Tactic 1: Semantic caching (Claude only)

### What it is

Traditional caching checks if input matches *exactly.* Semantic caching checks if meaning matches.

**Example:**

```
Request 1: "Summarize the latest AI research"
Request 2: "Give me a summary of recent advances in AI"
```

Traditional cache: Miss (different words)
Semantic cache: Hit (same meaning)

### How it works

Claude tracks the "semantic fingerprint" of your prompt system + messages. If a new request has 90%+ semantic overlap:
- You pay only 10% of the cached portion
- Full price for new content

### Real numbers

**Setup:**
```javascript
const client = new Anthropic();

const systemPrompt = `You are an expert financial analyst...`; // 150 tokens

const baseRequest = {
  model: "claude-3-5-sonnet",
  system: systemPrompt,
  max_tokens: 1000,
  messages: [
    { 
      role: "user", 
      content: "Analyze Q3 earnings for Tesla. Focus on gross margin trends." 
    }
  ]
};

// Request 1: Tesla
const resp1 = await client.messages.create(baseRequest);
// Cost: 450 input tokens @ $3 = $1.35

// Request 2: Apple (90% identical system + similar analysis request)
const resp2 = await client.messages.create({
  ...baseRequest,
  messages: [{
    role: "user",
    content: "Analyze Q3 earnings for Apple. Focus on gross margin trends."
  }]
});
// Cost: 50 input tokens (cached 400 @ $0.30) = $0.15 + new 50 @ $3 = $0.30

// Request 3: Microsoft (same pattern)
const resp3 = await client.messages.create({
  ...baseRequest,
  messages: [{
    role: "user",
    content: "Analyze Q3 earnings for Microsoft. Focus on gross margin trends."
  }]
});
// Cost: $0.30 (same caching applies)
```

**Total for 3 analyses:**
- Without caching: $1.35 × 3 = $4.05
- With caching: $1.35 + $0.30 + $0.30 = **$1.95**
- **Savings: 52%**

### When to use semantic caching

✅ **Great for:**
- Repeated workflows (daily reports, recurring analysis)
- Templates with varying data (earnings reports, summaries)
- Long system prompts (framework instructions, knowledge bases)
- Batch document processing

❌ **Not ideal for:**
- Completely unique requests
- Ad-hoc, one-off queries
- Real-time interactive chat

### Enable it in your code

```javascript
const response = await client.messages.create({
  model: "claude-3-5-sonnet",
  system: [{
    type: "text",
    text: "Your detailed system prompt here...",
    cache_control: { type: "ephemeral" } // Enable caching
  }],
  messages: [{
    role: "user",
    content: "Your prompt..."
  }]
});

// In response headers:
// "usage": {
//   "input_tokens": 200,
//   "cache_creation_input_tokens": 0,
//   "cache_read_input_tokens": 100  // 100 tokens from cache!
// }
```

---

## Tactic 2: Batch API (input optimization)

### What it is

Instead of sending 10 individual requests, send 1 batch with 10 items. Claude processes them together, charges less overhead.

**Savings: 20-30% on input tokens**

### Example: Processing 100 support tickets

**Traditional approach (100 API calls):**
```javascript
for (let i = 0; i < 100; i++) {
  const response = await client.messages.create({
    model: "claude-3-5-sonnet",
    system: "You are a support analyst. Categorize tickets.",
    messages: [{
      role: "user",
      content: tickets[i].text
    }]
  });
}
// 100 requests × 200 tokens overhead = 20,000 tokens wasted
```

**Batch approach (1 API call):**
```javascript
const requests = tickets.map(ticket => ({
  custom_id: ticket.id,
  params: {
    model: "claude-3-5-sonnet",
    max_tokens: 200,
    system: "You are a support analyst. Categorize tickets.",
    messages: [{
      role: "user",
      content: ticket.text
    }]
  }
}));

const batchResponse = await client.beta.messages.batches.create({
  requests: requests
});
// 1 request, minimal overhead, same results
```

**Token savings:**
- Traditional: 100 × 200 tokens = 20,000 wasted
- Batch: ~2,000 wasted overhead
- **Savings: 18,000 tokens = $54**

### When to use batching

✅ **Great for:**
- Processing lists of documents
- Bulk categorization / tagging
- Daily batch processing (reports, summaries)
- Non-urgent work (can wait 1-5 hours)

❌ **Not ideal for:**
- Real-time requests
- Interactive chat
- Requests that depend on prior responses

---

## Tactic 3: Prompt chaining with context reuse

### What it is

Instead of passing the entire prompt each time, reuse context from prior responses.

### Example: Multi-step analysis

**Anti-pattern:**
```javascript
// Step 1: Summarize document
const summary = await claude.create({
  system: "Summarize documents",
  messages: [{
    role: "user",
    content: fullDocument // 10,000 tokens
  }]
});

// Step 2: Extract insights
const insights = await claude.create({
  system: "Extract insights",
  messages: [{
    role: "user",
    content: fullDocument + "Based on this document, extract insights..." // 10,000 + overhead
  }]
});

// Total: ~20,000 tokens
```

**Better pattern:**
```javascript
const conversation = [];

// Step 1: Summarize
const summary = await claude.create({
  system: "You are an analyst",
  messages: [
    { role: "user", content: fullDocument },
  ]
});
conversation.push({ role: "assistant", content: summary.content });

// Step 2: Extract insights (reuse context)
const insights = await claude.create({
  system: "You are an analyst",
  messages: [
    ...conversation,
    { role: "user", content: "Now extract key insights from your summary." }
  ]
});
// Total: ~10,000 tokens (document sent once)
```

**Savings: ~50%**

---

## Tactic 4: Structured output caching

### What it is

Return responses in a predictable JSON format. Reuse parsing logic across requests.

### Example: Categorizing support tickets

**Without structure:**
```javascript
const response = await claude.create({
  messages: [{
    role: "user",
    content: "Categorize this ticket: 'My login is broken'"
  }]
});

// Response: "This ticket is about authentication. It's urgent..."
// Now you parse the string yourself (fragile)
```

**With structure:**
```javascript
const response = await claude.create({
  messages: [{
    role: "user",
    content: `Respond in JSON:
{
  "category": "string (bug|feature|support)",
  "priority": "string (low|medium|high|critical)",
  "summary": "string"
}`
  }]
});

// Response: {"category":"bug","priority":"critical","summary":"Auth failure"}
// Parsing is deterministic
```

**Token savings:**
- Structured output often uses fewer tokens (no prose fluff)
- Parsing is faster (no regex, no NLP)
- Enables better caching (predictable format)

---

## Tactic 5: Few-shot examples (done right)

### Wrong: Verbose examples

```javascript
const prompt = `
Examples:
- Input: "The quick brown fox jumps over the lazy dog"
  Output: "Animal: fox, Action: jump, Speed: quick, Description of the fox: brown"
- Input: "The cat sleeps on the mat"
  Output: "Animal: cat, Action: sleep, Location: mat"
- Input: "Birds fly in the sky"
  Output: "Animal: bird, Action: fly, Location: sky"

Now analyze: "${userInput}"
`;
// 180 tokens for examples alone
```

### Right: Compact examples

```javascript
const examples = [
  { input: "fox jumps", output: '{"animal":"fox","action":"jump"}' },
  { input: "cat sleeps", output: '{"animal":"cat","action":"sleep"}' },
  { input: "birds fly", output: '{"animal":"bird","action":"fly"}' }
];

const prompt = `Analyze animals & actions. Return JSON.
Examples: ${JSON.stringify(examples)}
Analyze: "${userInput}"`;
// 45 tokens for examples (75% savings)
```

**Rule:** Use 2-3 examples in JSON. Claude learns fast.

---

## Tactic 6: Dynamic system prompts

### What it is

Instead of one static system prompt, customize it based on the request.

### Example:

**Anti-pattern:**
```javascript
const systemPrompt = `You are an expert in many fields: 
- Software engineering
- Financial analysis
- Writing
- Design
- Data science
...

Your task is to help with: ${task}`;

// System prompt is huge even though user only needs one expertise
```

**Better:**
```javascript
const expertise = {
  "coding": "Expert software engineer. Write production-quality code.",
  "finance": "CFA-level financial analyst. Cite sources, use ratios.",
  "writing": "Pulitzer-prize winning writer. Clear, engaging prose."
};

const systemPrompt = expertise[task] || expertise["coding"];
// System prompt is 1/5 the size for most requests
```

**Token savings: 40-60%**

---

## Tactic 7: Prompt caching through templating

### What it is

Pre-compute expensive prompts, reuse them.

### Example: Code review

**Dynamic (recalculated each time):**
```javascript
const reviews = codeBlocks.map(code => 
  claude.create({
    system: generateReviewGuidelines(), // Computed each time
    messages: [{ role: "user", content: code }]
  })
);
```

**Cached:**
```javascript
const reviewGuidelines = generateReviewGuidelines(); // Computed once

const reviews = codeBlocks.map(code =>
  claude.create({
    system: reviewGuidelines, // Reused, cached
    messages: [{ role: "user", content: code }]
  })
);
```

**Savings: 90% on system prompt tokens**

---

## Optimization checklist

Before every production deployment:

- [ ] Semantic caching enabled for repeated workflows?
- [ ] Batch API used for non-urgent bulk work?
- [ ] Prompt chaining with context reuse?
- [ ] Structured output (JSON) used?
- [ ] Few-shot examples are compact (< 50 tokens)?
- [ ] System prompts are dynamic (role-based)?
- [ ] No redundant text sent multiple times?
- [ ] Compression already applied?

**Do this, and you'll cut costs by 60-70% while improving response quality.**

---

## ROI summary

| Tactic | Implementation time | Token savings | Annual savings (10-person team) |
|--------|-------------------|--------------|------------------------------|
| Semantic caching | 15 min | 40-50% | $2,400-3,000 |
| Batch API | 30 min | 20-30% | $1,200-1,800 |
| Prompt chaining | 20 min | 30-40% | $1,800-2,400 |
| Structured output | 10 min | 15-25% | $900-1,500 |
| Few-shot optimization | 5 min | 20-30% | $1,200-1,800 |
| Dynamic system prompts | 15 min | 40-60% | $2,400-3,600 |

**Total: ~1-2 hours of work = $10,000-15,000 in annual savings**

**That's $5,000-7,500 per hour of dev time. Worth it.**

---

**Next:** [TokenOptim: Monitoring & Automation](/blog/tokenoptim-setup)
