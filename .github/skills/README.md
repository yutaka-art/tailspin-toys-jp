# スキル

スキルは、このリポジトリで繰り返し発生するタスクに対して、Copilot（および人間のコントリビューター）が従える、再利用可能で自完結型のプレイブックです。各スキルは、`name` と `description` のフロントマターを持つ `SKILL.md` として専用フォルダーにあります。タスクがその説明に一致すると、Copilot が自動的に読み込みます。

このファイルは、人が一目で存在するスキルを把握できるようにする **索引** です。これ自体はスキルではなく（`SKILL.md` ではない）、自動で読み込まれることはありません — スキルを追加・名前変更・削除したときは最新に保ってください（[`build-and-verify-docs`](./build-and-verify-docs/SKILL.md) の PR 時の整合性チェックを参照）。

## リポジトリ固有のスキル

これらは、この Astro + Starlight ワークショップコンテンツリポジトリに固有の規約をエンコードしたものです。

| スキル | 内容 | 使う場面 |
|---|---|---|
| [`build-and-verify-docs`](./build-and-verify-docs/SKILL.md) | 正式なビルド / プレビュー / 検証のプロセス: 開発サーバー、クリーンビルド、ページ数の不変条件、lychee のリンクチェック、および構造的なずれに対する **PR 時の整合性チェック**。 | `docs/` 配下の変更をビルド・プレビュー・検証するとき。すべてのコミット / PR の前。 |
| [`check-content-alignment`](./check-content-alignment/SKILL.md) | 差分（ステージング済み / 未ステージングまたはブランチ範囲）をスキャンしてコンテンツの変更を検出し、変更された散文を重複または並行して含む他のレッスン（かつて共有されていたが現在はページ間でコピーされた文章や、VS Code / CLI / App / Cloud の各ハーネスで教えられる同じ概念）を見つけ、重複コピーがずれないようにします。候補ファイルと行範囲を報告しますが、コンテンツの編集はしません。 | `docs/` 配下のレッスンコンテンツを編集した後、コミット / PR 作成の前に、一緒に変更すべき他のページを見つけるとき。 |
| [`validate-site-playwright`](./validate-site-playwright/SKILL.md) | オプションのより深い **ブラウザ** QA: Playwright MCP サーバーをローカルプレビューに対して実行し、ページのレンダリング確認、コンソール / ハイドレーションエラーの検出、画像切れの発見、Starlight コンポーネントのマウント確認を行います。 | ページのレンダリング方法を変更する PR の前に、より深いレンダリング / ビジュアル検証を行うとき。`build-and-verify-docs` を補完します（置き換えるものではありません）。 |

## 汎用スキル

このリポジトリのコンテンツモデルに固有ではない、汎用的なスキルです。

| スキル | 内容 | 使う場面 |
|---|---|---|
| [`make-repo-contribution`](./make-repo-contribution/SKILL.md) | リポジトリの規約やコミットメッセージルールを含め、正しい issue → ブランチ → コミット → PR のワークフローを強制します。 | issue の起票、ブランチ作成、コミット、プッシュ、PR 作成を行うとき。 |
| [`publish-to-pages`](./publish-to-pages/SKILL.md) | プレゼンテーションや Web コンテンツ（PPTX、PDF、HTML、Google スライド）を、公開された GitHub Pages の URL に公開します。 | GitHub Pages を介してプレゼンテーション / HTML アーティファクトを公開・共有するとき。 |
| [`update-markdown-file-index`](./update-markdown-file-index/SKILL.md) | Markdown ファイルのセクションを、フォルダー内のファイルの索引 / テーブルで更新します。 | Markdown ドキュメント内の生成されたファイル索引を最新に保つとき。 |

## スキルを追加する

1. `.github/skills/<skill-name>/SKILL.md` を、`name` と `description` のフロントマター付きで作成します。`description` には、*スキルが何をするか* **と** *いつトリガーするか* を記述します — これによって Copilot が読み込むかどうかを判断します。
2. 本文は手順に焦点を絞ります。スキルがリポジトリのツールを動かす場合は、コマンドを再度ドキュメント化するのではなく、単一の情報源（source of truth）を指します。
3. 上記の適切なテーブルに行を追加します。
4. スキルがリポジトリマップに含めるべき構造的なものであれば、[`../copilot-instructions.md`](../copilot-instructions.md) と [`../../AUTHORING.md`](../../AUTHORING.md) にも記載します。
