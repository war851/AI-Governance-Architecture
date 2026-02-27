# Claude Code: Configuration Sources
> Reference list of all canonical sources for extending and customising the Claude Code configuration guide.  
> Last updated: February 2026

---

## Anthropic Official Docs (Primary)

| Topic | URL |
|---|---|
| Memory & CLAUDE.md hierarchy (the 5 levels) | https://docs.anthropic.com/en/docs/claude-code/memory |
| settings.json full schema & all options | https://docs.anthropic.com/en/docs/claude-code/settings |
| Hooks guide (patterns & examples) | https://code.claude.com/docs/en/hooks-guide |
| Hooks reference (all events & fields) | https://code.claude.com/docs/en/hooks |
| Claude Code root docs index | https://docs.anthropic.com/en/docs/claude-code |

---

## Anthropic Engineering Blog (Best Practices Narrative)

| Topic | URL |
|---|---|
| Agentic coding best practices | https://www.anthropic.com/engineering/claude-code-best-practices |
| Official CLAUDE.md blog post | https://claude.com/blog/using-claude-md-files |

---

## Boris's Site (Workflow Patterns)

| Topic | URL |
|---|---|
| Full site (all 4 threads, 40 tips) | https://howborisusesclaudecode.com/ |

---

## When to Use Which Source

When adding or customising something, use this decision tree:

1. **Is it a new config key or permission flag?**  
   → Start with `https://docs.anthropic.com/en/docs/claude-code/settings`

2. **Is it a new CLAUDE.md level behaviour or @import rule?**  
   → Start with `https://docs.anthropic.com/en/docs/claude-code/memory`

3. **Is it a hook event or automation trigger?**  
   → Start with `https://code.claude.com/docs/en/hooks-guide`, then cross-reference `https://code.claude.com/docs/en/hooks`

4. **Is it a workflow or agentic pattern?**  
   → Start with `https://www.anthropic.com/engineering/claude-code-best-practices`

5. **Is it about daily usage habits, Plan Mode, or subagent patterns?**  
   → Start with `https://howborisusesclaudecode.com/`

---

## Update Cadence

The Anthropic docs update frequently — the `settings` page in particular gained ~15 new keys between December 2025 and February 2026.  
**Recommended: re-read the `settings` and `memory` docs every 2–4 weeks.**

---

*Companion document to: `claude-code-configuration-guide.md`*
