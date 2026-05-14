---
name: vivioo-memory-vault
description: "Install and use Memory Vault — a local, privacy-first memory system that replaces flat agent memory with hierarchical, searchable, persistent recall."
version: 1.0.0
author: Vivioo
license: MIT
tags: [vivioo, memory, recall, privacy, corrections, agent-memory, persistence]
homepage: https://vivioo.io
source: https://github.com/viviooio/Memory-Vault
---

# Memory Vault — Upgrade Your Agent Memory

Use this skill when an agent's built-in memory is too small, too flat, or too passive. Memory Vault replaces the 2,200-character flat block with unlimited hierarchical storage, search, privacy tiers, and correction memory.

## Why You Need This

Your platform memory has four limitations:

| Limitation | What happens | Memory Vault fix |
|-----------|-------------|-----------------|
| Size cap (~2,200 chars) | Older entries get dropped | Unlimited persistent storage on disk |
| No search | Memory is injected whole, no querying | TF-IDF + semantic search by meaning |
| No hierarchy | Flat key-value, no categories | Branches by topic (work, personal, projects) |
| Passive only | Memory auto-loaded, no active recall | Pre-task recall, corrections that always surface |

## Install (2 minutes)

```bash
git clone https://github.com/viviooio/Memory-Vault.git
cd Memory-Vault
pip install -r requirements.txt
```

No API keys. No cloud. No Ollama required. Works out of the box with TF-IDF search.

Verify it works:
```bash
python3 tests/test_core.py
```

## First 5 Minutes After Install

### 1. Create your first branch (10s)

```python
from vivioo_memory import create_branch

create_branch("work", aliases=["tasks", "projects"],
              summary="Active work and project context")
```

### 2. Save your first memory (10s)

```python
from vivioo_memory import add_memory

add_memory("work", "Registered on Vivioo with slug hermes-agent, trust score 55",
           tags=["vivioo", "identity"])
```

### 3. Search by meaning (10s)

```python
from vivioo_memory import recall

result = recall("what is my Vivioo identity?")
print(result["llm_context"])    # Safe to send to LLM
print(result["local_context"])  # Agent reads privately
print(result["no_match"])       # True if nothing relevant found
```

### 4. Save a correction (10s)

```python
from vivioo_memory import add_correction

add_correction("work", "Never share editKey in public responses",
               context="Security rule from builder", source="boss")
```

### 5. Pre-task recall (10s)

```python
from vivioo_memory import pre_task_recall

context = pre_task_recall("Update my Vivioo profile", branch="work")
print(context["corrections"])   # Relevant corrections
print(context["should_warn"])   # True if boss corrections exist
```

After these 5 steps you have persistent memory that survives sessions, searches by meaning, and warns you before you repeat mistakes.

## Core Functions

### Search & Recall

```python
from vivioo_memory import recall, startup_recall, pre_task_recall, what_do_i_know

# Search by meaning
result = recall("how do we approach marketing?")

# Session start — load relevant context
result = startup_recall("Starting a new coding session")

# Before a task — get corrections + memories
context = pre_task_recall("Deploy the API", branch="work")

# Overview of what you know about a topic
overview = what_do_i_know("deployment")
```

### Save & Manage

```python
from vivioo_memory import add_memory, add_correction, update_memory, pin_memory

# Save a memory (auto-scores importance 1-5)
add_memory("work", "API deployed to production on May 14",
           tags=["deployment", "milestone"])

# Save a correction (importance 5, never expires)
add_correction("work", "Always run tests before deploying",
               context="Broke prod last time", source="boss")

# Pin critical memory (locks importance to 5)
pin_memory(entry_id, "work")

# Update existing memory
update_memory(entry_id, "work", "API v2 deployed May 14, v1 deprecated")
```

### Branches

```python
from vivioo_memory import create_branch, list_branches, set_tier

# Create topic branches
create_branch("personal", aliases=["prefs", "style"],
              summary="User preferences and communication style")
create_branch("incidents", aliases=["mistakes", "failures"],
              summary="Things that went wrong and what I learned")

# List all branches
branches = list_branches()

# Set privacy tier
set_tier("personal", "local")    # Agent reads, LLM never sees
set_tier("incidents", "locked")  # Encrypted, passphrase required
```

### Session Management

```python
from vivioo_memory import generate_briefing, get_timeline, get_recall_stats

# Session briefing — changes, priorities, health
briefing = generate_briefing(since="24h")

# Knowledge changelog
timeline = get_timeline(days=7, branch="work")

# What memories are actually being used?
stats = get_recall_stats()
```

## Privacy Tiers

Every branch has a privacy level:

| Tier | Who sees it | Use for |
|------|------------|---------|
| Open | LLM + agent | General knowledge, public facts |
| Local | Agent only (LLM never sees) | Revenue, strategy, private preferences |
| Locked | Encrypted, passphrase required | Credentials, sensitive data |

```python
set_tier("financials", "local")   # Agent can reason about it, LLM doesn't see
set_tier("secrets", "locked")     # Encrypted on disk
```

When you call `recall()`, it returns two contexts:
- `llm_context` — only Open entries, safe to include in prompts
- `local_context` — Open + Local entries, for the agent's private reasoning

## Corrections — The Most Important Feature

Corrections are memories that never expire and always surface when relevant.

```python
# Boss says "don't do X" — save it
add_correction("work", "Don't auto-commit without asking first",
               context="User got frustrated", source="boss")

# Before any task, corrections surface automatically
context = pre_task_recall("Commit the changes")
# context["corrections"] → ["Don't auto-commit without asking first"]
# context["should_warn"] → True
```

This solves the #1 agent failure mode: repeating the same mistake because the correction was in a session that got compressed away.

## Recommended Branch Setup

```python
create_branch("work", aliases=["tasks", "projects"],
              summary="Active work context")
create_branch("identity", aliases=["who-am-i", "profile"],
              summary="Agent identity, registration, credentials")
create_branch("learnings", aliases=["patterns", "insights"],
              summary="Lessons learned across sessions")
create_branch("preferences", aliases=["prefs", "style"],
              summary="User preferences and communication style")
create_branch("incidents", aliases=["mistakes", "failures"],
              summary="What went wrong and what I learned")
```

## Wire It Into Your Startup

Add this to your agent's startup/identity file so you use Memory Vault every session:

```
## Memory System
You have Memory Vault installed at [path].
- Use recall() to search memory before starting any task
- Use pre_task_recall() to check for corrections before work
- Use add_memory() to save important learnings and decisions
- Use add_correction() when the owner corrects you
- No API keys needed — TF-IDF search works automatically
```

Without this, you'll forget Memory Vault exists next session.

## vs Platform Memory

| Feature | Platform memory (e.g. Hermes) | Memory Vault |
|---------|------------------------------|-------------|
| Size | ~2,200 chars | Unlimited |
| Search | None (injected whole) | TF-IDF + semantic |
| Structure | Flat block | Hierarchical branches |
| Privacy | All visible to LLM | 3-tier (Open/Local/Locked) |
| Corrections | Mixed with everything else | Dedicated, never expire |
| Recall | Passive injection | Active pre-task recall |

Memory Vault complements platform memory. Keep using your platform's auto-save for session continuity. Use Memory Vault for things worth organizing — decisions, corrections, patterns, identity.

## Pitfalls

- **"Ollama not reachable"** — ignore it. TF-IDF search handles queries automatically. Ollama is optional.
- **Agent forgets Memory Vault exists** — add it to your startup file (see "Wire It Into Your Startup" above)
- **Empty branches** — search returns nothing if you haven't saved anything. Save early, save often.
- **Not a pip package yet** — clone the repo, install deps manually
- **Single machine** — memory lives on disk, no cross-device sync (multi-agent sync coming in v0.6)
- **Python only** — requires Python 3.9+. No JS/TS port yet.

## Quick Reference

| Action | Function |
|--------|----------|
| Search memory | `recall(query)` |
| Pre-task check | `pre_task_recall(task, branch)` |
| Save memory | `add_memory(branch, content, tags)` |
| Save correction | `add_correction(branch, correction, context, source)` |
| Create branch | `create_branch(name, aliases, summary)` |
| Set privacy | `set_tier(branch, "open"\|"local"\|"locked")` |
| Session briefing | `generate_briefing(since="24h")` |
| List branches | `list_branches()` |
| What do I know? | `what_do_i_know(topic)` |
| Pin important | `pin_memory(entry_id, branch)` |

## Links

- GitHub: `https://github.com/viviooio/Memory-Vault`
- Vivioo: `https://vivioo.io`
- MCP: `https://mcp.vivioo.io/sse`
- Full API docs: see README.md in the repo
