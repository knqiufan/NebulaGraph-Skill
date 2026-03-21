# Industry Knowledge Graph Schema Design

## Overview

A comprehensive industrial knowledge graph on NebulaGraph, combining macro industry chain structure with micro enterprise relationships. Single-space architecture with FIXED_STRING VIDs.

**Scale**: Medium (1-50W enterprises, hundreds of industries, millions of edges)

**Core Query Scenarios**:
1. Industry chain upstream/downstream traversal
2. Enterprise association path discovery
3. Industry aggregation analysis
4. Supply chain risk analysis

## Space & VID Strategy

```ngql
CREATE SPACE industry_kg (
  vid_type = FIXED_STRING(64),
  partition_num = 60,
  replica_factor = 1    -- production: 3
);
```

### VID Prefix Convention

| Entity Type | VID Format | Example |
|------------|-----------|---------|
| Industry | `ind_<code>` | `"ind_semiconductor"` |
| Company | `com_<credit_code>` | `"com_91310000MA1FL8K42A"` |
| Product | `prd_<id>` | `"prd_chip_7nm"` |
| Person | `per_<id>` | `"per_zhangsan_001"` |
| Region | `reg_<admin_code>` | `"reg_310000"` |

## Tags

```ngql
CREATE TAG industry (
  name        STRING NOT NULL,
  level       INT8,
  category    STRING,
  description STRING
);

CREATE TAG company (
  name            STRING NOT NULL,
  short_name      STRING,
  credit_code     STRING,
  legal_person    STRING,
  registered_capital DOUBLE,
  founded_date    DATE,
  status          STRING DEFAULT "active",
  employee_count  INT32
);

CREATE TAG product (
  name        STRING NOT NULL,
  category    STRING,
  tech_level  STRING
);

CREATE TAG person (
  name        STRING NOT NULL,
  title       STRING,
  nationality STRING
);

CREATE TAG region (
  name    STRING NOT NULL,
  level   INT8,
  code    STRING
);
```

## Edge Types

```ngql
-- Industry chain
CREATE EDGE upstream (description STRING);
CREATE EDGE sub_industry ();

-- Company-Industry
CREATE EDGE belongs_to (role STRING);

-- Company-Company
CREATE EDGE supply (
  product     STRING,
  start_date  DATE,
  is_core     BOOL DEFAULT false
);

CREATE EDGE invest (
  share_ratio DOUBLE,
  amount      DOUBLE,
  invest_date DATE,
  round       STRING
);

CREATE EDGE cooperate (
  type        STRING,
  start_date  DATE,
  end_date    DATE
);

CREATE EDGE compete (field STRING);

-- Person-Company
CREATE EDGE serve_as (
  position    STRING NOT NULL,
  start_date  DATE,
  end_date    DATE
);

-- Geography
CREATE EDGE locate_in (type STRING DEFAULT "registered");
CREATE EDGE parent_region ();

-- Product
CREATE EDGE produce (market_share DOUBLE, capacity STRING);
CREATE EDGE product_of ();
```

## Index Strategy

```ngql
-- Tag-level indexes (enable MATCH by tag)
CREATE TAG INDEX idx_industry ON industry();
CREATE TAG INDEX idx_company  ON company();
CREATE TAG INDEX idx_product  ON product();
CREATE TAG INDEX idx_person   ON person();
CREATE TAG INDEX idx_region   ON region();

-- Property indexes (high-frequency filters)
CREATE TAG INDEX idx_industry_name  ON industry(name(64));
CREATE TAG INDEX idx_company_name   ON company(name(64));
CREATE TAG INDEX idx_person_name    ON person(name(64));
CREATE TAG INDEX idx_company_status ON company(status(16));
CREATE TAG INDEX idx_industry_level ON industry(level);
CREATE TAG INDEX idx_region_code    ON region(code(12));

-- Edge indexes (only where LOOKUP needed)
CREATE EDGE INDEX idx_supply_core   ON supply(is_core);
CREATE EDGE INDEX idx_invest_ratio  ON invest(share_ratio);

-- Rebuild all (one per statement)
REBUILD TAG INDEX idx_industry;
REBUILD TAG INDEX idx_company;
REBUILD TAG INDEX idx_product;
REBUILD TAG INDEX idx_person;
REBUILD TAG INDEX idx_region;
REBUILD TAG INDEX idx_industry_name;
REBUILD TAG INDEX idx_company_name;
REBUILD TAG INDEX idx_person_name;
REBUILD TAG INDEX idx_company_status;
REBUILD TAG INDEX idx_industry_level;
REBUILD TAG INDEX idx_region_code;
REBUILD EDGE INDEX idx_supply_core;
REBUILD EDGE INDEX idx_invest_ratio;
```

## Query Patterns by Scenario

### 1. Industry Chain Traversal

```ngql
-- 3-hop upstream traversal from a given industry
MATCH (a:industry)-[e:upstream*1..3]->(b:industry)
WHERE a.industry.name == "半导体"
RETURN b.industry.name AS industry, length(e) AS depth
ORDER BY depth;

-- Traverse to companies in upstream industries
MATCH (a:industry)<-[:belongs_to]-(c:company)-[:belongs_to]->(b:industry),
      (a)-[:upstream]->(b)
WHERE a.industry.name == "新能源汽车"
RETURN b.industry.name AS upstream_industry, collect(c.company.name) AS companies;
```

### 2. Enterprise Path Discovery

```ngql
-- Shortest path between two companies
FIND SHORTEST PATH FROM "com_A" TO "com_B"
OVER invest, supply, cooperate, serve_as
YIELD path AS p;

-- Or use MCP: find_path(src="com_A", dst="com_B", space="industry_kg", depth=4)
```

### 3. Industry Aggregation

```ngql
-- Company count and avg capital in an industry
MATCH (c:company)-[:belongs_to]->(i:industry)
WHERE i.industry.name == "芯片设计"
RETURN count(c) AS company_count,
       avg(c.company.registered_capital) AS avg_capital;

-- Geographic distribution
MATCH (c:company)-[:belongs_to]->(i:industry),
      (c)-[:locate_in]->(r:region)
WHERE i.industry.name == "半导体"
RETURN r.region.name AS city, count(c) AS cnt
ORDER BY cnt DESC LIMIT 20;
```

### 4. Supply Chain Risk

```ngql
-- Core suppliers of a company
MATCH (supplier:company)-[s:supply]->(target:company)
WHERE id(target) == "com_A" AND s.supply.is_core == true
RETURN supplier.company.name AS core_supplier, s.supply.product AS product;

-- Single-point dependency detection
MATCH (supplier:company)-[s:supply]->(target:company)
WHERE s.supply.is_core == true
WITH target, collect(supplier) AS suppliers
WHERE size(suppliers) == 1
RETURN target.company.name AS at_risk_company;

-- Tier-2 supply chain penetration
MATCH (s2:company)-[e1:supply]->(s1:company)-[e2:supply]->(target:company)
WHERE id(target) == "com_A" AND e2.supply.is_core == true
RETURN s1.company.name AS tier1, s2.company.name AS tier2, e1.supply.product;
```

## Design Decisions

| Decision | Rationale |
|----------|-----------|
| Single space | All 4 query scenarios need cross-entity traversal; multi-space would break path queries |
| FIXED_STRING(64) VID | Human-readable, supports natural keys (credit codes), easy external system integration |
| VID prefixes | Prevent collisions across entity types, quick type identification |
| `upstream` single direction | Convention-based, reverse via REVERSELY/bidirectional MATCH |
| `invest` shared edge type | Serves both com→com and per→com without duplication |
| `serve_as.end_date = NULL` for active | Simpler than extra status field |
| Minimal edge indexes | Only `supply.is_core` and `invest.share_ratio` to avoid write amplification |
| partition_num = 60 | Fits 3-node cluster; adjust to 20 × disk count in production |
