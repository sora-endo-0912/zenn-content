---
title: "VOICEVOX の CPU 版 Docker、並列 4 でも逐次と同じ 92 秒だった"
emoji: "🔊"
type: "tech"
topics: ["voicevox", "docker", "typescript", "nodejs"]
published: true
---
> 本記事は要約です。初出: https://aether-echoes.com/posts/voicevox-cpu-docker-sequential-vs-parallel-synthesis

## 4 分の動画に 11 分半。音声合成の for ループを疑った

記事 1 本を 12 段落の台本にして 4 分の解説動画を作ると、音声合成 115 秒、Remotion の render 410 秒、upload 15 秒で合計 11 分 30 秒かかる。音声側は段落ごとに `await synthesize(...)` を順に回しているだけなので、Promise.all で 2 本、4 本と並べれば半分になると読んだ。

ならなかった。1 秒も。

## 逐次 91.5 秒、並列 2 で 91.7 秒、並列 4 で 92.6 秒

同じ 12 段落、同じ話者、同じコンテナ（`voicevox_engine:cpu-latest` 0.25.2）で逐次 / 並列 2 / 並列 4 を各 2 周した。wall time は 3 条件とも 91〜94 秒で並び、72 本の wav は段落ごとに sha256 まで一致した。並列にしても音は変わらず、速くもならない。

この環境では音声 1 秒を作るのに 0.53〜0.55 秒かかり、段落の文字数と綺麗に比例する。`speedScale` を 1.15 にすると音声が 14% 短くなり、合成時間もほぼ同じ比率で縮む。合成時間は入力の文字数ではなく、出力する音声の長さで決まっている。

## 並列にすると audio_query が 20 ミリ秒から 9 秒に化ける

逐次のときの audio_query は 20 ミリ秒前後。並列 2 にした途端、3 段落目の audio_query が 7,943 ミリ秒になった。テキストの読みを返すだけの処理に 8 秒はかからない。前の段落の synthesis が終わるまで待たされている。並列 4 では待ちが最大 25 秒まで伸びた。

クライアントで何本投げても、エンジンは 1 リクエストずつ順に片付ける。Promise.all は「並列に合成する」のではなく「エンジンの前に列を作る」だけだった。

## CPU は何をしても 196% で止まっている

決め手は `docker stats` の 1 行だった。コンテナには 4 CPU 見えているのに、6 周とも 196% 前後で張り付いている。voicevox_engine の README に「CPU スレッド数が未指定なら論理コア数の半分を使う」とあり、起動コマンドは `--cpu_num_threads` を渡していない。4 の半分で 2 スレッド、196% はぴったりその数字だ。

並列化で埋まる余白は最初から無かった。触るなら `--cpu_num_threads 4` か Docker Desktop の CPU 割り当てで、どちらも音声側の 2 分を縮める話にしかならない。11 分を縮めたければ削る場所は render の方だ。

本家では条件別の計測表と並列時の段落ごとの内訳ログ、wav の mtime から昨日の 115 秒を裏取りした手順、動画 1 本の所要を見積もる式（音声 ≒ 読み上げ秒数 × 0.5、render ≒ 動画秒数 × 1.7）、そして compose が使えず `docker run` で立てている理由まで載せている。

全文はこちら: https://aether-echoes.com/posts/voicevox-cpu-docker-sequential-vs-parallel-synthesis
