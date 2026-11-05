---
title: GitHub Copilot Credit Efficiency Guide
description: Comprehensive guide to optimizing token usage and reducing AI credits consumption in GitHub Copilot
tags: [copilot, optimization, token-efficiency, credits, vscode]
---

# GitHub Copilot Credit Efficiency Guide

A comprehensive guide to maximizing efficiency and minimizing token consumption when using GitHub Copilot. Learn proven strategies to reduce AI credit usage while maintaining or improving code quality.

## Table of Contents

1. [Token Billing Structure](#token-billing-structure)
2. [VS Code Version and Infrastructure Improvements](#vs-code-version-and-infrastructure-improvements)
3. [Prompt Caching Optimization](#prompt-caching-optimization)
4. [Tool Search and Loading](#tool-search-and-loading)
5. [Background Todo Agent](#background-todo-agent)
6. [Tool Output Compression](#tool-output-compression)
7. [Optimized Model Selection](#optimized-model-selection)
8. [Cache Usage Optimization](#cache-usage-optimization)
9. [Prompt Engineering Best Practices](#prompt-engineering-best-practices)
10. [Agent Skills and Customization](#agent-skills-and-customization)
11. [Practical Tips and Techniques](#practical-tips-and-techniques)
12. [Cost Optimization Checklist](#cost-optimization-checklist)

---

## Token Billing Structure

Understanding how tokens are billed is fundamental to optimizing costs.

### Billing Formula

$$\text{Bill} = \text{Output Tokens} + \text{Input Tokens} + \text{Cache Tokens}$$

### Token Composition

Every request to Copilot includes multiple components that consume tokens:

| Component | Purpose | Impact |
|-----------|---------|--------|
| **System Instructions** | Core agent behavior | Cached if prefix unchanged |
| **Copilot Instructions** | Custom agent directives | Cached if prefix unchanged |
| **Tool Schema** | Tool definitions and parameters | Can be deferred with tool search |
| **Conversation History** | Previous messages in session | Cached across turns |
| **File Contents** | Code being analyzed/edited | Uncached if changed |

### Cost Implications

- **Cached input tokens**: Up to 10x cheaper than uncached input (depending on model)
- **Output tokens**: Generated content, paid at full rate
- **Uncached input tokens**: Highest cost per token
- **Cache retention**: Extended caching (24h) requires configuration

### Pricing Tiers by Model Type

Different models have different token costs:
- **Lighter models**: Lower per-token cost, suitable for simple tasks
- **Mid-tier models**: Balanced cost and capability
- **Reasoning models**: Higher per-token cost, best for complex analysis
- **Frontier models**: Highest capability, use only when necessary

---

## VS Code Version and Infrastructure Improvements

### Minimum Requirements

| Milestone | Version | Feature | Recommendation |
|-----------|---------|---------|-----------------|
| Extended Caching | 1.118 | Improved prompt cache retention | Upgrade to access |
| Background Todo Agent | 1.119 | Experimental lightweight todo management | Opt-in feature |
| **Accurate Billing** | **1.120+** | **Usage-based billing accuracy** | **Required for reliable metrics** |
| Latest Stable | Latest | All newest optimizations | Recommended default |

**Important**: VS Code 1.120 or later is required for accurate usage-based billing information. Older versions may display incorrect model pricing and inaccurate usage data.

### Recommendation

Use the **latest stable version** of VS Code rather than downgrading to access a specific optimization. The cumulative improvements across versions provide better overall efficiency than targeting a single feature.

---

## Prompt Caching Optimization

### How Caching Works

In an agentic session, many request components repeat across turns:
- System instructions
- Tool definitions
- Repository context
- Conversation history
- Previous messages

This repeated beginning is called the **prompt prefix**. When requests share the exact same prefix, the inference provider can reuse cached model state instead of recomputing from scratch.

### Cache Benefits

- **Cost Reduction**: Cached tokens cost up to 10 times less than uncached tokens
- **Latency Improvement**: Faster model responses
- **Higher Quality**: Consistent context across multi-turn conversations

### Enabling Extended Prompt Caching

By default, cache lives in GPU memory and expires after 5-10 minutes of inactivity. Enable extended caching to keep cache for up to 24 hours:

```json
{
  "prompt_cache_retention": "24h"
}
```

This moves the cache to GPU-local storage, maintaining reusability across longer time gaps.

### When Caching Is Broken

Cache is invalidated when:
- System instructions change
- Tool schema definitions change
- New tools are added to context
- Repository context is modified

**Strategy**: Keep prompts stable within a session, and use separate sessions for different tasks to maximize cache reuse.

### Cache Hit Rate Experiments

**Reported Token Reduction**:
- GPT-5.4: 9.81% median tokens per turn reduction
- GPT-5.5: 8.61% median tokens per turn reduction

These are production A/B test results from VS Code engineering team.

---

## Tool Search and Loading

### Problem: Tool Definition Overhead

Agents can access many tools:
- Built-in tools (read, edit, terminal, search, git)
- MCP server tools
- Extension-provided tools

Historically, every tool was loaded into context on every request with:
- Full name and description
- Complete JSON parameter schema

This consumes tokens even if the agent never uses most tools.

### Solution: Deferred Tool Loading

Tool search allows the model to **load tool definitions on demand**:

1. **Initial request**: Model sees only lightweight metadata (name + description)
2. **When needed**: Heavy parameter schemas load into context
3. **Cache preservation**: Deferred tools added at end of context window, keeping prefix reusable

### Performance Gains

| Model | Token Reduction |
|-------|-----------------|
| GPT-5.4 | 9.81% |
| GPT-5.5 | 8.61% |

### Best Practice

Only include tools relevant to the task. Unused tool definitions are pure token waste.

---

## Background Todo Agent

### What Is It?

An experimental lightweight agent that manages task progress instead of the primary model.

### Why It Matters

**Before**: Primary model spends tokens on:
- Creating todo lists
- Updating todo status
- Managing progress tracking
- Re-reading todos on each turn

**After**: Background agent handles todo management, freeing primary model for actual implementation.

### Enabling Background Todo Agent

Add to VS Code settings:

```json
{
  "github.copilot.chat.agent.backgroundTodoAgent.enabled": true
}
```

### Best For

- Multi-step development tasks
- Long-running agent sessions
- Complex projects with many dependencies
- Tasks where progress tracking takes significant overhead

### Impact

- Reduced main agent token consumption
- Better context available for implementation
- Clearer progress tracking
- Faster agent execution

---

## Tool Output Compression

### Overview

Tool output compression reduces the size of command outputs sent to the model, minimizing unnecessary token consumption.

### Feature 1: `chat.tools.compressOutput.enabled`

**Purpose**: Automatically compress tool output before sending to the model

**Configuration**:
```json
{
  "chat.tools.compressOutput.enabled": true
}
```

**How it works**:
- Removes redundant lines
- Truncates verbose output
- Keeps relevant error messages and key information
- Filters out debugging noise

**Example**:
```
Before (hundreds of lines):
Build output with warnings, progress messages, unrelated logs...

After (compressed):
[10 key lines showing only errors and summary]
```

**Token Impact**: 20-40% reduction in terminal output tokens

### Feature 2: `chat.tools.terminal.outputDeltas`

**Purpose**: Send only changed portions of terminal output, not full history

**Configuration**:
```json
{
  "chat.tools.terminal.outputDeltas": true
}
```

**How it works**:
- Each turn only sends new output since last request
- Previous output stays in model's context window
- Avoids repeating static content
- Maintains full history without re-transmission

**Example**:
```
Turn 1: Send 1000 lines of initial output
Turn 2: Send only 5 new lines (the delta)
Turn 3: Send only 3 new lines (the delta)
```

**Token Impact**: 70-90% reduction in repeat output tokens

### Combined Benefit

Using both settings together:
1. Compression removes noise
2. Deltas avoid repetition
3. Model gets clean, recent information only

**Total Savings**: 40-70% reduction in tool output tokens per turn

---

## Optimized Model Selection

### Choose the Right Model for the Right Task

Model selection is one of the fastest ways to improve cost efficiency.

**Common Mistake**: Using the most capable (expensive) model for every task.
**Result**: Increased token usage without better outcomes.

### Model Categories

#### Reasoning Models
- **Best for**: Architecture decisions, complex debugging, system design, deep analysis
- **Cost**: Highest token consumption
- **Use when**: Problem requires multistep reasoning
- **Avoid for**: Simple tasks, refactoring, documentation

#### Mid-Tier Models
- **Best for**: Execution-heavy work when plan is clear
- **Cost**: Moderate token consumption
- **Use when**: Implementation is straightforward
- **Avoid for**: Novel architecture or complex debugging

#### Lighter Models
- **Best for**: Refactoring, formatting, documentation, routine changes
- **Cost**: Lowest token consumption
- **Use when**: Task scope is well-defined
- **Avoid for**: Novel problems requiring reasoning

### AI Model Comparison by Task

| Model | Provider | Task Area | Primary Use Case | Cost Level |
|-------|----------|-----------|------------------|------------|
| **GPT-5.3-Codex** | OpenAI | Agentic software development | Long-horizon autonomous coding | Very High |
| **GPT-5.6 Luna** | OpenAI | Fast simple tasks | Quick, cost-efficient responses | Low |
| **GPT-5.6 Sol** | OpenAI | Deep reasoning & debugging | Complex reasoning over large codebases | High |
| **GPT-5.6 Terra** | OpenAI | General-purpose coding | Balanced everyday interactive tasks | Medium-High |
| **GPT-6 Astra** | OpenAI | Long-horizon autonomous tasks | Extended autonomous coding sessions | Very High |
| **GPT-6 Luna** | OpenAI | Fast simple tasks | Quick, cost-efficient responses | Low |
| **GPT-6 Sol** | OpenAI | Interactive & agentic coding | All-round development tasks | High |
| **Claude Fable 5** | Anthropic | Routine tasks | Fast execution for simple work | Low |
| **Claude Fable 5.1** | Anthropic | Routine tasks | Fast execution for simple work | Low |
| **Claude Haiku 4.5** | Anthropic | Quick responses | Cost-efficient lightweight tasks | Very Low |
| **Claude Haiku 5.5** | Anthropic | Quick responses | Cost-efficient lightweight tasks | Very Low |
| **Claude Opus 4.8** | Anthropic | Complex tasks | Advanced reasoning and analysis | High |
| **Claude Sonnet 5** | Anthropic | Balanced tasks | Medium-complexity work | Medium |
| **Claude Sonnet 5.5** | Anthropic | Balanced tasks | Medium-complexity work | Medium |
| **Gemini 3.8 Flash** | Google | Fast execution | Quick, efficient responses | Low |
| **Gemini 4 Argon** | Google | Complex tasks | Advanced reasoning | High |

### Reasoning Level Configuration

Some models support configurable reasoning levels:

- **Regular level**: Default, use for most tasks
- **High level**: Better for complex problems, consumes more tokens
- **Strategy**: Use regular by default, increase only for harder tasks

### Copilot Fusion (Recommended)

Copilot Fusion intelligently selects the best execution pattern for each task:

**Effort Levels**:
| Level | Focus | Best For |
|-------|-------|----------|
| **Fusion Standard** | Balanced cost, quality, latency | Most day-to-day work |
| **Fusion High** | Moderate token savings | When cost matters |
| **Fusion Max** | Frontier capability | Most complex problems |

**Benefits**:
- Main conversation stays on same model (preserves cache)
- Subagents use cheaper models for focused work
- Automatic orchestration of best execution pattern

**Current Incentive**: 30% discount on model costs (through Nov 30, 2026)

### Subagent Strategy

Run subagents on cheaper models:
- Subagents have separate sessions (no inherited history)
- Context scoped to single focused task
- Lighter model usually sufficient
- Doesn't affect main agent's cache

---

## Cache Usage Optimization

### Preserving Cache Across Turns

Cache is your most powerful cost-reduction tool. Maximize it with these strategies:

### Strategy 1: Keep Prefix Stable

**Do**:
- Use same system instructions within a session
- Maintain consistent tool set
- Keep repository context unchanged within a task

**Don't**:
- Change instructions mid-session
- Add/remove tools frequently
- Switch models repeatedly

**Impact**: Each instruction/tool change breaks cache, resetting to full price.

### Strategy 2: Use Copilot Fusion

Copilot Fusion keeps main conversation on same model, preserving cache:
- Main agent stays on chosen model
- Reviewers/subagents use appropriate cheaper models
- Cache stays warm throughout session

### Strategy 3: Session Separation

**Use separate sessions for**:
- Different projects
- Different tasks
- Different instruction contexts
- Unrelated work

**Why**: Each session rebuild isn't cheaper, but separate sessions prevent cache pollution.

### Strategy 4: Avoid Mid-Session Model Switching

**Cost of switching models mid-session**:
1. Cache for old model is abandoned
2. New model has no cached context
3. Full price for entire request

**Better approach**:
- Use Copilot Fusion (handles model orchestration efficiently)
- Finish work with one model before switching
- Use subagents for model variety (separate cache)

### Strategy 5: Extended Cache Retention

Enable 24-hour cache retention so pauses don't break cache:

```json
{
  "prompt_cache_retention": "24h"
}
```

**Scenario**: Working on a task intermittently throughout the day
- Morning session: Cache builds up
- Lunch break: Without extended retention, cache expires
- Afternoon session: Cache still warm, saves 90% on prefix tokens

---

## Prompt Engineering Best Practices

### Principle 1: Be Concise with Prompts

**Before** (45 tokens):
```
Could you please help me write a Python function that reads the CSV file 
and returns a filtered dataframe where the status column is active?
```

**After** (14 tokens):
```
Write Python fn: read CSV return df where status is active
```

**Reduction**: 69% fewer tokens
**Key technique**: Remove filler words, use shorthand, be direct

### Principle 2: Provide Clear Guidance

Well-structured prompts have three qualities:

1. **Clear task definition**
   - Don't say: "fix this issue"
   - Do say: "Fix authentication failure on POST /login; returns 401 for valid credentials"

2. **Relevant context upfront**
   - Include specific files that matter
   - Mention relevant services or logs
   - Point to exact errors

3. **Clear stopping condition**
   - Define what "done" looks like
   - Prevent scope drift
   - Avoid unnecessary refactoring

### Principle 3: Examples of Concise Prompts

#### Example 1: Bug Fix
❌ Long: "There seems to be a problem with the user authentication system and I'm not sure how to fix it. Can you look at it?"

✅ Short: "Fix: auth fails on /login POST with valid credentials. Check: src/auth.ts lines 42-50"

**Token saving**: ~70%

#### Example 2: Feature Implementation
❌ Long: "I need to add pagination to our API endpoints. The API should support limiting results and I need to implement this across multiple endpoints."

✅ Short: "Add pagination to /users API: support limit/offset params, return pagination metadata"

**Token saving**: ~60%

#### Example 3: Code Review
❌ Long: "Can you review this code and let me know if there are any issues with how I've implemented the database queries?"

✅ Short: "Review src/db.ts for SQL injection risks in user queries"

**Token saving**: ~75%

### Principle 4: Output Rules

When creating agent instructions or prompts, follow these output rules:

```
# Output Rules

1. Code only – no explanation unless explicitly asked
2. No markdown preamble ("here is the code", "```")
3. No closing summaries ("I've completed this...")
4. Avoid introductory text
5. Focus on deliverable
```

**Example good instruction**:
```markdown
Write: Python function to parse JSON and validate against schema.

- Parameters: json_str, schema_dict
- Return: (bool, list[str]) for (valid, errors)
- Output: code only
```

---

## Agent Skills and Customization

### Caveman Skills Concept

In VS Code's Copilot extension, "Caveman Skills" refers to custom, focused instruction sets that guide agent behavior for specific tasks.

**Purpose**: Define exactly what behavior you want without lengthy explanations.

### Using Agent Files for Different Tasks

**Principle**: Use different agent instruction files (`.agent.md`, `.instructions.md`) for different task types.

**Structure**:
```
project/
├── .instructions.md           # General project instructions
├── .agent-code-review.md      # Custom code review agent
├── .agent-implementation.md   # Custom implementation agent
├── .agent-debugging.md        # Custom debugging agent
└── .agent-research.md         # Custom research agent
```

### Example: Code Review Agent File

Create `.agent-code-review.md`:
```markdown
---
description: Focused code review agent
---

# Code Review Agent

## Scope
Review staged changes for security, performance, and correctness issues.

## Rules
1. Focus on logic errors only
2. Flag SQL injection, XSS vulnerabilities
3. No style suggestions
4. Suggest fixes for critical issues
5. Output: bullet-list format

## Do Not
- Refactor unrelated code
- Add new features
- Change working code
- Request changes to passing tests
```

### applyTo Pattern

Limit agent instruction scope to specific files:

```markdown
---
applyTo:
  - "**/*.test.ts"
  - "src/services/**"
  - "!src/legacy/**"
description: Test and service layer rules
---
```

This prevents irrelevant instructions from consuming tokens on unrelated files.

### Excluding Files from Context

Reduce token usage by excluding files from search/edit:

**In VS Code settings**:
```json
{
  "copilot.excludeFiles": [
    "**/node_modules/**",
    "**/.git/**",
    "**/dist/**",
    "**/build/**",
    "**/*.lock"
  ]
}
```

**In workspace settings** (`.vscode/settings.json`):
```json
{
  "files.exclude": {
    "**/*.log": true,
    "**/temp/**": true,
    "**/.cache/**": true
  },
  "copilot.exclude": [
    "vendor/**",
    "third-party/**"
  ]
}
```

**Impact**: Prevents agent from including unnecessary files in context, saving 10-30% tokens per request.

---

## Practical Tips and Techniques

### Tip 1: Keep Sessions Short and Focused

**What**: Complete one task per session, then start a new session

**Why**:
- Each session accumulates history
- History is sent with every request (unless cached)
- Unrelated history pollutes context

**Example**:
- Session 1: Add authentication feature
- New session: Implement user profile page
- New session: Fix bug in payment flow

**Token savings**: 20-40% per subsequent turn

### Tip 2: Treat Instructions Like Code

**Principle**: Keep instructions clean, modular, and maintainable

**Do**:
- Use consistent formatting
- Group related rules
- Use clear section headers
- Version control instructions

**Don't**:
- Mix unrelated rules
- Use vague language
- Include examples within rules
- Update instructions mid-session

**File structure**:
```
project/
├── .instructions.md           # Core rules
├── .agent.md                  # Agent behavior
└── docs/INSTRUCTIONS.md       # Full documentation
```

### Tip 3: Reference What You Need

**Strategy**: Point agent to specific files/lines instead of pasting content

**Before** (includes full file):
```
I need help with this file:
[paste 500 lines of code]
```

**After** (specific reference):
```
Debug src/auth.ts lines 42-60, specifically the verifyToken function
```

**Token savings**: 70-90%

### Tip 4: Match Model to Task Complexity

| Task | Recommended Model | Reasoning |
|------|-------------------|-----------|
| Refactor function | Claude Haiku 4.5 | Simple structural change |
| Fix bug with logs | GPT-5.6 Luna | Quick analysis needed |
| Implement feature | GPT-5.6 Terra | Balanced approach needed |
| Design architecture | GPT-5.6 Sol | Deep reasoning required |
| Long-running agent | GPT-5.3-Codex | Autonomous task handling |

### Tip 5: Use Agent Mode for Complex Tasks

**When to use Agent Mode**:
- Multi-file refactoring
- Long-running implementations
- Tasks requiring multiple iterations
- Complex debugging with tool usage

**How to enable**:
- Toggle "Agent" in Copilot Chat
- Or use keyboard shortcut
- Or use `/agent` command

**Why it helps**:
- Background Todo Agent manages progress
- Extended context window available
- Tool use is optimized
- Better at multi-step tasks

**When NOT to use**:
- Simple questions
- Single-file edits
- Quick refactoring
- Status checks

---

## Optimizing by Phase

### Phase 1: Research

**Goal**: Gather information efficiently

**Strategies**:
- Use lighter models for initial exploration
- Search specific files/patterns
- Avoid large context dumps
- Create separate research session

**Model choice**: Claude Haiku 4.5 or GPT-5.6 Luna

### Phase 2: Planning

**Goal**: Define approach without implementation

**Strategies**:
- Outline approach clearly
- Use architecture-focused model
- Document decisions
- Get confirmation before building

**Model choice**: GPT-5.6 Sol or Claude Opus 4.8

### Phase 3: Implementation

**Goal**: Execute plan efficiently

**Strategies**:
- Reference planning outputs (not full context)
- Use appropriate model for complexity
- Keep instructions focused
- Use Agent mode for multi-step work

**Model choice**: GPT-5.6 Terra or Claude Sonnet 5

### Phase 4: Testing & Verification

**Goal**: Confirm quality with minimal tokens

**Strategies**:
- Run full test suite first
- Only ask agent about failures
- Use lighter model for routine fixes
- Keep test output compressed

**Model choice**: Claude Haiku 4.5 or GPT-5.6 Luna

---

## Using Separate Conversations

### When to Start New Session

| Scenario | Reason | Token Savings |
|----------|--------|---------------|
| Different project | Different context, files, instructions | 30-50% |
| Different task type | Different agent rules apply | 20-40% |
| Long pause | Cache expires, restart saves setup | 20-30% |
| Scope creep detected | Focus conversation | 15-25% |
| Unrelated question | Prevent history pollution | 10-20% |

### When to Reuse Session

| Scenario | Reason |
|----------|--------|
| Same task continuation | Cached context still relevant |
| Related tasks | Shared codebase/instructions |
| Iterative work | Building on previous context |
| Same conversation thread | Maintaining context quality |

---

## Cost Tips Command

### Using `/chronicle:cost-tips`

Run this command inside a Copilot Chat session:

```
/chronicle:cost-tips
```

**What it does**:
- Analyzes your recent activity
- Identifies inefficient patterns
- Suggests cost-reduction opportunities
- Shows savings potential

**Example output**:
```
Based on your recent sessions:
- 3 sessions with unrelated history (save 25% by splitting)
- Using reasoning model for simple tasks (save 40% with Haiku)
- 2 mid-session model switches (save 30% with Fusion)
- Large output from terminal (enable compression to save 60%)
```

**When to run**: After every major feature or weekly

---

## Cost Optimization Checklist

### Infrastructure Level

- [ ] VS Code version 1.120 or later
- [ ] Extended prompt caching enabled (`prompt_cache_retention: "24h"`)
- [ ] Background Todo Agent enabled (if using complex tasks)
- [ ] Tool output compression enabled
- [ ] Terminal output deltas enabled
- [ ] File exclusions configured (node_modules, dist, etc.)

### Model Selection Level

- [ ] Using Copilot Fusion for intelligent model selection
- [ ] Reasoning models used only for complex analysis
- [ ] Subagents use cheaper models
- [ ] Reasoning level set to regular (increased only when needed)
- [ ] Appropriate model selected per task type

### Session Management Level

- [ ] Starting new sessions for different projects
- [ ] Keeping sessions focused and short
- [ ] Using separate chats for different task types
- [ ] Reusing chat for same task continuation
- [ ] No unnecessary context accumulation

### Prompt Engineering Level

- [ ] Prompts are concise (removing filler words)
- [ ] Task definition is clear
- [ ] Relevant context provided upfront
- [ ] Stopping condition defined
- [ ] Specific file references used (not full code dumps)

### Instruction Customization Level

- [ ] Agent files organized by task type
- [ ] `applyTo` patterns limiting scope
- [ ] Instructions versioned and maintained
- [ ] Unnecessary instructions removed
- [ ] Output rules defined clearly

### Monitoring Level

- [ ] Running `/chronicle:cost-tips` regularly
- [ ] Tracking token usage per session
- [ ] Identifying expensive patterns
- [ ] Testing cost optimizations
- [ ] Adjusting strategy based on data

---

## Real-World Scenarios

### Scenario 1: Adding a New Feature

**Old Approach** (Expensive):
```
Session 1: "Add user profile feature, here's 5000 lines of context"
↓ Model reads everything, costs high
↓ Accumulates history
↓ Each turn costs more
```

**Optimized Approach**:
```
Session 1 (Research): Use Haiku
"Outline user profile architecture for [project]"
↓ Cheap exploration

Session 2 (Plan): Use GPT-5.6 Sol  
"Design profile feature: auth, storage, API - reference arch from session 1"
↓ Specific planning

Session 3 (Implement): Use GPT-5.6 Terra + Agent mode
"Implement profile feature: reference src/models/user.ts, API in src/routes"
↓ Efficient execution

Result: 60% cost reduction, better code quality
```

### Scenario 2: Debugging Complex Issue

**Old Approach**:
```
"Here are all the logs and code, please help"
[sends 10,000 lines]
↓ Model spends tokens parsing noise
```

**Optimized Approach**:
```
"Debug: /login returns 401 for valid credentials
Error in src/auth.ts:42 (verifyToken)
Logs show: [specific error message]"
↓ Concise, focused
↓ Agent knows exactly where to look
Result: 75% cost reduction, faster diagnosis
```

### Scenario 3: Refactoring Codebase

**Old Approach**:
```
Session 1: "Refactor module A"
Session 2: "Then module B"
Session 3: "Then module C"
↓ No cache reuse
↓ Context rebuilds each time
```

**Optimized Approach**:
```
Session (Agent mode): "Refactor modules A, B, C with background todo tracking"
↓ Uses Background Todo Agent
↓ Maintains cache across all modules
↓ Efficient execution
Result: 50% cost reduction, better tracking
```

---

## Troubleshooting Efficiency

### Problem: High Token Usage Per Turn

**Diagnosis**:
```
Run: /chronicle:cost-tips
Look for: "Your prompts are verbose" or "Large outputs being sent"
```

**Solutions**:
1. Enable output compression
2. Use output deltas
3. Reduce prompt length by 50%
4. Exclude large files from context

### Problem: Cache Keeps Breaking

**Diagnosis**:
```
Check if:
- Instructions change between turns
- Tools added/removed mid-session
- Models switched mid-session
```

**Solutions**:
1. Keep instructions stable within session
2. Use Copilot Fusion (automatic orchestration)
3. Avoid manual model switching
4. Enable extended cache retention

### Problem: Wrong Model Selected

**Diagnosis**:
```
Check: Current model cost vs. task complexity
```

**Solutions**:
1. Use Copilot Fusion for auto-selection
2. Manually select appropriate model
3. Use lighter models for simple tasks
4. Run `/chronicle:cost-tips` for suggestions

### Problem: Session Growing Too Large

**Diagnosis**:
```
Check: Message count, accumulated context, history length
```

**Solutions**:
1. Archive old sessions
2. Start new session for new task
3. Use separate chats for different concerns
4. Remove irrelevant messages

---

## Advanced Optimization

### A/B Testing Cost Improvements

**Test**: Does Copilot Fusion save costs for your workflow?

```
Week 1 (Baseline): Track tokens without Fusion
Week 2 (Fusion): Track tokens with Fusion enabled
Week 3 (Analysis): Compare results
```

### Measuring Cache Effectiveness

Monitor your cache hit rate:

1. Run identical task in VS Code 1.118+ (caching capable)
2. Check: First turn (no cache) vs. Turn 5+ (cached)
3. Calculate: Token reduction percentage
4. Expected: 70-90% reduction in prefix tokens

### Custom Agent Experiments

Test custom agent instructions:

```
Control: Standard Copilot behavior
Test A: Concise output rules
Test B: Specific file scoping
Test C: Combined optimizations

Measure: Tokens per turn, task completion time, quality
```

---

## Summary of Key Takeaways

| Priority | Action | Expected Saving |
|----------|--------|-----------------|
| 🔴 Critical | Update to VS Code 1.120+ | Baseline accuracy |
| 🔴 Critical | Enable prompt caching | 70-90% on prefixes |
| 🟠 High | Use appropriate model per task | 40-60% per task |
| 🟠 High | Enable output compression | 40-70% on tool output |
| 🟡 Medium | Keep sessions focused | 20-40% per session |
| 🟡 Medium | Use Copilot Fusion | 20-30% overall |
| 🟢 Low | Concise prompts | 50-70% per prompt |
| 🟢 Low | File exclusions | 10-30% per request |

### Implementation Roadmap

**Week 1**:
- Upgrade VS Code to latest stable
- Enable prompt caching (24h retention)
- Enable output compression

**Week 2**:
- Configure file exclusions
- Enable Background Todo Agent
- Start using Copilot Fusion

**Week 3**:
- Optimize prompts for conciseness
- Create agent instruction files
- Test cost-tips recommendations

**Week 4+**:
- Monitor and iterate
- Run A/B tests
- Refine based on patterns

---

## Resources

- [VS Code Blog: Improving Token Efficiency](https://code.visualstudio.com/blogs/2026/06/17/improving-token-efficiency-in-github-copilot)
- [GitHub Copilot: Optimize AI Usage](https://docs.github.com/en/copilot/tutorials/optimize-ai-usage)
- [GitHub Copilot: AI Model Comparison](https://docs.github.com/en/copilot/reference/ai-models/model-comparison)
- [GitHub Copilot: Supported Models](https://docs.github.com/en/copilot/reference/ai-models/supported-models)

---

**Last Updated**: November 5, 2026
**Document Version**: 1.0
**Audience**: GitHub Copilot Users, Teams, Enterprises

---

## Notes

This guide is based on:
- VS Code engineering team research (June 2026)
- GitHub official documentation
- Production A/B testing results
- Real-world usage patterns

Recommendations are prioritized by impact-to-implementation ratio. Start with high-impact changes first for fastest results.
