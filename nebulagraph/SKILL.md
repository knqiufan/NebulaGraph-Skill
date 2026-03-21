---
name: nebulagraph
description: Use when the user asks to query NebulaGraph, write nGQL, create a graph space, design a graph schema, find paths in a graph, explore graph neighbors, insert vertices or edges, build a knowledge graph, model graph data, traverse a graph, mentions NebulaGraph, nGQL, graph database schema design, or interacts with nebulagraph-mcp-server MCP tools (list_spaces, get_space_schema, execute_query, find_path, find_neighbors).
---

# NebulaGraph Skill

NebulaGraph is a distributed graph database using **nGQL** (SQL-like + openCypher) with a **strong-schema** model. For comprehensive syntax see `nGQL-reference.md`; for schema design see `data-modeling.md`.

## Workflow

**1. Categorize**: Exploration ("what spaces/schema/data?") | Query ("find/get/show") | Mutation ("insert/create/update/delete") | Schema Design ("design/model").

**2. Execute**:
- **Exploration**: `list_spaces()` → `get_space_schema(space)` → `find_neighbors(vertex, space)` for sampling
- **Query** (check schema first): src+dst path → `find_path()` | around vertex → `find_neighbors()` | pattern/filter → MATCH via `execute_query()` | known VID traversal → GO | by property → LOOKUP | by VID → FETCH | subgraph → GET SUBGRAPH
- **Mutation**: Check schema → compose DDL/DML → `execute_query()`. Warn about ~20s heartbeat delay after schema changes
- **Schema Design**: Load `data-modeling.md` → requirements → VID strategy → tags/edges → indexes → DDL

## MCP Tools

### `list_spaces()` — Discover available graph spaces. No parameters.

### `get_space_schema(space)` — Get tags, edges, indexes. Always call before writing queries.

### `execute_query(query, space)` — Run any nGQL statement. Query first, space second.
- **Auto-prepends `USE space;`** — never include USE in your query
- nGQL uses `==` for equality (not `=`); string literals in `"double quotes"`
- Prefix query with `PROFILE` for execution metrics

### `find_path(src, dst, space, depth=3, limit=10)` — Find all paths between two vertices.
- Searches all edge types bidirectionally (no `edge_types`/`direction` params)
- Default depth 3; increase to 5 for sparse graphs

### `find_neighbors(vertex, space, depth=1)` — Get connections around a vertex.
- Undirected, all edge types (no `edge_types`/`direction`/`limit` params)
- depth=1 for immediate neighbors; increase for broader exploration

## Quick nGQL

```ngql
CREATE SPACE g (vid_type=FIXED_STRING(64), partition_num=20, replica_factor=1);
CREATE TAG person (name STRING NOT NULL, age INT32);
CREATE EDGE knows (since DATETIME, weight DOUBLE DEFAULT 1.0);
CREATE TAG INDEX idx ON person(); REBUILD TAG INDEX idx;
MATCH (v) WHERE id(v) == "p1" RETURN v;
MATCH (a:person)-[e:knows]->(b) WHERE a.person.name == "Tim" RETURN b, e;
MATCH (a)-[e:knows*1..3]->(b) WHERE id(a) == "p1" RETURN DISTINCT b;
GO FROM "p1" OVER knows YIELD dst(edge), knows.weight;
LOOKUP ON person WHERE person.age > 30 YIELD id(vertex), person.name;
FETCH PROP ON person "p1" YIELD person.name, person.age;
FIND SHORTEST PATH FROM "p1" TO "p2" OVER knows YIELD path AS p;
GET SUBGRAPH 2 STEPS FROM "p1" BOTH knows YIELD VERTICES AS v, EDGES AS e;
INSERT VERTEX person (name, age) VALUES "p1":("Alice", 30), "p2":("Bob", 25);
INSERT EDGE knows (since) VALUES "p1"->"p2":(datetime("2023-01-01T00:00:00"));
UPDATE VERTEX ON person "p1" SET age = 31;
DELETE VERTEX "p1" WITH EDGE;
```

## Common Pitfalls

1. **`==` not `=`** for equality in WHERE. `=` is only for SET in UPDATE/UPSERT.
2. **Property access needs tag**: `v.person.name` not `v.name`.
3. **String VIDs need double quotes**: `"p1"`. INT64 VIDs are bare: `100`.
4. **LOOKUP needs pre-built index**. CREATE + REBUILD first.
5. **~20s heartbeat delay** after CREATE/ALTER TAG/EDGE before data operations.
6. **Variable-length edge is a list**: filter with `ALL(x IN e WHERE ...)` not `e.prop`.
7. **`DELETE VERTEX ... WITH EDGE`** to avoid dangling edges.
8. **No `USE` in `execute_query`** — MCP tool handles it automatically.

## Reference Router

Read local files on demand for detailed syntax. Use official doc URLs as fallback for edge cases.

### Core Reference (read when writing queries)

| Topic | Local File | Official Docs |
|-------|-----------|---------------|
| Core syntax (types, DDL/DML, MATCH, GO, paths, indexes) | `nGQL-reference.md` | — |
| Schema design (VID, tags, edges, partitions, patterns) | `data-modeling.md` | — |
| Workflows, error diagnosis, MCP tool patterns | `examples.md` | — |

### Detailed Reference (read only when needed)

| Topic | Local File | Official Docs |
|-------|-----------|---------------|
| Operators (comparison, boolean, string, set, list, precedence) | `topics/operators.md` | https://docs.nebula-graph.com.cn/3.8.0/3.ngql-guide/5.operators/1.comparison/ |
| Built-in functions (math, string, datetime, aggregation, list, type, schema, predicate) | `topics/functions.md` | https://docs.nebula-graph.com.cn/3.8.0/3.ngql-guide/6.functions-and-expressions/1.math/ |
| Clauses (YIELD, SAMPLE, INNER JOIN, WITH, UNWIND, ORDER BY, GROUP BY) | `topics/clauses.md` | https://docs.nebula-graph.com.cn/3.8.0/3.ngql-guide/8.clauses-and-options/yield/ |
| Composite queries, variables, property references ($- $^ $$) | `topics/composite-queries.md` | https://docs.nebula-graph.com.cn/3.8.0/3.ngql-guide/4.variable-and-composite-queries/1.composite-queries/ |
| SHOW statements (all variants) | `topics/show-statements.md` | https://docs.nebula-graph.com.cn/3.8.0/3.ngql-guide/7.general-query-statements/6.show/1.show-charset/ |
| Full-text index (Elasticsearch integration) | `topics/fulltext-index.md` | https://docs.nebula-graph.com.cn/3.8.0/3.ngql-guide/15.full-text-index-statements/1.search-with-text-based-index/ |
| Administration (JOB, KILL QUERY/SESSION, users, roles) | `topics/admin.md` | https://docs.nebula-graph.com.cn/3.8.0/3.ngql-guide/4.job-statements/ |

### Online-Only Reference (no local file, consult official docs directly)

| Topic | Official Docs |
|-------|---------------|
| Graph patterns | https://docs.nebula-graph.com.cn/3.8.0/3.ngql-guide/1.nGQL-overview/3.graph-patterns/ |
| Keywords & reserved words | https://docs.nebula-graph.com.cn/3.8.0/3.ngql-guide/1.nGQL-overview/keywords-and-reserved-words/ |
| Geography (geo functions & types) | https://docs.nebula-graph.com.cn/3.8.0/3.ngql-guide/6.functions-and-expressions/14.geo/ |
| Data type details (numeric, string, datetime, NULL) | https://docs.nebula-graph.com.cn/3.8.0/3.ngql-guide/3.data-types/1.numeric/ |
| Full-text index deployment | https://docs.nebula-graph.com.cn/3.8.0/4.deployment-and-installation/6.deploy-text-based-index/2.deploy-es/ |
| Import/Export tools | https://docs.nebula-graph.com.cn/3.8.0/nebula-importer/use-importer/ |
| Best practices & system design | https://docs.nebula-graph.com.cn/3.8.0/8.service-tuning/2.graph-modeling/ |
