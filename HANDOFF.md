# 引き継ぎ資料 — Tsumugi OS / MADO

この資料は、Claude Code のデスクトップアプリで行った企画・設計の会話を、ターミナルの Claude Code（Claude Console）に引き継ぐためのものです。
**ここに書いてあることはすべてユーザーと合意済みです。** 決め直さずにこのとおり実装してください。迷ったときや資料に書いていないことを決めるときは、ユーザーに聞いてください。

- 作成日: 2026-11-09
- 状態: **設計完了・実装は未着手**（コードは 1 行もない。このフォルダにあるのはこの資料、`CLAUDE.md`、`docs/mado-mockup.html` だけ）
- 次にやること: 「11. 実装マイルストーン」の **1. 土台** から順に進める

---

## 1. 何を作るか
- `C:\Users\yukin\IdeaProjects\Tsumugi` に、Rust 製 x86_64 OS「**Tsumugi（紡）**」を一から作る
- 最低ラインは **MS-DOS / FreeDOS と同等以上の機能**、**FreeDOS 付属ソフトの Tsumugi 版**
- その上で、DOS 上の Windows 3.x のようにシェルから起動する簡易デスクトップ「**MADO（窓）**」を別リポジトリで作る。見た目は **Windows 2000 風**

## 2. ユーザーが決めたこと（一覧）
| 項目 | 決定 |
|---|---|
| 起動方式 | **rust-osdev/bootloader 0.11**（BIOS/UEFI ディスクイメージを生成） |
| QEMU | 未導入。**winget で入れてよい**と承認済み: `winget install SoftwareFreedomConservancy.QEMU` |
| 互換性 | **機能同等・独自 API**。DOS の .COM/.EXE は動かさない |
| 実行ファイル形式 | **AYA 形式**（拡張子 `.AYA`、読み「あや」＝綾）。DOS の .EXE にあたる |
| 日本語表示 | **後回し**。画面表示は ASCII・英語。キー配列は **JIS**（US に切替可） |
| デスクトップ | 名前 **MADO**、`C:\>MADO` で起動。見た目は **Windows 2000 風**（イメージ図で決定済み） |
| デスクトップのアプリ | **別々の AYA アプリ**として同時に動かす（カーネルはマルチタスク） |
| 起動の流れ | **DOS 画面（`C:\>`）で止まる**。MADO は手で起動（AUTOEXEC.BAT に書けば自動起動も可） |
| MADO のログオン | **Windows 2000 風ログオン**（ロゴ → ユーザー名・パスワード）。ユーザーごとに設定と My Documents |
| 画面解像度 | **Settings と MODE で切替**（既定 1024x768、640x480 / 800x600 / 1280x1024） |
| 起動表示 | **文字だけ**（DOS 風、ロゴ画面なし） |
| DOS 画面の配色 | **黒地に灰色文字**（`COLOR` で変更可） |
| バージョン | **0.1 から**。マイルストーン n 完了で `0.n`、全 13 完了で `1.0`。ビルド番号は xtask が自動で増やす |
| 設定ファイル | **`CONFIG.SYS` + `AUTOEXEC.BAT`**（DOS と同じ名前・役割） |
| 動作環境 | **今は QEMU / 仮想マシンだけ**（PS/2、IDE、Bochs VBE）。実機対応は後続 |
| 壁紙 | **単色＋模様**（既定は青単色。綾織り・市松・麻の葉風など独自模様を選べる） |
| スクリーンセーバー | **入れる** |
| デスクトップマスコット（伺か風） | **今回は入れない**（後続候補） |
| 文書の言語 | **日本語**（README・docs・コミットメッセージ・コードコメント）。OS 内の HELP と画面表示は英語 |
| CI | **GitHub Actions で回す**（push ごとにビルド＋ヘッドレス QEMU テスト、タグで Releases） |
| エラーメッセージ | **DOS そのまま**の定型文 |
| 終了時 | **そのまま電源オフ**（書き戻し後に QEMU が閉じる） |
| DOS 画面の Ctrl+Alt+Del | **確認してから再起動**（`Restart the computer? (Y/N)`） |
| 開発キット | **用意する**（`cargo xtask new-app NAME [--gui]`、日本語 docs） |
| QEMU の標準構成 | **メモリ 256MB、C: 512MB**（D: は試験用 32MB） |
| リポジトリ | OS 本体 **ukagaka-irono/Tsumugi**、デスクトップ **ukagaka-irono/MADO**（どちらも公開・現在空） |
| ライセンス | 両方 **MIT**（著作権者 ukagaka-irono） |
| commit / push | **マイルストーンごとに main へ commit して push してよい**（承認済み）。日本語のコミットメッセージ |
| 流用の禁止 | FreeDOS / Windows のコード・画像・フォント・ロゴ・名称は使わない。全部独自実装・独自デザイン |

## 3. 環境（2026-11-06 時点で確認済み）
- Windows 11 Pro。シェルは PowerShell（Git Bash も可）
- Rust: `C:\Users\yukin\.cargo\bin` に rustup あり。**PATH が通っていない**ので、最初に PATH へ追加するかフルパスで呼ぶ
  - toolchain: stable 1.99.1（既定）、**nightly-2026-10-28（rust-src あり）**
  - `rust-toolchain.toml` を置けば target（x86_64-unknown-none / x86_64-unknown-uefi）と llvm-tools は rustup が自動で入れる
- QEMU: **未導入**（winget で導入する。入れたら `C:\Program Files\qemu\qemu-system-x86_64.exe`）
- WSL: ディストリなし
- `gh` CLI: 使える（GitHub にログイン済み）
- `C:\Users\yukin\IdeaProjects\Tsumugi` は **git 未初期化**。GitHub の 2 リポジトリはどちらも**空**
- MADO リポジトリはまだ clone していない。`C:\Users\yukin\IdeaProjects\MADO` に置く予定（マイルストーン 11 で作る）

## 4. 全体アーキテクチャ
DOS の COMMAND.COM と同じく、**シェルもユーザー空間のアプリ `COMMAND.AYA`** にする。カーネルは CONFIG.SYS の `SHELL=` に書かれたシェルを起動するだけ（無ければエラー表示して停止）。
- カーネル: **プロセス毎のアドレス空間 + プリエンプティブなマルチタスク**（PIT 100Hz のラウンドロビン）。テキストモードでは DOS と同じ見た目（シェルが前面プログラムの終了を待つ）だが、`|` は本物の並行パイプ、MADO では複数アプリが同時に動く
- 端末: 文字アプリは **stdin/stdout + ANSI エスケープ**だけで画面を操作する。カーネルの画面コンソールも MADO のコマンドプロンプト窓も同じ ANSI 端末として振る舞うので、EDIT のような全画面アプリも窓の中で動く。窓用にカーネルは **疑似端末(PTY)** を提供する
- MADO: `MADO.AYA` が画面（フレームバッファ）・キーボード・マウスを専有するウィンドウマネージャ。各 GUI アプリは別プロセスで、**共有メモリ（窓の画素バッファ）+ メッセージキュー**で MADO とやりとりする。MADO 終了でテキスト画面に戻る
- システムコールは `int 0x80`（IDT の DPL=3、TSS の rsp0 でカーネルスタックへ自動切替）

### 4.1 起動の流れと CONFIG.SYS
1. bootloader → カーネル。黒地に灰色の文字で起動ログ:
   ```
   Starting Tsumugi...

   Tsumugi Version 0.1 (Build 0001)
   Memory: 256 MB
   Drive C: TSUMUGI  FAT32  512 MB
   Drive D: (not formatted)
   Processing CONFIG.SYS...
   ```
2. カーネルが `C:\CONFIG.SYS` を読む（無ければ既定値）。起動中に F8 を押すと 1 行ずつ確認しながら実行
3. `SHELL=` のシェルを起動 → シェルが `AUTOEXEC.BAT` を実行 → `C:\>`

既定の CONFIG.SYS:
```
REM Tsumugi system configuration
SHELL=C:\COMMAND.AYA /P
SCREEN=1024x768
COLOR=07
KEYB=JIS
LASTDRIVE=Z
BREAK=ON
```
既定の AUTOEXEC.BAT:
```
@ECHO OFF
PATH C:\TSUMUGI;C:\MADO
PROMPT $P$G
SET TEMP=C:\TEMP
VER
```

### 4.2 終了
- `SHUTDOWN`（DOS）と MADO の Shut down は、ディスクキャッシュを書き戻してから**そのまま電源オフ**（ACPI、QEMU では 0x604 ポートに 0x2000）。`REBOOT` / Restart は再起動（キーボードコントローラ 0x64 に 0xFE）
- 書き戻し中は `Shutting down...` を表示。MADO では各アプリに終了通知を送り、未保存なら「保存しますか？」
- DOS 画面の Ctrl+Alt+Del は `Restart the computer? (Y/N)` で確認してから再起動

### 4.3 エラーメッセージ（DOS そのまま）
`Bad command or file name` / `File not found` / `Path not found` / `Invalid drive specification` / `Invalid directory` / `Access denied` / `Invalid parameter` / `Required parameter missing` / `Insufficient disk space` / `Directory not empty` / `File cannot be copied onto itself` / ディスク失敗時 `General failure reading drive C` + `Abort, Retry, Fail?`
定型文は `userlib` の 1 か所にまとめる（日本語化の時に差し替えるため）。

## 5. 「DOS 同等以上」の機能範囲
| 領域 | DOS にあるもの（同等） | 上乗せ（以上） |
|---|---|---|
| コンソール | 文字画面、16 色属性、カーソル、ANSI エスケープ（NANSI 相当を標準装備） | 高解像度フレームバッファ(1024x768→128x48)、スクロールバック |
| 入力 | キーボード、Ctrl+C で中断、KEYB で配列切替(JIS/US)、マウス（CTMOUSE 相当を標準装備） | 行編集・履歴(↑↓)・Tab 補完（DOSKEY 相当を標準装備） |
| ファイルシステム | FAT12/16/32 読み書き、複数ドライブ(C: D: …)、ディレクトリ、属性、ボリュームラベル | 長いファイル名(LFN)、ディスクキャッシュ（LBACACHE 相当） |
| シェル内部コマンド | DIR CD MD RD TYPE COPY DEL REN MOVE CLS COLOR ECHO SET PATH PROMPT VER VOL DATE TIME REM PAUSE EXIT SUBST | REBOOT SHUTDOWN HISTORY |
| リダイレクト | `>` `>>` `<` `\|` | パイプは並行実行 |
| バッチ | `.BAT`、AUTOEXEC.BAT、`%1`〜`%9` `%VAR%`、IF (EXIST/ERRORLEVEL/==)、GOTO、CALL、SHIFT、FOR、CHOICE | |
| プログラム | 実行・終了コード(ERRORLEVEL)・引数・環境変数 | リング 3 で保護実行、マルチタスク |
| 時計・電源 | RTC 日付時刻、タイマー、電源（FDAPM 相当） | ACPI/QEMU シャットダウン、再起動 |
| GUI | （DOS 単体には無し） | MADO |

### 5.1 AYA 形式
- 中身は x86_64 静的リンク ELF64（ELF ヘッダ検証に失敗したら「AYA 形式ではありません」）
- 拡張子省略で起動可能。検索順はカレント → PATH、同名なら `.AYA` → `.BAT`
- 終了コードが `ERRORLEVEL` になる
- GUI アプリも同じ AYA 形式。MADO 外で起動すると「MADO の中で実行してください」と出して終了
- `x86_64-unknown-none` で静的リンク（`relocation-model=static`、リンカスクリプトで 0x40_0000 配置。各プロセスが自分の空間を持つので全部同じ番地で良い）→ xtask が `NAME.AYA` にリネームしてディスクへ

## 6. FreeDOS 付属ソフトの Tsumugi 版
FreeDOS 1.3 の BASE パッケージが基準。AYA アプリは `C:\TSUMUGI\` に置く。

| 種類 | AYA アプリ |
|---|---|
| シェル | COMMAND（`C:\COMMAND.AYA`） |
| テキスト | MORE, FIND, SORT, FC, COMP, CHOICE |
| ファイル | ATTRIB, TREE, XCOPY, DELTREE, REPLACE, MOVE（ディレクトリ移動対応） |
| 編集 | EDIT（全画面、メニュー・検索置換・複数ファイル・マウス）, EDLIN |
| ディスク | FORMAT, FDISK(MBR), CHKDSK(=DOSFSCK, 修復付き), LABEL, UNDELETE, DEFRAG, DISKCOPY/DISKCOMP（ドライブ⇔イメージファイル） |
| システム | MEM, MODE（解像度・行数・色・キーリピート）, KEYB, HELP |
| 開発 | DEBUG（16 進ダンプ・編集・x86_64 逆アセンブル） |

- **OS に組み込み**: NANSI(ANSI), LBACACHE(キャッシュ), FDAPM(電源), CTMOUSE(マウス), SWSUBST(SUBST), DOSKEY
- **作らない**: HIMEMX, EMM386, DEVLOAD, SHARE, EXE2BIN, SYS, MIRROR/UNFORMAT, RECOVER, NLSFUNC/DISPLAY/CPIDOS, APPEND/ASSIGN, PRINT

## 7. デスクトップ MADO
### 7.1 本体 `MADO.AYA`
- 背景・デスクトップアイコン、窓の重なり、タイトルバー、ドラッグ移動、リサイズ、最小化/最大化/閉じる、フォーカス、タスクバー、スタートメニュー、クイック起動、Run ダイアログ
- Alt+Tab で窓切替、Ctrl+Alt+Del で「応答しないアプリの強制終了」ダイアログ
- 起動時に `C:\MADO\MADO.INI`（既定）とユーザーごとの MADO.INI を読む。メニュー項目は `C:\MADO\MENU.INI`
- 終了で全アプリを閉じてテキスト画面に戻る

### 7.2 見た目（決定済み）
**決定したイメージ図: [`docs/mado-mockup.html`](docs/mado-mockup.html)**（ブラウザで開くと見られる）。手本はユーザーが貼った Windows 2000 Build 2031 の画面写真で、タスクバーだけ下端に変えたもの。マイルストーン 11 で MADO リポジトリの `docs/mockup.html` に移し、実装後の screendump と見比べる基準にする。
- 背景は青 `#3a6ea5` の単色。左端に白文字ラベルのデスクトップアイコン（My Documents / Drives / TERM / Trash / 置いたファイル）
- **タスクバーは画面下端**（`MADO.INI` で上端にも変更可）: 左に「MADO」スタートボタン（独自の 4 色ロゴ）、区切り線、クイック起動アイコン、区切り線、開いている窓のボタン（アイコン＋省略タイトル、前面は押し込み表示）、右端に凹んだ通知領域（キーボード・音量・ディスクのアイコン、時計）
- スタートメニュー: 左に縦帯「Tsumugi MADO」、Programs ▸ / Documents ▸ / Settings ▸ / Search ▸ / Help / Run... / Log off... / Shut down...
- 窓は灰色 `#d4d0c8`、白・灰・濃灰の縁による立体枠。アクティブ窓のタイトルバーは紺→水色の横グラデーション（`#0a246a`→`#a6caf0`）で太字白、非アクティブは灰→銀（`#808080`→`#c0c0c0`）。右端に最小化・最大化・閉じる（閉じるは少し離す）
- メニュー項目はアクセスキーに下線、Alt で操作可。メニュー下にツールバー（区切り線入り）、アドレス欄（白い凹み枠、選択中は紺の反転）
- 窓の下端に凹んだステータスバー（複数区画）
- FILES は左に紺色の Web 表示風パネル（フォルダ名を大きく、説明文、▸ 操作リンク）、右にアイコン一覧、クラシックなスクロールバー
- CLOCK はカレンダー兼用（左に灰色の縦ナビバー、右に月表示）
- タスクバーのすぐ上、左下に浮動の**キーボード状態バー**（A／JIS・US／CAPS・KANA ランプ、クリックで配列切替）
- タスクバーのすぐ上、右下に白文字で **`Tsumugi MADO (Build 0001)`**
- UI フォントは 8pt 相当の小さなビットマップフォント（独自または自由ライセンス。Tahoma は使わない）
- **似せるのは雰囲気まで**: Windows のロゴ・旗マーク、アイコン画像、効果音、フォント、「Windows」「Start」の名称は使わない。アイコンは独自に描く 16x16 / 32x32 ビットマップ
- 配色はテーマとして差し替え可能（「Windows 2000 風」が既定）

### 7.3 ログオン（Windows 2000 風）
- `MADO` 起動 → 青い起動ロゴ画面（「Tsumugi MADO」ロゴ、進捗バー）→ 「Ctrl+Alt+Del を押してログオン」→ ユーザー名・パスワードのダイアログ（OK / Cancel / Options、Cancel で DOS に戻る）
- 初回だけセットアップ画面で最初のユーザー（管理者）を作る。以降は Settings の Users で追加・削除・パスワード変更
- ユーザー情報は `C:\MADO\USERS.DAT`（**ソルト付きハッシュ**、平文は持たない）
- ユーザーごとのフォルダ `C:\USERS\<名前>\`（My Documents、デスクトップ、個人の MADO.INI）
- Shut down を押すと「Shut down / Restart / Log off / Exit to DOS」を選ぶダイアログ
- FAT に権限は無いので、ログオンは**ユーザー切り分けのため**でファイル保護ではない。README と HELP に明記

### 7.4 画面解像度の切替
- QEMU の標準 VGA（Bochs VBE / BGA）ドライバをカーネルに入れ、実行中に解像度を変える（`fb_set_mode` syscall）。対応しない環境では起動時の解像度に固定
- DOS 側は `MODE 800x600`、CONFIG.SYS は `SCREEN=`、MADO 側は Settings の Display（変更後 15 秒以内に「この設定を保持しますか？」、無応答なら戻す）

### 7.5 GUI ライブラリ `madolib`
描画プリミティブ（線・矩形・円・塗り・ビットマップ転送）、フォント描画、部品（ボタン、ラベル、テキスト入力、複数行テキスト、リスト、スクロールバー、メニューバー、チェック/ラジオ、メッセージボックス、ファイルを開く/保存ダイアログ）、イベントループ

### 7.6 付属アプリ
| アプリ | 内容 |
|---|---|
| FILES | エクスプローラー相当。ツールバー・アドレス欄、フォルダツリー、アイコン/詳細表示、コピー/移動/削除/改名/新規フォルダ、ドラッグ&ドロップ、関連付け起動（.TXT→NOTEPAD 等）。Drives / Documents / Trash もこれで開く |
| ごみ箱 | FILES で削除すると `C:\TRASH\` へ移動、元に戻す/空にする |
| NOTEPAD | 開く/保存、検索、切り取り/コピー/貼り付け（MADO のクリップボード） |
| CALC | 電卓（標準＋16 進） |
| CLOCK | 時計＋カレンダー（アナログ/デジタル、月表示、アラーム） |
| PAINT | 鉛筆・直線・矩形・円・塗りつぶし・色パレット、BMP で保存/読み込み |
| MINES | 地雷探しゲーム（独自デザイン） |
| CARDS | 一人用トランプ（クロンダイク、カード絵柄は独自描画） |
| SETTINGS | Display（解像度・配色テーマ・壁紙の色と模様・スクリーンセーバー）、Taskbar（位置・時計）、Keyboard（JIS/US）、Date/Time、Users |
| スクリーンセーバー | MADO 本体の機能。無操作 N 分で起動、解除時にパスワードを求める設定あり。種類: 糸が流れて織り上がる「Weave」、流れる文字「Marquee」、星空、真っ黒 |
| TERM | コマンドプロンプト窓。PTY 上で COMMAND.AYA を動かし、文字アプリ（EDIT 含む）を窓の中で実行。複数同時に開ける |

## 8. フォルダ構成
```
Tsumugi/                     ← このフォルダ（ukagaka-irono/Tsumugi）
  Cargo.toml                 # workspace: kernel, xtask, userlib, apps/*; default-members=["xtask"]
  rust-toolchain.toml        # nightly, rust-src, llvm-tools-preview, targets x86_64-unknown-none / x86_64-unknown-uefi
  .cargo/config.toml         # alias xtask = "run -p xtask --"
  .github/workflows/ci.yml   # ビルド＋ヘッドレステスト、タグで Releases
  docs/                      # 日本語の開発資料（設計、syscall 一覧、AYA アプリの作り方）
  disk/                      # C: に入れるファイル（CONFIG.SYS, AUTOEXEC.BAT, README.TXT, HELP 文書, サンプル）
  kernel/src/
    main.rs                  # entry_point!、BootloaderConfig（物理メモリ全体マップ、FB 最小 1024x768、
                             #   dynamic_range を上位半分へ寄せてユーザー領域と衝突させない）、初期化、起動ログ、シェル起動、panic
    config.rs                # CONFIG.SYS の解析（F8 で 1 行ずつ確認）
    serial.rs                # COM1、serial_println!（デバッグ・自動テスト用）
    console/                 # フォント描画・文字グリッド・16 色・カーソル・スクロール・ANSI 解釈
    arch/                    # GDT/TSS、IDT、PIC、PIT、int 0x80(DPL3)、コンテキストスイッチ
    memory/                  # ページテーブル、フレームアロケータ、ヒープ、プロセス用アドレス空間、共有メモリ
    drivers/                 # keyboard(JIS/US)、mouse(PS/2, IRQ12)、ata(PIO, 4 台, キャッシュ)、rtc、power、bga
    fs/                      # fat.rs（FAT12/16/32・LFN 自作）、vfs.rs（ドライブ文字・パス・SUBST）
    process/                 # AYA ローダー、プロセス表、スケジューラ、wait/exit、Ctrl+C
    ipc/                     # パイプ、PTY、メッセージキュー
    syscall.rs               # int 0x80: プロセス(spawn wait exit kill getpid yield sleep)、ファイル(open close read
                             #   write seek stat setattr readdir mkdir rmdir unlink rename chdir getcwd dup2 pipe sync)、
                             #   端末(ioctl: size/raw/echo)、環境(args getenv)、時刻、sbrk/meminfo、ディスク(blk_*)、
                             #   keyb_layout、電源(shutdown/reboot)、GUI(fb_acquire/release/set_mode, input_acquire, shm_*, mq_*)
  userlib/                   # no_std ランタイム：_start、syscall、print!、alloc、引数、ファイル、ANSI/TUI ヘルパ、
                             #   DOS 定型エラーメッセージ、FAT 構造体
  apps/<name>/               # 文字アプリ（COMMAND と FreeDOS 相当一式 + HELLO）
  templates/                 # new-app の雛形
  xtask/src/main.rs          # build / run [--uefi] / test / new-app NAME [--gui]（Windows と Linux 両対応）

MADO/                        ← C:\Users\yukin\IdeaProjects\MADO（ukagaka-irono/MADO）、マイルストーン 11 で作る
  Cargo.toml                 # workspace: madolib, mado, apps/*
  rust-toolchain.toml        # Tsumugi と同じ nightly
  .github/workflows/ci.yml   # Tsumugi も checkout して一緒にビルド・テスト
  docs/                      # mockup.html（決定したイメージ図）、MADO プロトコル、GUI アプリの作り方
  madolib/  mado/  apps/<files, notepad, calc, clock, paint, mines, cards, settings, term>/
  disk/                      # MADO.INI、MENU.INI、アイコンなど C:\MADO\ に入れるもの
```
- **境界**: Tsumugi カーネルは汎用の仕組み（フレームバッファ/入力の専有、共有メモリ、メッセージキュー、PTY）だけ。窓・メッセージ形式・部品など MADO 固有のものは全部 MADO リポジトリ側
- **依存**: MADO は `userlib = { path = "../Tsumugi/userlib" }`（2 つを隣同士に clone する前提、README に明記）
- **ビルド連携**: Tsumugi の xtask が `../MADO` を見つけたら一緒にビルドして `C:\MADO\` に入れる（`--mado <path>` で指定、無ければ MADO 抜き）
- 使ってよい補助クレート: x86_64, uart_16550, pic8259, pc-keyboard, noto-sans-mono-bitmap, linked_list_allocator, spin（ホスト側の xtask では bootloader, fatfs）。**FAT・ATA・スケジューラ・ローダー・syscall・IPC・シェル・MADO・全アプリは自作**。バージョンは実装時に `cargo add` で最新を取る
- xtask の流れ: カーネル・全アプリ（＋MADO）をビルド → bootloader で `target/tsumugi-{bios,uefi}.img` → ホスト側 `fatfs` で FAT32 の `target/c-drive.img`(512MiB) に `COMMAND.AYA`・`TSUMUGI\*.AYA`・`MADO\*`・`disk/` をコピー、空の `target/d-drive.img`(32MiB) → QEMU（`-m 256M`、IDE で C:/D:、PS/2 マウス、`-vga std`、`-serial stdio`、UEFI 時は QEMU 同梱 edk2 を pflash）。QEMU は PATH → `C:\Program Files\qemu` の順に探す

## 9. 開発キット
- `cargo xtask new-app NAME` → `apps/NAME/` に文字アプリの雛形。次の build で `C:\TSUMUGI\NAME.AYA` に入る
- `cargo xtask new-app NAME --gui` → MADO 側 `apps/NAME/` に GUI アプリの雛形（窓を開いてボタンでメッセージボックス）
- 日本語 docs: はじめての AYA アプリ、syscall 一覧、userlib API、ANSI 端末、MADO プロトコルと madolib 部品一覧
- `cargo doc` で API リファレンス。雛形は CI でもビルドする

## 10. CI（GitHub Actions）
- Tsumugi: push と PR で ubuntu-latest に nightly Rust と `qemu-system-x86` を入れ、`cargo xtask build` → `cargo xtask test`。失敗時はシリアルログと screendump をアーティファクトに保存
- MADO: Tsumugi を隣に checkout して同じくビルド・テスト
- タグ `v0.n` で `tsumugi-bios.img` / `tsumugi-uefi.img` / `c-drive.img` を zip にして Releases に添付（説明は日本語）

## 11. 実装マイルストーン
各段階で **QEMU で動作確認 → commit → push → バージョンを 0.n に上げて git タグ `v0.n`**。

| # | 内容 | リポジトリ |
|---|---|---|
| 1 | **土台**: QEMU 導入、git 初期化（日本語 README・MIT LICENSE・.gitignore）、ワークスペース・xtask、シリアル＋画面出力、CI | Tsumugi |
| 2 | **割り込み**: GDT/TSS、IDT、例外、PIC、PIT、キーボード入力エコー | Tsumugi |
| 3 | **メモリ**: ページング・フレームアロケータ・ヒープ | Tsumugi |
| 4 | **ディスク**: ATA PIO、FAT 読み書き、VFS、複数ドライブ、起動ログのドライブ表示 | Tsumugi |
| 5 | **プロセス**: AYA ローダー、リング 3、プロセス毎アドレス空間、スケジューラ、syscall、ファイルハンドル、パイプ、ANSI 端末、userlib、CONFIG.SYS、HELLO.AYA | Tsumugi |
| 6 | **COMMAND.AYA**: 行編集・履歴・補完、内部コマンド、リダイレクト/パイプ、PATH、Ctrl+C、ERRORLEVEL、DOS 定型エラー、RTC、SHUTDOWN/REBOOT、Ctrl+Alt+Del 確認 | Tsumugi |
| 7 | **バッチ・開発キット**: AUTOEXEC.BAT と制御構文、`xtask new-app`（文字アプリ）と docs | Tsumugi |
| 8 | **FreeDOS 相当 (1)**: MORE FIND SORT FC COMP CHOICE ATTRIB TREE XCOPY DELTREE REPLACE MOVE | Tsumugi |
| 9 | **FreeDOS 相当 (2)**: マウスドライバ、Bochs VBE 解像度切替、MEM MODE KEYB HELP EDLIN EDIT | Tsumugi |
| 10 | **FreeDOS 相当 (3)**: FORMAT FDISK CHKDSK LABEL UNDELETE DEFRAG DISKCOPY DISKCOMP DEBUG | Tsumugi |
| 11 | **MADO 基盤**: [Tsumugi] 画面・入力の専有と返却、共有メモリ、メッセージキュー、PTY → [MADO] リポジトリ作成、madolib、MADO.AYA（窓管理・タスクバー・スタートメニュー・デスクトップアイコン）、GUI 雛形と docs | 両方 |
| 12 | **MADO アプリ (1)**: ログオン/ユーザー、TERM, NOTEPAD, CALC, CLOCK, SETTINGS | MADO |
| 13 | **MADO アプリ (2)**: FILES, PAINT, MINES, CARDS, スクリーンセーバー、壁紙模様 → **1.0** | MADO |

**後続（今回の範囲外）**: 実機対応（USB 起動・xHCI・AHCI/NVMe）、BMP 壁紙、RAM ディスク、CD-ROM + ISO9660、ZIP/UNZIP、パッケージ管理、ネットワーク、サウンド、日本語表示、伺か風デスクトップマスコット

### 11.1 マイルストーン 1 の具体的な手順
1. PATH に `C:\Users\yukin\.cargo\bin` を追加（セッション内）
2. `winget install SoftwareFreedomConservancy.QEMU`（承認済み）
3. `git init -b main`、`git remote add origin https://github.com/ukagaka-irono/Tsumugi.git`
4. `.gitignore`（`/target`、`*.img`）、`LICENSE`（MIT、Copyright (c) 2026 ukagaka-irono）、日本語 `README.md`
5. `rust-toolchain.toml`、ワークスペース `Cargo.toml`、`.cargo/config.toml`
6. `kernel/`: `#![no_std] #![no_main]`、`bootloader_api::entry_point!`、COM1 シリアル出力、フレームバッファに 8x16 程度の等幅フォントで文字描画（黒地に灰色）、`Starting Tsumugi...` と `Tsumugi Version 0.1 (Build 0001)` を表示、panic ハンドラ、`hlt` ループ
7. `xtask/`: `build`（kernel を x86_64-unknown-none でビルド → BIOS/UEFI イメージ）、`run [--uefi]`、`test`（ヘッドレス起動してシリアル出力に期待文字列があるか確認、isa-debug-exit で終了コードを返す）
8. `.github/workflows/ci.yml`
9. QEMU で起動ログが出るのを確認 → 日本語メッセージで commit → push → タグ `v0.1`

## 12. 検証のしかた
- `cargo xtask build`: 警告なしで全部生成
- `cargo xtask run` / `run --uefi`: 起動ログ → `C:\>` → `MADO` でログオン → デスクトップ → 抜けられる（各マイルストーンでユーザーにも見てもらう）
- `cargo xtask test`（手元と CI）: ヘッドレス QEMU でテスト用 `AUTOEXEC.BAT` を実行。内部コマンド、DOS 定型エラー、リダイレクト・並行パイプ、IF/GOTO/FOR/CHOICE、ERRORLEVEL、各文字アプリ、D: に対する FDISK→FORMAT→コピー→DEL→UNDELETE→CHKDSK→DEFRAG→LABEL。シリアル出力を期待文字列と照合、最後に SHUTDOWN で QEMU が自分で終了すること
- 全画面アプリと MADO: QEMU モニタ（`-monitor tcp:127.0.0.1:4444,server,nowait`）の `sendkey` / `mouse_move` / `mouse_button` で操作し、`screendump x.png -f png` で撮った画面を確認。`docs/mado-mockup.html` と見比べる
- 例外系: わざとページフォルトを起こすアプリで、そのアプリだけ終了してシェル/MADO が生き残ること。無限ループのアプリがあっても他の窓が動き、強制終了できること
- ホストの `fatfs` で C:/D: イメージを読み戻し、OS が書いた・FORMAT した FAT が正しいか、SHUTDOWN 前の書き込みが残っているか

## 13. 進め方の約束
- 決定事項（2 章）を勝手に変えない。変えたほうが良さそうなら理由を添えてユーザーに聞く
- 各マイルストーンの終わりに、何ができたか・どう確かめたか・残っていることを日本語で短く報告する
- commit / push / タグはマイルストーン完了時に行ってよい（承認済み）。それ以外の外部への操作（Releases の手動作成、リポジトリ設定の変更など）は聞いてから
- コミットメッセージ・README・docs・コードコメントは日本語。OS の画面に出る文字は英語（ASCII）
- コミットの末尾に `Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>`
