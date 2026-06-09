# GraphRAG — HUST Knowledge Graph

Fix community report errors when using Microsoft GraphRAG to build a knowledge graph with the DeepSeek V4 Flash model.

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

Or run step-by-step:

```bash
# Initialize project
python3 -m graphrag init --root project

# Copy input data
cp /path/to/data.txt project/input/

# Apply DeepSeek JSON patch
python3 scripts/patch_graphrag.py

# Set API keys
cat > project/.env << EOF
DEEPSEEK_API_KEY=sk-xxx
GRAPHRAG_API_KEY=cf_xxx
EOF

# Copy configuration
cp settings.yaml project/

# Build the index (use TMPDIR for writable temp storage)
cd project
TMPDIR=./tmp python3 -m graphrag index
```

### 3. Export Rich GraphML (for Gephi / yEd)

```bash
python3 scripts/export_graphml.py > project/output/rich_graph.graphml
```

## Output

```text
project/output/
├── entities.parquet          # 573 extracted entities
├── relationships.parquet     # 1,182 relationships with descriptions
├── communities.parquet       # 91 detected communities
├── community_reports.parquet # 89 LLM-generated community reports
├── graph.graphml             # Default minimal GraphML from GraphRAG
├── rich_graph.graphml        # Full GraphML with directions & descriptions
└── lancedb/                  # Vector search indices
```

## Key Files

| File | Purpose |
|------|---------|
| `scripts/graphrag_workflow.py` | End-to-end workflow: install → init → patch → index |
| `scripts/patch_graphrag.py` | Fix the `response_format` issue with DeepSeek |
| `scripts/export_graphml.py` | Export rich GraphML with descriptions and edge directions |
| `settings.yaml` | Configuration template (entity types, chunking, LLMs) |

## Why is the patch needed?

DeepSeek V4 Flash rejects `response_format: json_object` and returns a `405` error.

The patch removes `response_format` from `community_reports_extractor.py` and manually parses JSON from `response.content`.

## Tech Stack

- **LLM**: DeepSeek V4 Flash (via DeepSeek API)
- **Embeddings**: BGE-M3 (via Cloudflare Workers AI)
- **Vector Store**: LanceDB
