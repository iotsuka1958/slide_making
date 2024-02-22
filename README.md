# slide_making


音声付きのmp4ファイルをrstudioで作成する。
使うのは ari パッケージ。別途 webshot パッケージ（裏で動く phantomjs をインストールしておく必要あり）。ffmpeg を別途インストール（rstudio が ffmepg を見つけられるように path を通しておくこと）。amazon polly の ID も必要（これがけっこうてまどった）。

参考にしたのは
[ここ](https://qiita.com/kazutan/items/3b7db5cc572057e551ed)
と
[ここ](https://johnmuschelli.com/ari_paper/)

```
ari_narrate(script = "hoge.Rmd", slides = "hoge.html", output = "video.mp4", voice = "Takumi", delay = 10, zoom = 2, capture_method = "iterative")
```

hoge.Rmd（読み原稿をコメントアウトした形で入力したやつ）およびそれでつくった hoge.html を用意する。出力動画を video.mp4 とする。voice は、現状で Takumi（男性）と Mizuki（女性）のどちらかを指定。zoom=2くらいにしておいたほうができあがり画像がきれい。


## ioslides と xaringan

xaringan が対応しているとのことだが、webshot で画像をうまく取得できない。市松模様になっちゃう。

ということで ioslides_presentation でやってみる。

## 音声が google chrome のタブで再生されない

職場のマシン (windows11) で実行した結果、できあがったmp4ファイルは、どういうわけか google chrome のタブで再生すると、映像は流れるが音が出ない。スピーカーのアイコンがグレーアウトした状態。（ほかのソフトで再生すると音は出る。ただし映像が最初黒いが、なんかしてるうちに見られるようになる。ようわからない）

google meetで動画共有したいのだが、このままだと音声が流れない。

```
ffmpeg -i video.mp4 video.mp3
```
として、音声を抽出して video.mp3 を作ってから、音声を上書きすることにした。
（ちょっとださいが、まにあわせ）

実際の命令はつぎのとおり。

```
ffmpeg -i video.mp4 -i video.mp3 -c:v copy -c:a aac -map 0:v:0 -map 1:a:0 output.mp4
```

結果でき上ったoutput.mp4はgoogle chromeのタブ上で音声も映像も再生される。

映像と音声の結合に関して、参考にしたサイトは[ここ](https://qiita.com/niusounds/items/f69a4438f52fbf81f0bd)

> ポイントは `-c:v copy` を指定して動画を再エンコードしないようにしていることと、`-map 0:v:0 -map 1:a:0` を指定して元の動画ファイルに音声トラックが含まれている場合でも確実に音声ファイルの音声トラックを利用するようにしているところです。
元の動画ファイルに音声が含まれていないのであれば、`-map` の指定はなくても良いです。

## pdf で作成したスライドから mp4 をつくる

### Rmd から自動作成するときの問題点

Rmarkdown ファイルおよびそれでつくった html ファイルから、mp4 を作成するのは、すごくかんたん。ほぼ自動的にできる。

ただ、次のようなデメリットもある。

- html から画面をキャプチャするのに、ari::ari_narrate() を走らせると、うまくいかないことがある（裏で動いている webshot::webshot()、さらにその裏で動いている phantomjs が原因らしい。ぐぐると phantomjs は開発が終了しているらしくて、非推奨らしい）
- とりあえず mp4 はできあがっても、キャプチャした画像が、html で見たときと違うところがでてくることがある（htmlでは表示できている文字がうまくでないところがある。色がとぶところもある）
- ioslides_presentation でつくった html ファイルだとうまくいくが、xaringan でつくった html ファイルだと、うまくキャプチャできない
- ioslides_presentaion で widescreen: true とすると、最後の数枚のスライドがうまくキャプチャできない

Rmd さえつくってしまえば、すごくかんたんで魅力的なやり方だが、細部にこだわりたい場合もあるだろう。ということで、pdf で作成したスライドから mp4 をつくってみる。



### 手順


1. pdf スライドを用意
1. 各ページを png に変換
1. 別途、読み原稿を準備（詳細はあとで）
1. ari::ari_spin() で mp4 作成
1. 音がうまくのっていない場合は、先ほど書いたやり方で音を乗せなおす

### pdf → png

いろいろあるだろうが、次のやり方がかんたん。

```
pngs <- pdftools::pdf_convert("slide.pdf", dpi = 300)
```

これで、slide.pdf が slide_1.png, slide_2.png ... みたいにページごとの png ファイルに変換される。と同時にオブジェクト pngs に、できあがった複数のファイル名が格納されるので、のちのち便利。

dpi で解像度を 300 に指定している。

### 読み原稿の準備

スライドの枚数分の読み原稿を作成し、オブジェクト notes に格納する。

たとえば3枚のスライドなら、つぎのように長さ3のオブジェクトを作成する。

```
notes <- c("1枚目の原稿です", "ここは2枚目です", "もちろんここは3枚目の原稿")
```

ここが、いちばんめんどくさいかも。

### ari::ari_spin() で mp4 作成

ページごとの png と 読み原稿が準備できれば、もうあとちょっと。

いまオブジェクト  pngs には、png ファイル名がはいっている。オブジェクト notes には、読み原稿が格納されている。

```
ari_spin(png, notes, output = "output.mp4", voice = "Takumi", divisible_height = TRUE)
```

これで output.mp4 のできあがり。:clap:

これだと
- pdf 見た目どおりの動画ができる:rocket:

## Other Documents

pdf から動画ができるということは、たとえば powerpoint でつくったスライドなんかも pdf にすれば、動画ができるということになる。

ノートつきの pptx ファイルなら次のやり方でノートを取り出せるようだ。

```
notes <- ariExtra::pptx_notes("slide.pptx")
```

powerpoint の世界単体で動画はできるんだろうが、AI 音声つきの動画ができるというのが強みだとおもう。

わたしは powerpoint を使うことはないから、ありがたみはない。
