# TripLink システム構成と開発手法（HonoをWorkersにデプロイ、Nextと分離）

## 🎯 システム概要

**目的**  
- ユーザーが旅行計画を作成・共有・共同編集  
- PWAとしてオフラインでも利用可能  
- 同じ旅程を共有するメンバー間で差分同期・マージを実装  
- 将来的にAIワークフロー（dify MCP）を統合予定  

**特徴**  
- ローカルファースト PWA  
- IndexedDB ⇄ SQLite (libSQL / Turso)  
- Hono APIをCloudflare WorkersにデプロイしてクラウドDBに同期  
- Tursoのみ使用（D1は併用しない）  
- Clerkによるユーザー認証・チーム招待・同期管理  
- Node.js 20 + pnpm + Next.js (App Router)  
- Windows/Linux混在チームでも DevContainer で統一環境  
- モノレポ管理でNextプロジェクトとHono APIを分離

---

## 🏗️ 全体構成図

```
[ブラウザ / PWA]
  └─ IndexedDB (オフラインキャッシュ)
       │
       ▼
[Next.js (App Router)]
  ├─ SQLite (local libSQL)
  ├─ Clerk (ユーザー認証 & 同期制御)
  └─ Hono API (Cloudflare Workers)
        │
        ▼
[Turso Cloud Database (libSQL)]
  - 差分同期・マージ戦略
  - ローカル SQLite と双方向同期
```

---

## 🔹 各コンポーネントの役割

| コンポーネント | 役割 | 特徴 |
|----------------|------|------|
| IndexedDB | PWAのオフライン保存 | ブラウザ上のローカルDB。オフライン編集を保持 |
| Next.js (App Router) | UI / ページ遷移 / SQLiteラッパー | Node.js 20 + pnpm。SQLite(libSQL)とHono APIを仲介 |
| SQLite (libSQL) | ローカルDB | libSQL互換。IndexedDBとほぼ同一SQLで扱える |
| Hono API | APIサーバー / データ同期 | Cloudflare Workers上で稼働。クライアントとTursoを仲介、衝突解決や差分マージを担当 |
| Clerk | 認証・ユーザー管理 | OAuth/SSO対応。セッション管理、アクセス制御 |
| Turso (libSQL) | クラウドDB / 共有データ | オフライン同期対応。差分マージ、マルチデバイス対応 |

---

## ⚡ 特徴・設計ポイント

1. **ローカルファーストPWA**  
   - ユーザーはオフラインでも編集可能  
   - IndexedDBとSQLite(libSQL)で即時保存

2. **双方向同期・差分マージ**  
   - Hono APIがローカルSQLite ↔ Turso Cloudを同期  
   - 衝突時はユーザーに解決を委ねるUIを用意

3. **ユーザー認証・同期制御**  
   - Clerkでログイン・チーム招待・アクセス制御  
   - ユーザーごとの同期対象を管理

4. **Edge対応**  
   - Hono APIはCloudflare Workers上で稼働  
   - Turso libSQLはEdge/Node両方で接続可能

5. **チーム開発用 DevContainer**  
   - Node.js 20 + pnpm  
   - Windows/Linux混在チームでも同一環境  
   - 開発用リポジトリをコンテナにアタッチ可能  

6. **モノレポ管理**  
   - Next.jsプロジェクトとHono APIは同一リポジトリ内で分離  
   - DevContainerリポジトリとは別管理

---

## 🗂️ 推奨ディレクトリ構成（モノレポ）

```
triplink-monorepo/           # モノレポ本体
├── apps/
│   ├── web/                 # Next.js (PWA/UI)
│   │   ├── app/
│   │   ├── db/
│   │   ├── components/
│   │   ├── hooks/
│   │   ├── public/
│   │   ├── styles/
│   │   ├── utils/
│   │   ├── .env
│   │   └── package.json
│   └── hono-api/            # Hono API (Cloudflare Workers)
│       ├── src/
│       ├── package.json
│       ├── wrangler.toml
│       └── .env
├── libs/                    # 共通ライブラリ
│   ├── db/
│   ├── utils/
│   └── types/
└── package.json              # モノレポ用
└── pnpm-workspace.yaml       # ワークスペース管理
```

---

## 💡 開発手法

1. **DevContainer を使った統一開発環境**  
   - Node.js 20 + pnpm  
   - Windows/Linux混在チームでも同じ環境  
   - 開発用リポジトリをコンテナにマウントして作業

2. **ローカルファースト開発**  
   - IndexedDB によるオフライン保存  
   - ローカル SQLite (libSQL) で即時反映  
   - Turso クラウドに差分同期

3. **API抽象化**  
   - Hono APIがフロントと Turso の間を仲介  
   - 衝突解決やマージ戦略を API 層で実装

4. **認証とチーム管理**  
   - Clerk によるログイン・チーム招待  
   - 同期対象やアクセス権限を制御

5. **モノレポ運用**  
   - `apps/web` と `apps/hono-api` を分離  
   - 共通ライブラリは `libs/` に集約  
   - pnpm workspace で依存管理

6. **将来拡張**  
   - AIワークフロー（dify MCP）と接続可能  
   - Cloudflare WorkersでHono APIを高速レスポンス
