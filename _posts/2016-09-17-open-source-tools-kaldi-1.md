--- 
layout: post
title: 有趣的开源软件：语音识别工具Kaldi (一)
---

Kaldi是一个非常强大的语音识别工具库，主要由Daniel Povey开发和维护。目前支持GMM-HMM、SGMM-HMM、DNN-HMM等多种语音识别的模型的训练和预测。其中DNN-HMM中的神经网络还可以由配置文件自定义，DNN、CNN、TDNN、LSTM以及Bidirectional-LSTM等神经网络结构均可支持。

目前在Github上这个项目依旧非常活跃，可以在
[https://github.com/kaldi-asr/kaldi](https://github.com/kaldi-asr/kaldi)
下载代码，以及在
[http://kaldi-asr.org/](http://kaldi-asr.org/)
查看它的文档。

## 下载以及安装

与其他开源软件一样，首先Clone它在Github上的代码

    $ git clone https://github.com/kaldi-asr/kaldi

Clone下来之后按照INSTALL文件的指示，需要先完成tools文件夹下的编译安装，然后再去编译src下的内容。因此，先去tools文件夹：

    $ cd kaldi/tools

在tools文件夹下依旧有一个INSTALL，我们根据它的指示，一步一步完成安装。首先，需要运行extras/check_dependencies.sh这个脚本来检查一些依赖的环境是否存在并且正确配置。

    $  extras/check_dependencies.sh
    extras/check_dependencies.sh: automake is not installed.
    extras/check_dependencies.sh: autoconf is not installed.
    extras/check_dependencies.sh: neither libtoolize nor glibtoolize is installed
    extras/check_dependencies.sh: subversion is not installed
    extras/check_dependencies.sh: we recommend that you run (our best guess):
      sudo apt-get install automake autoconf libtool subversion
    You should probably do: 
      sudo apt-get install libatlas3-base
    /bin/sh is linked to dash, and currently some of the scripts will not run properly.  We recommend to run:
      sudo ln -s -f bash /bin/sh

这个输出的结果不同的Linux会不相同(我的是Ubuntu 16.04)。根据check_dependencies.sh输出结果的提示，安装缺的包，以及配置正确的环境

    $ sudo apt-get install automake autoconf libtool subversion
    $ sudo apt-get install libatlas3-base
    $ sudo ln -s -f bash /bin/sh

然后再重新运行一遍check_dependencies.sh

    $ extras/check_dependencies.sh
    extras/check_dependencies.sh: all OK.

如果输出以上结果，那么我们可以继续安装了

    $ make -j 16

其中-j 16是开16个job同时进行编译，这个可以根据CPU内核的数量进行指定。确定没有错误后切换到src文件夹

    $ cd ../src
    
这里面也包含了一个INSTALL文件，按照里面的步骤编译和安装

    $ ./configure

运行完成后在最后一行可以看到SUCCESS，如果没有的话那应该是哪个步骤出问题了，可以去检查一下上面几个步骤是否有错误

    $ make depend
    $ make -j 16
    
检查一下编译是否有错误，如果没有错误的话make脚本会在屏幕的最后一行输出Done。至此Kaldi的编译安装完成了，可以愉快的开始训练模型了:P
