# KBASIC: Korean Applesoft keywords on K-HANGEUL

> 한국어: [KBASIC.md](KBASIC.md). Reference tables (Korean): [REFERENCE.txt](REFERENCE.txt)

KBASIC lets you type and list Applesoft BASIC with Korean keywords (`출력`, `만약`, `반복` ...). The program in memory and on disk is ordinary Applesoft: every keyword is still the single-byte token, so a program saved with Korean keywords loads and runs on any Applesoft, and an English program lists in Korean.

## What you need
- The K-HANGEUL driver, v1.1b or later (it has the two small KBASIC hook calls). `src/dsk/KBASIC.DSK` already contains it.
- `KBASIC` (2.5 KB binary at `$8A00`, 12 sectors on the disk).

## Use
`KBASIC.DSK` loads everything at boot: `HELLO` loads the fonts, then runs the text file `KSTART` (`BRUN K-HANGEUL`, `BRUN KBASIC`). You should see **`KBASIC V1.4 OK`** (the KBASIC version) above the first `]` prompt, followed by a memory report like the driver's start-up banner (free memory, `HIMEM`, `LOMEM`). If you do not see it, KBASIC is not active and `LIST` will show English keywords.

On another disk, copy `K-HANGEUL` (v1.1b or later) and `KBASIC`, and run:
```
BRUN KBASIC          after the driver (not together with TYPE / STYPE; see limits)
```
`BRUN KBASIC` prints `KBASIC V1.4 OK`, or `KBASIC: NEEDS THE KBASIC.DSK DRIVER` when no K-HANGEUL driver is found (then nothing works: use the driver on `KBASIC.DSK`). The same KBASIC runs with a plain 16K language card and with a Saturn 128K card: install looks at the driver's COUT entry in main RAM (`$940A`: `8D` on a plain card, `20` on a Saturn card). The memory report is printed with Applesoft ROM routines, not the driver's banner code (which is not resident on a Saturn card).
Then type Hangul in Hangul mode (Ctrl-K ... Ctrl-A) as usual:

```
]10 만약 1 그러면 출력 "OK"        (Ctrl-K, keystrokes, Ctrl-A, ...)
]LIST
10  만약 1 그러면  출력 "OK"
```

- A Korean keyword is the **whole token**. Where the English token ends in `=`, `:`, `#` or `$`, do not type it: `색상 3` is `COLOR= 3`, `상한 8192` is `HIMEM: 8192`, `문자(65)` is `CHR$(65)`. A token that ends in `(` (`탭이동`, `빈칸`, `스크린`) is followed by the `(` you type.
- English keywords still work, and mixing is fine.
- `REM` is `#` (at the start of a statement; `PR#` / `IN#` are not affected).
- `TO` is `부터`; `까지` and `ON`'s second word `조건` are accepted when typing (aliases); `LIST` prints `부터` and `분기`.
- Hangul variable names work (`점검$ = "KB"`): Ctrl-K and Ctrl-A are skipped while a program runs. Applesoft only looks at the first two characters of a name, so a Hangul name is told apart by about its first syllable. A name must not start with a keyword (the keyword wins, as in Applesoft: `FORTUNE`).
- Quotes, `REM` text and `DATA` text are never converted.
- DOS commands (`CATALOG`, `LOAD`, `SAVE`, `BRUN` ...) stay English. `LOAD` and `SAVE` have no Korean word.

The full word list is the keyword table below; the data file is `src/kbasic/keywords.tsv`.

## Typing Hangul
Korean keywords are typed like any Hangul in K-HANGEUL. **Ctrl-K** enters Hangul mode and **Ctrl-A** leaves it. Between them, press the jamo keys of the **2-set (dubeolsik)** layout (upper-case mode, the Apple II+ default). Key positions match a real Korean keyboard:

```
 Q  W  E  R  T  Y  U  I  O  P        ㅂ ㅈ ㄷ ㄱ ㅅ ㅛ ㅕ ㅑ ㅐ ㅔ
  A  S  D  F  G  H  J  K  L           ㅁ ㄴ ㅇ ㄹ ㅎ ㅗ ㅓ ㅏ ㅣ
   Z  X  C  V  B  N  M                ㅋ ㅌ ㅊ ㅍ ㅠ ㅜ ㅡ
```

- **Tense consonants and ㅒ ㅖ:** ㄲ ㄸ ㅃ ㅆ ㅉ ㅒ ㅖ are the keys `-` `=` `*` `<` `>` `;` `+`. On the Apple II+ that is `-` (ㄲ), Shift+`-` (ㄸ), Shift+`:` (ㅃ), Shift+`,` (ㅆ), Shift+`.` (ㅉ), `;` (ㅒ), Shift+`;` (ㅖ).
- **Apple IIe, IIc, IIgs:** Shift+`-` gives `_` and Shift+`;` gives `:`; in Hangul mode the driver turns `_` into ㄸ and `:` into ㅖ. ㅃ is Shift+8 (`*`). Also, hold **Open-Apple** (Left Alt in AppleWin) and press R E Q T W O P to get ㄲ ㄸ ㅃ ㅆ ㅉ ㅒ ㅖ. Software cannot read the Shift key on these machines.
- Compound vowels and final clusters are typed as consecutive keys (ㅘ = `H K`, ㄳ = `R T`).
- To type digits, spaces, symbols or English after a keyword, leave Hangul mode first with Ctrl-A. Re-enter with Ctrl-K for a Hangul string inside quotes.

Some keywords as keystrokes:

| Keyword | Keys |
|---|---|
| 출력 | Ctrl-K `C N F F U R` Ctrl-A |
| 만약 | Ctrl-K `A K S D I R` Ctrl-A |
| 그러면 | Ctrl-K `R M F J A U S` Ctrl-A |
| 반복 | Ctrl-K `Q K S Q H R` Ctrl-A |
| 한글 | Ctrl-K `G K S R M F` Ctrl-A |

The left-arrow (backspace) key erases. The full key chart and the editing keys are in section 5 of the K-HANGEUL manual (`docs/MANUAL_en.md`).

## Keyword table
(generated from `src/kbasic/keywords.tsv`)

<!-- KEYWORDS -->
The Korean word replaces the whole English token. An alias is accepted when typing only; `LIST` prints the first word.

| Applesoft token | Korean | Note |
|---|---|---|
| `END` | 그만 |  |
| `STOP` | 멈춤 |  |
| `CONT` | 지속 |  |
| `RUN` | 실행 |  |
| `NEW` | 새로 |  |
| `LIST` | 코드 |  |
| `CLEAR` | 초기화 |  |
| `LET` | 넣기 |  |
| `GOTO` | 가기 |  |
| `GOSUB` | 호출 |  |
| `RETURN` | 돌아 |  |
| `POP` | 추출 |  |
| `ON` | 분기 |  |
| `IF` | 만약 |  |
| `THEN` | 그러면 |  |
| `FOR` | 반복 |  |
| `TO` | 부터 |  |
| `STEP` | 간격 |  |
| `NEXT` | 다음 |  |
| `DEF` | 함수 |  |
| `FN` | 기능 |  |
| `POKE` | 램넣기 |  |
| `WAIT` | 대기 |  |
| `ONERR` | 오류 |  |
| `RESUME` | 재개 |  |
| `PRINT` | 출력 |  |
| `INPUT` | 입력 |  |
| `GET` | 키받기 |  |
| `READ` | 읽기 |  |
| `DATA` | 자료 |  |
| `RESTORE` | 처음 |  |
| `DIM` | 배열 |  |
| `HOME` | 비움 |  |
| `TEXT` | 텍스트 |  |
| `NORMAL` | 보통 |  |
| `INVERSE` | 반전 |  |
| `FLASH` | 점멸 |  |
| `SPEED=` | 속도 | the `=`/`:`/`#`/`$` is part of the word: do not type it |
| `TAB(` | 탭이동 | you type the `(` yourself |
| `HTAB` | 가로탭 |  |
| `VTAB` | 세로탭 |  |
| `SPC(` | 빈칸 | you type the `(` yourself |
| `PR#` | 보내기 | the `=`/`:`/`#`/`$` is part of the word: do not type it |
| `IN#` | 들이기 | the `=`/`:`/`#`/`$` is part of the word: do not type it |
| `TRACE` | 추적시작 |  |
| `NOTRACE` | 추적중지 |  |
| `HIMEM:` | 상한 | the `=`/`:`/`#`/`$` is part of the word: do not type it |
| `LOMEM:` | 하한 | the `=`/`:`/`#`/`$` is part of the word: do not type it |
| `GR` | 저해상 |  |
| `COLOR=` | 색상 | the `=`/`:`/`#`/`$` is part of the word: do not type it |
| `PLOT` | 점표시 |  |
| `HLIN` | 가로선 |  |
| `VLIN` | 세로선 |  |
| `HGR` | 고해상1 |  |
| `HGR2` | 고해상2 |  |
| `HCOLOR=` | 고색상 | the `=`/`:`/`#`/`$` is part of the word: do not type it |
| `HPLOT` | 선긋기 |  |
| `DRAW` | 그려 |  |
| `XDRAW` | 엑스그려 |  |
| `SCALE=` | 크기 | the `=`/`:`/`#`/`$` is part of the word: do not type it |
| `ROT=` | 회전 | the `=`/`:`/`#`/`$` is part of the word: do not type it |
| `AT` | 에서 |  |
| `ABS` | 절대수 |  |
| `SGN` | 부호 |  |
| `INT` | 정수 |  |
| `SQR` | 루트 |  |
| `SIN` | 사인 |  |
| `COS` | 코사인 |  |
| `TAN` | 탄젠트 |  |
| `ATN` | 아크탄 |  |
| `EXP` | 지수 |  |
| `LOG` | 로그 |  |
| `RND` | 난수 |  |
| `CHR$` | 문자 | the `=`/`:`/`#`/`$` is part of the word: do not type it |
| `ASC` | 아스키 |  |
| `STR$` | 글자로 | the `=`/`:`/`#`/`$` is part of the word: do not type it |
| `VAL` | 글자값 |  |
| `LEN` | 길이 |  |
| `LEFT$` | 좌문 | the `=`/`:`/`#`/`$` is part of the word: do not type it |
| `RIGHT$` | 우문 | the `=`/`:`/`#`/`$` is part of the word: do not type it |
| `MID$` | 중간문 | the `=`/`:`/`#`/`$` is part of the word: do not type it |
| `FRE` | 잔고 |  |
| `POS` | 위치 |  |
| `PEEK` | 램보기 |  |
| `PDL` | 패들 |  |
| `SCRN(` | 스크린 | you type the `(` yourself |
| `USR` | 어셈 |  |
| `AND` | 그리고 |  |
| `OR` | 혹은 |  |
| `NOT` | 아닌 |  |
| `CALL` | 부르기 |  |
| `DEL` | 삭제 |  |
| `STORE` | 테이프저장 |  |
| `RECALL` | 테이프읽기 |  |
| `SHLOAD` | 도형읽기 |  |
| `REM` | # |  |
| `TO` | 까지 | alias (typing only) |
| `ON` | 조건 | alias (typing only) |

Stay English: `+ - * / ^ > = <`, `&`, `LOAD`, `SAVE` and all DOS commands.
<!-- /KEYWORDS -->

## How it works
- **Typing:** at the end of a typed line the driver calls `KB_IN`. It copies each Hangul run that is a keyword as the English letters (`출력` becomes `PRINT`), so Applesoft's own tokenizer makes the token. Other Hangul runs are left alone. Spaces, digits and symbols are moved out of the Hangul run; a lone `Ctrl-A` is dropped.
- **LIST:** K-HANGEUL lists with its own routine in driver RAM, which prints the letters of a keyword with one call (return address `$F6A2` on the stack, called from `$F6D1`). `KB_OUT` holds those letters back and prints the Korean word when the next character arrives.
- **Variable names:** a stub of 30 bytes at `$0350` replaces the end of the zero-page `CHRGET` and skips `$0B` and `$01` in program text (not in the typed line at `$02xx`).
- **Memory:** `$8A00-$93FF`; `HIMEM` and the string top move down to `$8A00`. BASIC memory is 21,501 bytes before and about 19,000 after (`$4001-$89FF`).

## Limits and notes
- **TYPE and STYPE** use the same memory (`$8E00-$93FF` and `$8A00-$93FF`), so they cannot be loaded in the same session as KBASIC.
- After `HAN` the add-on keeps working. On `KBASIC.DSK` a cold boot loads it again by itself (`KSTART`); on another disk run `BRUN KBASIC` after the driver.
- The `LIST` hook relies on the K-HANGEUL driver's `LIST` code addresses (`$F6A2`, `$F6D1`) and on DOS 3.3 being in the output chain. If they change, `LIST` prints the English keyword; typing is not affected.
- A Korean word ending in a consonant followed directly by a vowel can be read two ways by the two-set keyboard; the driver and KBASIC read it the same way.
- **Korean error messages and Korean DOS commands come from the K-HANGEUL driver (v1.9 or later)**, not from KBASIC. Tables: [REFERENCE.txt](REFERENCE.txt) (Korean).
  - Applesoft errors read `?GOSUB 없음 오류`, `?0으로 나눔 오류`, and `?줄없음 오류 줄 10` inside a program. `BREAK IN 10` is `중단 줄 10`; `INPUT`'s `?EXTRA IGNORED` and `?REENTER` are `?입력 무시` and `?다시 입력`. The wording is short, with few spaces (`파일없음`, `용량초과`).
  - Only the text changes. Error codes (`PEEK(222)`) and `ONERR` are the same, but a program that reads the DOS or Applesoft error **text** now gets Korean.
  - DOS commands have Korean names too (the English names still work). Since v1.9: `LOAD` 받기, `DELETE` 제거, `BSAVE` 기저장, `BLOAD` 기받기, `BRUN` 기가동 (v1.9's `부르기` and `기부르기` clashed with the CALL word and were changed in v1.10). The old names give `?문법 오류`. A Korean DOS command name must be the first word of the line, and a KBASIC word with the same spelling is converted by KBASIC first (`실행` becomes `RUN`, `읽기` becomes `READ`: same result). A DOS name must not clash with a KBASIC word (the Korean word for CALL is `부르기`).
  - `포맷` (`INIT`) formats a disk on the Saturn card too.
  - On a Saturn card that is not 128K, and after `PR#0`, the messages are English. Monitor, Integer BASIC and ProDOS text is English.
- Not done: Korean words for `&`.

## Sample programs on KBASIC.DSK
KBASIC must be active (it is, after booting `KBASIC.DSK`): the samples use Hangul variable names, which need the add-on. Then `LOAD name`, `LIST`, `RUN`. All are text only (no graphics). Their Korean-keyword sources are in `src/samples/*.kbas`; `build_disk.py` converts and tokenizes them.

| File | Lines | What it shows |
|---|---|---|
| `KHELLO` | 8 | `입력`/`출력`, a Hangul string variable (`이름$`): type your name in Hangul mode |
| `KGUGU` | 12 | times table: `반복 ... 부터`, `다음`, `만약 ... 혹은 ... 그러면`, input check |
| `KSUM` | 12 | `자료`/`읽기` (DATA/READ), sum and average |
| `KGUESS` | 83 | number-guessing game: `호출`/`돌아` (GOSUB), `분기 ... 가기` (ON GOTO), `난수`, `정수`, `배열`, `램보기`/`램넣기` to wait for a key, difficulty choice, guess history, rating |
| `KSCORE` | 80 | grade book: five arrays, `자료` with Hangul names, bubble sort, `함수 기능` (DEF FN), `가로탭` columns, text bar chart, grades |

The converter refuses Hangul variable names that Applesoft would break: a name that starts with a keyword, whose letters hide an English keyword (e.g. `학생` is `GKRTOD`, which contains `TO`), or whose first two letters match another name.

## Music on KBASIC.DSK (MPLAY)
`KBASIC.DSK` also carries the Mockingboard player `MPLAY` from the `proj-apple2-mockingboard-pc` project, two short songs and three programs. You need a Mockingboard card.

| File | What it is |
|---|---|
| `MPLAY` | the resident player (binary, `$7D00`); `BRUN MPLAY` adds the command `& MPLAY "NAME"` |
| `ARTY - HOPE ORIGINAL MIX.MBF`, `CHPN-P24.MBF` | two short songs (`.MBF` files, ordinary DOS text files) |
| `KMUSIC` | Korean sample: Hangul title and menu, then plays the song you choose |
| `MUSIC` | the Mockingboard project's English menu |
| `PLAYMBF` | asks for a file name from `CATALOG` and plays it |

- **Run `RUN KMUSIC`** (or `MUSIC`). Any key stops the song.
- **`MPLAY` is not loaded at boot.** It must be installed after KBASIC, as the first statement of a program (`PRINT CHR$(4);"BRUN MPLAY"`), because it needs an empty string heap. The three programs do this themselves.
- **Memory:** once installed, `MPLAY` takes `$7100-$89FF` (6,400 bytes) and lowers `HIMEM` to `$7100`, leaving about 12,500 bytes of BASIC memory instead of 18,941. The song ring is 4 slots of 512 bytes; `& MPLAY n` changes it. The block stays until you reboot.
- **Space:** the disk has about 50 KB free (`NALGAE3` to `NALGAE16` are left off), so a few songs of that size fit. The build (`build_disk.py`) takes `MPLAY` and the songs from the Mockingboard project; see [BUILD_en.md](BUILD_en.md).
- Tested in MAME without a card (the program runs and `HIMEM` becomes the new value); playback itself is checked by the Mockingboard project's tests, not here.

How to build the disk and run the tests: [BUILD_en.md](BUILD_en.md).

## Renumbering lines (RENUMBER)

`KBASIC.DSK` carries Apple's `RENUMBER` (the Applesoft renumber and merge program from the DOS 3.3 System Master), changed so that it runs under KBASIC. It also handles programs written with Korean keywords: line numbers after `그러면` (THEN), `가기` (GOTO) and so on follow the new numbers.

-   **Install:** `RUN RENUMBER`, press RETURN at the prompts until `RENUMBER IS INSTALLED AND READY` appears, then `LOAD` your program. The machine code stays at `$8200-$89FF` when you load another program.

-   **Use:** type `&` in English.

    | Input | Effect |
    |---|---|
    | `&` | renumber everything as 10, 20, 30... |
    | `& FIRST 100, INC 5` | start at 100, step 5 |
    | `& FIRST 500, INC 10, START 30, END 70` | renumber only old lines 30-70, as 500, 510... |
    | `& HOLD`, `& MERGE` | keep the program on hold, load another one, then merge the held one into it |

-   **Memory:** `HIMEM` drops from `$8A00` to `$8200`; BASIC memory shrinks by about 2 KB. Do not press RESET while it works: it destroys the program.

-   **It cannot be used together with `MPLAY`.** `MPLAY` installs only while `HIMEM` is `$8A00` (otherwise `OUT OF MEMORY`), so it cannot go on after `RENUMBER`; and if `MPLAY` goes first, `RENUMBER` replaces the `&` vector and `& MPLAY` stops working. Use one, then reboot. `KUNLOAD` does not remove `RENUMBER`.

-   **What was changed (3 bytes):** the original dies on the KBASIC disk for three reasons. (1) Its last line does `MAXFILES 3`, which moves `HIMEM` back above KBASIC: now `NOMON I,O`. (2) It tests zero page `$02` to see whether Applesoft is in ROM; the driver changes that warm-start jump, so the test always says "ROM" now. (3) Its relocator stores ROM addresses reduced by the base page of a stock 48K DOS (`$8E`); one `JMP` into ROM gets the byte that matches the install address `$8200`. The patched file is made at build time from a DOS 3.3 System Master image and goes only on the disk; the original is not kept in the repository (see [BUILD_en.md](BUILD_en.md)). It is Apple's program: check before you hand the disk to others.

-   **Disk space:** to make room, `NALGAE3` to `NALGAE16` of the K-HANGEUL disk are left off (only `NALGAE1` and `NALGAE2` stay). About 50 KB are free.

## Playing a tune on the speaker (SPLAY)

`& SPLAY "tune"` plays a text tune on the Apple II's own speaker, with no sound card. The tune text is the same as for `& MPLAY "!tune"` of the Mockingboard project, so one tune works with both. Try `RUN KSPLAY`.

-   **Install:** `PRINT CHR$(4);"BRUN SPLAY"` as the first statement of a program (or typed), after KBASIC. If you use `MPLAY` too, `BRUN MPLAY` first and then `BRUN SPLAYM` (the same program linked just under `MPLAY`).

-   **Use:** type `&` and `SPLAY` in English. The argument is any string expression: `& SPLAY "T140 O4 L4 CCGGAAG2"` or `& SPLAY A$`. A number gives `?TYPE MISMATCH`. Any key stops it.

    | In the text | Meaning |
    |---|---|
    | `C D E F G A B` | a note, then `#` or `+` (sharp) or `-` (flat), then a length, then dots |
    | `R` | a rest (length and dots as for a note) |
    | `L4` | default length: 1 whole, 2 half, 4 quarter, 8 eighth ... up to 64 |
    | `O4` | octave 0-7 (C in octave 4 is middle C) |
    | `<` `>` | one octave down / up |
    | `T120` | tempo, quarter notes per minute, 32-255 |
    | `V12` | volume: accepted and ignored (the speaker has one volume) |
    | `/` | the speaker has one voice: what follows the first `/` is not played |

    A leading `!` is ignored. The start is `T120 L4 O4`. Spaces are ignored, but a number must follow its letter directly (`L8`, not `L 8`). A note of 3 ticks (1/50 s) or more ends with a short silence so that repeated notes can be told apart.

-   **Memory:** a 1,280-byte block lowers `HIMEM` by that much: `$8A00` to `$8500` (`SPLAY`), or `$7100` to `$6C00` (`SPLAYM`). Install it before the program makes any string (otherwise `OUT OF MEMORY`). `SPLAY` cannot go before `MPLAY` (`MPLAY` needs `HIMEM` at `$8A00`); with `MPLAY` use `SPLAYM`, last, and do not use `& MPLAY n` after it. The `&` vector is chained, so `& MPLAY` keeps working. It cannot be used together with `RENUMBER` (which takes `$8200-$89FF`).

-   **Accuracy:** the pitch is counted in processor cycles from the same period table as `MPLAY`. In MAME the notes were within 0.4 % of the right frequency and the tempo within 1 %. It is a plain square wave on the speaker; Ctrl-Reset in the middle of a tune is not handled. Not tried on real hardware.

## Removing KBASIC and the add-ons (KUNLOAD)

`BRUN KUNLOAD` removes KBASIC and the add-ons under it, leaving the K-HANGEUL driver: it prints `SPLAY OFF`, `MPLAY OFF` and `KBASIC OFF` for what it found (`NOT LOADED` if there was nothing). It takes them off from the top of the `&` chain, the last one installed first, puts back the `&` routine each one had replaced, wipes its memory and sets `HIMEM` back (to `$9400` when KBASIC was loaded). Typing and `LIST` are English again. An add-on that something else installed on top of is left alone. `RENUMBER` is not removed (it is Apple's code and relocates itself): reboot to get rid of it. KUNLOAD loads at `$02A0-$03C7`, the end of the input buffer and page 3, because the text page is overwritten by the driver while it prints.
