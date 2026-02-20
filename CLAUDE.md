# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

---

## tahsinrk Fork: Read This First

This is a fork of `zhongweili/nanobanana-mcp-server`. The upstream repo is tracked via `git remote upstream`.

### Our Customizations (do not revert)

All fork changes are tagged with `(tahsinrk fork)` comments in the code.

1. **`readOnlyHint: False`** in `tools/generate_image.py` -- the tool writes files to disk, the upstream `True` was incorrect.
2. **Rate limiter** in `tools/generate_image.py` -- sliding-window, default 10 RPM. Configured via `NANOBANANA_RATE_LIMIT_RPM` env var.
3. **Daily budget cap** in `tools/generate_image.py` -- estimated spend tracker, default $5/day. Configured via `NANOBANANA_DAILY_BUDGET_USD` and `NANOBANANA_COST_PER_IMAGE` env vars. Resets at midnight. This is an estimate (not exact API billing) using $0.15/image average.
4. **Descriptive filenames** -- images are named from the prompt (3-5 keywords + date), e.g. `toronto-skyline-golden-hour_feb11-26.png`. Collision-safe with `_2`, `_3` suffixes. Code in `utils/validation_utils.py`, applied in `enhanced_image_service.py`, `pro_image_service.py`, and `image_storage_service.py`.

### Defaults for Pro Model

- **`thinking_level`**: Always `HIGH` unless Tahsin asks to change it or Claude has a reason to lower it (confirm first).
- **`enable_grounding`**: `True` by default. Grounding runs a Google Search before generation to pull visual references for real-world subjects (landmarks, products, people). Keeps images factually accurate. Less useful for abstract/creative prompts but generally harmless to leave on.

### Environment Variables (our additions)

| Variable | Default | Purpose |
|----------|---------|---------|
| `NANOBANANA_RATE_LIMIT_RPM` | `10` | Max generate_image calls per minute |
| `NANOBANANA_DAILY_BUDGET_USD` | `5.00` | Estimated daily spend ceiling |
| `NANOBANANA_COST_PER_IMAGE` | `0.15` | Estimated cost per image for budget tracking |

### Upstream Merge Protocol

When Tahsin asks to check for upstream updates:

```bash
cd "/Users/tkhan/Dropbox/Claude Code/nanobanana-mcp-server"
git fetch upstream
git log upstream/main --oneline -10   # Review what changed
git diff main..upstream/main --stat   # See which files changed
```

**Before merging:** Review changes to these files carefully (they contain our customizations):
- `nanobanana_mcp_server/tools/generate_image.py` (rate limiter, budget cap, readOnlyHint)

**Merge only after reviewing:** `git merge upstream/main`

**If conflicts arise** in our customized files, always preserve our rate limiter and budget cap code. The upstream author's changes should be merged around our additions.

**Never auto-update.** Always review upstream changes before merging. Pin to known-good versions.

### Output Directory

Images go to `/Users/tkhan/Dropbox/Claude Code/nanobanana-images/`. Set via `IMAGE_OUTPUT_DIR` in the MCP config (`~/.claude.json`).



For full architecture documentation, development commands, configuration reference, and troubleshooting, see `ARCHITECTURE.md` in this directory. That file covers the layered architecture, service components, model selection logic, FastMCP integration patterns, and common issues.