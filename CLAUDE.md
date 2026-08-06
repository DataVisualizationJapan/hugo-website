# CLAUDE.md

Data Visualization Japan の Hugo サイト。YouTube 講演動画の書き起こし記事を蓄積している。

## PR 運用（最重要）

**このリポジトリは PR が数分でマージされる。**実績では作成から 1分39秒〜9時間、中央値で10分程度。
「PR を出した → まだ開いているはず」という前提で動くと必ず事故る。

- **プッシュ前に必ず `git fetch origin main` してマージ済みか確認する。**
  ブランチ名を使い回しているため、マージ済み履歴の上にコミットを積んでも
  `git push` は成功し、git からは何の警告も出ない。無症状で進行する。
- **マージ済みの PR は再利用できない。**追加作業は main からブランチをやり直し、
  新規 PR として出す（`git rebase origin/main` → `git push --force-with-lease`）。
- **PR 本文を書く・更新する直前に、必ず PR の state と commits を取得し直す。**
  記憶で書かない。過去に「まだプッシュしていない変更」を本文に書いてしまった事故がある。
- PR 本文は実際にマージされる差分だけを説明する。作業途中で追加コミットしたら本文も直す。

`.git/hooks/pre-push` に、マージ済み履歴の上へのプッシュを止めるフックを置いている
（コミットされないので、環境を作り直したら再作成が必要）。

## テーマ

`themes/stack` は **git submodule**。直接編集しない。上書きはサイト側で行う。

- テンプレート → `layouts/`（テーマと同じパスに置けば上書きされる）
- スタイル → `assets/scss/custom.scss`
- 記事一覧のスタイルは Stack 標準の default / compact / tile に加えて
  `article-list--card`（`layouts/partials/article-list/card.html`）を自作している

Stack の CSS 変数（`--card-background` / `--shadow-l1` / `--shadow-l2` /
`--card-border-radius` / `--card-padding` / `--accent-color`）だけを使えば
ライト／ダーク両対応になる。色を直書きしない。

`html { font-size: 62.5% }` なので **1rem = 10px**。

## 記事の規約

`content/posts/<slug>/index.md`（ページバンドル）。front matter は TOML（`+++`）。

```toml
+++
title = "登壇者名／講演タイトル"
slug = "..."
date = "YYYY-MM-DD"          # イベント開催日

author = "Data Visualization Japan"
speakers = ["山田太郎"]        # taxonomy キー。スペースなし

event = "2022-meetup"        # content/events/<id> のフォルダ名

videoId = "..."              # YouTube 動画ID
image = "cover.jpg"          # YouTube サムネイル（同じディレクトリに配置）

categories = ["meetup", "2022"]   # 2つ目は「イベントの開催年」
tags = [...]                      # 既存のタグ語彙に合わせる
summary = "検索結果・SNSシェア用の1〜2文"
+++
```

本文冒頭にイベント名の1文、`<!--more-->`、`---`、`{{< youtube <videoId> >}}` を置く。

**イベントページとの相互リンクが必須。**`content/events/<id>/index.md` に追記する。

```toml
[[sessions]]
  title = "..."
  speaker = "山田 太郎"     # 表示用。姓名の間にスペースあり
  videoId = "..."
  post = "<slug>"          # ← ディレクトリ名ではなく slug
```

**落とし穴**: `hugo.toml` の permalinks が `posts = '/:slug/'` なので、
`session.post` はディレクトリ名ではなく **slug** と一致させる必要がある。
両者が食い違っている記事が過去にあった。

`categories[1]` は**記事の date の年ではなくイベントの開催年**。
新しい年を使うときは `data/category_hierarchy.toml` にも追加する。

## 書き起こしの方針

`yt-dlp` の自動字幕（ASR）を整文する。誤変換が多いので文脈から直す。

- **特定できない固有名詞は書かない。**推測で人名を書くくらいなら伏せる。
  誤った人名を出す方が、名前がないより有害。
- 見出しを立て、重要な箇所を太字にし、読める日本語にする。逐語ではない。
- 数字・団体名は可能な限り裏を取る。ASR は数字をよく間違える。

### yt-dlp

```bash
yt-dlp --skip-download --write-auto-subs --sub-langs "ja-orig,ja" --sub-format json3 \
  --extractor-args "youtube:player_client=android_vr" \
  --sleep-requests 8 --retries 3 -o "/tmp/yt/%(id)s.%(ext)s" <URL>
```

- **`Sign in to confirm you're not a bot` はレート制限であって「字幕なし」ではない。**
  取得済みの動画で再試行する対照実験で切り分ける。数分〜10分待てば回復する。
- サムネイルは **`img.youtube.com`** から取る。`i.ytimg.com` はプロキシに拒否される。
  `maxresdefault.jpg`（1280x720）→ なければ `hqdefault.jpg`。

## 検証

ビルドは Netlify と同じバージョンで確認する（`netlify.toml` の `HUGO_VERSION`、現在 0.149.0 extended）。

コミット前に確認すること:

- front matter が TOML としてパースできる
- 必須フィールドが揃っている／`slug` が一致している
- イベントページとの相互リンクが slug で張れている
- `categories[1]` がイベントの開催年と一致し、`category_hierarchy.toml` に存在する
- 整文中に紛れ込んだ非日本語文字（キリル文字など）がない — 過去に混入した事故がある

**完了報告をするときは、必ずチャンネル一覧と突き合わせる。**
`yt-dlp --flat-playlist` で全動画IDを取り、リポジトリの `videoId` と照合する。
セッション内で扱った本数を数えただけで「全部終わった」と報告して、
実際は半分近く残っていた事故がある。
