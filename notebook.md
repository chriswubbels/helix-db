# Project Notebook

> Living document for decisions, dead ends, and work state.
> Update via the `notebook` skill. Keep entries dated.
> Decisions can be terse. Dead ends and state need enough detail to avoid re-treading.

## Decisions

[2026-03-30] Intuition retrieval hook uses "deliver last, search now" pattern — results from prompt N arrive at prompt N+1 to avoid blocking. One-turn delay is acceptable for background context. (file: ~/.claude/hooks/intuition-retrieve.py)

[2026-03-30] Sonnet (not Haiku) for retrieval subagent — relevance assessment requires genuine reasoning about whether an idea connects to current work; Haiku would just trust similarity scores. (file: ~/.claude/hooks/intuition-retrieve-worker.py)

[2026-03-30] Multi-layer filtering: subagent rates poor/good/excellent → worker discards poor → hook requires 1 excellent OR 2+ good before injecting. Silence is the expected outcome. (file: ~/.claude/hooks/intuition-retrieve.py)

[2026-03-30] Leave Helix DB on Luna — always-on server, all three hosts (waterhouse, dengo, luna) can reach it, latency negligible over LAN.

[2026-03-30] Added `defaultMode: auto` to global Claude settings for persistent autonomous permissions without `--dangerously-skip-permissions` flag. (file: ~/.claude/settings.json)

## Dead Ends

[2026-03-30] intuition-context.py (SessionStart hook) tried synchronous subagent search at session start — timed out consistently because MCP server startup + embedding + search takes 40-60s, far exceeding hook timeout. Resolution: async two-phase pattern in intuition-retrieve.py.

[2026-03-30] Initial subagent timeout of 45s too tight — first real search timed out. MCP server startup + embedding generation + search + reasoning needs ~40-60s. Resolution: bumped to 90s. (file: ~/.claude/hooks/intuition-retrieve-worker.py)

## Open Questions

Should Intuition capture bar be raised? 709 ideas in 3 months is aggressive; many are project-specific rather than cross-project insights. Consider telling Haiku to only capture insights useful *in a different project*.

Is Intuition retrieval actually being used proactively? The CLAUDE.md instruction to "search when something feels familiar" doesn't work because task pressure always wins. The new hook addresses this — monitor whether it actually surfaces useful ideas in practice.

Should a cooldown be added to the retrieval hook? Every prompt spawns a sonnet subagent (~40-60s, real API cost). Rapid-fire prompts could add up. Consider skipping if last search was <60s ago.

## Current State

**Done:**
- Helix/Intuition evaluation: 709 ideas, 60/102 active days, 3 hosts, decent quality
- Intuition retrieval hook implemented and tested end-to-end
- `defaultMode: auto` configured globally

**In flight:**
- Retrieval hook is live but needs real-world observation across multiple sessions
- Need to verify hook fires correctly on other hosts (dengo, luna) — hooks are in ~/.claude/ which is per-host

**Next:**
- Deploy retrieval hooks to dengo and luna if results are promising on waterhouse
- Consider one-time pruning of low-value ideas from Helix
- Monitor retrieval log for injection frequency and quality

## Key Files

- `~/.claude/hooks/intuition-retrieve.py` — UserPromptSubmit hook: delivers previous results, spawns background search
- `~/.claude/hooks/intuition-retrieve-worker.py` — Background worker: runs sonnet subagent to search + assess relevance
- `~/.claude/hooks/intuition.py` — Stop hook: captures ideas from conversations (existing)
- `~/.claude/hooks/intuition-worker.py` — Background worker for idea capture (existing)
- `~/.intuition/retrieval_results/` — Results handoff directory between worker and hook
- `~/.intuition/retrieval_state.json` — Tracks last-processed message UUID per session
- `~/.intuition/retrieval_log.txt` — Diagnostic log for retrieval operations
