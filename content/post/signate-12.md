---
title: "緯度経度から付近の地図を画像で保存するtip"
date: 2019-11-25T14:09:24+07:00
draft: false
type: "post"
---
先日、SIGNATEのマイナビコンペが終了しました。<br>
<br>
自分のチームではアイデア賞を入賞しましたが、そこで行った処理の一部についてまとめようと思います。（全体の振り返りはまた今度）<br>
<br>
コードは最後に書きます<br>
<h2>どんな処理をするのかな</h2>
1.foliumで地図データのhtmlを生成<br>
2.seleniumでhtmlファイルをクロップし、pngファイルで保存する<br>
するだけです。<br>
<br>
ただ、調べてもあまり情報がなかったので書こうかなと思いました。
<h2>①foliumで地図データのhtmlを生成</h2>
<p>そもそもfoliumを知っている人はどれくらいいるのでしょうか。少なくとも僕は今まで知りませんでした。</p>
<br>
<a href="https://qiita.com/pork_steak/items/f551fa09794831100faa">https://qiita.com/pork_steak/items/f551fa09794831100faa</a>
<br>
ここの記事がわかりやすかったです。<br>
<br>
試しにやってみるとこのようになります。（読み込みが終わってないですが）<br>
<br>
引数に緯度経度を渡してあげれば勝手にファイルを作ってくれるのでかんたんにできます。<br>
<br>
ちなみに白黒や地形図などいろんな種類の地図を出力させることもできます。<br>
<style>
.resizeimage { width: 75%; }
.resizeimage img { width: 100%; }
</style>
<br>
<img src=https://cdn.shortpixel.ai/client/q_glossy,ret_img,w_1166/https://chizuchizu.com/blog/wp-content/uploads/2019/11/Screenshot-from-2019-11-19-19-28-34.png width="700px">
<br>
<h2>②seleniumでhtmlファイルをクロップすし、pngファイルで保存する</h2>
driver.save_screenshot()<br>
<br>
でできます。引数には保存するパスを指定します。<br>
<br>
地図データが意外に重いので2秒くらいは待ったほうがいいと思います（time.sleep(2)）<br>
<img src=https://cdn.shortpixel.ai/client/q_glossy,ret_img,w_1186/https://chizuchizu.com/blog/wp-content/uploads/2019/11/Screenshot-from-2019-11-19-19-45-50.png width="700px">
<br>

気を使うとしたら、ファイルのパスですね。<br>
<br>
driver.get()では絶対パスが必要です。<br>
<br>
sys.pathでワーキングディレクトリを取得するので、それに相対パスにくっつけることで絶対パスにすることができます。<br>
<br>

```python
driver.get("file:///{}/sample.html".format(sys.path[0]))</code></pre></div>
```

<h2>まとめ</h2>
あらかんたん。調べてみれば案外かんたんにできるものですね。<br>
<br>
マイナビコンペではこれを使って全データの地図を取得しました。<br>
<br>
データ数が多く、1回のループに5秒くらいかかったのでjoblibで全コア使用し一晩ぶん回してやっと終わりました。<br>
<br>
train, test合わせて20GB超えてしまったので驚きました（Google Driveの100GBプラン入っててよかった）<br>
<br>
 <br>
<br>
コードをおいておきます

```python
import folium
from selenium import webdriver
from selenium.webdriver.chrome.options import Options
import time
import sys
import chromedriver_binary
 
copyright_osm = '&copy; <a href="http://osm.org/copyright">OpenStreetMap</a> contributors'
 
 
def make_html(lng, lat):
    # zoom_startは拡大率みたいなもの（大きいほうが拡大される）
    map_data = folium.Map(location=[lng, lat],
                          attr=copyright_osm,
                          zoom_start=18)
    map_data.save("sample.html")
    get_img()
 
 
def get_img():
    options = Options()
    options.add_argument("--headless")
    driver = webdriver.Chrome(options=options)
 
    driver.get("file:///{}/sample.html".format(sys.path[0]))
 
    time.sleep(2)
 
    driver.save_screenshot("sample.png")
 
 
make_html(35.68, 139.78)
```