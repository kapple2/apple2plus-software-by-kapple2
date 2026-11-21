# KBASIC versions

<details>
<summary>KBASIC versions (click to unfold; folded by default)</summary>

| KBASIC | Date | K-HANGEUL driver (MD5) | Changes |
|---|---|---|---|
| v1.4 | 2026-11-17 | v1.10 (923db28292c06d72db0d08d153faaf6f), DIST-github `main` ca2ca4a | Driver v1.10: the Korean DOS names LOAD = `받기` and BLOAD = `기받기` (v1.9's `부르기` and `기부르기` clashed with KBASIC's CALL word `부르기`; do not use v1.9). KBASIC code unchanged. KBASIC.web and emulator.js (KAPPLE ][) added. |
| v1.3 | 2026-11-17 | v1.8 (b07dfc3e2965783c2de8cad474a69542), DIST-github `main` 96647f3 | Driver fixes a bug in v1.6/v1.7 where the output of SPC() and TAB() was swallowed (`PRINT "C";SPC(5);"D"` printed CD). KBASIC code unchanged. KBASIC.js (browser port) now prints the same Korean error messages, including its own (graphics, paddles, DOS). v1.2 should not be used. |
| v1.2 | 2026-11-17 | v1.7 (2730f3b0b9313b7304945be1ed2efdf2), DIST-github `main` 8b5b6d7 | Applesoft error messages in Korean (driver feature, both cards): `?줄없음 오류`, also BREAK (`중단 줄 10`), ?EXTRA IGNORED and ?REENTER; DOS and Applesoft error texts are shorter and have no spaces, so programs that read that text see the new wording (tables in the driver's docs/MANUAL.md sections 8, 8a, 8b). Driver v1.7: INIT (포맷) formats on the Saturn card too; Saturn Korean messages no longer pass through COUT. Driver no longer reprints its banner on HAN/Ctrl-Reset/INIT; main RAM `$1CC0-$1E50` is the error filter's (plain card). KBASIC code unchanged. |
| v1.1 | 2026-11-16 | v1.3 (9458c28fabda0e84f1b49144ff4e6160), uncommitted in the driver repo at that time | Version number now kept and raised on every change. Driver brings line editing like stock Apple II (left arrow then Return gives the text left of the cursor, Ctrl-X cancels) and, since driver 58b7b68, the Saturn backspace-then-Return fix. KBASIC code itself unchanged. |
| v1.0 | before 2026-11-16 | 1.2b | First numbered version; plain 16K card and Saturn 128K card. |

</details>

## KBASIC Editor (its own version; the full history is in `src/kbasic_editor/RELEASE.txt`)

<details>
<summary>KBASIC Editor versions (click to unfold; folded by default)</summary>

| Editor | Date | Changes |
|---|---|---|
| v1.02 | 2026-11-17 | Korean DOS command names (받기, 제거, 기저장, ...) are coloured as DOS words when a string starts with one. KBASIC itself is unchanged (v1.4). |
| v1.01 | 2026-11-17 | File > Export (tokenized Applesoft file, or straight into a DOS 3.3 `.dsk`) and File > Import (back to Korean-keyword source), with checks; `--export` / `--import` without a window. KBASIC itself is unchanged (then v1.1). |
| v1.00 | 2026-11-16 | First numbered editor: colours, tabs, theme (follow system / light / dark), word wrap, About box. |

</details>
