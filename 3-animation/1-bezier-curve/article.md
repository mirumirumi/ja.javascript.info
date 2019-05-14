# ベジェ曲線

ベジェ曲線は、CSSアニメーションやその他多くの場所で形状を描くために、コンピュータグラフィックスで使用されます。

実際には非常に単純なものですが、一度学んでおくと、ベクトルグラフィックスや高度なアニメーションの世界が受け入れやすくなるでしょう。

[cut]

## 制御点(Control points)

[ベジェ曲線](https://en.wikipedia.org/wiki/B%C3%A9zier_curve) は制御点/コントロールポイントによって定義されます。

制御点は 2, 3, 4 またはそれ以上あるかもしれません。

例えば、2つの制御点の曲線です:

![](bezier2.png)

3つの制御点の曲線です:

![](bezier3.png)

4点の曲線です:

![](bezier4.png)

これらの曲線をよく見るとすぐに気づくでしょう:

1. **点は必ずしも曲線上にあるわけではありません。** それは全く普通のことであり、後で曲線がどうやって作られるのかを見ていきます。
2. **曲線の次数は点の数から1を引いた数になります。** 2つの点の場合は線形曲線(直線)であり、3つの点の場合 -- 2次曲線(放物線), 4つの点の場合は -- 3次曲線を持ちます。
3. **曲線は常に制御点の[凸包(convex hull)](https://en.wikipedia.org/wiki/Convex_hull) の内側にあります。:**

    ![](bezier4-e.png) ![](bezier3-e.png)

最後の性質により、コンピュータグラフィックスでは、交差テスト(intersection tests)を最適化することができます。凸包が交差しない場合、曲線もまた交差しません。したがって、凸包の交差を最初にチェックすることで、非常に高速な "交差なし" の結果を得ることができます。交差や凸包のチェックはより簡単です。なぜなら、それらは四角形、三角形など(上の図を参照)であり、曲線よりもはるかに単純な図形だからです。

描画のためのベジェ曲線の主な価値は -- 点を動かすことによって、曲線が *直感的に分かりやすい方法で* 変化するということです。

下の例で、マウスを使って制御点を移動させて見ましょう:

[iframe src="demo.svg?nocpath=1&p=0,0,0.5,0,0.5,1,1,1" height=370]

**お気づきのように、曲線は接線 1 -> 2 and 3 -> 4 に沿って伸びます。**

いくつかの練習を経て、必要な曲線を得るための点の配置の仕方が明らかになります。そして、複数の曲線を繋ぐことで、実際になにかを得ることができます。

ここではいくつかの例です:

![](bezier-car.png) ![](bezier-letter.png) ![](bezier-vase.png)

## Maths

ベジェ曲線は数学的な式を使って記述することができます。

この後すぐに見ていきますが -- 知る必要はありません。が、完全性のためにここで説明します。

与えられた制御点の座標 <code>P<sub>i</sub></code>: 最初の制御点は座標 <code>P<sub>1</sub> = (x<sub>1</sub>, y<sub>1</sub>)</code> であり、2番目は: <code>P<sub>2</sub> = (x<sub>2</sub>, y<sub>2</sub>)</code> などとなり、曲線の座標は区分 `[0,1]` からのパラメータ `t` によって決まる式で記述されます。

- 2点の曲線の式:

    <code>P = (1-t)P<sub>1</sub> + tP<sub>2</sub></code>
- 3点の場合:

    <code>P = (1−t)<sup>2</sup>P<sub>1</sub> + 2(1−t)tP<sub>2</sub> + t<sup>2</sup>P<sub>3</sub></code>
- 4点の場合:

    <code>P = (1−t)<sup>3</sup>P<sub>1</sub> + 3(1−t)<sup>2</sup>tP<sub>2</sub>  +3(1−t)t<sup>2</sup>P<sub>3</sub> + t<sup>3</sup>P<sub>4</sub></code>

これらはベクトル方程式です。

これらを座標ごとに書き直すことができます。例えば、3点の曲線の場合:

- <code>x = (1−t)<sup>2</sup>x<sub>1</sub> + 2(1−t)tx<sub>2</sub> + t<sup>2</sup>x<sub>3</sub></code>
- <code>y = (1−t)<sup>2</sup>y<sub>1</sub> + 2(1−t)ty<sub>2</sub> + t<sup>2</sup>y<sub>3</sub></code>

<code>x<sub>1</sub>, y<sub>1</sub>, x<sub>2</sub>, y<sub>2</sub>, x<sub>3</sub>, y<sub>3</sub></code> の代わりに、3つの制御点の座標を入れます。

例えば、制御点が `(0,0)`, `(0.5, 1)` と `(1, 0)` の場合、方程式は次のようになります:

- <code>x = (1−t)<sup>2</sup> * 0 + 2(1−t)t * 0.5 + t<sup>2</sup> * 1 = (1-t)t + t<sup>2</sup> = t</code>
- <code>y = (1−t)<sup>2</sup> * 0 + 2(1−t)t * 1 + t<sup>2</sup> * 0 = 2(1-t)t = –t<sup>2</sup> + 2t</code>

今、 `t` が `0` から `1` まで実行されると、各 `t` の値 `(x,y)` の集合が曲線を形成します。

恐らくあまりに科学的な内容で、なぜ曲線がそのように見えるのか、そしてそれらが制御点にどのように依存するのかはあまり明白ではないと思います。

なので、ここでは理解しやすい描画アルゴリズムを紹介します。

## ド・カステリョのアルゴリズム

[ド・カステリョ(De Casteljau's)のアルゴリズム](https://en.wikipedia.org/wiki/De_Casteljau%27s_algorithm) は曲線の数学的定義と同じですが、それがどのように構築されているかを視覚的に示しています。

3点の例を見てみましょう。

ここにデモとその説明が続きます。

点はマウスで動かすことができます。"play" ボタンを押して実行してみてください。

[iframe src="demo.svg?p=0,0,0.5,1,1,0&animate=1" height=370]

**3点のベジェ曲線を構築するド・カステリョのアルゴリズムです。**

1. 制御点を描画します。上のデモでは、それらは `1`, `2`, `3` とラベル付けされているものです。
2. 制御点 1 -> 2 -> 3 の間に線分を作ります。上の例では、それらは <span style="color:#825E28">茶色の線</span> です。
3. パラメータ `t` は `0` から `1` まで移動します。上の例では、一区切り `0.05` が使用されています。: ループは `0, 0.05, 0.1, 0.15, ... 0.95, 1` となります。

    それぞれの `t` の値について: 

    - 各 <span style="color:#825E28">茶色</span> の線分上では、その線の開始部分から `t` に比例した距離にある点を取ります。2つの線分があるので、2つの点があります。

        例えば、 `t=0` の場合 -- 両方の点は線分の先頭になり、`t=0.25` の場合は -- 開始から25%の線分の長さになります。`t=0.5` だと -- 50%(中央), `t=1` では -- 線分の終わりになります。

    - 点を結びます。下の図では結んでいる線分は <span style="color:#167490">青</span> で描かれています。


| `t=0.25` の場合           | `t=0.5` の場合          |
| ------------------------ | ---------------------- |
| ![](bezier3-draw1.png)   | ![](bezier3-draw2.png) |

4. 今、<span style="color:#167490">青</span> の線分は、同じ `t` の値に比例した距離で点を取ります。つまり、`t=0.25` (左図)の場合、線分の左端から1/4の位置に点があり、`t=0.5` (右図)の場合は -- 線分の中央に点があります。上の図では、点は <span style="color:red">赤</span> です。

5. `t` が `0` から `1` まで実行にするにつれて、`t` の各値は曲線に点を追加していきます。このような点の集合がベジェ曲線を形成します。それは上の図にある赤色の方物線です。

これは3点の場合の処理ですが、4点の場合も同様です。

4点の場合のデモです(マウスで点を移動させることができます):

[iframe src="demo.svg?p=0,0,0.5,0,0.5,1,1,1&animate=1" height=370]

アルゴリズム:

- 制御点は 1 -> 2, 2 -> 3, 3 -> 4 の線分で結ばれています。ここでは、3つの <span style="color:#825E28">茶色</span> の線分があります。
- `0` から `1` までの間隔における各 `t` に対して:
    - 先頭から `t` に比例した距離で、それらの線分上の点を取ります。これらの点は結ばれ、2つの <span style="color:#0A0">緑の線分</span> ができます。
    - 緑の線分上で、`t` に比例した点を取ります。これで1つの <span style="color:#167490">青の線分</span> が得られます。
    - 青の線分上で `t` に比例した点を取ります。上の例では、それは <span style="color:red">赤</span> です。
- これらの点が一緒に曲線を形成します。

このアルゴリズムは再帰的であり、任意の制御点の数で一般化できます。

与えられた N 個の制御点において、それらを結合し N-1 個の線分を得ます。

次に、`0` から `1` までのそれぞれの `t` について:
- 各線分上で `t` に比例した距離の点を取り、それらを結ぶと -- N-2 個の線分ができます。
- N-2 個の線分に対して、`t` に比例した距離の点を取り、それらを結びます -- N-3 個のセグメントができます。
- 1点だけを持つまで繰り返します。これらの点が曲線を作ります。

移動する曲線の例です:

[iframe src="demo.svg?p=0,0,0,0.75,0.25,1,1,1&animate=1" height=370]

その他の点の場合:

[iframe src="demo.svg?p=0,0,1,0.5,0,0.5,1,1&animate=1" height=370]

弧状の形:

[iframe src="demo.svg?p=0,0,1,0.5,0,1,0.5,0&animate=1" height=370]

滑らかではないベジェ曲線:

[iframe src="demo.svg?p=0,0,1,1,0,1,1,0&animate=1" height=370]

アルゴリズムが再帰的なので、5, 6, またはさらに多くの制御点を使用して、任意の順番のベジェ曲線を作ることができます。しかし、実際にはあまり有用ではありません。通常は 2, 3 の点を取り、複雑な線の場合は複数の曲線をくっつけます。開発と計算がよりシンプルになります。

```smart header="与えられた点を *通る* 曲線を描く方法？"
我々はベジェ曲線のために制御点を使います。ご覧の通り、制御点は曲線上にはありません。もしくはより正確に言うと最初と最後の制御点は曲線上にありますが、それ以外はありません。

別のタスクが必要なときもあります。: すべての制御点が単一の滑らかな曲線上にあるように、*複数の点を経由して* 曲線を描くというものです。このタスクは [補間/内挿](https://en.wikipedia.org/wiki/Interpolation) と呼ばれ、ここでは説明しません。

このような曲線の数式があります。例えば [ラグランジュ多項式](https://en.wikipedia.org/wiki/Lagrange_polynomial)です。

コンピュータグラフィックスでは、多くの点を結ぶ滑らかな曲線を構築するために[スプライン補間](https://en.wikipedia.org/wiki/Spline_interpolation) がよく使用されます。
```

## サマリ

ベジェ曲線は制御点により定義されます。

私たちはベジェ曲線の2つの定義を見てきました。

1. 数式を使った定義
2. 描画プロセスを使った定義: ド・カステリョのアルゴリズム

ベジェ曲線の良い特性:

 - 制御点を移動させることで、マウスで滑らかな線を描くことができます。
 - 複雑な形状は、いくつかのベジェ曲線で作ることができます。

利用法:

- コンピュータグラフィックス、モデリング、ベクトルグラフィックエディタ。フォントはベジェ曲線で記述されています。
- Web 開発では、-- Canvas 上のグラフィックスと SVG 形式で使われています。ちなみに、上の "ライブ" の例は SVG で書かれています。実際には、パラメータとして異なる点が与えられた単一のドキュメントです。別ウィンドウで開き、ソースを見ることができます: [demo.svg](demo.svg?p=0,0,1,0.5,0,0.5,1,1&animate=1).
- CSS アニメーションでは、アニメーションの経路とスピードを記述するために使われています。