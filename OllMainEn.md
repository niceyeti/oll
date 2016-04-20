OLL: Online Learning Library

# Introduction #

**OLL** is a library for online-learning algorithms, which is specialized for large-scale, but sparse, learning tasks such as Natural Language Processing tasks. While these algorithms are very efficient in terms of speed, and space (linear in the number of examples, and features), its performances are comparable to the batch-style learning methods such as SVMs, MEs.

OLL provides C++ library, and stand-alone programs for learning, predicting.

Currently, OLL supports following algorithms:

Perceptron
Averaged Perceptron
Passive Agressive (PA, PA-I, PA-II)
ALMA (modified slightly from original)
Confidence Weighted Linear-Classification.

# News #

  * 2008 12/24 0.02 Release
> Bug fix, and shuffling support
  * 2008 12/05 0.01 Release

# Download #
> [Please download the latest version from here.](http://code.google.com/p/oll/downloads/list)

# Install #

Get the latest source tarball (oll-**.tar.gz,** is a version number), type the followings. Note that you can change the install directory by replacing "./confiugre" with "./configure" --prefix=dir
```
>tar xzvf oll-*.tar.gz
>cd oll-*
>./configure
>make
>sudo make install
```

# Usage #

Programs: oll\_train oll\_test oll\_line

## Data Format ##

The data format for trainfile, and testfile are same as that for libsvm, tinySVM. Each line containes one example; the label followed by the fired featureID:value pairs. These featureIDs are not necessarily sorted.

(BNF-like representation)
```
<class> .=. +1 | -1 
<feature> .=. integer (>=1)
<value> .=. real
<line> .=. <class> <feature>:<value><feature>:<value> ... <feature>:<value>
```

Example
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
oll\_train read training examples from _trainfile_ and then save the trained model to _modelfile_. If _devfile_ is specified, the program apply the model to _devfile_ and print the accuracy.

The second argument denotes the training method. For exapmle, "P" means that Perceptron is used as a training method. Other training methods are shown in above. I recommend PA1 and CW.

_-C_ specifies the ratio of the ragularization term. You many need to choose the optimal paramter by the result of the accuracy of _dev\_file_ (The default is C=1.0).

_-b_ specifies the bias. This add the virtual feature to each example with the specied value. The default is b=1.0.

_-I_ specifies the number of iteration at training. In general, -I=10, 20 is enough, but CW can acheive the good performance with only one iteration. _-I=0_ is special: the program sees each example and train the model on the fly, without storing examples.

_-s_ specifies the use of training examples shuffling. When _-s=1_, the shuffling is enabled, and _-s=0_, the shuffling is disabled. Since the online training strongly depends on the order of training examples, it is better to use shuffling.

## oll\_test ##
```
oll_test testfile modelfile [-v]
```
Read test data from _testfile_, the trained model from _modelfile_, and print the result of the accuracy to stdout.

If you specify _-v_, the results of each example (score) will be printed to stdout.



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
_oll\_line_ supports interactive Train, Predict, Save, and Load. The options are same as _oll\_train_ (-I is not supported, _oll\_line_ always use -I=0). Read the operation from stdin. The format is as follows;

```
   # (1|-1) id:val id:val ...
```
# is either A or T.
A performs train. The follow-on format is same as the that for the file in _oll\_train
T performs predict, and print the result to stdout. The follow-on format is same as the that for the file in_oll\_train_. (The label will be ignored. Please use any value here)._

```
   # filename
```
S performs save the model to _modelfile_. Note: The number of space between S and modelfile should be 1.
L performs load the model from _modelfile_.

## Example ##
```
  >./oll_train P train1 model1 -C=2.0 -b=1.0 -I=10
　Read Done.
　..........FINISH!
　>./oll_test test1 model1
  Accuracy 93.355% (4664/4996)
  (Answer, Predict): (p,p):2309 (p,n):185 (n,p):147 (n,n):2355
```

Read training examples from _train1_, train the model using Perceptron algorihm, and save the trained model to _model1_. The regularization paratemr is 2.0, the bias is 1.0, and the number of iteration is 10.

Read test examples from _test1_, predict the labels of them, and print the accuracy to stdout: Accuracy (Corrected/All) (p=positive, n=negative).
```
　>./oll_train CW dat/xaa model dat/xab -I=0
  FINISH!
  Accuracy 96.437% (4818/4996)
  (Answer, Predict): (p,p):2422 (p,n):72 (n,p):106 (n,n):2396
```
Read training examples from _train2_ ,train the model using Confidence Weighted Linear-Classification, and save the trained model to _mode2_. It also print the accuracy of the devfile using the model.
```
　>./oll_line PA1
  A 1 0:1.0 1:2.0 2:-1.0   (Input)
  A -1 0:-0.5 1:1.0 2:-0.5 (Input)
  T 0 0:1.0 1:1.0　　　　　  (Input)
  0.171429　　　　　　　　　 (Output)
　S model1                 (Input)
  >./oll_line PA1
  L model1
  T 0 0:1.0 1:1.0　　　　　(Input)
  0.171429　　　　　　　　　(output)
```

## Experiments ##

Data news20.binary　[libsvm data link](http://www.csie.ntu.edu.tw/~cjlin/libsvmtools/datasets/binary.html#news20.binary)
  * # of classes: 2
  * # of data: 19,996
  * # of features: 1,355,191
Shuffled, and splited the data into 15000 training examples and 4996 test examples. Used the default C value (C=1.0)

In the column of SVM (linear), and SVM (2nd polynomial Kernel), I used [tinySVM](http://chasen.org/~taku/software/TinySVM/) for
SVM optimization (used default parameters). I didn't limit the number of iterations for SVM.

-I=10
| Method      | P   | AP    | PA   | PA1  | PA2 | CW | SVM (linear) |
|:------------|:----|:------|:-----|:-----|:----|:---|:-------------|
| Training Time (s) | 0.54 | 0.56  | 0.58 | 0.59 | 0.60 |  1.39 | 1122.60      |
| Accuracy (%) | 94.696 | 95.336 | 96.477 | 96.457 | 96.457 | 96.497 | 96.237       |

-I=1
| Method      | P    | AP   | PA   | PA1  | PA2  | CW |
|:------------|:-----|:-----|:-----|:-----|:-----|:---|
| Training Time (s) | 0.05 | 0.09 | 0.07 | 0.08 | 0.08 | 0.21 |
| Accuracy (%) | 93.355 | 94.035 | 96.157 | 96.137 | 95.917 |  96.437 |

I know that I still need to check results of other implementations/methods. I will continue the experiments, but I welcome the comments about the results with other methods/implementations.

## Reference ##
  * Perceptron
> > Rosenblatt, "The Perceptron: A Probabilistic Model for Information Storage and Organization in the Brain", Cornell Aeronautical Laboratory, Psychological Review, v65, No. 6, pp. 386-408, 1958.
> > Y. Freund, and R. E. Schapire, "Large margin classification using the perceptron algorithm". COLT 1998
  * Averaged Perceptron
> > M. Collins, "Discriminative Training Methods for Hidden Markov Models: Theory and Experiments with Perceptron Algorithms", EMNLP 2002.
  * Passive Agressive
> > K. Crammer, O. Dekel. J. Keshet, S. Shalev-Shwartz, Y. Singer, "Online Passive-Aggressive Algorithms.", Journal of Machine Learning Research, 2006.
  * Confidence Weighted
> > M. Dredze, K. Crammer, F. Pereira. "Confidence-Weighted Linear Classification", ICML 2008



