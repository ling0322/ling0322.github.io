--- 
layout: post
title: 有趣的开源软件：语音识别工具Kaldi (二) 
---

在上一篇blog中简单的介绍了Kaldi的安装方法
[有趣的开源软件：语音识别工具Kaldi (一)](http://ling0322.info/2016/09/17/open-source-tools-kaldi-1.html)
在这篇blog中继续Kaldi模型训练的步骤，介绍一下在模型训练之前的一些数据准备的工作。因为我也是正在学习语音识别和Kaldi，有些地方不一定说的很正确，如果发现错误，还请指正。

在Kaldi源代码树中，有一个叫做egs的文件夹，在这个文件夹中保存着一些Kaldi在公共数据集上的训练步骤（shell脚本）以及测试的结果。其中，中文的语音识别公共数据集一共有三个，分别是

- gale_mandarin: 中文新闻广播数据集(LDC2013S08, LDC2013S08)
- hkust: 中文电话数据集(LDC2005S15, LDC2005T32)
- thchs30: 清华大学30小时的数据集，可以在[http://www.openslr.org/18/](http://www.openslr.org/18/)下载

在这blog中使用的是hkust数据集进行实验。

## 目录结构

hkust数据集相关的脚本以及实验结果位于kaldi/egs/hkust，它的目录结构如下

	.
	├── README.txt
	└── s5
	    ├── cmd.sh
	    ├── conf
	    │   ├── cmu2pinyin
	    │   ├── decode.config
	    │   ├── fbank.conf
	    │   ├── mfcc.conf
	    │   ├── pinyin2cmu
	    │   └── pinyin_initial
	    ├── local
	    │   ├── create_oov_char_lexicon.pl
	    │   ├── ext
	    │   │   ├── 195k_chinese_word2char_map
	    │   │   ├── hkust_word2ch_tran.pl
	    │   │   ├── score_basic_ext.sh
	    │   │   └── score.sh
	    │   ├── hkust_data_prep.sh
	    │   ├── hkust_extract_subdict.pl
	    │   ├── hkust_format_data.sh
	    │   ├── hkust_normalize.pl
	    │   ├── hkust_prepare_dict.sh
	    │   ├── hkust_segment.py
	    │   ├── hkust_train_lms.sh
	    │   ├── nnet
	    │   │   ├── run_cnn.sh
	    │   │   ├── run_dnn.sh
	    │   │   └── run_lstm.sh
	    │   ├── nnet2
	    │   │   ├── run_5d.sh
	    │   │   └── run_convnet.sh
	    │   ├── nnet3
	    │   │   ├── run_ivector_common.sh
	    │   │   ├── run_lstm.sh
	    │   │   └── run_tdnn.sh
	    │   ├── score_basic.sh
	    │   ├── score_sclite_conf.sh
	    │   ├── score_sclite.sh
	    │   └── score.sh
	    ├── path.sh
	    ├── RESULTS
	    ├── run.sh
	    ├── steps -> ../../wsj/s5/steps
	    └── utils -> ../../wsj/s5/utils

其中README.txt是对于这个数据集的一些说明性的东西，对于这个数据集不熟悉的话可以去看一下。

s5/run.sh包含了在这个数据集上所有的训练步骤，包括数据预处理、训练以及测试gmm/dnn/lstm/blstm/tdnn等模型、实验结果统计在内的各种脚本。理论上来说只要相关环境配置正确，运行run.sh就能完成整个训练步骤。但是Kaldi的官方文档还是建议能够将这个文件里面的脚本一步一步粘贴到shell里面运行。这样既能够容易的发现错误，又可以对Kaldi整个运行的步骤有所了解。

s5/RESULTS里面保存着最近的实验结果。这边稍简单贴几条(CER也就是character error rate)：

- mono0a(mono-phone的GMM-HMM): %CER 80.89
- tri1(最简单的tri-phone GMM-HMM): %CER 60.01
- tri5a_mmi_b0.1(用MMI损失函数): %CER 43.95
- dnn5b_pretrain-dbn_dnn: %CER 39.42
- cnn5c_pretrain-dbn_dnn: %CER 38.80
- tdnn_sp: %CER 33.79
- lstm_sp_ld5: %CER 33.51

s5/conf就是一些训练所要用到的配置文件。

s5/{local, steps, utils}里面则是run.sh所要用到的一些脚本文件。

## 数据处理

在run.sh最开始的部分主要是一些数据的预处理步骤， 在运行它之前首先把hkust的数据放在某个固定的地方(在STEP1中会用到)。然后切换到run.sh所在的路径，在我的计算机上，它位于~/Documents/kaldi/egs/hkust/s5

    $ pwd
    /home/ling0322/Documents/kaldi/egs/hkust/s5

因为这个实验中是单机跑，所以需要运行cmd.sh中的几条命令，并且把queue.pl修改成run.pl

    $ export train_cmd="run.pl --mem 8G"
    $ export decode_cmd="run.pl --mem 8G"
    $ export mkgraph_cmd="run.pl --mem 12G"

把这些环境变量export出去，接着可以开始一步一步执行run.sh中的脚本了

### STEP 1 

    $ local/hkust_data_prep.sh /home/ling0322/Documents/hkust-data/LDC2005S15 /home/ling0322/Documents/hkust-data/LDC2005T32

这一步主要做的是将和hkust的数据复制到data文件夹下，以及一些数据格式的转换工作，期间还会使用到mmseg对文本做简单的分词。

### STEP 2

下一步工作是生成音素词典，音素词典记录着每一个词发音所包含的音节序列。比如中文词语的音素就可以是声母韵母拆开的序列，依据这样的规则像“测试”的音素序列就可以是“c e4 sh i4”。

不过要成功运行这段脚本，还需要安装一些它依赖的环境

    $ sudo apt install gawk swig python-numpy python-dev
    $ local/hkust_prepare_dict.sh

在输出中简单检查一下输出，特别是检查以下两行的第一列数字，如果是0的话就是上面某一步骤出错了

    10894 data/local/dict/lexicon-ch/words-ch-oov.txt
    19467 data/local/dict/lexicon-ch/lexicon-ch-iv.txt

### STEP 3

接下去是准备tri-phone模型的决策树question集合以及编译Transducer L。Transducer L用于将音素序列映射成词语序列。关于各个Transducer有什么用可以去参考[Some Kaldi Notes](http://jrmeyer.github.io/kaldi/2016/02/01/Kaldi-notes.html)。要具体弄清楚Transducer到底是什么，还是需要去看这篇paper：Speech Recognition with Weighted Finite-State Transducers

    $ utils/prepare_lang.sh data/local/dict "<UNK>" data/local/lang data/lang
    
接着是去训练3-gram的语言模型，由这个语言模型生成Transducer G，然后将L和G这两个Transducer拼接起来，产生Transducer LG。Transducer LG可以用来对给定的输入的音素序列，使用3-gram语言模型（G）找出最有可能的词语序列。

    $ local/hkust_train_lms.sh
    $ local/hkust_format_data.sh

看到

    Done training LM of type 3gram-mincount

以及

    hkust_format_data succeeded
    
这一步就成功了

### STEP 4

现在可以开始从声音数据中抽取MFCC特征。MFCC中文简称为梅尔频率倒谱系数，是一种尽可能接近人类听觉系统而抽出的一种声音特征的表示方法。这个略微偏向信号处理，具体过程可以去参考语音信号处理相关的书。

在执行下面步骤之前，最好再次检查确认一下$train_cmd是否设置正确

    $ echo "$train_cmd"
    run.pl --mem 8G

然后运行

    $ mfccdir=mfcc
    $ for x in train dev; do 
    $   steps/make_mfcc.sh --cmd "$train_cmd" --nj 10 data/$x exp/make_mfcc/$x $mfccdir || exit 1;
    $   steps/compute_cmvn_stats.sh data/$x exp/make_mfcc/$x $mfccdir || exit 1;
    $ done
    
### STEP 5

最后一步是清理训练数据，移除一些没用的片段

    $ utils/fix_data_dir.sh data/train
    fix_data_dir.sh: kept 197387 utterances out of 197391
    fix_data_dir.sh: old files are kept in data/train/.backup
    
至此，数据准备以及预处理阶段就成功完成了，接着就可以开始训练模型阶段了。