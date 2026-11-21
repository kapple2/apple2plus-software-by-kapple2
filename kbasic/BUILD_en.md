# Building KBASIC.DSK

> 한국어: [BUILD.md](BUILD.md)

How to build `dsk/KBASIC.DSK` from source: the tools, how the folders and code are organised, and what each build step
does. For what KBASIC is and how it works see [../README.md](../README.md); for the code and memory map see
[../DEVELOPMENT.md](../DEVELOPMENT.md); for using KBASIC see [KBASIC_en.md](KBASIC_en.md).

## 1. Quick build (Windows)

From this folder (`src\`):

```
build_all.bat                  K-HANGEUL driver + KBASIC + disk
build_all.bat nodriver         skip the driver build, use the existing K-HANGEUL build\ output
build_all.bat nodriver test    ... and run the MAME keyword test (expect "98/98 entries equal")
```

The result is `dsk\KBASIC.DSK`. `build_all.bat` runs, in order:

1. `build.bat` of the K-HANGEUL repository (unless `nodriver`);
2. `python src\tools\build_disk.py` (keyword table, KBASIC, samples, disk);
3. `python -X utf8 src\tools\test_keywords.py` (only with `test`).

Run `build_all.bat` from cmd or PowerShell. The scripts assume the project root is the folder above `src\`
(`D:\dev\proj-apple2\proj-apple2-korean-basic`).

## 2. Tools

| Tool | Version | Used for | Where the build expects it |
|---|---|---|---|
| Python 3 | 3.12 tested | all build and test scripts | `python` on the PATH |
| Pillow | any recent | `tools/shot.py`, `shot_x.py` only (hi-res screen to PNG) | `pip install pillow` |
| cc65 (`ca65`, `ld65`) | 2.19 | assembling KBASIC (and the K-HANGEUL driver) | `CC65` in `tools/build_disk.py`, `D:\opt\devtools\asm\cc65\bin` |
| AppleCommander (`acx`) | 14.1, needs Java | writing files to the DOS 3.3 disk image | `ACX` in `tools/build_disk.py`, `acx` on the PATH |
| MAME | 0.289 (`apple2p` ROMs) | keyword test and screenshots only | `MAME` in `tools/mame_run.py`, `D:\opt\classic\_emul\mame0289b_x64` |
| K-HANGEUL repository (branch `main`) | v1.5 or later | the driver, its fonts, the base disk, `hello.bas` | `KH` in `tools/build_disk.py`, `D:\dev\proj-apple2\proj-apple2-korean-high-resolution\DIST-github` |
| Mockingboard project | optional | `MPLAY`, the songs and its disk builder | `MB` in `tools/build_disk.py`, `D:\dev\proj-apple2\proj-apple2-mockingboard-pc` |

### Python environment

Use a virtual environment so the build does not depend on whatever is installed system-wide (strongly encouraged):

```
python -m venv .venv
.venv\Scripts\activate
pip install -r requirements.txt
```

`requirements.txt` (in this folder) lists the only third-party package, Pillow, used by `tools/shot.py` and
`tools/shot_x.py`. Everything else (`build_disk.py`, `gen_table.py`, `kbas2bas.py`, `atokenize.py`, `test_keywords.py`,
`mame_run.py`) uses the standard library, so the build and the keyword test also work without installing anything.
`.venv/` is git-ignored. `build_all.bat` uses the `python` on the PATH, so activate the venv first in the same console.
`gen_table.py` also imports `nbyte.py` (the Hangul key encoder) from `src	ools` of the K-HANGEUL repository, so that
repository must be present.

If a tool lives elsewhere, change the constant named above. If the Mockingboard project is missing the build prints a
message and skips `MPLAY`, the songs and the `MUSIC` menu (the rest of the disk is complete).

MAME needs a language card in slot 0 (`-sl0 lang`, the default of the test scripts). To try a Saturn 128K card use
`-sl0 ssram`.

## 3. Folder and code structure

```
proj-apple2-korean-basic\            project root (not in git, except src\)
    HANDOVER.md                      notes for the next session
    prd.md, ANALYSIS*.md, KEYWORD_*.md   requirements, analysis and the approved keyword proposal
    dos33blank.dsk, hangul-software\ inputs; never modified
    mame\                            MAME state and test output (scratch, created on first use)
    src\                             git repository (branch main)
        README.md, DEVELOPMENT.md, build_all.bat, requirements.txt
        dsk\KBASIC.DSK               the product (tracked)
        kbasic\
            kbasic.s                 the add-on (ca65)
            kbasic.cfg               ld65 memory map
            keywords.tsv             approved keyword list: English token, Korean word, alias flag
            gen_table.py, keywords_load.py   read keywords.tsv, write the generated tables
            build\                   generated: table.inc, msg_a.inc, msg_b.inc, drv.inc, KBASIC.bin,
                                     kbasic.map, kbasic.lst, and driver\ (snapshot of the driver files)
        samples\*.kbas               sample programs, Korean-keyword source
        kbasic.web\                  KBASIC in the browser (jsbasic based; node test/run_tests.js, gen_data.py, gen_reference.py)
        emulator.js\                 KAPPLE ][ (Apple ][js copy; npm install, npx webpack --mode=production)
        kbasic_editor\, editor-syntax\   the editor and the syntax files
        tools\                       build and test scripts (below)
        docs\                        KBASIC.md, KBASIC_en.md, BUILD.md, BUILD_en.md
```

### `kbasic/kbasic.s`

One source file, three ld65 segments (see `kbasic.cfg`; the whole module is at `$8A00`, size `$0A00`, and must end below
`$9400` where the driver's entry stubs start; the linker reports an overflow if it does not):

- `CODE`: entry jumps (`$8A00` BRUN entry, `"KB"` signature at `$8A03`, `KB_IN` at `$8A05`, `KB_OUT` at `$8A08`), the
  table lookup, the typed-line rewriter, the LIST output filter and the generated keyword table (`build/table.inc`).
- `VARS`: the module's variables.
- `INSTALL`: the one-time install code (driver check, `CHRGET` patch, `HIMEM` change, start-up report). It is the last
  part of the file and the 240-byte line buffer `SRC` (segment `SRCBUF`, `$9310-$93FF`, no file bytes) is placed over its
  tail, so it is lost after install.

The hooks run inside the K-HANGEUL driver with its card mapped: they call no ROM routines and never touch
`$C080-$C08F`. The install code runs from main RAM under DOS's output hook, so it prints through DOS's real output
routine (`($AA53)`), not through the driver, and uses the Applesoft ROM for numbers.

### Generated files

`gen_table.py` turns `keywords.tsv` into `build/table.inc` (lookup table) and `build/msg_a.inc`, `build/msg_b.inc` (the
Korean words of the start-up report). `build_disk.py` writes `build/drv.inc` (`DRV_COUT`, the address of the driver's
COUT routine, from the driver's label file). Do not edit anything in `kbasic/build/`.

### `tools/`

| Script | What it does |
|---|---|
| `build_disk.py` | the whole build, see section 4 |
| `kbas2bas.py` | converts Korean-keyword source (`samples/*.kbas`) to English keywords and Hangul key codes (Ctrl-K ... Ctrl-A), and checks Hangul variable names |
| `make_renumber.py` | makes the patched `RENUMBER` from a DOS 3.3 System Master image (see step 4) |
| `atokenize.py` | tokenizes English Applesoft text into a binary Applesoft program |
| `embed_keywords.py` | refreshes the generated keyword table inside `docs/KBASIC_en.md` (between the `<!-- KEYWORDS -->` markers) |
| `test_keywords.py` | MAME test: types every keyword in Korean and in English and compares the tokens |
| `mame_run.py` | runs a list of steps (wait, type, keys, memory dumps) in headless MAME; `run(disk, steps, out, extra=[...])` |
| `shot.py` | same, then saves the hi-res screen as a PNG |
| `shot_x.py` | `shot.py` plus extra MAME arguments after the PNG name, e.g. `-sl0 ssram` |
| `showbuf.py` | prints a dump of the driver's text buffer as text |

## 4. What `build_disk.py` does

Run it from the project root: `python src/tools/build_disk.py`. It starts from a **copy** of `DIST-github\disk\K-HANGEUL.DSK`; the original is never changed. It fails with "Permission denied" if an emulator (AppleWin, MAME) has `src/dsk/KBASIC.DSK` open: close it first. The disk has about 42 KB free; adding files can make `acx` fail with a disk-full error.

What the script does, in order:

1. **Driver.** Does not build the driver: run `src\driver\build.bat` in the K-HANGEUL repository first (three variants plus `tools\satpack.py`; it makes `build\K-HANGEUL.bin` for both plain and Saturn cards). The script stops if `build\K-HANGEUL.bin` or `driver.lbl` is missing, copies both into `src/kbasic/build/driver/` (so a K-HANGEUL rebuild cannot change them mid-run), and writes `src/kbasic/build/drv.inc` with the address of the driver's `COUT` routine (the hooks call it; install does not).
2. **Keyword table.** `src/kbasic/gen_table.py` reads `src/kbasic/keywords.tsv` and writes `build/table.inc` (the lookup table) and `build/msg_a.inc`, `build/msg_b.inc` (the Korean words of the load-time memory report).
3. **KBASIC.** Assembles `src/kbasic/kbasic.s` and links it with `kbasic.cfg` into `build/KBASIC.bin` (load address `$8A00`).
4. **Disk.** Assembles `src/splay/splay.s` twice (`SPLAY` at `$8500`, `SPLAYM` at `$6C00`). Copies the K-HANGEUL disk to `src/dsk/KBASIC.DSK` and puts `K-HANGEUL` (the driver made by `build.bat`), `KBASIC` and `KUNLOAD` on it. Then `tools/make_renumber.py` reads Apple's `RENUMBER` from a DOS 3.3 System Master image (the path in the environment variable `KBASIC_DOS33`, else one of the paths listed in that file), checks size and CRC, patches 3 bytes (explained in the file and in [KBASIC_en.md](KBASIC_en.md)), and the script puts it on the disk. Without such an image the step is skipped.
5. **Samples.** Each `src/samples/*.kbas` is converted by `kbas2bas.py` (Korean keywords to English keywords, Hangul to Ctrl-K ... Ctrl-A key codes, with checks on Hangul variable names), tokenized by `atokenize.py`, and written as an Applesoft file.
6. **Boot.** Writes `HELLO` (loads the fonts, then `EXEC KSTART`) and the text file `KSTART` (`BRUN K-HANGEUL`, `BRUN KBASIC`).
7. **Music.** Loads the Mockingboard project's `src\mb_disk\build_disk.py` as a module and calls it with `--base` on a copy of the disk. That puts `MPLAY` (at the address of `install` in `MPLAY.lbl`), `PLAYMBF` and the songs listed in `MB_SONGS` (two short ones) on the disk; then the `MUSIC` menu is written (`KMUSIC` is an ordinary sample from step 5). `MPLAY` is not in `KSTART`: it must be installed last, as the first statement of a program. After changing `MPLAY`, rebuild it in the Mockingboard project first, then run this script.
8. **Docs.** Runs `embed_keywords.py`, which refreshes the generated keyword table in `KBASIC_en.md`. It leaves a document alone if the `<!-- KEYWORDS -->` markers are missing; the table in `KBASIC.md` is kept by hand.

## 5. Tests and checks

```
python -X utf8 src/tools/test_keywords.py
```

Boots the disk headless in MAME on an Apple II+, types every keyword of the table once in Korean (plus `(` where the
token ends in `(`) and once in English, reads the program back and compares the tokens. It must end with
`98/98 entries equal` (a couple of minutes). It mounts a copy of the disk under `mame/`, never `dsk/KBASIC.DSK`.

Visual check of the boot report and a Korean `LIST`, on a plain card and a Saturn card:

```
echo [["wait",2200],["type","LOAD KHELLO"],["wait",300],["type","LIST"],["wait",300]] > steps.json
python -X utf8 src/tools/shot_x.py src/dsk/KBASIC.DSK steps.json plain.png -sl0 lang
python -X utf8 src/tools/shot_x.py src/dsk/KBASIC.DSK steps.json saturn.png -sl0 ssram
```

Wait about 2200 frames for the boot. Do not `CATALOG` the disk in a screenshot: it is long and scrolls. Expect
`KBASIC V1.4 OK`, the free-memory line, `HIMEM $8A00 (35328) LOMEM $4003 (16387)`, then the Korean listing.

Other tools in `src/tools`, for looking at the emulated machine:

| Script | What it does |
|---|---|
| `mame_run.py disk steps.json out.txt` | runs a list of steps (wait, type, key presses, memory dumps) in headless MAME |
| `shot.py disk steps.json out.png` | the same, then saves the hi-res screen as a PNG |
| `shot_x.py disk steps.json out.png [MAME args]` | `shot.py` with extra MAME arguments, e.g. `-sl0 ssram` for a Saturn card |
| `showbuf.py out.txt` | prints a dump of the driver's text buffer as text |

MAME keeps its own state under `mame/` in the project (created on first use). The tests mount a copy of the disk there, never `src/dsk/KBASIC.DSK` itself.

## 6. Changing things

| To change | Edit | Then |
|---|---|---|
| a Korean keyword | `src/kbasic/keywords.tsv` (a word must be unique, not the start of another word, and at least two syllables) | build; run the test; update the tables in the docs |
| a sample program | `src/samples/NAME.kbas` (new files are picked up by name) | build |
| the add-on code | `src/kbasic/kbasic.s` | build; run the test |
| the version text | the `KBASIC V1.4 OK` string in `kbasic.s` (`install`) | build; update the docs |
| the driver | `src/driver/driver.s` in the K-HANGEUL repository (not here) | run its `build.bat`, then rebuild this disk and run the test |

The add-on must end below `$9400` (the driver's stubs start there): the linker stops with an overflow error if it does not.

The two hooks KBASIC needs (`KBIN`, `KBOUT`) are now part of the K-HANGEUL driver itself, so there is nothing to patch. The old `driver-hooks.patch` was deleted (it stays in git history).

Never edit the original `K-HANGEUL.DSK` or `dos33blank.dsk`; the build always works on copies.

## 7. Troubleshooting

- `missing ...K-HANGEUL.bin: run build.bat` : build the driver first (`build_all.bat` without `nodriver`).
- `Segment 'INSTALL' overflows memory area 'MAIN'` : the module grew past `$9400`; shrink the code (no room to move the
  limit, the driver's stub is at `$940A`).
- `KBASIC: NEEDS THE KBASIC.DSK DRIVER` at boot : the byte at `$940A` is not `8D` or `20`, so no K-HANGEUL driver is
  loaded (or it is a build without the KBASIC hooks).
- `LIST` prints English keywords : the driver's `LIST` code addresses moved (`$F6A2`, `$F6D1`); typing still works.
- `mockingboard project not found` : the music step is skipped, not an error.
