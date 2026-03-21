# NebulaGraph Skill

A Claude Code skill that enables intelligent interaction with NebulaGraph databases through the nebulagraph-mcp-server. It provides nGQL query generation, schema design guidance, and workflow orchestration for graph database operations.

## File Structure

```
nebulagraph/
  SKILL.md              # Entry point — workflow, MCP tools, quick patterns, Reference Router
  nGQL-reference.md     # Core nGQL syntax — types, DDL/DML, MATCH, GO, functions, indexes
  data-modeling.md      # Schema design guide — VID strategy, tags, edges, partitions, patterns
  examples.md           # Scenario-based examples — exploration, query composition, error diagnosis
  topics/               # Detailed reference (read on demand)
    operators.md        # Comparison, boolean, string, set, list operators & precedence
    functions.md        # All built-in function signatures & examples
    clauses.md          # YIELD, SAMPLE, INNER JOIN, WITH, UNWIND, ORDER BY, GROUP BY
    composite-queries.md # Pipe, variables, property references ($- $^ $$)
    show-statements.md  # All SHOW statement variants
    fulltext-index.md   # Elasticsearch full-text search integration
    admin.md            # JOB management, KILL QUERY/SESSION, users & roles
```

### Three-Layer Progressive Disclosure

| Layer | Content | When Loaded |
|-------|---------|-------------|
| **L1** | `SKILL.md` — workflow, quick syntax, common pitfalls, Reference Router | Always (on skill activation) |
| **L2** | `nGQL-reference.md`, `data-modeling.md`, `examples.md`, `topics/*.md` | On demand (when deeper syntax needed) |
| **L3** | Official doc URLs in Reference Router | Fallback (edge cases: geography, keywords, deployment, etc.) |

## Prerequisites

1. A running NebulaGraph cluster (v3.x)
2. [nebulagraph-mcp-server](https://github.com/nebula-contrib/nebulagraph-mcp-server) installed and configured

Install the MCP server:

```bash
pip install nebulagraph-mcp-server
```

Add to your Claude Code MCP config (`~/.claude/settings.json` or project `.claude/settings.json`):

```json
{
  "mcpServers": {
    "nebulagraph": {
      "command": "python",
      "args": ["-m", "nebulagraph_mcp_server"],
      "env": {
        "NEBULA_VERSION": "v3",
        "NEBULA_HOST": "127.0.0.1",
        "NEBULA_PORT": "9669",
        "NEBULA_USER": "root",
        "NEBULA_PASSWORD": "nebula"
      }
    }
  }
}
```

> **Note**: `NEBULA_VERSION` must be `v3`. Adjust `NEBULA_HOST` and `NEBULA_PORT` to match your cluster.

## Installation

Copy the `nebulagraph/` directory to one of these locations:

**Global** (available in all projects):
```
~/.claude/skills/nebulagraph/
```

**Project-level** (available only in this project):
```
<project-root>/.claude/skills/nebulagraph/
```

## Verifying Installation

1. Start a new Claude Code session
2. Try a trigger phrase like "Query NebulaGraph for all spaces" or "Design a graph schema for user relationships"
3. Claude should activate the skill and use the MCP tools

## Trigger Phrases

The skill activates when you mention:
- NebulaGraph, nGQL, graph database schema
- Graph space, graph paths, graph neighbors
- Knowledge graph, graph data modeling
- MCP tool names: `list_spaces`, `get_space_schema`, `execute_query`, `find_path`, `find_neighbors`
- Actions: query/write nGQL, create graph space, design graph schema, find paths, explore neighbors, insert vertices/edges, traverse a graph

## MCP Tools Available

| Tool | Purpose |
|------|---------|
| `list_spaces()` | Discover available graph spaces |
| `get_space_schema(space)` | Inspect tags, edges, indexes |
| `execute_query(query, space)` | Run any nGQL statement |
| `find_path(src, dst, space, depth, limit)` | Find paths between two vertices |
| `find_neighbors(vertex, space, depth)` | Explore connections around a vertex |

## Customization

- Edit `SKILL.md` frontmatter `description` to adjust trigger conditions
- Add domain-specific patterns to `examples.md`
- Extend `nGQL-reference.md` with custom functions or UDFs
- Add organization-specific modeling conventions to `data-modeling.md`
- Add new topic files under `topics/` and register them in the Reference Router table in `SKILL.md`
- Add official doc URLs to the Online-Only Reference table for topics without local files
