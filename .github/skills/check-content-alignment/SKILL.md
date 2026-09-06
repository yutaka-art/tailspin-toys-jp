---
name: check-content-alignment
description: Find workshop lessons that should change alongside an edit. Scans a diff (staged, unstaged, or a branch range) of the Copilot Workshops docs, extracts what changed, then searches the rest of docs/** for duplicated or parallel passages that now risk drifting out of sync — prose that used to be a shared partial and is now copied across pages, the same concept taught across the VS Code / CLI / App / Cloud harnesses, and cross-references to the changed page. Use after editing lesson content under docs/, before committing or opening a PR, or whenever asked to "check content alignment", "find related content to update", "what else should change", or "check for drift". Reports candidate files with line ranges and rationale; it does NOT edit content.
---

# コンテンツの整合性チェック

このワークショップは **純粋な Markdown** で構成されており、パーシャル（部分テンプレート）はありません。概念的に共有される散文は、単一ソース化されるのではなく、レッスン間で **インラインに複製** されています。これにより編集は簡単になりますが、1 回の編集ですべてのコピーが更新されるというパーシャルシステムの保証は失われます。このスキルはその代替となるガードレールです。ある変更が与えられると、同様に変更すべき *他の* 箇所を見つけ出し、複製されたコピーがずれていかないようにします。

このスキルは **助言的かつ読み取り専用** です — 候補とその根拠を提示し、人間（またはエージェント）が対応できるようにします。レッスンコンテンツを編集することはありません。

ステップで別途指定がない限り、すべてのコマンドは **リポジトリルート** から実行してください。

## 使うタイミング

- `docs/**` 配下の 1 つ以上のレッスンを編集した後、コミットや PR 作成の前。
- 「コンテンツの整合性をチェックして」「更新すべき関連コンテンツを見つけて」「他に何を変更すべき？」「ずれをチェックして」と依頼されたとき。
- **content-alignment のエージェント型ワークフロー**（`.github/workflows/content-alignment.md`）のローカル版として。このワークフローはプルリクエスト上で同じ分析を自動実行し、その結果をコメントします。プッシュする *前* にずれを検出するには、このスキルをローカルで実行してください。

## ずれの表面（何を探すか）

このリポジトリには、繰り返し現れる 3 種類の複製があります。ほとんどの整合性のずれはこのいずれかに該当します。

1. **かつて共有されていたコールアウトとステップ。** いくつかの短いコールアウトや演習ステップは、かつて `_shared/` に置かれ、多くのレッスンにインポートされていました。現在はそれぞれの利用先へ逐語的にコピーされています。1 つのコピーを変更すると、通常は他のコピーにも同じ変更が必要です。注意すべき波及度の高い散文の例:
   - 「Copilot CLI を起動 / **Allow all**」の承認コールアウト（複数の CLI/app ページで使われていた）。
   - 「**Approve and run workflows**」のステップ（`cloud/5-iterating.md` と `vscode/6-iterating.md` に登場）。
   - 共有された前提条件、MCP セットアップ、まとめの説明文。
2. **ハーネスをまたぐ並行した概念。** 同じ考え方が、ハーネスごとに最大 4 回教えられます: `cli/`、`vscode/`、`app/`、`cloud/`。概念的な変更（MCP の仕組み、カスタムインストラクションとは何か、エージェントがどのように変更を提案するか、Tailspin Toys デモアプリの説明）は、多くの場合、他のハーネスの対応するレッスンにも同じ修正が必要になります。
3. **相互参照と共有された事実。** 名前や見出しが変更されたレッスンへの参照スタイルリンク、散文中のレッスン番号、公開 URL の形（`/cli/3-generating-code/`）、デモアプリのリポジトリ URL（`github.com/github-samples/tailspin-toys/...`）、ツール / ライブラリ名、複数ページから参照されているスクリーンショット。

## 手順

### 1. 差分を取得する

レビュー対象に合った比較を選びます:

```bash
# 未ステージ + ステージ済みの作業ツリーの変更（デフォルト: 進行中の自分の編集をレビュー）
git diff -- docs
git diff --staged -- docs

# ブランチ全体と、マージ先のベースを比較（PR 規模の変更をレビュー）
git fetch origin
git diff origin/main...HEAD -- docs
```

`docs` に限定してください — このスキルが対象とするのはコンテンツのずれだけです。

### 2. 意味的な変更を抽出する

変更された各レッスンについて、単に動いた行ではなく、*意味として実際に何が変わったか* を要約します:

- 言い換えや修正された **事実**（バージョン、ライブラリ名、個数、挙動、UI ラベル、デモアプリの説明）。
- かつて共有されていた散文のように見える **コールアウト / ステップ** の編集（ずれの表面 #1 を参照）。
- 新規・名称変更・削除された **見出し、レッスンタイトル、ファイル**（これらは相互参照を無効にします）。
- 変更された **リンク**、レッスン番号、URL、画像参照。

純粋な書式（折り返し、空白）や、本当にそのページ固有の変更は無視します。

### 3. 並行するコピーを見つける

意味的な変更ごとに、残りのコンテンツから一致すべき箇所を検索します。変更後の文言（およびその類義語）をクエリとして使い、常に兄弟ハーネスも検索します。

```bash
# 同じコールアウト/ステップ/文を含む他のページを探す（特徴的なフレーズを引用）
grep -rn "Approve and run workflows" docs --include='*.md'

# 他のハーネスの並行レッスンを探す（同じ概念、異なるハーネス）
grep -rln "custom instruction" docs/cli docs/vscode docs/app docs/cloud --include='*.md'

# 名称/見出しを変更したページへの相互参照を探す
grep -rn "2-custom-instructions" docs --include='*.md'
```

すでに変更したファイルは候補リストから除外します。

### 4. 候補を報告する

簡潔で実行可能なレポートを作成します。各候補について次を示します:

- 更新が必要かもしれない箇所の **ファイル** とおおよその **行範囲**（または見出し）。
- **理由**: どの変更に並行しているか、そしてどのずれのカテゴリか（共有コピー、ハーネス横断、相互参照）。
- **推奨アクション**: 反映すべき具体的な編集、または「要確認 — 意図的に異なる可能性あり」。

元の変更ごとにグループ化します。最も確度の高い一致（逐語的な共有コピーの複製）を先頭に置き、確度の低いハーネス横断の「これも反映したいかもしれない」項目は分けます。並行する箇所が *意図的に* ハーネス固有であり、変更すべきでない場合は明確に示します。

### 5. 引き継ぐ

このスキルは推奨で止まります。ユーザーが整合させる編集を求める場合は、それを後続のコンテンツ編集タスクとして扱い、通常の執筆規約に従って変更を適用し、その後 **build-and-verify-docs** を再実行してください。

## 境界

- **読み取り専用。** このスキルの一部としてレッスンコンテンツを編集しないでください。
- **docs コンテンツのみ。** instruction ファイル、スキル、その他のリポジトリドキュメントを指摘しないでください — 対象は `docs/**` だけです。
- **助言であり、ゲートではありません。** 偽陽性は想定内で、判断するのは人間です。実際のずれを見逃すより、境界的な候補を（低確度と明示した上で）提示することを優先してください。
