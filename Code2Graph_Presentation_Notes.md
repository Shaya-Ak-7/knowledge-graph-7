# Code2Graph — Presentation Handout

**Knowledge Graph-Based Structural Context for AI-Assisted Software Development**  
Team 20 · Akshaya Kumar R S (910623108002) · Sachin J (910623108041) · Senthil Pandian G S (910623108044)  
Supervisor: Mrs. Ilamathi S., AP / AI & DS · Dept. of AI & DS, K.L.N. College of Engineering  
Academic Year 2026–2027 · 56 slides

---

## 1. K.L.N. COLLEGE OF ENGINEERING, POTTAPALAYAM – 630 612
- Department of Artificial Intelligence and Data Science
- Knowledge Graph-Based Structural Context
- for AI-Assisted Software Development
- Code2Graph — a code-intelligence system built on Tree-sitter parsing, NetworkX graph analytics and MCP
- Team No: 20        Academic Year: 2026 – 2027
- Supervisor:  Mrs. Ilamathi S.,  Assistant Professor / AI & DS
- Team:  Akshaya Kumar R S (910623108002)   ·   Sachin J (910623108041)   ·   Senthil Pandian G S (910623108044)

**Speaker notes:** Good morning. We are Team 20, and our project is 'Knowledge Graph-Based Structural Context for AI-Assisted Software Development' — implemented as a system called Code2Graph. The whole idea is that AI coding assistants need the *right* code context, and the right context is mostly about relationships between files and symbols, not more lines of text.

## 2. Index
- Contents of the presentation
- 1
- Abstract
- 2
- Introduction
- 3
- Problem Statement
- 4
- Existing System
- 5
- Proposed System
- 6
- Literature Review
- 7
- System Architecture
- 8
- List of Modules
- 9
- Module Description
- 10
- Output Screenshots

**Speaker notes:** These are the ten sections we will cover: abstract, introduction, problem statement, existing system, proposed system, literature review, system architecture, list of modules, module description and the output screenshots.

## 3. Abstract
- What the project is, and what it delivers
- ▪   A persistent code-intelligence graph built from a software repository
- ▪   Tree-sitter parsing + language-specific extractors for nodes and edges
- ▪   Graph queries, shortest paths, community analysis, change impact
- ▪   Local, LLM-free indexing · live watcher · broken-reference healing · MCP access
- 01

**Speaker notes:** Section 1 — Abstract. One line each: what the project is, what it builds, and what you will see.

## 4. Abstract — The Problem We Address
- Section 01
- 01
- ▪   AI-assisted programming tools need relevant code context to understand a software project and reason about changes.
- ▪   Broad context retrieval and file-level search can overlook the relationships between components.
- ▪   They can equally include irrelevant information, wasting context and tokens.
- ▪   Result: assistants answer with partial or wrong mental models of the codebase.
- What Code2Graph does
- Builds a persistent graph from a software repository
- Scans supported source and configuration files
- Tree-sitter-based parsing + language-specific extractors
- Represents files, modules, classes and functions as nodes
- How the graph is used
- Graph queries and shortest-path tracing
- Component explanation and community analysis
- Change-impact inspection before a refactor
- Focused structural context handed to AI assistants

**Speaker notes:** The problem: AI tools are given broad context or file-level context. They miss relationships between components and they also pull in irrelevant code. So the assistant gets an incomplete and noisy picture of the project.

## 5. Abstract — System Summary
- Section 01
- 01
- ▪   Relationships such as imports, calls, inheritance and dependencies are represented as graph edges.
- ▪   Code-only indexing runs locally — no LLM API calls and no token consumption.
- ▪   Optional semantic workflows process supported documents and media when a model provider is configured.
- ▪   A file-system watcher incrementally updates the graph as files change.
- ▪   Missing-target references are recorded and reconciled during updates, so suitable relationships recover when their targets return.
- ▪   The graph is accessed through a CLI, IDE-specific agent skills, and an optional MCP server for compatible assistants.
- ▪   Goal: help developers and AI assistants navigate codebases and reason about cross-file changes more effectively.
- ▪   Limitation: static analysis may not resolve all dynamic or runtime-dependent behaviour.

**Speaker notes:** Code2Graph builds a persistent graph from the repository: files, modules, classes and functions are nodes; imports, calls, inheritance and dependencies are edges. Code-only indexing is fully local — no API key, no tokens. A watcher keeps the graph current and missing-target references are recorded and later healed. The graph is served through a CLI, agent skills and an optional MCP server.

## 6. Introduction
- Why a code graph matters for AI-assisted development
- ▪   A repository is more than a collection of files
- ▪   Relationships are the missing context — not more text
- ▪   Deterministic structure first, semantic inference second
- ▪   One graph, many surfaces: CLI, agent skills, MCP
- 02

**Speaker notes:** Section 2 — Introduction.

## 7. Introduction — From Files to a Code Graph
- Section 02
- 02
- ▪   Modern projects are large, multi-file and highly interconnected; a repository is more than a collection of files.
- ▪   Code2Graph models symbols, files, imports, calls, inheritance and other relationships as a graph that coding agents can query.
- ▪   This makes it easier to follow execution paths, understand dependencies, and identify the impact of a change.
- ▪   Instead of repeatedly re-scanning the entire project, an assistant asks a focused structural question.
- ▪   Turning a source tree into a navigable code-intelligence graph keeps AI context small and relevant.
- The central idea
- ▪  Files and symbols become nodes; imports, calls, containment, inheritance and dependencies become edges.
- ▪  The graph is persisted (graph.json), so the expensive work happens once and is reused by every later question.
- ▪  A watcher keeps it current, and every extracted relationship carries a confidence label: EXTRACTED, INFERRED or AMBIGUOUS.

**Speaker notes:** A repository is more than files. Code2Graph models symbols, files, imports, calls and inheritance as a graph that agents can query, so execution paths, dependencies and change impact become graph questions instead of guesswork.

## 8. Introduction — Capabilities at a Glance
- Section 02
- 02
- Understand a repo
- Find entry points, symbols, modules, calls, imports and dependencies
- Trace behaviour
- Query relationships, neighbours, communities, shortest-path tracing
- Assess a change
- Inspect callers and other nodes affected by a refactor
- Keep it current
- Saves, deletes and renames trigger incremental updates and healing
- ▪   Usable three ways: an agent skill inside a coding assistant, a terminal CLI, or an MCP server with a live watcher.
- ▪   Code indexing needs no API key — a key is only required for optional document / media semantic workflows.

**Speaker notes:** Five capability groups: understand a repo, trace behaviour, assess a change, keep references current, and optionally explore documents and media with a semantic provider. Everything structural is local and deterministic.

## 9. Problem Statement
- What is missing in today's context pipelines
- ▪   File-level retrieval loses structure
- ▪   Irrelevant context inflates tokens and error rates
- ▪   Cross-file impact is hard to see
- ▪   Broken references are silently lost
- 03

**Speaker notes:** Section 3 — Problem statement.

## 10. Problem Statement
- Section 03
- 03
- ▪   AI assistants must be given the right context before they can reason about a change — but choosing that context is the hard part.
- ▪   Broad context retrieval and file-level search can overlook relationships between components.
- ▪   The same strategies pull in irrelevant information, which costs tokens and increases hallucination risk.
- ▪   Manual navigation across a large codebase is slow, and cross-file impact of a change is easy to miss.
- ▪   During refactors, renames and deletions, references break — and tooling rarely tracks what broke or why.
- ▪   There is no single, persistent, queryable structure that both developers and agents can share.
- Requirement
- A persistent structural context layer that is (a) focused, (b) locally computable, (c) incrementally updated, and (d) queryable by an AI assistant through a standard interface.

**Speaker notes:** The assistant must be given the right context before it reasons about a change. Broad retrieval misses structure and adds noise; refactors break references and the tooling rarely tracks what broke.

## 11. Problem Statement — Consequences
- Section 03
- 03
| Gap | Consequence for the developer / assistant |
| No relationship model | The assistant sees files, not how components call, inherit from or depend on each other. |
| Whole-file or chunk retrieval | Context windows fill with unrelated code; answers drift from the real implementation. |
| Repeated full scans | Slow, token-heavy and repeated for every question asked about the repository. |
| No broken-reference tracking | Refactoring silently drops edges; the assistant then reasons on a stale mental model. |
| Tool-specific integrations | Every assistant needs a bespoke integration instead of one standard graph interface. |
- ▪   These gaps motivate a locally-computed graph that is small enough to query but complete enough to trust.

**Speaker notes:** The consequence table: each gap — no relationship model, whole-file retrieval, repeated scans, no broken-reference tracking, tool-specific integrations — translates into a practical failure for the developer or the assistant.

## 12. Existing System
- How code context is handled today
- ▪   IDE search, grep and file trees
- ▪   Language Server Protocol (LSP) navigation
- ▪   RAG over code chunks and whole files
- ▪   Static analysers, docs generators and code-search platforms
- 04

**Speaker notes:** Section 4 — Existing system.

## 13. Existing System
- Section 04
- 04
| Approach | What it gives you | What it does not give you |
| IDE search / grep, file tree | Fast lexical lookup; go-to-file navigation | No relationship model; misses indirect references |
| Language Server Protocol | Symbol-level go-to-definition and find-references | Per-workspace, IDE-bound; no persistent queryable graph |
| RAG over code chunks / whole files | Text similarity retrieval for an LLM | Structure-blind; pulls irrelevant code; no impact analysis |
| Docs generators / static analysers | Reports, coverage, dependency listings | Batch outputs; not queryable mid-task by an agent |
| Code-search platforms | Cross-repo search at organisation scale | Heavy infrastructure; not a local, self-updating graph |

**Speaker notes:** How code context is handled today: IDE search, LSP, RAG over chunks, static analysers and code-search platforms. Each is useful, and each misses something.

## 14. Existing System — Limitations
- Section 04
- 04
- ▪   No unified, persistent relationship graph covering the whole repository.
- ▪   File-level or chunk-level retrieval misses structural links such as calls, inheritance and imports.
- ▪   Retrieved context is often irrelevant, raising token cost and hallucination risk.
- ▪   Updates are manual — there is no automatic incremental rebuild when a file changes on disk.
- ▪   Broken references after a rename or delete are neither recorded nor reconciled.
- ▪   Integration with assistants is ad hoc instead of using a standard protocol.
- ▪   Code indexing and semantic (document/media) extraction are not clearly separated.

**Speaker notes:** The limitations that matter for our problem: no unified persistent graph, no structural links, irrelevant context, manual updates, no broken-reference handling, ad hoc integrations, and code vs semantic extraction not separated.

## 15. Proposed System
- Code2Graph — design and advantages
- ▪   Persistent graph of nodes and relationships
- ▪   Local structural extraction, optional semantic layer
- ▪   Incremental watcher with broken-reference healing
- ▪   CLI, agent skills and MCP server access
- 05

**Speaker notes:** Section 5 — Proposed system.

## 16. Proposed System — Code2Graph
- Section 05
- 05
| Design decision | How Code2Graph implements it |
| Graph as the single model | Files, modules, classes, functions and other symbols are nodes; imports, calls, inheritance and dependencies are edges. |
| Local deterministic parsing | Tree-sitter-backed extractors per language, plus language-specific resolvers; no code is sent to a model for structural indexing. |
| Confidence-labelled edges | EXTRACTED, INFERRED and AMBIGUOUS labels make the strength of every relationship explicit. |
| Incremental by default | A watchdog-based watcher debounces saves/renames and re-parses only the affected files. |
| Self-healing references | Edges whose endpoint disappears are stored as BROKEN metadata and restored (HEALED) when the target returns. |
| Many access surfaces | Terminal CLI, IDE/agent skill files, and optional MCP server exposing graph tools to assistants. |

**Speaker notes:** Our design decisions: the graph is the single model, parsing is local and deterministic, every edge carries a confidence label, updates are incremental, references heal themselves, and the graph is exposed through CLI, agent skills and MCP.

## 17. Proposed System — Features & Advantages
- Section 05
- 05
- Query
- Natural-language questions resolved by BFS/DFS traversal of the graph
- Paths
- Shortest path between two symbols, with relation and confidence shown
- Explain
- Plain-language explanation of a node and its neighbours
- Impact
- Reverse traversal listing everything a change can break
- Communities
- Louvain clustering + god-node detection for architecture insight
- Healing
- Broken and healed reference diagnostics exposed to the agent
- ▪   Advantages: focused context instead of whole files · measurable token reduction · no API key for code · stays current while you type · survives refactors.

**Speaker notes:** Six features: query, path, explain, impact, communities and healing. The advantage line summarises it: focused context instead of whole files, with measurable token reduction.

## 18. Proposed System — Existing vs Proposed
- Section 05
- 05
| Aspect | Existing approaches | Proposed — Code2Graph |
| Context unit | Whole files / text chunks | Graph nodes with their relationships |
| Relationships | Not modelled (or IDE-local only) | calls, imports, inherits, contains, method, uses … |
| Update model | Manual re-run / re-index | Debounced incremental watcher, atomic writes |
| Broken references | Silently dropped | Recorded as BROKEN, reconciled to HEALED |
| Assistant integration | Custom per-tool glue | Standard MCP tools and resources |
| Cost | Large prompts, repeated scans | Local AST pass; ~4.6× fewer tokens per query (sample run) |

**Speaker notes:** Side-by-side comparison. Notice the last row: on our sample repository the graph-based answers used about 4.6 times fewer tokens than sending the whole corpus.

## 19. Literature Review
- Background areas and the technologies behind the design
- ▪   Knowledge graphs in software engineering
- ▪   Static analysis and incremental parsing (Tree-sitter)
- ▪   Graph analytics: communities, centrality, shortest paths
- ▪   Model Context Protocol for assistant integration
- 06

**Speaker notes:** Section 6 — Literature review.

## 20. Literature Review — Background Areas
- Section 06
- 06
| Area | Key idea taken into this project |
| Knowledge graphs in software engineering | Represent files, symbols and relationships explicitly so navigation, tracing and impact analysis become graph queries. |
| Static analysis / AST parsing | Tree-sitter offers robust, incremental, error-tolerant parsing across many languages — the base of the extraction layer. |
| Code intelligence platforms & LSP | Symbol-level go-to-definition and find-references prove the value of a relationship model, but stay IDE/workspace bound. |
| Graph analytics | Community detection (Louvain), centrality and shortest paths expose architectural hubs and bridging components. |
| Incremental indexing | File-system watchers with debouncing keep derived artefacts fresh with minimal recomputation. |
| Model Context Protocol | A standard way to expose tools and resources so any compatible assistant can query the same graph. |

**Speaker notes:** The background areas we studied: knowledge graphs in software engineering, static analysis and Tree-sitter parsing, code-intelligence platforms and LSP, graph analytics, incremental indexing, and the Model Context Protocol.

## 21. Literature Review — Concepts & Scope
- Section 06
- 06
- ▪   Relationship confidence: distinguishing EXTRACTED, INFERRED and AMBIGUOUS edges keeps deterministic parsing separate from semantic guessing.
- ▪   Broken-reference handling: retaining unresolved edges as metadata (with reason, missing node and source location) instead of dropping them.
- ▪   Refactoring support: reverse traversal over callers and dependents gives change-impact analysis before the edit lands.
- ▪   Deterministic vs. semantic extraction: structural indexing never depends on an LLM; semantic extraction is optional and flagged when unavailable.
- ▪   Repository hygiene: ignore rules, size caps, URL/path validation and label sanitisation protect the local corpus.

**Speaker notes:** The concepts we adopted: confidence labels for relationships, keeping broken references as recoverable metadata, reverse traversal for change impact, and a strict separation between deterministic extraction and semantic inference.

## 22. Literature Review — Technology Stack
- Section 06
- 06
- Python 3.10+
- Core language / runtime
- Tree-sitter + grammars
- Incremental multi-language parsing
- NetworkX
- Graph construction, path, centrality, Louvain
- watchdog
- File-system events with debouncing
- MCP SDK (stdio/HTTP)
- Assistant-facing tools and resources
- rapidfuzz · numpy · tomli
- Matching, numerics, configuration
- ▪   Output formats: graph.json · GRAPH_REPORT.md · interactive graph.html · SVG · GraphML · Obsidian vault · Cypher · call-flow HTML.
- ▪   Testing: pytest suites covering extraction, resolution, graph validation, caching, watching and healing.

**Speaker notes:** Technology stack: Python 3.10+, Tree-sitter with per-language grammars, NetworkX, watchdog, the MCP SDK, and smaller helpers like rapidfuzz. Outputs range from graph.json to interactive HTML, SVG, GraphML, Obsidian and Cypher.

## 23. System Architecture
- Pipeline, layers and data flow
- ▪   detect → extract → build → cluster → analyse → report/export
- ▪   Modular stages exchanging plain dicts and NetworkX graphs
- ▪   Persisted graph + cache + manifest for incremental work
- ▪   Watcher and MCP server as the live runtime
- 07

**Speaker notes:** Section 7 — System architecture.

## 24. System Architecture — Pipeline
- Section 07
- 07
- detect()
- extract()
- build()
- cluster()
- analyse
- report
- export
- ▪   Each stage lives in its own module; stages communicate through plain Python dicts and NetworkX graphs.
- ▪   detect(root) → directory scan summary: files grouped by category, totals, warnings, scan root.
- ▪   extract(paths, *, root) → {nodes, edges}; collect_files(target) expands a directory into that list.
- ▪   build(extractions) → nx.Graph: deduplicates nodes, normalises relationships, refuses dangling endpoints.
- ▪   cluster(G) → {community_id: [node_id, …]} — the graph itself is not mutated.
- ▪   analyse helpers: god_nodes, surprising_connections, suggest_questions, find_import_cycles, graph_diff.
- ▪   report.generate(…) → GRAPH_REPORT.md string;  export.to_*(G) → json / html / svg / graphml / cypher / obsidian …

**Speaker notes:** The pipeline is detect, extract, build, cluster, analyse, report and export. Every stage is its own module and they exchange plain dictionaries and NetworkX graphs.

## 25. System Architecture — Layered View
- Section 07
- 07
- Access layer
- Terminal CLI   ·   IDE / agent skill files   ·   MCP server (stdio + HTTP)   ·   graph.html / call-flow HTML
- Query & analysis layer
- query · path · explain · affected · god-nodes · communities · graph_diff · benchmark · broken_references / heal_status
- Graph store
- graph.json (nodes, links, hyperedges, broken_references, healing_status)   ·   in-memory NetworkX cache   ·   manifest + cache for incremental work
- Extraction layer
- Tree-sitter extractors per language   ·   cross-file resolvers   ·   config/manifest/schema ingesters   ·   semantic (optional LLM) pass
- Source layer
- Repository files: source code, configuration, Markdown, documents, media
- dataflow

**Speaker notes:** The same system as layers: an access layer of CLI/agent skills/MCP; a query and analysis layer; the graph store; the extraction layer; and the source files underneath.

## 26. System Architecture — Extraction & Graph Model
- Section 07
- 07
- ▪   1. Discover files — language, ignore and safety rules are applied before anything is parsed.
- ▪   2. Parse code locally — Tree-sitter extractors build file and symbol nodes plus relationships (imports, calls, containment, inheritance).
- ▪   3. Resolve relationships — cross-file resolvers use current parse results plus read-only symbol data from unchanged files.
- ▪   4. Build and validate — the builder deduplicates nodes, normalises relationships and refuses dangling endpoints.
- ▪   5. Persist outputs — graph.json is the queryable graph; cache and manifest files support incremental work.
- Extraction schema
- { “nodes”: [ {id, label, source_file, source_location} ],
- “edges”: [ {source, target, relation, confidence} ] }
- validate.py enforces it before build() consumes it.
- Edge direction & confidence
- Arc order carries direction (source → target); _src/_tgt markers preserve it on reload.
- EXTRACTED · INFERRED · AMBIGUOUS — ambiguous edges are flagged for review.

**Speaker notes:** Extraction in five steps — discover files, parse locally with Tree-sitter, resolve relationships across files, build and validate the graph without dangling endpoints, then persist the outputs. The schema and the confidence labels are shown at the bottom.

## 27. System Architecture — Live Update & Healing
- Section 07
- 07
- ▪   When the MCP server starts, a background watcher monitors the project that contains the configured graph.
- ▪   A short debounce (default 0.75 s) groups editor save events; a move event contributes both the old and the new path.
- ▪   Only changed, locally parseable files are re-parsed; stored AST data for unchanged files is reused as resolution context.
- ▪   Stale nodes from deleted or renamed-away files are removed, then nodes and edges are reconciled with the previous graph.
- ▪   If a relationship loses an endpoint it is kept out of the active graph and recorded under graph.broken_references as BROKEN (with reason, missing node, affected files, source location).
- ▪   On the next rebuild the healer compares the previous graph with the new node set: a returning target restores the edge and marks the record HEALED. A unique same-file, same-location match can heal a rename; ambiguous matches are never guessed.
- ▪   MCP tools broken_references and heal_status expose this state to the agent; updates are atomic and serialised per project.

**Speaker notes:** Live behaviour: the watcher debounces events, re-parses only affected files, reconciles the graph, and records any relationship that lost its endpoint as BROKEN with a reason. On the next rebuild a returning target restores the edge and marks it HEALED. The MCP tools broken_references and heal_status expose this to the agent.

## 28. List of Modules
- The building blocks of the implementation
- ▪   Core pipeline modules
- ▪   Extractor and ingester modules
- ▪   Runtime modules: watcher, server, healing
- ▪   Support modules: security, validation, caching
- 08

**Speaker notes:** Section 8 — List of modules.

## 29. List of Modules — Core Pipeline
- Section 08
- 08
| # | Module | Responsibility |
| 1 | detect.py | Scan a directory into a file summary grouped by category |
| 2 | extract.py | Path list → {nodes, edges}; collect_files expands directories |
| 3 | extractors/ | Language-specific Tree-sitter extractors and resolvers |
| 4 | build.py | Extraction → NetworkX graph; dedup, normalise, validate |
| 5 | cluster.py | Louvain community detection over the graph |
| 6 | analyze.py | God nodes, surprises, suggested questions, cycles, diff |
| 7 | report.py | Generate GRAPH_REPORT.md from graph + analysis |
| 8 | export.py | JSON, HTML, SVG, GraphML, Obsidian, Canvas, Cypher |
| 9 | wiki.py | One Markdown article per community + index |

**Speaker notes:** The core pipeline modules with their responsibilities: detect, extract, the language extractors, build, cluster, analyse, report, export and wiki.

## 30. List of Modules — Runtime & Support
- Section 08
- 08
| # | Module | Responsibility |
| 10 | serve.py | MCP server over stdio or HTTP, with in-memory graph cache |
| 11 | watch.py | Watch a directory and rebuild the graph on code changes |
| 12 | healing.py / broken refs | Record BROKEN edges, reconcile them to HEALED |
| 13 | cache.py | Semantic cache: nodes / edges / hyperedges + pending files |
| 14 | manifest.py | Per-file stamps that drive incremental re-extraction |
| 15 | security.py | URL, path and label validation before any external input |
| 16 | validate.py | Enforce the extraction schema before build() |
| 17 | ids.py · paths.py | Deterministic node ids and graph path handling |
| 18 | benchmark.py | Corpus vs. subgraph token-cost comparison |
| 19 | callflow_html.py | Mermaid architecture / call-flow HTML report |

**Speaker notes:** Runtime and support modules: the MCP server, the watcher, healing, the semantic cache, the manifest that drives incremental work, security validation, schema validation, id/path helpers and the benchmark.

## 31. Module Description
- What each module does, and with what interface
- ▪   Verified entry points and data contracts
- ▪   Analysis, reporting and export behaviour
- ▪   Runtime, caching and healing mechanics
- 09

**Speaker notes:** Section 9 — Module description.

## 32. Module Description — Extraction & Build
- Section 09
- 09
| Module · entry point | Input → Output |
| detect.detect(root) | Directory → scan summary dict (files by category, totals, warning, scan_root) |
| extract.extract(paths, *, root) | List of file paths → {nodes, edges} dict |
| extract.collect_files(target) | Directory → the file path list consumed by extract() |
| build.build(extractions) / build_from_json | Extraction dict(s) → nx.Graph |
| cluster.cluster(G) | Graph → {community_id: [node_id, …]} (graph not mutated) |
| validate.validate_extraction / assert_valid | Extraction dict → schema error list / raises on invalid |
| extractors.extract_<lang>(path) | One file → nodes and edges for that language |
- ▪   extract() takes a list of paths and a keyword-only root — node ids and source_file values are derived relative to that root.

**Speaker notes:** Real entry points and their contracts for extraction and build. Note that extract() takes a list of paths and a keyword-only root, because node ids are derived relative to that root.

## 33. Module Description — Analysis & Output
- Section 09
- 09
| Function | What it produces |
| analyze.god_nodes(G) | Most-connected nodes — the architectural hubs |
| analyze.surprising_connections(G) | Cross-community links worth a second look |
| analyze.suggest_questions(G, …) | Question candidates this graph is uniquely able to answer |
| analyze.find_import_cycles(G) | Detected import cycles |
| analyze.graph_diff(G_old, G_new) | Node/edge level difference between two graphs |
| report.generate(G, …) | GRAPH_REPORT.md: summary, hubs, surprises, communities, gaps |
| export.to_json / to_html / to_svg / to_graphml | Persisted graph and visual artefacts |
| export.to_obsidian / to_canvas / to_cypher | Obsidian vault + canvas, and Cypher for graph databases |
| wiki.to_wiki(G, communities, …) | One Markdown article per community plus index.md |

**Speaker notes:** Analysis and output functions: god nodes, surprising connections, suggested questions, import cycles, graph diff, the report generator, the exporters and the wiki writer.

## 34. Module Description — Runtime, Cache & Healing
- Section 09
- 09
| Function | Behaviour |
| serve.serve(graph_path) | MCP stdio server; loads the graph, starts the project watcher |
| serve.serve_http(graph_path, *, host, port) | HTTP transport for the same tool surface |
| watch.watch(path, debounce) | Rebuild the graph when watched files change |
| watch.check_update(path) | Report whether a re-extraction is still pending |
| cache.check_semantic_cache / save_semantic_cache | Reuse cached nodes / edges / hyperedges; list files still needing extraction |
| healing + graph.broken_references | Store unresolved edges with reason, missing node and affected files |
| graph.healing_status | Per-file summaries consumed by the heal_status MCP tool |
| security.validate_url / safe_fetch / validate_graph_path / sanitize_label | Guard every external input and every label written to the graph |
- ▪   MCP tools: query_graph · shortest_path · get_node · get_neighbors · get_community · god_nodes · graph_stats · broken_references · heal_status · list_prs · get_pr_impact · triage_prs.

**Speaker notes:** Runtime, cache and healing: the stdio/HTTP MCP server, the watcher and its update check, the semantic cache, broken-reference storage and healing status, and the security validators. The MCP tool list is at the bottom.

## 35. Output Screenshots
- Real runs of Code2Graph on a 15-file sample repository
- ▪   Indexing and clustering from an empty output folder
- ▪   Queries, paths, explanations and impact
- ▪   Exports, MCP session, healing, graph views
- 10

**Speaker notes:** Section 10 — Output screenshots. Every screenshot that follows is a real run of the tool on a 15-file sample repository; the same run gives 65 nodes, 110 edges and 9 communities.

## 36. output/01 — Indexing a repository
- Section 10
- 10
- code2graph extract . --code-only parses 15 code files locally — no API key, no tokens — then cluster-only writes the graph, GRAPH_REPORT.md and graph.html.

**Speaker notes:** First run: extracting the repository. Everything is local AST work — no API key and no tokens — and the graph is written to code2graph-out/graph.json.

## 37. output/02 — Graph query and hubs
- Section 10
- 10
- A focused question is answered over the graph (BFS, context filtered to calls) and god-nodes lists the architectural hubs of the project.

**Speaker notes:** A focused query, and the god-nodes command that lists the architectural hubs of the project.

## 38. output/03 — Natural-language traversal
- Section 10
- 10
- The same question with a wider budget returns 19 nodes and 29 edges, each with source file, line and community — the assistant's view of the graph.

**Speaker notes:** The same question with a wider budget: 19 nodes and 29 edges returned, each with its source file, line and community.

## 39. output/04 — Path tracing and explanation
- Section 10
- 10
- code2graph path "RequestHandler" "Task" returns a 4-hop route through the imports; code2graph explain "AuthService" lists all 8 relationships with relation type and confidence.

**Speaker notes:** Path tracing and explanation: a four-hop route between RequestHandler and Task, and a full explanation of AuthService including inferred edges.

## 40. output/05 — Change-impact analysis
- Section 10
- 10
- code2graph affected "Model" performs a reverse traversal and lists every node a change to Model can reach — imports, inherits, calls and re-exports.

**Speaker notes:** Change impact: a reverse traversal from Model shows everything a change could reach — imports, inherits, calls and re-exports.

## 41. output/06 — Token-reduction benchmark
- Section 10
- 10
- code2graph benchmark compares the whole corpus against the subgraph an answer needs: ~4.6× fewer tokens per query on this sample repository.

**Speaker notes:** The benchmark command quantifies the benefit: roughly 4.6 times fewer tokens per query than using the whole corpus.

## 42. output/07 — Export formats
- Section 10
- 10
- The same graph is exported as interactive HTML, SVG, GraphML and a Mermaid call-flow report — one graph, many downstream views.

**Speaker notes:** One graph, many exports: interactive HTML, SVG, GraphML and a Mermaid call-flow report.

## 43. output/08 — MCP tool catalog
- Section 10
- 10
- code2graph-mcp serves 12 tools over stdio and starts the background watcher; graph_stats answers with node, edge, community and confidence counts.

**Speaker notes:** The MCP server: 12 tools over stdio, a background watcher, and graph_stats reporting nodes, edges, communities and the confidence split.

## 44. output/09 — MCP graph queries
- Section 10
- 10
- Inside a client session, shortest_path traces RequestHandler → Task in 4 hops and get_neighbors returns every relationship of AuthService with its confidence label.

**Speaker notes:** Inside a client session: shortest_path and get_neighbors, with every relationship carrying its relation type and confidence label.

## 45. output/10 — MCP natural-language query
- Section 10
- 10
- query_graph answers a question in the assistant's own tool call; the reply reports the traversal, the node list and the token budget it respected.

**Speaker notes:** A natural-language query answered through the MCP tool call, including the traversal details and the token budget the server respected.

## 46. output/11 — Broken reference recorded
- Section 10
- 10
- After app/models/task.py is deleted, code2graph update rebuilds the graph to 64 nodes / 100 edges and records the lost relationships as BROKEN metadata.

**Speaker notes:** Healing, part one: a source file is deleted and the update records the lost relationships as BROKEN metadata — no dangling edges and no phantom nodes.

## 47. output/12 — Reference healed
- Section 10
- 10
- When the file returns, the next rebuild reconciles every record: all 9 edges are restored and marked HEALED instead of leaving dangling or phantom nodes.

**Speaker notes:** Healing, part two: when the file returns, all nine records are reconciled and marked HEALED.

## 48. output/13 — Healing diagnostics over MCP
- Section 10
- 10
- heal_status reports the file as HEALTHY with 4 healed references and 0 broken; broken_references confirms the project has no unresolved edges left.

**Speaker notes:** The agent-facing diagnostics: heal_status reports HEALTHY with four healed references, and broken_references confirms nothing is outstanding.

## 49. output/14 — Graph view (graph.json)
- Section 10
- 10
- The persisted graph rendered as a network: node size is degree, colour is community and hub nodes are labelled — the same data the interactive graph.html draws.

**Speaker notes:** The persisted graph drawn as a network — node size is degree and colour is community.

## 50. output/15 — Community detection
- Section 10
- 10
- Louvain clustering over the sample repository, with each community named after its highest-degree node.

**Speaker notes:** Community detection output: nine communities for the sample repository, each named after its highest-degree node.

## 51. output/16 — Edge confidence and relations
- Section 10
- 10
- Every edge carries a confidence label (EXTRACTED / INFERRED / AMBIGUOUS) and a relation type such as calls, imports, imports_from, contains, method or rationale_for.

**Speaker notes:** Every edge has a confidence label and a relation type, which is what makes the graph trustworthy for an assistant.

## 52. output/17 — GRAPH_REPORT.md (1 of 3)
- Section 10
- 10
- report.py writes a human-readable report: corpus summary, extraction confidence split, community hubs and the god nodes of the project.

**Speaker notes:** The generated report, part one: corpus summary, confidence split, community hubs and god nodes.

## 53. output/18 — GRAPH_REPORT.md (2 of 3)
- Section 10
- 10
- Surprising connections show cross-module links with their confidence, followed by import cycles and the detected communities with cohesion scores.

**Speaker notes:** The generated report, part two: surprising cross-module connections, import cycles and the communities with their cohesion scores.

## 54. output/19 — GRAPH_REPORT.md (3 of 3)
- Section 10
- 10
- Knowledge gaps (thin communities) and suggested questions — each pointing at a bridge node the graph is uniquely positioned to explain.

**Speaker notes:** The generated report, part three: knowledge gaps and the suggested questions the graph is uniquely able to answer.

## 55. Conclusion & Future Scope
- Summary
- ▪   Code2Graph turns a source tree into a persistent, queryable knowledge graph of files, symbols and relationships.
- ▪   Structural extraction is local and deterministic — code-only indexing needs no API key and consumes no tokens.
- ▪   Incremental watching plus broken-reference healing keeps the graph trustworthy across edits, renames and deletions.
- ▪   One graph serves a terminal CLI, IDE/agent skills and an MCP server, so any compatible assistant can reuse the same context.
- ▪   Measured on the sample repository: 65 nodes, 110 edges across 15 files, ~4.6× fewer tokens per query than whole-corpus context.
- Future scope
- ▪  Broaden dynamic/runtime analysis so behaviour that static parsing cannot resolve becomes visible in the graph.
- ▪  Expand the semantic layer for documents and media with configurable providers, and grow the language/grammar coverage.
- ▪  Deeper PR and cross-repository impact workflows, and richer human-facing exports (wiki, canvas, dashboards).

**Speaker notes:** In conclusion: Code2Graph gives AI assistants and developers a focused, locally computed, self-healing structural context. Future work is broader dynamic analysis, a richer semantic layer for documents and media, and deeper PR / cross-repository workflows.

## 56. Thank You
- Questions & Discussion
- Team 20  ·  Akshaya Kumar R S  ·  Sachin J  ·  Senthil Pandian G S
- Department of Artificial Intelligence and Data Science, K.L.N. College of Engineering
- Supervisor: Mrs. Ilamathi S., AP / AI & DS

**Speaker notes:** Thank you. We are happy to take questions.
