OLL: オンライン学習ライブラリ

# はじめに #

**OLL** は様々なオンライン学習をサポートした機械学習ライブラリであり、特に自然言語処理など、大規模、かつ疎な学習問題に最適化されています。これらのオンライン学習手法は速度面、作業領域面で非常に効率的（学習サンプル数、素性種類数に比例）でありながら、SVMsやMEsなどのバッチ学習と同程度の精度を達成します。

学習、推定を行なうプログラムとC++ libraryを提供しています。

現在サポートしている学習手法は次の通りです。
  * Perceptron [F. Rosenblatt 1958]
  * Averaged Perceptron [M. Collins 2002]
  * Passive Agressive (PA, PA-I, PA-II) [K. Crammer, et. al. 2006]
  * ALMA (modified slightly from original) [H. Daume, 2007]
  * Confidence Weighted Linear-Classification [M. Dredze, 2008]

# 更新情報 #

  * 2008 12/24 0.02 リリース
> > バグフィックス、シャッフリングのサポート
  * 2008 12/05 0.01 リリース

# ダウンロード #

> [ここの最新版をダウンロードしてください](http://code.google.com/p/oll/downloads/list)

# インストール方法 #

最新のソースコード(oll-**.tar.gz,**はversion番号）をダウンロードしてきたら、次のようにしてインストールしてください。./configureの際に、./configure --prefix=ディレクトリとすることで、インストールするディレクトリを指定できます。
```
>tar xzvf oll-*.tar.gz
>cd oll-*
>./configure
>make
>sudo make install
```

# 使い方 #

以下ではプログラム oll\_train oll\_test oll\_lineの使い方を説明します。
C++ライブラリの使い方については少々お待ちください（興味がある人はoll\_train.cppなどを参考にしてください）。

## データフォーマット ##

訓練時、テスト時に利用する訓練ファイル、テストファイルのデータフォーマットはlibsvm, tinySVMなどと同様のフォーマットです。各行が一訓練例になっており、最初に正例なら+1, 負例なら-1, その後に、発火した素性番号:値の列を並べます。素性番号はソートされていなくても構いません。

(BNF-like representation)
```
<class> .=. +1 | -1 
<feature> .=. integer (>=1)
<value> .=. real
<line> .=. <class> <feature>:<value><feature>:<value> ... <feature>:<value>
```

例
```
+1 0:1.0 201:2.2 744:-0.3 15:3.0
-1 47:2.0 66:0.1 733:1.0 500:1.0
+1 3:1.0 201:2.2 300:-0.3 15:0.35
+1 4:2.0 103:0.4 405:-0.3 11:1.5
```

## oll\_train ##
```
oll_train (P|AP|PA|PA1|PA2|CW) trainfile modelfile [devfile] -C=FLOAT -b=FLOAT -I=INT -s=BOOL
P   : Perceptron
AP  : Averaged Perceptron
PA  : Passive Agressive
PA1 : Passive Agressive I
PA2 : Passive Agressive II
PAK : Kernelized Passive Agressive
CW  : Confidence Weighted
AL  : ALMA HD (heavy)
-C    Regularization Paramter (Default 1.0)
-b    Bias (Default 0.0)
-I    Iteration (Default 10, 0 = one pass without storing)
-s    1: Enable Shuffle, 0: Disable Shuffle (Default 1)
```
oll\_trainはtrainfileに書かれた訓練ファイルを読み込み、学習結果をmodelfileに書き込みます。devfileが指定されている場合、学習結果をdevfileで試し、精度を表示します。

第2引数では学習手法を指定します。例えば"P"を指定した場合はperceptronで学習します。それぞれの学習手法は上の通りです。お勧めはPA1とCWです。

-Cで正則化項で学習の割合を指定します。これはdevfileでの精度で調整することが必要です（指定しない場合は1.0になります。）。

-bはバイアス項です。各訓練事例に指定した値を持つ仮想的な素性を追加します。指定しない場合は0.0になります。

-I は学習データを何回繰り返し見るかを指定します。大抵は10～20ですが、CWだけは1回だけでも大体良い精度に達成します。-I=0が特別で、データを順に1度だけ見て学習します（繰り返し回数は1)が、訓練データは保存しないため効率的です。

-s は訓練データをシャッフルするかどうかを指定します。1の時はシャッフルし、0の時はシャッフルしません。オンライン学習では、訓練データを見る順番が非常に影響するので、シャッフルした方が良いです。-I=0の時は-sの値は無視されます。

## oll\_test ##
```
oll_test testfile modelfile [-v]
```
ファイルtestfileに書かれたテストデータを読み込み、modelfileから学習結果を読み込み、精度の結果を表示します。

-vを指定すると、各訓練データに対する推定結果（スコア）を１行ずつ表示します。



## oll\_line ##
```
   oll_line (P|AP|PA|PA1|PA2|CW) -C=FLOAT -b=FLOAT -I=INT
   P   : Perceptron
   AP  : Averaged Perceptron
   PA  : Passive Agressive
   PA1 : Passive Agressive I
   PA2 : Passive Agressive II
   PAK : Kernelized Passive Agressive
   CW  : Confidence Weighted
   AL  : ALMA HD (heavy)
   -C    Regularization Paramter (Default 1.0)
   -b    Bias (Default 0.0)

   Standard Input Format
   A (1|-1) id:val id:val ... (Add training example)
   T (1|-1) id:val id:val ... (Test Example)
   S modelfile (Save current model to modelfile)
   L modelfile (Load current model to modelfile)
```
oll\_lineではインタラクティブに訓練、推定、保存、読み込みを行います。起動時のオプションはoll\_trainと同じです（-Iは繰り返しみないのでありません。-I=0と同じです）
標準入力から１行ずつ命令を送ります。命令フォーマットは以下の通りです。
```
   # (1|-1) id:val id:val ... (Add training example)
```
#はA,T,S,Lのいずれかです。
Aは学習を行います。その後に続くフォーマットはoll\_train, oll\_testで利用するデータフォーマットと同様です。
Tは推定を行い推定結果を標準出力に出します。その後に続くフォーマットはoll\_train, oll\_testで利用するデータフォーマットと同様です（ラベル部分は無視されます。適当に1,-1を入れておいてください）。
Sは学習結果をmodelfileに保存します。注：Sとmodelfileの間のスペースは必ず1つにしてください。
Lは学習結果をmodelfileから読み込みます。

## 使用例 ##
```
  >./oll_train P train1 model1 -C=2.0 -b=1.0 -I=10
　Read Done.
　..........FINISH!
　>./oll_test test1 model1
  Accuracy 93.355% (4664/4996)
  (Answer, Predict): (p,p):2309 (p,n):185 (n,p):147 (n,n):2355
```
ファイルtrain1に書かれた訓練データを読み込みパーセプトロンで学習し、学習結果をファイルmodelに保存します。正則化パラメータは2.0, バイアス項は1.0、繰り返し回数は10です。次に学習結果を読み込み、test1に対して推定を行い、精度（正解数/問題数）と（正解、予想）の組み合わせ（p=正例：ラベルが+1だったもの、n=負例：ラベルが-1だったもの）を表示します。
```
　>./oll_train CW dat/xaa model dat/xab -I=0
  FINISH!
  Accuracy 96.437% (4818/4996)
  (Answer, Predict): (p,p):2422 (p,n):72 (n,p):106 (n,n):2396
```
ファイルtrain2に書かれた訓練データを読み込みConfidence Weighted Linear-Classificationで学習し、学習結果をmode2に保存します。繰り返し回数は1回ですが、訓練データは保存せず読み込んだ順に学習します。また、学習結果をdev2に対し適用し、精度を表示します。
```
　>./oll_line PA1
  A 1 0:1.0 1:2.0 2:-1.0  （入力）
  A -1 0:-0.5 1:1.0 2:-0.5（入力）
  T 0 0:1.0 1:1.0　　　　　（入力）
  0.171429　　　　　　　　　（出力）
　S model1
  >./oll_line PA1
  L model1
  T 0 0:1.0 1:1.0　　　　　（入力）
  0.171429　　　　　　　　　（出力）
```

## 実験 ##

実験データ news20.binary　[libsvmリンク](http://www.csie.ntu.edu.tw/~cjlin/libsvmtools/datasets/binary.html#news20.binary)
  * # of classes: 2
  * # of data: 19,996
  * # of features: 1,355,191
シャッフルし、15000例の訓練データと4996例のテストデータに分けた。Cのチューニングは特に行なっていない（C=1.0)

SVM(linear), SVM(2nd polynomial Kernel)の訓練には[tinySVM](http://chasen.org/~taku/software/TinySVM/)を利用しました（パラメータはデフォルト）。-I=10の表に含まれていますが、繰り返し回数は制限していません（10回などと制限した場合性能は著しく低下します。

-I=10
| 手法      | P   | AP    | PA   | PA1  | PA2 | CW | SVM (linear) |
|:--------|:----|:------|:-----|:-----|:----|:---|:-------------|
| 学習時間 (s) | 0.54 | 0.56  | 0.58 | 0.59 | 0.60 |  1.39 | 1122.60      |
| 精度 (%)  | 94.696 | 95.336 | 96.477 | 96.457 | 96.457 | 96.497 | 96.237       |

-I=1
| 手法      | P    | AP   | PA   | PA1  | PA2  | CW |
|:--------|:-----|:-----|:-----|:-----|:-----|:---|
| 学習時間 (s) | 0.05 | 0.09 | 0.07 | 0.08 | 0.08 | 0.21 |
| 精度 (%)  | 93.355 | 94.035 | 96.157 | 96.137 | 95.917 |  96.437 |

これらの結果はまだ部分的なものであり、更なるデータ、手法、実装での比較が必要です。私も実験を続けますが、比較した実験結果のコメントなどは大歓迎です。

## 参考文献 ##
  * Perceptron
> > Rosenblatt, "The Perceptron: A Probabilistic Model for Information Storage and Organization in the Brain", Cornell Aeronautical Laboratory, Psychological Review, v65, No. 6, pp. 386-408, 1958.
> > Y. Freund, and R. E. Schapire, "Large margin classification using the perceptron algorithm". COLT 1998
  * Averaged Perceptron
> > M. Collins, "Discriminative Training Methods for Hidden Markov Models: Theory and Experiments with Perceptron Algorithms", EMNLP 2002.
  * Passive Agressive
> > K. Crammer, O. Dekel. J. Keshet, S. Shalev-Shwartz, Y. Singer, "Online Passive-Aggressive Algorithms.", Journal of Machine Learning Research, 2006.
  * Confidence Weighted
> > M. Dredze, K. Crammer, F. Pereira. "Confidence-Weighted Linear Classification", ICML 2008