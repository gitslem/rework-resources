# Building AI Agents with GPT, Claude & Open-Source LLMs

## Overview
AI Agents are autonomous systems that can perceive their environment, reason about it, make decisions, and take actions—often iteratively until a goal is achieved. Unlike chatbots that respond to individual queries, agents plan, execute, and refine their approach.

**Key Insight:** Agents combine LLM reasoning with tool integration to solve complex, multi-step problems autonomously.

---

## Part 1: Agent Fundamentals

### What's an AI Agent?
An AI agent is a software system that:
1. **Observes** the environment (inputs, data, context)
2. **Reasons** about the situation using an LLM
3. **Decides** what action to take next
4. **Acts** by calling tools, APIs, or code
5. **Learns** from outcomes and iterates

### Agent vs Chatbot
| Aspect | Chatbot | Agent |
|--------|---------|-------|
| Interaction | Responds to queries | Plans and executes autonomously |
| Scope | Single-turn or multi-turn conversation | Multi-step, goal-oriented tasks |
| Tools | Limited or none | Extensive tool access |
| Memory | Conversation history | State, memory, knowledge bases |
| Decision Making | Follows predefined flows | Reasons dynamically |
| Example | "What's the weather?" | "Book me a flight, hotel, and rental car for next week" |

---

## Part 2: Core Agent Components

### 1. Language Model (Brain)
The LLM that powers reasoning. Examples:
- **GPT-4** (OpenAI): Most capable, expensive
- **Claude 3.5 Sonnet** (Anthropic): Great reasoning and long context
- **Llama 2/3** (Meta): Open-source, self-hosted option
- **Mistral** (Mistral AI): Fast, efficient open-source

**Model choice considerations:**
- Cost per API call vs capability
- Speed (latency requirements)
- Context window (how much information fits)
- Instruction following quality
- Availability (cloud vs self-hosted)

### 2. Memory Systems
Agents need to remember information across steps.

**Types of memory:**
- **Short-term/Working**: Current task context and recent steps
- **Long-term**: Past interactions, learned knowledge, patterns
- **External**: Databases, vector stores, knowledge bases

**Implementation:**
- Conversation history (simple)
- Vector databases (Pinecone, Weaviate) for semantic search
- Graph databases for relationships
- Structured memory modules

### 3. Tool Integration
Tools extend agent capabilities beyond pure reasoning.

**Common tools:**
- **Search**: Web search, document search
- **APIs**: REST APIs, database queries
- **Code Execution**: Run Python, JavaScript
- **File Operations**: Read/write files
- **Third-party Services**: Email, Slack, Jira, etc.

**Function Calling:**
Modern LLMs support function calling—specifying available functions and letting the model decide which to call.

```json
{
  "name": "get_weather",
  "description": "Get current weather for a location",
  "parameters": {
    "type": "object",
    "properties": {
      "location": {"type": "string"}
    },
    "required": ["location"]
  }
}
```

### 4. Decision Making Engine
Determines next action based on:
- Current state
- Available tools
- Progress toward goal
- Error handling

---

## Part 3: Agent Patterns

### ReAct (Reasoning + Acting)
The most popular agent pattern combines explicit reasoning with actions.

**Pattern:**
```
Thought: [Agent reasons about the problem]
Action: [Agent decides what tool to use]
Observation: [Tool result]
Thought: [Agent interprets the result]
Action: [Next step]
...
Final Answer: [Conclusion]
```

**Example:**
```
Question: What's the population of France's capital?

Thought: I need to find France's capital, then find its population
Action: search("France capital")
Observation: Paris is the capital of France

Thought: Now I need Paris's population
Action: search("Paris population")
Observation: Paris has approximately 2.2 million people

Final Answer: Paris, the capital of France, has approximately 2.2 million people.
```

**Benefits:**
- Transparent reasoning (you see the agent's thoughts)
- Error recovery (agent can recognize and correct mistakes)
- Multi-step tasks naturally supported

### Chain-of-Thought (CoT)
Ask the agent to think through problems step-by-step before acting.

```
Let's break this down:
1. First, I'll [step 1]
2. Then, I'll [step 2]
3. Finally, I'll [step 3]
```

### Agentic Loops
A loop that repeats until a stopping condition:

```
while not task_complete:
    thought = llm.think(current_state, tools_available)
    action = llm.decide_action(thought)
    observation = execute_action(action)
    state.update(observation)
    
    if llm.is_done(observation):
        task_complete = True
        return final_answer
```

**Stopping conditions:**
- Agent reaches the goal
- Maximum iterations exceeded
- Error encountered
- No valid tools available

---

## Part 4: Building Agents with LangChain

### Overview
LangChain is the most popular framework for building agents. It provides:
- Agent executor (handles loops)
- Tool management
- Memory integration
- Prompt templates

### Basic Example

```python
from langchain import OpenAI, Tool
from langchain.agents import initialize_agent, AgentType
from langchain.tools import Tool

# Initialize LLM
llm = OpenAI(api_key="your-key", temperature=0)

# Define tools
def search_web(query):
    # Implementation
    return results

def calculator(expression):
    return eval(expression)

tools = [
    Tool(
        name="Web Search",
        func=search_web,
        description="Useful for searching the internet"
    ),
    Tool(
        name="Calculator",
        func=calculator,
        description="Useful for math calculations"
    )
]

# Create agent
agent = initialize_agent(
    tools=tools,
    llm=llm,
    agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION
)

# Run agent
result = agent.run("What's 25 * 4? Then search for interesting facts about the number")
```

### Advanced: Custom Tools

```python
class DatabaseTool(BaseTool):
    name = "Database Query"
    description = "Query our database"
    
    def _run(self, query: str) -> str:
        # Execute database query
        return results
    
    async def _arun(self, query: str) -> str:
        # Async version
        return results
```

### Memory Integration

```python
from langchain.memory import ConversationBufferMemory

memory = ConversationBufferMemory(
    memory_key="chat_history",
    return_messages=True
)

agent = initialize_agent(
    tools=tools,
    llm=llm,
    memory=memory,
    agent=AgentType.CONVERSATIONAL_REACT_DESCRIPTION
)
```

---

## Part 5: Model Comparison

### GPT-4 (OpenAI)
**Pros:**
- Most capable model available
- Excellent instruction following
- Great tool calling support
- Largest community and examples

**Cons:**
- Most expensive ($0.03-0.06 per 1K tokens)
- Slower than GPT-3.5
- Higher latency
- Rate limits

**Best for:** Complex reasoning, high-accuracy requirements, when cost isn't primary concern

---

### Claude 3.5 Sonnet (Anthropic)
**Pros:**
- Excellent reasoning and long-context understanding
- 200K token context window (GPT-4: 128K)
- Strong instruction following
- Moderate cost ($0.003-0.015 per 1K tokens)

**Cons:**
- Newer, fewer examples than GPT-4
- Slightly slower for some tasks
- Smaller developer community (growing)

**Best for:** Long-document processing, nuanced reasoning, cost-conscious teams

---

### Llama 2/3 (Meta)
**Pros:**
- Completely open-source
- Can self-host (no API costs)
- Strong capabilities for open-source
- Good instruction following

**Cons:**
- Requires infrastructure (server/GPU)
- Slower than commercial APIs
- Smaller community
- Setup complexity

**Best for:** Privacy-critical applications, high-volume workloads, organizations with engineering resources

---

### Mistral (Mistral AI)
**Pros:**
- Fast inference
- Good quality for speed
- Competitive pricing
- Modular (different size options)

**Cons:**
- Less capable than GPT-4/Claude
- Smaller community
- Fewer advanced features

**Best for:** Fast responses needed, cost optimization, speed-critical applications

---

## Part 6: Real-World Agent Examples

### Example 1: Data Analysis Agent
```
Task: "Analyze Q3 sales data and identify trends"

Agent steps:
1. Access sales database
2. Query Q3 data
3. Run statistical analysis
4. Generate visualizations
5. Identify anomalies
6. Create summary report
```

**Tools needed:**
- Database access
- Data analysis library (Pandas, NumPy)
- Visualization library
- Report generation

---

### Example 2: Customer Support Agent
```
Task: "Help customer troubleshoot login issues"

Agent steps:
1. Ask customer for details
2. Check account status in database
3. Review recent login attempts
4. Suggest solutions based on error
5. Reset password if needed
6. Document ticket
```

**Tools needed:**
- Customer database
- Authentication system
- Ticket management
- Email/notification service

---

### Example 3: Research Agent
```
Task: "Research and summarize the latest developments in quantum computing"

Agent steps:
1. Search for recent articles
2. Fetch full article content
3. Extract key information
4. Compare findings
5. Identify emerging trends
6. Create summary document
```

**Tools needed:**
- Web search
- Web scraping
- PDF extraction
- Content summarization

---

## Part 7: Advanced Techniques

### Tool Use Optimization
- **Batching**: Make multiple tool calls simultaneously
- **Caching**: Remember tool results to avoid redundant calls
- **Filtering**: Pre-filter tools to only relevant ones
- **Adaptive selection**: Choose tools based on task type

### Error Handling & Recovery
```python
try:
    result = agent.run(user_query)
except ToolExecutionError as e:
    # Agent tries to recover
    agent.memory.add_error(e)
    retry_with_modified_approach()
except MaxIterationsExceeded:
    # Too many loops, return partial result
    return best_effort_answer()
```

### Long-Context Processing
For long documents:
1. **Chunking**: Break into manageable pieces
2. **Hierarchical**: Process summaries first, details on demand
3. **Streaming**: Process as data arrives
4. **Vector embedding**: Use semantic search instead of linear scan

### Multi-Agent Systems
Multiple agents specialized in different areas:
```
Main Agent (Coordinator)
├── Research Agent
├── Analysis Agent
├── Writing Agent
└── Editing Agent
```

Agents collaborate, passing results between them.

---

## Part 8: Best Practices

### 1. **Define Clear Goals**
Agents need explicit, measurable goals. Vague goals lead to infinite loops.

✅ Good: "Find flights under $500 from NYC to LA for next Friday"
❌ Bad: "Help me travel"

### 2. **Tool Design**
- Keep tools focused and single-purpose
- Provide clear descriptions
- Include error messages
- Return structured data

### 3. **Safety & Guardrails**
- Limit iterations (prevent infinite loops)
- Restrict tool access (don't give dangerous tools)
- Monitor costs (API calls add up)
- Validate tool outputs

### 4. **Monitoring & Debugging**
- Log agent reasoning
- Track tool execution
- Monitor costs per agent run
- Alert on failures

```python
agent.verbose = True  # See agent's thoughts
agent.max_iterations = 10  # Prevent loops
```

### 5. **Prompt Engineering**
Agent behavior is heavily influenced by system prompts.

```
You are a helpful research assistant. Your goal is to find accurate information
quickly. When you have enough information to answer the user's question, 
respond with your final answer. Do not continue searching after you've found 
sufficient information.

Available tools: [tool list]
```

---

## Part 9: Challenges & Solutions

| Challenge | Solution |
|-----------|----------|
| Agent loops endlessly | Set max_iterations, better stopping conditions |
| Hallucinated tool results | Validate against actual data, use real tools |
| Costs spiral | Monitor token usage, optimize prompts |
| Slow execution | Use parallel tool calls, optimize tools |
| Tools fail silently | Implement error handling, return errors to agent |
| Irrelevant tool calls | Improve tool descriptions, better tool selection |

---

## Part 10: Future Trends

### Emerging Technologies
- **Reasoning models**: Models trained specifically for step-by-step reasoning
- **Agentic frameworks**: Purpose-built agent platforms (CrewAI, AutoGPT)
- **Multi-modal agents**: Vision + language for richer understanding
- **Continual learning**: Agents that improve from past interactions
- **Specialized agents**: Domain-specific pre-trained agents

### Best Practices Evolving
- From ReAct → more specialized patterns per domain
- From single LLM → orchestration of multiple models
- From tools → full integration with business systems
- From standalone → embedded in applications

---

## Summary

**AI Agents:**
- Combine reasoning with action for complex problem-solving
- Follow patterns like ReAct for transparent, error-recoverable behavior
- Gain power through tool integration and external data access
- Can be built with frameworks like LangChain

**Model Selection:**
- GPT-4: Best capability, highest cost
- Claude: Great reasoning, longer context, moderate cost
- Llama: Open-source, self-hosted option
- Mistral: Speed-optimized, cost-efficient

**Success Factors:**
- Clear goals and stopping conditions
- Well-designed tools with good descriptions
- Proper error handling and monitoring
- Iterative refinement based on real usage

---

## Resources

- LangChain Documentation: https://docs.langchain.com/
- OpenAI Function Calling: https://platform.openai.com/docs/guides/function-calling
- Anthropic Agents Guide: https://docs.anthropic.com/claude/docs/build-with-claude
- LangSmith (debugging): https://smith.langchain.com/
- Hugging Face Agent Hub: https://huggingface.co/docs/agents/

---

*This guide was created by **Rework Digital** - Resources Department for automation professionals.*

Questions? Reach out: resource@reworkdigital.io | Follow on GitHub: https://github.com/Reworkdigital-io
