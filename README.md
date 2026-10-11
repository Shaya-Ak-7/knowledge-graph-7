# Code2Graph

> **Code2Graph is a temporary project name; the final name is still TBD.**

Code2Graph turns a source tree into a navigable code-intelligence graph. Use it as an agent skill (the invocation depends on the host app), from the terminal, or as an MCP server. When the MCP server is running, its watcher keeps the graph current and records/heals broken references as files change.

## What it helps with

- **Understand a repo:** find entry points, symbols, modules, calls, imports, and dependencies.
- **Trace behavior:** query relationships, neighbors, communities, or the shortest path between two components.
- **Assess a change:** inspect the callers and other nodes affected by a refactor.
- **Keep references current:** saves, deletes, and renames trigger incremental updates; missing endpoints are recorded as `BROKEN`, and matching references can heal when their target returns.
- **Explore outside code:** optional semantic workflows support documents and other media when a model/provider is configured.

## Choose how to use it

### 1. Agent skill — interactive workflow (slash command when supported)

This is the interactive workflow for Claude Code and other supported coding-agent platforms. Python 3.10 or newer is required.

From the Code2Graph source checkout, install the package and MCP dependencies:

```bash
python -m pip install -e '.[mcp]'
```

Install the skill for your agent (Claude example):

```bash
# Global install
code2graph install --platform claude

# Or, from the target project, install project-scoped instructions
code2graph install --project --platform claude
```

Open the target project in Claude Code and run:

```text
/code2graph .
```

For a specific project path, use `/code2graph /path/to/project`. After its first graph is built, ask questions in the agent chat or use commands such as:

```text
/code2graph query "Where does authentication start?"
/code2graph path "AuthService" "UserRepository"
/code2graph explain "SessionManager"
/code2graph ./ --mode deep
/code2graph ./ --update
```

`/code2graph .` builds the graph; later codebase questions can reuse it. Other clients use their own skill/command mechanism; the installer places the right Code2Graph skill or instructions for each one. Run `code2graph install --help` for the platform keys.

#### Agent and editor support

Run the listed installer while the Code2Graph environment is active. The per-client `code2graph <client> install` forms add that client's native setup; the generic `code2graph install --platform <key>` installs the skill for any key shown by `code2graph install --help`. For project-scoped skill installs, run `code2graph install --project --platform <key>` from the target repo where supported. Slash-command syntax varies by client: Claude Code and VS Code Copilot Chat explicitly use `/code2graph`; Cursor, Gemini CLI, Kilo, Kiro, and Antigravity also provide slash-command usage. Other clients load the installed skill or instruction file through their native mechanism—don't assume every app uses that exact slash command.

| Client | Install command |
| --- | --- |
| Claude Code | `code2graph claude install` |
| OpenAI Codex | `code2graph codex install` |
| OpenCode | `code2graph opencode install` |
| Cursor | `code2graph cursor install` |
| Gemini CLI | `code2graph gemini install` |
| VS Code Copilot Chat | `code2graph vscode install` |
| GitHub Copilot CLI | `code2graph copilot install` |
| Kilo Code | `code2graph kilo install` |
| Aider | `code2graph aider install` |
| CodeBuddy | `code2graph codebuddy install` |
| Amp | `code2graph amp install` |
| Generic Agent Skills clients | `code2graph agents install` (`skills` is an alias) |
| OpenClaw | `code2graph claw install` |
| Factory Droid | `code2graph droid install` |
| Trae / Trae CN | `code2graph trae install` / `code2graph trae-cn install` |
| Hermes | `code2graph hermes install` |
| Kiro | `code2graph kiro install` |
| Pi | `code2graph pi install` |
| Devin | `code2graph devin install` |
| Google Antigravity | `code2graph antigravity install` |
| Kimi | `code2graph install --platform kimi` |
| Windows-targeted Claude bundle | `code2graph install --platform windows` |
| Windows-targeted Antigravity bundle | `code2graph install --platform antigravity-windows` |

`windows` and `antigravity-windows` are OS-specific skill bundles, not separate coding apps. For any client that supports stdio MCP, add the server configuration in the MCP section below; skill installation and MCP connection are separate options.

### 2. Terminal CLI

Python 3.10 or newer is required. After installing the package as above, go to the project you want to index:

```bash
cd /path/to/your-project

# Local AST/code indexing; no API key needed
code2graph extract . --code-only

# Ask a graph question or trace a path
code2graph query "How does the request reach the database?"
code2graph path "RequestHandler" "Database"

# Rebuild code manually, or run a standalone watcher
code2graph update .
code2graph watch .
```

The default graph is `code2graph-out/graph.json`; cache and manifest data are kept alongside it. `code2graph --help` lists other commands, including `affected`, `cluster-only`, `label`, `export`, `clone`, `merge-graphs`, `add`, and `diagnose`.

### 3. MCP — live graph and healing tools

Create the initial graph once, as shown above. Then register the MCP server with your agent client. Example stdio configuration (use absolute paths; the client config file varies by app):

```json
{
  "mcpServers": {
    "code2graph": {
      "command": "/absolute/path/to/venv/bin/code2graph-mcp",
      "args": ["/absolute/path/to/your-project/code2graph-out/graph.json"]
    }
  }
}
```

Restart/reload the MCP client after saving its config. The MCP server starts a background watcher for that graph's project. A debounce groups editor events; changed files are re-parsed, unchanged parse data is reused, and the graph is atomically updated. No manual `update` step is needed for ordinary code saves/deletes while the server is running.

Available MCP tools include:

| Tool | Use it to… |
| --- | --- |
| `query_graph` | Search the graph with a natural-language question. |
| `shortest_path` | Trace a connection between two concepts or symbols. |
| `get_node`, `get_neighbors` | Inspect a symbol and its direct relationships. |
| `get_community`, `god_nodes`, `graph_stats` | Understand graph structure and architectural hubs. |
| `broken_references` | List unresolved edges with missing endpoint, source, and reason. |
| `heal_status` | Check broken/healed reference state for a file. |
| `list_prs`, `get_pr_impact`, `triage_prs` | Inspect PR state and graph impact when GitHub access is configured. |

The MCP server also exposes graph report, statistics, hub, surprise, audit, and suggested-question resources. Its in-memory graph cache reloads when the persisted graph changes.

## How automatic healing works

1. A save, create, delete, or rename event is debounced.
2. Only affected, locally parseable files are parsed again; unchanged files remain available as resolution context.
3. The old and refreshed node/edge sets are reconciled.
4. An edge with a missing endpoint is kept out of the active graph and recorded in `graph.broken_references` with `BROKEN` status and a reason—no dangling edge or fake endpoint node is added.
5. If the node returns, the edge can be restored and its record marked `HEALED`. A unique same-file/same-location symbol match can also heal a rename; ambiguous matches are not guessed.
6. `broken_references` and `heal_status` expose this state to the agent.

Set `CODE2GRAPH_WATCH_DEBOUNCE` to change the debounce interval (default `0.75` seconds). Set `CODE2GRAPH_OUT` before starting the process to use a different output directory.

## Supported inputs and limits

The local structural pipeline covers many programming languages through dedicated parsers and resolvers. Code indexing does not need an API key. Markdown-family files with local structural extraction can be refreshed locally. PDFs, images, and other semantic-only content need an LLM-capable extraction workflow; without a configured provider, Code2Graph flags pending semantic work instead of claiming it parsed the content. Dynamic or ambiguous language behavior may not be fully resolvable by static analysis.

## Development and tests

```bash
uv sync --extra mcp --extra terraform --group dev
uv run pytest tests -q
```

`tests/` covers language extraction, imports/resolution, graph validation and deduplication, caches, CLI/MCP behavior, file deletion, incremental watching, and broken-reference healing. Skill artifacts are generated from `tools/skillgen/` and checked with `python -m tools.skillgen --check`.

See [ARCHITECTURE.md](ARCHITECTURE.md) and [docs/how-it-works.md](docs/how-it-works.md) for implementation details.

## Project status

Code2Graph is experimental. Its name is provisional, and MCP schemas, graph metadata, and command surface may evolve.

## License

See `LICENSE`, `LICENSE-MIT`, and `NOTICE` for the applicable license terms and required notices.
