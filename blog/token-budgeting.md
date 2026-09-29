# Building a Token Budget for Your Team: Cost Control 101

**Meta:** Keywords: AI budget, token spending, cost per team member, API cost management, budget tracking, financial governance
**Slug:** `/blog/token-budgeting`
**Read time:** 6 min

## The problem: Flying blind on costs

Most teams have no idea how much they're spending on AI APIs. They set up a key, forget about it, and get a shock at month-end.

Real scenario: A 10-person team thought they'd spend $500/month on Claude. They spent $4,200. No one was tracking usage.

**This session fixes that.**

## Your token budget framework

### Step 1: Define your baseline

**Questions to ask:**
- How many team members use AI daily?
- What's your average prompt size?
- How many requests per person per day?
- What's your tolerance for surprises? (e.g., max $5k/month?)

**Example: Engineering team of 10**

| Role | Users | Prompts/day | Avg tokens/prompt | Daily tokens |
|------|-------|-----------|------------------|--------------|
| Devs | 8 | 40 | 300 | 96,000 |
| PMs | 1 | 15 | 200 | 3,000 |
| Designers | 1 | 10 | 150 | 1,500 |
| **Total** | **10** | **65** | **~285** | **100,500** |

**Monthly:** 3.015M tokens

### Step 2: Map costs by model

Using **Claude 3.5 Sonnet** ($3 input / $15 output):

Assuming 70% input, 30% output:
- Input: 2.1M tokens × $3 = **$6.30**
- Output: 904K tokens × $15 = **$13.56**
- **Total: $19.86/day = $596/month**

But you probably use multiple models. Add **ChatGPT-4o** for 20% of requests:

- Claude: $596/month
- ChatGPT: $118/month
- **Total: $714/month**

### Step 3: Set per-person quotas

**Budget: $714/month ÷ 10 people = $71.40 per person**

In tokens (Claude 3.5):
- **$71.40 ÷ $0.003/1K tokens = 23,800 tokens/person/month**
- **~792 tokens/person/day**

This means each person has a "budget" of ~792 tokens/day before alerts.

### Step 4: Implement tracking

**Option A: Built-in API monitoring**
```bash
# Claude API includes usage stats
curl https://api.anthropic.com/v1/messages \
  -H "anthropic-version: 2023-06-01" \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -d '{"model":"claude-3-5-sonnet","messages":[...]}'

# Response includes:
# "usage": {"input_tokens": 234, "output_tokens": 56}
```

**Option B: TokenOptim (free)**
- Tracks tokens in real-time
- Shows cost per request
- Alerts when approaching budget
- Generates reports

**Option C: Billing dashboard**
- AWS CloudWatch (if using Bedrock)
- OpenAI usage dashboard
- Anthropic console

### Step 5: Set alerts

**Recommended thresholds:**
- 🟢 Green: 0–75% of budget (keep going)
- 🟡 Yellow: 75–90% (heads up, slow down next week)
- 🔴 Red: 90%+ (stop, review, adjust)

**Monthly budget: $714**
- Green: < $535
- Yellow: $535–$642
- Red: > $642

### Step 6: Review weekly

Every Monday, check:
1. How many tokens did the team use last week?
2. Who used the most?
3. Any outliers or waste?

**Simple spreadsheet (update manually or via API):**

| User | Mon | Tue | Wed | Thu | Fri | Weekly | Monthly (proj) |
|------|-----|-----|-----|-----|-----|--------|---------------|
| Alice (dev) | 2.1M | 1.8M | 2.2M | 1.9M | 2.0M | 10M | 40M |
| Bob (dev) | 1.9M | 2.0M | 1.8M | 2.1M | 2.0M | 9.8M | 39.2M |
| ... | ... | ... | ... | ... | ... | ... | ... |
| **Total** | **15M** | **14.2M** | **15.1M** | **14.8M** | **14.9M** | **74M** | **296M** |

**Cost projection:** 296M tokens × $0.003/1K = **$888/month** (over budget by $174)

### Step 7: Optimize & adjust

If you're over budget, you have options:

#### A. Compression (easiest)
If each person compresses prompts by 20%, you save 20% overall = $177/month saved.

**Action:** Share the token compression guide with your team.

#### B. Model switching
Switch 30% of requests from ChatGPT-4o ($5/1M) to Claude Sonnet ($3/1M) = $30–50/month saved.

#### C. Batch processing
Instead of 10 individual requests, batch into 1 larger request = 30–40% fewer tokens.

#### D. Raise budget
If the ROI is there (devs are 10x faster, PMs make better decisions), justify a higher budget to leadership.

## Token budget template

```markdown
# Token Budget for Q4 2026

## Team: Engineering (10 people)

### Baseline (from historical usage)
- Average tokens/person/day: 792
- Average tokens/team/day: 7,920
- **Monthly baseline: 237,600 tokens**

### Cost projection (Claude 3.5 Sonnet)
- Input (70%): 166,320 × $0.003 = $499
- Output (30%): 71,280 × $0.015 = $1,069
- **Total: $1,568/month** (or $15,680/year)

### Cost per person
- **$1,568 ÷ 10 = $156.80/month**
- **$1,881/year per person**

### Budget limits
- **Soft cap: $1,400/month** (10% buffer)
- **Hard cap: $1,700/month** (alert leadership)
- **Per-person limit: $140/month** (yellow flag)

### Optimization targets
1. Compression: reduce 20% of tokens → **Save $313/month**
2. Caching: reuse prompts, enable semantic caching → **Save $235/month**
3. Model switching: use Sonnet for 80%, Opus for 20% → **Save $100/month**

**Potential savings: $648/month = $7,776/year**

## Review cadence
- Weekly: Monday 10am, check usage + alerts
- Monthly: Plan optimizations
- Quarterly: Adjust budget + limits
```

## Tools to automate this

1. **TokenOptim** — Set alerts, track team usage, export reports
2. **Langchain callbacks** — Log tokens automatically in code
3. **API logs + Python script** — Parse CloudWatch/API logs, generate reports
4. **Zapier / Make** — Trigger alerts when spending exceeds threshold

## Common mistakes

❌ **Setting budget too low**
- Team gets frustrated, stops using AI
- You lose productivity gains
- Fix: Budget for actual usage + 20% buffer

❌ **Not tracking input vs. output costs**
- Output can cost 5x more than input (Opus)
- You optimize the wrong thing
- Fix: Tag all requests with model + direction

❌ **Ignoring caching opportunities**
- Semantic caching saves 90% on repeated queries
- You're leaving money on the table
- Fix: Design prompts for reusability

❌ **No weekly reviews**
- Budget creeps up without anyone noticing
- Fix: Set a recurring calendar reminder

## Your action items

1. **This week:** Calculate your baseline (use the template above)
2. **This week:** Set alerts in your chosen tool
3. **Next week:** Review with your team, share budget limits
4. **Ongoing:** Weekly 15-min budget review

**Do this, and you'll cut AI costs by 30-40% while staying compliant and transparent.**

---

**Next:** [How to Get Your Team to Compress Prompts](/blog/team-adoption)
