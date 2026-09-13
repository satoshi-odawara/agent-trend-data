# agent-trend-data

「データ収集システム(agent-trend-radar)」と「記事作成システム」の間で、
データと記事原稿を受け渡すためのハブリポジトリです。このリポジトリ自身は
収集ロジックも記事生成ロジックも持たず、決められたディレクトリ構造でデータと
原稿を保持するだけの存在です。

## このリポジトリの役割

- データ収集システムが収集したスナップショットを、日付ごとに蓄積する
- 記事作成システムが最新データを参照しやすいよう `latest/` を維持する
- 記事作成システムからの「こういうデータが欲しい」という要望を `data-requests/` で受け取り、蓄積する
- 記事作成システムが書いた記事原稿を `articles/` に保管する

## データの読み方

- まずは [`latest/metrics.json`](./latest/metrics.json) を見てください。常に最新のスナップショットのコピーが置かれています。
- 過去の推移(時系列)が必要な場合は [`snapshots/YYYY-MM-DD/metrics.json`](./snapshots/) を日付ごとに参照してください。過去データは追記のみで、上書き・削除はされません。
- 各フィールドの意味・型については [`schema/SCHEMA.md`](./schema/SCHEMA.md) を参照してください。

## ディレクトリ構成

```
agent-trend-data/
├── schema/SCHEMA.md          # フィールドの意味・型・バージョン定義
├── snapshots/YYYY-MM-DD/     # 収集結果のスナップショット(追記のみ)
├── latest/                   # 最新スナップショットへの導線
├── data-requests/
│   ├── pending/               # 記事作成システムからの要望(未対応)
│   └── done/                  # 対応済みの要望
└── articles/YYYY-MM/         # 記事原稿
```

## ライセンス

データ(`snapshots/`, `latest/`, `manifest.json` 等)は [CC0 1.0 Universal](./LICENSE) の下で提供され、著作権が放棄されています。自由に二次利用してください。
