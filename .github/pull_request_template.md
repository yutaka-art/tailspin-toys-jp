## 概要

<!-- この PR が何を、なぜ変更するのかを 1〜2 文で説明してください。 -->

## 変更の種類

<!-- 該当するものすべてにチェックを入れてください。 -->

- [ ] ワークショップコンテンツ（レッスンの Markdown、画像）
- [ ] サイトシェル（`website/` の Astro + Starlight ラッパー）
- [ ] Copilot 設定（`.github/copilot-instructions.md`、instructions、agents、skills）
- [ ] リポジトリの整備（CI、dependabot、README、ライセンス）
- [ ] その他:

## 検証

<!-- レビューを依頼する前に、ビルドとリンクチェックが通ることを確認してください。 -->

- [ ] `cd website && rm -rf dist && npm run build` が成功する（目標: 36 ルート × 6 ロケール + リダイレクト 1 = 404 を除いて 217 ページ。ビルドは 404 を含めて 218 個の HTML ファイルを報告。意図的な変更があれば明記）
- [ ] Lychee のリンクチェックが通る:
      `mkdir -p /tmp/lychee-root && ln -sfn $PWD/website/dist /tmp/lychee-root/copilot-workshops && lychee --offline --no-progress --root-dir /tmp/lychee-root 'website/dist/**/*.html'`
- [ ] 変更した外部 GitHub URL を手動でクリックして確認した（lychee はオフラインで実行されるため）

## スクリーンショット

<!-- 見た目に関わる変更の場合のみ。それ以外はこのセクションを削除してください。 -->

## レビュアーへの補足

<!-- レビュアーが知っておくべきこと（不確かな箇所、意図的に先送りしたフォローアップなど）があれば記載してください。 -->
