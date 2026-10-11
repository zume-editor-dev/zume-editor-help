# Security analysis

Zume Editor has tools for binary triage, data transforms and signature
scanning — useful for reverse engineering, CTFs and authorized security work.
Each is also available headlessly to AI agents through the
[MCP server](mcp.md#tools) (the `entropy`, `transform` and `yara_scan` tools).

## YARA Scan

**Code → YARA Scan…** scans the active document with a set of YARA rules.

The dialog asks for **the path to a YARA rules file** (`.yar` / `.yara`) — not
the rules themselves. You write the rules in a file first, then point the dialog
at it.

1. Create a rules file (see the example below) and save it, e.g.
   `C:\rules\malware.yar` or `~/rules/strings.yara`.
2. Open the document you want to scan, then run **Code → YARA Scan…** (it is
   also in the command palette).
3. Enter the full path to the rules file and press Enter.
4. Matches are listed in the results panel as `rule  $string  line N @ offset`.
   Click a row (or press F4) to jump to that line.

The command works on a normal text / code tab (not on large-file, hex or Git
tabs).

### Example rules file

```yara
rule suspicious_strings
{
    strings:
        $a  = "cmd.exe /c" nocase
        $b  = "powershell" nocase wide
        $mz = { 4D 5A ?? 00 }              // PE header (?? is a wildcard byte)
        $url = /https?:\/\/[a-z0-9.\-]+/ nocase
    condition:
        $mz and (2 of ($a, $b, $url))
}
```

### What is supported

A practical subset of the YARA language:

- **Strings** — text (`"..."`), hex (`{ 4D 5A ?? 00 }`, with `??` byte and `?`
  nibble wildcards) and regular expressions (`/regex/`).
- **String modifiers** — `nocase`, `wide`, `ascii`, `fullword`.
- **Conditions** — a string name (`$a`), `all of them` / `any of them` /
  `N of them`, `N of ($a*)`, `#a <op> N` (match count), `$a at N` (match at an
  offset), `filesize <op> N`, `and` / `or` / `not`, parentheses, and `//` …
  `/* … */` comments.

It is **not** the full libyara engine: modules (`pe`, `hash`, `math`, …), hex
jumps (`[4-6]`) and alternation (`( A | B )`) are not supported, and a rule that
uses them reports a typed error instead of matching.

!!! tip "Inline rules / batch scanning"
    To paste rules directly, or to scan a whole file or directory at once, use
    the `yara_scan` tool over [MCP](mcp.md#tools) — it accepts the rules as text
    and a file, a directory or inline bytes as the target.

## Transform recipes

**Code → Apply Convert Recipe…** runs a pipeline of transforms over the
selection (or the whole document), left to right. Steps are separated by `|`
(or spaces / commas); a step argument follows a colon. For example:

```
from_base64 | xor:cafe | gunzip
```

Besides every converter in [File conversion](convert.md) (base64, gzip, URL,
JWT decode, …), the recipe adds the primitives `xor:<key>`, `rot:<n>` / `rot13`,
`not`, `reverse`, `rc4:<key>`, `base58.encode` / `base58.decode`,
`base85.encode` / `base85.decode`, and `from_hex` / `to_hex`. A key argument may
be `hex:cafe` or `str:secret` (a bare value is treated as hex when it looks like
hex).

## Entropy

In the [Hex / Binary editor](hex.md), the entropy map and the entropy overview
strip show the Shannon entropy of the file per region (0–8 bits/byte), so packed
or encrypted areas stand out.

## Scan for Secrets

**Code → Scan for Secrets** lists likely secrets (API keys, tokens, private
keys, credentials) in the document, each with its line; a row jumps to it.
**Edit → Redact Selection** masks the selected text.
