# Lexers and Unicode text

Until ANTLR 4.7, generated lexers only supported part of the Unicode standard
(code points up to `U+FFFF`).

With ANTLR 4.7 and later, lexers as well as all languages' runtimes
support the full range of Unicode code points up to `U+10FFFF`, as
long as the input `CharStream` is opened using `CharStreams.fromPath()`
or the equivalent method for your runtime's language.

# Unicode Code Points in Lexer Grammars

To refer to Unicode [code points](https://en.wikipedia.org/wiki/Code_point)
in lexer grammars, use the `\u` string escape plus up to 4 hex digits. For example, to create
a lexer rule for a single Cyrillic character by creating a range from
`U+0400` to `U+04FF`:

```ANTLR
CYRILLIC : '\u0400'..'\u04FF' ; // or [\u0400-\u04FF] without quotes
```

Unicode literals larger than U+FFFF must use the extended `\u{12345}` syntax. For example, to create a lexer rule for a selection of smiley faces
from the [Emoticons Unicode block](http://www.unicode.org/charts/PDF/U1F600.pdf):

```ANTLR
EMOTICONS : ('\u{1F600}' | '\u{1F602}' | '\u{1F615}') ; // or [\u{1F600}\u{1F602}\u{1F615}]
```

Finally, lexer char sets can include Unicode properties. Each Unicode code point has at least one property that describes the type group to which it belongs (e.g. alpha, number, punctuation). Other properties can be the language script or special binary properties and Unicode code blocks. That means however, that a property specifies a group of code points, hence they are only allowed in lexer char sets.

```ANTLR
EMOJI : [\p{Emoji}] ;
JAPANESE : [\p{Script=Hiragana}\p{Script=Katakana}\p{Script=Han}] ;
NOT_CYRILLIC : [\P{Script=Cyrillic}] ;
```

See [lexer-rules.md](lexer-rules.md#lexer-rule-elements) for more detail on Unicode
escapes in lexer rules.

# CharStreams and UTF-8

## Java Target
If your lexer grammar contains code points larger than `U+FFFF`, your
lexer client code must open the file using `CharStreams.fromPath()` or
equivalent in your runtime's language, or input values larger than
`U+FFFF` will *not* match.

For backwards compatibility, the existing `ANTLRInputStream` and
`ANTLRFileStream` APIs only support Unicode code points up to `U+FFFF`.

The existing `TestRig` command-line interface supports all Unicode
code points.

## Other Targets
Other language targets usually have their `ANTLRInputStream` extended to support the full Unicode range. See the target documentation for supported input encodings (e.g. UTF-8) and other related details.

# Java Example

If you have generated a lexer named `UnicodeLexer`:

```Java
public static void main(String[] args) {
  CharStream charStream = CharStreams.fromPath(Paths.get(args[0]));
  Lexer lexer = new UnicodeLexer(charStream);
  CommonTokenStream tokens = new CommonTokenStream(lexer);
  tokens.fill();
  for (Token token : tokens.getTokens()) {
    System.out.println("Got token: " + token.toString());
  }
}
```
