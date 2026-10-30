# Template: role brain (a skill that carries judgement, not knowledge)

> A **role brain** is a skill for a recurring kind of work — design, marketing, code, finance — that holds the *house judgement* for that work: what the standards are, where the room's taste lives, which procedures to reach for, and what has already gone wrong.
>
> It is not a tutorial. Whatever a fresh model already knows does not belong here. It is not a wiki either: if it can be looked up on demand, it is a link, not a file.

## Shape

```
<role>-brain/
  SKILL.md          frontmatter + five sections (below) — always short, < 60 lines
  standards.md      the house minimums, each with its origin and its gate
  skill-map.md      which installed procedure for which step, with «when NOT to load»
  checklists.md     one list per deliverable type
  lessons.md        real failures, dated, with the action that followed
  sources.md        filtered material: repos, authors, courses — with a verdict
  tools.md          what can be run, where it is, what it is for  (optional but recommended)
```

Seven files is a ceiling, not a target. A brain that grows past it is usually absorbing material that belongs in `notes/`.

## SKILL.md: the five sections

1. **What this is** — one paragraph. The brain, not the discipline.
2. **How to work this domain with the owner** — the handful of rules that change behaviour: language, directness, honesty over enthusiasm, what to verify.
3. **Flow** — numbered steps with **gates** at the ends. This is the section that turns a pile of advice into a procedure.
4. **Files of this brain** — one line per file, saying what it holds. This is the index the model reads when it needs more.
5. **Origin** — when it was created, from what, with evidence. A brain without provenance cannot be trusted or pruned.

The frontmatter `description` must say **when to use it**, not what it contains — that sentence is all the model sees before deciding to load the rest.

## Rules that keep it honest

- **A brain grows from real work, not from repositories.** An idea read in someone else's project is an idea until it has been used and it has cost or taught something.
- **Every standard carries its origin.** «We always do X» is worthless; «X because on <date> Y failed» is a rule.
- **Proposed standards are marked as proposed.** Inventing house judgement is the fastest way to make a brain decorative.
- **Third-party material lives outside the brain.** Course notes, papers and blog posts go to `notes/` and are cited in `sources.md`. A brain holding a 40 KB article is a filing cabinet wearing a lab coat.
- **The map lists procedures that exist.** Before naming a skill in `skill-map.md`, confirm the folder is installed; renamed and removed skills are the most common rot in this file.
- **One skill per step, not three.** Loading every «how to write» procedure at once produces three voices and no standard.

## Verifier

The brain is a structure, so check the structure deterministically — a script that reads the folder and fails if:

```
☐ SKILL.md exists and its description mentions when to use it (> 120 chars)
☐ the five sections are present (what · how · flow · files · origin)
☐ standards.md, skill-map.md, checklists.md, lessons.md, sources.md exist
☐ no file inside is a third-party article (size > 40 KB is the smell)
```

Run it from the same weekly check that audits installed procedures. A brain that silently loses its flow section still looks like a brain — which is exactly why the check must not depend on someone noticing.

## Anti-patterns seen in practice

| Smell | What it means |
|---|---|
| Standards live as bullets inside SKILL.md | The file will pass 60 lines and stop being read; move them out and link |
| No flow section | The brain is a glossary, not a way of working |
| No skill map | The model loads whatever it remembers instead of the procedure for this step |
| Lessons list without dates or actions | Anecdotes, not a machine change |
| A `material/` folder with someone else's PDFs | The brain is now a shelving unit; move it to notes and cite it |
| Three brains with three different shapes | The owner has to relearn each one; the shape itself is the standard |
