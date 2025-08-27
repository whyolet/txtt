# txtt

`txtt` (text tree) is a lightweight, indentation-based format for lists and maps of text strings.

## Example

```
- hello world
-
  multiple lines
  of text
[
  - nested list
{
  key: text line
  multiple:
    lines
    of text
  list[
    - item
  map{
    key: value
# comment
```

The same as JSON:

```
[
  "hello world",
  "multiple lines\nof text",
  [
    "nested list"
  ],
  {
    "key": "text line",
    "multiple": "lines\nof text",
    "list": [
      "item"
    ],
    "map": {
      "key": "value"
    }
  }
]
```

## Why

* JSON is too verbose, yet lacks comments,
* YAML leads to [implicit typing](https://hitchdev.com/strictyaml/why/implicit-typing-removed/) surprises,
* alternatives are not simple enough, from A to Z:
  * [CSON](https://github.com/bevry/cson)
  * [edn](https://github.com/edn-format/edn)
  * [ELDF](https://eldf.org/)
  * [ENO](https://eno-lang.org/)
  * [HCL](https://github.com/hashicorp/hcl)
  * [HJSON](https://hjson.github.io/)
  * [HOCON](https://github.com/lightbend/config/blob/master/HOCON.md)
  * [HRON](https://github.com/mrange/hron)
  * [INI](https://en.wikipedia.org/wiki/INI_file)
  * [ION](https://amzn.github.io/ion-docs/)
  * [JONF](https://github.com/whyolet/jonf) 
  * [JSON5](https://json5.org/)
  * [Jsonnet](https://jsonnet.org/)
  * [NestedText](https://nestedtext.org/)
  * [ODDL](https://openddl.org/)
  * [OGDL](https://ogdl.org/spec/)
  * [Property list](https://en.wikipedia.org/wiki/Property_list)
  * [SDLang](https://sdlang.org/)
  * [S-expression](https://en.wikipedia.org/wiki/S-expression)
  * [StrictYAML](https://hitchdev.com/strictyaml/)
  * [TOML](https://toml.io/en/)
  * [XML](https://www.w3.org/XML/)

## Rules

### File

* `txtt` format supports Unicode as is, without escape sequences like `\uFFFF`, default encoding is UTF-8.
* `txtt` file is parsed line by line with a simple strict state machine.
* Initial state is a list:
  * to support zero, one, or multiple root values in one file naturally,
  * without additional concepts like "document" and separators like `---` in YAML.

### List

```
- text line
-
  multiple lines
  of text
[
  # list
{
  # map

# comment
```

* List is a sequence of zero or more of:
  * `- ` (dash, one space) followed by a text line.
  * `-` (dash, no space) followed by a newline and indented multiline text.
  * `[` followed by a newline and indented list.
  * `{` followed by a newline and indented map.
  * `#` followed by a comment.
  * Empty line which is not part of anything above. It is ignored.
* List ends like [Indented value](#indented-value), so the closing `]` is not supported.
* Inline structure like `[- text1 - text2 [] {} /* */]` is not supported.
* Empty lists:
  ```
  [
  ```

  ```
  {
    key[
  ```

### Text line

```
- text line
{
  key: text line
```

* Text line ends before a newline or the end of file.
* Text line is returned as a literal string value, never a number, boolean, or anything else.
* Leading and trailing whitespace is not trimmed.
* Escape sequences are not supported because:
  * any character except the newline can be represented here as is,
  * newline is supported by indented multiline text.

### Newline

* Both parser and formatter treat newline as `\n` only,
* because accepting `\r\n` and single `\r` to implement the [robustness principle](https://en.wikipedia.org/wiki/Robustness_principle) would actually lead to the [lack of robustness](https://en.wikipedia.org/wiki/Robustness_principle#Criticism),
* and normalization of `\r\n` and `\r` to `\n` would require a special way to preserve `\r` in the output like a `"quoted text with escape sequences like \r\n"`,
* which would increase complexity without much value added.

### Indented value

* Indented value ends before:
  * non-empty line with decreased indentation,
  * end of file.
* A value indented with N spaces is returned after deletion of exactly N first spaces from each line, even if some lines start with more than N spaces.
* One indentation level is exactly 2 spaces to keep it standardized, compact, and readable.

```
quotes[
  {
    text:
      You can have
      any color you want,

        as long as it's black.
    author: Henry Ford
  {
    text: Any color you like.
    author: https://github.com/psf/black
```

### Multiline text

```
-
  multiple lines
  of text
{
  key:
    multiple lines

    of text

  # empty text
  key2:

# empty text
-
```

* Multiline text is returned as a literal string value.
* It includes empty lines if any.
* It includes the trailing newline as recommended for whole files.
* After the [Indented value](#indented-value) rule deletes the indentation, remaining leading and trailing whitespace is not trimmed.
* Escape sequences are not supported because any unicode character including any kind of newline can be represented here as is.

### Map

```
key1: text line
key2:
  multiple lines
  of text
key3[
  # list
key4{
  # map

# comment
```

* Map is a sequence of zero or more of:
  * Key followed by `: ` (colon, one space) and a text line.
  * Key followed by `:` (colon, no space), a newline, and indented multiline text.
  * Key followed by `[`, a newline, and indented list.
  * Key followed by `{`, a newline, and indented map.
  * `#` followed by a comment.
  * Empty line which is not part of anything above. It is ignored.
*  Map ends like [Indented value](#indented-value), so the closing `}` is not supported.
* Inline structure like `{key1: text1, key2[- text2 - text3], key3{} /* */}` is not supported.
* Empty maps:
  ```
  {
  ```

  ```
  {
    key{
  ```

### Key

```
unquoted key: value
```

```
"quoted: key": value
```

```
unquoted"
multiline key: value
```

```
"quoted key: key[ key{

key"" key": value
```

```
# empty key
: value
```

* Key is either quoted or unquoted.
* Quoted key:
  * starts with `"` character,
  * followed by zero or more of:
    * `""`
    * any character other than `"`
  * ends with `"` character.
* Quoted key is returned as a literal string value after deleting the first and the last `"` and replacing each `""` with a single `"`.
* Unquoted key ends before any of `:[{` characters.
* Unquoted key is returned as a literal string value.
* If the end of file or the end of indented map is reached and the key is still not ending, then it is an invalid key.

### Comment

* Comment ends with a newline or before the end of file.
* Comment is ignored.
* Comment styles other than `#` are not supported to keep it standardized and minimalist.
* Inline comments are not supported to keep it simple and unambiguous:
  ```
  # You can always comment before
  - https://example.com/#section4
  # or after.
  ```

## Roadmap

* Review.
* Strict grammar file:
  * for syntax highlighters and linters,
  * to generate parsers and formatters for multiple languages.
* Generate and test a VS Code/Cursor extension.
* Generate and test reference parser and formatter for JS.
* Use it.
