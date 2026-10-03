# Encodings & Line Endings

## Encodings

Zume Editor detects the encoding on open and shows it in the status bar. Click the
encoding there to convert, or use **Document → Set Encoding**.

Supported encodings include:

- **Unicode** - UTF-8, UTF-8 with BOM, UTF-16 LE/BE, UTF-32 LE/BE.
- **Japanese** - Shift-JIS (CP932), EUC-JP.
- **CJK** - GBK, Big5, EUC-KR.
- **Western / Cyrillic / Greek / Turkish / Hebrew / Arabic / Baltic / Thai /
  Vietnamese** code pages (ISO-8859-*, Windows-125x, and more).

If a character cannot be represented in the chosen encoding, the app warns you
before saving rather than losing it silently.

## Line endings

- Detects and shows **LF**, **CRLF** or **CR**; a mixed file is reported as mixed.
- Convert with **Document → Convert Line Endings**. New lines you type use the
  file's own ending.
