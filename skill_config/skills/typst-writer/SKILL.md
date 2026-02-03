---
name: typst-writer
description: Comprehensive Typst guide covering syntax, patterns, templates, and idiomatic code writing. Use when creating or editing Typst (.typ) files, answering questions about syntax, working with markup, functions, tables, math, or using Typst packages.
---

# Typst Writer

Complete guide for writing correct and idiomatic Typst code, covering syntax reference, patterns, best practices, package usage, templates, and troubleshooting.

## Quick Start

**This skill is for ALL Typst-related tasks:**
- Writing or editing .typ files
- Answering syntax questions
- Working with templates and packages
- Debugging common errors

## Core Syntax

### Text Formatting

```typst
*bold text*
_italic text_
`inline code`
```

### Headings

```typst
= Level 1 Heading
== Level 2 Heading
=== Level 3 Heading
```

### Lists

```typst
- Unordered item
- Another item
  - Nested item

+ Ordered item
+ Second item
  + Nested ordered
```

### Links and References

```typst
#link("https://typst.app")[Typst Website]
@label-name          // Reference a labeled element
#ref(<label-name>)   // Alternative reference syntax
```

## Math Mode

Inline math uses `$...$`, display math uses `$ ... $` with spaces:

```typst
The equation $E = m c^2$ is famous.

Display equation:
$ integral_0^infinity e^(-x^2) dif x = sqrt(pi) / 2 $
```

Common math notation:
- Subscripts: `x_1`, `a_(n+1)`
- Superscripts: `x^2`, `e^(i pi)`
- Fractions: `a/b` or `frac(a, b)`
- Square roots: `sqrt(x)`, `root(3, x)` for cube root
- Summations: `sum_(i=0)^n`
- Greek letters: `alpha`, `beta`, `gamma`, `theta`

## Function Calls

Typst uses `#` to call functions:

```typst
#image("photo.png", width: 50%)
#table(
  columns: 3,
  [Header 1], [Header 2], [Header 3],
  [Cell 1], [Cell 2], [Cell 3],
)
#figure(
  image("diagram.svg"),
  caption: [A diagram showing──architecture],
)
```

## Document Structure

### Page Setup

```typst
#set page(
  paper: "a4",
  margin: (x: 2cm, y: 2.5cm),
  header: [Document Header],
  footer: context [Page #counter(page).display()],
)
```

### Text Configuration

```typst
#set text(
  font: "New Computer Modern",
  size: 11pt,
  lang: "en",
)

#set par(
  justify: true,
  leading: 0.65em,
  first-line-indent: 1em,
)
```

## Show Rules

Transform elements using show rules:

```typst
// Style all links
#show link: set text(fill: blue)

// Custom heading appearance
#show heading: it => block(
  fill: luma(230),
  inset: 8pt,
  radius: 4pt,
  it
)

// Replace text patterns
#show "TODO": text(fill: red, weight: "bold")[TODO]
```

## Tables

```typst
#table(
  columns: (auto, 1fr, 1fr),
  align: (left, center, right),
  stroke: 0.5pt,
  inset: 8pt,
  fill: (_, row) => if row == 0 { luma(230) },

  [*Name*], [*Value*], [*Unit*],
  [Length], [42], [cm],
  [Width], [10], [cm],
)
```

## Figures and Captions

```typst
#figure(
  image("chart.png", width: 80%),
  caption: [Sales data for Q1 2024],
) <fig:sales>

As shown in @fig:sales, sales increased.
```

## Bibliography

```typst
// In document
#bibliography("refs.bib", style: "ieee")

// Citation
@smith2024 shows that...
#cite(<smith2024>, form: "prose")
```

## Variables and Functions

### Variables

```typst
#let title = "My Document"
#let primary-color = rgb("#1a73e8")
```

### Functions

```typst
// Define custom functions
#let highlight(body) = box(
  fill: yellow.lighten(60%),
  inset: 4pt,
  radius: 2pt,
  body
)

Use like: #highlight[important text]
```

### Context and State

```typst
// Access page/document context
#context {
  let current-page = counter(page).get().first()
  [Page #current-page of #counter(page).final().first()]
}

// State management
#let note-counter = counter("notes")

#let note(body) = {
  note-counter.step()
  super(context note-counter.display())
  // Store note for later
}
```

## Conditionals

```typst
#if condition [
  Content when true
] else [
  Content when false
]

// Inline conditional
#text(fill: if important { red } else { black })[Text]
```

## Loops

```typst
// For loop over array
#for item in (1, 2, 3) [
  Item: #item \
]

// For loop with index
#for (i, item) in items.enumerate() [
  #(i + 1). #item \
]

// While loop
#let i = 0
#while i < 3 {
  [Item #i]
  i += 1
}
```

## Layout Patterns

### Two-Column Layout

```typst
#set page(columns: 2, margin: (x: 1.5cm, y: 2cm))

// Or manual columns
#columns(2, gutter: 1em)[
  Left column content.
  #colbreak()
  Right column content.
]
```

### Boxed Content

```typst
#box(
  fill: luma(240),
  inset: 1em,
  radius: 4pt,
  width: 100%,
)[
  Important note or callout.
]

// Reusable callout
#let callout(title, body) = block(
  fill: rgb("#e8f4f8"),
  inset: 1em,
  radius: 4pt,
  width: 100%,
)[
  #text(weight: "bold")[#title] \
  #body
]
```

## Common Templates

### Academic Paper

```typst
#set document(
  title: "Paper Title",
  author: "Author Name",
)

#set page(paper: "us-letter", margin: 1in)
#set text(font: "Times New Roman", size: 12pt)
#set par(justify: true, first-line-indent: 0.5in)
#set heading(numbering: "1.1")

#align(center)[
  #text(size: 14pt, weight: "bold")[Paper Title]

  Author Name \
  Institution \
  #link("mailto:email@example.com")
]

#outline()

= Introduction
...
```

### Letter

```typst
#set page(paper: "us-letter", margin: 1in)
#set text(size: 11pt)

#align(right)[
  Your Name \
  Your Address \
  #datetime.today().display()
]

#v(1cm)

Dear Recipient,

#lorem(50)

Sincerely,

#v(1cm)

Your Name
```

## Imports and Packages

### Import from Local File

```typst
// Import from local file
#include "chapter1.typ"

// Import functions/variables from file
#import "utils.typ": format-date, highlight
```

### Import from Typst Universe

```typst
// Import from Typst Universe package
#import "@preview/cetz:0.3.4": canvas, draw

#canvas({
  draw.circle((0, 0), radius: 1)
})
```

## Popular Packages

### Essential Packages

| Package | Purpose |
|---------|---------|
| `drafting` | Annotations/comments for work-in-progress docs |
| `gentle-clues` | Callouts, tips, notes, admonitions |
| `showybox` | General-purpose text boxes with headers and footers |
| `fletcher` | Graphs, flowcharts, automata, trees with automatic placing |
| `tablem` | Write tables in markdown-like format |
| `polylux` | Presentations and slides |
| `cezt` | General diagrams/drawings with explicit coordinates |

### Searching for Packages

**Typst Universe**: https://typst.app/universe
**GitHub repositories**: Use `gh search repos --language "typst"` or SearXNG

## Common Mistakes to Avoid

- ❌ Calling things "tuples" - Typst only has arrays
- ❌ Using `[]` for arrays - use `()` instead
- ❌ Accessing array elements with `arr[0]` - use `arr.at(0)`
- ❌ Forgetting `#` prefix for code in markup context
- ❌ Mixing up content blocks `[]` with code blocks `{}`

## Debugging Tips

Use `#repr(value)` to inspect values during debugging:
```typst
#repr(some-variable)  // Shows type and value
#type(value)          // Shows just the type
```

Check diagnostics from tinymist LSP for undefined variables, type mismatches.

## Troubleshooting

### Missing Font Warnings

If you see "unknown font family" warnings, remove font specification to use system defaults.

### Compilation Errors

Common fixes:
- **"expected content, found ..."**: Wrap value in brackets: `[#value]`
- **"expected expression, found ..."**: Check function signature and argument types
- **"unknown variable"**: Check spelling, ensure `#let` before use

## Documentation Resources

- [Typst Official Documentation](https://typst.app/docs)
- [Typst Universe (Packages)](https://typst.app/universe)
- [tinymist LSP](https://github.com/Myriad-Dreamin/tinymist)

## Working with Templates

Templates provide complete document styling and structure.

### Finding Templates

Browse templates at https://typst.app/universe/search/?kind=templates
Filter by category: report, paper, thesis, cv, etc.

### Using a Template

1. Import template: `#import "@preview/template-name:version": *`
2. Apply it with `#show: template-name.with(param: value, ...)`
3. Consult template documentation for required and optional parameters

**Example:**
```typst
#import "@preview/bubble:0.2.2": bubble

#let doc-md = (
  title: [My report],
  author: [Claude],
  date: datetime(year: 2025, month: 12, day: 8),
)

#set document(..doc-md)

#show: bubble.with(
  ..doc-md,
  date: doc-md.date.display("[day]-[month]-[year]"),
  subtitle: [A detailed analysis],
)
```

## Complete Document Structure Example

### Academic Paper Pattern

```typst
#set document(
  title: "Document Title",
  author: "Your Name",
  date: auto,
)

#set page(
  paper: "a4",
  margin: (x: 2.5cm, y: 2.5cm),
  numbering: "1",
)

#set text(size: 11pt)
#set par(justify: true, first-line-indent: 0.5in)
#set heading(numbering: "1.")

// Title page
#align(center)[
  #text(size: 24pt, weight: "bold")[Document Title]

  #v(1cm)
  #text(size: 14pt)[Your Name]

  #v(0.5cm)
  #datetime.today().display()
]

#pagebreak()

// Table of contents
#outline(title: "Table of Contents", indent: auto)

#pagebreak()

// Main content
= Introduction
Your introduction text...

= Methods
...

= Results
...

= Conclusion
...

#pagebreak()

// Bibliography
#bibliography("refs.bib", title: "References", style: "ieee")
```

### Presentation Pattern

```typst
#import "@preview/polylux:0.5.0": *

#show: polylux.slide(
  title: [My Presentation],
  authors: ([Your Name]),
  subtitle: [Subtitle Here],
)
```

Your content follows...
