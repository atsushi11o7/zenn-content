---
name: new-zenn-article
description: Zenn の新規記事を作成する。slug/title/type/emoji を確定させてから zenn new:article で生成する。「新しい記事を書きたい」「記事を作って」「Zenn の記事を始める」と言われたら使う。
---

# Zenn 新規記事作成

各値の制約は CLAUDE.md の「記事執筆のルール」を参照する。本スキルは **決め方の指針** と **実行手順** に集中する。

## Step 1: 記事の概要を確認

ユーザーから以下を聞き取る（分かるものから順に）。

- 何について書くか（技術トピック、解決したい問題）
- 想定読者（初心者 / 中級者 / 同じ技術を使う開発者など）
- 記事の種類（`tech`：手順・解説 / `idea`：考察）

## Step 2: slug / title / emoji を提案

ユーザーの確認を取りながら値を確定する。各フィールドの制約は CLAUDE.md 参照。

### slug の決め方の指針

- 内容を表す英単語の組み合わせを推奨
- 例：`nextjs-app-router-error-handling`
- **公開後の変更は URL が変わるため特に慎重に**

### title の決め方の指針

- 検索キーワードを含む
- 煽りすぎず内容を正確に表す

### emoji の決め方の指針

- 記事内容を象徴するもの（例：`🚀` `🦊` `📝` `⚡`）

## Step 3: 事前チェックして実行

コマンド実行前に以下をチェック。

- `articles/<slug>.md` が既に存在しないか
- リポジトリルート（`package.json` がある場所）にいるか

問題なければ実行：

```bash
npx zenn new:article \
  --slug <slug> \
  --title "<title>" \
  --type <type> \
  --emoji <emoji>
```

## Step 4: topics を埋める

コマンドで指定できないので、生成されたファイルを開いて topics を追加する（CLAUDE.md の制約に従う）。

zenn.dev で実在する人気タグ例：`react` `typescript` `aws` `docker` `nextjs` `python` `go` `nodejs` `vue` `nestjs`

## Step 5: 画像ディレクトリの確認

画像を使う予定があるかユーザーに確認する。

- **使う予定がある**：`images/<slug>/` ディレクトリを作成し、空のままでも Git で追跡できるよう `.keep` を置く（リポジトリ慣習：`articles/.keep`、`books/.keep` と同じ）
- **使わない / 未定**：作成しない（後から必要になった時点で作ればよい、と完了報告で案内する）

## Step 6: 完了報告

- 生成された記事ファイルパスを伝える
- 画像ディレクトリを作成した場合はそのパスも伝える
- 作成しなかった場合は、必要になったら `images/<slug>/` を手動で作るよう案内する
- `published: false` のままであることを明示する
- 公開時は `/publish-article` を使うよう案内する

## 重要な制約

- ユーザーの確認なく勝手にコマンドを実行しない
- slug は必ずユーザー承認を取る（公開後の変更が困難なため）
- topics は実在するタグから選ぶ（独自タグは検索性が下がる）
- `published` を勝手に `true` にしない
- 骨組み（見出し）の自動生成は本スキルの責務外
