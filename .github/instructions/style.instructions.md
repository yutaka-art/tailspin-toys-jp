---
description: 'Tailwind CSS v4 のスタイリングパターンとダークテーマのガイドライン'
applyTo: '**/*.{astro,css}'
---

# Tailwind CSS ガイドライン

## Tailwind CSS v4 の設定

このプロジェクトでは、`@tailwindcss/vite` プラグインを通じて Tailwind CSS v4.1.14 を使用しています。

### グローバル CSS のセットアップ

- `global.css` で Tailwind をインポートすること: `@import "tailwindcss";`
- 独立した `tailwind.config.js` ファイルは使用しない
- 設定は Vite プラグインを通じて行う

## ダークテーマのスタイリング

すべての UI コンポーネントは、必ずダークテーマの配色を使用すること:

### カラーパレット

- 背景色: `bg-slate-800`、`bg-slate-900`、`bg-slate-950`
- 文字色: `text-slate-100`、`text-slate-200`、`text-slate-300`
- ボーダー色: `border-slate-700`、`border-slate-600`
- ホバー／フォーカス状態向けのアクセントカラー

### よく使うパターン

- カードやコンテナ: `bg-slate-800 rounded-xl p-6 shadow-lg`
- ホバー効果: `hover:bg-slate-700 transition-colors duration-200`
- ボーダー: `border border-slate-700`
- 視覚的なアクセントとしてのグラデーション: `bg-gradient-to-br from-slate-800 to-slate-900`
- 背景のぼかし効果: `backdrop-blur-sm bg-slate-900/50`

### レスポンシブデザイン

- レスポンシブ用の接頭辞を使用すること: `sm:`、`md:`、`lg:`、`xl:`
- モバイルファーストのアプローチを採用する
- あらゆる画面サイズで可読性を確保する

## ユーティリティクラス

- 可能な限りカスタム CSS よりユーティリティクラスを優先すること
- レイアウト、余白、色、タイポグラフィなど、意味のあるまとまりでグループ化する
- ユーティリティの組み合わせは、読みやすく保守しやすい状態に保つ

## モダンな UI パターン

- 角丸: `rounded-lg`、`rounded-xl`、`rounded-2xl`
- なめらかなトランジション: `transition-all duration-200 ease-in-out`
- 奥行きを出すためのシャドウ: `shadow-md`、`shadow-lg`、`shadow-xl`
- アクセシビリティのためのフォーカス状態: `focus:ring-2 focus:ring-blue-500`
