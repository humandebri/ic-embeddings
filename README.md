# ICP embedding canisters

[![Rust checks](https://github.com/humandebri/ic-embeddings/actions/workflows/ci.yml/badge.svg)](https://github.com/humandebri/ic-embeddings/actions/workflows/ci.yml)

Internet Computer（ICP）の canister 上で、文章の embedding（検索用ベクトル）を計算する実装です。外部の API サーバーを使わず、オンチェーンの query 呼び出しで embedding を得られます。

```text
文章 → クライアントの tokenizer → token ID → canister の embed（query）→ 384 次元ベクトル
```

- **公開 canister をすぐ使える**：多言語モデル Bekko a25m を mainnet で公開しています。API キー不要、匿名で呼び出せます。
- **tokenizer はクライアント側**：canister は文字列ではなく token ID を受け取ります。Python と JavaScript の例を用意しています。
- **ベクトルの保存と検索はアプリ側**：この canister は embedding を返すだけです。インデックスや類似検索は利用するアプリで実装します。

## クイックスタート

mainnet の **`xrg4h-oiaaa-aaaal-qxrya-cai`** を [icp CLI](https://dfinity.github.io/icp-cli/) から呼び出します。

モデルの状態を確認します。`active = true` と `weights_verified = true` なら利用できます。

```sh
icp canister call xrg4h-oiaaa-aaaal-qxrya-cai modelInfo '()' \
  --query --network ic --identity anonymous
```

BOS（`2`）と EOS（`1`）だけを渡す最小の動作確認です。成功すると `Ok` の中に 384 次元の `embedding` が返ります。

```sh
icp canister call xrg4h-oiaaa-aaaal-qxrya-cai embed \
  '(vec { 2 : nat32; 1 : nat32 })' \
  --query --network ic --identity anonymous
```

実際の文章を埋め込む手順（Python・Node.js）、戻り値、エラー、制限は **[公開 API 利用ガイド](docs/BEKKO25-PUBLIC-API.md)** にまとめています。

## モデル一覧

| モデル | 向いている用途 | 入力上限 | 提供形態 | ドキュメント |
| --- | --- | --- | --- | --- |
| **Bekko a25m INT8 compact** | 多言語検索。まず試すならこれ | 256 tokens／1 query | mainnet で公開中 | [公開 API](docs/BEKKO25-PUBLIC-API.md)・[構築](docs/BEKKO25-COMPACT.md) |
| Bekko a25m F32 語彙版 | compact 版より語彙表の精度を優先したい場合 | 256 tokens／1 query | 自分で構築 | [構築・運用](docs/BEKKO25.md) |
| Bekko a8m | 計算量の少ない軽量な多言語モデル | 256 tokens／1 query | 自分で構築 | [構築・運用](docs/BEKKO.md) |
| Granite Small English R2 | 英語の長文（複数 query で処理） | 1,365 tokens（分割クライアントで 5,000 tokens） | 自分で構築 | [API](docs/API.md)・[分割クライアント](docs/SHARDED.md) |

入力上限には BOS/EOS や CLS/SEP などの特殊 token を含みます。上限を超える入力は切り詰めずにエラーを返すため、長い文章はクライアント側で分割してください。

## 利用前に知っておくこと

- **モデルごとにベクトル空間が異なります。** 次元数が同じでも、別のモデルや量子化方式のベクトルは比較できません。同じ検索インデックスには、同じモデルで作った文書ベクトルとクエリベクトルだけを保存してください。Bekko a25m は応答の `embedding_space` で識別できます。Bekko a8m と Granite はこのフィールドを返さないため、利用側でモデルを記録してください。
- **query の応答は認証されていません。** 通常の query は単一 replica で実行され、certified data も付きません。結果の改ざん耐性が必要な用途では注意してください。
- **性能値は保証ではありません。** 命令数や速度は入力長・ビルド・実行環境で変わります。計測記録は特定の条件での実測です。

## 自分で構築・運用する

公開 canister を使うだけなら構築は不要です。独自の canister を立てる場合は、モデルごとのガイドに従ってください。各 canister はモデル重みを Wasm に含めないため、ビルド後に controller から重みをアップロードして有効化します。

- [Bekko a25m INT8 compact](docs/BEKKO25-COMPACT.md)：公開 canister と同じ構成
- [Bekko a25m F32 語彙版](docs/BEKKO25.md)
- [Bekko a8m](docs/BEKKO.md)
- [Granite Small English R2](docs/DEVELOPMENT.md)

ビルドには Rust（[rust-toolchain.toml](rust-toolchain.toml) で固定）、`wasm32-unknown-unknown` target、Python 3.12、[icp CLI](https://dfinity.github.io/icp-cli/) を使います。検証には PocketIC 11.0.0 が必要です。

このリポジトリの [mainnet ID 対応](.icp/data/mappings/ic.ids.json) は公開中の canister を指しています。自分で配備するときは、ローカル環境（`-e local`）で構築・検証してください。

## ドキュメント

- [ドキュメント一覧](docs/README.md)：目的別の読み方とリポジトリ構成
- [計測結果の一覧](benchmarks/README.md)：性能・精度・運用検証の記録
- [Bekko a25m を選んだ理由](docs/BEKKO-MODEL-SELECTION.md)：モデル比較と、その限界

## ライセンス

このリポジトリのソースコードは [MIT License](LICENSE) で提供します。モデルの重みや評価用データには、それぞれの提供元の利用条件が適用されます。
