---
name: publish-article
description: Zenn 記事を公開する。published を true に変更し、commit と push を行って Zenn の自動デプロイを発火させる。「公開して」「published true にして」「publish」「Zenn に出す」と言われたら使う。
---

# Zenn 記事の公開

## 前提

「公開可能な状態」の記事に対して使うスキル。本文の修正や校正は **`/pre-publish-check`** で済ませておくこと。

## Step 1: 対象記事の特定

ユーザーが対象を明示していなければ、`articles/` 内で `published: false` の記事一覧を提示して選んでもらう。

## Step 2: pre-publish-check の確認

ユーザーに以下を確認する。

> 「`/pre-publish-check` を実行済みですか？ まだなら先に実行することを推奨します。」

未実行なら強く推奨する（ただしユーザーが「不要」と判断すれば尊重する）。

## Step 3: 公開意思の最終確認

公開後の影響を伝えた上で、明示的な承認を得る。

> 以下の記事を公開します。よろしいですか？
>
> - タイトル：<title>
> - スラッグ：<slug>
> - 公開後 URL：https://zenn.dev/<username>/articles/<slug>
>
> 公開後にスラッグを変更すると URL が変わり、被リンクが切れます。

## Step 4: 変更ファイルの事前確認

`git status` を実行し、コミット予定のファイルを把握する。

- 対象記事 Markdown：`articles/<slug>.md`
- 関連画像：`images/<slug>/` 配下（未コミット分があれば追加）
- 想定外のファイル変更（CLAUDE.md / 他の記事など）があれば、ユーザーに確認する

## Step 5: published を true に変更

frontmatter の `published: false` を `published: true` に書き換える。

## Step 6: コミットとプッシュ

```bash
git add articles/<slug>.md images/<slug>/
git commit -m "Publish: <title>"
git push
```

- コミットメッセージは `Publish: <title>` 形式
- `<title>` が長く 1 行に収まらない場合は要約し、本文に full title を書く
- 画像が `images/<slug>/` にない場合は `git add` から外す

## Step 7: 反映確認の案内

> Zenn 側で自動デプロイされます（通常 1〜2 分）。
> https://zenn.dev/<username>/articles/<slug> で公開を確認してください。
>
> 反映されない場合：
> - GitHub の Actions タブでデプロイのログを確認
> - Zenn ダッシュボードの「デプロイ履歴」を確認

## 重要な制約

- ユーザーの**明示的な承認**なしに `published: true` への変更や push を行わない
- `/pre-publish-check` 未実行の場合は強く推奨する（強制はしない）
- `git push --force` などの破壊的操作は絶対に使わない
- 公開済みの記事を非公開に戻す場合：`published: false` に戻して push すれば下書きに戻る（ただし URL は失効する）
