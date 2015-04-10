# 前半

## どう実装するか考える
	
1.方向キーを取得する（入力）
2.リアルタイムにデータを送信する（通信）
3.方向に対応した画像を描画する（出力）
	
という流れを、もっと具体的にしていきます。


1.方向キーを取得する（入力）
- jQueryのon関数でkeypressイベントを取得
2.リアルタイムにデータを送信する（通信）
- milkcocoaのdataStore.send関数でどの方向のキーが押されたかを送信する
- 別ファイルのdataStore.on関数でどの方向のキーが押されたかを取得する 
3.方向に対応した画像を描画する（出力）
- jQueryのattr操作でimgタグに対応する画像URLを代入する

といった具合で、jQueryとmilkcocoaによる実装を行いましょう。
早速始めていきます。


## お手本のソースコードをzipファイルとしてダウンロード
[お手本](https://github.com/shogochiai/milkcocoa-sync-practice)にアクセスして、右下のzip downloadのボタンを押します。

## 入力ページ
### htmlで入力を行うために必要なファイルを読み込み
まずはinput/index.htmlを読みます
必要なファイルが読み込まれているだけです。

```:input/index.html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8"/>
    <meta http-equiv="X-UA-Compatible" content="IE=edge"/>
    <title>input test</title>
  </head>
  <body>
    <script src="https://cdn.pubnub.com/pubnub-dev.js"></script>
    <script src="milkcocoa-sync.js"></script>
    <script src="https://ajax.googleapis.com/ajax/libs/jquery/2.1.3/jquery.min.js"></script>
    <script src="input.js"></script>
  </body>
</html>
```

- pubnub
- jquery
- milkcocoa-sync
- input.js

を読み込む必要があります。		
このpubnubというファイルは、milkcocoa-syncの同期機能の元になっているライブラリです。

## 入力通信処理をmilkcocoaで実装
さて、input/index.htmlで読み込まれているinput/input.jsではmilkcocoaを実行します。
まずは
console.log(MilkCocoa);
を実行してみて、ちゃんとhtmlファイル中でmilkcocoaを読み込めているか確認しましょう。

```:input/input.js
(function(){
	console.log(MilkCocoa);
}());
```

### インスタンス生成
var milkcocoa = new MilkCocoa();
と書かれた行で、通信処理を行うためのMilkCocoaオブジェクトをmilkcocoa変数に格納しました。


```:input/input.js
(function(){
    var milkcocoa = new MilkCocoa();
}());
```

### dataStoreオブジェクト
var ds = milkcocoa.dataStore("hoge");
を実行すると、DataStoreオブジェクトを生成し、dsという変数に格納することができます。

```:input/input.js
(function(){
    var milkcocoa = new MilkCocoa();
    var ds = milkcocoa.dataStore("something");
}());
```

### send関数
先ほど生成したDataStoreオブジェクトから
ds.send();
という風にsend関数を呼び出すことで「入力データの送信」ができます。

```:input/input.js
(function(){
    var milkcocoa = new MilkCocoa();
    var ds = milkcocoa.dataStore("something");
    ds.send({});
}());
```

### どの矢印キーを押したかという情報を渡す
jQueryのkeydown関数を用いて、キーボードを叩いたことを検出します。ちなみに、documentはページ全体を示します。
コールバック関数という、keydownの度に呼ばれる関数があり、コールバック関数の引数eが、「どのキーが押されたか」の情報を持っています。
以下が、矢印キーを示すkeyCodeと、それを用いた条件分岐です。


```:input/input.js
(function(){
    var milkcocoa = new MilkCocoa();
    var ds = milkcocoa.dataStore("something");

    $(document).keydown(function(e){
        if(e.keyCode == 38){
            ds.send({arrow: "up"});
        } else if(e.keyCode == 40) {
            ds.send({arrow: "down"});
        } else if(e.keyCode == 37) {
            ds.send({arrow: "left"});
        } else if(e.keyCode == 39) {
            ds.send({arrow: "right"});
        } else {
         	 console.log("I'm not arrow!");
        }
    });
}());
```




## 出力ページ
### 入力ページと同様に必要なものを読み込むhtml
ただ1点異なるのは、bodyタグ内にimgタグがあることです。
入力ページで他の人が押した矢印キーに対応した画像が表示されるためですね。


```:output/index.html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8"/>
    <meta http-equiv="X-UA-Compatible" content="IE=edge"/>
    <title>output test</title>
  </head>
  <body>
    <img id="arrow-img" src="img/placeholder.jpeg" width="300px"></img>
    <script src="https://cdn.pubnub.com/pubnub-dev.js"></script>
    <script src="milkcocoa-sync.js"></script>
    <script src="https://ajax.googleapis.com/ajax/libs/jquery/2.1.3/jquery.min.js"></script>
    <script src="output.js"></script>
  </body>
</html>
```

### 入力を受け取る処理
ds.on("send", cb);
で行えます。（cbはコールバック関数）

```:output/output.js

(function(){

    var milkcocoa = new MilkCocoa();
    var ds = milkcocoa.dataStore("something");

    ds.on("send", function(e){});
}());


```


### on関数の中で画像を張り替える処理
jQueryのattr("src", url)関数を用います。
imgタグのsrc属性を書き換えることで、画像の入れ替えを行います。

```:output/output.js
(function(){

    var milkcocoa = new MilkCocoa();
    var ds = milkcocoa.dataStore("something");

    ds.on("send", function(e){
        $("#arrow-img").attr("src", "img/arrow-"+ e.arrow + ".jpeg");
    });
}());
```

## 完成

![スクリーンショット 2015-04-09 17.10.53.png](https://qiita-image-store.s3.amazonaws.com/0/26475/b7631ad3-ef00-367f-2911-40484b1a9312.png "スクリーンショット 2015-04-09 17.10.53.png")


[inputページ](http://milkcocoa-sync-input.bitballoon.com/)
[outputページ](http://milkcocoa-sync-output.bitballoon.com/)

inputページとoutputページを隣同士に並べて、inputページをアクティブにした状態で矢印ボタンを押してみましょう。

outputページの画像が変更されることを確認したら、成功です。

## 入力と出力についてもっと考えてみよう
### あるシグナルを、不特定多数の人が送ってくるってどういうことだろう？
milkcocoa-syncを使えば、例えばチャットのような文字データだったり、クリック位置だったり、ありとあらゆるデータを瞬間的に共有できる。
1対1の通信を考えたり、n対1の通信を考えたりすると、作れる表現のイメージが湧きやすいかもしれませんね。

### 入力がKinectのモーションデータだったら？
Kinectの入力データは時系列に沿って断続的に渡される座標データかと思われます。従って、send関数を連打してその度にon関数が呼ばれる具合になると思われます。

### 出力がjThreeの3DCGだったら？
on関数で受け取った座標データをjThreeの3DCGオブジェクトに反映させ続ける訳なので、タイムラグが少々ありそうですが、基本的に今回のサンプルをベースに開発できます。

## 前半まとめ
と、いう訳で、KinectとjThreeを連携するための必要最低限となるsend/on関数の使い方について学びました。

後半では、その他の関数についても学んで、milkcocoaの応用イメージをつけていきましょう。



