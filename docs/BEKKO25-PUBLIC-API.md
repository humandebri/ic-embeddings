# Bekko a25m 公開 embedding API

ICP mainnet の `xrg4h-oiaaa-aaaal-qxrya-cai` で、Bekko a25m の INT8 compact 版を利用できます。`embed` と `modelInfo` は匿名を含む誰でも呼び出せる query です。利用者は API キー、controller 権限、モデル重みのアップロードを必要としません。

| 項目 | 値 |
| --- | --- |
| Canister | `xrg4h-oiaaa-aaaal-qxrya-cai` |
| モデル | `hotchpotch/bekko-embedding-v1-a25m` |
| モデル revision | `44f0b8af0f487acd0ccf1a7cb7ae7a29a6dfc09c` |
| 入力 | token ID 2〜256 個。先頭 BOS `2`、末尾 EOS `1` を含む |
| 出力 | 384 次元の L2 正規化済み embedding |
| Embedding space | `bekko-a25m-44f0b8af-vocab-i8row-w8a8-bf16kv-qk13-v1` |
| Interface | [Candid 定義](../crates/bekko25-canister/bekko25.did) |

2026-11-04 に匿名 principal `2vxsx-fae` から `modelInfo` と `embed` の成功を確認しました。稼働状況は変化するため、利用時にも `modelInfo` を確認してください。

## CLI ですぐ試す

[icp CLI](https://dfinity.github.io/icp-cli/) をインストールして、次を実行します。`active = true` と `weights_verified = true` が推論可能な状態を示します。

```sh
icp canister call xrg4h-oiaaa-aaaal-qxrya-cai modelInfo '()' \
  --query --network ic --identity anonymous
```

最小の動作確認は、BOS と EOS だけを渡して実行できます。これは空の入力のテストであり、実際の文章は後述の tokenizer で変換してください。

```sh
icp canister call xrg4h-oiaaa-aaaal-qxrya-cai embed \
  '(vec { 2 : nat32; 1 : nat32 })' \
  --query --network ic --identity anonymous
```

成功時は `variant { Ok = record { embedding; embedding_space; tokens; instructions } }`、入力エラーなどの場合は `variant { Err = "..." }` が返ります。`embedding` は `vec float32`、`tokens` は入力数、`instructions` は canister が推論処理内で計測した命令数です。`instructions` は通信時間やメッセージ全体の費用を表しません。

## 日本語テキストを埋め込む

このリポジトリのルートで実行します。Python 3.12 とネットワーク接続が必要です。以下は tokenizer だけを使うため、モデル重みや PyTorch は不要です。初回は Hugging Face から固定 revision の tokenizer を取得します。

```sh
python3 -m venv .cache/bekko25-client-venv
.cache/bekko25-client-venv/bin/pip install 'transformers==5.20.2'
mkdir -p .cache/bekko25

.cache/bekko25-client-venv/bin/python scripts/bekko_tokens.py --model a25m \
  --text 'インターネットコンピューターの検索' \
  --output .cache/bekko25/client-example.did

icp canister call xrg4h-oiaaa-aaaal-qxrya-cai embed \
  --args-file .cache/bekko25/client-example.did \
  --query --network ic --identity anonymous
```

文章をファイルから渡す場合は `--text` の代わりに `--input path/to/text.txt` を使います。生成されるのは `(vec { ... })` 形式の Candid 引数です。tokenizer は暗黙の切り捨てを行わず、BOS/EOS を含め 256 tokens を超えた場合はエラーにします。長い文章は分割してそれぞれを埋め込んでください。

検索クエリと検索対象の文書は、どちらも同じ tokenizer と `embed` を使います。Bekko は `query:` や `passage:` などの接頭辞を必要としません（[モデルの説明](https://huggingface.co/hotchpotch/bekko-embedding-v1-a25m/blob/44f0b8af0f487acd0ccf1a7cb7ae7a29a6dfc09c/README.md)）。文書のベクトルを保存し、検索時にクエリのベクトルとの類似度を比較します。ベクトルの保存と検索は利用するアプリ側で行います。

## JavaScript / Node.js から呼ぶ

アプリ側に [ICP JavaScript SDK](https://js.icp.build/core/latest/libs/agent/) をインストールします。以下は Node.js 24 と `@icp-sdk/core` 6.1.1 で確認した、`embed` のみの最小 Candid 定義です。実アプリでは上記の `.did` から型付き binding を生成することもできます。

```sh
npm install @icp-sdk/core@6.1.1
```

```js
import { Actor, HttpAgent } from '@icp-sdk/core/agent';

const idlFactory = ({ IDL }) => {
  const Embedding = IDL.Record({
    embedding_space: IDL.Text,
    instructions: IDL.Nat64,
    tokens: IDL.Nat32,
    embedding: IDL.Vec(IDL.Float32),
  });
  return IDL.Service({
    embed: IDL.Func(
      [IDL.Vec(IDL.Nat32)],
      [IDL.Variant({ Ok: Embedding, Err: IDL.Text })],
      ['query'],
    ),
  });
};

// identity を指定しないため匿名で呼び出します。
const agent = await HttpAgent.create({ host: 'https://icp-api.io' });
const actor = Actor.createActor(idlFactory, {
  agent,
  canisterId: 'xrg4h-oiaaa-aaaal-qxrya-cai',
});

// 「インターネットコンピューターの検索」を上記 tokenizer で変換した例。
// 実際の入力では、利用するテキストごとに同じ revision で token 化します。
const tokenIds = [2, 23908, 6352, 29273, 184527, 6352, 235372, 46302, 1];
const result = await actor.embed(tokenIds);
if ('Err' in result) throw new Error(result.Err);

const { embedding, embedding_space, tokens, instructions } = result.Ok;
console.log({
  dimensions: embedding.length, // 384
  embedding_space,
  tokens,
  instructions: instructions.toString(), // nat64 は bigint
});
// embedding を検索用のベクトルとして使用します。
```

上のコードを `example.mjs` に保存して `node example.mjs` で実行できます。`modelInfo` なども使う場合は、[Candid 定義](../crates/bekko25-canister/bekko25.did)から binding を生成するか、必要なメソッドを `idlFactory` に追加してください。

## 利用時の注意

- 入力 ID は `0` の padding を含めず、モデルの語彙範囲内である必要があります。canister は文字列を直接受け取らず、tokenizer を内蔵していません。
- 文書と検索クエリには同じ tokenizer と `embedding_space` を使ってください。F32 語彙表版や a8m は別の空間なので、同じ検索インデックスに混ぜないでください。
- query 結果にアプリ独自の certified data は付いていません。通常の query は単一 replica で実行され、update のような consensus による検証はありません。[ICP の認証付きデータの説明](https://docs.internetcomputer.org/guides/security/data-integrity-and-authenticity/)も参照してください。
- [ICP の query 上限](https://docs.internetcomputer.org/references/resource-limits/)は 1 回あたり 50 億命令です。現在の本番版は 255 tokens の実測で最大約 49.54 億命令に達しており、最大長付近の全入力が成功する保証はありません。長い入力では余裕を持って分割してください。最適化版は mainnet 未適用です。[測定結果](../benchmarks/bekko25-compact/RESULTS.md#attention-instruction-headroom)を参照してください。
- [通常の query 自体に cycles は課金されません](https://docs.internetcomputer.org/concepts/cycles/)が、canister の保存や運用には cycles が必要で、利用可能性は運用状態に依存します。負荷が高い場合は遅延や失敗が起こりえます。
- `beginUpload`・`upload`・`activate` は管理者専用の update です。利用者が呼び出す必要はありません。

canister の構築・重み・アップグレードは [compact 版の運用ガイド](BEKKO25-COMPACT.md)を参照してください。
