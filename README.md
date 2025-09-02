# ezlib

This package can be used in two main ways

- Enable all of features with the `default` option, then disable one by one
  - Best for new documents of class article, book, report, or acmart
- Enable features one by one
  - Best when integrating into an existing document
  - Best when using a class other than article, book, report, or acmart
- If no options are passed, the package should do (almost) nothing
  - The "almost": loading kvoptions and etoolbox, and defining some private commands

Each of the features below can be enabled or disabled with an option

- The option always has the same name as the feature
- How to enable/disable a boolean feature
  - Enable xyz: pass `xyz` or `xyz=true` to this package
  - Disable xyz: pass `xyz=false` to this package
- How to enable/disable a non-boolean feature
  - Enable xyz: pass `xyz=default`, or pass another value
    - Can use `xyz=true` as alias for `xyz=default`, but this is discouraged
  - Disable xyz: pass `xyz=none`
    - Can use `xyz=false` as alias for `xyz=none`, but this is discouraged
- How to enable/disable many features at once
  - The default is set by an option `default`, which can have one of two values
    - `default=minimal`: everything disabled by default
    - `default=maximal`: everything enabled by default
  - Shortcuts
    - Leaving the `default` option absent is equivalent to `default=minimal`
    - Passing `default` with no value set is equivalent to `default=maximal`
  - With `default=maximal`, can still disable with `xyz=false` or `xyz=none`
- Some features have further options associated with them
  - See details below

Full feature list, with non-boolean features marked:

- ezlist
- ezeq
- ezrestate
- delimiters
- notation
- cleveref
- theorems
- algorithm
- mathic
- packages
- font (non-boolean)
- number within (non-boolean)
- margin (non-boolean)
- headings (non-boolean)

Throughout, many packages are loaded with options

- There are two ways to change/disable options for a package `pkg`
  - Load pkg before this package
  - Call `\PassOptionsToPackage{options}{pkg}` before this package
    - `\PassOptionsToPackage{}{pkg}` will prevent options
- To clarify, the \PassOptionsToPackage behavior is specific to this package
  - Usually, \PassOptionsToPackage adds additional options
  - But this package detects if \PassOptionsToPackage has been called already
- Random hack: `\PassOptionsToPackage{no ezlib}{pkg}` prevents pkg from loading
  - This could break stuff, but should work for things like natbib and microtype


## Feature: ezlist

Boolean feature (enabled by `default=maximal`)

Provides a single macro, `\*`, for making lists

Example usage:

```
\* hello
\* world
\** use multiple * for sublists
\** like this
\* back to top level
\* use a / to end the list
\*/
```

Can also use an optional argument to use numbers/letters, e.g.

```
\*[(a)] labeled (a)
\* labeled (b)
\** no label in this sublist
\* labeled (c)
\**[(i)] labeled (i)
\** labeled (ii)
\*/
```

### Advanced usage

More generally, use `[]` to pass any options the enumitem package supports

Optional feature: explicit start/end:

- Explicit start: use `\*[...]`, `\**[...]`, etc. with no text after.
  - Should be followed by another `\*`, `\**`, etc.
  - This is useful if you want to pass options in a separate line
- Explicit end: use `\**/`, `\***/`, etc.
  - This lets you have a sublist in the middle of an item

Example:

```
\*[(a), afterheading]
\* this is the first item, labeled (a)
\** the list options got their own line
\** see below for more about the \texttt{afterheading} option
\* this one is (b), and there's a sublist
\** with
\** some
\** stuff
\**/ but now we resume item (b) from the outer list
\* before finally going to (c)
\* more details about explicit begin
\** you need to pass options with [] for an explicit begin
\***
\*** so the line immediately above this appears
\** but passing an empty list of options counts as passing options, so
\***[]
\*** so the line immediately above this is omitted
\** edge case: when an explicit begin is immediately followed by an explicit end\dots
\***[]
\***/ \dots\ the list has a single empty item
\*/
```

Additional notes:

- The environment `\*` uses is called `ezlist`
  - For modifying options with `\setlist`, cleveref name, etc.
- This package loads enumitem with option `shortlabels` and does some other formatting
  - See top of guide for how to disable package options
- This package defines an enumitem key `afterheading` for lists after inline headings
  - It ensures the list starts on a new line after the heading
  - It prevents page breaks between the heading and the list
  - Main use case is lists that appear at the start of a theorem/lemma/etc.

### Subfeature: subenv (disable with `ezlist subenv=false`)

Also introduces enumitem key `subenv` for referencing parts of therems/lemmas/etc.

- Works with ezlist, but unfortunately not other list types
- Usage: use `\*[subenv]` inside a theorem/lemma/etc., then use `\label` within items
  - Result: `\cref` for those labels yields a `subtheorem` reference, e.g. "Theorem 1.2(c)"
- A few options to refer to just the part label, e.g. just "(c)"
  - `\subref{lbl}`, assuming subcaption package is loaded
    - The subcaption package's options might affect how it looks
  - `\ref{sub@lbl}`, where `lbl` is the original label
    - `\cref` and `\Cref` should work, too
- Supported usage
  - Should work with an ezlist inside any environment whose name is a counter
  - But does not work with lists other than ezlist
  - Should only be used with top-level lists

Editing subenv label appearance:

- Redefine the `\subenvFormatLabel` command to change the item labels
  - Should take one argument, which is the counter name
  - Default is `(\alph{#1})`, yielding (a), (b), (c)...
  - Example: changing to `(\roman{#1})` would yield (i), (ii), (iii)...
- Redefine the `\subenvFormatRef` command to change what the references look like
  - Should take one argument, which is the counter name
  - Default is to do whatever `\subenvFormatLabel` does


## Feature: ezeq

Boolean feature (enabled by `default=maximal`)

Makes `\[` and `\]` a (slightly) smarter equation interface

- Automatically chooses between equation, align, and gather
  - Conservative choice based on seeing whether `&` or `\\` appears in the text
- Automatically numbers only labeled equations, if desired
  - If using `\tag`, put it right after `\label`, as in `\label{...} \tag{...}`
- Enables `\allowdisplaybreaks` inside the `\[ \]`
  - If it appears anywhere in the `\[ \]`, it applies to the whole thing
  - It is local to the `\[ \]` in which it appears
  - The optional argument of `\allowdisplaybreaks` is not supported
  - Remember that LaTeX provies `\\*` for non-breakable newlines
- Automatically inserts `\qedhere` if immediately before `\end{proof}`
  - When `ezlist` is activated, also inserts it before `\*/ \end{proof}`
  - Won't do anything if there's already a `\qedhere`
  - Put `\noAutoQed` somewhere in the equation to disable this for one equation
    - `\noAutoQed` has no other effect
  - Use `ezeq auto qed=false` to disable this globally
- Also slightly changes how `\qedhere` behaves
  - The new version is simpler and a bit less buggy for equation numbers on the right
  - But the new version is incompatible with equation numbers on the left
  - If this package detects equation numbers are on the left, will use the old version

Other configuration:

- `ezeq number labeled` (default): only number labeled equations
  - Equivalent to `ezeq number labeled=true` or `ezeq number all=false`
- `ezeq number all`: number all equations
  - Equivalent to `ezeq number labeled=false` or `ezeq number all=true`
- `ezeq auto qed` (boolean): whether to automatically place `\qedhere` in equations when at the end of a proof
  - Defaults to true


## Feature: ezrestate

Boolean feature (enabled by `default=maximal`)

Simple interface for thmtools's restatable theorems

- `\restatably` or `\restatably*`: makes any theorem/lemma/etc. restatable
  - Usage: `\restatably \begin{theorem} \label{lbl} ... \end{theorem}`
    - Spaces or newlines are okay
  - The theorem/lemma/etc. must start with a \label
- `restate` or `\restate*`: restates a `\restatably` stated theorem/lemma/etc.
  - Usage: `\restate*{lbl}` or `\restate*\ref{lbl}`
    - The `\ref` syntax is for convenient autocomplete, e.g. on Overleaf
  - Can be used multiple times

Starred vs. not starred

- Both `\restatably` and `\restate` can be starred (followed by `*`) or unstarred
  - Unstarred indicates the "main version", on which the numbering is based
  - Starred indicates that numbers should be copied from the main version
- Each label should appear once unstarred, and otherwise starred
  - Either the `\restatably` or a `\restate` can be the starred one
- The star always appears immediately after the macro
  - E.g. `\restatably* \begin{...}` or `\restate*\ref{...}`


## Feature: delimiters

Boolean feature (enabled by `default=maximal`)

Provides different "group" macros for sizing parentheses and similar

- `\gp{x}`: normal size
- `\gp*{x}`: automatically sized using \left/\right
- `\gp[size]{x}`: manually sized, where `size` is one of `\big`, `\Big`, `\bigg`, or `\Bigg`

See the mathtools `\DeclarePairedDelimiter` documentation for more details

Variants of `\gp`:

- `\gp` or `\pgp`: `()`
- `\sqgp` or `\bgp`: `[]`
- `\curlgp` or `\Bgp`: `\{\}`
- `\pbgp`: `(]`
- `\bpgp`: `[)`
- `\vgp`: `||`
- `\Vgp`: `\|\|`
- `\agp`: `\langle\rangle`
- `\floor`: `\lfloor\rfloor`
- `\ceil`: `\lceil\rceil`

Separators that automatically scale:

- `\mid`: `|`
- `\Mid`: `\|`

Generic version:

- `\lr`: like `\gp`, but after the optional `*`, takes delimiters as input
- `\lrStarAfter`: like `\lr`, but takes the delimiters first, then checks for `*`
- Limitation: neither `\lr` nor `\lrStarAfter` supports manual sizing
- Examples:
  - `\lr(){...}` is like `\gp{...}`
  - `\lr*(){...}` is like `\gp*{...}`
  - `\lrStarAfter(){...}` is like `\gp{...}`
  - `\lrStarAfter()*{...}` is like `\gp*{...}`
  - Delimiters need not be the same, e.g. `\lr*[){...}` is like `\bpgp*{...}`
- Intended usage
  - `\lr` is for direct usage in equations
  - `\lrStarAfter` is useful as the end of a macro definition
    - But the `\gp` family is even better, because it supports manual sizing


## Feature: notation

Boolean feature (enabled by `default=maximal`)

Various math notation shortcuts and conventions

Provides `\lr` and `\lrStarAfter` from delimiters above (but not the others)

Probability (best with delimiters option on, too):

- `\P`, `\E`, and `\Var`: have interface like delimiters
  - But they also support subscripts/superscripts and primes
  - If delimiters option isn't on, then manual sizing doesn't work
- `\given`: synonym for `\mid`
- Example: `\E_{X \sim \pi}*{\frac{1}{2} X^2 \given X > 3}`
- Change appearance with the following two options
  - `probability style` (default: `probability style=bf`)
    - Also supported: `bb`, `rm`, `sf`, and any X where a `\mathX` command exists
  - `probability delim` (default: `probability delim=sq`)
    - Also supported: `p`, `curl`, and any X where an `\Xgp` command exists
      - Without delimiters loaded, supports `p`, `sq`/`b`, and `curl`/`B`
    - Also supported: `[]`, `()`, and any valid followup to `\lrStarAfter`
      - Limitation: manual sizing won't work
- `\DeclareProbabilityCommand`: declare new commands that work like `\P`
  - Example: `\DeclareProbabilityCommand{\Cov}{Cov}`

Shortcuts for letters in different math fonts:

- `\bbA`, ..., `\bbZ`: short for `\mathbb{A}`, ..., `\mathbb{Z}` (uppercase only)
- `\calA`, ..., `\calZ`: short for `\mathcal{A}`, ..., `\mathcal{Z}` (uppercase only)
- `\bfA`, ..., `\bfz`: short for `\mathbf{A}`, ..., `\mathbf{z}` (lowecase and uppercase)
- `\sfA`, ..., `\sfz`: short for `\mathsf{A}`, ..., `\mathsf{z}` (lowecase and uppercase)

Other:

- `\1`: indicator function `\mathbb{1}`
- `\d`: differential symbol, use as `\d x` or `\d{x}`
- `\epsilon` and `\phi`: now refer to `\varepsilon` and `\varphi`
  - Use `\LaTeXepsilon` and `\LaTeXphi` for old versions
- `\argmin`, `\argmax`, `\liminf`, `\limsup`: math operators
- `\widebar`: like `\widetilde`, and a little nicer than `\overline`
  - Omitted if another package defines it first, e.g. newtxmath


## Feature: cleveref

Boolean feature (enabled by `default=maximal`)

Loads the cleveref package and a bit more

- Uses `capitalize` option by default
  - See top of guide for how to disable package options
- `\cCrefname`: simultaneous `\crefname` and `\Crefname`
- Allows spaces in comma-separated list of labels in `\cref` and `\Cref`

Mechanism for registering alternate versions of names:

- `\crefShortened`: wrap around a `\crefname` or similar to register a shortened version
  - Example: `\crefShortened{\cCrefname{theorem}{Thm.}{Thms.}}`
  - To omit the final "and": `\crefShortened{\def\creflastconjunction{, }\def\crefpairconjunction{, }}`
- `\crefShorten`: redefine all names using the shortened versions
  - This just executes all the code registered with `\crefShortened`
  - Redefinitions are local to any enclosing group


## Feature: theorems

Boolean feature (enabled by `default=maximal`)

Declares a default set of theorem environments

Theorem style synonyms, with support for class-specific analogues:

- `\ezlibPlain`: plain or class-specific analogue (e.g. acmplain for acmart)
- `\ezlibDefinition`: definition or analogue
- `\ezlibRemark`: remark or analogue
  - For some classes, e.g. acmart, this is the same as `\ezlibDefinition`
- If any of the above are already defined, they will not be redefined
  - This allows for changing styles by redefining the commands

Other:

- `\begin{case}[...] ... \end{case}`: cases in a proof
- `\noqed`: suppress the QED at the end of a proof
- `\cleartheorem`: undefines a theorem environment and its counter


## Feature: algorithm

Boolean feature (enabled by `default=maximal`)

Loads the alrogithm package and slightly tweaks algorithm formatting

- Passes the value of `number within` as an option to the algorithm package
  - See top of guide for how to disable package options


## Feature: mathic

Boolean feature (enabled by `default=maximal`)

Corrects spacing around inline math in italic text

- Redefines `$` in terms of `\( \)` and `\[ \]`
  - `$ $` becomes `\( \)`, possibly with an extra italic space tweak after (see below)
  - `$$ $$` becomes `\[ \]`
  - This changes the character code of `$`
- Attempts compatibility with tikz
  - Specifically, restores the character code of `$` within tikzpicture
- Attempts to also correct spacing after, but it's kind of a hack
  - Use `mathic after=false` to disable this
  - In contrast, spacing before is handled by mathtools
  - This applies only to `$ $`, not `\( \)`


## Feature: packages

Boolean feature (enabled by `default=maximal`)

Loads various packages, some with options

- booktabs
- calc
- caption, with options `font=small, labelfont=bf, labelsep=period`
- graphicx
- microtype
- natbib, with options `square, numbers, sort&compress`
- subcaption, with options `font=small, labelformat=simple`
  - If options are passed, also tweaks `\thesubfigure` to include parentheses
  - If ezlist subenv is enabled, uses `\subenvFormatLabel` in `\thesubfigure`
- Not a package, but also turns on `\frenchspacing`

See top of guide for how to disable package options


## Feature: font

Non-boolean feature (behavior under `default=maximal` described below)

Changes fonts used throughout the document

Possible values:

- `font=none`: don't change any fonts
- `font=libertinus` or `font=libertine`: use font set based around Libertinus (fork of Libertine and Biolinum)
- `font=charter`: use font set based around Charter and Vera Sans
- `font=palatino`: use font set based around Palatino
- Unless `font=none`, also loads fontenc with option `T1`

Other configuration:

- `ttfont=inconsolata`: use Inconsolata (default) for monospaced text
- `ttfont=sourcecodepro`: use Source Code Pro for monospaced text

Behavior with `default=maximal`:

- If a known built-in LaTeX class is loaded, default to `font=libertinus`
  - These are: article, report, book
- Otherwise, conservatively default to `font=none`
  - This avoids conflicts with journal/conference class files
- Either way, can be manually overridden by setting font explicitly


## Feature: number within

Non-boolean feature (behavior under `default=maximal` described below)

Selects what level numbering of theorems/equations/figures/etc. occurs within

Valid options:

- `number within=none`: use global numbering
  - Default under `default=minimal`
- `number within=default`: number within chapters or sections
  - Uses `chapter` if the document has chapters, `section` otherwise
  - This is the default under `default=maximal`
- Any other setting: number within the given environment
  - E.g. `number within=section`, `number within=subsection`, etc.


## Feature: margin

Non-boolean feature (behavior under `default=maximal` described below)

Sets the page margins using the geometry package

- If absent or set to `margin=none`, has no effect
- If set to any other value, loads geometry with that margin value
  - To set more complex margins, load geometry before this package

Behavior with `default=maximal`:

- If a known built-in LaTeX class is loaded, default to `margin=1.25in`
  - These are: article, report, book
- Otherwise, conservatively default to `margin=none`
  - This avoids conflicts with journal/conference class files
- Either way, can be manually overridden by setting margin explicitly


## Feature: headings

Non-boolean feature (behavior under `default=maximal` described below)

Formats section titles, headers, and more

Valid options:

- `headings=none`: leave LaTeX defaults
  - Default under `default=minimal`
- `headings=default`: choose between `fancy` and `none`
  - If a known built-in LaTeX class is loaded, use `fancy`
  - Otherwise, use `none`
  - This is the default under `default=maximal`
- `headings=fancy`: reformat section titles and headers
  - Section titles are styled in sans if the font has a nice sans pairing
  - Headers choose between two styles depending on if `\chapter` is defined
  - E.g. `number within=section`, `number within=subsection`, etc.
- `headings=inline`: use inline subsection titles
  - E.g. for short documents with a strict page limit
  - Equivalent to using `headings=fancy` and `headings inline=true` (see below)

Other configuriation:

- `headings sf` (boolean): set to true to make headings sans
  - Defaults to true if the font has a nice sans pairing, false otherwise
- `headings inline` (boolean): use inline subsection and lower-level headings
  - Defaults to false, but set to true by `headings=inline`
- `headings inline period` (boolean): if using inline headings, add period after heading
  - Defaults to true
