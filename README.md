# slide_making


音声付きの mp4 ファイルを rstudio で作成する。
使うのは ari パッケージ。別途 webshot パッケージ（裏で動く phantomjs をインストールしておく必要あり）。ffmpeg を別途インストール（rstudio が ffmepg を見つけられるように path を通しておくこと）。amazon polly の ID も必要（これがけっこうてまどった）。

参考にしたのは
[ここ](https://qiita.com/kazutan/items/3b7db5cc572057e551ed)
と
[ここ](https://johnmuschelli.com/ari_paper/)

## Rmd $\longrightarrow$ mp4

hoge.Rmd（読み原稿をコメントアウトした形で入力したやつ）およびそれでつくった hoge.html を用意する。
ここまでできれば、ほとんど終わったようなもの。

Rstudioで実際に打つコマンドはつぎのとおり。

```
ari::ari_narrate(
  script = "hoge.Rmd",
  slides = "hoge.html",
  output = "video.mp4",
  voice = "Takumi",
  delay = 10,
  zoom = 2,
  capture_method = "iterative"
)
```

出力動画を video.mp4 とする。voice は、現状で Takumi（男性）と Mizuki（女性）のどちらかを指定。
zoom=2 くらいにしておいたほうができあがりの画像がきれい。
delay の値が小さいと、画像がうまく取得できないことがある。ただし、大きくすると、各ページを画像ファイルに落としこむのに時間がかかる。
capture_method は "vectorized" と "iterative" があって、前者のほうが早いらしいが、ioslides
 で作成したスライドの場合は後者にしろということだ。

## ioslides と xaringan

xaringan が対応しているとのことだが、webshot で画像をうまく取得できない。ほとんど真っ黒になっちゃう。

ということで ioslides_presentation でやってみた。

## 音声が google chrome のタブで再生されない

職場のマシン (windows11) で実行した結果、できあがった mp4 ファイルは、どういうわけか google chrome のタブで再生すると、映像は流れるが音が出ない。スピーカーのアイコンがグレーアウトした状態。（ほかのソフトで再生すると音は出る。ただし映像が最初黒いが、なんかしてるうちに見られるようになる。ようわからない）

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

結果できあがったoutput.mp4はgoogle chromeのタブ上で音声も映像も再生される。

映像と音声の結合に関して、参考にしたサイトは[ここ](https://qiita.com/niusounds/items/f69a4438f52fbf81f0bd)

> ポイントは `-c:v copy` を指定して動画を再エンコードしないようにしていることと、`-map 0:v:0 -map 1:a:0` を指定して元の動画ファイルに音声トラックが含まれている場合でも確実に音声ファイルの音声トラックを利用するようにしているところです。
元の動画ファイルに音声が含まれていないのであれば、`-map` の指定はなくても良いです。

## pdf で作成したスライドから mp4 をつくる

### Rmd から自動作成するときの問題点

Rmarkdown ファイルおよびそれでつくった html ファイルから、mp4 を作成するのは、すごくかんたん。ほぼ自動的にできる。

ただ、次のようなデメリットもある。

- html から画面をキャプチャするのに、ari::ari_narrate() を走らせると、うまくいかないことがある（裏で動いている webshot::webshot()、さらにその裏で動いている phantomjs が原因らしい。ぐぐると phantomjs は開発が終了しているらしくて、非推奨らしい。どうやら、一部の unicode 文字がはいっていると蹴られるみたい）
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

### pdf スライド

最初から pdf で作成すればそれでよし。

もとのスライドを html でつくった場合は、なんらかの手段で pdf に変換する。
この段階で、きれいな pdf になっていないとだめ。

powerpoint で作成した "hoge.pptx" を pdf にするには、`docxtractr::convert_to_pdf("hoge.pptx"")`とする方法が[ここ](https://johnmuschelli.com/ari_paper/)
で紹介されているが、手元の pptx ファイルで試したところ、きれいな pdf にならなかった。

adobe acrobat で pdf 化したら、うまくいった。

### pdf $\longrightarrow$ png

pdf が用意できれば、つぎは画像ファイルに変換。
png が推奨らしい。

いろいろあるだろうが、次のやり方がかんたん。

```
pngs <- pdftools::pdf_convert("slide.pdf", dpi = 300)
```

これで、slide.pdf が slide_1.png, slide_2.png ... みたいにページごとの png ファイルに変換される。と同時にオブジェクト pngs に、できあがった複数のファイル名が格納されるので、のちのち便利。

dpi で解像度を 300 に指定している。

試していないが、poppler の pdftoppm を使うのもありかな。これは Rstudio上じゃなくて、ターミナルでやっておく。

```
pdftoppm -png slide.pdf hoge
```

とすれば、各ページが hoge-1.png, hoge-2.png みたいになる。
ただし、この場合は、できあがった複数の png ファイルを、オブジェクト pngs に格納しておく。

```
pngs <- list.files(pattern = "^hoge") 
```

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


## 落穂ひろい

### 減量

できあがった mp4 を減量するには
``` 
ffmpeg -i hoge.mp4 -r 1/10 -ac 1 -ar 16000 oyoyo.mp4
```
とかする。`-r 1/10`で1/10[フレーム/秒]にしている。この数字が大きくなるとファイルサイズが大きくなる。なお、1未満の数値は小数ではなく分数を使えということらしい。この数値が動画の長さに関係してくるとのこと。`-ac 1`は音声をモノラルにする。`-ar 16000`は音声のサンプリングレートを16[khz]にしている。
