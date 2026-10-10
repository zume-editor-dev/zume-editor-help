# Security analysis

Zume Editor has tools for binary triage, data transforms and signature
scanning — useful for reverse engineering, CTFs and authorized security work.
Each is also available headlessly to AI agents through the
[MCP server](mcp.md#tools) (the `entropy`, `transform` and `yara_scan` tools).

## YARA Scan

**Code → YARA Scan…** scans the active document with a set of YARA rules.

1. Run the command (it is also in the command palette).
2. Enter the path to a `.yar` / `.yara` rules file.
3. Matches are listed in the results panel as `rule  $string  line N @ offset`.
   Click a row (or press F4) to jump to that line.

A practical subset of the YARA language is supported: text, hex
(`{ 4D 5A ?? 00 }`, with `??` / `?` wildcards) and `/regex/` strings with the
`nocase` / `wide` / `ascii` / `fullword` modifiers, and conditions using
`all`/`any`/`N of them`, `N of ($a*)`, `#count`, `$s at N`, `filesize`, and
`and` / `or` / `not`. It is not the full libyara engine (no `pe` / `hash` /
`math` modules).

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
