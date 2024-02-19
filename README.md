# slide_making


音声付きのmp4ファイルをrstudioで作成する。

```
ari_narrate(script = "ioslides.Rmd", slides = "ioslides.html", output = "video.mp4", voice = "Takumi", delay = 16, capture_method = "iterative")
```

職場のマシンで実行した結果、できあがったmp4ファイルは、どういうわけかgoogle chromeのタブで再生すると、映像は流れるが音が出ない。スピーカーのアイコンがグレーアウトした状態。（ほかのソフトで再生すると音は出る。ただし映像が最初黒いが、なんかしてるうちに見られるようになる。ようわからない）

meetで動画共有したいのだが、このままだと音声が流れない。

```
ffmpeg -i video.mp4 video.mp3
```
として、音声を抽出してvideo.mp3を作ってから、音声を上書きすることにした。
（ちょっとださいが、まにあわせ）

実際の命令はつぎのとおり。

```
ffmpeg -i video.mp4 -i audio.wav -c:v copy -c:a aac -map 0:v:0 -map 1:a:0 output.mp4
```

結果でき上ったoutput.mp4はgoogle chromeのタブ上で音声も映像も再生される。

映像と音声の結合に関して、参考にしたサイトは[ここ](https://qiita.com/niusounds/items/f69a4438f52fbf81f0bd)

> ポイントは-c:v copyを指定して動画を再エンコードしないようにしていることと、-map 0:v:0 -map 1:a:0を指定して元の動画ファイルに音声トラックが含まれている場合でも確実に音声ファイルの音声トラックを利用するようにしているところです。
元の動画ファイルに音声が含まれていないのであれば、-mapの指定はなくても良いです。

