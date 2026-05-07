# AGENTS.md

## What this repo is

A Master's thesis writing notes repository titled **"数字孪生驱动的工业机器人关键动力学参数辨识"** (Digital Twin-Driven Key Dynamics Parameter Identification of Industrial Robots). All content is in Chinese. This is a documentation/notes repo — not a code project.

## File structure

| Path | Purpose |
|---|---|
| `论文内容.19501994.md` | Main thesis content (Chapters 1–6) — **primary file to edit** |
| `毕业论文笔记一.19191263.md` | Bilingual notes on a review paper |
| `毕业论文笔记二.19473709.md` | Notes on excitation trajectory optimization |
| `工作汇报.19416035.md` | Weekly work progress reports to advisor |
| `sync_notes.bat` | Copies all `.md` files to `E:\apps\vscode\cnblogs_documents\note` for cnblogs publishing |
| `Proposal&Design/` | Word documents (proposal, thesis draft, progress reports) |
| `png/` | Image assets referenced by markdown |
| `.github/skills/wenshu-writing-compliance/SKILL.md` | Writing compliance review skill |

## File naming convention

Files use the pattern `<name>.<numeric_id>.md`. The numeric ID (e.g., `19501994`) is the cnblogs post ID. Keep this suffix when creating new note files — it is used by the sync/publishing workflow.

## Writing compliance skill (repo-local)

The repo has a custom skill at `.github/skills/wenshu-writing-compliance/SKILL.md` that enforces:
- Replace "机械臂" with "机器人" (unless explicitly requested otherwise)
- Simplify parenthetical expressions to comma-separated clauses
- Avoid `**bold**:` heading patterns; use numbered lists instead
- Chinese punctuation and academic writing conventions

Run this skill (`mode=full` or `mode=diff`) after any markdown edits.

## No build/lint/test

There are no build commands, linters, or test suites. This is purely a documentation repo.

## Git conventions

Commit messages are in Chinese and describe the content version or changes (e.g., "内容版本0.0.5 删除了很多小结").

## Monthly notes save convention (global skill)

Triggered by user keywords like "存档", "保存到月度笔记", "追加到月记".

- **Target directory**: `E:\apps\vscode\cnblogs_documents\note\`
- **File pattern**: `<YY>_x月.<cnblogs_post_id>.md` where `<YY>` = last two digits of current year
- **File selection** (priority):
  1. User specifies filename → use directly
  2. Glob search `<YY>_x月.*.md` in target directory → take first match
  3. No match → tell user to create the monthly cnblogs post first
- **Append format**: `# <YY>.<M>.<DD>` heading (e.g. `# 26.4.29`), blank line, then content
- **Same-day merge**: if heading already exists, append content under the same heading (do not duplicate it)
- Use markdown formatting; preserve tables, code blocks, LaTeX, Mermaid diagrams, etc.
- Uses the global skill (not repo-local): `save-monthly-note`
