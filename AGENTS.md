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

## Phase 1（イベント / 記事テンプレ / CTA）で追加した構成

- 記事作成の運用フローは `docs/content-workflow.md` に集約。記事・イベントの作り方はまずここを参照。
- 記事アーキタイプ `archetypes/posts.md`：`event` / `videoId` / `speakers` / `summary` などを含む定型。`hugo new posts/<slug>/index.md` で展開。
- イベントセクション `content/events/`：1イベント=1ページバンドル。`eventDate`（実開催日時）で開催予定/過去を自動判定。`sessions` 配列に各講演（`title`/`speaker`/`videoId`/`post`）を持つ。アーキタイプは `archetypes/events.md`。
  - 一覧テンプレ `layouts/events/list.html`：`eventDate` を `now` と比較して「開催予定」↑ /「過去」↓ に自動分割。
  - 個別テンプレ `layouts/events/single.html`：概要＋申込リンク＋セッション一覧（YouTubeサムネ＋書き起こしリンク）。
  - `[[menu.main]]` に `events`（weight 25）を追加済み。
- 次回イベント CTA `layouts/partials/cta/next-event.html`：events から最も近い未来を自動抽出。引数 `"bar"`（全ページ下部バー）/ `"block"`（記事末尾）。未来イベントが無ければ非表示。
  - 全ページ表示は `layouts/partials/footer/custom.html`（テーマの空フック）から `"bar"` を呼ぶ。
  - 記事末尾は `layouts/partials/article/components/footer.html` の末尾で `"block"` を呼ぶ。
  - スタイルは `assets/scss/custom.scss` の「Data Visualization Japan — イベント / CTA」ブロック（テーマの CSS 変数を使用）。
- ソーシャル導線は `hugo.toml` の `[[menu.social]]`（YouTube / Facebook / connpass / RSS）。アイコンが無いものは `assets/icons/<name>.svg` に Tabler 形式の SVG を追加（`brand-youtube` / `brand-facebook` / `calendar-event` を追加済み）。`layouts/partials/helper/icon.html` は SVG が無いとビルドエラーになる点に注意。

## Phase 2（構造化データ / 登壇者 / RSS）で追加した構成

- 構造化データ（JSON-LD）は `layouts/partials/head/custom.html`（テーマの head フック）で出力。
  - ホーム → `Organization`（`[[menu.social]]` の外部URLを `sameAs` に集約。RSS・相対URLは除外）。
  - 記事（`videoId` 有）→ `VideoObject`。イベント個別ページ → `Event`（`format` に「オンライン」を含めば VirtualLocation + OnlineEventAttendanceMode）。
  - `<script type="application/ld+json">{{ ... | jsonify | safeJS }}</script>` の `safeJS` は必須。付けないと `<script>` 内で二重エンコードされる。
  - 同ファイル末尾で RSS フィードの autodiscovery `<link rel="alternate">` も出力。
- 登壇者タクソノミー：`hugo.toml` の `[taxonomies]` に `speaker = "speakers"` を追加（category / tag も併記が必要）。記事 front matter は `speakers = [...]`。`/speakers/` と `/speakers/<名前>/` はテーマ標準のタクソノミーテンプレートで自動生成。メニューに `speakers`（weight 45, icon `user`）。
  - 方針：`tags` は「トピック」、`speakers` は「登壇者」。人名は tags ではなく speakers に入れる（既存2記事は移行済み）。
