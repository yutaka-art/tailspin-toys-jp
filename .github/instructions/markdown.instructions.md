---
description: 'Markdown conventions for repository docs, READMEs, and workshop lessons'
applyTo: '**/*.md'
---

# Markdown の規約

このファイルは、リポジトリ全体の Markdown 執筆をカバーします。`docs/**` 配下の公開レッスンコンテンツはプレーンな `.md` です。リポジトリドキュメントや instruction ファイルも Markdown ですが、主に github.com 上で読まれます。**どこでも GitHub の admonition 構文（`> [!NOTE]`）を使います** — 以下を参照してください。

## ハードラップしない

段落をハードラップしないでください。各段落、リスト項目、ブロッククォートは 1 行に収め、エディターのソフトラップに任せます。改行は実際の構造的な区切り（段落間、リスト項目、見出し、コードフェンス、テーブル行など）にのみ使います。

## admonition（コールアウト）

リポジトリ内のすべてのコールアウトに **GitHub の admonition 構文** を使います — 公開レッスンコンテンツ *と* リポジトリドキュメントの両方です。`[!TYPE]` マーカーはそれ自体を `>` 接頭の行に置き、本文は続く `>` 接頭の行に書きます:

```markdown
> [!NOTE]
> 続く引用行に書く本文。

> [!TIP]
> 本文。

> [!IMPORTANT]
> 本文。

> [!WARNING]
> 本文。

> [!CAUTION]
> 本文。
```

GitHub の admonition の 5 種類は `NOTE`、`TIP`、`IMPORTANT`、`WARNING`、`CAUTION` です。github.com 上ではネイティブにレンダリングされます。`docs/**` 配下の公開レッスンコンテンツでは、remark プラグイン（`remark-github-admonitions-to-directives`、`website/astro.config.mjs` で組み込み）がビルド時に次の対応で Starlight のアサイドに変換します:

| GitHub のタイプ | Starlight のアサイド |
| ----------- | --------------- |
| `NOTE`      | note            |
| `TIP`       | tip             |
| `IMPORTANT` | note            |
| `WARNING`   | caution         |
| `CAUTION`   | caution         |

Starlight の `:::` アサイドディレクティブはどこでも使わ **ない** でください — GitHub の admonition のみで執筆し、プラグインに変換を任せます。

### カスタムタイトル

GitHub の admonition 構文にはカスタムタイトルの形式がありません。コールアウトに見出しが必要な場合は、最初の本文行として **太字の導入行** に置き、次に空の引用行、その後に本文を続けます:

```markdown
> [!TIP]
> **Copilot Chat を開く**
>
> 本文。
```

### ネスト不可。隣接するコールアウトは空行で区切る

GitHub の admonition はネストできません。別のコールアウトの「中に」コールアウトが必要な場合は、代わりに **兄弟** のブロッククォートとして出力します。連続する admonition は空行で区切る **必要があります** — それがないと Markdown はそれらを 1 つのブロッククォートにマージし、2 つ目の `[!TYPE]` マーカーがリテラルなテキストとしてレンダリングされます:

```markdown
> [!TIP]
> 1 つ目のコールアウト。

> [!CAUTION]
> 2 つ目のコールアウト — 上の空行に注意。
```

## 見出し

- リポジトリドキュメント（README、CONTRIBUTING、instruction ファイル、スキルなど）: `# タイトル` から始め、セクションには `##` を使います。
- レッスンページ（`docs/**/*.md`）: 本文に `# H1` は置きません — タイトルはフロントマターから取得されます。本文の見出しは `##` から始めます。

## レッスンページのフロントマター

すべてのレッスンページにはフロントマターが必要です:

```yaml
---
title: "Exercise N - Lesson title"
description: "One-sentence summary (optional, used for SEO/meta)."
---
```

## コードフェンス

コードフェンスには常に言語をタグ付けします:

````markdown
```bash
echo "hello"
```

```python
def f(): pass
```
````

## リスト

- 順序なし: `-`（`*` や `+` ではなく）。
- 順序あり: `1.`、`2.`、... — Markdown は自動番号付けしますが、可読性のために実際の順序数を使います。

## ファイル名、UI 要素、リテラル値

[GitHub Docs スタイルガイド][github-style] に従います。要約すると:

- **バッククォート**: ファイル名、ディレクトリ名、コード、設定キー、ユーザーが入力するテキストやコード/設定にそのまま現れるリテラルテキストに使います。例: `` `.github/agents/accessibility.md` ``、`` `package.json` ``、`` `src/server/app.py` ``、`` `Code lacks documentation` ``（受講者がフィールドに入力するリテラル値）。
- **太字**: ユーザーが操作する UI 要素 — ボタン、タブ、メニュー項目、フィールドラベル、ダイアログ名に使います。例: `**Issues**` タブ、`**New issue**` ボタン、`**Title**` フィールド、`**Assign to Copilot**`。
- **太字**: 概念を初めて導入するときの強調に、控えめに使います。

目安: 読者が選択するものなら太字、入力したり、ファイルシステムで読んだり、コード内で目にするものならバッククォートです。

## アクセシビリティ

アクセシビリティの規約 — 説明的なリンクテキスト、代替テキスト、見出しの階層、平易な言葉、入力手段に依存しない動作動詞（「click」ではなく **select**） — は [`markdown-accessibility.instructions.md`](./markdown-accessibility.instructions.md) にあります。

## キーボードショートカット

`<kbd>` タグを使い、キーを `+` でつなぎ（スペースなし）、[GitHub Docs スタイルガイド][github-style-keyboard] に従って Mac の修飾キーを綴って書きます。Mac のショートカットを先に、次に Windows/Linux を示します:

```markdown
Press <kbd>Command</kbd>+<kbd>B</kbd> (Mac) or <kbd>Ctrl</kbd>+<kbd>B</kbd> (Windows/Linux).
```

Mac の修飾キー: **Command**、**Option**、**Control** と綴って書きます — ⌘、⌥、⌃、Cmd、Opt は使いません。Windows/Linux: **Ctrl**、**Alt**（略記）を使います。

## リンク

- 実用的な場合は参照スタイルのリンクを使います。参照定義は、そのリンクを持つファイルの末尾に置きます。
- 読者が自分のロケールに到達できるよう、URL からロケールコード（`/en/`、`/en-us/` など）を除去します。
- 説明的なリンクテキスト（「click here」を使わない、リンクだけの文にしない）については [`markdown-accessibility.instructions.md`](./markdown-accessibility.instructions.md) を参照してください。

## 画像

Markdown の画像構文を使い、パスは Markdown ファイルからの相対パスにします:

```markdown
![画像を説明する代替テキスト](../_images/some-screenshot.png)
```

## パスの規約

- パスごとのレッスン: `cli/`、`vscode/`、`cloud/`、`app/`。ファイルはレッスンの順に番号付けされます: `1-installing.md`、`2-custom-instructions.md` など。
- 補助画像は `_images/` ディレクトリにあり、`website/src/content.config.ts` によってルーティングから除外されます。

## リポジトリ間のリンク

受講者のテンプレートリポジトリは `github.com/github-samples/tailspin-toys` です。*この* リポジトリ内のファイルを、あたかもテンプレートであるかのようにリンクしないでください — それらのパスはもうここには存在しません。

[github-style]: https://docs.github.com/contributing/style-guide-and-content-model/style-guide
[github-style-keyboard]: https://docs.github.com/contributing/style-guide-and-content-model/style-guide#keyboard-shortcuts
