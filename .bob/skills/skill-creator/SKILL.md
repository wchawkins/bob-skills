---
name: skill-creator
description: Create and improve IBM Bob skills through conversation. Use this whenever the user wants to build a new skill, turn a workflow into a skill, write or fix a SKILL.md, or improve a skill's description so Bob triggers it reliably. Trigger on phrases like "make a skill", "turn this into a skill", "write a SKILL.md", or "Bob isn't using my skill", even if the user doesn't say the word "skill-creator".
---

# Skill Creator (for IBM Bob)

Help the user build a Bob skill by talking in chat. The end product is a `SKILL.md`
(plus any supporting files) that the user saves into a skill folder. You do the
interviewing and writing; the user does the saving.

Keep Bob's constraints in mind — Bob skills are primitive:
- A skill is a folder under `.bob/skills/` (project) or `~/.bob/skills/` (global).
- The folder contains a `SKILL.md`: YAML front matter (`name`, `description`) then instructions.
- Supporting files (checklists, templates, references) can live in the same folder; Bob reads them once the skill activates.
- The `description` is the ONLY thing Bob uses to decide whether to activate the skill. No description = the skill is ignored.
- Skills require Advanced mode.

## Workflow

### 1. Capture intent

Figure out three things, pulling from the conversation first if the user is asking to
turn an existing workflow into a skill:

1. **What should this skill make Bob do?** (the core task/workflow)
2. **When should it trigger?** (what the user would be saying or working on)
3. **What's the expected output?** (a file, a review, a formatted message, code, etc.)

Ask about edge cases, formats, and any reference material the skill should bundle.
Don't over-interview a simple skill — get what you need and move on.

### 2. Write the SKILL.md

**name**: kebab-case, descriptive (`security-review`, not `review`). This is also the folder name.

**description**: The trigger. Pack in BOTH what the skill does AND when to use it.
Bob tends to under-trigger, so make the description a little pushy — list concrete
phrases and contexts. Compare:
- Weak: `Code review skill`
- Strong: `Review code for bugs, security issues, and best practices. Use whenever the user shares code and asks for review, mentions a PR, or asks "is this safe?" — even if they don't say "review".`

**instructions** (everything below the `---`): Write imperative, actionable steps.
Explain *why* a step matters rather than stacking MUSTs. Point to any supporting files
by name with a note on when to read them.

Use this shape:

```markdown
---
name: skill-name
description: What it does + when to trigger (pushy, with example phrases).
---

# Skill Name

One line on what this skill is for.

## Steps

1. First action.
2. Second action.
3. ...

Follow the template in `template.md` / checklist in `checklist.md` when relevant.
```

### 3. Decide on supporting files

If the workflow needs a template, checklist, style guide, or long reference, put it in
a separate file in the skill folder and reference it from `SKILL.md`. Keep `SKILL.md`
focused on the workflow; move bulky detail out. Skip this for simple skills.

### 4. Deliver it

Show the user the final `SKILL.md` (and any supporting files), then tell them exactly
where to put it:

```
.bob/skills/<skill-name>/SKILL.md        # project-specific
~/.bob/skills/<skill-name>/SKILL.md      # global / reusable everywhere
```

Remind them: project-level skills win over global ones with the same name, and Bob asks
permission before activating a skill unless they turn on Auto-Approve for Skills in
settings.

### 5. Iterate

If Bob isn't triggering the skill, the fix is almost always the **description** — make it
more specific and add the phrases the user actually types. If Bob does the wrong thing
once triggered, tighten the **instructions**. Adjust, hand back the updated file, repeat.

## Writing principles

- **Single responsibility.** One skill, one job. Split unrelated workflows into separate skills.
- **Imperative voice.** "Check for X", not "You might want to check for X".
- **No surprises.** A skill's behavior should match what its description implies. Don't build skills for unauthorized access, data exfiltration, or anything malicious.
- **Start simple, refine.** Draft it, reread with fresh eyes, tighten.
