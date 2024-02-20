# slide_making


音声付きのmp4ファイルを rstudio で作成する。
使うのは webshot パッケージとariパッケージ。ffmpeg を別途インストールする必要あり。amazon polly の ID も必要。

参考にしたのは[ここ](https://qiita.com/kazutan/items/3b7db5cc572057e551ed)

```
ari_narrate(script = "ioslides.Rmd", slides = "ioslides.html", output = "video.mp4", voice = "Takumi", delay = 16, capture_method = "iterative")
```
## ioslidesとxaringan

xaringan が対応しているとのことだが、webshot で画像をうまく取得できない。市松模様になっちゃう。

ということでioslides_presentationでやってみる。

## 音声がgoogle chromeのタブで再生されない

職場のマシン(windows11)で実行した結果、できあがったmp4ファイルは、どういうわけか google chrome のタブで再生すると、映像は流れるが音が出ない。スピーカーのアイコンがグレーアウトした状態。（ほかのソフトで再生すると音は出る。ただし映像が最初黒いが、なんかしてるうちに見られるようになる。ようわからない）

google meet で動画共有したいのだが、このままだと音声が流れない。

```
ffmpeg -i video.mp4 video.mp3
```
として、音声を抽出して video.mp3 を作ってから、音声を上書きすることにした。
（ちょっとださいが、まにあわせ）

実際の命令はつぎのとおり。

```
ffmpeg -i video.mp4 -i video.mp3 -c:v copy -c:a aac -map 0:v:0 -map 1:a:0 output.mp4
```

結果でき上った output.mp4 は google chrome のタブ上で音声も映像も再生される。

映像と音声の結合に関して、参考にしたサイトは[ここ](https://qiita.com/niusounds/items/f69a4438f52fbf81f0bd)

> ポイントは -c:v copy を指定して動画を再エンコードしないようにしていることと、-map 0:v:0 -map 1:a:0 を指定して元の動画ファイルに音声トラックが含まれている場合でも確実に音声ファイルの音声トラックを利用するようにしているところです。
元の動画ファイルに音声が含まれていないのであれば、-map の指定はなくても良いです。

## phantomjsがうまくいかない

職場のマシン(Windows11)ではうまくいくが，linux だと phantomjs がうまくいかない。

linux ではインストールのときもつまづいたが
[これ](https://stackoverflow.com/questions/76751971/installing-phantomjs-in-r)を参考にうまく行った。ただ、その後、調子が悪く、ローカルにある html をキャプチャしようとして
```
webshot::webshot("hoge.html")
```
とすると、
```
PhantomJS has crashed. Please read the bug reporting guide at
<http://phantomjs.org/bug-reporting.html> and file a bug report.
Error in webshot::webshot("hoge.html") : 
  webshot.js returned failure value: -8
```
とエラーが出る。インターネット上の url を指定するとキャプチャできるのも不思議。ググってもよくわからない。

phantomjsがうまくいかないから、webshot がうまくいかなくって、けっきょく ari もうまくいかない。まがりなりにも windows ならできるというのが、いかがなものか。