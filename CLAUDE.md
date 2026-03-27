# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

---

## tahsinrk Fork: Read This First

This is a fork of `zhongweili/nanobanana-mcp-server`. The upstream repo is tracked via `git remote upstream`.

#### Our Customizations (do not revert)

All fork changes are tagged with `(tahsinrk fork)` comments in the code.

1. **`readOnlyHint: False`** in `tools/generate_image.py` -- the tool writes files to disk, the upstream `True` was incorrect.
2. **Rate limiter** in `tools/generate_image.py` -- sliding-window, default 10 RPM. Configured via `NANOBANANA_RATE_LIMIT_RPM` env var.
3. **Daily budget cap** in `tools/generate_image.py` -- estimated spend tracker, default $5/day. Configured via `NANOBANANA_DAILY_BUDGET_USD` and `NANOBANANA_COST_PER_IMAGE` env vars. Resets at midnight. This is an estimate (not exact API billing) using $0.15/image average.
4. **Descriptive filenames** -- images are named from the prompt (3-5 keywords + date), e.g. `toronto-skyline-golden-hour_feb11-26.png`. Collision-safe with `_2`, `_3` suffixes. Code in `utils/validation_utils.py`, applied in `enhanced_image_service.py`, `pro_image_service.py`, and `image_storage_service.py`.

#### Model Tiers

- **NB2** (Nano Banana 2, `gemini-3.1-flash-image-preview`): Default model. Flash speed + 4K resolution. No thinking_level support.
- **Pro** (`gemini-3-pro-image-preview`): Highest quality. 4K. Supports `thinking_level` (default: `None`/auto, set `high` for max quality) and `enable_grounding` (default: `True`). NB2 also supports grounding.
- **Flash** (`gemini-2.5-flash-image-preview`): Legacy speed tier. NB2 is better in most cases.

#### Environment Variables (our additions)

| Variable | Default | Purpose |
|----------|---------|---------|
| `NANOBANANA_RATE_LIMIT_RPM` | `10` | Max generate_image calls per minute |
| `NANOBANANA_DAILY_BUDGET_USD` | `5.00` | Estimated daily spend ceiling |
| `NANOBANANA_COST_PER_IMAGE` | `0.15` | Estimated cost per image for budget tracking |

#### Upstream Merge Protocol

Upstream uses `master` branch (not `main`). The fork and upstream have unrelated git histories, so `git merge` fails. Use the "rebase from upstream" approach: create a branch from upstream/master, re-apply our customizations, then force-push to our master.

```bash
cd "/Users/tkhan/Dropbox/Claude Code/claude-skills/nanobanana-mcp-server"
git fetch upstream master
git log upstream/master --oneline -10   # Review what changed
git diff master..upstream/master --stat   # See which files changed
```

**Before merging:** Review changes to these files carefully (they contain our customizations):
- `nanobanana_mcp_server/tools/generate_image.py` (rate limiter, budget cap, readOnlyHint)

**After taking upstream's code:** Re-apply all items from "Our Customizations" above, plus preserve our CLAUDE.md and `.github/workflows/check-upstream.yml`. Never auto-update — always review upstream changes before merging.

#### Output Directory

Images go to `/Users/tkhan/Dropbox/Claude Code/claude-skills/nanobanana-mcp-server/nanobanana-images/`. Set via `IMAGE_OUTPUT_DIR` in the MCP config (`~/.claude.json`).

For full architecture docs, dev commands, and troubleshooting, see `ARCHITECTURE.md`.
