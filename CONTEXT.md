# Tablito

A language-agnostic specification for table pretty-printing. It defines shared vocabulary, behavior, and generated test cases so that implementers can build consistent table-printing libraries across different programming languages.

## Language

**Tablito**:
The specification itself — not a library and not generated code. It defines vocabulary, behavior, and test cases; implementers write the actual library per language against it.
_Avoid_: Library, tool, framework (when referring to the spec itself)

**Collection**:
The language-native input data structure Tablito extracts table rows from — e.g. an array of objects in Java, a list of lists in Clojure.
_Avoid_: Input, dataset, rows (rows are what Tablito produces from a collection, not what it receives)

**Row**:
One item of the Collection, in its original language-native shape (an object, a list, etc.). Passed whole to Decorators and to the Comparator, which pick whatever fields they need from it.
_Avoid_: Item, record, entry

**Table**:
The set of Columns plus an optional Comparator and an optional Filter, declared together, that Tablito applies to a Collection to produce output through a Driver.
_Avoid_: Report, grid

**Filter**:
At most one per table, declared alongside its Columns as part of the table definition: a predicate that receives a whole row and decides whether it appears in the output. Shaped like the Comparator — a single row-scoped function, not a composable chain; combining conditions is the caller's job when writing that one predicate.
_Avoid_: Query, condition, where-clause

**Header**:
The display title of a Column, shown by a Driver above its cells.
_Avoid_: Title, label (Header is the spec term)

**Driver**:
A pluggable output target that renders a prepared table into a final form — e.g. stdout (with ANSI coloring) or HTML. Every driver implements a shared, whole-table driver contract, so a new driver can be added without touching extraction, sorting, or layout. For each cell, a driver first asks its Decorator for the representation matching the driver's own format (e.g. an HTML driver asks for HTML); if the Decorator doesn't offer one, the driver falls back to the plain string and handles its own cutting/quoting/wrapping (e.g. fitting content to a terminal's width). Decides *how* content fits on the page — never *what* is displayed, that's the Decorator's job.
_Avoid_: Renderer, formatter, output format

**Decorator**:
The extractor nested inside a Column. Receives the whole row plus the Column itself, and produces a Cell Descriptor for that position. Has visibility on the entire row, so it can combine multiple fields from the data store into a single displayed column, or set styling conditionally (e.g. an Alarm on negative numbers). The default Decorator (extract by field name, null if the field is absent) computes Alignment automatically from the value's type (e.g. numbers right, text left) and always sets Weight/Alarm to none; there is no separate Column-level setting to override this — a caller who needs different styling writes a custom Decorator instead. Decides *what* is displayed and its semantic styling, never how that styling is realized on the page — that's the Driver's job.
_Avoid_: Extractor, mapper, accessor (use these only informally; Decorator is the spec term)

**Cell Descriptor**:
The structure a Decorator returns for one cell:
- **Contents**: the printable value — a plain string, or a richer format-specific representation (e.g. HTML) that a matching Driver can ask for instead (a Driver falls back to the string if it doesn't understand a richer representation); null renders as a blank cell.
- **Alignment**: left, center, or right.
- **Weight**: none, lite, italics, or bold.
- **Alarm**: none, yellow, or red — a semantic severity, not a raw color.
- **Min-visible-width**: the minimum number of characters that must stay visible even if the Driver truncates the cell.

Alignment/Weight/Alarm are semantic hints, not rendering instructions: each Driver decides how to realize them in its own medium (e.g. stdout might turn Alarm=red into a colored cell background, HTML might turn it into a CSS class).
_Avoid_: Style, cell metadata

**Column**:
A named unit the caller declares (or gets by default) that bundles a Decorator with a Header. Carries no styling settings of its own — all per-cell styling (Alignment, Weight, Alarm) comes from what the Decorator computes.
_Avoid_: Field (Field refers to raw data in the Collection; Column is what Tablito renders)

**Comparator**:
At most one per table, declared alongside its Columns as part of the table definition: a function that receives two whole rows and decides their order, looking at whatever columns it needs internally — not a per-column setting. Tablito provides a standard helper that builds a Comparator from an ordered list of (Column, direction) pairs (e.g. by this Column ascending, then that Column descending); callers may also supply a fully custom row-vs-row Comparator.
_Avoid_: Sorter, ordering function, column comparator
