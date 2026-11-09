# ドキュメント一覧

目的に合わせて読むページを選んでください。プロジェクトの概要は [README](../README.md) にあります。

## 公開 canister を使う

| ページ | 内容 |
| --- | --- |
| [Bekko a25m 公開 API](BEKKO25-PUBLIC-API.md) | mainnet canister の呼び出し方、Python・Node.js の例、戻り値、エラー、制限 |
| [Bekko a25m を選んだ理由](BEKKO-MODEL-SELECTION.md) | 他モデルとの比較結果と、その比較の限界 |

## 自分で canister を構築・運用する

| ページ | 内容 |
| --- | --- |
| [Bekko a25m compact](BEKKO25-COMPACT.md) | 公開 canister と同じ INT8 語彙版。ビルド、検証、インデックスの移行（英語） |
| [Bekko a25m](BEKKO25.md) | F32 語彙版。ビルド、重みのロード、API、アップグレード、リリース検証（英語） |
| [Bekko a8m](BEKKO.md) | 軽量版。ビルド、ローカル実行、API、RAG の chunk 長（英語） |
| [Granite API](API.md) | 重みのロードと `startEmbedding` / `continueEmbedding` |
| [Granite 開発手順](DEVELOPMENT.md) | ビルド、参照データの生成、検証、計測の再実行 |
| [Granite 分割クライアント](SHARDED.md) | 1,366〜5,000 tokens を複数 query で処理する方式 |

## 過去の記録

| ページ | 内容 |
| --- | --- |
| [Granite の設計調査](history/granite-feasibility.md) | 実装前の調査メモ。現在の仕様ではありません |

計測結果は [benchmarks/README.md](../benchmarks/README.md)、モデル比較などの実験コードは [experiments/README.md](../experiments/README.md) にまとめています。

## リポジトリ構成

| パス | 内容 |
| --- | --- |
| `crates/bekko-core` | Bekko 共通の encoder 実装（a8m・a25m） |
| `crates/bekko25-core` | Bekko a25m 固有の設定 |
| `crates/bekko-canister` | Bekko a8m の canister |
| `crates/bekko25-canister` | Bekko a25m F32 語彙版の canister と Candid 定義 |
| `crates/bekko25-compact-canister` | Bekko a25m INT8 compact の canister |
| `crates/bekko-bench`, `crates/bekko25-bench` | PocketIC での検証ハーネスと重みのアップロードツール |
| `crates/core`, `crates/canister`, `crates/bench` | Granite の推論実装、canister、検証ハーネス |
| `crates/kernel-gen` | Granite と Bekko で共有する SIMD GEMM の生成 |
| `scripts/` | 重みの変換、tokenizer、ビルド、検証の補助スクリプト |
| `benchmarks/` | 計測結果と検証記録 |
| `experiments/` | 本体とは独立した比較実験 |

モデル重み、生成した参照データ、ビルド成果物は `.cache/` と `target/` に置かれ、Git では管理しません。
