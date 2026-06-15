---
name: skill-creator
description: >
  This skill should be used when the user wants to create a new skill or update an existing skill
  that extends Claude's capabilities with specialized knowledge, workflows, or tool integrations.
---

# Skill Creator

This skill provides guidance for creating effective skills.

## About Skills

Skills are modular, self-contained packages that extend Claude's capabilities by providing
specialized knowledge, workflows, and tools. Think of them as "onboarding guides" for specific
domains or tasks—they transform Claude from a general-purpose agent into a specialized agent
equipped with procedural knowledge that no model can fully possess.

### What Skills Provide

1. Specialized workflows - Multi-step procedures for specific domains
2. Tool integrations - Instructions for working with specific file formats or APIs
3. Domain expertise - Company-specific knowledge, schemas, business logic
4. Bundled resources - Scripts, references, and assets for complex and repetitive tasks

### Anatomy of a Skill

Every skill consists of a required SKILL.md file and optional bundled resources:

```
skill-name/
├── SKILL.md (required)
│   ├── YAML frontmatter metadata (required)
│   │   ├── name: (required)
│   │   └── description: (required)
│   └── Markdown instructions (required)
└── Bundled Resources (optional)
    ├── scripts/          - Executable code (Python/Bash/etc.)
    ├── references/       - Documentation intended to be loaded into context as needed
    └── assets/           - Files used in output (templates, icons, fonts, etc.)
```

#### SKILL.md (required)

**Metadata Quality:** The `name` and `description` in YAML frontmatter determine when Claude will use the skill. Be specific about what the skill does and when to use it. Use third-person (e.g., "This skill should be used when..." instead of "Use this skill when...").

#### Bundled Resources (optional)

##### Scripts (`scripts/`)

Executable code (Python/Bash/etc.) for tasks that require deterministic reliability or are repeatedly rewritten.

- **When to include**: When the same code is being rewritten repeatedly or deterministic reliability is needed
- **Benefits**: Token efficient, deterministic, may be executed without loading into context

##### References (`references/`)

Documentation and reference material intended to be loaded as needed into context.

- **When to include**: For documentation that Claude should reference while working
- **Best practice**: If files are large (>10k words), include grep search patterns in SKILL.md
- **Avoid duplication**: Information should live in either SKILL.md or references files, not both

##### Assets (`assets/`)

Files not intended to be loaded into context, but rather used within the output Claude produces.

- **When to include**: When the skill needs files used in the final output (templates, images, boilerplate)

### Progressive Disclosure Design Principle

Skills use a three-level loading system to manage context efficiently:

1. **Metadata (name + description)** - Always in context (~100 words)
2. **SKILL.md body** - When skill triggers (<5k words)
3. **Bundled resources** - As needed by Claude

## Skill Creation Process

Follow these steps in order, skipping only when clearly not applicable.

### Step 1: Understand the Skill with Concrete Examples

To create an effective skill, clearly understand concrete examples of how the skill will be used.

Questions to ask:
- "What functionality should this skill support?"
- "Can you give some examples of how this skill would be used?"
- "What would a user say that should trigger this skill?"

Ask one at a time. Conclude when there is a clear sense of the functionality the skill should support.

### Step 2: Plan the Reusable Skill Contents

Analyze each example to identify what scripts, references, and assets would be helpful when executing these workflows repeatedly.

- Identify code that gets rewritten each time → candidate for `scripts/`
- Identify reference material looked up repeatedly → candidate for `references/`
- Identify boilerplate/templates used in output → candidate for `assets/`

### Step 3: Initialize the Skill

Use the bundled `init_skill.py` script to scaffold the skill directory:

```bash
python ~/.claude/skills/skill-creator/scripts/init_skill.py <skill-name> --path ~/.claude/skills
```

This creates the skill directory with SKILL.md template and example resource directories. Delete any unneeded example files after initialization.

### Step 4: Edit the Skill

The skill is being created for another instance of Claude to use. Focus on including information that would be beneficial and non-obvious.

**Writing style:** Use **imperative/infinitive form** (verb-first), not second person. Use objective, instructional language ("To accomplish X, do Y" rather than "You should do X").

**SKILL.md description:** Write in third-person — "This skill should be used when..." not "Use this skill when..."

To complete SKILL.md, answer:
1. What is the purpose of the skill, in a few sentences?
2. When should the skill be used?
3. How should Claude use the skill in practice? Reference all bundled resources so Claude knows how to use them.

### Step 5: Iterate

After testing the skill, notice struggles or inefficiencies and update SKILL.md or bundled resources accordingly.
