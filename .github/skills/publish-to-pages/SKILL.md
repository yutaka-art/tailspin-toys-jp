---
name: publish-to-pages
description: 'Publish presentations and web content to GitHub Pages. Converts PPTX, PDF, HTML, or Google Slides to a live GitHub Pages URL. Handles repo creation, file conversion, Pages enablement, and returns the live URL. Use when the user wants to publish, deploy, or share a presentation or HTML file via GitHub Pages.'
---

# publish-to-pages

あらゆるプレゼンテーションや Web コンテンツを、一括で GitHub Pages に公開します。

## 1. 前提条件のチェック

これらは静かに実行します。エラーのみ表示します:

```bash
command -v gh >/dev/null || echo "MISSING: gh CLI — install from https://cli.github.com"
gh auth status &>/dev/null || echo "MISSING: gh not authenticated — run 'gh auth login'"
command -v python3 >/dev/null || echo "MISSING: python3 (needed for PPTX conversion)"
```

`poppler-utils` はオプションです（`pdftoppm` による PDF 変換）。これでブロックしないでください。

## 2. 入力の判定

ユーザーが提供したものから入力の種類を判定します:

| 入力 | 判定方法 |
|-------|-------|
| HTML ファイル | 拡張子 `.html` または `.htm` |
| PPTX ファイル | 拡張子 `.pptx` |
| PDF ファイル | 拡張子 `.pdf` |
| Google スライドの URL | URL に `docs.google.com/presentation` を含む |

指定がなければ、ユーザーに **リポジトリ名** を尋ねます。デフォルト: 拡張子を除いたファイル名。

## 3. 変換

### 大きなファイルの扱い

どちらの変換スクリプトも、大きなファイルを自動検出し、**外部アセットモード** に切り替えます:
- **PPTX:** 20MB 超、または画像 50 枚超 → 画像を `assets/` に別ファイルとして保存
- **PDF:** 20MB 超、または 50 ページ超 → ページの PNG を `assets/` に保存
- 150MB 超のファイルは警告を表示します（PPTX の場合は代わりに PDF 経由を提案）

これにより、個々のファイルを GitHub の 100MB 制限を十分に下回るサイズに保ちます。小さいファイルは引き続き、自完結型の単一 HTML を生成します。

`--external-assets` または `--no-external-assets` で動作を強制できます。

### HTML
変換は不要です。ファイルをそのまま `index.html` として使用します。

### PPTX
変換スクリプトを実行します:
```bash
python3 SKILL_DIR/scripts/convert-pptx.py INPUT_FILE /tmp/output.html
# 大きなファイルの場合は、外部アセットを強制:
python3 SKILL_DIR/scripts/convert-pptx.py INPUT_FILE /tmp/output.html --external-assets
```
`python-pptx` がない場合は、ユーザーに伝えます: `pip install python-pptx`

### PDF
同梱のスクリプトで変換します（`pdftoppm` のため `poppler-utils` が必要）:
```bash
python3 SKILL_DIR/scripts/convert-pdf.py INPUT_FILE /tmp/output.html
# 大きなファイルの場合は、外部アセットを強制:
python3 SKILL_DIR/scripts/convert-pdf.py INPUT_FILE /tmp/output.html --external-assets
```
各ページは PNG としてレンダリングされ、スライドナビゲーション付きで HTML に埋め込まれます。
`pdftoppm` がない場合は、ユーザーに伝えます: `apt install poppler-utils`（macOS では `brew install poppler`）。

### Google スライド
1. URL からプレゼンテーション ID（`/d/` と `/` の間の長い文字列）を抽出します。
2. PPTX としてダウンロードします:
```bash
curl -L "https://docs.google.com/presentation/d/PRESENTATION_ID/export/pptx" -o /tmp/slides.pptx
```
3. その後、上記の変換スクリプトで PPTX を変換します。

## 4. 公開

### 公開範囲（可視性）
リポジトリはデフォルトで **public**（公開）として作成されます。ユーザーが `private` を指定した（またはプライベートリポジトリを希望する）場合は `--private` を使います。ただし、プライベートリポジトリでの GitHub Pages には Pro、Team、または Enterprise プランが必要であることに注意してください。

### 公開する
```bash
bash SKILL_DIR/scripts/publish.sh /path/to/index.html REPO_NAME public "Description"
```

ユーザーが要求する場合は `public` の代わりに `private` を渡します。

このスクリプトは、リポジトリを作成し、`index.html`（あれば `assets/` も）をプッシュし、GitHub Pages を有効化します。

**注意:** 外部アセットモードを使う場合、出力される HTML は `assets/` 内のファイルを参照します。公開スクリプトは `assets/` ディレクトリを自動検出し、HTML ファイルとともにコピーします。HTML ファイルとその `assets/` ディレクトリが同じ親ディレクトリにあることを確認してください。

## 5. 出力

ユーザーに伝えます:
- **リポジトリ:** `https://github.com/USERNAME/REPO_NAME`
- **ライブ URL:** `https://USERNAME.github.io/REPO_NAME/`
- **注意:** Pages が公開されるまで 1、2 分かかります。

## エラー処理

- **リポジトリがすでに存在する:** 番号（`my-slides-2`）や日付（`my-slides-2026`）を末尾に付けることを提案します。
- **Pages の有効化に失敗:** それでもリポジトリ URL を返します。ユーザーはリポジトリの Settings で手動で Pages を有効化できます。
- **PPTX の変換に失敗:** ユーザーに `pip install python-pptx` の実行を伝えます。
- **PDF の変換に失敗:** `poppler-utils` のインストール（`apt install poppler-utils` または `brew install poppler`）を提案します。
- **Google スライドのダウンロードに失敗:** プレゼンテーションが一般公開されていない可能性があります。ユーザーに閲覧可能にするか、PPTX を手動でダウンロードするよう依頼します。
