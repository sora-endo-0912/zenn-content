---
title: "ループ型 Transformer は推論を隠すのか — GPT-6 Astra を運用者目線で読む"
emoji: "🔁"
type: "tech"
topics: ["llm", "transformer", "openai", "ai", "machinelearning"]
published: true
---
> 本記事は要約です。初出: https://aether-echoes.com/posts/looped-transformers-hidden-reasoning-gpt6-astra-operator-view

## 結論: ループ型 Transformer は推論を隠す構造ではない

2026 年 9 月に出た GPT-6 Astra について「ループ型 Transformer（同じ層を何度も通す構造）で思考を隠している」という見立てが広がった。発端は The Information の「recurrent depth 採用」という未確認報道で、そこから「推論トレースが短いのは内部に隠しているからでは」という憶測が一人歩きした。

Sebastian Raschka の解説を土台に読み直すと、この因果はつながらない。層をループさせることと、思考を出力トークンに出すかどうかは別のレイヤーの話だ。課金される側が気にすべきは「隠れたか」ではなく、監視可能性とコスト構造がどう変わるかだと私は読んだ。

## 減るのは「保存するパラメータ」だけ

ループ型は同じブロックを使い回し、重みを増やさずに実効的な深さを稼ぐ。Nanbeige4.2-3B は 22 ブロックを 2 周させ、44 層相当の計算をしながら重みは 22 セット分しか持たない。

落とし穴はここだ。減るのは保存パラメータだけで、計算量は 44 層相当のまま、KV キャッシュも周回ごとに別途必要になる。パラメータが半分でも財布が半分になるわけではない。

## 短いトレースは「隠蔽」ではなく「効率」

賢いモデルほど少ない下書きトークンで解く。トレースが短いのはループ特有の現象ではなく、問題解決の効率差だ。Full-Bandwidth Transformer の所見でも、latent feedback がトレースを短くする効果は instruction tuning 後に消えた。トレースの長さは学習依存であって、構造で決まらない。

## 運用者に効く 3 点

- **監視可能性**: 出力トークンから意思決定を追う従来の監視が効きにくくなる
- **コスト構造**: 内部の周回数は API 側から見えない。「安くなる」と早合点しない
- **推論時スケーリング**: 難しいトークンにだけ深く計算を割り当てられる点は素直に利点

本家では Universal Transformers から Mixture-of-Recursions / Ouro までの系譜を表で整理し、規模で損得が反転する話と、ARC-AGI-3 の跳躍を構造だけで説明できない理由まで掘っている。

全文はこちら: https://aether-echoes.com/posts/looped-transformers-hidden-reasoning-gpt6-astra-operator-view
