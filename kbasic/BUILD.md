KBASIC.DSK 만들기
=================

> English: [BUILD_en.md](BUILD_en.md)

원본에서 KBASIC 디스크를 만들고, 들어가는 것을 바꾸고, 시험하는 방법입니다.
KBASIC이 무엇이고 어떻게 쓰는지는 [KBASIC.md](KBASIC.md)를 보세요.

빠른 빌드 (Windows)
-------------------

`src\` 폴더에서 실행합니다.

    build_all.bat                  K-HANGEUL 드라이버 + KBASIC + 디스크
    build_all.bat nodriver         드라이버 빌드는 건너뛰고 K-HANGEUL의 기존 build\ 결과를 씀
    build_all.bat nodriver test    ... 그리고 MAME 키워드 시험까지 (`98/98 entries equal`이 나와야 함)

결과는 `dsk\KBASIC.DSK`입니다. `build_all.bat`은 차례대로 (1) K-HANGEUL 저장소의
`build.bat`(`nodriver`이면 건너뜀), (2) `python src\tools\build_disk.py`(키워드 표,
KBASIC, 예제, 디스크), (3) `python -X utf8 src\tools\test_keywords.py`(`test`일 때만)를
실행합니다. cmd나 PowerShell에서 실행하세요. 프로젝트 루트는 `src\`의 윗
폴더(`D:\dev\proj-apple2\proj-apple2-korean-basic`)라고 가정합니다.

필요한 것
---------

| 도구 | 쓰는 곳 | 이 프로젝트가 기대하는 곳 |
|---|---|---|
| Python 3 (3.12로 시험) | 빌드와 시험 스크립트 | PATH의 `python` |
| Pillow (`pip install pillow`) | `src/tools/shot.py`만. 고해상도 화면을 PNG로 | |
| cc65 2.19 (`ca65`, `ld65`) | 드라이버와 KBASIC 어셈블 | `D:\opt\devtools\asm\cc65\bin` |
| AppleCommander 14.1 (`acx`, Java 필요) | 디스크 이미지에 파일 넣기 | PATH의 `acx` (`C:\bin\acx`) |
| MAME 0.289, `apple2p` 롬 | 시험에만 | `D:\opt\classic\_emul\mame0289b_x64` |
| 머킹보드 프로젝트 | 음악 파일용 `MPLAY`, 곡, 그 프로젝트의 디스크 만들기 | `D:\dev\proj-apple2\proj-apple2-mockingboard-pc`. 먼저 거기서 `MPLAY`(`src\mb_basic\build.bat`이 `src\dist\MPLAY.bin`, `.lbl`을 만듦)와 `mbf_out\`의 곡을 만들어 두세요 |
| K-HANGEUL 저장소 | 드라이버 원본 | `D:\dev\proj-apple2\proj-apple2-korean-high-resolution\DIST-github`, `main` 브랜치(v1.10 이상: 한글 오류 메시지, 한글 DOS 이름) |

### 파이썬 환경

가상환경(venv)을 쓰기를 권합니다. 그러면 시스템에 깔린 것에 빌드가 좌우되지 않습니다.

    python -m venv .venv
    .venv\Scripts\activate
    pip install -r requirements.txt

`requirements.txt`(`src\`에 있음)에는 타사 패키지가 Pillow 하나뿐이고,
`tools/shot.py`와 `shot_x.py`만 씁니다. 나머지 스크립트(`build_disk.py`,
`gen_table.py`, `kbas2bas.py`, `atokenize.py`, `test_keywords.py`, `mame_run.py`)는
표준 라이브러리만 쓰므로 아무것도 깔지 않아도 빌드와 키워드 시험은 됩니다.
`.venv/`는 git이 무시합니다. `build_all.bat`은 PATH의 `python`을 쓰므로 같은
콘솔에서 먼저 venv를 켜세요. `gen_table.py`는 K-HANGEUL 저장소의 `src\tools`에 있는
`nbyte.py`(한글 자판 코드 변환)도 불러오므로 그 저장소가 있어야 합니다.

경로는 `src/tools/build_disk.py` 위쪽의 상수(`KH`, `CC65`, `ACX`)와
`src/tools/mame_run.py`의 `MAME`에 있습니다. 도구가 다른 곳에 있으면 거기서
고치세요.

MAME에는 슬롯 0에 언어 카드가 있어야 합니다(MAME 폴더의 `ini\apple2p.ini`에
`sl0 lang`). 새턴 카드를 시험하려면 `-sl0 ssram`을 씁니다(`src/tools/shot_x.py`는 MAME 인자를 더 받는 `shot.py`입니다). 슬롯 4에 모킹보드를 넣지 마세요.

폴더와 코드 구조
----------------

    proj-apple2-korean-basic\            프로젝트 루트 (src\만 git에 있음)
        HANDOVER.md                      다음 세션을 위한 메모
        prd.md, ANALYSIS*.md, KEYWORD_*.md   요구사항, 분석, 확정한 키워드 제안
        dos33blank.dsk, hangul-software\ 입력 파일. 절대 바꾸지 않음
        mame\                            MAME 상태와 시험 출력 (임시, 처음 쓸 때 생김)
        src\                             git 저장소 (main 브랜치)
            README.md, DEVELOPMENT.md, build_all.bat, requirements.txt
            dsk\KBASIC.DSK               결과물 (git에 있음)
            kbasic\
                kbasic.s                 부가 프로그램 (ca65)
                kbasic.cfg               ld65 메모리 지도
                keywords.tsv             확정한 키워드: 영어 토큰, 한글 낱말, 별칭 표시
                gen_table.py, keywords_load.py   keywords.tsv를 읽어 생성 파일을 씀
                build\                   생성됨: table.inc, msg_a.inc, msg_b.inc, drv.inc,
                                         KBASIC.bin, kbasic.map, kbasic.lst, driver\ (드라이버 파일 사본)
            samples\*.kbas               한글 키워드로 쓴 예제 프로그램
            kbasic.web\                  브라우저에서 도는 KBASIC (jsbasic 바탕; node test/run_tests.js, gen_data.py, gen_reference.py)
            emulator.js\                 KAPPLE ][ (Apple ][js 복제; npm install, npx webpack --mode=production)
            kbasic_editor\, editor-syntax\   편집기와 구문 색칠 파일
            tools\                       빌드와 시험 스크립트
            docs\                        KBASIC.md, KBASIC_en.md, BUILD.md, BUILD_en.md

`kbasic.s`는 파일 하나이고 ld65 세그먼트가 셋입니다(`kbasic.cfg`: 모듈 전체가
`$8A00`, 크기 `$0A00`). 드라이버의 입구 stub이 `$9400`에서 시작하므로 모듈은
`$9400` 아래에서 끝나야 하고, 넘으면 링커가 넘침 오류를 냅니다.

-   `CODE`: 입구 점프(`$8A00` BRUN 입구, `$8A03`의 `KB` 표식, `$8A05` `KB_IN`,
    `$8A08` `KB_OUT`), 표 찾기, 입력 줄 고쳐 쓰기, LIST 출력 걸러내기, 키워드 표
    (`build/table.inc`).
-   `VARS`: 모듈 변수.
-   `INSTALL`: 한 번만 도는 설치 코드. 파일의 맨 끝이고, 240바이트 줄 버퍼 `SRC`
    (`SRCBUF`, `$9310-$93FF`, 파일에는 바이트가 없음)가 그 꼬리 위에 놓이므로
    설치가 끝난 뒤에는 사라집니다.

훅은 드라이버가 자기 카드를 켠 채로 부르므로 ROM 루틴을 부르지 않고
`$C080-$C08F`도 건드리지 않습니다. 설치 코드는 드라이버 밖의 DOS 출력 훅
아래에서 돌기 때문에 DOS의 진짜 출력 루틴(`($AA53)`)으로 찍고, 숫자는 애플소프트
ROM으로 찍습니다. 코드와 메모리 지도의 자세한 내용은
[../DEVELOPMENT.md](../DEVELOPMENT.md)를 보세요.

빌드
----

    python src/tools/build_disk.py

결과는 `src/dsk/KBASIC.DSK`입니다. 스크립트는 `DIST-github\disk\K-HANGEUL.DSK`의
**사본**에서 시작하며, 원본은 바꾸지 않습니다. 에뮬레이터(AppleWin, MAME)가
그 디스크를 열고 있으면 "Permission denied"로 멈춥니다. 먼저 닫으세요.

스크립트가 하는 일은 차례대로 이렇습니다.

1.  **드라이버.** 드라이버는 이 스크립트가 만들지 않습니다. 먼저 K-HANGEUL
    저장소에서 `src\driver\build.bat`을 실행하세요(세 가지 빌드와
    `tools\satpack.py`; 일반 카드와 새턴 카드 모두를 위한
    `build\K-HANGEUL.bin`이 나옵니다). `build\K-HANGEUL.bin`이나 `driver.lbl`이
    없으면 스크립트가 멈춥니다. 두 파일을 `src/kbasic/build/driver/`로 복사하고
    (K-HANGEUL을 다시 빌드해도 도중에 바뀌지 않도록), 드라이버 `COUT` 루틴의
    주소만 `src/kbasic/build/drv.inc`에 씁니다(후크가 부르며, 설치 코드는
    쓰지 않습니다).

2.  **키워드 표.** `src/kbasic/gen_table.py`가 `src/kbasic/keywords.tsv`를 읽어
    `build/table.inc`(찾아보기 표)와 `build/msg_a.inc`, `build/msg_b.inc`(올릴
    때 보여 주는 메모리 안내의 한글 낱말)를 씁니다.

3.  **KBASIC.** `src/kbasic/kbasic.s`를 어셈블하고 `kbasic.cfg`로 링크해
    `build/KBASIC.bin`(올리는 주소 `$8A00`)을 만듭니다.

4.  **디스크.** `src/splay/splay.s`를 두 번 조립합니다(`SPLAY`는 `$8500`, `SPLAYM`은 `$6C00`). K-HANGEUL 디스크를 `src/dsk/KBASIC.DSK`로 복사하고
    `K-HANGEUL`(`build.bat`이 만든 드라이버), `KBASIC`, `KUNLOAD`를 넣습니다. 그다음
    `tools/make_renumber.py`가 DOS 3.3 System Master 이미지(환경 변수
    `KBASIC_DOS33`의 경로, 없으면 그 파일에 적힌 경로)에서 애플의 `RENUMBER`를 읽어
    크기와 CRC를 확인하고 3바이트를 고칩니다([KBASIC.md](KBASIC.md) 참고). 스크립트는
    그것을 디스크에 넣습니다. 이미지가
    없으면 이 단계는 건너뜁니다.

5.  **예제.** `src/samples/*.kbas`를 `kbas2bas.py`가 바꾸고(한글 키워드는 영어
    키워드로, 한글은 Ctrl-K ... Ctrl-A 자판 코드로 바꾸며, 한글 변수 이름은
    검사합니다), `atokenize.py`가 토큰으로 만들어 애플소프트 파일로 넣습니다.

6.  **부팅.** `HELLO`(글꼴을 올린 뒤 `EXEC KSTART`)와 텍스트 파일 `KSTART`
    (`BRUN K-HANGEUL`, `BRUN KBASIC`)를 씁니다.

7.  **음악.** 머킹보드 프로젝트의 `src\mb_disk\build_disk.py`를 모듈로 불러
    디스크 사본에 `--base`로 부릅니다. `MPLAY`(`MPLAY.lbl`의 `install` 주소에),
    `PLAYMBF`, `MB_SONGS`에 적은 곡(짧은 것 두 개)이 디스크에 들어가고, 이어서
    `MUSIC` 메뉴를 씁니다(`KMUSIC`은 5단계의 보통 예제입니다). `MPLAY`는
    `KSTART`에 없습니다. 맨 나중에, 프로그램의 첫 문장으로 올려야 하기
    때문입니다. `MPLAY`를 고쳤으면 먼저 머킹보드 프로젝트에서 새로 만든 뒤
    이 스크립트를 돌리세요.
8.  **문서.** `embed_keywords.py`가 `KBASIC_en.md`의 키워드 표를 새로
    만듭니다. `<!-- KEYWORDS -->` 표시가 없는 문서는 건드리지 않습니다
    (`KBASIC.md`의 표는 손으로 관리합니다).

시험
----

    python -X utf8 src/tools/test_keywords.py

MAME에서 디스크를 화면 없이 띄우고, 표의 모든 키워드를 한글로 한 번, 영어로 한
번 입력해 프로그램을 읽어 와 토큰을 비교합니다. `98/98 entries equal`로
끝나야 하고, 몇 분 걸립니다.

에뮬레이트한 기계를 들여다보는 `src/tools`의 다른 도구입니다.

| 스크립트 | 하는 일 |
|---|---|
| `mame_run.py 디스크 steps.json out.txt` | 단계 목록(기다림, 입력, 글쇠, 메모리 덤프)을 화면 없는 MAME에서 실행 |
| `shot.py 디스크 steps.json out.png` | 같은 일을 한 뒤 고해상도 화면을 PNG로 저장 |
| `shot_x.py 디스크 steps.json out.png [MAME 인자]` | MAME 인자를 더 받는 `shot.py`. 새턴 카드는 `-sl0 ssram` |
| `showbuf.py out.txt` | 드라이버 글자 버퍼 덤프를 글자로 보여 줌 |

MAME의 자체 상태는 프로젝트의 `mame/` 아래에 있습니다(처음 쓸 때 만들어집니다).
시험은 `src/dsk/KBASIC.DSK` 자체가 아니라 그곳에 둔 사본을 올립니다.

바꾸려면
--------

| 바꿀 것 | 고칠 곳 | 그다음 |
|---|---|---|
| 한글 키워드 | `src/kbasic/keywords.tsv` (낱말은 유일해야 하고, 다른 낱말의 앞부분이면 안 되고, 두 음절 이상이어야 함) | 빌드, 시험, 문서의 표 고치기 |
| 예제 프로그램 | `src/samples/이름.kbas` (새 파일은 이름으로 찾아 넣습니다) | 빌드 |
| 부가 프로그램 코드 | `src/kbasic/kbasic.s` | 빌드, 시험 |
| 버전 문구 | `kbasic.s`의 `KBASIC V1.4 OK` 글자(`install`) | 빌드, 문서 고치기 |
| 드라이버 | K-HANGEUL 저장소의 `src/driver/driver.s`(여기가 아님) | 거기서 `build.bat`을 실행한 뒤 이 디스크를 다시 빌드하고 시험 |

부가 프로그램은 `$9400` 아래에서 끝나야 합니다(그 위는 드라이버의 stub 자리).
넘으면 링커가 넘침 오류로 멈춥니다.

KBASIC에 필요한 훅 둘(`KBIN`, `KBOUT`)은 이제 K-HANGEUL 드라이버 자체에 들어
있으므로 패치할 것이 없습니다. 옛 `driver-hooks.patch`는 지웠습니다(git 기록에는 남아 있음).

원본 `K-HANGEUL.DSK`와 `dos33blank.dsk`는 절대 고치지 마세요. 빌드는 항상 사본으로
합니다.

문제 해결
---------

-   `missing ...K-HANGEUL.bin: run build.bat`: 드라이버를 먼저 빌드하세요
    (`nodriver` 없이 `build_all.bat`).
-   `Segment 'INSTALL' overflows memory area 'MAIN'`: 모듈이 `$9400`을 넘었습니다.
    코드를 줄여야 합니다(드라이버 stub이 `$940A`에 있어 한계를 옮길 수 없음).
-   부팅 때 `KBASIC: NEEDS THE KBASIC.DSK DRIVER`: `$940A`의 첫 바이트가 `8D`나
    `20`이 아니므로 K-HANGEUL 드라이버가 올라와 있지 않습니다(또는 KBASIC 훅이 없는
    빌드입니다).
-   `LIST`가 영어 키워드로 나옴: 드라이버의 `LIST` 코드 주소(`$F6A2`, `$F6D1`)가
    바뀐 것입니다. 입력은 그대로 됩니다.
-   `mockingboard project not found`: 음악 단계를 건너뜁니다. 오류가 아닙니다.
