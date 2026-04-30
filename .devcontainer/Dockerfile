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