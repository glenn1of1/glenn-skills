# glenn-skills

A Claude Code marketplace with one plugin, also called `glenn-skills`, that holds my skills. Each skill is a folder under `plugins/glenn-skills/skills/`.

## Skills

| Skill | What it does |
|---|---|
| `startup-mvp` | Multi-agent MVP scoping and implementation planning (STRATEGY -> RESEARCH -> PLANNING -> REVIEW) with GO/NO-GO gates. It plans; it does not write product code. |

## Install

```bash
claude plugin marketplace add glenn1of1/glenn-skills
claude plugin install glenn-skills@glenn-skills
```

Or from a local clone:

```bash
claude plugin marketplace add ./glenn-skills
claude plugin install glenn-skills@glenn-skills
```

Try it for one session without installing:

```bash
claude --plugin-dir ./glenn-skills/plugins/glenn-skills
```

## Use

Start Claude Code in the project you want to plan, then run:

```text
/glenn-skills:startup-mvp <describe your MVP idea>
```

The skill creates `MANIFEST.md` in the current directory and fills it in phase by phase. It does not edit your project's code.

`startup-mvp` must be invoked explicitly (`disable-model-invocation: true`) because a full run launches several agents, two of them on Opus. Remove that line in its `SKILL.md` to let Claude trigger it on its own.

Per-invocation effort levels need Claude Code v2.1.292 or later.

## Layout

```text
.claude-plugin/marketplace.json
plugins/glenn-skills/
  .claude-plugin/plugin.json
  skills/
    startup-mvp/
      SKILL.md
      agents/          # agent prompts, read by the skill via ${CLAUDE_SKILL_DIR}
```

Agent prompts sit inside each skill's folder on purpose: files in a plugin's top-level `agents/` folder are auto-registered as subagents, and these are prompts that the skill reads and passes on instead.

## Adding a skill

Create `plugins/glenn-skills/skills/<skill-name>/SKILL.md`, then run `claude plugin validate .` from the repo root. Bump `version` in `plugin.json` so installed copies update.
