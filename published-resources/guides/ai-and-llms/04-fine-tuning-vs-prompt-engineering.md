# Fine-Tuning vs Prompt Engineering: When to Use What

## Overview
When you want to improve LLM performance on a specific task, you have two main approaches:
1. **Prompt Engineering**: Optimize your input (the prompt)
2. **Fine-Tuning**: Optimize the model's weights through training

The choice between them has massive cost and complexity implications. This guide helps you decide.

---

## Part 1: Prompt Engineering Explained

### What It Is
Crafting and optimizing the text prompt you send to an LLM to get better results.

### Examples
```
Generic Prompt:
"Classify this customer review"

Better Prompt:
"You are a professional customer service analyst. Classify this customer 
review as positive, negative, or neutral. Consider tone, specific complaints, 
and recommendations. Be precise."

With Examples:
"Classify customer reviews as positive, negative, or neutral. Here are examples:
[3-5 examples with classifications]
Now classify this: [customer review]"
```

### Techniques
- Adding context and background
- Providing examples (few-shot learning)
- Explicit instructions and constraints
- Role-based prompting
- Chain-of-thought reasoning

### Cost & Speed
- **Cost**: Minimal. Slightly more tokens used = slightly higher cost
- **Speed**: Immediate. Changes take effect instantly
- **Implementation**: Hours to days of experimentation

### Limitations
- Can't make a model fundamentally smarter or more capable
- Works best for tasks the model already understands
- Quality ceiling based on base model capabilities

---

## Part 2: Fine-Tuning Explained

### What It Is
Training the model on your specific data to update its weights (parameters).

**The Process:**
```
1. Prepare training data (100s-1000s of examples)
2. Set up training configuration
3. Train for hours/days on GPU
4. Evaluate performance
5. Deploy fine-tuned model
6. Maintain and update over time
```

### Example Training Data
```json
[
  {
    "instruction": "Classify this customer review",
    "input": "Great product, arrived quickly!",
    "output": "positive"
  },
  {
    "instruction": "Classify this customer review",
    "input": "Terrible quality, broke immediately",
    "output": "negative"
  }
]
```

### Fine-Tuning Methods

**Full Fine-Tuning:**
- Update all model parameters
- Requires significant GPU resources
- Takes hours/days
- Most expensive

**LoRA (Low-Rank Adaptation):**
- Update only small adapter layers
- ~1000x cheaper than full fine-tuning
- Takes minutes/hours
- Growing standard

**QLoRA:**
- LoRA + quantization (model compression)
- Cheapest option
- Can run on consumer GPUs

### Cost & Speed

| Metric | LoRA/QLoRA | Full Fine-Tune |
|--------|-----------|----------------|
| Training cost | $50-500 | $1,000-10,000+ |
| Training time | 30 min - 2 hours | 8-48+ hours |
| GPU requirements | Consumer GPU OK | A100/H100 needed |
| Model size | Adapters only | Full model |
| Inference cost | Same as base | Potentially higher |

### Benefits
- Model becomes specialized for your domain
- Better performance on specific tasks
- Can learn patterns unique to your data
- Potentially lower inference costs (shorter prompts needed)

### Challenges
- Requires significant training data (100+ examples minimum)
- Requires GPU/compute resources
- Takes time (hours to days)
- Ongoing maintenance if data changes
- Risk of overfitting on small datasets
- Deployment complexity

---

## Part 3: Side-by-Side Comparison

| Factor | Prompt Engineering | Fine-Tuning |
|--------|-------------------|------------|
| **Cost** | $1-10/month | $100-10,000+ upfront |
| **Speed to implement** | Hours-days | Days-weeks |
| **Iteration time** | Seconds | Hours-days |
| **Data requirements** | None | 100+ examples |
| **Technical complexity** | Low | High |
| **Expertise needed** | Writing/creativity | ML engineering |
| **Performance gains** | 10-30% | 50%+ (when applicable) |
| **Maintenance** | Minimal | Ongoing |
| **Model understanding** | General knowledge | Your specific domain |
| **Best for tasks** | Broad, varied | Specific, repetitive |

---

## Part 4: When to Use Prompt Engineering

### Use Prompt Engineering When:

1. **You're Getting Started**
   - Unknown if fine-tuning will help
   - Want quick wins
   - Budget is tight

2. **Task Variety Is High**
   - Many different types of requests
   - Use cases change frequently
   - Would need many fine-tuned models

3. **You Don't Have Training Data**
   - No historical examples
   - Data collection is expensive
   - New business domain

4. **Quality Is Already Acceptable**
   - Base model handles task reasonably well
   - Marginal improvements don't justify cost
   - Speed matters more than perfection

5. **Frequent Changes**
   - Policies update often
   - Knowledge base changes regularly
   - Don't want retraining overhead

### Example: Customer Support Bot
```
Start with prompt engineering:
- System prompt defining role and tone
- Retrieved knowledge base documents
- Few-shot examples of good responses

If this achieves 80%+ satisfaction → Stop
If not, then consider fine-tuning
```

---

## Part 5: When to Use Fine-Tuning

### Use Fine-Tuning When:

1. **Prompt Engineering Plateaus**
   - Tried 20+ prompt variations
   - Still not good enough
   - Need significant quality jump

2. **You Have Quality Training Data**
   - 100+ examples minimum
   - 500+ is better
   - Data is clean and consistent

3. **Task Is Specific & Repetitive**
   - Same type of request
   - Consistent patterns
   - Clear right/wrong answers

4. **Cost Savings Matter**
   - Fine-tuned model needs shorter prompts
   - Inference cost savings > training cost
   - High volume of requests

5. **Specialized Domain Knowledge**
   - Domain-specific terminology
   - Unusual patterns
   - Legal, medical, technical domains

### Example: Contract Review System
```
Fine-tuning makes sense because:
- You have 1000s of past contracts (training data)
- Task is specific (identify risky clauses)
- Patterns are consistent
- High volume of requests (amortizes cost)
- Domain expertise matters (legal knowledge)
```

---

## Part 6: Cost Analysis

### Prompt Engineering Cost
```
Team cost: 1 person × 2 weeks × $100/hr = $4,000
Token costs: Maybe $100/month in API calls
Total first year: ~$4,000 + $1,200 = $5,200
Per user: $5,200 / 10,000 users = $0.52/user
```

### Fine-Tuning Cost (LoRA)
```
Data preparation: 1 person × 1 week = $2,000
LoRA fine-tuning: $200-500
GPU inference (if dedicated): $100-500/month
Total first year: $2,000 + $300 + $1,800 = $4,100
Per user: $4,100 / 10,000 users = $0.41/user
```

**In this example: Similar cost, but fine-tuning wins on per-user basis at scale.**

---

## Part 7: Hybrid Approach (Recommended)

### Start with Prompting
```
1. Write baseline prompt
2. Test on sample data
3. Iterate and improve
4. Measure performance
5. If good enough → Ship it
6. If not → Continue to fine-tuning
```

### Upgrade with Fine-Tuning
```
1. Collect training data (from production if possible)
2. Fine-tune on top of base model
3. Combine with optimized prompt
4. Deploy fine-tuned model
5. Continue collecting data for iterative improvement
```

### Real Example: Sentiment Analysis
```
Phase 1 (Prompting):
- Write prompt with examples
- Achieves 75% accuracy
- Takes 3 days

Phase 2 (Fine-Tuning):
- Collect 500 examples from production
- Fine-tune LoRA
- Achieves 92% accuracy
- Takes 2 weeks

Ongoing:
- Monitor performance
- Collect mislabeled examples
- Retrain quarterly
```

---

## Part 8: Decision Framework

### Step 1: Assess the Task
- Is it repetitive and specific? → Fine-tuning candidate
- Is it varied and open-ended? → Prompting better
- Is it new and unknown? → Start with prompting

### Step 2: Check Your Resources
- Have you got 100+ quality examples? → Can fine-tune
- Do you have ML engineers? → Can handle fine-tuning
- Limited budget? → Prompt engineering only
- Large scale/volume? → Fine-tuning ROI improves

### Step 3: Measure Current Performance
- Run base model + good prompt
- How far from acceptable? 
  - 20% away → Probably just needs prompting
  - 50%+ away → May need fine-tuning
  - Already good → Stop

### Step 4: Make the Call

```
Decision Tree:

Do you have time for experimentation?
├─ No → Fine-tune if you have data
└─ Yes →
    Is your task specific?
    ├─ No → Use prompting
    └─ Yes →
        Do you have 100+ examples?
        ├─ No → Use prompting
        └─ Yes →
            Has prompting hit a wall?
            ├─ No → Keep prompting
            └─ Yes → Fine-tune
```

---

## Part 9: Real-World Cases

### Case 1: Customer Support Classification
**Scenario:** Classify incoming support tickets by category

**Solution:** Prompt Engineering
- Wrote system prompt defining categories
- Added 5 examples of each category
- Achieved 88% accuracy
- Total cost: $3,000
- Time to production: 1 week

**Why not fine-tune?**
- Good prompt already got 88%
- Had diverse tickets (would need large dataset)
- Cost not justified for marginal gains

---

### Case 2: Medical Record Extraction
**Scenario:** Extract key fields from patient records

**Solution:** Fine-Tuning + Prompting
- Started with prompt → 72% accuracy
- Collected 500 annotated records
- Fine-tuned with LoRA
- Combined with optimized prompt → 96% accuracy
- Total cost: $5,000
- Time: 3 weeks

**Why fine-tune?**
- High volume (10,000s of records/year)
- Task is specific and repetitive
- Regulatory requirements demand high accuracy
- Fine-tuning cost justified by accuracy + volume

---

## Part 10: Best Practices

### Prompt Engineering
1. **Test systematically**: Change one element at a time
2. **Keep examples relevant**: Match your actual use cases
3. **Measure impact**: Use metrics, not just gut feel
4. **Iterate incrementally**: Small improvements compound
5. **Document what works**: Build a library of good prompts

### Fine-Tuning
1. **Start small**: Don't collect 10,000 examples if 500 suffice
2. **Ensure quality**: Bad data makes bad models
3. **Version your models**: Track which data produced which results
4. **Validate on test data**: Don't evaluate on training data
5. **Monitor in production**: Quality degrades over time

---

## Summary

**Prompt Engineering:**
- Start here (always)
- Fast, cheap, good for variety
- Can work very well
- Minimal infrastructure

**Fine-Tuning:**
- Consider when prompting plateaus
- Requires data + compute
- Significant cost + complexity
- Worth it for specific, high-volume tasks

**Recommendation:**
1. Start with prompt engineering (hours)
2. Measure and iterate (days-weeks)
3. Only fine-tune if ROI justifies it (weeks)

**Most successful teams:**
- Use good prompt engineering as baseline
- Add fine-tuning only where needed
- Treat fine-tuning as ongoing refinement
- Combine both techniques

---

## Resources

- OpenAI Fine-tuning Guide: https://platform.openai.com/docs/guides/fine-tuning
- Anthropic Prompt Engineering: https://docs.anthropic.com/claude/docs/build-with-claude
- LLaMA Fine-tuning: https://github.com/meta-llama/llama-recipes
- LoRA Paper: https://arxiv.org/abs/2106.09685
- Weights & Biases Fine-tuning Guide: https://wandb.ai/guides/fine-tuning

---

*This guide was created by **Rework Digital** - Resources Department for automation professionals.*

Questions? Reach out: resource@reworkdigital.io | Follow on GitHub: https://github.com/Reworkdigital-io
