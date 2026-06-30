# AI301 Contribution
## Contribution [1]: Feature Proposal: Tool Response Compression for Token Optimization
## Contribution [2] : Issue Recreation: Reducing Token Waste in AI Agent Workflows
## Contribution [3] : Issue Solution: Implementing Tool Response Compression for Token Optimization
## Contribution [4] : PR Submission: Implementing Tool Response Compression for Token Optimization
**Contribution Number:** 4  
**Student:** Suhas Ramesh Vittal  
**Issue:** https://github.com/traceloop/opentelemetry-mcp-server/issues/11 
**Status:** Phase IV Completed

---

**Phase I** -> https://github.com/Suhasrv2403/su26-ai301-contribution/blob/main/README.md
**Phase II** -> https://github.com/Suhasrv2403/opentelemetry-mcp-server/blob/feat/tool-response-compression/CONTRIBUTION.md
**Phase III & IV** -> https://github.com/traceloop/opentelemetry-mcp-server/pull/49
## Why I Chose This Issue

I chose this issue because it addresses a real efficiency problem I find compelling: reducing token waste in AI agent workflows.

---

## Understanding the Issue

### Problem Description

This issue aims to reduce the number of tokens consumed by the observability data when queried by the AI agents.
For example, if you query 100 traces, you get 100 objects each repeating the same field names like "model", "provider", 
"count" over and over. That's a lot of wasted tokens.

### Expected Behavior
Return this:

```json
{
  "columns": ["model", "provider", "count"],
  "rows": [["gpt-4", "openai", 48], ["gpt-3.5", "openai", 12]]
}
```

### Current Behavior
Instead of returning this:

```json
[
  {"model": "gpt-4", "provider": "openai", "count": 48},
  {"model": "gpt-3.5", "provider": "openai", "count": 12}
]
```

### Affected Components

- `src/opentelemetry_mcp/tools/` — all 8 tool files that return arrays of objects
- `src/opentelemetry_mcp/config.py` — ServerConfig for the compression toggle
- `src/opentelemetry_mcp/tools/compression.py` — new utility file

---

## Reproduction Process

### Environment Setup

- Cloned the repo and ran `uv sync` to install dependencies
- Had to install `uv` first since it wasn't available globally (`curl -LsSf https://astral.sh/uv/install.sh | sh`)
- Python version pinned to 3.13.1 via `.python-version` — uv handled this automatically
- All commands must be prefixed with `uv run` (e.g. `uv run pytest`, `uv run mypy src/`)
- Resolved 9 ruff import ordering errors across tool files using `uv run ruff check --fix .`
- Fixed mypy errors caused by inconsistent attribute names (`config.compression` vs `config.compress_responses`) across tool files

### Steps to Reproduce

1. Clone the repository: `git clone https://github.com/traceloop/opentelemetry-mcp-server.git`
2. Install dependencies: `uv sync`
3. Query any tool (e.g. `list_llm_models`) with multiple results
4. Observe the JSON response — every object in the array repeats the same field names (`model`, `provider`, `count`, etc.) for every row
5. For a response with 100 traces, field names are duplicated 100 times — wasting 30-60% of tokens on every AI agent call

### Reproduction Evidence

- **Branch:** https://github.com/Suhasrv2403/opentelemetry-mcp-server/tree/feat/tool-response-compression
- **My findings:** Tool responses return arrays of uniform objects where field names repeat on every row. The more rows returned, the more tokens wasted. This directly impacts AI agent API cost, latency, and context window utilization.

---

## Solution Approach

### Analysis

The root cause is that all 8 tool functions build a result dict containing a list of uniform objects and serialize it directly with `json.dumps()`. There is no compression step. Field names are repeated for every row even though they are identical across all objects.

### Proposed Solution

Introduce a `compact_json()` utility that recursively converts uniform arrays of dicts into a column/row tabular format, and hook it into all 8 tool files before the final `json.dumps()` call.

### Implementation Plan

**Understand:** Tool responses return arrays of uniform objects where field names repeat on every row, wasting 30-60% of tokens in AI agent workflows.

**Match:** This maps to a classic data compression pattern — the same approach used in columnar data formats like Apache Parquet and Arrow where repeated keys are deduplicated.

**Plan:**
1. Create `src/opentelemetry_mcp/tools/compression.py` with `compact_json()` — a recursive utility that converts uniform arrays of dicts into `{"columns": [...], "rows": [...]}` format
2. Add `compress_responses: bool = True` to `ServerConfig` in `config.py`, reading from `COMPRESS_RESPONSES` env var
3. Hook `compact_json()` into 8 high-impact tool files before the final `json.dumps()` call
4. Write unit tests in `tests/test_compression.py` covering basic compression, pass-through edge cases, nested structures, threshold behavior, and losslessness
5. Document the config option in `.env.example`

**Implement:** https://github.com/Suhasrv2403/opentelemetry-mcp-server/tree/feat/tool-response-compression

**Review:**
- ✅ `uv run ruff format .`
- ✅ `uv run ruff check .`
- ✅ `uv run mypy src/`
- ✅ `uv run pytest`

**Evaluate:** 17 new unit tests added, all passing. 92 total tests pass, 2 skipped (pre-existing).

---

## Testing Strategy

### Unit Tests

- ✅ Basic compression of uniform array
- ✅ Losslessness — reconstructed data equals original
- ✅ Non-uniform keys pass through unchanged
- ✅ Missing keys pass through unchanged
- ✅ Empty array pass through
- ✅ Single item list pass through
- ✅ Non-dict list pass through
- ✅ Primitives pass through unchanged
- ✅ Below threshold pass through
- ✅ Nested dict with array compresses recursively
- ✅ Multiple nested arrays compress independently
- ✅ Deeply nested compression
- ✅ Custom threshold respected
- ✅ Threshold zero always compresses
- ✅ Real search_traces response shape
- ✅ Real list_models response shape
- ✅ compress_responses disabled skips compression

### Integration Tests

- No existing integration tests were broken by these changes
- Tool files operate at 0% coverage in existing integration tests — they mock at the backend level

### Manual Testing

- Ran full test suite before and after changes — 77 tests passing before, 92 passing after (17 new)
- Verified ruff, mypy, and pytest all pass cleanly

---

## Implementation Notes

### Week 1 Progress

- Identified and claimed issue #11
- Read all 8 tool files to understand response shapes
- Implemented `compact_json()` with recursive dict traversal, uniform key detection, and threshold-based compression
- Added `compress_responses` toggle to `ServerConfig`
- Hooked compression into all 8 tool files via `config` parameter passed from `server.py`
- Wrote 17 unit tests, all passing
- Resolved ruff import ordering errors and mypy attribute name inconsistencies

### Code Changes

- **Files modified:** `config.py`, `server.py`, `tools/list_models.py`, `tools/search.py`, `tools/search_spans.py`, `tools/errors.py`, `tools/expensive_traces.py`, `tools/slow_traces.py`, `tools/list_llm_tools.py`, `tools/trace.py`
- **Files created:** `tools/compression.py`, `tests/test_compression.py`, `CONTRIBUTION.md`
- **Approach decisions:** Used `ServerConfig` parameter injection rather than global import to avoid circular imports; chose savings-ratio threshold over min-rows for more accurate compression gating

---

## Pull Request

**PR Link:** https://github.com/traceloop/opentelemetry-mcp-server/pull/49

**PR Description:** 

I implemented tool response compression for the opentelemetry-mcp-server to reduce token consumption in AI agent 
workflows. I created a compact_json() utility function that recursively converts uniform arrays of objects into a 
compact column/row tabular format, eliminating repeated field names across response rows. I added a compress_responses 
configuration option (enabled by default, toggleable via COMPRESS_RESPONSES env var) and integrated the compression 
logic into all 8 high-impact tool functions. I wrote 17 unit tests covering compression correctness, losslessness, 
edge cases, nested structures, and configuration behavior. After opening the PR, I addressed automated code review 
feedback from CodeRabbit, fixing a recursion bug where nested arrays inside list items weren't being compressed, 
tightening environment variable validation to avoid silently misinterpreting typos as disabling the feature, and 
improving a test to validate the actual environment-parsing contract rather than mutating config directly.

**Status:** Phase IV Complete — PR submitted

---

## Learnings & Reflections

### Technical Skills Gained

- Working with `uv` as a Python package manager
- Recursive JSON transformation patterns
- MyPy strict type checking in a real codebase
- Pydantic `BaseModel` configuration with environment variable parsing

### Challenges Overcome

- Import ordering errors across 9 files — resolved with `uv run ruff check --fix .`
- Inconsistent attribute names across tool files caught by mypy
- Threshold logic bug where `threshold=0.0` wasn't compressing small arrays — fixed by switching from absolute size comparison to savings ratio

### What I'd Do Differently Next Time

- Establish the attribute name convention before touching multiple files to avoid mypy errors
- Write the utility function and tests first before hooking into tool files

---

## Resources Used

- https://github.com/traceloop/opentelemetry-mcp-server/issues/11
- https://traceloop.com/docs/openllmetry/contributing/overview
- https://docs.astral.sh/uv/