
# GANとDCGAN走り書き

## @moratorium08

---


## 生成モデル

--

データセットがあたえられる。データセットが従う確率分布を求めよ！

というのが問題。つまり、データを生成するようなモデルを考えたい。

--

#### 絵が描ける！

大量の画像を学習させることで、その画像から、新しい画像を生成できる。

--

ネットで見かけたもの

* 2次元イラスト
* 人の顔
* 風景
* 部屋の画像
* 数字
* 家紋
* etc...

---

## Generative Adversarial Nets

[arXiv](https://arxiv.org/abs/1406.2661)
（前半部分だけをかなりざっと）

---
## Introduction

--

* Deep learningの識別モデルでの成功技術
	* 誤差逆伝播、Dropout、ReLU
* 生成モデルにおいては、最尤推定などをするときに生じる、大量の確率計算の推測することやReLUの恩恵を受けることが難しい

なんとかならないか。

--

#### Generative Adversarial Nets(GAN)の特徴


生成モデルと識別モデルを敵対させる。これだけ。

以後識別モデルをD(Discriminator)、生成モデルをG(Generator)とする

--

つまり

* Dは、与えられた入力が生成モデル由来が本物のデータセット由来かを学習
* Gは、Dが間違えるようなデータを生成することを学習

Gはまさにデータセットそっくりなデータを作るようになる


--


ちょうど偽札を作る人と警察の関係
これを続けていくとイタチごっこ的に学習していく。

---

## Related Work

--

* Deep Boltzmann Machine
	* 尤度最大化が複雑で近似が必要
* Generative Stochastic Networks
	* 明示的には尤度を計算しない
	* 誤差逆伝播をする
* Variational Auto Encoder
	* 二つのネットを持つが片方は近似に使う

---

## Adversarial nets

--

データ$\boldsymbol{x}$から、元の確率分布$p_g$を学習することを目標

まず、二つの多層パーセプトロンを用意する。

* 一つは、ノイズ$\boldsymbol{z}$を入力として、$G(\boldsymbol{z};\theta_g)$をマップ
	* $G$は、微分可能な関数で、重みとして$\theta_g$をもつニューラルネット。

* もう一つは、$D(\boldsymbol{x};\theta_g)$として、出力として一つのスカラー値
	* これは、データ$\boldsymbol{x}$が、サンプルデータから来た確率

--

#### 学習のステップ

$D$が$\boldsymbol{x}$の出所を正しく発見できるような確率を最大化しつつ、同時に$G$の学習として$log(1-D(\boldsymbol{z}))$を最小化するように学習する。

すなわち、$D$と$G$は、価値関数$V(G,D)$のminmaxゲーム
$$
min\_{G}max\_{D}V(D,G)=\\boldsymbol{E}\_{\\boldsymbol{x}\\sim p_{data}{\\boldsymbol(x)}}[logD(\\boldsymbol{x})] +$$

$$
 \\boldsymbol{E}\_{\\boldsymbol{z}\\sim p_z(\\boldsymbol{z})}[log(1-D(G(\\boldsymbol{z})))]
$$

により最適化する。


---
### 問題点
・安定性が低い

--

そこで CNNの力を借りるという発想が生まれる。

---

## Unsupervised Representation Learning With Deep Convolutional Generative Adversarial Networks

[arXiv](https://arxiv.org/abs/1511.06434)
(全部やります）

---


## Abstract

--


* 教師あり学習に対しては、CNNは大きな成果
* 教師なし学習に対しても適用
* GもDも対象をよく階層的に捉えることができた
* 学習データは画像の特徴として適用可能

---

## Introduction

--

GANはあんまり安定しない。この論文では、
* 多くの場合に安定に収束するようなDC-GANの設計
* Discriminatorを画像分類のタスクに使う
* GANの学習を可視化
* Generatorのvectorは簡単な算術操作に対して有意義な性質を持つ

--


#### 自然な画像生成

パラメトリックなものとノンパラメトリックなものがあるが、主にパラメトリックなものがよい。
パラメトリックな手法としては、
* テクスチャ合成
	* 自然なものを作るのは最近になってもあまり成功しない
* Variational Sampling
	* ぼける
* GAN
	* ノイズがつよいし、理解不能なものになる
	* Laplacian Pyramid を使ってもやはり微妙
* RNNやDeconvolutionもある程度成功しているが、実用されていない

--

#### CNNを視覚化する
NN自体、ややブラックボックスだが、CNNが畳み込み的な近似を行っていることや、勾配降下は理想的な画像を検査していると示されている
---


## Approach And Model Architecture

--

CNNでGANを拡張するのは簡単ではないが、できた。

方針の本質は、<strong>最新のCNNの手法を適用すること</strong>

--

* Poolingを StrideのあるConvolutionに置き換える
* 畳み込み層の特徴の上（？）にある全結合層を取り除く
	* Global Average Poolingがこの例（らしい（よくわからない
	* とにかく深いネットワークでは隠れ層で全結合は削除する

--

* Batch Normalizationを使う
	* 全部に適用するのはよくなく、不安定になるため、GのoutputとDのinputには適用しない。
* GeneratorにReLUを使う。ただしoutputはtanh
*  DiscriminatorにLeakyReLUを使う

--

#### 参考

* ReLU

$$
f(x) = max(0, x)
$$

* LeakyReLU
$$
f(x) = max(cx, x);
$$
(c は0.001みたいな小さい値)

多分職人芸。

---

## Details of Adversarial Training

--

LSUN, Imagenet-1k, 画像のデータセットで実験

学習については、
* 画像はGeneratorのoutputであるtanhにあうように[-1,1]に正規化
* 128のミニバッチでSGDで学習
* LeakyReLUのleakは0.2に設定
* またOptimizerにAdamを使用。学習率は0.001推奨だが0.0002の方がよかった。またmomentumは0.5がよかった


---

## Empirical Validation Of DCGANs Capabilities

--

一般に、教師なし学習の能力を測るために、教師なし学習を特徴抽出にして、これらの特徴から学習してみるという手法がある。これで学習したところ、Exemplar CNNにはおよばないものの、良い精度が出ているとわかる。

--

![CIFAR10](https://lh3.googleusercontent.com/-qFPouGpmeaA/WDA0_4P8aqI/AAAAAAAAFic/IabHt-2HDLQbh0kG2aWRag0bkSfl1_JbgCLcB/s0/%25E3%2582%25B9%25E3%2582%25AF%25E3%2583%25AA%25E3%2583%25BC%25E3%2583%25B3%25E3%2582%25B7%25E3%2583%25A7%25E3%2583%2583%25E3%2583%2588+2016-11-19+20.13.34.png "table1.png")

--

また他の解析として、StreetView House Numbers(SVHN)を用いても行った。良さそう。

--

![enter image description here](https://lh3.googleusercontent.com/-nGoJ9CgKzOg/WDBMH0zRUfI/AAAAAAAAFi4/7ek7ZXByHkIANZ_qITo8xC2w3Jwsuk2FACLcB/s0/%25E3%2582%25B9%25E3%2582%25AF%25E3%2583%25AA%25E3%2583%25BC%25E3%2583%25B3%25E3%2582%25B7%25E3%2583%25A7%25E3%2583%2583%25E3%2583%2588+2016-11-19+21.56.41.png "スクリーンショット 2016-11-19 21.56.41.png")

---



## Investigating And Visualizing The Internals Of The Networks

--

* もしG内にトレーニングデータが保持されているだけだとすると、内部には潜在的に歪んだ形で保持されていることになるはず。
* もしそうでないとすると、画像は何らかの追加と削除が加えられた"良い表現"になっているはず。

--

![result1](https://lh3.googleusercontent.com/-jp0Lffi3nho/WDCKv3BQ1kI/AAAAAAAAFjY/XQPCuzh-9Y4TgYJOqhC0D1FlXrlalxhvQCLcB/s0/%25E3%2582%25B9%25E3%2582%25AF%25E3%2583%25AA%25E3%2583%25BC%25E3%2583%25B3%25E3%2582%25B7%25E3%2583%25A7%25E3%2583%2583%25E3%2583%2588+2016-11-20+2.22.22.png "スクリーンショット 2016-11-20 2.22.22.png")

--
　
二つの点を補間していくと、""意味的に""（つまり見た目上）、連続的に変化している。つまり、データを暗記しているわけではない。
　
--


#### Visualizerの視覚化

CNNにも、各層が特定のパターンにマッチするようになるという話があった。

![CNN](http://cs231n.github.io/assets/cnnvis/filt1.jpeg "cnn.png")

--

DCGANにもそれが言えて、Bedroomの画像データセットを使って学習させたところ、特定のパターンにマッチするようなフィルターができた。
![visualizer](https://lh3.googleusercontent.com/-7_bKg0h44lM/WDOoKowPkRI/AAAAAAAAFjw/6nbjv_ajmZQe6W4voqU82hn7r8IqM6vCACLcB/s0/%25E3%2582%25B9%25E3%2582%25AF%25E3%2583%25AA%25E3%2583%25BC%25E3%2583%25B3%25E3%2582%25B7%25E3%2583%25A7%25E3%2583%2583%25E3%2583%2588+2016-11-22+11.06.11.png "visualizer.png")

--


#### ものを描くのを忘れさせる

何らかの表現により主要なBedroomのモノを認識しているはず。窓をわすれさせてみよう。


ロジスティック回帰を使って、２番目に大きな畳み込み層が、窓を表している時には正、関係のないランダムな値を表している時は負の値になるように学習して、ゼロよりかけ離れている値を落とすことで、"窓を忘れさせる"実験をした。

--

![enter image description here](https://lh3.googleusercontent.com/-bnkpBcqLZhE/WDOrzgx-jwI/AAAAAAAAFj8/4UU6aQwgaNAqN5JQ1MioAbM31sR1taqNgCLcB/s0/%25E3%2582%25B9%25E3%2582%25AF%25E3%2583%25AA%25E3%2583%25BC%25E3%2583%25B3%25E3%2582%25B7%25E3%2583%25A7%25E3%2583%2583%25E3%2583%2588+2016-11-22+11.20.32.png "window.png")

実際別のものに置き換わったり、壁に置き換わったりしている。

--

####  顔の算術的表現

Word2Vecでは、 "King" - "Man" + "Woman"を表すベクトル表現のうち、もっとも近いものが"Queen"になるという話があった。
これを画像に適用すると、意味論的に算術計算のようなことができた。

--

![Arithm1](https://lh3.googleusercontent.com/-itf5qVm14tQ/WDOuaAciLCI/AAAAAAAAFkM/adxTuVEX1gUfzWsb7JGm3J151tmDMDOeACLcB/s0/%25E3%2582%25B9%25E3%2582%25AF%25E3%2583%25AA%25E3%2583%25BC%25E3%2583%25B3%25E3%2582%25B7%25E3%2583%25A7%25E3%2583%2583%25E3%2583%2588+2016-11-22+11.31.25.png "arithm1.png")

--

![arithm2](https://lh3.googleusercontent.com/-vpf3RPOTQyQ/WDOuhg4aXPI/AAAAAAAAFkU/Wa1FxEllGD4ykSGx6xQdhfWTWcRtnYN5wCLcB/s0/%25E3%2582%25B9%25E3%2582%25AF%25E3%2583%25AA%25E3%2583%25BC%25E3%2583%25B3%25E3%2582%25B7%25E3%2583%25A7%25E3%2583%2583%25E3%2583%2588+2016-11-22+11.31.30.png "Arithm2.png")

--


![enter image description here](https://lh3.googleusercontent.com/-83bAJ-hefWk/WDOuoTTz6OI/AAAAAAAAFkc/AAEsh3l02e0vhnEslv6rncjIgBrv1DODACLcB/s0/%25E3%2582%25B9%25E3%2582%25AF%25E3%2583%25AA%25E3%2583%25BC%25E3%2583%25B3%25E3%2582%25B7%25E3%2583%25A7%25E3%2583%2583%25E3%2583%2588+2016-11-22+11.31.35.png "Arithm3.png")

--


さらに、人間の顔の姿勢についても、左を向いている人と右を向いている人の補間をとると、少しずつ回転した。

![turn](https://lh3.googleusercontent.com/-aV86CpbGUts/WDOvUvNRyPI/AAAAAAAAFks/3Zc_yHw5FN4822PJm9AdN_0X806__gjHQCLcB/s0/%25E3%2582%25B9%25E3%2582%25AF%25E3%2583%25AA%25E3%2583%25BC%25E3%2583%25B3%25E3%2582%25B7%25E3%2583%25A7%25E3%2583%2583%25E3%2583%2588+2016-11-22+11.36.47.png "turn.png")

---

## Conclusion And Future Work

--

* まだまだ、発散の問題の解決は必要。
* 映像とか音声にこれらの概念を用いたら面白そう？


---

## 言い訳

また実装ができていない・・・・・・・・（駒場祭がなかったらきっとできていたんだって言っている）。

--

## 好きな実装

[chainer-DCGAN](http://mattya.github.io/chainer-DCGAN/)

![cute girl](https://lh3.googleusercontent.com/-UhdSrLpsg3c/WDQXG0H0pHI/AAAAAAAAFlw/Shz8uaIYjzcfjwMViwtSCGSXpEZcKk95wCLcB/s0/cutegirl.png "cutegirl.png")



