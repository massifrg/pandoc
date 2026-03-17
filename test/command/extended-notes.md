```
% pandoc -f markdown -t markdown
This text has a footnote[^1] that is referenced twice[^1],
and another footnote[^2] referenced only once.

[^1]: Double-reference footnote.

[^2]: Single-reference footnote.
^D
This text has a footnote[^1] that is referenced twice[^2], and another
footnote[^3] referenced only once.

[^1]: Double-reference footnote.

[^2]: Double-reference footnote.

[^3]: Single-reference footnote.
```
```
% pandoc -f markdown+multiref_notes -t markdown
This text has a footnote[^1] that is referenced twice[^1],
and another footnote[^2] referenced only once.

[^1]: Double-reference footnote.

[^2]: Single-reference footnote.
^D
This text has a footnote[[^1]]{noteref="1"} that is referenced
twice[[^2]]{noteref="1"}, and another footnote[^3] referenced only once.

[^1]: Double-reference footnote.

[^2]: Double-reference footnote.

[^3]: Single-reference footnote.
```
```
% pandoc -f markdown+multiref_notes -t markdown+multiref_notes
This text has a footnote[^1] that is referenced twice[^1],
and another footnote[^2] referenced only once.

[^1]: Double-reference footnote.

[^2]: Single-reference footnote.
^D
This text has a footnote[^1] that is referenced twice[^1], and another
footnote[^2] referenced only once.

[^1]: Double-reference footnote.

[^2]: Single-reference footnote.
```
```
% pandoc -f markdown+multiref_notes+keep_noterefs -t markdown+multiref_notes+keep_noterefs
This text has a footnote[^double] that is referenced twice[^double], and
another footnote[^single] referenced only once.

[^double]: Double-reference footnote.

[^single]: Single-reference footnote.
^D
This text has a footnote[^double] that is referenced twice[^double], and
another footnote[^single] referenced only once.

[^double]: Double-reference footnote.

[^single]: Single-reference footnote.
```
```
% pandoc -f markdown+keep_noterefs -t markdown+keep_noterefs
This text has a footnote[^double] that is referenced twice[^double], and
another footnote[^single] referenced only once.

[^double]: Double-reference footnote.

[^single]: Single-reference footnote.
^D
This text has a footnote[^double] that is referenced twice[^double], and
another footnote[^single] referenced only once.

[^double]: Double-reference footnote.

[^single]: Single-reference footnote.
```
