# Total.js skills

AI knowledge for the [Total.js Platform](https://www.totaljs.com): what a coding assistant has to know to write correct Total.js 5 code. The same content is published in several shapes, so Claude, ChatGPT / Codex, Gemini, Kimi and other assistants can each read it in the way they support.

## Skills

| Skill | Use it for |
|---|---|
| [totaljs-cookbook](skills/totaljs-cookbook/SKILL.md) | Building Total.js 5 apps: architecture, actions, routing, plugins, auth, PostgreSQL, uploads, WebSockets, jobs, views, SEO, anti-patterns |

## Which file is for which assistant

| File | Read by |
|---|---|
| [skills/totaljs-cookbook/SKILL.md](skills/totaljs-cookbook/SKILL.md) + [references/](skills/totaljs-cookbook/references/) | Agents that support `SKILL.md` skills (Claude Code and others). Loads the rules first and one topic file at a time. |
| [AGENTS.md](AGENTS.md) | Codex / ChatGPT, Kimi and other agents that read `AGENTS.md` |
| [CLAUDE.md](CLAUDE.md), [GEMINI.md](GEMINI.md) | Claude Code and Gemini CLI; both point to the same content |
| [COOKBOOK.md](COOKBOOK.md) | Any chat assistant: the whole cookbook in one file |
| [llms.txt](llms.txt) | Assistants that browse the web: an index with raw URLs of every section |

## Usage

### Chat assistants (ChatGPT, Gemini, Kimi, Claude)

Give the assistant the raw URL and ask it to read it first:

```
Read https://raw.githubusercontent.com/totaljs/skills/main/COOKBOOK.md and follow it for all Total.js code in this conversation.
```

When the assistant cannot open URLs, attach `COOKBOOK.md` as a file, or add it to the project knowledge (ChatGPT Projects / GPTs, Claude Projects, Gemini Gems).

For a single topic, use the URL of one section from [llms.txt](llms.txt).

### Claude Code

All skills as a plugin (updates with the repository):

```
/plugin marketplace add totaljs/skills
/plugin install totaljs@totaljs
```

Or copy one skill into the project (`.claude/skills/`) or your home directory (`~/.claude/skills/`):

```sh
git clone https://github.com/totaljs/skills
cp -R skills/skills/totaljs-cookbook your-project/.claude/skills/
```

### Gemini CLI

```sh
gemini extensions install https://github.com/totaljs/skills
```

### Codex, Kimi and any other coding agent

Put the repository next to your code and point the agent's instruction file to it. This works with every agent that reads `AGENTS.md` (or its own equivalent such as `GEMINI.md`):

```sh
git clone https://github.com/totaljs/skills .ai/totaljs
```

Add to `AGENTS.md` of your project:

```markdown
## Total.js

This is a Total.js 5 project. Before writing or reviewing code, read `.ai/totaljs/skills/totaljs-cookbook/SKILL.md` and the reference files it points to. Follow them over habits from other frameworks.
```

If the agent supports `SKILL.md` skills, you can copy `skills/totaljs-cookbook` into its skills directory instead.

## Structure

```
AGENTS.md                         instructions for agents (repository map, editing rules)
CLAUDE.md, GEMINI.md              pointers for Claude Code and Gemini CLI
llms.txt                          index with raw URLs
COOKBOOK.md                       the whole cookbook in one file (generated)
skills/
	<skill-name>/
		SKILL.md                  required: frontmatter + instructions
		references/               longer docs the agent reads only when needed
		scripts/                  optional: scripts the agent runs
		assets/                   optional: templates and files used in the output
scripts/build.js                  generates COOKBOOK.md from the references
.claude-plugin/                   Claude Code plugin + marketplace manifest
gemini-extension.json             Gemini CLI extension manifest
```

## Editing the cookbook

The source of truth is `skills/totaljs-cookbook/references/*.md`, one topic per file. `COOKBOOK.md` is generated; do not edit it by hand.

```sh
node scripts/build.js
```

Run it after every change and commit the result. When a section is added, renamed or removed, also update the table in `SKILL.md` and the list in `llms.txt`.

## Adding a skill

1. Create `skills/<skill-name>/SKILL.md`. Name the folder `totaljs-<topic>` in lowercase with hyphens, for example `totaljs-views`, `totaljs-flow` or `totaljs-ui`.
2. Start the file with the frontmatter. `name` must be the same as the folder name:

	```markdown
	---
	name: totaljs-<topic>
	description: What the skill does and when to use it, with the words a developer would use in a request.
	---
	```

	The agent reads only the `description` when it decides whether to load the skill, so name the files, functions and tasks that the skill covers.
3. Write the instructions below the frontmatter. Keep `SKILL.md` under about 500 lines and move long reference material into `references/`, linked from `SKILL.md`.
4. Keep one topic in one skill. A separate skill is better than a long one that covers several unrelated parts of the platform.
5. Add a row to the table in this README and a line to `AGENTS.md` and `llms.txt`.
