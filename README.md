# kabukeisan-jobs

EDINET API から XBRL ファイルを取得して処理するバッチジョブ（Go 実装）

## 概要

このプロジェクトは、EDINET API から有価証券報告書の XBRL ファイルを取得し、パースしてデータベースに登録するバッチ処理システムです。
TypeScript で実装された`batch`フォルダの機能を Go で再実装したものです。

## 技術スタック

- **言語**: Go 1.21+
- **ORM**: Ent (Prisma と同様のスキーマ駆動型 ORM)
- **データベース**: MySQL
- **依存関係管理**: Go Modules

## セットアップ

### 1. 依存関係のインストール

```bash
go mod download
go mod tidy
```

### 2. Ent コードの生成

Ent のコードを生成する前に、依存関係がインストールされていることを確認してください。

```bash
go generate ./ent
```

または

```bash
go run -mod=mod entgo.io/ent/cmd/ent generate ./ent/schema
```

**注意**: Ent コードが生成されるまで、一部のインポートエラーが表示される場合があります。これは正常です。

### 3. 環境変数の設定

`.env`ファイルを作成し、以下の環境変数を設定してください：

```env
DATABASE_URL=mysql://user:password@localhost:3306/database_name
EDINET_API_KEY=your_edinet_api_key
DOC_PROCESS_CONCURRENCY=5
```

### 4. データベースマイグレーション

```bash
go run cmd/main.go migrate
```

## 使用方法

### バッチ処理の実行

```bash
go run cmd/main.go <dateFrom> <dateTo>
```

例：

```bash
go run cmd/main.go 20240101 20240131
```

- `dateFrom`: 開始日（YYYYMMDD 形式）
- `dateTo`: 終了日（YYYYMMDD 形式）

## プロジェクト構造

```
jobs/
├── cmd/
│   └── main.go              # メインエントリーポイント
├── ent/
│   └── schema/              # Entスキーマ定義
│       ├── company.go
│       ├── balance_sheet.go
│       ├── profit_loss.go
│       ├── cash_flow.go
│       ├── capital_expenditure.go
│       └── debt.go
├── internal/
│   ├── config/
│   │   └── constants.go     # 定数定義
│   ├── database/
│   │   └── client.go        # データベースクライアント
│   ├── domain/
│   │   └── company/         # ドメイン層
│   │       └── company.go
│   ├── types/
│   │   └── interfaces.go    # 型定義
│   └── utils/
│       ├── api.go           # EDINET APIクライアント
│       ├── date.go          # 日付ユーティリティ
│       ├── file.go          # ファイル操作
│       ├── xbrl_parser.go   # XBRLパーサー
│       └── xbrl_adapter.go  # XBRLアダプター
├── go.mod
├── go.sum
└── README.md
```

## 主な機能

1. **EDINET API 連携**: 有価証券報告書のドキュメントリストを取得
2. **ZIP ファイル処理**: ダウンロードした ZIP ファイルの展開
3. **XBRL パース**: XBRL ファイルから財務データを抽出
4. **データベース登録**: パースしたデータをデータベースに保存
5. **並列処理**: 複数のドキュメントを並列で処理

## 開発

### テストの実行

```bash
go test ./...
```

### コード生成（Ent）

スキーマを変更した後、以下のコマンドでコードを再生成：

```bash
go generate ./ent
```

## 注意事項

- EDINET API の利用には API キーが必要です
- 大量のデータを処理する場合、適切な並列度（`DOC_PROCESS_CONCURRENCY`）を設定してください
- データベース接続プールの設定を適切に行ってください
