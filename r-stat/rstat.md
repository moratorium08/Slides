## 統計課題発表

#### 640726I 桂 宏行

---

## やったこと

* Pythonを用いて、Twitterのデータをスクレイピング
* スクレイプしたデータを処理
* Rを用いて、解析

--

## 取得したデータ

* ユーザーのフォローワーの数
* ユーザーの直近20ツイートの「お気に入り」の数の平均

--

## なぜ、20ツイートか？

--

* Twitter APIの仕様上、一度に取れるツイート数が20ツイートなので
* それ以上も取得可能だが、APIカウントを消費する
* より多くのユーザーを取得したかったので、この数で抑えた

--

## データセット

* 基本的に、僕がフォローしている人
* それに加えて、有名人、スパムユーザーを追加

--

## 予想

* 当然だが、フォローワーが多い方が、「お気に入り」の平均も大きそうである

---

## 解析

* 分布を表示してみる
* 線形回帰をやってみる

---

## 解析結果

--

## 分布

![enter image description here](https://lh3.googleusercontent.com/-cnG-hGqbf34/WHUKbe3mafI/AAAAAAAAJO8/PCVjw94lQbwqv-HQ1n1ofK1uRzDvw851ACLcB/s0/%25E5%2588%2586%25E5%25B8%2583.png "分布.png")

---

## 線形回帰の結果

--

### 推定結果
```
              Estimate Std. Error t value Pr(>|t|)
(Intercept)     -3.8184     0.3293  -11.59   <2e-16 ***
log(followers)   0.5632     0.0439   12.83   <2e-16 ***

Adjusted R-squared:  0.3043
```

--

### フィッティングした図

![enter image description here](https://lh3.googleusercontent.com/-U1jqu3mG1YY/WHUNiNVd_PI/AAAAAAAAJPg/kszdK8wx-ZM_sCwwDr3u2II15_TlDBJBQCLcB/s0/%25E3%2583%2595%25E3%2582%25A9%25E3%2583%25AD%25E3%2583%25BC.png "フォロー.png")

--

* R-squaredが、0.304なので、よくない回帰である
	* もっと良い解析手法があるかもしれない・・・
* しかし、ある種の傾向は見える
	* "一般ユーザー"の軍団は比較的線形回帰の枠の中に入る
	* 有名人としていた人@AbeShinzoは、これらの軍から大きく外れている。
	* スパムユーザーとしていた人@SEKIGAN_KUNなども、顕著に、これらの群から外れている
	* ボット群も軍団から外れている
	* 有益アカウント系(NHK_PR,zishin3256,..)などは、固まって群から離れている

---

### 診断プロット

![enter image description here](https://lh3.googleusercontent.com/-cSfibLEuFHU/WHUPulrda_I/AAAAAAAAJRM/RZ7GW3SwvtkGEHy_Ml9cvLIqL1aUIlsqgCLcB/s0/%25E8%25A8%25BA%25E6%2596%25AD%25E3%2583%2595%25E3%2582%259A%25E3%2583%25AD%25E3%2583%2583%25E3%2583%2588.png "診断プロット.png")

--

* これを見ても、外れ値として、有名人や、スパムユーザーが異常値になっている
* @nannrakanokiや@yaneuraohは、ツイート数が少なく、一つのツイートの粒度が高いようなアカウントで、これも外れ値になっている。

---

### どうでもよいが、興味深いこと

![enter image description here](https://lh3.googleusercontent.com/-9-s0oaY3tl0/WHURsjwlkfI/AAAAAAAAJSA/lZsLYjN-_bU65GDHQzAvvcmeCv5_vbiWwCLcB/s0/%25E3%2583%2595%25E3%2582%25A9%25E3%2583%25AD%25E3%2583%25BC2.png "フォロー2.png")

この部分

---

見にくいが、

* yuyushiki_anime (ゆゆ式アニメ公式アカウント）
* nganime (New Game!アニメ公式アカウント)
* sakuratrick_pr (桜Trickアニメ公式アカウント）

という漫画タイムきらら系列のアニメ公式アカウントが、非常に近い場所にかたまっている。


---

## さいごに

* 他にもfollowings/followersをパラメタにしてみる、followingをパラメタにしてみるなどの試行をしてみたが、あまり良い結果が得られず、さすがに、この事象はもう少し複雑なものだとおもった。

