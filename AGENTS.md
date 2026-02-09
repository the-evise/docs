# Repository Guidelines

## Scope & Purpose

This repository documents **React + TypeScript component design patterns** and directly connected architecture principles. It is concise, practical, and reference-grade - targeted at intermediate to advanced engineers building large-scale, maintainable systems.

* Focus: Component design, state/data flow, composition, event handling, scalability.
* Language: TypeScript-first, React idiomatic.
* Style: Formal, minimal, and evidence-based.

## Documentation Structure & Writing Framework

Each pattern document should follow this structure:

### Document Template (`#` = main heading)

* `# Pattern Name`

    * `## Problem Signals`
      Bullet-pointed engineering pain points the pattern solves.

    * `## Core Idea`
      One concise paragraph describing what the pattern is and what it enables.

    * `## Reference Details`

        * **Definition**
        * **When to Use**
        * **When to Avoid**
        * **Mechanics** (how it works, diagram or short explanation)
        * **TypeScript Example**
        * **Scalability Notes**
        * **Related Patterns**

    * `## Summary`
      Bullet-point recap of the pattern's value and constraints.

    * `## Next Steps`
      Point to related patterns or deeper integrations.

### Baseline Checklist (Uncontrolled Components as standard)

* Use the exact heading order above; do not rename sections.
* Problem Signals = 3-5 bullets, no narration.
* Core Idea = 2-3 short paragraphs, declarative tone.
* Reference Details = Definition, When to Use, When to Avoid, Mechanics, TypeScript Example, Scalability Notes, Related Patterns.
* Mechanics = numbered list with bold labels and 1-2 sentence explanations.
* TypeScript Example = minimal API surface, config-in/events-out, no UI styling.
* Scalability Notes = Strengths, Weaknesses, System-Level Considerations.
* Summary = 2-3 bullets; Next Steps = 1-2 bullets pointing to related patterns.

## Project Structure

```
react-patterns-docs/
|-- docs/
|   |-- 00-guides/
|   |   `-- glossary.md
|   |-- 03-patterns/
|   |   |-- controlled-component.md
|   |   |-- uncontrolled-component.md
|   |   |-- compound-components.md
|   |   |-- renderless-component.md
|   |   |-- slots-composition.md
|   |   |-- smart-dumb-components.md
|   |   |-- event-action-pattern.md
|   |   |-- co-located-state.md
|   |   `-- context-gateway.md
|   |-- principles/              # Architectural principles that support patterns
|   |-- architecture/            # Folder structures, layering, etc.
|   |-- references/              # External links, RFCs, etc.
|   `-- index.md                 # Home page (VitePress entry)
```

## Editing and Authoring Rules

* Follow kebab-case for filenames and directories.
* One pattern per file.
* Use TypeScript in all code examples.
* Avoid tutorials; documents must be reference-style.
* Every new term must be added to `00-guides/glossary.md`.

## Useful Commands

* `rg --files` - list all doc files.
* `rg "Controlled Component" 03-patterns` - search for pattern usage.

## Contribution & Review Process

* Use short, lowercase, sentence-style commit messages.
* PRs must list:

    * changed files
    * brief summary of pattern or section changes
    * glossary updates if applicable
* No automated tests; sanity check TS examples and consistency.

## Patterns Roadmap (Initial Set)

To be implemented first:

1. Controlled Component
2. Uncontrolled Component
3. Compound Components
4. Renderless Component
5. Slots / Composition
6. Smart vs Dumb Components
7. Event - Action Pattern
8. Co-located State Pattern
9. Context Gateway Pattern

More to follow from OpenAI Codex analysis.

---

All documents must align with the VitePress build system and be scalable for long-term use as a personal and public engineering reference.
