# NebulaGraph Skill for Claude Code

A Claude Code skill that enables intelligent interaction with NebulaGraph databases through the nebulagraph-mcp-server. It provides nGQL query generation, schema design guidance, and workflow orchestration for graph database operations.

## File Structure

```
nebulagraph/
  SKILL.md              # Entry point — workflow decision tree, MCP tool reference, quick patterns
  nGQL-reference.md     # Complete nGQL syntax reference — types, DDL/DML, MATCH, GO, functions, indexes
  data-modeling.md      # Schema design guide — VID strategy, tags, edges, partitions, patterns
  examples.md           # Scenario-based examples — exploration, query composition, error diagnosis
```

## Prerequisites

1. A running NebulaGraph cluster (v3.x)
2. [nebulagraph-mcp-server](https://github.com/nicholasgasior/nebulagraph-mcp-server) configured in your Claude Code MCP settings

Add to your Claude Code MCP config (`~/.claude/settings.json` or project `.claude/settings.json`):

```json
{
  "mcpServers": {
    "nebulagraph": {
      "command": "python",
      "args": ["-m", "nebulagraph_mcp_server"],
      "env": {
        "NEBULA_ADDRESS": "127.0.0.1:9669",
        "NEBULA_USER": "root",
        "NEBULA_PASSWORD": "nebula"
      }
    }
  }
}
```

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
