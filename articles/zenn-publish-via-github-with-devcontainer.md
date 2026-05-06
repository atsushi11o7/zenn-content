---
title: "Dev Container で Zenn の執筆環境を整えて、GitHub 連携で記事を投稿するまで"
emoji: "🚀"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["zenn", "devcontainer", "docker", "vscode", "github"]
published: true
---

## はじめに

普段は Qiita で記事を書いているのですが、Zenn にも投稿してみることにしました。

Zenn には Web エディタで書く方法と、GitHub リポジトリと連携する方法がありますが、今回は GitHub 連携を選びました。
GitHub でコンテンツを管理できるのは魅力的ですし、単純に使ってみたかったからです。

ただ、ローカルに Node.js などを入れるのは少し抵抗がありました。
環境を汚したくないですし、別のマシンでも同じ状態をすぐ再現したい…。

そこで今回は Dev Container を使って、執筆環境ごとコンテナ化することにしました。

VS Code の Reopen in Container を実行するだけで、Zenn CLI やプレビューがすぐ立ち上がるので、どこでも同じ条件で記事を書けます。

この記事では、その環境構築から実際に Zenn に記事を投稿するまでの流れをまとめていきます。


## 前提

記事の手順を進めるには、以下が必要です。

- **Docker**
- **VS Code**
- **GitHub アカウント**
- **Zenn アカウント**

## 1. GitHub リポジトリを用意して Zenn と連携する

Zenn の公式ドキュメントに [GitHub リポジトリとの連携手順](https://zenn.dev/zenn/articles/connect-to-github) があります。
基本的にはこの流れに沿って進めます。

### GitHub にリポジトリを作成する

記事を管理するための空のリポジトリを GitHub に作成します。今回は `zenn-content` という名前で作りました。

### Zenn のデプロイ設定で連携する

Zenn のダッシュボードのサイドバーから「**GitHub 連携**」を開くと、初期状態では何も連携されていません。「**リポジトリを連携する**」ボタンを押すと、GitHub の認可フローに進みます。

![Zenn の GitHub 連携画面（連携前）](/images/zenn-publish-via-github-with-devcontainer/zenn-deploy-settings-before.jpeg)

認可フローに進むと、**Zenn Connect** という GitHub App に、どのリポジトリへのアクセスを許可するかを選ぶ画面になります。すべてを許可する必要はないので、「**リポジトリのみを選択**」から先ほど作った `zenn-content` リポジトリだけを指定しました。

![Zenn Connect のリポジトリ選択画面](/images/zenn-publish-via-github-with-devcontainer/zenn-connect-install.jpeg)

### 連携を確認する

連携が完了すると Zenn の画面に戻り、「**リポジトリ設定**」タブに連携済みのリポジトリが表示されます。

![Zenn の GitHub 連携画面（連携後）](/images/zenn-publish-via-github-with-devcontainer/zenn-deploy-settings-after.jpeg)

ここまで来れば、あとは記事を push するだけで Zenn 側に反映される状態です。

## 2. Dev Container を構築する

ここからは Dev Container を組み立てていきます。VS Code 側の準備として、まず **Dev Containers 拡張機能** をインストールしておきます。これが入っていれば、後で `Reopen in Container` を実行したときにコンテナ環境を立ち上げられます。

### ディレクトリ構成

リポジトリ直下に `.devcontainer/` ディレクトリを作り、2 つのファイルを置きます。

```text
.devcontainer/
├── Dockerfile
└── devcontainer.json
```

それぞれの役割は次のとおりです。

- `Dockerfile` — コンテナイメージの中身（ベース OS、Node.js、追加で入れるツール）を定義する
- `devcontainer.json` — Dev Container としての挙動（マウント、環境変数、VS Code 拡張など）を設定する

### Dockerfile

`.devcontainer/Dockerfile` は次のような内容です。

```dockerfile
FROM node:24-trixie-slim

# ビルド中の対話プロンプト抑止 & npm のアップデート通知抑止
ENV DEBIAN_FRONTEND=noninteractive \
    npm_config_update_notifier=false

# Claude Code のインストールスクリプト取得用
RUN apt-get update && apt-get install -y --no-install-recommends \
      curl \
 && rm -rf /var/lib/apt/lists/*

WORKDIR /workspace

USER node

# Claude Code
RUN curl -fsSL https://claude.ai/install.sh | bash

# Zenn CLI のプレビューサーバ用
EXPOSE 8000

CMD ["bash"]
```

ベースには `node:24-trixie-slim` を選びました。Zenn CLI を動かすには Node.js が必要なので、Node.js の公式イメージを使用します。`slim` を選ぶことでイメージサイズも抑えられます。

執筆には Claude Code を活用したいので、`RUN curl -fsSL https://claude.ai/install.sh | bash` で Claude Code をインストールします。`curl` は、そのインストールスクリプトを取得するために入れています。

最後の `EXPOSE 8000` は Zenn CLI のプレビューサーバ用です。次の `devcontainer.json` の `forwardPorts` と組み合わせて、ホストのブラウザから見られるようにします。

### devcontainer.json

`.devcontainer/devcontainer.json` の中身は次のとおりです。

```json
{
  "name": "Zenn Writing",
  "build": {
    "dockerfile": "Dockerfile"
  },
  "features": {
    "ghcr.io/atsushi11o7/devcontainer-features/base-utils:2": {
      "locale": "C.UTF-8",
      "timezone": "Asia/Tokyo"
    }
  },
  "workspaceFolder": "/workspace",
  "workspaceMount": "source=${localWorkspaceFolder},target=/workspace,type=bind",
  "containerEnv": {
    "LANG": "C.UTF-8",
    "LC_ALL": "C.UTF-8"
  },
  "postCreateCommand": "test -f package.json && npm install || true",
  "customizations": {
    "vscode": {
      "extensions": [
        "anthropic.claude-code",
        "yzhang.markdown-all-in-one",
        "DavidAnson.markdownlint"
      ]
    }
  },
  "remoteUser": "node",
  "forwardPorts": [8000]
}
```

`build.dockerfile` で先ほどの Dockerfile を指定しています。

`features` には自作の Dev Container Feature を入れています。`ghcr.io/atsushi11o7/devcontainer-features/base-utils:2` は `git` や `tzdata` など、毎回入れたくなる基本的なユーティリティをまとめてセットアップしてくれる Feature です。同じ構成にする必要はないので、必要なツールはご自身で別途入れてください。

`workspaceFolder` と `workspaceMount` で、ホスト側のリポジトリを `/workspace` にバインドマウントしています。コンテナ内で編集した内容がそのままホスト側に反映されます。

`postCreateCommand` の `test -f package.json && npm install || true` は、`package.json` が存在するときだけ `npm install` を走らせる作りです。Zenn CLI を入れる前の初回ビルドでもエラーにならないようにしています。

`customizations.vscode.extensions` で、コンテナに自動で入る VS Code 拡張を指定しています。

- `anthropic.claude-code` — Claude Code（執筆補助）
- `yzhang.markdown-all-in-one` — Markdown のショートカットや TOC 生成
- `DavidAnson.markdownlint` — Markdown の表記ブレや空行ミスを警告

### コンテナを起動する

ここまで揃えたら、リポジトリを VS Code で開きます。Dev Containers 拡張が入っていれば、画面右下に「Reopen in Container」のダイアログが出るので、それを選ぶだけでコンテナのビルドと起動が走ります。表示されない場合は、コマンドパレット（`F1` または `Ctrl/Cmd + Shift + P`）から「Dev Containers: Reopen in Container」を実行できます。

完了するとコンテナ内に VS Code がアタッチされ、ターミナルもそのままコンテナ側に切り替わります。以降の作業はすべてコンテナ内で進められます。

## 3. Zenn CLI をセットアップする

コンテナ内のターミナルから、Zenn CLI を入れていきます。まず `package.json` を作り、`zenn-cli` を依存として追加します。

```bash
npm init -y
npm install zenn-cli
```

続けて Zenn CLI の初期化コマンドを実行します。

```bash
npx zenn init
```

これで `articles/` と `books/` の雛形ディレクトリが作られます。記事は `articles/` の中に Markdown で書いていきます。

## 4. 記事を書いてプレビューする

### 記事を新規作成

`npx zenn new:article` で記事のテンプレートを作成します。今回の記事は次のように実行しました。

```bash
npx zenn new:article \
  --slug zenn-publish-via-github-with-devcontainer \
  --title "Dev Container で Zenn の執筆環境を整えて、GitHub 連携で記事を投稿するまで" \
  --type tech \
  --emoji 🚀
```

それぞれのオプションは以下のとおりです。

- `--slug` — 記事の URL に使われる識別子。`articles/<slug>.md` のファイル名にもなる
- `--title` — 記事タイトル
- `--type` — `tech`（技術記事）か `idea`（アイデア記事）
- `--emoji` — 記事のサムネイルに表示される絵文字

実行すると `articles/<slug>.md` が作られ、先頭には次のような frontmatter が入っています。

```markdown
---
title: "Dev Container で Zenn の執筆環境を整えて、GitHub 連携で記事を投稿するまで"
emoji: "🚀"
type: "tech"
topics: []
published: false
---
```

`topics` には任意のタグ（最大 5 個）を、`published` は公開タイミングまで `false` のままにしておきます。あとは Markdown で本文を書いていくだけです。

### プレビューサーバを起動

書きながら見た目を確認するには `npx zenn preview` でローカルサーバを立ち上げます。

```bash
npx zenn preview
```

`devcontainer.json` の `forwardPorts: [8000]` が効いているので、ブラウザで `http://localhost:8000` を開くとプレビューが表示されます。VS Code 側でもポート転送が検知されて通知が出るので、そこから直接ブラウザを開くこともできます。

![Zenn のプレビュー画面](/images/zenn-publish-via-github-with-devcontainer/preview-screen.jpeg)

## 5. push して Zenn に反映する

書き終わったら、frontmatter の `published` を `true` に変更します。あわせて `topics` に記事の内容に合うタグを最大 5 個まで入れておきます。

```markdown
---
title: "Dev Container で Zenn の執筆環境を整えて、GitHub 連携で記事を投稿するまで"
emoji: "🚀"
type: "tech"
topics: ["zenn", "devcontainer", "docker", "vscode", "github"]
published: true
---
```

あとは GitHub に push するだけです。

```bash
git add articles/zenn-publish-via-github-with-devcontainer.md
git commit -m "Publish article"
git push
```

push をトリガーに Zenn 側のデプロイが自動で動き、しばらくすると Zenn のダッシュボードやプロフィールページに記事が公開されます。反映状況は Zenn のデプロイ履歴で確認できます。

## おわりに

ここまで、Dev Container と GitHub 連携で Zenn の執筆環境を作って、最初の記事を投稿するまでをまとめてきました。

そして、この記事が、ここのプロセスで作った環境の中で書かれた最初の 1 本です。

執筆には Claude Code を活用しています。Agent や Skill 機能を使って、節ごとの下書きの作成、公開前の機械チェック、編集者視点でのレビューを進めました。この執筆ワークフローについては、次の記事で改めて解説しようと思います。
