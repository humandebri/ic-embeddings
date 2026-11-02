# Bekko a25m compact runtime validation

2026-10-30. Local PocketIC 11.0.0, Application subnet. The old F32-vocabulary canister and the compact canister were separately installed with their pinned packs. These are local measurements, not mainnet latency, throughput or cycles guarantees. Canister status values exclude other memory and query-time temporary peaks.

| Metric | F32 vocabulary | INT8 vocabulary and INT8-resident linear weights |
|---|---:|---:|
| Wasm heap | 174,718,976 B | 151,388,160 B |
| Stable memory | 418,447,360 B | 124,518,400 B |
| Heap + stable | 593,166,336 B | 275,906,560 B |

All 475 inputs completed below 4.9B instructions: 100 Japanese fixture inputs, 18 multilingual inputs, one seeded input for each length 2–256, and 102 repeated-token boundary inputs. For the seeded 256-token input, instructions were 4,794,538,776 -> 4,796,913,842 (+0.0495%); the maximum observed in this set was 4,808,472,349. This is not a proof for all possible inputs.

For the 50-question same-article Japanese fixture, hits@1 were 42 -> 42, hits@3 were 49 -> 49, and hits@5 were 50 -> 50. The correct document's rank improved on two questions and worsened on one; the top-five set changed on three. The benchmark labels only the originating passage relevant and is too small to establish retrieval parity on a real application corpus. The separate [memory-study report](../bekko25-memory-study/RESULTS.md) describes earlier native results; the additional 100-question holdout from 20 distinct JaQuAD articles produced 79 -> 80 hits@1 and 100 -> 100 hits@5, with one correct rank worsening. That holdout was native-only.

The compact canister rejected a direct upgrade over the F32 storage magic without erasing the original verified pack; after restoring the old Wasm, the original pack activated successfully. A compact-to-compact upgrade retained the verified pack and required activation. Unauthorized activate, beginUpload and upload calls were rejected. A separate test upgraded the compact canister from raw Cargo Wasm to the ICP recipe artifact; 15 lengths matched bit-for-bit after the upgrade and remained below 4.9B instructions.

- F32 raw Wasm SHA-256: `e6559a5fba9b544c1377f06ec595c37f27498015b4b19311fb04a77b7c6e072c`.
- Compact raw Wasm SHA-256: `46e8310487895dcd245390312e4aba6097e3ea8e1457bf61505f9748f81c7840`.
- Compact ICP recipe Wasm SHA-256: `de3aba3fc737bb5f3c74defdfd9c85920389525067948d52e577cffb2a0c8ba9`.
- F32 pack SHA-256: `8fc88d56d68f550d1a920bab7d1ec42af9449240048393bd6daabbfcfa861724`.
- Compact pack SHA-256: `71b8c47a0a508dd5cd3e704d83934e900c8e634d2c52e7a02a85e882fa5be9c4`.

[Build, verification and migration instructions](../../docs/BEKKO25-COMPACT.md). The detailed generated vectors are ignored by Git; `scripts/report_bekko25_compact.py` checks the pinned fixture and summarizes them.

The shared linear-weight change was also tested on the F32-vocabulary a25m canister: after an upgrade from runtime 2 to runtime 3, all 416 inputs matched bit-for-bit, including all supported lengths and repeat-token boundaries. Heap fell from 174,718,976 B to 151,453,696 B with stable memory unchanged at 418,447,360 B. The highest observed instruction count was 4,806,305,351. The runtime-3 raw Wasm SHA-256 was `90af5fc5de25318d6b562a72ea9502ff6c7aeaad3fed50b69aaabef6ba0d00be`. Reproduce with `bekko25-stable-verify`, the old/new Wasm paths and hashes, and `BEKKO25_VERIFY_LINEAR_I8=1`.

The shared Bekko kernel's a8m Wasm was separately upgraded and all 15 pinned cases matched bit-for-bit. Granite's raw Wasm SHA-256 matched the pre-change build (`61f9740eb09e28c232b967d520a9db806d0975bf1dfe6c56ccda8121e5aaf54f`); its generated direct kernels remain i16.

## Review fixes and revalidation

2026-10-31: re-ran the compact comparison after separating the packages and adding build provenance, stale-result protection, checked memory subtraction and retrieval-quality gates. All 475 cases passed, with minimum corresponding-vector cosine 0.981328 and maximum 4,808,472,349 instructions. Japanese hits@1 were 42 -> 42 and hits@5 were 50 -> 50; one correct rank worsened. The heap and stable-memory measurements remained 151,388,160 B and 124,518,400 B. The local report independently validated the build record and gates.

The reviewed raw compact Wasm SHA-256 was `8048f2220cf4b91d473b5aef7e617f776421c861921028bdb7ce1ee0d629c021`; its ICP recipe Wasm SHA-256 was `e77802fd1943cccb9ec016fe6b42df4c5ad490d298337cb22f62d1481eb09729`. A separate raw-to-recipe upgrade test matched outputs bit-for-bit on 15 inputs. Package unit tests and an isolated PocketIC `modelInfo` check confirmed that both the normal and `--all-features` builds of the original package select the original F32 pack and embedding space. Their Wasm hashes differed; they were not claimed to be bit-identical artifacts.

## Recipe artifact and input attestation

The 475-case comparison was repeated against the actual ICP recipe artifact `e77802fd1943cccb9ec016fe6b42df4c5ad490d298337cb22f62d1481eb09729`, linked to raw Cargo Wasm `8048f2220cf4b91d473b5aef7e617f776421c861921028bdb7ce1ee0d629c021` and `icp.yaml` SHA-256 `36c26675abfae6549667c15d6c331810b6f5350a2d8fa4eb941e7f91e42fe650`. PocketIC's installed and upgraded module hashes matched the recipe artifact. All 475 inputs passed; the maximum observed was 4,808,472,349 instructions. Japanese hits@1 were 42 -> 42 and hits@5 were 50 -> 50. The report rechecks the source/build/recipe records and hashes of both input fixtures. See the [compact guide](../../docs/BEKKO25-COMPACT.md) for installing the verified file and checking a live canister's module hash.

## Attention instruction headroom

2026-11-02. The original mainnet compact runtime reported up to 4,953,609,977 instructions on ten sampled 255-token queries. This exceeded the local 4.9B target. The same Wasm and English input reported 4,844,589,977 instructions on PocketIC 16 and 4,952,649,977 on mainnet; local measurements therefore do not establish the mainnet margin.

The new runtime shares the intersection of four Attention windows while retaining the original key order. In PocketIC 16, a disposable canister was installed with the previous recipe, loaded with the pinned compact pack, and upgraded to the new recipe. All 384 output components matched bit-for-bit for 14 inputs. English examples:

| Tokens | Old instructions | New instructions | Reduction |
|---:|---:|---:|---:|
| 192 | 3,518,020,853 | 3,450,480,937 | 67,539,916 |
| 255 | 4,844,589,977 | 4,737,827,513 | 106,762,464 |
| 256 | 4,832,875,000 | 4,724,403,602 | 108,471,398 |

The actual new recipe was also upgraded over the prior compact recipe and compared to the prior 475-case output corpus: all vectors matched bit-for-bit, the verified pack was retained, and reactivation succeeded. The highest reported count was 4,737,767,513. The corpus comprises Japanese and multilingual fixtures, seeded inputs of lengths 2–256, and repeated-token boundaries; it does not prove equivalence for every input. The separate a8m Wasm was byte-for-byte unchanged by this build.

The F32-vocabulary a25m canister was independently upgraded from runtime 3 to runtime 4 in PocketIC 11. All 416 inputs matched bit-for-bit, stable memory remained 418,447,360 B, and the post-activation heap was 151,453,696 B. The new F32 raw Wasm SHA-256 is `9bb600a6ad0171a834cedb12165d59c600c63d6459b9fca13ab279a4f3cb2e16`. Reproduce the lifecycle sweep with `bekko25-stable-verify` and `BEKKO25_VERIFY_ATTENTION=1`.

The standard PocketIC 11 compact release verifier and independent provenance report also passed all 475 inputs against the F32 baseline. On the 50-question Japanese fixture, hits@1 were 42 -> 42 and hits@5 were 50 -> 50. The new compact maximum was 4,701,787,513 instructions in this environment; its heap and stable memory were 151,388,160 B and 124,518,400 B. This small retrieval fixture does not establish quality on other data.

The old recipe SHA-256 is `e77802fd1943cccb9ec016fe6b42df4c5ad490d298337cb22f62d1481eb09729`; the new recipe SHA-256 is `e6ab7a0d507d8fd54cc2a724432afee73fafe8b435ce946244da224e49ccad64`. The compact pack SHA-256 remains `71b8c47a0a508dd5cd3e704d83934e900c8e634d2c52e7a02a85e882fa5be9c4`. Detailed output vectors remain local. The new recipe has not been measured on mainnet, so the local reductions are not a mainnet guarantee.
