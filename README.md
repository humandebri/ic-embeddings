# ICP embedding canisters

[![Rust checks](https://github.com/humandebri/ic-granite-embedding/actions/workflows/ci.yml/badge.svg)](https://github.com/humandebri/ic-granite-embedding/actions/workflows/ci.yml)

文章を検索しやすいベクトルに変換する、Internet Computer（ICP）向けの canister 実装です。テキストを呼び出し側で token ID に変換し、canister が embedding を計算します。

```text
文章 → クライアントの tokenizer → token ID → canister の embed → ベクトル
```

## 利用できるモデル

| モデル | 状態・用途 | 入力上限 | 詳細 |
| --- | --- | --- | --- |
| **Bekko a25m INT8 compact** | 多言語検索向け。mainnet に公開中 | 256 tokens／1 query | [公開 API 利用ガイド](docs/BEKKO25-PUBLIC-API.md) |
| Bekko a25m F32 語彙版 | 独自 canister を構築・運用する場合 | 256 tokens／1 query | [運用ガイド](docs/BEKKO25.md) |
| Bekko a8m | 軽量な多言語モデル。独自 canister を構築・検証 | 256 tokens／1 query | [モデルと利用手順](docs/BEKKO.md) |
| Granite Small English R2 | 英語向けのローカル PoC | 通常 API は 1,365 tokens、追加クライアントは 5,000 tokens | [API とクライアント](docs/API.md)、[分割クライアント](docs/SHARDED.md) |

各上限には BOS/EOS や CLS/SEP などの特殊 token を含みます。モデルや量子化方式ごとに embedding の空間は異なります。同じ検索インデックスには、同じモデル・同じ数値仕様で作った文書と検索クエリのベクトルを保存してください。Bekko a25m は応答の `embedding_space` で識別できます。Bekko a8m と Granite はこのフィールドを返さないため、利用側でモデルを識別してください。

## 公開 Bekko a25m を試す

mainnet の **`xrg4h-oiaaa-aaaal-qxrya-cai`** は、匿名を含め誰でも利用できます。API キーやモデル重みのアップロードは不要です。

[icp CLI](https://dfinity.github.io/icp-cli/) をインストールし、モデルの状態を確認します。

```sh
icp canister call xrg4h-oiaaa-aaaal-qxrya-cai modelInfo '()' \
  --query --network ic --identity anonymous
```

次は BOS と EOS のみで `embed` を呼ぶ動作確認です。成功すると `Ok` の中に 384 次元の `embedding` が返ります。

```sh
icp canister call xrg4h-oiaaa-aaaal-qxrya-cai embed \
  '(vec { 2 : nat32; 1 : nat32 })' \
  --query --network ic --identity anonymous
```

これは空入力のテストです。実際の文章を使う場合は、クライアントで固定 revision の tokenizer を実行してから token ID を送ります。**[公開 API 利用ガイド](docs/BEKKO25-PUBLIC-API.md)** に Python と JavaScript（Node.js）で文章を token 化して呼び出す手順、戻り値、エラー、入力制限をまとめています。

検索クエリと文書は、同じ tokenizer と `embed` で変換できます。ベクトルの保存と類似検索は利用するアプリ側で行います。通常の query 応答はアプリ独自の certified data を持ちません。

## 自分で構築・運用する

各モデルには別々の canister と API があります。

- **Bekko a25m INT8 compact**：[ビルド・重みの準備・検証・移行](docs/BEKKO25-COMPACT.md)
- **Bekko a25m F32 語彙版**：[ビルド・利用・アップグレード](docs/BEKKO25.md)
- **Bekko a8m**：[ビルド・利用・検証](docs/BEKKO.md)
- **Granite Small English R2**：[API と重みのロード](docs/API.md)、[開発手順](docs/DEVELOPMENT.md)、[長文の分割クライアント](docs/SHARDED.md)

各 canister はモデル重みを Wasm に含めません。独自に配備する場合は、対象モデルのガイドに従って重みを準備・ロードしてください。
このリポジトリの [mainnet ID 対応](.icp/data/mappings/ic.ids.json)は公開中の Bekko a25m compact を指します。mainnet への配備は既存 canister に影響するため、ローカルでの構築・検証と区別してください。

## 計測とモデル選定

性能値はモデル・入力長・実行環境によって異なります。本番の限界値を単一の計測から保証することはできません。

- [Bekko a25m compact の実測と本番 upgrade 検証](benchmarks/bekko25-compact/RESULTS.md)
- [Bekko a25m を検討した理由と比較上の限界](docs/BEKKO-MODEL-SELECTION.md)
- [多言語検索の比較](benchmarks/multilingual-global/RESULTS.md)
- [Granite の性能・長文検証](benchmarks/query-fusion/RESULTS.md)
- [すべての計測記録](benchmarks/README.md)

## ライセンス

このリポジトリのソースコードは [MIT License](LICENSE) で提供します。モデルの重みや評価用データには、それぞれの提供元の利用条件が適用されます。
