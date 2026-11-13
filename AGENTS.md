# Total.js skills

This repository holds AI knowledge for the [Total.js Platform](https://www.totaljs.com). It contains documentation only, no application code.

## If you are here to learn how to write Total.js code

1. Read [skills/totaljs-cookbook/SKILL.md](skills/totaljs-cookbook/SKILL.md). It has the rules that always apply and a table that maps a task to the reference file to read.
2. Read the matching files in [skills/totaljs-cookbook/references/](skills/totaljs-cookbook/references/) before writing code.
3. If you can load only one file, read [COOKBOOK.md](COOKBOOK.md). It is the same content in one file.

Total.js is a global/singleton framework, not Express or Nest. Follow the cookbook over habits from other frameworks and over memory of Total.js 3/4.

## Map

```
AGENTS.md                         this file (Codex / ChatGPT, Kimi, and other agents)
CLAUDE.md, GEMINI.md              pointers to this file for Claude Code and Gemini CLI
llms.txt                          index with raw URLs for assistants that read from the web
COOKBOOK.md                       the whole cookbook in one file (generated)
skills/
	totaljs-cookbook/
		SKILL.md                  entry point: rules + task → reference table
		references/NN-topic.md    one topic per file (source of truth)
scripts/build.js                  generates COOKBOOK.md
.claude-plugin/                   Claude Code plugin + marketplace manifest
gemini-extension.json             Gemini CLI extension manifest
```

## If you are editing this repository

- The source of truth is `skills/<skill>/references/*.md`. Never edit `COOKBOOK.md` by hand; run `node scripts/build.js` after every change and commit the result.
- A reference file starts with `# N. Title` and is named `NN-topic.md`. Link other sections as `[section N](NN-topic.md)`; the build turns these into anchors.
- When a file is added, renamed or removed, update the table in `SKILL.md`, the list in `llms.txt` and the table in `README.md`.
- Keep `SKILL.md` under about 500 lines. Details belong in `references/`.
- Write in English. Code examples follow the cookbook's own style: tabs, semicolons, single quotes, `var`.
- State only what is true for Total.js 5. Check claims against the `total5` source when unsure.
