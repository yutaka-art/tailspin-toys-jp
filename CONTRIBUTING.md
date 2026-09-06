# コントリビューション

**Copilot Workshops** へのコントリビューションに関心をお寄せいただきありがとうございます。このリポジトリは、ワークショップのコンテンツ（Markdown のソースと、それを公開する Astro + Starlight サイト）を管理しています。

## 行動規範

このプロジェクトは [コントリビューター行動規範（Contributor Code of Conduct）](./CODE_OF_CONDUCT.md) のもとで公開されています。参加することで、その条項を遵守することに同意したものとみなされます。

コントリビューションは、[プロジェクトのオープンソースライセンス](./LICENSE) のもとで公開されます。

## まず最初に読むもの

**コンテンツを執筆・編集** したい場合は、まず [`AUTHORING.md`](./AUTHORING.md) をご覧ください。全体像の考え方、ファイル構成、レッスンや画像を追加・編集するための手順、ローカルプレビューのワークフロー、スタイル規約について説明しています。

## プルリクエストの送信

1. リポジトリを [フォーク](https://github.com/github-samples/copilot-workshops/fork) してクローンします。
2. トピックブランチを作成します（`git checkout -b my-change`）。
3. 変更を加えます。PR は 1 つの論理的な変更に絞り、焦点を明確に保ってください。
4. フォークにプッシュし、[プルリクエストを作成](https://github.com/github-samples/copilot-workshops/compare) します。
5. CI とレビューを待ちます。

## マージ前に

PR で CI（`pages.yml`）が成功している必要があります。CI では次が実行されます:

- **`pages.yml` ビルド** — `npm run build`（Astro サイトのビルド）。
- **Lychee** — ビルドされた `website/dist/` に対するオフラインリンクチェック。

プッシュする前に、[AUTHORING.md → Building and verifying](./AUTHORING.md#building-and-verifying) で説明されている一連のローカル検証（クリーンビルド、ページ数チェック、lychee のリンクチェック）を実行してください。

## コミットメッセージ

Conventional Commits のプレフィックスの使用を推奨します: `docs:`、`chore:`、`fix:`、`ci:`、`feat:`。

AI の支援を受けた場合は、`Co-authored-by` トレーラーを含めてください:

```
Co-authored-by: Copilot <223556219+Copilot@users.noreply.github.com>
```

## 参考リンク

- [オープンソースへのコントリビューション方法](https://opensource.guide/how-to-contribute/)
- [プルリクエストの使い方](https://help.github.com/articles/about-pull-requests/)
