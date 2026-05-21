# GraphRAG — HUST Knowledge Graph

Build a domain-specific knowledge graph from training regulations using [GraphRAG](https://microsoft.github.io/graphrag/).

## Quick Start

### 1. Install

```bash
pip install graphrag pandas pyarrow
```

### 2. Run (single command)

```bash
python3 scripts/graphrag_workflow.py \
  --data /path/to/your/text.txt \
  --work-dir project \
  --deepseek-key sk-xxx \
  --cloudflare-key cf_xxx
```

Or step-by-step:

```bash
# Init project
python3 -m graphrag init --root project

# Copy data
cp /path/to/data.txt project/input/

# Patch DeepSeek JSON fix
python3 scripts/patch_graphrag.py

# Set API keys
cat > project/.env << EOF
DEEPSEEK_API_KEY=sk-xxx
GRAPHRAG_API_KEY=cf_xxx
EOF

# Copy settings
cp settings.yaml project/

# Index (use TMPDIR for writable temp)
cd project
TMPDIR=./tmp python3 -m graphrag index
```

### 3. Export rich GraphML (for Gephi / yEd)

```bash
python3 scripts/export_graphml.py > project/output/rich_graph.graphml
```

## Output

```
project/output/
├── entities.parquet          # 573 entities
├── relationships.parquet     # 1,182 relationships with descriptions
├── communities.parquet       # 91 communities
├── community_reports.parquet # 89 LLM-generated reports
├── graph.graphml             # Minimal GraphML (GraphRAG default)
├── rich_graph.graphml        # Full GraphML (directed, with descriptions)
└── lancedb/                  # Vector search indices
```

## Key Files

| File | Purpose |
|------|---------|
| `scripts/graphrag_workflow.py` | End-to-end: install → init → patch → index |
| `scripts/patch_graphrag.py` | Fix `response_format` issue with DeepSeek |
| `scripts/export_graphml.py` | Export rich GraphML with descriptions & direction |
| `settings.yaml` | Config template (entity types, chunking, LLMs) |

## Why the patch?

DeepSeek V4 Flash rejects `response_format: json_object` (405 error). The patch removes `response_format` from `community_reports_extractor.py` and parses JSON manually from `response.content`.

## Tech Stack

- **LLM**: DeepSeek V4 Flash (via DeepSeek API)
- **Embeddings**: BGE-M3 (via Cloudflare Workers AI)
- **Vector Store**: LanceDB
