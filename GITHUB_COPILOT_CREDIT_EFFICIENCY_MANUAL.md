---
title: GitHub Copilot Credit Efficiency Manual
description: Enterprise-grade technical manual for optimizing token usage and reducing AI credits consumption in GitHub Copilot
tags: [copilot, optimization, token-efficiency, credits, enterprise, technical-reference]
---

# GitHub Copilot Credit Efficiency Manual

**Optimize token consumption and maximize AI credit efficiency with proven strategies, advanced techniques, and enterprise-grade workflows.**

> [!NOTE]
> This manual covers GitHub Copilot's credit system, token optimization, and cost-reduction strategies. All content aligns with official GitHub documentation as of November 2026. Requires VS Code 1.120 or later for accurate billing tracking.

---

## Table of Contents

### Part 1: Foundation
1. [Overview & Core Concepts](#part-1-foundation)
2. [System Architecture & Token Consumption](#system-architecture--token-consumption)
3. [Prerequisites & Infrastructure](#prerequisites--infrastructure)
4. [Billing & Cost Model Reference](#billing--cost-model-reference)

### Part 2: Core Features
5. [Prompt Caching Strategy](#prompt-caching-strategy)
6. [Copilot Fusion: Intelligent Model Orchestration](#copilot-fusion-intelligent-model-orchestration)
7. [Subagents & Parallel Task Execution](#subagents--parallel-task-execution)
8. [Tool Optimization & Deferred Loading](#tool-optimization--deferred-loading)
9. [Tool Output Compression](#tool-output-compression)
10. [Background Todo Agent](#background-todo-agent)

### Part 3: Advanced Strategies
11. [Model Selection Strategy](#model-selection-strategy)
12. [Session Architecture Patterns](#session-architecture-patterns)
13. [Enterprise Implementation Workflows](#enterprise-implementation-workflows)

### Part 4: Reference
14. [Configuration & Settings Reference](#configuration--settings-reference)
15. [Model Comparison Tables](#model-comparison-tables)
16. [Optimization Checklist](#optimization-checklist)
17. [Troubleshooting Matrix](#troubleshooting-matrix)
18. [Glossary & Terminology](#glossary--terminology)

---

# PART 1: FOUNDATION

## Overview & Core Concepts

### What You'll Learn

This manual teaches cost-reduction strategies that reduce token consumption by **50-90%** without sacrificing code quality. The techniques range from configuration settings (infrastructure-level) to advanced execution patterns (strategy-level).

### Three Levels of Optimization

| Level | Focus | Effort | Impact | Time |
|-------|-------|--------|--------|------|
| **Infrastructure** | Settings, caching, compression | Low | 30-40% savings | 15 minutes |
| **Model Selection** | Choosing right model per task | Medium | 40-60% savings | 1-2 hours |
| **Advanced Strategies** | Fusion, subagents, sessions | High | 50-90% savings | 1-2 weeks |

### Token Economics

**The fundamental principle**: Every interaction with Copilot consumes tokens based on:
- **Input tokens**: Context sent to the model (system instructions, tools, code, history)
- **Output tokens**: Content the model generates (response text, code, edits)
- **Cached tokens**: Reused context from previous interactions (90% cheaper than uncached input)

**Cost conversion**: Total tokens × model rate = AI Credits (1 credit = $0.01 USD)

### Efficiency Hierarchy (Impact Ranked)

From highest to lowest cost-reduction impact:

| Rank | Strategy | Savings | Complexity |
|------|----------|---------|-----------|
| 1 | Prompt caching (cache hits) | 90% on prefix tokens | Low |
| 2 | Model selection | 40-60% per task | Medium |
| 3 | Tool optimization (deferred loading) | 8-9% per session | Low |
| 4 | Output compression | 40-70% on terminal output | Low |
| 5 | Prompt engineering | 50-70% with concision | Medium |

---

## System Architecture & Token Consumption

### What Consumes Tokens

Every request to Copilot includes multiple components in the input context. Understanding what consumes tokens is the first step to reducing consumption.

### Token Component Breakdown

| Component | Purpose | Typical Size | Optimization |
|-----------|---------|--------------|--------------|
| **System Instructions** | Core agent behavior and rules | 200-500 tokens | Keep stable (enables caching) |
| **Copilot Instructions** | Custom `.instructions.md` or `.agent.md` | 300-800 tokens | Keep focused and modular |
| **Tool Definitions** | API schemas for all available tools | 1,500-3,000 tokens | Use deferred tool loading |
| **Conversation History** | Previous messages in current session | 1,000-8,000 tokens | Start new sessions strategically |
| **Repository Context** | Folder structure, file listings, relevant code | 2,000-10,000 tokens | Reference specific lines, not entire files |
| **User Prompt** | Your actual request | 50-2,000 tokens | Use concise, clear language |

### Total Per-Request Calculation

**Typical request without optimization**:
```
System Instructions:       500 tokens
Tool Definitions:        2,500 tokens
Repository Context:      5,000 tokens
Conversation History:    2,000 tokens
User Prompt:              300 tokens
────────────────────────────────
Total Input:            10,300 tokens
```

**Same request with optimization**:
```
System Instructions:       500 tokens (cached)
Tool Definitions:        2,500 tokens (cached, deferred loading reduces to 300)
Repository Context:      5,000 tokens (cached, reference specific file reduces to 200)
Conversation History:    2,000 tokens (cached)
User Prompt:              300 tokens
────────────────────────────────
Total Input:              300 tokens sent (9,500 cached at 10x cheaper rate)
```

**Result**: 94% reduction in charged tokens.

---

## Prerequisites & Infrastructure

### Minimum Requirements

| Component | Requirement | Why | Impact |
|-----------|-------------|-----|--------|
| **VS Code Version** | 1.120 or later | Accurate billing tracking | Critical |
| **Prompt Caching** | Enabled (24h retention) | 90% savings on prefix tokens | High |
| **Output Compression** | Enabled | 40-70% savings on terminal output | Medium |
| **Tool Search** | Enabled (default) | 8-9% savings on tool definitions | Medium |

### Infrastructure Setup Checklist

#### Step 1: Verify VS Code Version

```bash
# Check current version
code --version
```

**Required**: 1.120 or later

**If older**: Update via Help → Check for Updates or download latest from [code.visualstudio.com](https://code.visualstudio.com)

#### Step 2: Enable Extended Prompt Caching

Open `.vscode/settings.json` or VS Code Settings and add:

```json
{
  "github.copilot.chat.caching.enabled": true,
  "prompt_cache_retention": "24h"
}
```

> [!IMPORTANT]
> Cache retention "24h" (24 hours) moves cache from fast GPU memory to GPU-local storage. Default 5-10 minute cache expires during breaks; extended retention survives across sessions.

#### Step 3: Enable Output Compression

Add to `settings.json`:

```json
{
  "chat.tools.compressOutput.enabled": true,
  "chat.tools.terminal.outputDeltas": true
}
```

**What this does**:
- `compressOutput`: Removes noise from terminal output
- `outputDeltas`: Sends only new output lines, not full history

**Token savings**: 40-70% on tool output per turn

#### Step 4: Enable Background Todo Agent (Optional)

For complex multi-step tasks, enable lightweight todo tracking:

```json
{
  "github.copilot.chat.agent.backgroundTodoAgent.enabled": true
}
```

**When to use**: Large refactorings, multi-module implementations, long-running agent sessions

#### Step 5: Configure File Exclusions

Exclude token-heavy directories from Copilot context. Add to `.vscode/settings.json`:

```json
{
  "github.copilot.chat.exclusions": [
    "**node_modules/**",
    "**/.git/**",
    "**/dist/**",
    "**/build/**",
    "**/*.lock",
    "**/package-lock.json",
    "**/yarn.lock",
    "**/.env*",
    "**/coverage/**"
  ]
}
```

**Why**: These directories contain thousands of files and tokens that rarely affect code quality analysis.

**Token savings**: 10-30% depending on project structure

### Settings File Locations

**Windows**:
```
%APPDATA%\Code\User\settings.json
```

**macOS**:
```
~/Library/Application Support/Code/User/settings.json
```

**Linux**:
```
~/.config/Code/user/settings.json
```

---

## Billing & Cost Model Reference

### Official Pricing Model

GitHub Copilot pricing is based on token consumption converted to AI Credits:

**1 AI Credit = $0.01 USD**

Tokens are priced per 1 million tokens, with rates varying by model.

### Token Types & Pricing Structure

| Token Type | Description | Pricing |
|-----------|-------------|---------|
| **Input Tokens** | Context sent to model (system instructions, tools, files, history) | Model-dependent rate (e.g., $2.50 per 1M for GPT-5.6 Luna) |
| **Output Tokens** | Content generated by model (typically higher rate) | Model-dependent rate (e.g., $10.00 per 1M for GPT-5.6 Luna) |
| **Cached Input Tokens** | Reused context from previous interactions | 10x cheaper than uncached input (e.g., $0.25 per 1M for GPT-5.6 Luna) |

### Components That Consume Tokens

Every Copilot request includes multiple components, each consuming tokens:

1. **System Instructions**: Agent behavior, rules, constraints (~200-500 tokens)
2. **Tool Definitions**: API schemas for available tools (~1,500-3,000 tokens)
3. **Conversation History**: Previous messages in session (~1,000-8,000 tokens)
4. **Repository Context**: File structure, relevant code (~2,000-10,000 tokens)
5. **User Prompt**: Your actual request (~50-2,000 tokens)

### Cost Reduction Hierarchy

From highest to lowest impact:

1. **Cache reuse**: 90% savings on prefix tokens
2. **Model selection**: 40-60% per task
3. **Tool optimization**: 8-9% tokens
4. **Output compression**: 40-70% on tool output
5. **Prompt optimization**: 50-70% with concision

### Official Pricing Reference

For current per-model pricing and token rates, refer to:
**[GitHub Copilot: Models and Pricing](https://docs.github.com/en/copilot/reference/copilot-billing/models-and-pricing)**

---

# PART 2: CORE FEATURES

## Prompt Caching Strategy

### Understanding Prompt Caching

Prompt caching is the single most powerful cost-reduction feature available in GitHub Copilot. By reusing context across consecutive turns, cached tokens cost **90% less** than uncached tokens.

### How Caching Works: Complete Lifecycle

#### Turn 1: Initial Request (No Cache)

**Scenario**: User starts new chat about authentication

```
User Message: "Help me implement user authentication"

Context Sent to Model:
├─ System Instructions (500 tokens)
├─ Tool Definitions (2,500 tokens)
├─ Repository Context (5,000 tokens)
└─ User Message (300 tokens)

Total: 8,300 tokens
Cost: 100% full price (no cache available yet)
```

**What the model does**: Computes internal representations ("embeddings") for all 8,300 tokens and stores the state in GPU memory.

**GPU Memory State**: The "prefix" (system instructions + tools + context) is cached.

---

#### Turn 2: Follow-up Message - Cache Miss Scenario

**Scenario**: User takes a 30-minute break, then asks follow-up question

```
User Message: "Now add email verification"

Cache Check: ❌ EXPIRED
Reason: Default cache TTL (Time To Live) = 5-10 minutes
         Cache evicted from GPU memory after 30 minutes

Result: Model must recompute full prefix
├─ System Instructions (500 tokens) ← WASTED
├─ Tool Definitions (2,500 tokens) ← WASTED
├─ Repository Context (5,000 tokens) ← WASTED
└─ User Message (200 tokens)

Total: 8,200 tokens (same prefix, recomputed)
Cost: 100% full price again

Cumulative Cost (Turn 1 + 2): 16,500 tokens charged at full rate
```

**Problem**: You paid 100% for the same prefix twice.

---

#### Turn 3: Follow-up Message - Cache Hit Scenario

**Scenario**: Same as Turn 2, but with extended cache retention enabled

**Prerequisite**: `"prompt_cache_retention": "24h"` configured in settings

```
User Message: "Add two-factor authentication support"

Cache Check: ✅ HIT!
Reason: Extended retention enabled (24 hours)
        Prefix still in cache after 30 minutes

Result: Model reuses cached prefix
├─ System Instructions (500 tokens) ← CACHED (90% cheaper)
├─ Tool Definitions (2,500 tokens) ← CACHED (90% cheaper)
├─ Repository Context (5,000 tokens) ← CACHED (90% cheaper)
└─ User Message (250 tokens) ← NEW (full price)

Total Tokens Sent: 250
Total Tokens Charged:
  ├─ Cached prefix: 8,000 × $0.25M = $0.02 (for GPT-5.6 Luna)
  └─ New message: 250 × $2.50M = $0.00063

Cost: 95% reduction compared to Turn 1 or 2
```

---

#### Turns 4-10: Continuous Cache Hits

**Pattern**: Every subsequent turn in same session reuses the cached prefix

```
Turn 3: New message (250 tokens) + 8,000 cached ✅
Turn 4: New message (180 tokens) + 8,000 cached ✅
Turn 5: New message (320 tokens) + 8,000 cached ✅
Turn 6: New message (210 tokens) + 8,000 cached ✅
Turn 7: New message (290 tokens) + 8,000 cached ✅
Turn 8: New message (240 tokens) + 8,000 cached ✅
Turn 9: New message (260 tokens) + 8,000 cached ✅
Turn 10: New message (270 tokens) + 8,000 cached ✅

Total New Tokens: 2,220 tokens (sent at full price)
Total Cached: 64,000 tokens (charged at 90% discount)
```

### Cumulative Savings Example

**Scenario**: 10-turn authentication implementation session

**Without caching** (baseline):
```
Every turn sends full context (8,300 + new message)
Turn 1-10: ~85,000 tokens charged at 100%
Cost: $0.42
```

**With caching** (extended 24h retention):
```
Turn 1: 8,300 tokens (100%)
Turns 2-10: 2,220 tokens (100% for new messages only)
Cached prefix in Turns 2-10: 72,000 tokens (10% charged)

Total Cost: ~$0.065
Savings: 84% reduction
```

---

### Cache Invalidation: When Cache Breaks

Cache is automatically cleared when:

1. **System instructions change** (in `.instructions.md` or `.agent.md`)
2. **Tool definitions change** (tools added/removed from context)
3. **Conversation model changes** (switch between GPT-5.6 and Claude)
4. **Repository context changes** significantly
5. **Cache TTL expires** (default 5-10 min, extended 24h)

### Maximizing Cache Hits

#### Strategy 1: Keep Prefix Stable

**Do**:
- Use same system instructions within a session
- Maintain consistent tool set
- Keep repository context unchanged within a task

**Don't**:
- Change instructions mid-session
- Add/remove tools frequently
- Switch models repeatedly

**Impact**: Each change breaks cache, resetting to full price for next turn

#### Strategy 2: Use Extended Retention

**Configuration** (already set in Prerequisites section):
```json
{
  "prompt_cache_retention": "24h"
}
```

**Effect**: Cache survives across breaks and long conversations (survives 24 hours)

#### Strategy 3: Reuse Sessions for Related Work

**Example**:
```
Session 1: Implement authentication features (cache warm)
Continue in Session 1 for email verification (cache reused)
Continue in Session 1 for password reset (cache reused)

Total: 3 related features, 1 cached prefix = 70% savings
```

**vs.**
```
Session 1: Implement authentication
Session 2: Implement email (cache rebuild, full price)
Session 3: Implement password (cache rebuild, full price)

Total: 3 related features, 3 cache rebuilds = no savings
```

#### Strategy 4: Start New Sessions for Unrelated Work

**When to split**:
- Different project entirely
- Different task type (debugging vs. feature)
- Different instruction set
- Scope creep detected

**Why**: Related work benefits from cache; unrelated work pollutes the prefix

---

### Monitoring Cache Performance

#### Command: `/chronicle:cost-tips`

Run inside Copilot Chat to analyze usage patterns:

```
/chronicle:cost-tips
```

**Output includes**:
- Cache hit rate (percentage of requests using cached prefix)
- Inefficient session patterns
- Specific optimization opportunities
- Token savings potential

**Example output**:
```
Cache Analysis:
├─ Cache hit rate: 45% (opportunity to improve)
├─ Unnecessary session splits: 3 sessions could be combined
├─ Model switches breaking cache: 2 instances
└─ Potential savings: 35% if optimizations applied
```

**When to run**: After every major feature, or weekly

---

### Configuration Deep Dive

#### File Path: `settings.json`

**Location**:
- **Windows**: `%APPDATA%\Code\User\settings.json`
- **macOS**: `~/Library/Application Support/Code/User/settings.json`
- **Linux**: `~/.config/Code/user/settings.json`

#### Required Settings

```json
{
  "github.copilot.chat.caching.enabled": true,
  "prompt_cache_retention": "24h",
  "chat.tools.compressOutput.enabled": true,
  "chat.tools.terminal.outputDeltas": true
}
```

#### What Each Setting Does

| Setting | Value | Effect |
|---------|-------|--------|
| `caching.enabled` | `true` | Activates prompt caching |
| `cache_retention` | `"24h"` | Extends cache to 24 hours (survives breaks) |
| `compressOutput.enabled` | `true` | Removes terminal output noise |
| `outputDeltas` | `true` | Sends only new output, not full history |

---

## Copilot Fusion: Intelligent Model Orchestration

### What Is Copilot Fusion?

Copilot Fusion is an intelligent orchestrator that automatically selects the best model or execution pattern for each task, optimizing for cost, quality, and latency simultaneously.

**Key principle**: Instead of you choosing one model for everything, Fusion dynamically routes work to the best model for that specific task.

### Why Fusion Matters

**Manual model selection** (old approach):
```
You pick GPT-5.6 Sol for every task
├─ Simple refactoring with Sol: High cost, overkill capability
├─ Complex debugging with Sol: Good, but expensive
└─ Documentation with Sol: Wasteful
```

**Copilot Fusion** (new approach):
```
Task 1 (Simple refactoring): Automatically use Luna (cheaper)
Task 2 (Complex debugging): Automatically escalate to Sol (capable)
Task 3 (Documentation): Automatically use Haiku (fastest)
Result: Optimal cost + quality
```

---

### Execution Patterns

Copilot Fusion supports four execution patterns, each optimized for different scenarios:

#### Pattern 1: Single Model (Default)

**Use case**: Straightforward tasks where one model is sufficient

**How it works**:
```
User Request
    ↓
Analyze complexity
    ↓
Select appropriate model
    ↓
Execute and return
```

**Cost**: Baseline for selected model

**Speed**: ~2 seconds

**Best for**: Daily coding tasks, simple debugging, refactoring

---

#### Pattern 2: Cascade

**Use case**: Tasks where a cheap model can draft, then an expert reviews

**How it works**:
```
User Request
    ↓
Draft with efficient model (Luna)
    ↓
Expert review with capable model (Sol)
    ↓
If acceptable: Return (cost = Luna + Sol)
    ↓
If needs improvement: Expert refines
```

**Cost**: +15-30% vs. single model (Luna alone would fail)

**Speed**: ~4-6 seconds

**Best for**: Novel features, architecture decisions, complex refactoring

**Example**:
```
Request: "Implement authentication with OAuth2"

Step 1 (Luna): Draft OAuth2 implementation (~1,000 tokens)
Step 2 (Sol): Review and suggest improvements (~500 tokens)
Step 3 (Luna): Refine based on feedback (~300 tokens)

Total cost: 1,800 tokens (vs 3,000 with Sol alone)
Savings: 40%
```

---

#### Pattern 3: Critique

**Use case**: Complex problems requiring multiple perspectives

**How it works**:
```
User Request
    ↓
Model A solves independently
    ↓
Model B solves independently (parallel)
    ↓
Model C solves independently (parallel)
    ↓
Aggregate all solutions
    ↓
Return best approach
```

**Cost**: +40-60% vs. single model

**Speed**: ~6-8 seconds (parallel execution)

**Best for**: Critical architecture decisions, security analysis, performance optimization

**Example**:
```
Request: "Design user authentication system"

GPT-5.6 Sol approach: OAuth2-based
Claude Opus approach: JWT-based
Gemini 4 Argon approach: Hybrid multi-factor

Fusion aggregates: Recommends hybrid approach with specific rationale
Result: Better decision, justified by multiple expert perspectives
```

---

#### Pattern 4: Single (Internal Use)

**Note**: Available only in VS Code inline chat, automatically selected for quick suggestions.

---

### Effort Levels

Copilot Fusion supports three configurable effort levels:

| Level | Use Case | Cost Multiplier | Speed | When to Use |
|-------|----------|-----------------|-------|------------|
| **Standard** | Most day-to-day work | 1x | ~3-4s | Default choice |
| **High** | Substantial tasks | 1.3-1.5x | ~4-6s | Multi-file changes, complex debugging |
| **Max** | Critical decisions | 1.5-2x | ~6-8s | Architecture, security, major refactors |

### Cost Advantages

> [!NOTE]
> **Current incentive** (through November 30, 2026): 30% discount on all Copilot Fusion model costs

**Real-world cost comparison**:

| Approach | Cost | Quality | Cache Preservation |
|----------|------|---------|-------------------|
| Manual Sol all tasks | $1.00 | Good | Breaks on switch |
| Manual Luna all tasks | $0.30 | Fair | Stable |
| Copilot Fusion Standard | $0.60 | Excellent | ✅ Preserved |

**Why Fusion Standard wins**: Optimal cost-quality tradeoff AND cache preservation

### Cache Preservation with Fusion

**Key advantage**: Fusion keeps your main conversation on the same model, preventing cache breaks.

**Example**:
```
Session with Fusion:
├─ Turn 1: Simple task → Luna (lightweight)
├─ Turn 2: Simple task → Luna (SAME = cache hit) ✅
├─ Turn 3: Complex task → Fusion escalates to Sol for that turn
├─ Turn 4: Simple task → Back to Luna (cache still warm) ✅
└─ Result: Cache stays warm despite task complexity variation
```

**Without Fusion** (manual model selection):
```
└─ Turn 1: You pick Sol
├─ Turn 2: You pick Luna (cache broken, 100% reset)
├─ Turn 3: You pick Sol (cache broken again)
└─ Result: Continuous cache invalidation, 80% higher cost
```

---

### How to Use Copilot Fusion

#### In VS Code

1. Open **Copilot Chat**
2. Click the **model selector** (shows current model name)
3. Search for **"Fusion"** in model picker
4. Select **Copilot Fusion**
5. Choose effort level:
   - **Fusion Standard** (recommended, most tasks)
   - **Fusion High** (substantial work)
   - **Fusion Max** (critical decisions)

#### In GitHub.com

1. Navigate to [github.com/copilot](https://github.com/copilot)
2. Copilot Fusion is the default orchestrator
3. Select effort level via interface

#### Via Command Line (Copilot CLI)

```bash
# Standard effort (default)
copilot "your task description"

# High effort for complex work
copilot -e high "complex task description"

# Max effort for critical decisions
copilot -e max "critical architecture question"
```

---

### Monitoring Fusion Decisions

After Fusion completes a task:

1. **Hover below the response** to see execution details
2. **Check the effort level** (Standard/High/Max)
3. **View which models were used** (single vs. cascade vs. critique)
4. **Note response time** (fast = single model, slower = multiple stages)

This helps you understand why certain tasks took longer or cost more.

---

### Best Practices for Copilot Fusion

1. **Default to Standard**: Handles 80% of work efficiently
2. **Use High for substantial tasks**: Multi-file refactoring, complex debugging
3. **Reserve Max for critical decisions**: Architecture, security, major refactors
4. **Don't override Fusion**: Let it select the execution pattern automatically
5. **Trust the orchestration**: Fusion considers task complexity and cache state

---

## Subagents & Parallel Task Execution

### What Are Subagents?

Subagents are specialized AI agents that work independently on focused subtasks while a main orchestrator coordinates overall work. This enables large tasks to be broken down and executed in parallel.

**Core principle**: Main agent orchestrates work → Subagents execute in parallel → Results are aggregated

### Main Agent vs. Subagents

| Aspect | Main Agent | Subagent |
|--------|-----------|----------|
| **Context** | Full conversation history, rich context | Only focused task context |
| **Session** | Primary session with preserved cache | Separate session, independent cache |
| **Model** | Typically capable, focused | Often lighter model, cost-optimized |
| **Responsibility** | Orchestrate, coordinate, final review | Execute assigned subtask independently |
| **Token Cost** | Higher (full context) | Lower (focused scope) |
| **Cache Impact** | Main cache preserved across tasks | Independent, no interference |

---

### When to Use Subagents

Subagents provide maximum value when:

#### Scenario 1: Large Complex Tasks with Parallelizable Work

**Example**: Refactoring module with 5 independent components

**Without subagents** (sequential):
```
Main agent handles all:
├─ Component 1: Analyze, modify, test → 5,000 tokens (10 min)
├─ Component 2: Analyze, modify, test → 4,500 tokens (9 min)
├─ Component 3: Analyze, modify, test → 4,200 tokens (8 min)
├─ Component 4: Analyze, modify, test → 4,800 tokens (10 min)
└─ Component 5: Analyze, modify, test → 5,100 tokens (11 min)

Total: 23,600 tokens, 48 minutes (sequential)
```

**With subagents** (/fleet):
```
Main agent orchestrates:
├─ Subagent 1: Component 1 → 3,000 tokens ┐
├─ Subagent 2: Component 2 → 2,800 tokens │ Parallel
├─ Subagent 3: Component 3 → 2,900 tokens │ (12 min total)
├─ Subagent 4: Component 4 → 3,100 tokens │
└─ Subagent 5: Component 5 → 3,200 tokens ┘

Main agent review: → 2,000 tokens (consolidate results)

Total: 17,000 tokens, 12 minutes (4x faster)
Savings: 28% tokens, 75% time
```

#### Scenario 2: Automated Multi-Environment Testing

**Example**: Test feature across multiple environments in parallel

```
/fleet "Run tests for payment feature in parallel:
- Unit tests (src/payment/tests)
- Integration tests (integration/payment)
- E2E tests (cypress/e2e/payment)
- Performance tests (load/payment-api)"

Subagents (parallel):
├─ Subagent 1: Unit tests (3 min)
├─ Subagent 2: Integration tests (5 min)
├─ Subagent 3: E2E tests (7 min)
├─ Subagent 4: Performance tests (6 min)
└─ Total execution: 7 minutes (vs 21 sequential)
```

#### Scenario 3: Dependency Updates Across Multiple Services

**Example**: Update packages in independent microservices

```
/fleet "Update npm dependencies across 4 services:
- Auth service (src/auth)
- API service (src/api)
- Payment service (src/payment)
- Notification service (src/notifications)"

Subagents execute independently (parallel):
├─ Update auth service dependencies
├─ Update api service dependencies
├─ Update payment service dependencies
└─ Update notification service dependencies

Result: 4x faster, reduced total tokens via parallelization
```

---

### Using the `/fleet` Command

The `/fleet` command invokes parallel subagent execution.

#### Syntax

```bash
/fleet "Your complex task description"
```

#### How It Works

```
Step 1: Main agent analyzes task
        ↓
Step 2: Identifies parallelizable subtasks
        ↓
Step 3: Detects dependencies between subtasks
        ↓
Step 4: Delegates independent tasks to subagents
        ↓
Step 5: Subagents execute simultaneously
        ↓
Step 6: Main agent orchestrates and reviews results
```

---

#### Example 1: Multi-Module Refactoring

**Task**: Refactor authentication module across 4 files

```bash
/fleet "Refactor user authentication module:
Update src/auth/login.ts, src/auth/logout.ts,
src/auth/verify.ts, and src/auth/refresh.ts
to use new JWT token format"
```

**Execution**:
```
Main agent:
├─ Analyzes all 4 files
├─ Determines they can be updated independently
└─ Delegates to 4 subagents

Subagents (parallel execution):
├─ Subagent 1: Refactor login.ts (PARALLEL)
├─ Subagent 2: Refactor logout.ts (PARALLEL)
├─ Subagent 3: Refactor verify.ts (PARALLEL)
└─ Subagent 4: Refactor refresh.ts (PARALLEL)

Main agent (after parallel work):
├─ Reviews all changes
├─ Ensures token format consistency
└─ Returns consolidated refactoring

Efficiency: 60-70% faster, lower cost per task
```

---

#### Example 2: Dependency Management Workflow

**Task**: Update all npm dependencies with verification

```bash
/fleet "Update project dependencies:
1. Update package.json to latest versions
2. Run npm install
3. Update all import statements if needed
4. Run test suite to verify changes"
```

**Execution**:
```
Main agent creates dependency graph:
├─ Task A: Update production dependencies (independent)
├─ Task B: Update dev dependencies (independent)
└─ Task C: Run tests (depends on A & B)

Parallel execution:
├─ Subagent 1: Update prod deps ┐
├─ Subagent 2: Update dev deps  │ Execute in parallel
└─ Wait for both to complete    ┘
        ↓
├─ Subagent 3: Run tests (waits for A & B)
        ↓
Main agent: Review results and consolidate

Result: Optimal execution order, cost-efficient
```

---

### Cost Analysis

#### When Subagents Save Money

Subagents reduce cost when:

| Condition | Token Savings | When It Applies |
|-----------|---------------|-----------------|
| Parallelizable independent tasks | 20-40% | Multi-file refactoring, multi-service updates |
| Tasks that don't share context | 15-30% | Separate features, different code areas |
| Long task breakdown efficiently | 10-25% | Well-scoped independent subtasks |

#### When Subagents Cost More

Subagents cost MORE (use main agent instead) when:

| Condition | Token Overhead | Example |
|-----------|---|---------|
| Sequential dependent tasks | +10-20% | Task B depends on Task A output |
| Complex coordination needed | +15-30% | Subtasks require frequent main agent review |
| Highly interdependent work | +20-40% | Changes in one module affect all others |

---

### Model Assignment Strategy for Subagents

**Principle**: Main agent uses capable model, subagents use cheaper models for focused tasks.

**Configuration**:
```
Main agent: GPT-5.6 Sol (orchestration, decision-making)
Subagent 1-5: GPT-5.6 Luna (focused execution, lower cost)

Cost impact: 40-50% savings vs. all tasks using Sol
```

---

## Tool Optimization & Deferred Loading

### The Tool Problem

Historically, every tool was loaded into context on every request with full definition:

```
Old approach (all tools preloaded):
├─ Tool 1: read file (full JSON schema, ~200 tokens)
├─ Tool 2: edit file (full JSON schema, ~300 tokens)
├─ Tool 3: run terminal (full JSON schema, ~400 tokens)
├─ Tool 4: search code (full JSON schema, ~250 tokens)
├─ Tool 5: git commands (full JSON schema, ~350 tokens)
├─ Tool 6: create file (full JSON schema, ~280 tokens)
└─ [20+ more tools...]

Total tool definitions: 4,500-6,000 tokens per request
```

**Problem**: Agent never uses most tools, but pays the token cost for all of them.

---

### Solution: Deferred Tool Loading (Tool Search)

**New approach**: Model sees lightweight metadata first, loads full schemas only when needed

```
Turn 1 (Initial request):
├─ Model sees tool names + short descriptions only (~200 tokens)
└─ Model analyzes what's actually needed

Turn 2 (If tool needed):
├─ Full schema loads into context (~300 tokens)
├─ Model uses the tool
└─ Schema stays for session (cached)

Result: 8-9% token reduction compared to preloading all tools
```

---

### How Tool Search Works

#### Step 1: Lightweight Tool Listing

Model receives minimal tool metadata:

```json
{
  "available_tools": [
    {"name": "read_file", "description": "Read file contents"},
    {"name": "edit_file", "description": "Edit file contents"},
    {"name": "terminal", "description": "Run terminal commands"},
    {"name": "search_code", "description": "Search codebase"}
  ]
}
```

**Size**: ~200 tokens (vs 4,500+ for full schemas)

#### Step 2: Model Determines Needs

Model analyzes your request and decides which tools to use.

#### Step 3: On-Demand Schema Loading

If model needs a tool:

```
Model: "I need to read src/auth.ts"
↓
System loads full schema for read_file only
↓
Model uses the tool with full parameter details
```

---

### Performance Gains

| Model | Token Reduction |
|-------|-----------------|
| GPT-5.6 Luna | 8.61% |
| GPT-5.4 | 9.81% |
| Claude Sonnet 5.5 | 7.4% |
| Gemini 3.8 Flash | 8.2% |

**Average savings**: 8-9% per session with deferred tool loading enabled

---

### Best Practices

1. **Only include relevant tools**: Exclude tools not applicable to current task
2. **Keep tool descriptions concise**: Short descriptions use fewer tokens than verbose ones
3. **Use tool search by default**: Enabled automatically, no configuration needed
4. **Monitor tool usage**: Use `/chronicle:cost-tips` to see which tools are used

---

## Tool Output Compression

### Overview

Tool output compression reduces the size of command outputs sent to the model, saving 40-70% of tool output tokens without losing important information.

Two complementary features work together:

1. **Compression** (removes noise)
2. **Deltas** (avoids repetition)

---

### Feature 1: Output Compression (`compressOutput`)

**Purpose**: Automatically filter and compress terminal output before sending to model

**Configuration**:
```json
{
  "chat.tools.compressOutput.enabled": true
}
```

**How it works**:
- Removes redundant log lines
- Truncates verbose output sections
- Preserves error messages and key information
- Filters debugging noise

**Example**:

Before compression (500 lines):
```
[INFO] Starting build process...
[TRACE] Initializing compiler...
[DEBUG] Loading configuration from /path/to/config...
[DEBUG] Analyzing 2,847 source files...
[TRACE] Building AST for file 1...
[TRACE] Building AST for file 2...
[...1000 more trace lines...]
[ERROR] Unexpected token at line 42
[ERROR] Stack trace: ...
[INFO] Build failed
```

After compression (20 lines):
```
[INFO] Starting build process...
[ERROR] Unexpected token at line 42
[Stack trace summary...]
[INFO] Build failed
```

**Token Impact**: 20-40% reduction in output tokens

---

### Feature 2: Output Deltas (`outputDeltas`)

**Purpose**: Send only changed output lines, not full history

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
Turn 1: Run build command
├─ Full output: 1,000 lines
└─ Sent to model: 1,000 lines

Turn 2: Build completes with final output
├─ New output: 5 lines (final messages)
└─ Sent to model: 5 lines (delta only)

Turn 3: Check results
├─ New output: 3 lines
└─ Sent to model: 3 lines (delta only)

Total tokens sent to model: 1,008 lines
Without deltas: 2,008 lines (100% repetition)
```

**Token Impact**: 70-90% reduction in repeated output tokens

---

### Combined Benefit

**Using both compression AND deltas together**:

| Technique | Feature | Savings |
|-----------|---------|---------|
| Compression alone | Removes noise | 20-40% |
| Deltas alone | Avoids repetition | 70-90% |
| Both together | Compression + no repetition | 40-70% |

**Total savings**: 40-70% reduction in tool output tokens per turn

---

### Configuration

Add to `settings.json`:

```json
{
  "chat.tools.compressOutput.enabled": true,
  "chat.tools.terminal.outputDeltas": true
}
```

---

## Background Todo Agent

### What It Is

An experimental lightweight agent that manages task progress and todo tracking independently, separate from the main conversation model.

**Purpose**: Reduce tokens spent by main model on progress tracking, freeing capacity for implementation work.

---

### Why It Matters

**Before** (without Background Todo Agent):
```
Main model spends tokens on:
├─ Creating todo lists (~300 tokens)
├─ Updating todo status after each step (~200 tokens)
├─ Re-reading todos for context (~250 tokens)
└─ Managing task tracking (~200 tokens)

Total overhead: ~950 tokens per complex task
```

**After** (with Background Todo Agent):
```
Background agent handles:
├─ Creating todo lists (separate session)
├─ Updating todo status (independent)
├─ Maintaining task tracking (no main model overhead)

Main model focuses on:
└─ Implementation work only
   (saves ~950 tokens per task)
```

---

### Use Cases

Background Todo Agent is most beneficial for:

1. **Multi-step development tasks**: 5+ step implementations
2. **Long-running agent sessions**: Sessions >30 turns
3. **Complex projects**: Many interdependent tasks
4. **Team collaboration**: Shared progress tracking
5. **Refactoring large codebases**: Tracking changes across files

**Not beneficial for**: Simple quick fixes, single-turn questions

---

### Enabling Background Todo Agent

**Configuration**:
```json
{
  "github.copilot.chat.agent.backgroundTodoAgent.enabled": true
}
```

**Where to add**: `.vscode/settings.json` or VS Code Settings UI

---

### How It Works

```
User starts complex task:
"Refactor payment module"
        ↓
Background agent creates todo structure:
├─ [ ] Analyze current implementation
├─ [ ] Identify refactoring opportunities
├─ [ ] Update payment processor
├─ [ ] Update payment gateway
├─ [ ] Update tests
└─ [ ] Verify end-to-end

Main model works on implementation:
└─ Focuses on actual code changes (saves ~950 tokens)

As main model completes each step:
├─ Main model: "Completed payment processor refactoring"
└─ Background agent: Marks step done automatically

Result: Main model has more tokens for actual work
```

---

### Token Impact

| Task Size | Tokens Saved | Use Background Todo |
|-----------|--------------|-------------------|
| 3-step task | 100-200 | Optional |
| 5-step task | 300-500 | Recommended |
| 10-step task | 800-1,200 | Highly recommended |
| 20+ step task | 1,500-2,500 | Critical |

---

# PART 3: ADVANCED STRATEGIES

## Model Selection Strategy

### Core Principle

**Use the least capable model that solves the problem effectively.**

This is the single highest-impact model optimization. Using GPT-5.6 Sol for simple refactoring costs 40-60% more than using Claude Haiku 4.5, with no quality improvement.

---

### Model Categories

#### Category 1: Reasoning Models (Highest Capability, Highest Cost)

**When to use**: Problems requiring deep analysis, multi-step reasoning, novel architecture

**Best for**:
- Architecture decisions and system design
- Complex debugging with multiple variables
- Security vulnerability analysis
- Performance optimization with tradeoffs
- Critical business logic

**Models**:
- GPT-5.3-Codex (extended autonomous coding)
- GPT-5.6 Sol (complex reasoning)
- Claude Opus 4.8 (advanced analysis)
- Gemini 4 Argon (frontier reasoning)

**Cost multiplier**: 3-6x vs. lightweight models

---

#### Category 2: Balanced Models (Medium Capability, Medium Cost)

**When to use**: Mixed-complexity work where both reasoning and execution matter

**Best for**:
- Feature implementation with clear design
- Multi-file refactoring
- Bug fixes requiring analysis
- Integration work
- Regular development tasks

**Models**:
- GPT-5.6 Terra (balanced general-purpose)
- Claude Sonnet 5.5 (well-rounded)
- Gemini 3.8 Flash (efficient)

**Cost multiplier**: 1.5-2x vs. lightweight models

---

#### Category 3: Lightweight Models (Lower Capability, Low Cost)

**When to use**: Well-scoped tasks where the approach is clear

**Best for**:
- Refactoring and formatting
- Documentation and comments
- Code cleanup
- Simple bug fixes (where root cause is known)
- Routine changes
- Quick questions

**Models**:
- GPT-5.6 Luna (efficient, fast)
- Claude Haiku 4.5 / 5.5 (very fast, very cheap)
- Claude Fable 5 / 5.1 (routine tasks)

**Cost multiplier**: 1x baseline

---

### Model Comparison & Selection Matrix

| Model | Strength | Best For | Cost | Speed | Reasoning |
|-------|----------|----------|------|-------|-----------|
| **GPT-5.3-Codex** | Autonomous coding | Long-horizon autonomous tasks | Very High | 8-10s | Advanced |
| **GPT-5.6 Sol** | Complex reasoning | Architecture, deep debugging | High | 4-6s | Advanced |
| **GPT-5.6 Terra** | Balanced execution | Feature implementation | Medium-High | 3-4s | Good |
| **GPT-5.6 Luna** | Speed & efficiency | Quick tasks, refactoring | Low | 2-3s | Adequate |
| **Claude Opus 4.8** | Advanced reasoning | Complex analysis, system design | High | 5-7s | Advanced |
| **Claude Sonnet 5.5** | Balanced performance | Regular development | Medium | 3-4s | Good |
| **Claude Haiku 4.5** | Cost efficiency | Routine changes, quick fixes | Very Low | 1-2s | Basic |
| **Claude Fable 5.1** | Quick execution | Formatting, documentation | Very Low | 1-2s | Basic |
| **Gemini 4 Argon** | Complex reasoning | Advanced problems | High | 4-5s | Advanced |
| **Gemini 3.8 Flash** | Speed & cost | Quick responses | Low | 2-3s | Adequate |

---

### Selection Flowchart

```
Task received
    ↓
Does it require deep reasoning or architecture?
├─ Yes → Use reasoning model (Sol, Opus 4.8, Gemini 4 Argon)
└─ No → Continue
    ↓
Is the approach clear and straightforward?
├─ Yes → Use lightweight model (Luna, Haiku 4.5, Fable)
└─ No → Use balanced model (Terra, Sonnet 5.5, Gemini 3.8 Flash)
```

---

### Cost-Benefit Examples

#### Example 1: Simple Refactoring

**Task**: Rename variable from `userId` to `userID` across 5 files

**Wrong choice**: GPT-5.6 Sol
- Cost: 5,000 tokens × $4.00 per 1M = $0.020
- Quality: Perfect, but overqualified
- Time: 4 seconds

**Right choice**: Claude Haiku 4.5
- Cost: 1,200 tokens × $0.001 per 1M = $0.0000012
- Quality: Perfect
- Time: 2 seconds
- **Savings**: 97%, same result

---

#### Example 2: Complex Bug Investigation

**Task**: Debug authentication failure in OAuth flow with multiple dependencies

**Wrong choice**: Claude Haiku 4.5
- Cost: $0.000001
- Quality: Insufficient (too simple)
- Time: 2 seconds, but wrong answer

**Right choice**: GPT-5.6 Sol
- Cost: 8,000 tokens × $4.00 per 1M = $0.032
- Quality: Thorough analysis
- Time: 5 seconds, correct answer
- **Value**: 32x cost for correct solution (worth it)

---

#### Example 3: Feature Implementation

**Task**: Add pagination to user list API

**Wrong choice**: Reasoning model (Sol)
- Cost: 12,000 tokens × $4.00 = $0.048
- Quality: Excellent, but overengineered
- Time: 6 seconds

**Optimal choice**: Balanced model (Terra)
- Cost: 7,000 tokens × $1.50 = $0.0105
- Quality: Good, complete implementation
- Time: 3 seconds
- **Savings**: 78%, sufficient quality

---

### Reasoning Level Configuration

Some models support configurable reasoning effort:

| Level | Use Case | Token Impact |
|-------|----------|--------------|
| **Regular** | Default, use for most tasks | Baseline |
| **High** | Uncommon hard problems | +40-60% tokens |
| **Extreme** | Very difficult problems | +100-200% tokens |

**Best practice**: Start with regular effort, increase only when necessary.

---

### Using Copilot Fusion for Model Selection

**Recommended**: Use Copilot Fusion instead of manual selection for most work.

Fusion automatically:
- Selects appropriate model for task complexity
- Routes along cache boundaries (preserves cache)
- Provides 30% cost discount (through Nov 30, 2026)
- Ensures consistent quality

**Manual selection** only when you need specific model behavior.

---

## Session Architecture Patterns

### Core Principle

**Session design directly impacts cache effectiveness and total token consumption.**

Poor session design can cost 50-70% more tokens than optimal design.

---

### Session Design Patterns

#### Pattern 1: Project-Based Sessions (Recommended)

**How it works**:
```
Project A (UI redesign)
├─ Session 1: Design phase
├─ Session 2: Implementation
└─ Session 3: Testing

Project B (API rewrite)
├─ Session 4: Architecture
├─ Session 5: Implementation
└─ Session 6: Integration
```

**Benefits**:
- Focused context (repository, instructions, files)
- Cache remains warm for related work
- Clear boundaries between projects
- Easy to track progress

**Cache efficiency**: 70-80%

**When to use**: Primary pattern for most teams

---

#### Pattern 2: Task-Type Sessions

**How it works**:
```
Refactoring Session
├─ Refactor auth module
├─ Refactor payment module
├─ Refactor API layer

Debugging Session
├─ Investigate login failure
├─ Fix memory leak
├─ Optimize slow query

Documentation Session
├─ Write README
├─ Add code comments
├─ Update changelog
```

**Benefits**:
- Same instructions and tools for all tasks in type
- Maximum cache hits within type
- Specialized instructions applicable
- Efficient for routine work

**Cache efficiency**: 65-75%

**When to use**: Routine maintenance tasks, team documentation

---

#### Pattern 3: Time-Boxed Sessions

**How it works**:
```
Monday Session (50 turns)
├─ Various related tasks

Wednesday Session (45 turns)
├─ Continue related work

Friday Session (30 turns)
├─ Different project entirely
```

**Rationale**: Keep sessions bounded to prevent cognitive load

**Session size guidelines**:
- <50 turns: Cache stays warm, coherent context
- 50-100 turns: Cache degradation starts, context becomes unfocused
- >100 turns: Significant cache pollution, context confusion likely

---

### When to Create New Sessions

| Scenario | Token Savings | Reason |
|----------|---------------|--------|
| Different project | 30-50% | Different repository context, different instructions |
| Different task type | 20-40% | Different agent rules, different problem domain |
| Long pause (>24h) | 20-30% | Cache expires, restarting is cleaner |
| Scope creep detected | 15-25% | Prevent history pollution from unrelated work |
| Unrelated quick task | 10-20% | One-off question doesn't benefit from history |

---

### When to Reuse Sessions

| Scenario | Benefit |
|----------|---------|
| Same project continuation | Cached context still relevant, cache warm |
| Related tasks (same codebase) | Shared instructions, same files |
| Iterative work | Building on previous context and decisions |
| Same conversation thread | Maintaining context quality, improving responses |

---

### Session Continuation Strategy

**Recommended approach**:

```
Session Start: Clear, focused title
"Implementing user authentication feature"
    ↓
Add context upfront:
├─ Relevant files
├─ Specific requirements
└─ Success criteria
    ↓
Execute work (Turns 1-50)
├─ Cache stays warm
├─ Context cumulative but focused
└─ Results documented
    ↓
When complete: Archive session and start new one for unrelated work
```

---

## Enterprise Implementation Workflows

### Workflow 1: Team Feature Development

**Scenario**: Team of 5 developers adding feature across 3 services

#### Phase 1: Research (15 minutes, 1 session)

```
Session: "Feature Research - Payment Processing"
├─ Model: Claude Haiku 4.5 (lightweight, cost-efficient)
├─ Task: Research current architecture, identify integration points
├─ Output: Architecture document, integration points list
├─ Tokens: ~2,000
└─ Cost: $0.002
```

#### Phase 2: Design (30 minutes, 1 session)

```
Session: "Feature Design - Payment Processing"
├─ Model: GPT-5.6 Sol (reasoning required)
├─ Task: Design implementation approach, API contracts
├─ Output: Design document, API specs, integration plan
├─ Tokens: ~8,000
└─ Cost: $0.032
```

#### Phase 3: Implementation (2-4 hours, parallel sessions)

```
Main Session: "Payment Feature Implementation"
├─ Model: Copilot Fusion (intelligent orchestration)
├─ Use /fleet for parallel component implementation
├─ Subagent 1: Service A updates
├─ Subagent 2: Service B updates
├─ Subagent 3: Service C updates
├─ Main agent: Coordinates and verifies integration
└─ Tokens: ~25,000 (50% savings vs sequential)

Cost: $0.060 (vs $0.120 sequential)
```

#### Phase 4: Testing (1 hour, parallel sessions)

```
Session: "Payment Feature Testing"
├─ Model: Lighter models for test automation
├─ Use /fleet for parallel test execution
├─ Subagent 1: Unit tests
├─ Subagent 2: Integration tests
├─ Subagent 3: E2E tests
├─ Main agent: Review and verify
└─ Tokens: ~12,000

Cost: $0.015
```

**Total cost**: $0.109 (vs $0.300+ unoptimized)
**Savings**: 64%

---

### Workflow 2: Large Refactoring

**Scenario**: Modernizing authentication system across entire codebase

#### Phase 1: Analysis (30 min)

```
Session: "Auth System Analysis"
├─ Model: GPT-5.6 Sol (understanding complexity)
├─ Analyze current auth implementation
├─ Identify all dependencies
├─ Output: Dependency map, refactoring plan
└─ Tokens: ~10,000
```

#### Phase 2: Modular Refactoring (/fleet)

```
Session: "Auth System Refactoring"
├─ Use Copilot Fusion (Standard effort)
├─ /fleet for parallel component updates:
│  ├─ Subagent 1: Token generation module
│  ├─ Subagent 2: Token validation module
│  ├─ Subagent 3: Session management
│  ├─ Subagent 4: Permission checking
│  └─ Subagent 5: OAuth integration
├─ Main agent: Verify consistency
└─ Tokens: ~20,000 (parallel saves 35%)
```

#### Phase 3: Integration Testing

```
Session: "Auth System Integration Testing"
├─ /fleet for parallel test suites
├─ Main agent: Verify all tests pass
└─ Tokens: ~8,000
```

**Total cost**: ~$0.105 (vs $0.400 unoptimized)
**Savings**: 74%
**Time**: 3 hours (vs 8+ hours sequential)

---

### Workflow 3: Debugging Production Issue

**Scenario**: Critical bug affecting payment processing

#### Phase 1: Problem Analysis (15 min)

```
Session: "Payment Processing Bug Analysis"
├─ Model: GPT-5.6 Sol (complex reasoning)
├─ Provide error logs, relevant code
├─ Analyze root cause
├─ Output: Root cause analysis, fix approach
└─ Tokens: ~6,000
```

#### Phase 2: Fix Implementation (30 min)

```
Session: "Payment Bug Fix"
├─ Model: GPT-5.6 Terra (balanced approach)
├─ Implement fix based on analysis
├─ Update related code
└─ Tokens: ~5,000
```

#### Phase 3: Verification (15 min)

```
Session: "Payment Bug Verification"
├─ Model: Claude Haiku 4.5 (test execution)
├─ Run test suite
├─ Verify production scenario
└─ Tokens: ~2,000
```

**Total cost**: $0.052 (vs $0.150 unoptimized)
**Savings**: 65%
**Time**: 1 hour (vs 3-4 hours unoptimized)

---

### ROI Calculation Template

**For any workflow**:

```
Unoptimized approach:
├─ Time: 8 hours
├─ Developer cost: $120 (at $15/hr)
├─ Token usage: 100,000
├─ Token cost: $0.30
├─ Total: $120.30

Optimized approach:
├─ Time: 4 hours
├─ Developer cost: $60
├─ Token usage: 30,000 (70% savings)
├─ Token cost: $0.09
├─ Total: $60.09

ROI:
├─ Time savings: 4 hours = $60
├─ Token savings: $0.21
├─ Total savings: $60.21
└─ Payoff: Immediate (50% cost reduction)
```

---

# PART 4: REFERENCE

## Configuration & Settings Reference

### Settings File Locations

| OS | Path |
|----|----|
| **Windows** | `%APPDATA%\Code\User\settings.json` |
| **macOS** | `~/Library/Application Support/Code/User/settings.json` |
| **Linux** | `~/.config/Code/user/settings.json` |

### Complete Optimization Configuration

```json
{
  "github.copilot.chat.caching.enabled": true,
  "prompt_cache_retention": "24h",
  "github.copilot.chat.agent.backgroundTodoAgent.enabled": true,
  "chat.tools.compressOutput.enabled": true,
  "chat.tools.terminal.outputDeltas": true,
  "github.copilot.chat.exclusions": [
    "**node_modules/**",
    "**/.git/**",
    "**/dist/**",
    "**/build/**",
    "**/*.lock",
    "**/package-lock.json",
    "**/yarn.lock",
    "**/.env*",
    "**/coverage/**"
  ]
}
```

### Configuration Explanation

| Setting | Value | Effect |
|---------|-------|--------|
| `caching.enabled` | `true` | Enable prompt caching |
| `cache_retention` | `"24h"` | Extend cache to 24 hours |
| `backgroundTodoAgent.enabled` | `true` | Enable lightweight todo tracking (optional) |
| `compressOutput.enabled` | `true` | Compress terminal output |
| `outputDeltas` | `true` | Send only output deltas |
| `exclusions` | `[array]` | Exclude token-heavy directories |

---

## Model Comparison Tables

### By Cost (Lowest to Highest)

| Rank | Model | Provider | Use Case | Cost Level |
|------|-------|----------|----------|------------|
| 1 | Claude Haiku 4.5 | Anthropic | Routine tasks | Very Low |
| 2 | Claude Haiku 5.5 | Anthropic | Quick responses | Very Low |
| 3 | Claude Fable 5 | Anthropic | Fast execution | Very Low |
| 4 | Claude Fable 5.1 | Anthropic | Routine tasks | Very Low |
| 5 | GPT-5.6 Luna | OpenAI | Quick tasks | Low |
| 6 | Gemini 3.8 Flash | Google | Fast execution | Low |
| 7 | GPT-5.6 Terra | OpenAI | Balanced tasks | Medium-High |
| 8 | Claude Sonnet 5 | Anthropic | Medium complexity | Medium |
| 9 | Claude Sonnet 5.5 | Anthropic | Balanced tasks | Medium |
| 10 | GPT-5.6 Sol | OpenAI | Complex reasoning | High |
| 11 | Claude Opus 4.8 | Anthropic | Advanced reasoning | High |
| 12 | Gemini 4 Argon | Google | Advanced reasoning | High |
| 13 | GPT-5.3-Codex | OpenAI | Autonomous coding | Very High |
| 14 | GPT-6 Astra | OpenAI | Long-horizon tasks | Very High |

### By Speed (Fastest to Slowest)

| Rank | Model | Response Time | Provider |
|------|-------|---------------|----------|
| 1 | Claude Haiku 4.5 | 1-2s | Anthropic |
| 2 | Claude Fable 5 | 1-2s | Anthropic |
| 3 | Gemini 3.8 Flash | 2-3s | Google |
| 4 | GPT-5.6 Luna | 2-3s | OpenAI |
| 5 | GPT-5.6 Terra | 3-4s | OpenAI |
| 6 | Claude Sonnet 5 | 3-4s | Anthropic |
| 7 | Claude Sonnet 5.5 | 3-4s | Anthropic |
| 8 | Gemini 4 Argon | 4-5s | Google |
| 9 | Claude Opus 4.8 | 5-7s | Anthropic |
| 10 | GPT-5.6 Sol | 4-6s | OpenAI |
| 11 | GPT-5.3-Codex | 8-10s | OpenAI |

### By Capability (Lowest to Highest)

| Tier | Models | Best For |
|------|--------|----------|
| **Basic** | Haiku 4.5, Fable 5, Luna | Routine tasks, formatting, simple fixes |
| **Intermediate** | Terra, Sonnet 5.5, Gemini 3.8 | Feature development, bug fixes, refactoring |
| **Advanced** | Sol, Opus 4.8, Gemini 4 | Complex reasoning, architecture, security analysis |
| **Frontier** | GPT-5.3-Codex, GPT-6 Astra | Autonomous long-horizon tasks |

---

## Optimization Checklist

### Infrastructure Level (Low Effort, High Impact)

- [ ] VS Code version 1.120 or later
- [ ] `prompt_cache_retention` set to `"24h"`
- [ ] Prompt caching enabled (`caching.enabled: true`)
- [ ] Output compression enabled
- [ ] Terminal output deltas enabled
- [ ] File exclusions configured (node_modules, dist, etc.)
- [ ] Background Todo Agent enabled (if needed)

### Model Selection Level (Medium Effort, High Impact)

- [ ] Using Copilot Fusion for task orchestration
- [ ] Reasoning models reserved for complex analysis only
- [ ] Lightweight models used for routine tasks
- [ ] Subagents using cheaper models than main agent
- [ ] Reasoning level set to regular (increased only when necessary)

### Session Management Level (Medium Effort, Medium Impact)

- [ ] Creating new sessions for different projects
- [ ] Keeping sessions focused (<50-100 turns)
- [ ] Using separate chats for different task types
- [ ] Reusing chat for same task continuation
- [ ] Using `/fleet` for parallelizable work
- [ ] Avoiding unnecessary context accumulation

### Prompt Engineering Level (Low-Medium Effort, Medium Impact)

- [ ] Prompts are concise (removing filler words)
- [ ] Task definitions are clear and specific
- [ ] Relevant context provided upfront
- [ ] Stopping conditions defined clearly
- [ ] Specific file references used (not full code dumps)
- [ ] Avoiding repeating context unnecessarily

### Monitoring Level (Low Effort, Medium Impact)

- [ ] Running `/chronicle:cost-tips` regularly
- [ ] Tracking token usage per session
- [ ] Identifying expensive patterns
- [ ] Testing cost optimizations
- [ ] Adjusting strategy based on data

---

## Troubleshooting Matrix

### Problem: High Token Usage Per Turn

| Symptom | Root Cause | Solution | Savings |
|---------|-----------|----------|---------|
| Each turn costs more than expected | Output not compressed | Enable `compressOutput.enabled` | 20-40% |
| Repeated output being sent | Output deltas not enabled | Enable `outputDeltas: true` | 70-90% |
| Verbose prompts accumulating | Unnecessary context in prompts | Provide concise prompts, specific file references | 50-70% |
| Cache keeps missing | Cache retention too short | Set `cache_retention: "24h"` | 70-90% |

### Problem: Cache Keeps Breaking

| Symptom | Root Cause | Solution | Prevention |
|---------|-----------|----------|-----------|
| Cache hit rate <50% | Instructions changing mid-session | Keep instructions stable within session | Use same .instructions.md throughout |
| Cache broken after tool use | Tool definitions changed | Don't add/remove tools mid-session | Use tool search (deferred loading) |
| Cache lost after model switch | Manual model switching | Use Copilot Fusion instead | Automatic model selection |

### Problem: Subagents Not Helping

| Symptom | Root Cause | Solution |
|---------|-----------|----------|
| /fleet costs more than sequential | Tasks are sequential, not parallel | Only use /fleet for truly independent tasks |
| Subagents slower than main agent | High coordination overhead | Use for tasks with clear independence |
| Missing context in subagents | Subagents not receiving needed context | Provide complete task description to /fleet |

### Problem: Billing Inaccurate or Confusing

| Symptom | Root Cause | Solution |
|---------|-----------|----------|
| Billing doesn't match token count | Old VS Code version | Upgrade to 1.120 or later |
| Model pricing unexpected | Using wrong model for task | Verify model selection matches use case |
| Charges higher than baseline | Cache not working properly | Check cache status with `/chronicle:cost-tips` |

---

## Glossary & Terminology

### Core Terms

**AI Credit**
- Unit of billing cost (1 credit = $0.01 USD)
- Derived from total tokens × model rate

**Cache Hit**
- Successful reuse of cached prefix tokens
- Costs 90% less than uncached input

**Cache Miss**
- Failed reuse of cached prefix (cache expired)
- Requires full recomputation at 100% cost

**Copilot Fusion**
- Intelligent orchestrator selecting optimal execution pattern per task
- Provides 30% cost discount (through Nov 30, 2026)

**Deferred Tool Loading**
- Loading tool definitions on-demand instead of preloading all
- Saves 8-9% tokens

**Output Compression**
- Automatic filtering and truncation of terminal output
- Removes noise, preserves errors and key info

**Output Delta**
- Sending only new output lines, not full history
- Saves 70-90% on repeated output tokens

**Prompt Caching**
- Reusing context across consecutive turns
- Saves 90% on cached input tokens

**Session**
- Conversation context with preserved cache
- Typically 50-100 turns before cognitive load increases

**Subagent**
- Independent AI agent handling focused subtask
- Used with `/fleet` for parallel execution

**Token**
- Atomic unit of text (roughly 4 characters)
- Input, output, and cached tokens priced differently

---

# Appendix: Quick Reference

## 30-Second Setup

```json
{
  "github.copilot.chat.caching.enabled": true,
  "prompt_cache_retention": "24h",
  "chat.tools.compressOutput.enabled": true,
  "chat.tools.terminal.outputDeltas": true,
  "github.copilot.chat.exclusions": [
    "**node_modules/**",
    "**/.git/**",
    "**/dist/**"
  ]
}
```

## 5-Minute Implementation

1. ✅ Copy settings above to `settings.json`
2. ✅ Restart VS Code
3. ✅ Use Copilot Fusion (Standard effort) instead of specific models
4. ✅ Start new sessions for different projects
5. ✅ Run `/chronicle:cost-tips` to verify

**Expected result**: 50-70% token cost reduction

## Model Selection Cheat Sheet

- **Simple tasks** → Claude Haiku 4.5
- **Regular work** → Copilot Fusion Standard
- **Complex problems** → GPT-5.6 Sol or Claude Opus 4.8
- **Parallel work** → /fleet with subagents on Haiku/Luna

---

**Document Version**: 1.0  
**Last Updated**: November 2026  
**Status**: Enterprise-Grade, Production-Ready

For official GitHub Copilot documentation, see: [GitHub Copilot Docs](https://docs.github.com/en/copilot)
