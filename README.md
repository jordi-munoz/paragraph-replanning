# paragraph-replanning

A [Claude Code](https://claude.com/claude-code) skill that reverse-outlines an academic text and, on request, diagnoses its structure.

## Credit

The method is Patrick Dunleavy's paragraph re-planning technique. All credit for the approach goes to him; this skill only automates its first three stages. See:

- Patrick Dunleavy, "[What is paragraph re-planning?](https://medium.com/advice-and-help-in-authoring-a-phd-or-non-fiction/c9a7e694d4ec)", *Advice and help in authoring a PhD or non-fiction book* (Medium).
- Patrick Dunleavy (2003), *Authoring a PhD: How to Plan, Draft, Write and Finish a Doctoral Thesis or Dissertation*, Palgrave Macmillan, ch. 6.
- Patrick Dunleavy (2014), *Writing For Research* blog.

## What it does

Two modes, each writing a Markdown file and a PDF next to the source:

1. **Synoptic view** (stages 1 and 2). Copies every heading verbatim, summarizes each paragraph in one substantive line with its word count, numbers all paragraphs in a single sequence, and places figures and tables where they occur.
2. **Diagnosis** (stage 3). Measures section lengths, then identifies structural problems (argument sequence, heading-content fit, section balance, repetition, points made piecemeal, digressions, bittiness, multi-point paragraphs, exhibit placement) and turns them into a prioritized list of at most 12 structural actions, citing paragraph numbers.

The skill never rewrites the text, and it does not judge content, methods, literature or style. Designing a new structure (Dunleavy's stages 4 and 5, "Plan B") is done only if you ask.

Inputs: `.tex` (following `\input`/`\include`), `.docx`, `.pdf`, `.md`, `.txt`, or pasted text. Output follows the language of the source.

## Installation

Copy the `paragraph-replanning` folder into your skills directory:

- Personal: `~/.claude/skills/paragraph-replanning/`
- Project: `.claude/skills/paragraph-replanning/`

```bash
git clone https://github.com/jordi-munoz/paragraph-replanning.git
cp -r paragraph-replanning/paragraph-replanning ~/.claude/skills/
```

PDF export uses [Quarto](https://quarto.org) (or pandoc with a LaTeX engine). Without a TeX distribution the skill falls back to HTML.

## Usage

In Claude Code:

```
/paragraph-replanning paper/main.tex
/paragraph-replanning paper/main.tex diagnosis
```

or simply ask for "a reverse outline", "a synoptic view" or "the diagnosis" of a file.

## License

MIT. The method itself belongs to Patrick Dunleavy; please cite his work when you use it.
