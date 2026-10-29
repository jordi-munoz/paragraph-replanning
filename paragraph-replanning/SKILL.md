---
name: paragraph-replanning
description: Reverse-outline an academic text: a synoptic view (verbatim headings, numbered one-line paragraph summaries, exhibits) and, on request, a structural diagnosis with prioritized actions, as Markdown and PDF.
---

# Paragraph re-planning: synoptic view and diagnosis

This skill implements stages 1 to 3 of Patrick Dunleavy's paragraph re-planning method (Writing For Research, 2014; Authoring a PhD, 2003, ch. 6), also known as reverse outlining. It resurfaces the paragraph-by-paragraph structure of a finished draft so the author can see what the text actually says and does, diagnoses that structure, and lists the changes worth making, most important first.

Two modes, each producing one Markdown file and a PDF of it:
- Synoptic mode (stages 1 and 2): `<source-stem>_synoptic.md` and `.pdf`. This is the default when the user asks for paragraph re-planning, a reverse outline or a synoptic view.
- Diagnosis mode (stage 3): `<source-stem>_diagnosis.md` and `.pdf`. Run it when the user asks for the diagnosis, the analysis of the synoptic view, or stage 3. If no synoptic view exists yet for the text, produce it first, deliver it, then produce the diagnosis.

If the user invokes the skill with arguments (for example a file name and the word "diagnosis"), use them to pick the source file and the mode: $ARGUMENTS

The skill never rewrites the text and never drafts a full alternative structure. Designing a Plan B structure (stages 4 and 5) is done only if the user asks for it.

## Inputs

Accept .tex (follow \input and \include in order), .docx, .pdf, .md, .txt, or pasted text. If a file cannot be read, say so plainly rather than working from partial content.

# Synoptic mode

## Step 0. Build an inventory before summarizing

Segment the text into an ordered list of units of three types: heading, paragraph, exhibit. When a file is available, do this with a script rather than by eye, so that the paragraph count is exact and nothing is skipped. Save the inventory as a working JSON file (unit type, order, paragraph number, heading path, full text, word count); diagnosis mode reuses it. It is not delivered.

LaTeX:
- Strip comments (an unescaped % to end of line) and the preamble. Start at \begin{document}.
- A paragraph is a block of text delimited by blank lines or \par.
- Headings: \part, \chapter, \section, \subsection, \subsubsection and their starred variants. Record the title exactly as written, removing only formatting commands. Reconstruct automatic numbering (1, 1.1, 1.1.1) for unstarred headings.
- Exhibits: figure, table, sidewaysfigure, sidewaystable and starred variants. Take the \caption text and reduce it to its first sentence if longer than one line. Number figures and tables separately, as LaTeX would. Place each exhibit at the position of its environment in the source, and record the \label so references to it can be traced.
- Displayed equations, itemize/enumerate lists and quotations belong to the surrounding paragraph when the text resumes after them without a blank line, with \noindent, or with a lowercase continuation. A list or quotation standing alone between full paragraphs is its own unit.
- Footnotes are not paragraphs. Exclude \maketitle material, acknowledgements and the bibliography. Treat the abstract as an unnumbered unit under the heading Abstract. Include appendices, continuing the numbering.

Word (.docx): paragraphs are non-empty body paragraphs; headings are paragraphs in Heading styles (or Title); exhibits are Caption-style paragraphs or lines beginning with Figure n / Table n / Figura n / Taula n / Tabla n. Use python-docx or pandoc.

PDF: extract text and rebuild paragraphs from layout, dropping running heads, page numbers and footnotes. Tell the user in one line that segmentation from PDF is less reliable than from the source file.

Markdown or plain text: blank lines delimit paragraphs; # lines are headings.

For very long texts (book chapters, theses), work section by section, keeping a single running paragraph counter.

## Step 1. Summarize each paragraph in one line

Copy every heading and sub-heading exactly as it occurs. For each paragraph write a one-line summary. For each figure or table write its label and title on one line.

Rules for summaries:
- One line, one sentence. Aim for 25 words or fewer; never exceed 30 words.
- Substantive: state the specific claim, argument, mechanism, finding or decision the paragraph makes, with its actual content (cases, variables, direction and size of effects, dates, data sources) when these are the point.
- Say what the paragraph says and what it does. When the function matters (states a hypothesis, rebuts an objection, justifies a design choice, reports a robustness check), convey it through the content rather than by naming the function alone.
- Never formal or vacuous. Do not write summaries of the form "This paragraph discusses X", "The author talks about X", "Some considerations on X", "Further literature on X", "Presents the results", "Concluding remarks".
- Exact narrative precis, like a good tweet: no evaluation, no interpretation beyond what the paragraph says, no content imported from neighbouring paragraphs.
- If a paragraph makes two unrelated points, report both, separated by a semicolon. This is diagnostic information for the author and must not be smoothed over.
- Write in the language of the source text (Catalan, Spanish, English or other).
- No em dashes. No bold.

Examples:
- Vacuous: "Discusses the literature on economic voting."
  Substantive: "Economic voting studies assume national retrospection, which misses regional economic grievance in multilevel states."
- Vacuous: "Presents the results of the main model."
  Substantive: "Model 1: perceived fiscal grievance raises support for independence by about 12 points, net of national identity."
- Vacuous: "Explains the data."
  Substantive: "Uses 14 CEO barometer waves (2011-2019) pooled into one dataset of 26,000 respondents."

## Step 2. Number every paragraph summary in a single sequence

Number paragraph summaries 1, 2, 3 ... N from the beginning to the end of the text, across all sections and appendices, never restarting. Headings keep their own numbering and are not part of the sequence. Exhibits keep their own labels (Figure 1, Table 2) and are not part of the sequence. The abstract is unnumbered.

## Synoptic output file

Name the file `<source-stem>_synoptic.md` (`synoptic.md` for pasted text) and write it next to the source file. Structure:

```
---
lang: ca
papersize: a4
geometry: margin=2.5cm
fontsize: 11pt
---

# Synoptic view: <title of the text>

Source: <file name>. <N> paragraphs, <W> words in numbered paragraphs, <F> figures, <T> tables. Generated <date>.

## Abstract

Abstract [180 words]: <one-line summary>

## 1 Introduction

1. [142 words] <summary>
2. [98 words] <summary>

### 1.1 Background

3. [215 words] <summary>

> Table 1. Vote intention by national identity, 2006-2024

4. [176 words] <summary>
```

Formatting rules:
- The YAML header is used by the PDF export (see below). Set `lang` to the language of the source text (ca, es, en, fr...).
- Map the top heading level of the text to ##, the next to ###, and so on.
- Write every paragraph number explicitly and put a blank line before and after each exhibit line, so that numbers render exactly as written.
- After each paragraph number, give the length of the source paragraph in square brackets, in the language of the text ("[142 words]", "[142 paraules]", "[142 palabras]"). Take the count from the inventory, computed by script on the paragraph text without LaTeX commands, citation keys or comments; never estimate it. Write the abstract line as "Abstract [180 words]: <summary>".
- Do not use square brackets anywhere else in a summary line, so the count stays unambiguous.
- Exhibit lines are blockquotes beginning with the label.
- The file contains only the synoptic view: no commentary, diagnosis or suggestions.

## Synoptic final check

- The number of numbered lines equals the number of paragraphs in the inventory, and the sequence has no gaps or repeats.
- Every heading appears verbatim and in order; every exhibit appears at its position.
- No summary exceeds 30 words and none begins with a vacuous formula.
- Every numbered line carries a word count, and the counts add up to the total in the Source line.
- Spot-check at least five summaries against their source paragraphs for accuracy.

# Diagnosis mode

The diagnosis analyses the synoptic view as Dunleavy's stage 3 describes: how long each section is, what the sequence of arguments is, where digressions occur, how bitty or recursive the argument is, where the same point is repeated, and where one core point is made in bits at several stages of the text. It then turns the findings into a short list of actions sorted by importance. It works from the inventory and the synoptic view, but every finding about content must be verified against the full paragraphs, not the summaries alone.

The paragraph numbers of the synoptic view are the reference system throughout. Cite them in every finding and action (for example "paras 7, 23, 41").

## Scope: structure only, not peer review

The diagnosis is about the architecture of the text: how the paragraphs unfold, how sections and sub-sections are ordered and sized, where points recur or are split, and whether headings and exhibits sit where they belong. It is not a substantive or methodological review, and it takes the content of every paragraph as given.

In scope:
- the order of sections, sub-sections and paragraphs, and whether that order follows the argument the text itself sets out;
- where a step the text announces or relies on is structurally absent or misplaced (for example, expectations that the results section refers back to but that no earlier section states);
- repetition, piecemeal points, digressions, bittiness, multi-point paragraphs, heading-content fit, section balance, and exhibit placement.

Out of scope, never reported as findings or actions:
- whether claims are true, accurate, well supported or consistent with the evidence, including whether a summary or conclusion states the results correctly;
- research design, data, measurement, methods, statistics, robustness or the interpretation of results;
- the coverage, choice or accuracy of the literature and citations;
- novelty, contribution, framing or overclaiming;
- language, style, grammar, spelling and typos.

When a structural finding touches content, describe only the structural problem (where a point sits, how often it recurs, what it is separated from), not whether the point is right. If something out of scope looks serious, do not put it in the diagnosis; mention it in at most one line in the reply to the user, outside the files.

## Step D1. Measure section lengths (by script)

From the inventory compute, for every heading at every level: paragraph range, number of paragraphs, number of words, share of total words, and exhibits contained. Flag a section as long when it has more than 1.5 times, and short when it has less than half, the median paragraph count of sections at the same level. Flag sections with one or two paragraphs, and sub-sections that are the only child of their parent.

## Step D2. Tag topics and claims

For each numbered paragraph assign a topic tag of two to four words and record its core claim in a few words, in the working JSON. Use consistent tags so that the same subject always receives the same tag. The tags drive the next step; they are not printed in the diagnosis unless the user asks for them.

## Step D3. Identify problems

Check the following categories. Each finding gets an ID made of the category letter and a number (A1, R2, P1).

- A. Sequence of the argument: a result before its method, a hypothesis after its evidence, a concept used before it is defined, sections in an order that does not follow the argument.
- H. Heading and content mismatches: headings whose paragraphs do something other than what the heading announces, or content that belongs under another heading.
- L. Section lengths: the long, short and one-child sections flagged in D1, when the imbalance matters for the argument.
- R. Repetitions: paragraphs that make the same point, noting whether the later occurrence adds anything (new evidence, a qualification) or merely restates.
- P. Points made piecemeal: one core point whose elements are scattered across non-adjacent paragraphs, listing the paragraphs in order and what each contributes.
- D. Digressions: paragraphs whose topic does not serve the job of their section.
- B. Bittiness and recursion: repeated topic switches over a short run of paragraphs, or returns to a topic already left (give the tag sequence, for example "turnout, identity, turnout, identity, paras 18-21").
- M. Multi-point paragraphs: paragraphs whose summary needed two unrelated points.
- E. Exhibits: figures or tables far from the paragraphs that discuss them, exhibits never referred to, references to exhibits that do not exist (in LaTeX, trace \ref and \label).

Rules against repetition in the diagnosis:
- Report each problem once, in the category where it fits best. Never restate it in another category; if the link matters, add the other ID in brackets ("see P1").
- Merge overlapping findings. Several digressions in one section are one finding. A point made piecemeal that also produces near-repetitions is one P finding, not a P finding plus R findings.
- A finding is at most three lines: paragraph numbers, the problem, the evidence. Do not paraphrase paragraphs again; the synoptic view already summarizes them, so cite numbers.
- Do not report deliberate signposting (an introduction previewing results, a conclusion recapitulating them) unless the recapitulation adds material that appears nowhere in the body. Whether a recapitulation states the body's content correctly is out of scope.
- Omit categories with no findings, and name them in a single closing line.
- Never invent findings to fill a category. A short diagnosis of a well-structured text is a correct diagnosis.

## Step D4. Draw up the action list

Turn the findings into concrete actions. Each action:
- is one edit operation on the structure, stated in the imperative: move, merge, cut, split, retitle, relocate an exhibit, shorten, or add a structurally missing step (a transition, a section the text refers back to); never an action to change what a paragraph claims, to add analyses or evidence, or to correct wording;
- names the paragraphs, sections or exhibits involved and where they should go ("Move paras 23-25 to section 2, after para 9");
- gives the reason in one clause and the finding IDs it resolves;
- says what to change, not how to write it: no drafted sentences.

Sort the actions by importance, using these criteria in order:
1. Effect on the argument: changes that affect whether a reader can follow the main claim and its support (sequence, missing or misplaced steps, the main point made piecemeal) come before section-level problems, which come before paragraph-level ones.
2. Reach: actions that fix several findings or affect many paragraphs come before those that touch one.
3. Effort: among actions of equal weight, the quickest come first.

Label each action High, Medium or Low priority. List at most 12 actions; if minor ones remain, mention them in a single closing line by finding ID. One action may resolve several findings, and every finding should be covered by an action or by that closing line.

If the findings show that the order of sections itself is the problem, so that targeted moves would not fix it, make the first action "Rebuild the structure (Plan B)" with the reason, and leave out the targeted actions that a new structure would supersede.

## Diagnosis output file

Name the file `<source-stem>_diagnosis.md`, with the same YAML header as the synoptic file, and write it next to the source file. Structure:

```
---
lang: ca
papersize: a4
geometry: margin=2.5cm
fontsize: 11pt
---

# Diagnosis: <title of the text>

Source: <file name>. Companion to <source-stem>_synoptic.md. <N> paragraphs, <k> findings. Generated <date>.

## Priority actions

1. [High] Merge paras 7, 23 and 41 into a single paragraph in section 2; the grievance mechanism is now split across three sections (P1, R2).
2. [High] Move paras 30-32 (data and measurement) before para 26, where the first results appear (A1).
3. [Medium] ...

Minor: M1, E2.

## Section lengths

| Section | Paras | Count | Words | % words | Exhibits | Flag |
|---|---|---|---|---|---|---|
| 1 Introduction | 1-6 | 6 | 820 | 9 | | |
| 2 Theory | 7-24 | 18 | 2,950 | 31 | Fig. 1 | long |

## Findings

### A. Sequence of the argument

A1. Paras 26-29 report Model 1 before paras 30-32 define the grievance index it uses.

### P. Points made piecemeal

P1. ...

No findings in: H, D.
```

Writing rules for the diagnosis:
- Write in the language of the source text, in plain formal prose. No em dashes. No bold.
- Be specific and candid. No praise, no reassurance, no hedging that empties a finding.
- No rewritten text and no full alternative structure; targeted actions are the limit.

## Diagnosis final check

- Section length figures are computed by script and add up to the totals.
- Every paragraph number cited exists and points to the paragraph described.
- Each repetition and piecemeal finding has been checked against the full paragraphs.
- No problem appears in more than one category, and no finding paraphrases what the synoptic view already says.
- Every finding and action is structural: none judges the accuracy, support or consistency of claims, the methods or results, the literature, or the language (see Scope).
- The actions are sorted by the criteria above, and every finding is covered by an action or the closing "Minor" line.

# Export to PDF

After writing each Markdown file, render it to a PDF with the same stem, next to it.

1. Check for Quarto with `quarto --version`. Positron ships with it. Render with:

   ```
   quarto render <file>.md --to pdf
   ```

   The YAML header sets language, A4 paper and margins. If Quarto is not available but pandoc and a LaTeX engine are, use `pandoc <file>.md -o <file>.pdf --pdf-engine=lualatex`.
2. If no TeX distribution is installed, do not install software without asking. Render to a self-contained HTML instead (`quarto render <file>.md --to html -M embed-resources:true`), and tell the user in one line that `quarto install tinytex` enables PDF output. If neither Quarto nor pandoc is available, deliver the Markdown only and say so in one line.
3. Remove auxiliary files the render leaves behind (for example a `<file>_files` folder) unless the output needs them.
4. Check the PDF: paragraph numbers must match the Markdown (lists after an exhibit must resume at the right number, not restart at 1) and the section-length table must fit the page width. If the table overflows, reduce the margin to 2 cm or set `fontsize: 10pt` and render again.

# After delivery

Deliver the Markdown and the PDF. In the reply, state the key counts in one line (paragraphs and exhibits for the synoptic view; findings and High-priority actions for the diagnosis). After the synoptic view, offer in one line to run the diagnosis. After the diagnosis, offer in one line to design a Plan B structure (stages 4 and 5: four or five new sections of roughly equal length, every numbered paragraph allocated, with a clear logic within each section). Do not start either unasked.
