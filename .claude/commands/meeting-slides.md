# Meeting Slides Generator

Generate landscape LaTeX PDF slides from your project code and progress, for group meeting presentations.

## Prerequisites

- Claude Code CLI
- TeX Live with `xelatex` + `ctex` + common packages (`booktabs`, `fontspec`):
  - **Ubuntu/Debian/WSL**: `sudo apt install texlive-lang-chinese texlive-latex-extra texlive-fonts-recommended`
  - **macOS (Homebrew)**: `brew install --cask mactex`
  - **Windows**: use WSL (recommended) with the Ubuntu command above, or install [MiKTeX](https://miktex.org/) + the `ctex` package
- A project directory with source code that Claude Code can read

## Input

Ask the user for the following (if not provided):
- Reporting period (e.g., "5.31-6.13")
- Key work items and modules to present
- Topics already covered in previous meetings (to avoid repetition)
- Number of pages (default: 3-4)

## Output Directory

All files go under a date-stamped directory:
```
reports/meeting_YYYY_M_D/
├── meeting_YYYY_M_D.tex      — LaTeX source
├── meeting_YYYY_M_D.pdf      — compiled output
└── meeting_YYYY_M_D_script.md — speaker notes
```

## Workflow

### Step 1: Information Gathering

1. Read the user's progress table / screenshot, extract key work items
2. Browse relevant source code to confirm technical details (state machines, parameters, flows, etc.)
3. Confirm what to present vs. what was already covered

### Step 2: Page Planning

Analyze the actual work content and propose page allocation. **List the outline for each page and get user confirmation before writing LaTeX.**

Planning principles:
- One topic per page, avoid information overload
- System-level content → TikZ architecture diagrams
- Module details → flowcharts + parameter tables
- Technical postmortems → comparison diagrams (problem vs. solution)
- Don't repeat content from previous presentations

### Step 3: Write LaTeX

Use the following template skeleton:

```latex
\documentclass[landscape,11pt]{article}
\usepackage[UTF8]{ctex}
\usepackage{geometry}
\geometry{margin=0.7cm, footskip=0.6cm}
\usepackage{tikz}
\usetikzlibrary{arrows.meta, positioning, fit, backgrounds, calc, shadows, shapes.geometric}
\usepackage{xcolor}
\usepackage{booktabs}
\usepackage{amsmath}
\usepackage{fontspec}

% --- Page footer: thin separator + project/date left, page x/y right ---
% Replace <PROJECT NAME> and <YYYY.MM.DD> below with the actual project name and report date
\usepackage{fancyhdr}
\usepackage{lastpage}
\pagestyle{fancy}
\fancyhf{}
\fancyfoot[L]{\footnotesize\textcolor{black!50}{<PROJECT NAME> \,$\cdot$\, <YYYY.MM.DD>}}
\fancyfoot[R]{\footnotesize\textcolor{black!50}{\thepage\,/\,\pageref{LastPage}}}
\renewcommand{\headrulewidth}{0pt}
\renewcommand{\footrulewidth}{0.4pt}

% --- Color scheme (customize for your project) ---
\definecolor{mainblue}{RGB}{34,80,140}
\definecolor{wingreen}{RGB}{50,140,50}
\definecolor{secondaryblue}{RGB}{34,101,156}
\definecolor{accent}{RGB}{200,40,40}
\definecolor{accentorange}{RGB}{210,120,20}
\definecolor{lightbg}{RGB}{245,248,252}

% --- TikZ component styles ---
\tikzset{
  comp/.style={draw=black!70, thick, rounded corners=4pt, fill=white,
               minimum width=2.0cm, minimum height=0.7cm, align=center,
               font=\footnotesize\bfseries,
               drop shadow={shadow xshift=0.6pt, shadow yshift=-0.6pt, opacity=0.08}},
  sbox/.style={draw=black!40, thick, rounded corners=6pt, inner sep=8pt},
  arrow/.style={->, >=Stealth, thick, black!60},
  tag/.style={font=\scriptsize\bfseries, rounded corners=2pt, inner sep=2pt, text=white},
}

\begin{document}
% (header/footer handled globally by the fancyhdr pagestyle declared above)

% Separate pages with \newpage

\end{document}
```

**Writing principles**:
- Use `comp` style for TikZ nodes (shadow + rounded corners), `sbox` for grouping boxes
- **Use embedded color title bars for sbox, not `label=above`** — `label=above` floats outside the frame and overlaps with subsequent elements at the same y. Use `\node[fill=color, text=white, rounded corners=2pt, inner sep=3pt] at (cx, top_y) {Title};` embedded inside the sbox top
- Place arrow labels with `above`/`below` — avoid `left`/`right` near page edges (text gets clipped)
- **Budget gap width for arrow labels** — `\draw (a) -- node[above] {label} (b);` label width often exceeds the a→b gap and overflows into box b. Plan center distance ≥ label width + box width + 0.5cm margin; when overflow happens, move the source box outward rather than shrinking the label
- **Constrain multi-line description text with `text width`** — `\node[align=left] {long text};` without constraint extends infinitely right and crosses other frames. Must add `text width=Xcm` with `anchor=north west`
- **Use Manhattan routing for all arrows, never slanted** — for nodes at different (x,y), use `(a) -- ++(dx,0) |- (b)` for L-shaped paths
- **Loop/back arrows exit from box side, not bottom** — for RETREAT→SETTLE style back-arrows, exit from source box `west`/`east`, use `-|` to enter target `.south`/`.north` with one turn. Avoids overlap with internal arrows when routing around the bottom
- **When widening one gap, sync-move the other side's frame** — in horizontal multi-box layouts, widening A→B gap without moving C frame causes B→C collision. Both sides need to move together
- Keep each page within one screen, no overflow
- Footer (project name · date on the left, page X/Y on the right) is rendered globally by `fancyhdr` from the template — don't add per-page footers manually
- If descriptive text doesn't fit in boxes, extract it to a `\node` in the page's blank area

### Step 4: Compile & Iterate

Compile command:
```bash
cd reports/meeting_YYYY_M_D && xelatex -interaction=nonstopmode meeting_YYYY_M_D.tex
```

**Common issues checklist** (check after each compile):
- [ ] Overfull hbox → reduce minipage width or TikZ node width
- [ ] Node overlap → adjust coordinates or use `positioning` library's `right=of` syntax
- [ ] Text clipped → use `above`/`below` labels instead of `left`/`right`, avoid page edges
- [ ] Box doesn't cover content → increase `sbox` `minimum width/height`
- [ ] Font too small → base 11pt for `landscape`, `comp` uses `\footnotesize`, body uses `\normalsize`
- [ ] Title bar overlaps with nodes at same y → use embedded color title bar instead of `label=above`
- [ ] Arrow label overflows into destination box → widen source/target box center distance (label width + 0.5cm margin)
- [ ] Bottom multi-line text crosses other frames → add `text width=Xcm`
- [ ] Slanted arrows → use Manhattan routing `-- ++(dx,0) |-` or `|-` / `-|` with waypoints
- [ ] Loop/back arrow overlaps internal arrows → exit from source `west`/`east`, enter target `.south`/`.north`
- [ ] Widening one gap causes collision on the other side → sync-move the whole frame, consider both sides together

Iterate based on user feedback until satisfied.

### Step 5: Generate Speaker Notes

After the PDF is finalized, generate oral speaker notes for each page as `meeting_YYYY_M_D_script.md`.

Notes style:
- Conversational, easy to read aloud
- 1-2 minutes per page (200-400 words)
- Organized by spatial order of visual elements (left→right, top→bottom)
- Include specific technical parameters and key conclusions

## Customization

Users can customize this skill for their project by:

1. **Adding reference files**: Add paths to existing LaTeX reports or style guides at the bottom of this file
2. **Changing colors**: Modify the `\definecolor` section in the template
3. **Adjusting output directory**: Change the `reports/` path convention
4. **Adding project-specific knowledge**: Append domain-specific terms or conventions
