# Comic Market Kannai リポジトリ解説書

コミックマーケット準備会 **館内担当 Web チーム** が保有する GitHub Organization（[Comic-Market-Kannai](https://github.com/Comic-Market-Kannai)）のリポジトリ群を、新規メンバーの立ち上がりと全体把握のために解説する。

> 最終更新: 2026-07-26

---

## 1. 全体像

館内担当のシステムは、大きく **5 系統** に分かれる。

| 系統 | 目的 | 主なリポジトリ | 状態 |
| --- | --- | --- | --- |
| 🎫 受付システム | サークルの出欠・受付管理 | `kannai-attendance-server(-v2)` / `kannai-attendance-client(-v2)` | v1 → v2 リプレイス中 |
| 🔐 認証基盤 | 配下サービスの統一認証（SSO） | `AuthGate` | 開発中 |
| 🎙️ ネゴ管理 | ネゴペーパーのデジタル化 | `NeMSys` | 開発中 |
| 🌐 館内 Web | 館内担当向け業務 Web | `kannai-web` | 稼働中 |
| 🚦 混雑・その他 | 混雑管理・補助ツール | `CrowdedSystem` / `download_content_helper` | 構想段階 |

系統をまたぐ関係として、**AuthGate が受付システム・NeMSys 等の配下サービスに SSO を提供する**構図を想定している（AuthGate README: 「配下サービス（現状 2 サービス）に対して統一認証を提供」）。

```
                 ┌─────────────┐
                 │   AuthGate  │  SSO / 組織・権限
                 └──────┬──────┘
          ┌─────────────┼─────────────┐
          ▼             ▼             ▼
   受付システム      NeMSys       （今後の配下）
   (attendance)   (ネゴ管理)
```

---

## 2. リポジトリ一覧

| リポジトリ | 説明 | 主言語 | 公開 | 状態 |
| --- | --- | --- | --- | --- |
| [`kannai-web`](https://github.com/Comic-Market-Kannai/kannai-web) | 館内担当向け業務 Web（受付・各種レポート） | PHP / HTML | private | 稼働中 |
| [`NeMSys`](https://github.com/Comic-Market-Kannai/NeMSys) | ネゴ管理システム（デジタルネゴペ） | TypeScript | private | 開発中 |
| [`AuthGate`](https://github.com/Comic-Market-Kannai/AuthGate) | スタッフ向け SSO 認証基盤 | Python / Svelte | private | 開発中 |
| [`kannai-attendance-server-v2`](https://github.com/Comic-Market-Kannai/kannai-attendance-server-v2) | 受付サーバー v2（FastAPI リプレイス） | Python | private | 開発中 |
| [`kannai-attendance-client`](https://github.com/Comic-Market-Kannai/kannai-attendance-client) | 受付クライアント（Angular） | TypeScript | private | 稼働中 |
| [`kannai-attendance-server`](https://github.com/Comic-Market-Kannai/kannai-attendance-server) | 受付サーバー v1（Node.js + Redis） | JavaScript | private | 旧版（保守） |
| [`kannai-attendance-client-v2`](https://github.com/Comic-Market-Kannai/kannai-attendance-client-v2) | 受付クライアント v2 試作（Svelte） | Svelte | private | 🗄️ Archived |
| [`CrowdedSystem`](https://github.com/Comic-Market-Kannai/CrowdedSystem) | 混雑管理システム | — | private | 空（構想段階） |
| [`download_content_helper`](https://github.com/Comic-Market-Kannai/download_content_helper) | ダウンロードコンテンツ登録用ヘルパー | — | private | 空（構想段階） |
| [`.github`](https://github.com/Comic-Market-Kannai/.github) | Organization プロフィール・共通ドキュメント（本書） | — | public | — |
| [`demo-repository`](https://github.com/Comic-Market-Kannai/demo-repository) | GitHub 機能のデモ用 | HTML | private | サンプル |

---

## 3. 系統別解説

### 3.1 🎫 受付システム（Attendance）

サークル参加者の**出欠確認・ホール単位の出欠管理・QR コード検索**を行う、館内担当の中核システム。サーバー / クライアントとも v1 → v2 のリプレイスが進行中。

#### 構成の変遷

```
サーバー:  kannai-attendance-server (v1)     →  kannai-attendance-server-v2
           Node.js + Express + Redis            Python + FastAPI

クライアント: kannai-attendance-client (Angular 14)  ← 現行
              kannai-attendance-client-v2 (Svelte)   ← 試作・Archived（採用見送り）
```

#### `kannai-attendance-server`（v1・旧版）
- **技術**: Node.js v14.17.6+ / Express / Redis
- **仕組み**: 起動時に Google Spreadsheet（GSS）→ GAS 経由でサークル一覧をオンメモリにロード。出欠情報は Redis に `hset` で保持
- **ロール**: ①ユーザー（自ホールの出欠取得）②ホール管理者（自ホールのレポート作成）③管理者（全ホールアクセス・レポート）
- **v2 移行の背景**（server-v2 README より）: 「Redis の突然の死」「Express 重すぎ」「Node のバージョン追従がつらい」「エンジニア不足」等を理由に、後方互換性と保守要員確保のため FastAPI へ移行

#### `kannai-attendance-server-v2`（現行サーバー）
- **技術**: Python 3.11 推奨 / FastAPI / pipenv
- **起動**: `pipenv sync` → `pipenv run start`（`http://127.0.0.1:8000`、`/docs` に自動 API ドキュメント）
- **開発ルール**: main への直接 push 禁止、必ずプルリクエスト（宛先 `@m96-chan`）

#### `kannai-attendance-client`（現行クライアント）
- **技術**: Angular 14 / TypeScript / ng-bootstrap / dayjs / jsqr（QR スキャン）
- **機能**: サークル出欠管理（一覧・詳細）、ホール別出欠状況、QR コードによるサークル検索、出欠レポート
- **環境**: Node.js 20+、`npm start` で `http://localhost:4200/`

#### `kannai-attendance-client-v2`（🗄️ Archived）
- Svelte による作り直しの試作。**Archived** されており現在はメンテナンス対象外。現行は Angular 版（無印）を使用

---

### 3.2 🔐 AuthGate（SSO 認証基盤）

館内担当スタッフ向けの**統一認証基盤**。配下サービスに SSO を提供し、人事エクセルから組織ツリー・権限を管理する。

- **提供機能**
  - SSO 基盤（配下サービスへ統一認証）
  - 認証方式: サードパーティ認証（OAuth 2.0）+ スタッフ名 / スタッフ ID
  - OTP（TOTP ワンタイムパスワード）
  - 組織管理: 人事エクセルを期ごと（年 2 回）に取り込み、組織ツリー・権限を更新
  - 組織 API: 上位・下位組織の連結を含む組織情報を REST API で公開
- **技術スタック**
  - バックエンド: Python 3.12+ / FastAPI / SQLAlchemy 2.x / Alembic / Authlib / pyotp / python-jose / openpyxl / Pydantic v2 / uv
  - フロントエンド: SvelteKit / TypeScript / Tailwind CSS 4
  - DB: `DATABASE_URL` で PostgreSQL / MySQL / SQLite を切り替え（本番 PostgreSQL 16+ 推奨、開発 SQLite）
- **構成モジュール**（`backend/app/` 配下）: `auth`（OAuth・スタッフ ID・OTP・リスク判定）/ `sso`（JWT 発行・検証）/ `org`（組織・人事エクセル取り込み）/ `permission`（ロール・権限）/ `staff`（スタッフ管理）/ `audit`（監査ログ）/ `ban`（BAN 管理）
- **SDK**: サービス組み込み用の Python クライアント SDK（`sdk/python/`、FastAPI 用認証ミドルウェア付き）を同梱。組み込みガイドは `docs/integration.md`
- **前提環境**: Python 3.12+ / Node.js 22+ / uv
- ※ インフラ（本番配置先）は未定

---

### 3.3 🎙️ NeMSys（ネゴ管理システム）

**Ne**gotiation **M**anagement **Sys**tem。紙の「ネゴペーパー」に代わり、混雑予想サークルへのヒアリング（ネゴ）内容をデジタルで記録・管理する。

#### ネゴとは
混雑が予想されるサークルへスタッフが事前に行うヒアリング・調整。頒布物の種類・数量、売り子人数、列形成計画などを確認・すり合わせる。

#### 設計の要: 録音ファースト
過去バージョンの「ウィザードが面倒」「会話速度に入力が追いつかない」課題への対策として、

1. **1 画面フラットフォーム**: 全項目を 1 画面に表示、任意の順で入力・修正
2. **録音ファースト**: ヒアリング中は録音ボタンを押すだけ。オンライン復帰時に **STT →  LLM（意味抽出）→ 正規化** の 3 段パイプラインで各項目へ自動バインドし、スタッフは確認・修正のみ
   - 会場騒音対策として「スタッフが要点を復唱する運用」とセット
3. ステータス管理: `録音のみ(未同期)` → `バインド済み(要確認)` → `確認済み`

#### 技術スタック
- **方針**: TypeScript 一本化 / pnpm workspaces モノレポ（`client` / `server` / `shared`）/ インフラ・ランニングコスト 0 円（自前サーバー Docker 運用）
- **フロント（PWA）**: React + Vite / vite-plugin-pwa（Workbox）/ Dexie.js（IndexedDB、オフライン本体・音声 Blob 保存）/ MediaRecorder + Wake Lock / Tailwind CSS / Zod
- **バックエンド**: Hono（Node.js）/ PostgreSQL + Drizzle ORM / 軽量ジョブワーカー / `config.toml`（smol-toml）/ 認証は名前入力のみ（暫定）/ Docker Compose デプロイ（GHCR ビルド）
- **音声解析パイプライン**（会社契約の API 利用枠を使用）
  - ① STT: OpenAI `gpt-4o-transcribe`（第一候補）。`prompt` にドメイン語彙を投入
  - ② 意味抽出: Claude `claude-opus-4-8`（Anthropic API）+ Structured Outputs（`shared` の Zod スキーマ準拠を API 側で保証）
  - ③ 正規化: 決定的処理（数量・日時の正規化、Zod スキーマ検証・範囲チェック）
  - 音声原本と全文文字起こしは常に保持し、STT/LLM は後から差し替え可能なフォールバック設計
- **その他**: Claude Code 用の `.claude/skills`（client-dev / server-dev）、GitHub Actions（`ci.yml` / `client_deploy.yml`）を整備

---

### 3.4 🌐 kannai-web（館内 Web）

館内担当向けの業務 Web アプリケーション。**PHP 製のレガシー資産**で、館内担当の実務を幅広くカバーする。

- **技術**: PHP / HTML / CSS / JavaScript、Docker + docker-compose でローカル環境構築（README に Windows/WSL2 セットアップ手順を詳細記載）
- **主な画面**（`Kannai/` 配下より）
  - サークル管理: `CircleAll` / `CircleSearch` / `CircleIdou`（移動）
  - 各種レポート: `FacilityReport`（施設）/ `InjuriesReport`（負傷）/ `LostFound`（遺失物）
  - 頒布・給食: `Hanpu` / `Gohannnyuu`
  - 認証: `Login` / `Logoff`
  - 見出し・掲示: `Headline` 系
- 資産の大半は HTML（36MB 超）で、実装は PHP。歴史のある稼働システムのため、改修時は既存挙動の維持に注意

---

### 3.5 🚦 その他・構想段階

| リポジトリ | 内容 | 現状 |
| --- | --- | --- |
| `CrowdedSystem` | 混雑管理システム（description: 「混雑」） | リポジトリは空。構想段階 |
| `download_content_helper` | ダウンロードコンテンツ登録用ヘルパー | リポジトリは空。構想段階 |
| `demo-repository` | GitHub 機能デモ用のサンプル | 開発対象外 |

---

## 4. 技術スタック早見表

| 領域 | 採用技術 | 使用リポジトリ |
| --- | --- | --- |
| バックエンド | **FastAPI (Python)** | AuthGate, attendance-server-v2 |
| バックエンド | Hono (Node.js/TS) | NeMSys |
| バックエンド | Express (Node.js) | attendance-server（v1・旧） |
| バックエンド | PHP | kannai-web |
| フロントエンド | React + Vite (PWA) | NeMSys |
| フロントエンド | SvelteKit | AuthGate, attendance-client-v2（Archived） |
| フロントエンド | Angular 14 | attendance-client |
| DB / ストア | PostgreSQL | AuthGate, NeMSys |
| DB / ストア | Redis | attendance-server（v1） |
| ORM | SQLAlchemy / Drizzle | AuthGate / NeMSys |
| AI | OpenAI STT + Claude（意味抽出） | NeMSys |
| 認証 | OAuth 2.0 + TOTP（SSO） | AuthGate |

**傾向**: 新規・リプレイス案件はバックエンドを **FastAPI もしくは Hono（TypeScript）** に寄せ、少人数での保守性を重視。DB は PostgreSQL、インフラは自前サーバーの Docker 運用でランニングコストを抑える方針。

---

## 5. 新規メンバー向けメモ

- **開発フロー**: main への直接 push は避け、ブランチを切って **プルリクエスト** を作成する（attendance-server-v2 では宛先 `@m96-chan` を指定）
- **どこから読むか**
  - 受付の実務を理解したい → `kannai-web`（現行業務）と `kannai-attendance-*`（受付）
  - これからの主力 → `AuthGate`（認証基盤）、`NeMSys`（ネゴ管理）
- **バージョンの注意**: 受付システムは v1/v2、client の無印/v2 が混在する。**現行はサーバー v2（FastAPI）＋クライアント無印（Angular）**、client-v2（Svelte）は Archived
- 各リポジトリの詳細な環境構築手順は、それぞれの `README.md` を参照

---

*本書は Organization の各リポジトリの README・構成をもとに自動生成した俯瞰資料です。個別の最新仕様は各リポジトリを正とします。*
