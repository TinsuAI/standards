# AI collaboration policy

> **Status: stub.** Awaiting RFC.

This file will codify how AI agents (Claude, Codex, Gemini, …)
collaborate inside TinsuAI repos. Specifically: scope discipline,
commit-message conventions when AI authored, the boundary between
"AI executes" and "AI proposes and asks", how to record decisions
when AI participated.

Tentative principles (to be ratified):

- **Skill locale**: each repo's `AGENTS.md` (= `CLAUDE.md`) is the
  contract. AI agents read it before any non-trivial action.
- **Memory locale**: per-machine, per-project memory under
  `~/.claude/projects/.../memory/MEMORY.md` is private to the
  developer's machine. Don't lift those into committed docs.
- **Decision discipline**: AI proposes architecture / data /
  config changes; the human user explicitly confirms before AI
  executes. "Don't be a black box" — brief the user after a
  non-trivial fix.
- **Commit attribution**: no `Co-Authored-By` trailer for AI.
  Authorship is the human running the session.
- **Scope creep**: AI MAY NOT silently bundle unrelated changes.
  If a fix turns out to need scope expansion, AI surfaces the
  expansion before committing.
- **Cross-repo coordination**: when AI in repo A would change
  something that affects repo B, AI does not edit repo B; it tells
  the user, who runs an AI session in repo B.

Until ratified, follow the per-developer conventions in
`~/dotfiles/ai/RULES.md` and each repo's `AGENTS.md`.
