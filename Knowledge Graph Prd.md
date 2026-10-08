# 📄 PRODUCT REQUIREMENTS DOCUMENT (PRD)
## Knowledge Graph Based Code Intelligence System

---

## 1. PROJECT OVERVIEW

### What We Are Building:
A fully local, offline-capable **Knowledge Graph Based Code Intelligence System** that:
- Parses entire codebases into a structured knowledge graph
- Stores nodes (files, functions, classes) and edges (imports, calls, dependencies)
- Exposes the graph via MCP server to AI coding tools
- **Automatically** detects file changes and heals broken references in real-time

### Who Uses This:
- Developers using AI coding tools (Claude Code, Cursor, Codex, etc.)
- AI agents that need accurate codebase context

### Core Goal:
Instead of giving AI tools the **entire codebase** as context — give only **relevant files** based on graph traversal. Better accuracy, less token waste.

---

## 2. TECH STACK (Exact)

```
Language          : Python 3.11+
Code Parsing      : tree-sitter (Python bindings)
Graph Storage     : SQLite3
File Monitoring   : watchdog
MCP Server        : mcp (Python MCP SDK by Anthropic)
Graph Export      : pyvis (interactive HTML), markdown, JSON
Clustering        : python-louvain (Leiden community detection)
Git Analysis      : gitpython
Language Support  : tree-sitter grammars for all 28+ supported languages
```

---

## 3. SUPPORTED INPUT TYPES

```
Source Code Files:
- Python, JavaScript, TypeScript, Go, Rust, Java, C, C++,
  Ruby, PHP, Swift, Kotlin, Scala, R, Lua, Haskell,
  Terraform HCL, SQL, Salesforce Apex, and more
  (All 28 Tree-sitter language grammars)

Document Files:
- PDFs
- Images
- Video / Audio files (via faster-whisper transcription)
- Google Workspace files (Docs, Sheets, etc.)

Database:
- Live PostgreSQL schema (via --postgres flag)
```

---

## 4. SYSTEM ARCHITECTURE — Full Pipeline

```
Input (Codebase / Docs / DB)
        ↓
STEP 1: PARSING ENGINE
- Tree-sitter: parse all source files
- Document processor: PDFs, images, video
- PostgreSQL introspection: live schema
        ↓
STEP 2: NODE + EDGE EXTRACTION
- Nodes: files, functions, classes, concepts
- Edges: imports, calls, inheritance, references
- Edge labels: EXTRACTED, INFERRED, AMBIGUOUS
        ↓
STEP 3: STORAGE (SQLite)
- nodes table
- edges table
- memories table
        ↓
STEP 4: GRAPH PROCESSING
- Community detection (Leiden algorithm)
- God node detection (most connected nodes)
- Cross-file relationship resolution
        ↓
STEP 5: OUTPUT GENERATION
- graph.html (interactive visualization)
- GRAPH_REPORT.md (markdown summary)
- graph.json (GraphRAG-ready JSON)
        ↓
STEP 6: MCP SERVER
- Exposes all graph query tools to AI assistants
        ↓
STEP 7: REAL-TIME FILE WATCHER (OUR ADDITION)
- Watchdog monitors project folder
- Auto-detects file changes
- Auto-heals broken references
```

---

## 5. DATABASE SCHEMA

### Nodes Table:
```sql
CREATE TABLE nodes (
    id TEXT PRIMARY KEY,
    type TEXT,           -- file, function, class, concept
    name TEXT,
    file_path TEXT,
    line_start INTEGER,
    line_end INTEGER,
    language TEXT,
    community_id INTEGER,
    metadata JSON
);
```

### Edges Table:
```sql
CREATE TABLE edges (
    id TEXT PRIMARY KEY,
    from_id TEXT,
    to_id TEXT,
    relationship TEXT,   -- imports, calls, inherits, references
    confidence TEXT,     -- EXTRACTED, INFERRED, AMBIGUOUS
    status TEXT,         -- active, broken
    broken_reason TEXT,
    metadata JSON,
    FOREIGN KEY (from_id) REFERENCES nodes(id),
    FOREIGN KEY (to_id) REFERENCES nodes(id)
);
```

### Memories Table:
```sql
CREATE TABLE memories (
    id TEXT PRIMARY KEY,
    content TEXT,
    created_at TIMESTAMP,
    tags JSON
);
```

---

## 6. CLI COMMANDS

```bash
# Build graph for first time
/graphify

# Update graph (incremental)
/graphify --update

# Force full rebuild
/graphify --force

# Re-run clustering only
/graphify --cluster-only

# Build directed graph
/graphify --directed

# Connect PostgreSQL
/graphify --postgres "connection_string"

# Query graph
/graphify query "how does auth connect to database?"

# Find path between two symbols
/graphify path "UserService" "DatabasePool"

# Explain a symbol
/graphify explain "RateLimiter"

# Export architecture diagram (Mermaid HTML)
graphify export callflow-html

# PR triage
graphify prs --triage

# Find PR conflicts
graphify prs --conflicts

# Start real-time file watcher (OUR ADDITION)
/graphify --watch
```

---

## 7. MCP TOOLS — Full List

### Graph Query Tools:

| Tool | Input | Output |
|------|-------|--------|
| `query_graph` | Plain English question | Relevant nodes + edges |
| `get_node` | Node name or id | Full node details |
| `get_neighbors` | Node name | All connected nodes |
| `get_community` | Community name | All nodes in cluster |
| `god_nodes` | None | Most connected nodes |
| `graph_stats` | None | Graph size + overview |
| `shortest_path` | Two symbol names | Path between them |
| `graphify_rank_files` | Plain English question | Ranked relevant files |

### PR Tools:

| Tool | Input | Output |
|------|-------|--------|
| `list_prs` | None | All open PRs |
| `get_pr_impact` | PR number | Affected nodes + blast radius |
| `triage_prs` | None | AI ranked PR review queue |

### Memory Tools:

| Tool | Input | Output |
|------|-------|--------|
| `recall` | Query | Relevant memories |
| `memories_about` | Topic | Related stored notes |
| `remember` | Note content | Saved confirmation |

### Our Addition — Healing Tools:

| Tool | Input | Output |
|------|-------|--------|
| `broken_references` | None | All broken edges in project |
| `heal_status` | File path | Healing status of a file |

---

## 8. GRAPH OUTPUT FILES

```
graphify-out/
├── graph.html        # Interactive visualization (pyvis)
├── GRAPH_REPORT.md   # Plain English architecture summary
└── graph.json        # GraphRAG-ready structured JSON
```

### graph.html:
- Interactive node-edge visualization
- Click nodes to see details
- Filter by community/type

### GRAPH_REPORT.md:
- High level architecture summary
- Community descriptions
- God nodes list
- Key relationships

### graph.json:
- Full graph in JSON format
- Compatible with GraphRAG pipelines
- Committable to repo

---

## 9. GIT INTEGRATION

```bash
# Commit graph to repo
git add graphify-out/
git commit -m "update knowledge graph"

# Post-commit hook — auto rebuild AST graph after each commit
# Add to .git/hooks/post-commit:
graphify --update

# Custom git merge driver — union merge graph.json
# No conflict markers on parallel commits
```

---

## 10. OUR ADDITION — REAL-TIME WATCHDOG AUTO-HEAL

### Problem With Current System:
```
User saves file
      ↓
User must manually run "/graphify --update"
      ↓
Only then graph updates
```

### Our Solution:
```
User saves file
      ↓
Watchdog auto-detects (background, always running)
      ↓
Only changed file re-parsed
      ↓
Old nodes vs new nodes compared
      ↓
Missing/renamed nodes detected
      ↓
All connected edges validated
      ↓
Broken edges marked as "broken" with reason
      ↓
Graph updated — AI tools get accurate data instantly
```

### Healing Algorithm — Step by Step:

```python
"""
HEALING ALGORITHM LOGIC:

1. Watchdog detects file save event → triggers heal()

2. Re-parse changed file only → get new_nodes list

3. Load old_nodes for that file from SQLite

4. Compare:
   - Nodes in old but not in new → DELETED/RENAMED
   - Nodes in new but not in old → ADDED

5. For each deleted/renamed node:
   - Find ALL edges across entire graph pointing to it
   - For each edge:
       - Check if source file still references this node
       - If yes → mark edge as BROKEN, store reason
       - If no → mark edge as RESOLVED

6. For new nodes:
   - Check all existing broken edges
   - If any broken edge points to new node name → RESOLVE it

7. Update SQLite with new status

8. MCP tool "broken_references" now returns updated list
"""
```

---

## 11. BUG FIXES WE ARE IMPLEMENTING

### Fix 1 — Dangling Edge Detection:
```
Current: Dangling edges silently skipped (masked in build.py)
Our Fix: Dangling edges marked as BROKEN, exposed via MCP tool
```

### Fix 2 — Stale Nodes After File Delete:
```
Current: Deleted file nodes linger in graph, manual --force needed
Our Fix: Watchdog detects file delete, auto-removes stale nodes
```

### Fix 3 — Cross-file Import Crash:
```
Current: _resolve_cross_file_imports() runs unconditionally,
         crashes on non-Python projects
Our Fix: Add "if py_paths:" check before running
```

### Fix 4 — MCP Performance (Graph Cache):
```
Current: query_graph and shortest_path rebuild full graph copy
         on every single MCP call
Our Fix: Graph cached in memory, rebuilt only on file change event
```

### Fix 5 — Ghost Duplicate Nodes:
```
Current: Same symbol appears twice (AST + semantic extraction)
         Only fixed from v0.8.33 onwards
Our Fix: Explicit deduplication at extraction stage
```

---

## 12. FOLDER STRUCTURE

```
project/
├── core/
│   ├── parser.py           # Tree-sitter parsing logic
│   ├── extractor.py        # Node + edge extraction
│   ├── graph_builder.py    # Full graph construction
│   ├── graph_cache.py      # In-memory graph cache (Bug Fix 4)
│   └── cross_file.py       # Cross-file import resolution
│
├── storage/
│   ├── db.py               # SQLite connection + queries
│   ├── nodes.py            # Node CRUD operations
│   └── edges.py            # Edge CRUD + status management
│
├── watcher/
│   ├── file_watcher.py     # Watchdog setup + event handlers
│   └── healer.py           # Our Relationship Healing Algorithm
│
├── mcp_server/
│   ├── server.py           # MCP server setup
│   ├── tools_graph.py      # Graph query MCP tools
│   ├── tools_pr.py         # PR related MCP tools
│   ├── tools_memory.py     # Memory MCP tools
│   └── tools_healing.py    # Our broken_references + heal_status tools
│
├── output/
│   ├── html_export.py      # graph.html generation (pyvis)
│   ├── md_export.py        # GRAPH_REPORT.md generation
│   └── json_export.py      # graph.json generation
│
├── clustering/
│   └── community.py        # Leiden community detection
│
├── git/
│   └── git_analysis.py     # GitPython integration
│
├── graphify-out/
│   ├── graph.html
│   ├── GRAPH_REPORT.md
│   └── graph.json
│
├── tests/
│   ├── test_parser.py
│   ├── test_extractor.py
│   ├── test_healer.py
│   ├── test_mcp_tools.py
│   └── sample_codebase/    # Test project for integration tests
│
├── main.py                 # Entry point
├── config.py               # Configuration
└── requirements.txt
```

---

## 13. TESTING REQUIREMENTS

### Unit Tests:
```
- test_parser.py         → Tree-sitter correctly extracts nodes
- test_extractor.py      → Edges correctly identified
- test_healer.py         → Broken references correctly marked
- test_mcp_tools.py      → Each MCP tool returns correct output
- test_cache.py          → Graph cache updates on file change
- test_dedup.py          → Duplicate nodes not created
```

### Integration Tests:
```
- Full pipeline test: Sample codebase → parse → store → MCP query
- Healing test: Delete function → verify broken edges marked
- Rename test: Rename function → verify edges updated
- Delete file test: Delete file → verify stale nodes removed
- Multi-language test: Python + JS project → both parsed correctly
```

### Sample Codebase For Testing:
```
tests/sample_codebase/
├── main.py           # Calls functions from utils.py
├── utils.py          # Has calculate_price(), format_output()
├── orders.py         # Imports calculate_price from utils.py
└── cart.py           # Imports calculate_price from utils.py

Test scenario:
1. Build graph → 4 files, all edges active
2. Delete calculate_price() from utils.py
3. Verify: orders.py + cart.py edges marked BROKEN
4. Re-add calculate_price() to utils.py
5. Verify: edges automatically RESOLVED
```

---

## 14. COMPATIBLE AI TOOLS (via MCP)

```
- Claude Code
- Cursor IDE
- OpenAI Codex
- Windsurf IDE
- GitHub Copilot
- ChatGPT Desktop
- Gemini CLI
- 20+ other MCP-compatible tools
```

---

## 15. PERFORMANCE REQUIREMENTS

```
- Graph build (first time, 1000 file project): < 60 seconds
- Incremental update (single file change): < 2 seconds
- Watchdog detection to heal completion: < 3 seconds
- MCP tool response time: < 500ms (with cache)
- Memory usage: < 500MB for 10,000 node graph
```

---