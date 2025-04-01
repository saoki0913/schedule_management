# スケジュール管理システム

面接予定の管理と日程調整を効率化するためのWebアプリケーション

## 概要

このプロジェクトは、面接のスケジュール調整を自動化・効率化するためのシステムです。複数の面接官の空き時間を確認し、候補日時を提示、応募者が希望する日時を選択することで、面接予定の作成とTeamsミーティングの自動設定を行います。

### 主な機能

- 面接担当者の予定表から空き時間の自動検出
- 候補日時の提示と選択フォームの生成
- 予定選択リンクのメール共有機能
- Teamsミーティングの自動設定
- 予定の再調整機能
- メール通知機能

## プロジェクト構成

```
schedule_management/
├── api/                           # バックエンドAPI (Azure Functions + FastAPI)
│   ├── app/                       # アプリケーションのメインディレクトリ
│   │   ├── main.py               # FastAPIアプリケーションのエントリーポイント
│   │   ├── config.py             # 設定値の管理
│   │   ├── routers/              # APIエンドポイントの実装
│   │   │   ├── form.py          # フォーム関連のエンドポイント
│   │   │   └── schedule.py      # スケジュール関連のエンドポイント
│   │   ├── schemas/              # リクエスト/レスポンスのスキーマ定義
│   │   │   ├── __init__.py      # スキーマのエクスポート
│   │   │   ├── form.py          # フォーム関連のスキーマ
│   │   │   └── schedule.py      # スケジュール関連のスキーマ
│   │   ├── internal/             # 内部モジュール
│   │   │   ├── cosmos.py        # Cosmos DB関連の処理
│   │   │   └── graph_api.py     # Graph API関連の処理
│   │   └── utils/                # ユーティリティ関数
│   │       └── time_utils.py    # 時間関連のユーティリティ
│   │
│   ├── tests/                     # テストコード
│   │   ├── test_form.py          # フォーム関連のテスト
│   │   └── test_schedule.py      # スケジュール関連のテスト
│   │
│   ├── function_app.py            # Azure Functionsのエントリーポイント
│   ├── main.py                    # フォーム関数とスケジュール関数の実装
│   ├── requirements.txt           # Pythonの依存パッケージ
│   ├── .funcignore                # Azure Functionsのデプロイ除外設定
│   ├── host.json                  # Azure Functions設定ファイル
│   └── local.settings.json        # ローカル環境変数設定
│
└── schedule_client/               # フロントエンド (Next.js + TypeScript)
    ├── app/                       # Next.jsアプリディレクトリ
    │   ├── page.tsx               # ホームページ
    │   ├── layout.tsx             # 共通レイアウト
    │   ├── globals.css            # グローバルスタイル
    │   │
    │   ├── schedule/              # スケジュール管理ページ
    │   │   ├── page.tsx           # スケジュール設定画面
    │   │   └── components/        # スケジュール関連コンポーネント
    │   │       ├── ScheduleForm.tsx  # 日程設定フォーム
    │   │       └── CandidateList.tsx # 候補日時リスト表示
    │   │
    │   ├── appointment/           # 予定選択ページ
    │   │   └── page.tsx           # 候補日時選択画面
    │   │
    │   ├── interview/             # 面接情報ページ
    │   │   └── page.tsx           # 面接詳細画面
    │   │
    │   └── profile/               # プロフィールページ
    │       └── page.tsx           # ユーザープロフィール画面
    │
    ├── public/                    # 静的ファイル
    ├── node_modules/              # npmパッケージ
    ├── package.json               # 依存パッケージと設定
    ├── package-lock.json          # 依存パッケージのバージョン固定
    ├── tsconfig.json              # TypeScript設定
    ├── tailwind.config.js         # Tailwind CSS設定
    └── postcss.config.js          # PostCSS設定
```

### APIエンドポイント

バックエンドでは以下の主要なエンドポイントを提供しています：

#### フォーム関連
- `POST /storeFormData`: フォームデータを保存し、一意のトークンを返します
- `GET /retrieveFormData`: 保存されたフォームデータをトークンをもとに取得します

#### スケジュール関連
- `POST /get_availability`: 複数の面接官の空き時間を検索し、共通の空き時間を返します
- `POST /appointment`: 選択された日時で面接予定を作成し、Teamsミーティングを設定します
- `GET /reschedule`: 予定の再調整を行います

### フロントエンドページ構成

- **ホームページ**（`/`）: スケジュール管理システムのエントリーポイント
- **スケジュールページ**（`/schedule`）: 面接官の空き時間を検索し、候補日時を生成する画面
- **アポイントメントページ**（`/appointment`）: 候補日時から希望の日時を選択する画面
- **インタビューページ**（`/interview`）: 面接の詳細情報を表示する画面
- **プロフィールページ**（`/profile`）: ユーザープロフィール設定画面

### データフロー

1. 面接官のメールアドレスと日程制約をスケジュールページで入力
2. バックエンドAPIが各面接官のOutlookカレンダーから空き時間を検索
3. 共通の空き時間をスケジュールページに表示
4. 生成された候補日時から成るフォームのリンクを応募者に共有
5. 応募者がアポイントメントページで希望日時を選択
6. バックエンドAPIが選択された日時でTeamsミーティングを作成
7. 面接官と応募者にミーティング招待メールを送信

## 技術スタック

### バックエンド
- Python 3.9+
- Azure Functions
- FastAPI
- Azure Cosmos DB
- Microsoft Graph API
- MSAL (Microsoft Authentication Library)

### フロントエンド
- TypeScript
- Next.js
- Tailwind CSS

## セットアップ

### バックエンド

#### 前提条件
- Python 3.9以上
- Azure Functions Core Tools
- Azure CLI（オプション：Azureリソースの管理用）

#### 環境変数の設定
1. `api/local.settings.json`ファイルを作成し、以下の設定を追加：

```json
{
    "IsEncrypted": false,
    "Values": {
        "FUNCTIONS_WORKER_RUNTIME": "python",
        "COSMOS_DB_ENDPOINT": "your-cosmos-db-endpoint",
        "COSMOS_DB_KEY": "your-cosmos-db-key",
        "TENANT_ID": "your-tenant-id",
        "CLIENT_ID": "your-client-id",
        "CLIENT_SECRET": "your-client-secret",
        "BACKEND_URL": "http://localhost:7071",
        "FRONT_URL": "http://localhost:3000"
    }
}
```

#### 依存パッケージのインストール

```bash
cd api
# 仮想環境の作成と有効化
python -m venv .venv
source .venv/bin/activate  # Linux/macOS
.venv\Scripts\activate     # Windows

# 依存パッケージのインストール
pip install -r requirements.txt
```

#### ローカル実行

```bash
# Azure Functions Core Toolsを使用してローカルで実行
func start
```

### フロントエンド

#### 前提条件
- Node.js 18以上
- npm, yarnなどのパッケージマネージャー

#### 依存パッケージのインストール

```bash
cd schedule_client
npm install
```

#### 環境変数の設定
必要に応じて `.env.local` ファイルを作成し、APIのエンドポイントを設定します：

```
NEXT_PUBLIC_API_URL=http://localhost:7071
```

#### ローカル実行

```bash
npm run dev
```

## 使用方法

1. スケジュール管理ページで面接官の空き時間を選択
2. スケジュールフォームを作成してリンクを共有
3. 応募者が希望する日時を選択
4. 面接予定が自動的に作成され、参加者にTeamsミーティングリンクが送信される

## デプロイ

### バックエンド（Azure Functions）

```bash
cd api
func azure functionapp publish <function-app-name>
```

### フロントエンド（Next.js）

```bash
cd schedule_client
npm run build
# Vercelなどのプラットフォームにデプロイ
```

## ライセンス

このプロジェクトは社内利用を目的として開発されています。 