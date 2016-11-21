
# Unsupervised Representation Learning With Deep Convolutional Generative Adversarial Networks

---

## Abstract
* 教師あり学習に対しては、CNNは大きな成果
* 教師なし学習に対しても適用
* GもDも対象をよく階層的に捉えることができた
* 学習データは画像の特徴として適用可能

---

## Introduction
GANはあんまり安定しない。そこで、この論文では
* 多くの場合に安定に収束するようなDC-GANの設計
* Discriminatorを画像分類のタスクに使う
* GANの学習を可視化
* Generatorのvectorは簡単な算術操作に対して有意義な性質を持つ
を書いている

---

## Related Work

--

#### 画像解析の分野における教師なし学習の手法
* Clustering
	* k-meansみたいな。
* Auto Encoder
	* encode して decodeしても元のに戻せるように。よい特徴表現になる
* Deep belief Network
	* 良い。

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
NN自体、ややブラックボックスだが、CNNが畳み込み的な近似を行っていることや、勾配降下は理想的な画像を検査させることをしていることが示されている

---

## Approach And Model Architecture

--

CNNでGANを拡張するのは簡単ではないが、できた。

方針の本質は、<strong>最近発表されたの３つのCNNの手法を適用すること</strong>

* Poolingを StrideのあるConvolutionに置き換える
* 畳み込み層の特徴の上（？）にある全結合層を取り除く
	* Global Average Poolingがこの例（らしい（よくわからない
	* とにかく深いネットワークでは隠れ層で全結合は削除する
* Batch Normalizationを使う
	* 全部に適用するのはよくなく、不安定になるため、GのoutputとDのinputには適用しない。
* GeneratorにReLUを使う。ただしoutputはtanh
*  DiscriminatorにLeakyReLUを使う

---

## Details of Adversarial Training

LSUN, Imagenet-1k, 画像のデータセットで実験

学習については、
* 画像はGeneratorのoutputであるtanhにあうように[-1,1]に正規化
* 128のミニバッチでSGDで学習
* LeakyReLUのleakは0.2に設定
* またOptimizerにAdamを使用。学習率は0.001推奨だが0.0002の方がよかった。またmomentumは0.5がよかった


#### LSUN
過学習してしまい、サンプルを覚えているだけ、という可能性があるが、そうではないらしい（英語何言ってるかわからん・・・。Google翻訳してもわからん・・・つらい）

#### Faces
現代に生まれた人の名前をdbpediaからとってきて、検索し、画像をランダムにとってくる。それに対してOpenCVのface detectorを使って顔を取得する。

#### Image-1k
自然の写真。

---

## Empirical Validation Of DCGANs Capabilities

一般に、教師なし学習の能力を測るために、教師なし学習を特徴抽出にして、これらの特徴から学習してみるという手法がある。これで学習したところ、Exemplar CNNにはおよばないものの、良い精度が出ているとわかる。

![CIFAR10](https://lh3.googleusercontent.com/-qFPouGpmeaA/WDA0_4P8aqI/AAAAAAAAFic/IabHt-2HDLQbh0kG2aWRag0bkSfl1_JbgCLcB/s0/%25E3%2582%25B9%25E3%2582%25AF%25E3%2583%25AA%25E3%2583%25BC%25E3%2583%25B3%25E3%2582%25B7%25E3%2583%25A7%25E3%2583%2583%25E3%2583%2588+2016-11-19+20.13.34.png "table1.png")

また他の解析として、StreetView House Numbers(SVHN)を用いても行った。良さそう。

![enter image description here](https://lh3.googleusercontent.com/-nGoJ9CgKzOg/WDBMH0zRUfI/AAAAAAAAFi4/7ek7ZXByHkIANZ_qITo8xC2w3Jwsuk2FACLcB/s0/%25E3%2582%25B9%25E3%2582%25AF%25E3%2583%25AA%25E3%2583%25BC%25E3%2583%25B3%25E3%2582%25B7%25E3%2583%25A7%25E3%2583%2583%25E3%2583%2588+2016-11-19+21.56.41.png "スクリーンショット 2016-11-19 21.56.41.png")

--


## Investigating And Visualizing The Internals Of The Networks

もしG内にトレーニングデータが保持されているだけだとすると、内部には潜在的に歪んだ形で保持されていることになるはず。もしそうでないとすると、画像は何らかの追加と削除が加えられた"良い表現"になっているはず。
　結果は、
　![result1](https://lh3.googleusercontent.com/-jp0Lffi3nho/WDCKv3BQ1kI/AAAAAAAAFjY/XQPCuzh-9Y4TgYJOqhC0D1FlXrlalxhvQCLcB/s0/%25E3%2582%25B9%25E3%2582%25AF%25E3%2583%25AA%25E3%2583%25BC%25E3%2583%25B3%25E3%2582%25B7%25E3%2583%25A7%25E3%2583%2583%25E3%2583%2588+2016-11-20+2.22.22.png "スクリーンショット 2016-11-20 2.22.22.png")
なんかこれも英語が完全に正確に訳せないが（９つのランダムな点の補間ってどういうこと・・・）、個人的に解釈すれば、ランダムにとった点、を補間していくと、次第に画像が違うものに移っていくので、内部に特定の画像を保持するといった形ではない表現を保持していないとこれを達成するのは、難しい。したがって、Gは単なるデータセットの保存ではない

--



