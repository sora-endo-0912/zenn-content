---
title: "scroll-margin と scroll-padding の使い分け — 固定ヘッダーでアンカー先が隠れる問題を設計する"
emoji: "📐"
type: "tech"
topics: ["css", "frontend", "html", "アクセシビリティ"]
published: true
---
> 本記事は要約です。全文・コード例つきの初出はこちら → [scroll-margin と scroll-padding の使い分け（Aether Echoes）](https://aether-echoes.com/posts/scroll-margin-vs-scroll-padding-fixed-header-anchor)

固定ヘッダーのあるサイトで、ページ内リンクから見出しに飛ぶと、その見出しがヘッダーの裏に隠れて読めない——よくある不具合です。これを直す `scroll-margin` と `scroll-padding` は名前が似ているせいで混同されがちですが、**効く相手が違う**ので、そこを分けて考えると使い分けは自然に決まります。

## 二つの責務

- `scroll-padding` は**スクロールコンテナ**（多くは `:root` / `html`）に効く。「ビューポート内側にどれだけ安全地帯を作るか」を決める。
- `scroll-margin` は**スクロール先の要素**（見出しなど）に効く。「この要素の周りにどれだけ余白を持たせるか」を決める。

コンテナ側で一括して逃がすなら `:root { scroll-padding-block-start: var(--header-height); }`、要素側なら `:target { scroll-margin-block-start: var(--header-height); }`。見た目の結果はほぼ同じです。

## なぜ scroll-padding を第一選択にするか

差が出るのは**キーボード操作**です。`scroll-margin` はアンカーで飛んだ瞬間の停止位置しか補正しません。一方 `scroll-padding` はコンテナのスクロール量そのものを補正するため、スペースキーや Page Down の 1 画面送りにも効きます。`scroll-margin` だけだと、キーボードで読み進めるたびに上端の行がヘッダーの裏へ少しずつ食われていく。アクセシビリティの観点でも `scroll-padding` が筋がいいわけです。

## scroll-margin の出番

不要というわけではなく、**特定要素だけ止まる位置をずらしたいとき**と、`scroll-snap` で各スライドの吸着位置を要素ごとに調整したいときが出番です。横スクロールギャラリーで「最初の 1 枚だけ余白」のような例外は、コンテナの `scroll-padding` では表現しきれません。全体の基準はコンテナの `scroll-padding`、そこから外れる個別要素だけ `scroll-margin` で上書き——基準と例外という関係で重ねて使います。

## 実装の落とし穴

つまずくのは「ヘッダー高さの数値があちこちに散らばる」点。`64px` とハードコードすると、後でバナーを足して高さが変わったときアンカー位置だけ取り残されます。ヘッダー高さは **CSS 変数 1 個を SSoT** にして、`height` も `scroll-padding-block-start` も同じ変数を参照させれば防げます。`scroll-padding` は 2021 年に Baseline 入り済みで、今は対応を気にせず使えます。

## 持ち帰り

固定ヘッダーでアンカーが隠れたら、**まずコンテナに `scroll-padding-block-start`**、足りない個別要素だけ `scroll-margin` で上書き。判断材料は「コンテナ全体の話か、特定要素だけの話か」のひとつだけです。

---

全文（コード例・smooth scroll との相性・論理プロパティの解説つき）はこちら → **https://aether-echoes.com/posts/scroll-margin-vs-scroll-padding-fixed-header-anchor**
