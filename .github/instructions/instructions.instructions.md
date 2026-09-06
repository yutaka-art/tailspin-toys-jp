---
description: 'How to write and maintain instruction files (`.github/instructions/*.instructions.md`) for this workshop content repo'
applyTo: '**/*.instructions.md'
---

# instruction ファイルの執筆

このリポジトリで Copilot を導く、スコープ付きの instruction ファイルを作成・維持するためのガイダンスです。これは **コンテンツ専用** の Astro + Starlight ワークショップリポジトリなので、instruction ファイルは *Markdown 執筆規約* を支配します — アプリケーションコードは扱いません（それは `github-samples/tailspin-toys` にあります）。

このファイルは instruction ファイル固有の内容をカバーします。機械的な Markdown の書式（ハードラップしない、admonition 構文、見出し、リンクスタイル）については、instruction ファイルも [`markdown.instructions.md`](./markdown.instructions.md) に従います — それらのルールをここで繰り返さないでください。

## instruction ファイルの場所

- 場所: `.github/instructions/`。
- 命名: 小文字とハイフンで、`.instructions.md` で終わります（例: `markdown-accessibility.instructions.md`）。
- 1 ファイルにつき 1 つの関心事。既存のセット: `markdown`（書式）、`markdown-accessibility`（a11y）、`astro`（`docs/` サイトのラッパー）。新しいファイルは、本当に新しい関心事のときのみ追加します。それ以外は既存のものを拡張します。

## 必須のフロントマター

すべての instruction ファイルは YAML フロントマターで始まります:

```yaml
---
description: 'One sentence stating the purpose and scope'
applyTo: '**/*.md'
---
```

- **description** — シングルクォートで囲んだ 1 文。著者（および Copilot）がファイルを区別する手がかりなので、具体的にします。
- **applyTo** — instruction が適用されるファイルを選ぶ glob。このリポジトリで使われるパターン:
  - `'**/*.md'` — すべての Markdown ファイル（書式、アクセシビリティ）。
  - `'docs/**/*.{astro,mjs,ts,js}'` — サイトのラッパー。
  - `'**/*.instructions.md'` — このメタガイド。

## 構造

- 1 つの `#` H1 タイトルから始め、次に `##` セクション。（instruction ファイルはリポジトリドキュメントなので、レッスン Markdown とは異なり、*本文に* H1 を持ちます。）
- セクションは短く、スキャンしやすく保ちます。ルールを先に示し、曖昧さを取り除く場合にのみ簡潔な例を続けます。
- 2 つのファイルが同じ領域をカバーする場合は、一方を拠点に選び、もう一方はそちらを指します。重複したガイダンスはずれていきます。

## instruction の高度（ゴルディロックスゾーン）

結果を完全に定義する最小限のルールセットを目指します。ルールは仮想の失敗のためではなく、実際の失敗の後に追加します。網羅的な意思決定表よりも、シグナルの高い例を優先します。

| 高度 | 失敗モード | 結果 |
| --- | --- | --- |
| 過剰に詳細 | 脆い if-this-then-that の散文 | 列挙しなかったケースで破綻する |
| 過少に詳細 | 共有された文脈を仮定 | 汎用的で規約外れの出力 |
| 適切な高度 | ヒューリスティック + 1 例 | 安定し、新しいコンテンツに一般化する |

## 文体

- 命令形: 「Use」「Define」「Avoid」— 「you should」「it might be good to」は使わない。
- 具体的かつ実行可能に。曖昧な助言は、具体的な指示と、有用な場合は `Good`/`Avoid` のペアに置き換えます。
- ファイル名、パス、リテラル構文にはバッククォートを、UI ラベルには太字を使います（`markdown.instructions.md` に従う）。

## 例

規約を単に説明するのではなく、示します。対比にラベルを付けます。

**Good** — 構文を名指しし、コールアウトを示す:

```markdown
Use GitHub admonition syntax for callouts in published lesson content:

> [!TIP]
> Run the dev server before editing.
```

**Avoid** — 抽象的で実行不可能:

```markdown
Callouts should be done properly using the right syntax.
```

## 避けるべきパターン

- **仮想ルールの膨張** — まだ起きていない失敗のためのルールを組み込まない。
- **他のファイルの再掲** — 機械的な書式は `markdown.instructions.md` に、ビルド/検証のプロセスは [`build-and-verify-docs`](../skills/build-and-verify-docs/SKILL.md) スキルに委ねる。
- **ここでツールを文書化** — instruction ファイルは *コンテンツがどうあるべきか* を記述します。*どうビルド/検証/プレビューするか* はスキルに属します。
- **曖昧な語** — 「should」「might」「possibly」は結果を未定義のままにする。
- **上流ドキュメントからのコピペースト** — 代わりにこのリポジトリ向けに要約して文脈化する。

## メンテナンス

- 規約、パス、ファイルが名前変更されたときは、それに言及する instruction ファイルを更新します（[`build-and-verify-docs`](../skills/build-and-verify-docs/SKILL.md) の PR 時整合性パスがこれを検出します）。
- プロジェクト構造の進化に合わせて、`applyTo` の glob を正確に保ちます。
- リポジトリの実態を反映しなくなったルールは、蓄積させるのではなく削除します。
