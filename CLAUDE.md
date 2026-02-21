# CLAUDE.md — AltaCV Codebase Guide

This file provides guidance for AI assistants working on the AltaCV repository.

## Project Overview

AltaCV is a **LaTeX document class** (`altacv.cls`) for creating modern, visually styled CVs and résumés. It was created by LianTze Lim (liantze@gmail.com) and released under the LaTeX Project Public License (LPPL) v1.3.

The project provides:
- A reusable LaTeX class file (`altacv.cls`)
- A bare-bones user template (`sample.tex`)
- A complete worked example based on Marissa Mayer's résumé (`mmayer.tex`)
- Pre-compiled PDF outputs and PNG previews

## Repository Structure

```
AltaCV/
├── altacv.cls              # Core LaTeX document class (all formatting logic)
├── sample.tex              # Bare-bones CV template for users to customize
├── sample-p1sidebar.tex    # Page 1 sidebar content for sample.tex
├── sample.bib              # BibTeX bibliography entries (sample publications)
├── mmayer.tex              # Complete example CV (Marissa Mayer)
├── mmayer-p1sidebar.tex    # Page 1 sidebar content for mmayer.tex
├── page2sidebar.tex        # Page 2 sidebar content (referees) used by both examples
├── README.md               # Project documentation
├── LICENSE.md              # LPPL v1.3 license
├── nicethings_icons_readme.txt  # Icon set attribution
├── sample.pdf / sample.png      # Compiled output of sample.tex
├── mmayer.pdf / mmayer.png      # Compiled output of mmayer.tex
├── Globe_High.png               # Default profile photo placeholder
├── mmayer.jpg                   # High-res photo used in mmayer example
└── mmayer-wikipedia-cc-by-2_0.jpg  # Marissa Mayer profile photo (CC-BY 2.0)
```

## Compilation

**No Makefile or build scripts are present.** Compile directly with LaTeX:

```bash
# Recommended (best results)
lualatex sample.tex

# For bibliography support, run the full sequence:
lualatex sample.tex
biber sample
lualatex sample.tex
lualatex sample.tex
```

**Compiler requirements:**
- **LuaLaTeX is strongly recommended** — handles `fontawesome` and `academicons` fonts natively
- XeLaTeX is supported but requires `academicons.ttf` to be installed at the OS level (not just in the TEXMF tree)
- pdfLaTeX is **not supported** (requires Unicode font handling)

**Package dependencies** (included in TeX Live 2016+ and MikTeX 2.9+):
- `fontawesome` — icon symbols (email, phone, GitHub, etc.)
- `academicons` — academic icons (ORCID, Google Scholar, etc.)
- `xcolor` — color definitions
- `tikz` with `arrows` library — wheel chart and tag rendering
- `tcolorbox` with `skins` — colored boxes
- `enumitem` — list customization
- `graphicx` — image inclusion
- `etoolbox` — LaTeX utilities
- `dashrule` — dashed rule dividers
- `multirow`, `tabularx` — table layouts for achievements
- `changepage` — `adjustwidth` environment for header width
- `biblatex` with `biber` backend — bibliography management
- `geometry` — page layout
- Font: **Lato** (used in both sample templates; must be installed on the system)

## Core Class: `altacv.cls`

The entire formatting system is defined in `altacv.cls`. It loads `article` as the base class and extends it with CV-specific commands.

### Color System

Four named colors control the appearance and can be overridden in any `.tex` file:

| Color name  | Default             | Used for                         |
|-------------|---------------------|----------------------------------|
| `accent`    | `blue!70!black`     | Icons, skill ratings, highlights |
| `heading`   | `black`             | Section titles                   |
| `emphasis`  | `black`             | Bold text, job titles            |
| `body`      | `black!80!white`    | Regular body text                |

Override colors in the `.tex` preamble:
```latex
\definecolor{VividPurple}{HTML}{3E0097}
\colorlet{heading}{VividPurple}
\colorlet{accent}{VividPurple}
```

Two marker commands are also overridable:
```latex
\renewcommand{\itemmarker}{{\small\textbullet}}
\renewcommand{\ratingmarker}{\faCircle}
```

### Page Layout

Default margin (from `geometry`): `2cm` all sides.

Both sample files override to a two-column layout with a right sidebar:
```latex
\geometry{left=1cm, right=9cm, marginparwidth=6.8cm, marginparsep=1.2cm, top=1.25cm, bottom=1.25cm}
```

The sidebar content is placed in the margin via `\marginpar` (triggered by `\cvsection`).

### Commands Reference

#### Header

```latex
\name{Your Name}
\tagline{Your Tagline}
\photo{2.8cm}{image-filename-without-extension}
\personalinfo{
  \email{you@example.com}
  \phone{000-00-0000}
  \mailaddress{Street, City, Postcode}
  \location{City, Country}
  \homepage{www.example.com}
  \twitter{@handle}
  \linkedin{linkedin.com/in/yourprofile}
  \github{github.com/yourid}
  \orcid{orcid.org/0000-0000-0000-0000}
}
\makecvheader
```

Use `\printinfo{symbol}{detail}` to add custom contact fields beyond the built-in ones.

To extend the header across the full page width (past the right margin):
```latex
\begin{adjustwidth}{}{-8cm}
\makecvheader
\end{adjustwidth}
```
(The `8cm` = `marginparwidth` 6.8cm + `marginparsep` 1.2cm.)

#### Sections and Sidebar

```latex
% Section with sidebar content loaded from a file
\cvsection[sidebar-filename]{Section Title}

% Section without sidebar
\cvsection{Section Title}

% Subsection
\cvsubsection{Subsection Title}
```

Sidebar content is injected via `\marginpar` automatically. The sidebar file should contain only LaTeX content (no `\begin{document}`).

#### Work Experience / Events

```latex
\cvevent{Job Title}{Company/Institution}{Date Range}{Location}
```

- All four arguments are required; pass `{}` for an empty location.
- Add bullet points using a standard `itemize` environment below.
- Separate adjacent `\cvevent` blocks with `\divider`.

```latex
\cvevent{President \& CEO}{Yahoo!}{July 2012 -- Ongoing}{Sunnyvale, CA}
\begin{itemize}
\item Led the \$5 billion acquisition...
\end{itemize}

\divider

\cvevent{VP of Search}{Google}{2005 -- 2010}{Palo Alto, CA}
```

#### Achievements (for sidebar)

```latex
\cvachievement{icon}{Title}{Description text}
```

`icon` is a fontawesome command, e.g. `\faTrophy`, `\faHeartbeat`, `\faGraduationCap`.

#### Skills

```latex
% Skill with 1–5 dot rating
\cvskill{Skill Name}{4}

% Keyword tag (rounded box)
\cvtag{Python}
\cvtag{Machine Learning}
```

#### Wheel Chart

Used to visualize time distribution across activities:

```latex
% \wheelchart{outer radius}{inner radius}{data}
% Data format: value/text-width/color/label
\wheelchart{1.5cm}{0.5cm}{%
  6/8em/accent!30/{Sleep,\\beautiful sleep},
  8/8em/accent!60/Daytime job,
  5/6em/accent!20/Spending time with family
}
```

Values are relative (not percentages); the class calculates proportions automatically.

#### References / Referees

```latex
\cvref{Prof. Full Name}{Institution}{email@university.edu}{Address Line 1\\Address Line 2}
```

#### Divider

```latex
\divider
```

A dashed horizontal rule with `body!30` color, used to separate entries within a section.

#### Bibliography (Publications)

Load the `.bib` file in the preamble:
```latex
\addbibresource{sample.bib}
```

Print filtered bibliography sections:
```latex
\nocite{*}  % Include all entries
\printbibliography[heading=pubtype, title={\printinfo{\faBook}{Books}}, type=book]
\divider
\printbibliography[heading=pubtype, title={\printinfo{\faFileTextO}{Journal Articles}}, type=article]
\divider
\printbibliography[heading=pubtype, title={\printinfo{\faGroup}{Conference Proceedings}}, type=inproceedings]
```

## Sidebar File Pattern

Sidebar content lives in separate `.tex` files (no preamble or `\begin{document}`):

```latex
% example: sample-p1sidebar.tex
\cvsection{My Life Philosophy}
\begin{quote}``A meaningful quote.''
\end{quote}

\cvsection{Most Proud of}
\cvachievement{\faTrophy}{Achievement Title}{Details}
\divider
\cvachievement{\faHeartbeat}{Another Achievement}{More details}

\cvsection{Strengths}
\cvtag{Leadership}
\cvtag{Communication}

\cvsection{Languages}
\cvskill{English}{5}
\divider
\cvskill{Spanish}{4}

\cvsection{Education}
\cvevent{Ph.D.\ in Physics}{MIT}{2002 -- 2006}{}
```

Reference the sidebar file in the main `.tex` via `\cvsection[filename-without-extension]{Section Title}`.

## Creating a New CV

1. Copy `sample.tex` and `sample-p1sidebar.tex` as starting points.
2. Update the preamble colors, font, and geometry as needed.
3. Replace placeholder content in both files.
4. Add a `.bib` file for publications (or remove the Publications section).
5. Compile with LuaLaTeX (and Biber if using bibliography).

## Key Conventions

- **No build automation** — compilation is manual via `lualatex` / `biber`.
- **Sidebar files are plain LaTeX fragments** — they have no document structure.
- **Sidebar injection is positional** — the sidebar aligns to the top of the section heading where it is declared.
- **All colors are globally scoped** — redefine `accent`, `heading`, `emphasis`, and `body` in the preamble to change the entire document's theme.
- **fontawesome icons are used throughout** — reference `texdoc fontawesome` for the full symbol list.
- **Photo images must be in the same directory** as the `.tex` file (or on the LaTeX search path). The `\photo` command strips the extension automatically.
- **Ampersands and dollar signs in content must be escaped**: `\&`, `\$`.
- **Non-breaking spaces in abbreviations**: use `Prof.\ Alpha` or `Ph.D.\ in` to prevent bad line breaks after periods.
- **Spacing adjustments**: use `\smallskip`, `\medskip`, `\bigskip`, or `\vspace{Xpt}` when layout needs fine-tuning within sidebar files.

## License

LaTeX Project Public License (LPPL) v1.3. The project is "maintained" with LianTze Lim as current maintainer.

The `mmayer-wikipedia-cc-by-2_0.jpg` photo is licensed under CC-BY 2.0 from Wikipedia.
