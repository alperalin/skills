---
name: extract-skill
description: >
  Analyze the current conversation context to extract development patterns,
  tooling decisions, and rules into a reusable skill file. Use after completing
  non-trivial development tasks to capture learnings as skills.
user-invocable: true
argument-hint: "[skill-name]"
tools:vscode, execute, read, agent, browser, edit, search, web, todo
---

# Extract Skill from Context

You are a meta-skill that analyzes the current conversation to extract reusable development patterns and produce a new skill file.

## Process

### Phase 1: Context Analysis

Review the entire conversation history and identify:
- What type of work was done (feature, bugfix, refactor, migration, etc.)
- Which packages/directories were touched
- What tools and commands were used
- What patterns and conventions were followed
- What decisions were made and why
- What mistakes were made and corrected
- What the user explicitly asked for vs. what was inferred

Summarize your findings to the user before proceeding.

### Phase 2: Pattern Extraction (Interactive)

For each identified pattern, ask the user whether it should be captured as a rule. Use AskUserQuestion for each category:

**Category 1 — Scope & Trigger**
Ask: "What types of future tasks should this skill apply to?"
Offer concrete options derived from the work done (e.g., "API endpoint changes", "Database migration tasks", "Component refactoring").

**Category 2 — File Map**
Ask: "Should this skill include a file index like wql-architecture?"
If yes, list the key files touched and ask which ones are essential reference points vs. incidental.

**Category 3 — Rules & Conventions**
For each pattern you detected, ask: "Should this become a rule?"
Examples:
- "You always ran tests after modifying X — should this be a rule?"
- "You used pattern Y for Z — should future tasks follow this?"
- "You avoided doing A because of B — should this be documented?"

**Category 4 — Tooling & Commands**
Ask: "Which commands/tools used in this session should be included in the skill?"
List the specific commands that were used and let the user confirm/reject each.

**Category 5 — Quality Gates**
Ask: "What checks should be mandatory before considering a task done?"
Suggest options based on what was done (tests, lint, type check, manual verification, etc.)

### Phase 3: Skill Generation

Based on confirmed patterns, generate the skill file(s):

1. Create `.claude/skills/{skill-name}/SKILL.md` with:
   - Proper frontmatter (name, description under 250 chars, user-invocable)
   - Separation of concerns / architecture overview (if applicable)
   - Rules and conventions
   - Tooling and commands
   - Quality gates

2. If a file index is needed, create separate `*-index.md` files following the progressive disclosure pattern (keep SKILL.md under ~100 lines, put details in supporting files referenced via `${CLAUDE_SKILL_DIR}/`).

3. Show the user the generated skill content and ask for approval before writing.

### Phase 4: Refinement

After generating, ask:
- "Is there anything from this session that I missed?"
- "Should any of these rules be stricter or more relaxed?"
- "Do you want to merge any of this into an existing skill instead?"

Apply feedback and finalize.

## Guidelines

- **Ask often, assume little.** Use AskUserQuestion liberally. The user knows their intent better than you can infer.
- **Be specific, not generic.** Rules like "write clean code" are useless. Rules like "link columns must use X1 strategy for 1:1 relations" are useful.
- **Capture the WHY.** For each rule, include why it exists. A rule without reason will be ignored or misapplied.
- **Respect existing skills.** Before creating a new skill, check if an existing one should be updated instead. Read `.claude/skills/` first.
- **Progressive disclosure.** SKILL.md should be concise (~100 lines). Move file indexes, examples, and detailed references to supporting files.
- **Description quality matters.** Front-load action verbs and trigger scenarios in the description so Claude auto-invokes correctly.
- **Don't capture the obvious.** Skip patterns that are standard practice or already enforced by linters/types. Focus on project-specific decisions.
