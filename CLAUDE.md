# このリポジトリについて

Zenn の記事を GitHub 連携で公開するためのリポジトリです。Dev Container で執筆環境を構築しています。

## ディレクトリ構成

- `articles/` — 記事の Markdown ファイル（1 記事 = 1 ファイル、ファイル名が slug）
- `books/` — 本（複数記事をまとめたもの）の Markdown
- `images/` — 記事内で参照する画像（**画像を使う記事のみ** スラッグごとにサブディレクトリを作る）
  - 例：`images/zenn-publish-via-github-with-devcontainer/preview.png`
- `.devcontainer/` — Dev Container 設定
- `.claude/skills/` — このリポジトリ専用のスキル定義
- `.claude/agents/` — このリポジトリ専用のエージェント定義

## 記事執筆のルール

このリポジトリの全スキル・全エージェントはこのルールに従う。

### 文体・表記

- **「です・ます調」で統一**
- 半角英数字の前後には半角スペースを入れる
  - ✅ 「Dev Container を使う」
  - ❌ 「Dev Containerを使う」
- カタカナ語・固有名詞の表記は以下に統一
  - **Dev Container**（DevContainer / devcontainer ではなく。ファイル名や設定キーは除く）
  - **GitHub**（Github / github ではなく）
  - **Zenn**（zenn-cli のようなツール名は小文字のままで OK）
  - **VS Code**（VSCode / vscode ではなく）

### Markdown 記法

- 本文の見出しは `##` から始める（`#` は frontmatter の `title` が担うので使わない）
- コードブロックには必ず言語を指定する（` ```bash`、` ```json`、` ```dockerfile` など）
- 画像は **絶対パス** で参照する
  - 例：`![devcontainer.json の編集画面](/images/zenn-publish-via-github-with-devcontainer/devcontainer-config.png)`
- 画像にキャプションを付けたい場合は、画像の直下に `*キャプション*`

### slug（ファイル名）

- 記事ファイル名 `articles/<slug>.md` の `<slug>` 部分が Zenn の記事 URL になる
- `a-z`、`0-9`、ハイフン `-`、アンダースコア `_` のみ
- 12〜50 字
- **公開後に変更すると Zenn 上の URL が変わり被リンクが切れる**ため、初回決定時に慎重に決める

### frontmatter

| フィールド | 制約 |
| --- | --- |
| `title` | 70 文字以内（理想は 30〜50 字） |
| `emoji` | 1 文字 |
| `type` | `tech` または `idea` |
| `topics` | 半角英小文字のみ、最大 5 個、実在する人気タグを優先 |
| `published` | 真偽値（公開状態の制御は下記ワークフロー参照） |

## ワークフローと対応スキル

| フェーズ | スキル / エージェント | 補足 |
| --- | --- | --- |
| 新規作成 | `/new-zenn-article` | slug/title/type/emoji を確定して雛形生成 |
| 執筆 | `/draft-section` | 見出し + メモから節の下書きを生成（任意。手書きでも可） |
| プレビュー | （なし） | `npx zenn preview` で起動。forwardPorts: 8000 で自動的にブラウザに開く |
| 機械チェック | `/pre-publish-check` | CLAUDE.md ルールへの **適合** を機械的に判定（表記揺れ、frontmatter エラー、画像パス不存在など） |
| 編集レビュー | Agent: `zenn-content-reviewer` | **内容の質** を編集者視点で評価（論理展開、構成、読者適合、文体の読みやすさなど） |
| 公開 | `/publish-article` | `published: true` 化 → commit → push |

ユーザーから「新しい記事を書きたい」「公開前にチェックして」「Zenn に公開して」のような依頼があれば、コマンドを自分で組み立てるのではなく対応するスキル / エージェントを呼び出す。

`published` の `false` → `true` 切り替えは `/publish-article` 経由のみで行うこと（手動編集しない）。

## 環境

Dev Container 内で作業するのが前提。VS Code で `Reopen in Container` で起動する。
