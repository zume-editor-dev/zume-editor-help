# Security analysis

Zume Editor has tools for binary triage, data transforms and signature
scanning — useful for reverse engineering, CTFs and authorized security work.
Each is also available headlessly to AI agents through the
[MCP server](mcp.md#tools) (the `entropy`, `transform`, `yara_scan`,
`pe_info`, `elf_info` and `macho_info` tools).

## Executable Info (PE / ELF / Mach-O)

**Code → Executable Info…** reads an executable structurally and lists the
result in the results panel. The format is detected from the file's magic:

- **PE** (Windows EXE / DLL): the headers and security flags (ASLR / DEP /
  CFG), every section with its entropy, the imported DLLs and functions, the
  exports, and the resources a feature inventory needs — the version strings,
  the string tables, each dialog's caption and controls, the menu items and the
  manifest — plus the PDB path and any data appended past the last section.
- **ELF** (Linux executables and shared objects): type, machine, entry, the
  interpreter, segments and sections with entropy, the dynamic section (needed
  libraries, soname, rpath / runpath), imported and exported symbols with their
  versions, the local symbols when not stripped, the GNU build id, and the
  hardening the binary carries (PIE, NX stack, RELRO, stack canary, FORTIFY).
- **Mach-O** (macOS / iOS images, including fat / universal binaries, whose
  slices are listed and the first one read): CPU, file type, header flags,
  every load command, segments and sections with entropy, linked dylibs with
  versions, rpaths, UUID, platform / minimum OS / SDK, the entry point,
  imported and exported symbols, and whether the image is code-signed or
  encrypted.

When the active tab is itself such a file (a binary opened in the hex view),
that file is read; otherwise a file picker asks for one. The file is read on
a worker, so a large binary does not block the editor (the first 256 MiB are
read; a larger file says so in a warning row). Click a row (or press **F4**)
to copy that line to the clipboard; **Esc** closes the panel.

Rows that name code jump to the disassembly instead: the entry-point line,
an exported function, and an executable section or segment. Clicking one
opens the file in the hex view at that file offset and runs
**Hex → Disassemble** there with the architecture the file declares (x64,
x86, ARM64, ARM / Thumb), so the disassembly panel takes the report's place
from that offset; the hex view stays open for browsing.

![PE Info in the results panel](assets/screenshots/pe_info_panel.png){ loading=lazy }

### Disassemble Executable

**Code → Disassemble Executable…** goes straight to the listing. It takes
the active tab's file when it is a PE / ELF / Mach-O (otherwise a file picker
asks), reads the headers, and opens a small dialog asking where to start,
prefilled with the entry point's file offset. Type a file offset (`0x2D5F0`
or decimal) or the name of an exported function (`GetProcAddress`; a Mach-O
name works with or without its leading `_`) and press **Enter**: the file
opens in the hex view at that offset and the disassembly panel lists the
instructions from there, decoded as the architecture the file declares. A
name that is not an export, or an offset past the file, is refused with a
message. The same decoder is the MCP tool `disassemble`. Click into the
results panel and the keyboard walks it: **↑ / ↓**, **PageUp / PageDown**,
**Home / End** move the highlighted row, **Enter** activates it (the hex
caret moves to that instruction), **Esc** closes the panel.

![The start dialog over kernel32, an export name typed](assets/screenshots/disasm_prompt.png){ loading=lazy }

![The listing from GetProcAddress in the hex view](assets/screenshots/disasm_listing.png){ loading=lazy }

### Cross-References

**Code → Cross-References…** answers "which code uses this": it takes the
active tab's executable (or asks for one), opens a small dialog for the
target - an import or symbol name (`CreateFileW`, `KERNEL32.dll!CreateFileW`),
a fragment of a string, a `0x` address, or nothing for every reference - and
scans every executable section. The results panel lists the targets by
reference count (import / symbol / PLT or stub / string / code / data) and
then every referencing instruction with its address; clicking a reference
row (or pressing **F4**) opens the file in the hex view at that instruction
and disassembles from there. The same scan is the MCP tool `xrefs`.

![Cross-references of kernel32 to GetProcAddress](assets/screenshots/xrefs_panel.png){ loading=lazy }

The same readers are the MCP tools `pe_info`, `elf_info` and `macho_info`
(see [MCP](mcp.md#tools)). They never load or run the file; `disassemble`
decodes bytes and nothing more. Analyze only binaries whose licence permits
it.

### Decompiler tools — installation

Decompilation is **optional** and uses tools you install yourself. Zume bundles
none of them, downloads nothing, and launches a tool only when you point it at
it. Install just the ones you need:

| Tool | Decompiles | Install | Enable |
| --- | --- | --- | --- |
| **JDK** (Temurin / OpenJDK) | prerequisite for Java **and** Ghidra | [adoptium.net](https://adoptium.net/) — Java 17+ | set `JAVA_HOME`, or put `java` / `javap` on `PATH` |
| **Ghidra** | native **PE / ELF / Mach-O → C** | [ghidra-sre.org](https://ghidra-sre.org/) — unzip anywhere (needs a JDK) | editor: *Tools → Settings → Decompiler (Ghidra)*; MCP: `ZUME_GHIDRA_HOME` = the install dir |
| **ilspycmd** (ILSpy) | **.NET assembly → C# / IL** | `dotnet tool install -g ilspycmd` (needs the .NET runtime) | editor: *Tools → Settings → Decompiler (.NET / ILSpy)*; MCP: `ZUME_ILSPY` = the `ilspycmd` path, or `1` to find it on `PATH` |
| **CFR** | **Java `.class` / `.jar` → Java source** | download [https://repo1.maven.org/maven2/org/benf/cfr/0.152/cfr-0.152.jar](https://repo1.maven.org/maven2/org/benf/cfr/0.152/cfr-0.152.jar) from Maven Central (a single jar; needs a JDK) | MCP: `ZUME_CFR` = the jar path |

Notes:

- **javap** (JVM bytecode) and **jar** (class listing) ship with the JDK, so Java
  class listing and bytecode work with the JDK alone — CFR adds Java **source**.
- The `ZUME_*` variables are read from the **MCP server's** environment. For the
  Claude Code CLI that is the `env` block of the `zume` entry in `~/.claude.json`;
  restart the client after editing it. Optional result caches:
  `ZUME_GHIDRA_CACHE`, `ZUME_ILSPY_CACHE`, `ZUME_CFR_CACHE`.
- Ghidra and ilspycmd are also configurable from the editor's **Settings** dialog
  (they drive the editor's own *Code → Decompile…* commands); the `ZUME_*`
  variables drive the MCP tools used from Claude Code. The two are independent.
- Every tool **reads the target statically — it never runs it.** Licences differ:
  Ghidra is Apache-2.0, ilspycmd and CFR are MIT; none is redistributed by Zume.
  Decompiler output is derivative — see the per-tool sections below and their
  licence / clean-room warnings before you reuse anything.

### Decompile (Ghidra)

Zume does not include a decompiler, but it can drive one you already have:
**Code → Decompile (Ghidra)…** turns a function's machine code back into
C-like pseudocode using a local [Ghidra](https://ghidra-sre.org/) install.
It is **optional and off by default** - the menu item is greyed out until you
point Zume at Ghidra.

**Enabling it.** Open **Tools → Settings…**, find the *Decompiler (Ghidra)*
section, click the *Ghidra install:* row and choose your Ghidra directory (the
folder that contains `support/analyzeHeadless`). Ghidra needs a Java 17+
runtime; Zume finds one on your `PATH` or `JAVA_HOME`, so no extra setup is
usually needed. Clearing the path disables the feature again.

**Using it.** With the active tab an executable (or via the file picker),
choose **Code → Decompile (Ghidra)…**, then enter a function - an address
(`0x180001020`, e.g. one `functions` or `xrefs` reported), a name, or nothing
for the whole file (capped). Zume runs Ghidra headlessly on a background
thread (this can take a few minutes the first time) and opens the decompiled C
in a **read-only** tab. Results are **cached** by the file's size, modification
time and your request, so decompiling the same unchanged function again is
instant.

The same decompiler is the MCP tool `decompile`, which is off unless the
`ZUME_GHIDRA_HOME` environment variable points at a Ghidra install (set
`ZUME_GHIDRA_CACHE` to choose its cache directory).

!!! warning "Licence and clean-room"
    Ghidra statically analyses the binary - it does not run it - but
    decompiler output is **derivative** of the original and can resemble its
    source. Decompile only binaries whose licence permits it, and if you
    re-implement a feature, work from a behavioural specification, not from
    the emitted C.

### Decompile .NET (ILSpy)

A .NET assembly's real code is CIL in its metadata, so Ghidra cannot usefully
decompile it. For **managed** binaries Zume drives
[ILSpy](https://github.com/icsharpcode/ILSpy)'s command-line tool **ilspycmd**
instead: **Code → Decompile (.NET / ILSpy)…** turns a type back into C#. Like
the Ghidra integration it is **optional and off by default** - the menu item is
greyed out until you point Zume at `ilspycmd`.

**Installing ilspycmd.** It is a .NET global tool (the ILSpy desktop app is *not*
needed):

```
dotnet tool install -g ilspycmd
```

This needs the .NET runtime and installs `ilspycmd` (typically
`%USERPROFILE%\.dotnet\tools\ilspycmd.exe`), also putting it on your `PATH`.

**Enabling it.** Open **Tools → Settings…**, find the *Decompiler (.NET / ILSpy)*
section, click the *ilspycmd:* row and choose the `ilspycmd.exe` file. The
optional *IL cache:* row chooses where results are cached (leave it unset to use
a default location; the folder is created for you on first use). Clearing the
path disables the feature again.

**Using it.** With the active tab a .NET assembly (or via the file picker),
choose **Code → Decompile (.NET / ILSpy)…** and enter a fully qualified type
name (e.g. `Acme.Core.Engine`); leave it empty to list the assembly's types into
a tab first, then re-run with one of them. Zume runs ilspycmd on a background
thread and opens the decompiled C# in a **read-only** tab. Results are cached
like the Ghidra ones.

The same decompiler is available to MCP clients as `il_types` (the type
inventory, the managed counterpart of `functions`) and `il_decompile` (one type
to C#, or the whole assembly as a project), off unless the `ZUME_ILSPY`
environment variable is set - the path of `ilspycmd`, or `1` to find it on
`PATH` (`ZUME_ILSPY_CACHE` chooses the cache directory). The `decompile` tool
routes a managed target here automatically.

!!! warning "Licence and clean-room"
    Decompiled C# is **very close to the original source** (the IL keeps names
    and structure), so the licence / clean-room caution matters even more than
    for native code: decompile only assemblies whose licence permits it, and
    re-implement from a behavioural specification, not from the emitted C#.

### Decompile Java (javap + CFR)

Java `.class` / `.jar` files are JVM bytecode - neither Ghidra nor ILSpy is the
right tool. Zume uses the JDK's **javap** (bytecode) and a user-provided **CFR**
decompiler (Java source) instead, through the MCP tools `java_types` (list a
jar's classes) and `java_decompile` (one class to Java source, or `javap -c`
bytecode, or a whole jar to a source tree). It is **opt-in**:

- **javap / jar** come from the JDK, found via `JAVA_HOME` or `PATH` - the same
  JDK the Ghidra integration needs. Bytecode and class listing work with the JDK
  alone.
- **CFR** (Java-source decompilation) is a single jar you provide; set the
  `ZUME_CFR` environment variable (on the MCP server) to its path. `ZUME_CFR_CACHE`
  chooses the cache directory. CFR is MIT-licensed and, like Ghidra and ilspycmd,
  is not bundled, fetched or linked - Zume only launches the tool you installed.

**Installing CFR.** It is a single jar - there is nothing to install, just
download it once from Maven Central (or the [CFR releases](https://github.com/leibnitz27/cfr/releases)):

```
https://repo1.maven.org/maven2/org/benf/cfr/0.152/cfr-0.152.jar
```

Save it anywhere, e.g. `C:\Users\<you>\zume-app\cfr\cfr-0.152.jar`. A JDK must
also be present for `java` (to run CFR) and `javap` (bytecode) - the same JDK the
Ghidra integration uses.

**Enabling it.** Set these on the environment of the **MCP server** (for the CLI,
the `env` block of the `zume` entry in `~/.claude.json`), then restart the client:

```json
"env": {
  "JAVA_HOME": "C:\\Program Files\\Eclipse Adoptium\\jdk-25...-hotspot",
  "ZUME_CFR": "C:\\Users\\<you>\\zume-app\\cfr\\cfr-0.152.jar",
  "ZUME_CFR_CACHE": "C:\\Users\\<you>\\zume-app\\cfr-cache"
}
```

`JAVA_HOME` can be omitted if `java` / `javap` are already on `PATH`; class
listing and bytecode (`javap`) then work with the JDK alone, and `ZUME_CFR` adds
Java-source output (CFR). Non-ASCII string literals come back as `\uXXXX`
escapes, which is CFR's normal, lossless output.

The tools read the class files - they never run them.

!!! warning "Licence and clean-room"
    Decompiled Java is **derivative** of the original. Decompile only software
    whose licence / EULA permits reverse engineering and that you have the right
    to analyse, and re-implement from a behavioural specification, not by copying
    the emitted Java. Anything you build this way is your own responsibility.

## Interface recovery (Objective-C / Swift)

**Code → Objective-C / Swift Interface…** recovers the class interfaces a
macOS / iOS **Mach-O** carries in its Objective-C and Swift runtime metadata —
without a decompiler. It reads the `__objc_*` and `__swift5_*` sections and
writes a header to `<app data>/interfaces/<name>.h`, opened read-only, listing:

- **Objective-C** classes with their superclass and adopted protocols, every
  instance and class method (`-` / `+` with the selector, and the argument and
  return types where the metadata carries them), the instance variables, the
  declared properties with their attributes, and the categories that extend a
  class.
- **Swift** types recovered from `__swift5_types` / `__swift5_protos`, and the
  Swift symbols of the symbol table demangled to readable signatures
  (`Module.Type.method(label:) async throws -> T`). The demangler is
  **fail-closed**: a symbol is demangled correctly or left in its raw `_$s…`
  form — never guessed wrong.

When the active tab is such a file it is read; otherwise a file picker asks.
A fat / universal binary resolves to the slice read, and pointing the command
at a `.app` or `.framework` bundle resolves to the Mach-O inside it. Each
method row carries the address of its implementation, so activating it (click,
or **F4**) jumps to that method's body in the disassembly, like the Executable
Info code rows. The same engine names call sites in **Cross-References** and in
the MCP `functions` tool — an `objc_msgSend` site shows the `@selector(...)` it
sends, and a class reference shows the class.

This reads only the metadata Objective-C and Swift leave in the binary for
their own runtimes, not the method bodies; it never runs the target. It is most
useful on macOS / iOS binaries, where this metadata is richest.

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

The dialog that opens takes the recipe in its single text field; the hint
below the field lists every step the engine accepts, grouped as in the table
below, so you do not have to remember them. Type the steps, then press
**Enter** (or OK) to apply; **Esc** cancels.

![The transform-recipe dialog with its step list](assets/screenshots/convert_recipe_dialog.png){ loading=lazy }

### Every step

| Group | Steps |
| --- | --- |
| With an argument | `xor:<key>` (XOR every byte with the repeating key), `rc4:<key>` (RC4 encrypt / decrypt), `rot:<n>` (Caesar shift by *n*) |
| No argument | `from_hex`, `to_hex`, `rot13`, `not` (invert every bit), `reverse` (reverse the bytes), `base58.encode` / `base58.decode`, `base85.encode` / `base85.decode` |
| Short names | `from_base64` = `base64.decode`, `to_base64` = `base64.encode`, `from_base32` / `to_base32`, `gunzip` = `gzip.decompress`, `gzip` = `gzip.compress`, `url_decode` / `url_encode` |
| Encode | `base64.encode`, `base64url.encode`, `url.encode`, `qp.encode`, `html.encode`, `unicode.escape`, `rot13`, `base32.encode` |
| Decode | `base64.decode`, `base64url.decode`, `url.decode`, `qp.decode`, `html.decode`, `unicode.unescape`, `jwt.decode`, `base32.decode` |
| Data | `timestamp.unixToIso`, `timestamp.unixMsToIso`, `timestamp.filetimeToIso`, `timestamp.isoToUnix`, `csv.toJson`, `json.toCsv`, `csv.toTsv`, `tsv.toCsv`, `csv.toMarkdown`, `csv.toHtml`, `csv.toSql`, `json.toYaml`, `yaml.toJson`, `json.toToml`, `toml.toJson`, `json.toIni`, `ini.toJson`, `json.minify`, `json.pretty`, `html.toMarkdown` |
| Compress | `gzip.compress`, `gzip.decompress`, `zlib.compress`, `zlib.decompress`, `bzip2.compress`, `bzip2.decompress` |

The Encode / Decode / Data / Compress rows are the converters of
[File conversion](convert.md); any converter id works as a step. A key argument
(`xor:` / `rc4:`) is read as hex bytes when it looks like hex (`cafe`,
`deadbeef`) and as text otherwise; write `hex:cafe` or `str:cafe` to say which
explicitly.

!!! warning "A text key that happens to look like hex"
    Because a bare key is read as **hex whenever it is all hex digits**, a text
    password such as `cafe`, `decade` or `7a756d65` is taken as *bytes*, not
    letters — so `rc4:zume` uses the text key `zume` (there are non-hex letters),
    but `rc4:cafe` does **not** use the text `cafe`. Remove the ambiguity: write
    `rc4:str:<key>` for a text/ASCII key (e.g. `rc4:str:zume`) and
    `rc4:hex:<bytes>` — or just the bare hex — for a byte key (e.g.
    `rc4:7a756d65`, the hex of `zume`). The two forms give different keystreams,
    so matching a key check in a binary (which uses the raw key bytes) means
    using the hex form of those exact bytes.

!!! tip "Make it an error instead of a guess (strict mode)"
    To forbid the bare form outright, turn on strict mode in **Tools ▸
    Settings…**, under **Transform recipe keys** (the `strict_transform_keys`
    setting). Every `xor:`/`rc4:` key must
    then carry `hex:` or `str:`; a bare key is refused and the recipe is not
    applied. With strict mode off (the default), a bare key that looks like hex
    still applies, but the status line warns and names the `str:`/`hex:` forms.
    The `transform` MCP tool mirrors this: its response always carries a
    `warnings` list that flags such a bare key, and a `strict: true` argument
    refuses one.

### Examples

Select the encoded text (or run it on the whole document) and apply:

| Recipe | What it does |
| --- | --- |
| `from_base64` | Decode a Base64 blob |
| `from_base64 \| gunzip` | Decode Base64, then gzip-decompress the result |
| `from_hex \| xor:5a` | Hex-decode, then XOR every byte with `0x5A` |
| `from_hex \| xor:hex:deadbeef` | Hex-decode, then XOR with the repeating key `DE AD BE EF` |
| `from_base64 \| rc4:str:s3cret` | Base64-decode, then RC4-decrypt with the text key `s3cret` |
| `from_hex \| rc4:7a756d65` | Hex-decode, then RC4 with the byte key `7A 75 6D 65` (the hex of `zume`) |
| `rot13` | ROT13; `rot:3` is a Caesar shift of 3 |
| `base58.decode` | Decode a Base58 string (e.g. a wallet address / key) |
| `jwt.decode` | Expand a JWT into its decoded header / payload JSON |
| `from_base64 \| xor:cafe \| from_base64` | Peel a doubly-encoded, XOR-wrapped payload |

The result replaces the selection (or the document) as one undo step, so
`Ctrl+Z` reverts the whole recipe. If a step fails (e.g. input that is not valid
Base64), the editor shows which step failed and leaves the text unchanged.

## Entropy

In the [Hex / Binary editor](hex.md), the entropy map and the entropy overview
strip show the Shannon entropy of the file per region (0–8 bits/byte), so packed
or encrypted areas stand out.

## Scan for Secrets

**Code → Scan for Secrets** lists likely secrets (API keys, tokens, private
keys, credentials) in the document, each with its line; a row jumps to it.
**Edit → Redact Selection** masks the selected text.
