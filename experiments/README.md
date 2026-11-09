# 実験

公開 canister や各モデルの canister とは独立した比較・試作のコードです。本番の crate は変更せず、出力は `.cache/` と `benchmarks/` に保存します。結果の読み方は各計測記録を参照してください。

| ディレクトリ | 内容 | 結果 |
| --- | --- | --- |
| [multilingual](multilingual/README.md) | 多言語 embedding モデルの検索精度比較と、Wasm への試験移植 | [日本語](../benchmarks/multilingual/RESULTS.md)・[英日拡張](../benchmarks/multilingual-expanded/RESULTS.md)・[6言語](../benchmarks/multilingual-global/RESULTS.md) |
| `embeddinggemma2` | EmbeddingGemma 2 を canister で動かす試作（独立した Cargo workspace） | [結果](../benchmarks/embeddinggemma2/RESULTS.md) |
| [bekko25-memory-study](bekko25-memory-study/README.md) | Bekko a25m のメモリ容量と量子化による精度の調査 | [結果](../benchmarks/bekko25-memory-study/RESULTS.md) |
| [bekko25-stable-read](bekko25-stable-read/README.md) | 語彙表を stable memory から直接読む試作 | [結果](../benchmarks/bekko25-stable-read/RESULTS.md) |

試作の多くは後に本体へ取り込まれています。現在の仕様は [ドキュメント一覧](../docs/README.md) を参照してください。
