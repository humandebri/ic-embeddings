# Bekko a25m を選んだ理由

公開 canister に Bekko a25m を採用した根拠を、このリポジトリの実験結果から整理したものです（2026-10-30 時点）。比較の規模には限りがあるため、末尾の注意点もあわせて読んでください。

当初の日本語 JaQuAD の小規模比較では、256-token chunks の Recall@5 は a8m と multilingual-e5-small がともに 96.09%、a25m が 93.75% でした。a8m を有力候補としていた記録です。[初回比較](../benchmarks/multilingual/RESULTS.md)

その後、英語を中心に他言語も重視した比較で、a25m は精度と 1 query での実行可能性を両立する候補になりました。[6言語比較](../benchmarks/multilingual-global/RESULTS.md)

- 128-token chunks の同言語検索で、英語の Recall@5 は a25m 85.32%、a8m 83.71%、E5 78.67%。アラビア語は順に 78.77%、75.74%、71.55%。それぞれ SciFact と FiQA の小規模データを平均した W8A8/BF16 KV の Torch 参照値です。
- 英語の質問から韓国語の文書を検索する場合も、同じ実験では a25m 70.59%、a8m 64.62%、E5 44.80% でした。これは言語をまたぐ検索の一例であり、混在言語の索引を評価したものではありません。
- 比較時の PocketIC 実測では a25m は 128 tokens を約 26.08 億命令の 1 query で実行。一方、英語の参照精度が高かった EmbeddingGemma 2 は試作で 12 query を必要とし、E5 は canister の命令数を測定していませんでした。
- その後 a25m は追加の QK13 量子化と最適化により、256 tokens も 1 query で実行可能になりました。F32 語彙表版のローカル運用検証では全入力長の観測最大が約 48.44 億命令で、50 億命令上限までの余裕は約 3.12% でした。[運用検証](../benchmarks/bekko25-operations/RESULTS.md)。この数値は本番 compact 版の保証ではなく、本番で観測した上限付近の値は[公開 API の測定記録](../benchmarks/bekko25-compact/RESULTS.md#attention-instruction-headroom)を参照してください。

注意点として、6言語比較の各領域は 50 問で、英語以外は翻訳データです。a25m と a8m の同言語検索の差の 95% 区間はゼロを含むため、a25m の優位は確定していません。初回の日本語比較では a8m が上回っています。また、a25m の高精度には計算量の負担が伴います。a8m の 256-token 実測は約 19.43 億命令で、a25m とは別の実装・計測条件です。[初回比較](../benchmarks/multilingual/RESULTS.md)

したがって、a25m は「英語や多言語検索の精度を期待しつつ、256 tokens を 1 query に収めたい」という条件で有望だった、と整理できます。E5 の日本語スコアだけで乗り換えを決めず、実際の文書と質問で日本語・英語・言語間検索を比較し、必要なら E5 の ICP 実行コストも測って判断します。
