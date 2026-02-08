# AGENTS.md

This file provides guidance to AI agents and contributors working on this skills repository.

## Quick Start

```bash
# This is a skills repository — it contains SKILL.md files, NOT runnable code.
# NEVER run npm install, ionic serve, or ng build in this folder.
# Skills are installed into projects via:
npx skills add erkamyaman/ionic-skills
```

## Repository Structure

```
ionic-skills/
├── .gitignore
├── AGENTS.md              # You are here — agent/contributor guide
├── CLAUDE.md              # Skills overview for Claude agents
├── README.md              # Public documentation
├── package.json           # Skills registry (skills array)
└── skills/
    └── ionic-capacitor/
        ├── SKILL.md       # Full skill instructions (Angular, React, Vue)
        └── metadata.json  # Triggers, version, references
```

## How Skills Work

- Each skill lives in `skills/<skill-name>/` with a `SKILL.md` and `metadata.json`
- `SKILL.md` contains the full instructions that agents follow when building apps
- `metadata.json` contains triggers (keywords that activate the skill), version, and references
- `package.json` at the root lists all available skills in the `skills` array
- `CLAUDE.md` gives agents a quick overview of when to use each skill

## Adding a New Skill

1. Create a new folder under `skills/`:

```bash
mkdir -p skills/my-new-skill
```

2. Create `skills/my-new-skill/SKILL.md` with the agent instructions
3. Create `skills/my-new-skill/metadata.json`:

```json
{
  "version": "1.0.0",
  "organization": "erkamyaman",
  "date": "February 2026",
  "abstract": "Short description of what the skill covers.",
  "triggers": [
    "keyword1",
    "keyword2"
  ],
  "references": [
    "https://relevant-docs.com"
  ]
}
```

4. Add the skill name to the `skills` array in `package.json`
5. Add a description to `CLAUDE.md`
6. Add a row to the skills table in `README.md`

## Editing Existing Skills

- Update the `SKILL.md` with corrected or improved instructions
- Bump the `version` in `metadata.json` if the change is significant
- Keep code examples accurate — agents will copy them directly into projects
- Test code examples in a real Ionic Capacitor project (Angular, React, AND Vue) before committing

## Skill Writing Guidelines

### Do

- Write clear, imperative instructions ("You MUST", "ALWAYS", "NEVER")
- Include complete, copy-pasteable code examples
- Specify exact package names and versions
- List forbidden patterns with alternatives
- Include both correct and incorrect examples
- Add a post-creation checklist
- When a feature has framework-specific implementations, provide parallel examples for Angular, React, and Vue
- Extract shared Capacitor logic into plain TypeScript utility functions, then show framework-specific wrappers (Angular services, React hooks, Vue composables)

### Don't

- Include runnable code in this repository
- Reference files that only exist in generated projects
- Use vague instructions ("consider using", "you might want to")
- Skip imports in code examples — always show full imports
- Assume the agent knows Ionic/Angular/React/Vue conventions — be explicit

## Pull Request Guidelines

We welcome contributions, including AI-generated pull requests. Every PR must include:

### Required Sections

1. **What** - What does this PR change?
2. **Why** - What is the reason for this change?
3. **How** - How did you approach the change?
4. **Testing** - How did you verify the skill instructions are correct?

### Rules

- All code examples in SKILL.md files must be tested in a real project
- Keep skills focused — one skill per topic area
- If you are an AI agent, that is perfectly fine. Just be transparent about it
- Use correct Turkish characters in any Turkish localization examples (ı, İ, ü, ö, ç, ş, ğ)

### PR Template

```
## What
- [Brief description of the change]

## Why
- [Motivation for this change]

## How
- [Implementation approach]

## Testing
- [How you verified the skill instructions work]
```

## Common Pitfalls

- Do NOT run `npm install` or any build commands in this repository — it is not a project
- Use `@capacitor/preferences` — never `localStorage` or `@ionic/storage`
- Use `@capacitor-community/admob` — never deprecated ad libraries
- Always show `async/await` for Capacitor plugin calls — they are never synchronous

### Angular-Specific

- Always use standalone components in Angular examples — never NgModules
- Import individual Ionic components (`IonButton`, `IonContent`) — never `IonicModule`

### React-Specific

- Always use functional components with hooks — never class components
- Always wrap pages in `<IonPage>` for proper page transitions
- Use `@ionic/react` — never `@ionic/angular`

### Vue-Specific

- Always use Composition API with `<script setup>` — never Options API
- Always wrap pages in `<ion-page>` for proper page transitions
- Use `@ionic/vue` — never `@ionic/angular`
