# ベースイメージ
FROM node:20

# 必須ツール
RUN apt-get update && apt-get install -y \
    git \
    curl \
    sqlite3 \
    && rm -rf /var/lib/apt/lists/*

# pnpm インストール
RUN npm install -g pnpm@8.10.0

# 作業ディレクトリ
WORKDIR /workspace

# Turso CLI インストール
RUN curl -sSf https://get.turso.io/cli.sh | sh

# VS Code 用の Dev Container 推奨設定
ENV NODE_ENV=development
