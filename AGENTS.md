# AGENTS.md

## ポリシー

- 日本語で応答してください。


## 作業メモ

- カテゴリー系レイアウトをカスタマイズ済み。テーマ差し替え時は以下のファイルを確認すること。
  - `layouts/categories/terms.html`: トップレベルカテゴリを `data/category_hierarchy.toml` から読み込み表示。
  - `layouts/categories/list.html`: サブカテゴリ一覧＋本文表示、タームの固有情報を扱うテンプレート。
  - `assets/scss/custom.scss`: `.section-content.article-content` 周りで余白・画像・リストのスタイルを上書き。
  - `data/category_hierarchy.toml`: カテゴリ階層を定義。新テーマでも同様のロジックを使う場合は移植する。
- 左サイドバーのメニューは `hugo.toml` の `[menu.main]` で管理。トップ/検索/記事一覧/カテゴリー/タグを登録済みで、順序は weight で調整。
- 右サイドバーのウィジェットは `params.widgets.homepage` と `params.widgets.page` にカテゴリ/タグを設定済み。他のウィジェットを追加するならここを編集。
- カテゴリー・タグ一覧ページはテーマ標準の `themes/stack/layouts/partials/article-list/compact.html` を利用中。カスタマイズする場合は `layouts/partials/article-list/compact.html` にコピーして十分検証する。
- 検索ページ `content/page/search.md` は `outputs = ['HTML', 'JSON']` を設定済み。検索フォームを使う際は `/search/` の動作確認を行う。
