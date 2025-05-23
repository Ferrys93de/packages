<!--typst-begin-exclude-->

# cmarker
<!--typst-end-exclude-->

This package enables you to write CommonMark Markdown,
and import it directly into Typst.

<!--typst-begin-exclude-->
<table>
<tr>
<th><code>simple.typ</code></th>
<th><code>simple.md</code></th>
</tr>
<tr>
<td>

```typst
#import "@preview/cmarker:0.1.2"

#cmarker.render(read("simple.md"))
```
</td>
<td>

```markdown
# We can write Markdown!

*Using* __lots__ ~of~ `fancy` [features](https://example.org/).
```
</td>
</tr>
</table>
<table>
<tr><th><code>simple.pdf</code></th></tr>
<tr><td><img src="./examples/simple.png"></td></tr>
</table>
<!--typst-end-exclude-->

<!--raw-typst
#table(
	columns: (1.2fr, 1fr),
	[`simple.typ`],
	[`simple.md`],
	[```typst
#import "@preview/cmarker:0.1.2"

#cmarker.render(read("simple.md"))
	```],
	[```markdown
# We can write Markdown!

*Using* __lots__ ~of~ `fancy` [features](https://example.org/).
	```],
)
#table(columns: (100%), [`simple.pdf`], image("./examples/simple.png", height: auto))
-->

This document is available
in [Markdown](https://github.com/SabrinaJewson/cmarker.typ/tree/main#cmarker)
and [rendered PDF](https://github.com/SabrinaJewson/cmarker.typ/blob/main/README.pdf)
formats.

## API

We offer a single function:

```typc
render(
  markdown,
  smart-punctuation: true,
  blockquote: none,
  math: none,
  h1-level: 1,
  raw-typst: true,
  scope: (:),
  show-source: false,
) -> content
```

The parameters are as follows:
- `markdown`:
	The [CommonMark](https://spec.commonmark.org/0.30/) Markdown string to be processed.
	Parsed with the [pulldown-cmark](https://docs.rs/pulldown-cmark) Rust library.
	You can set this to `read("somefile.md")` to import an external markdown file;
	see the
	[documentation for the read function](https://typst.app/docs/reference/data-loading/read/).
	- Accepted values: Strings and raw text code blocks.
	- Required.

- `smart-punctuation`:
	Automatically convert ASCII punctuation to Unicode equivalents:
	- nondirectional quotations (" and ') become directional (“” and ‘’);
	- three consecutive full stops (...) become ellipses (…);
	- two and three consecutive hypen-minus signs (-- and ---)
		become en and em dashes (– and —).

	Note that although Typst also offers this functionality,
	this conversion is done through the Markdown parser rather than Typst.
	- Accepted values: Booleans.
	- Default value: `true`.

- `blockquote`:
	A callback to be used when a blockquote is encountered in the Markdown,
	or `none` if blockquotes should be treated as normal text.
	Because Typst does not support blockquotes natively,
	the user must configure this.
	- Accepted values: Functions accepting content and returning content, or `none`.
	- Default value: `none`.

	For example, to display a black border to the left of the text
	one can use:
	```typc
	box.with(stroke: (left: 1pt + black), inset: (left: 5pt, y: 6pt))
	```
	<!--raw-typst
	#(box.with(stroke: (left: 1pt + black), inset: (left: 5pt, y: 6pt)))[which displays like this.]
	-->

- `math`:
	A callback to be used when equations are encountered in the Markdown,
	or `none` if it should be treated as normal text.
	Because Typst does not support LaTeX equations natively,
	the user must configure this.
	- Accepted values:
		Functions that take a boolean argument named `block` and a positional string argument
		(often, the `mitex` function from
		[the mitex package](https://typst.app/universe/package/mitex)),
		or `none`.
	- Default value: `none`.

	For example, to render math equation as a Typst math block,
	one can use:
	```typc
	#import "@preview/mitex:0.2.4": mitex
	#cmarker.render(`$\int_1^2 x \mathrm{d} x$`, math: mitex)
	```
	<!--raw-typst
	which renders as: $integral_1^2 x dif x$
	-->

- `h1-level`:
	The level that top-level headings in Markdown should get in Typst.
	When set to zero,
	top-level headings are treated as text,
	`##` headings become `=` headings,
	`###` headings become `==` headings,
	et cetera;
	when set to `2`,
	`#` headings become `==` headings,
	`##` headings become `===` headings,
	et cetera.
	- Accepted values: Integers in the range [0, 255].
	- Default value: 1.

- `raw-typst`:
	Whether to allow raw Typst code to be injected into the document via HTML comments.
	If disabled, the comments will act as regular HTML comments.
	- Accepted values: Booleans.
	- Default value: `true`.

	For example, when this is enabled, `<!--raw-typst #circle(radius: 10pt) -->`
	will result in a circle in the document
	(but only when rendered through Typst).
	See also `<!--typst-begin-exclude-->` and `<!--typst-end-exclude-->`,
	which is the inverse of this.

- `scope`:
	When `raw-typst` is enabled,
	this is a dictionary
	providing the context in which the evaluated Typst code runs.
	It is useful to pass values in to code inside `<!--raw-typst-->` blocks.
	- Accepted values: Any dictionary.
	- Default value: `(:)`.

- `show-source`:
	A debugging tool.
	When set to `true`, the Typst code that would otherwise have been displayed
	will be instead rendered in a code block.
	- Accepted values: Booleans.
	- Default value: `false`.

This function returns the rendered `content`.

## Resolving Paths Correctly

Because of how Typst handles paths,
elements like images will by default resolve
relative to the project root of `cmarker` itself
and not your project.

To fix this,
one can override the `image` function in the scope the Typst code is evaluated.

```typst
#import "@preview/cmarker:0.1.2"

#cmarker.render(
  read("yourfile.md"),
  scope: (image: (path, alt: none) => image(path, alt: alt))
)
```

## Supported Markdown Syntax

We support CommonMark with a couple extensions.

- Paragraph breaks: Two newlines, i.e. one blank line.
- Hard line breaks (used more in poetry than prose):
	Put two spaces at the end of the line.
- `*emphasis*` or `_emphasis_`: *emphasis*
- `**strong**` or `__strong__`: **strong**
- `~strikethrough~`: ~strikethrough~
- `[links](https://example.org)`: [links](https://example.org/)
- `### Headings`, where `#` is a top-level heading,
	`##` a subheading, `###` a sub-subheading, etc
- `` `inline code blocks` ``: `inline code blocks`
-
	````
	```
	out of line code blocks
	```
	````
	Syntax highlighting can be achieved by specifying a language after the opening backticks:
	````
	```rust
	let x = 5;
	```
	````
	giving:
	```rust
	let x = 5;
	```
- `---`, making a horizontal rule:

---
-
	```md
	- Unordered
	- lists
	```
	- Unordered
	- Lists
-
	```md
	1. Ordered
	1. Lists
	```
	1. Ordered
	1. Lists
- `$x + y$` or `$$x + y$$`: math equations, if the `math` parameter is set.
- `> blockquotes`, if the `blockquote` parameter is set.
- Images: `![Some tiled hexagons](examples/hexagons.png)`, giving
	![Some tiled hexagons](examples/hexagons.png)
- Tables:
	```md
	| Column 1 | Column 2 |
	| -------- | -------- |
	| Row 1 Cell 1 | Row 1 Cell 2 |
	| Row 2 Cell 1 | Row 2 Cell 2 |
	```

| Column 1 | Column 2 |
| -------- | -------- |
| Row 1 Cell 1 | Row 1 Cell 2 |
| Row 2 Cell 1 | Row 2 Cell 2 |

## Interleaving Markdown and Typst

Sometimes,
you might want to render a certain section of the document
only when viewed as Markdown,
or only when viewed through Typst.
To achieve the former,
you can simply wrap the section in
`<!--typst-begin-exclude-->` and `<!--typst-end-exclude-->`:

```md
<!--typst-begin-exclude-->
Hello from not Typst!
<!--typst-end-exclude-->
```

Most Markdown parsers support HTML comments,
so from their perspective this is no different
to just writing out the Markdown directly;
but `cmarker.typ` knows to search for those comments
and avoid rendering the content in between.

Note that when the opening comment is followed by the end of an element,
`cmarker.typ` will close the block for you.
For example:

```md
> <!--typst-begin-exclude-->
> One

Two
```

In this code,
“Two” will be given no matter where the document is rendered.
This is done to prevent us from generating invalid Typst code.

Conversely, one can put Typst code inside
a HTML comment of the form
`<!--raw-typst […]-->`
to have it evaluated directly as Typst code
(but only if the `raw-typst` option to `render` is set to `true`,
otherwise it will just be seen as a regular comment and removed):

```md
<!--raw-typst Hello from #text(fill:blue)[Typst]!-->
```

## Limitations

- We do not currently support HTML tags, and they will be stripped from the output;
	for example, GitHub supports writing `<sub>text</sub>` to get subscript text,
	but we will render that as simply “text”.
	In future it would be nice to support a subset of HTML tags.
- We do not currently support Markdown footnotes.
	In future it would be good to.
- Although I tried my best to escape everything correctly,
	I won’t provide a hard guarantee that everything is fully sandboxed
	even if you set `raw-typst: false`.
	That said, Typst itself is well-sandboxed anyway.

## Development

- Build the plugin with `./build.sh`,
	which produces the `plugin.wasm` necessary to use this.
- Compile examples with `typst compile examples/{name}.typ --root .`.
- Compile this README to PDF with `typst compile README.md`.

<!--*///-->
