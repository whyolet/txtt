# txtt

`txtt` is a simple text tree format.

## Example

```
# list
[
  - text line
  -
    multiple lines
    of indented text
  [
    - list in list
  # closed by indentation

# map
{
  key1: text line
  key2:
    multiple lines
    of indented text
  key3[
    # empty list in map
  key4[
    - list in map
    {
      map: in list
  key5{
  key6{
    map: in map
  "quoted: key": value
```

## Why

Because:
* JSON is too verbose for humans,
* YAML is a [total disaster](https://noyaml.com/),
* [JONF](https://github.com/whyolet/jonf) and [alternatives](https://github.com/whyolet/jonf#motivation) listed there are not simple enough.

## Rules

### File

* `txtt` format supports Unicode.
* Default encoding is UTF-8.
* `txtt` file can be parsed line by line with a simple state machine.
* Initial state is a list:
  * to have zero, single, or multiple root values in one file naturally,
  * without additional concepts like "document" and separators like `---` in YAML.

### List

```
- text line
-
  multiple lines
  of indented text
[
  # list
{
  # map

# comment
```

List is a sequence of zero or more of:
* `- ` (dash, one space) followed by a text line.
* `-` (dash, no space) followed by a newline and indented multiline text.
* `[` followed by a newline and indented list.
* `{` followed by a newline and indented map.
* `#` followed by a comment.
* Empty line which is not part of anything above. It is ignored.

### Text line

```
- text line
{
  key: text line
```

* Text line ends before a newline or the end of file.
* Text line is returned as a literal string value.
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
* Indentation of indented value is deleted from each of its lines.
* One indentation level is exactly two spaces to keep it standardized, compact, and readable.

```
quotes[
  {
    text:
      You can have any color you want,

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
  of indented text
{
  key:
    multiple lines

    of indented text

  # empty text
  key2:

# empty text
-
```

* Multiline text is returned as a literal string value.
* It includes empty lines if any.
* It includes the trailing newline as recommended for whole files.
* Escape sequences are not supported because any unicode character including any kind of newline can be represented here as is.

### Map

```
key1: text line
key2:
  multiple lines
  of indented text
key3[
  # list
key4{
  # map

# comment
```

Map is a sequence of zero or more of:
* Key followed by `: ` (colon, one space) and a text line.
* Key followed by `:` (colon, no space), a newline, and indented multiline text.
* Key followed by `[`, a newline, and indented list.
* Key followed by `{`, a newline, and indented map.
* `#` followed by a comment.
* Empty line which is not part of anything above. It is ignored.

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
  - https://example.com/#section8
  # or after.
  ```

## Roadmap

* Review.
* Strict grammar file:
  * for syntax highlighters and linters,
  * to generate parsers and formatters for multiple languages.
* Use it.
