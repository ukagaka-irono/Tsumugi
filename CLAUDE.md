# Tsumugi OS

Rust 製 x86_64 OS「Tsumugi」と、その上で動く Windows 2000 風デスクトップ「MADO」（別リポジトリ）のプロジェクト。

**作業を始める前に必ず [HANDOFF.md](HANDOFF.md) を最後まで読むこと。** 設計とユーザーの決定事項はすべてそこにある。決定事項は勝手に変えない。

## すぐ使う情報
- Rust は `C:\Users\yukin\.cargo\bin`（PATH 未設定）。PowerShell なら `$env:Path = "$env:USERPROFILE\.cargo\bin;$env:Path"`
- toolchain は nightly（`rust-toolchain.toml` で固定）。カーネルとアプリの target は `x86_64-unknown-none`
- QEMU は `C:\Program Files\qemu\qemu-system-x86_64.exe`（未導入なら `winget install SoftwareFreedomConservancy.QEMU`、承認済み）
- ビルド・実行・テスト: `cargo xtask build` / `cargo xtask run [--uefi]` / `cargo xtask test`
- アプリの実行ファイル形式は AYA（`.AYA`、中身は静的リンク ELF64）
- MADO リポジトリは隣の `C:\Users\yukin\IdeaProjects\MADO`（マイルストーン 11 で作る）
- 決定したデスクトップの見た目: [docs/mado-mockup.html](docs/mado-mockup.html)

## ルール
- README・docs・コミットメッセージ・コードコメントは日本語。OS の画面表示は英語（ASCII）
- マイルストーン完了ごとに QEMU で確認 → main に commit → push → タグ `v0.n`（承認済み）
- コミット末尾: `Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>`
- FreeDOS / Windows のコード・画像・フォント・ロゴ・名称は使わない（MIT で公開するため全部独自）
- 低レベル補助クレート（x86_64, uart_16550, pic8259, pc-keyboard, noto-sans-mono-bitmap, linked_list_allocator, spin, ホスト側の bootloader, fatfs）以外は自作
- 各マイルストーンの終わりに、できたこと・確認方法・残りを日本語で短く報告する
