--- 
layout: post
title: 有趣的开源软件：OCR工具Tesseract
---

说到开源的OCR工具，最常用的应该就是Tesseract，它支持100多语言的OCR（当然包括中文）。Tesseract的历史可以追溯到1985年，起初是用C语言编写，其后部分改用C++实现。在2005年它由HP买下并且开源，之后从2006年开始起主要由Google开发和维护。

代码可以从 [github.com/tesseract-ocr/tesseract](https://github.com/tesseract-ocr/tesseract) 下载，可以看到的是目前这个项目依旧是非常活跃的。

在Win10 RS1自带的Bash中（对应Ubuntu 14.04发行版）可以直接安装tesseract

    $ sudo apt-get install tesseract-ocr

然后中文的语言包

    $ sudo apt-get install tesseract-ocr-chi-sim

安装完成之后运行起来也很简单

    $ tesseract -l chi_sim ocr_input.jpg ocr_result.txt

大致的效果就是类似于这样的图片（当然用于识别的原图比这个更加清晰）

![OCR Input](/assets/ocr_input.jpg)

识别的结果是

    以 自然语言人机交互为主要目标的自动语音识别 (ASR)， 在近几十年来一直是
    研究的热点o 在2000 年以前， 有众多语音识别相关的核心技术涌现出来， 例如: 混
    合高斯模型 (GMM)、 隐马尔可夫模型 (HMM)、 梅尔倒谱系数 (MFCC) 及其差分、
    刀元词组语言模型 (LM)、 鉴别性训练以及多种自适应技术o 这些技术极大地推进了
    ASR 以及相关领域的发展o 但是比较起来， 在2000 年到 2010 年间， 虽然GMM_HMM
    序列鉴别性训练这种重要的技术被成功应用到实际系统中， 但是在语音识别领域中无
    论是理论研究还是实际应用， 进展都相对缓慢与平淡o

当然Tesseract也可以通过C++ API来调用，它的接口也非常简洁，例如

{% highlight c++ %}
#include <tesseract/baseapi.h>
#include <leptonica/allheaders.h>

int main() {
  char *outText;

  tesseract::TessBaseAPI *api = new tesseract::TessBaseAPI();
  // Initialize tesseract-ocr with Chinese, without specifying tessdata path
  if (api->Init(NULL, "chi_sim")) {
    fprintf(stderr, "Could not initialize tesseract.\n");
    exit(1);
  }

  // Open input image with leptonica library
  Pix *image = pixRead("ocr_input.jpg");
  api->SetImage(image);
  // Get OCR result
  outText = api->GetUTF8Text();
  printf("OCR output:\n%s", outText);

  // Destroy used object and release memory
  api->End();
  delete [] outText;
  pixDestroy(&image);

  return 0;
}
{% endhighlight %}


在编译前首先需要安装tesseract-ocr-dev

    $ sudo apt-get install tesseract-ocr-dev

然后就可以直接编译和运行刚才的代码

    $ g++ -o tesseract-test tesseract-test.cc -llept -ltesseract
    $ ./tesseract-test > ocr_result.txt

ocr_result.txt就是OCR的结果。

和大多数的库一样Tesseract以及它自带的chi-sim模型只是一种通用的解决方案，如果要获得更加好的结果，还是需要自己去训练针对特定任务的模型。关于如何去训练，可以参考[TrainingTesseract](https://github.com/tesseract-ocr/tesseract/wiki/TrainingTesseract)。

