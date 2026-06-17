# IBM Bob Skills Repository

This repository contains custom skills for IBM Bob, an AI coding assistant. Skills extend Bob's capabilities by providing specialized workflows and domain expertise.

## Available Skills

### 🏗️ fusion-hci-pricing
Size and price IBM Fusion HCI clusters for Watsonx and LLM inference workloads. Automatically calculates GPU requirements based on model parameters and VRAM needs, then matches to the appropriate hardware configuration with pricing.

**Use when:** Sizing infrastructure for AI/ML workloads, deploying foundation models, or getting Fusion HCI quotes.

### 🔍 code-review
Review code for bugs, security vulnerabilities, and best practices. Provides structured findings with severity levels.

**Use when:** Reviewing pull requests, auditing code quality, or checking for security issues.

### 🛠️ skill-creator
Create and improve Bob skills through conversation. Guides you through building a `SKILL.md` file with proper structure and trigger descriptions.

**Use when:** Building new skills, converting workflows into skills, or fixing skill activation issues.

## Installation

### Clone the Repository

```bash
git clone <repository-url>
cd <repository-name>
```

### Install Skills

Skills can be installed at the project level or globally:

**Project-level installation** (recommended for team sharing):
```bash
# Skills are already in .bob/skills/ after cloning
# Bob will automatically detect them when you open this project
```

**Global installation** (available across all projects):
```bash
# Copy individual skills to your global skills directory
cp -r .bob/skills/fusion-hci-pricing ~/.bob/skills/
cp -r .bob/skills/code-review ~/.bob/skills/
cp -r .bob/skills/skill-creator ~/.bob/skills/
```

## Using Skills

### Activating a Skill

Skills activate automatically when Bob detects relevant context in your message. You can also explicitly activate a skill:

```
Use the fusion-hci-pricing skill to size a cluster for Llama 3.3 70B
```

Or in Advanced mode, Bob may prompt you to activate a skill when appropriate.

### Skill Requirements

- **Mode:** Skills require Bob's **Advanced mode** (🛠️ Advanced)
- **Activation:** Skills activate based on their `description` field - the more specific your request, the better

### Example Usage

**Fusion HCI Pricing:**
```
I need to run Llama 3.3 70B and Granite 8B in production. What Fusion HCI config do I need?
```

**Code Review:**
```
Review this code for security issues and best practices
```

**Skill Creator:**
```
Help me create a skill that scans for exposed API keys
```

## Skill Structure

Each skill is a folder under `.bob/skills/` containing:

```
skill-name/
├── SKILL.md              # Required: skill definition and instructions
├── supporting-file.md    # Optional: reference data, templates, etc.
└── ...                   # Additional supporting files as needed
```

### SKILL.md Format

```markdown
---
name: skill-name
description: What the skill does and when to trigger it. Be specific and include common phrases users might say.
---

# Skill Instructions

Detailed instructions for Bob on how to execute this skill...
```

**Key points:**
- `name`: kebab-case identifier (also the folder name)
- `description`: The ONLY thing Bob uses to decide whether to activate the skill. Make it detailed and include trigger phrases.
- Instructions: Written for Bob, not the user. Be imperative and actionable.

## Creating New Skills

Use the `skill-creator` skill to build new skills interactively:

```
Help me create a skill for [your use case]
```

Or manually:

1. Create a folder under `.bob/skills/` with a kebab-case name
2. Add a `SKILL.md` file with YAML front matter and instructions
3. Add any supporting files (templates, reference data, etc.)
4. Test by asking Bob to use the skill

## Skill Development Tips

### Writing Good Descriptions

The `description` field is critical - it's how Bob decides to activate your skill. Make it:

- **Specific:** Include concrete phrases users might say
- **Comprehensive:** Cover multiple ways to express the same intent  
- **Action-oriented:** Describe both what the skill does AND when to use it

**Example:**
```yaml
description: Review code for bugs, security issues, and best practices. Use whenever the user shares code and asks for review, mentions a PR, or asks "is this safe?" — even if they don't say "review".
```

### Supporting Files

Skills can reference supporting files in their folder:

```markdown
When sizing clusters, read `model-catalog.md` for parameter counts and `tshirt-calculator.md` for pricing.
```

Bob will read these files when the skill activates.

### Testing Skills

1. Open a project with the skill installed
2. Switch to Advanced mode (🛠️ Advanced)
3. Use phrases from the skill's description
4. Check if Bob activates the skill automatically
5. Refine the description if activation is unreliable

## Contributing

To add a new skill to this repository:

1. Create the skill folder under `.bob/skills/`
2. Write a clear `SKILL.md` with comprehensive description
3. Add supporting files if needed
4. Update this README with the skill description
5. Test thoroughly before committing

## Troubleshooting

**Skill not activating:**
- Check that you're in Advanced mode
- Make the `description` more specific with trigger phrases
- Try explicitly mentioning the skill name in your request

**Skill not found:**
- Verify the folder is under `.bob/skills/` (project) or `~/.bob/skills/` (global)
- Ensure `SKILL.md` exists with valid YAML front matter
- Restart Bob/VS Code after adding new skills

**Skill activates incorrectly:**
- Narrow the `description` to be more specific
- Remove ambiguous trigger phrases
- Consider if the skill overlaps with another skill's description

## License

[Add your license information here]