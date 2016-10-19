---
title: 跨语言情感分析
date: 2016-10-19 10:46:20
categories: 语义计算               
tags: 情感分析
---

##### 1. 基本任务和方法
任务：NLPCC2013跨语言情感分类（英文->中文）
环境：Ubuntu 服务器（4 Intel(R) Xeon(R)CPU E5-2609 v3  @1.90GHz），Anacanda2.4（64-bit），python2.7
基本方法：（1）中文SVM分类；（2）英文SVM分类；（3）英文SVM自学习
评价标准：准确率(Accuracy)
Accuracy = #system_correct / #system_total
其中#system_correct表示分类准确的评论数，#system_total表示测试集内全部评论数。

##### 2. 数据预处理
主要包括XML原文件解析，语料翻译，去除特殊符号。
（1）  解析XML
所需要的模块：
``` python
from xml.etree import ElementTree as ET  #引入解析xml文件的模块  
import re  
import gTrans  
```
详细代码：googleTranslate.extractXMLEnglishText()
（2）  调用google translator API，进行翻译
训练数据英文翻译成中文，测试数据中文翻译成英文，翻译完成后，每篇评论都有中英两种表示。
所需要的模块：
``` python
import re  
import string  
import urllib,urllib2  #引入接入网络接口API的模块  
import gTrans 
```
详细代码：googleTranslate.translate(text, f, t)
参数说明：text为输入文本，f为原语言，t为目标语言
（3）  去除标点符号
语料中有很多符号，影响分类效果，需要单独去除。
详细代码：googleTranslate.delPunctuationStr(str)
##### 3. 模型训练
``` python
def subClassifier(language)：  
 	#合并同语言的训练数据，提取训练数据，代码省略  
 	#classifier() 训练模型  
	classifier(train_data, train_target,test_data, test_target, language)  
```
调用模块：调用Python的机器学习模块sklearn，使用pipline简化过程。
算法：朴素贝叶斯，SVM，随机森林。
（1）  英文分类器
训练数据：已标注的英文语料 + 少量已标注的中文语料翻译成英文标注语料
测试数据：未标注的中文测试语料翻译成英文语料
参数设置：
以SVM为例：
``` python
text_clf_svm = Pipeline([('vect', CountVectorizer(stop_words ='english')),  
     ('tfidf', TfidfTransformer()),  
     ('clf', svm.SVC( C=1,kernel = 'linear')),  
     ])
```
（2）  中文分类器
训练数据：已标注的英文语料翻译成中文标注语料 + 少量已标注的中文语料
测试数据：未标注的中文测试语料
参数设置：同（1）
（3）自学习分类器
利用已有的英文标注数据D0训练一个弱分类器M0，然后将中文未标注数据翻译成英文D1，弱分类器预测D1，将D1中得分最高的topK条数据（K条正向评论，K条负向评论）加入到D0 中，构成新的训练集，同时D1中删除对应的topK条评论。
不断迭代上面的过程，直到准确率不再上升终止（考虑到时间问题，只迭代了50次）。

##### 4. 结果对比与分析
4.1 实验结果

```
	algorithm\data | book | dvd | music
	:----------- | :-----------: | -----------:
	NB(EN) | 0.698 | 0.72375 | 0.70625
	SVM(EN) | 0.76 | 0.77375 | 0.76325
	RandomForest(EN) | 0.68275 | 0.6995 | 0.6835
	NB(CN)| 0.67075 | 0.70225 | 0.69225
	SVM(CN) | 0.72475 | 0.7355 | 0.72475
	RandomForest(CN) | 0.6598 | 0.68022 | 0.67023
	RandomForest(EN) | 0.68275 | 0.6995 | 0.6835
	selfLearningSVM(EN) | 0.808 | 0.801 | 0.78925
```

4.2 结果分析
1） SVM比NB，RandForest结果都高，SVM(EN)的测试准确率在76~77%之间，。
2） 英文分类器比中文分类器准确率高，因为英文分类器的训练集是已标注的数据，中文分类器的训练数据是从英文标注数据翻译过来的，机器翻译的准确性影响最后的效果。
3） 自学习过程不断通过不断增大训练集，使得分类器的效果更准确。
4） 分类器训练运行时间供参考，机器配置对实验时间有很大的影响。
 
4.3 总结
本文尝试了多种跨语言情感分类的方法，主要以SVM为主。由于训练有监督的分类器需要大量的已标注数据（本任务只提供12000条评论），所以采用了利用已有数据训练弱分类器，然后通过自学习过程，不断迭代增加训练数据的方法，得到更准确的分类器。实验证明这一方法确实有效，需要注意的是，考虑到自学习过程存在一定的误差，本文只在12000条未标注评论（小于已有训练数据）选择得分最高的topK加入训练集中。可做的改进是将中英文子分类的结果结合起来，即co-training过程，提高准确性。
另外，神经网络也可以用来做情感分类的过程，由于时间限制，神经网络的实验在报告里省略了。
##### 5. 问题及解决办法
（1）解析语料时出现：xml.etree.Element.ParseError
原因：有一个text里没有文本（只有评论id），需要提前判断做相应的处理
（2）翻译限制：
尝试了几种google翻译小工具，如goslate，报错HTTPError: HTTP Error 503: Service Unavailable。在官网查到原因：Google has updated its translation service recently with a ticketmechanism to prevent simple crawler program like goslate from accessing。
python 调用google translate API进行翻译的内容详见另一篇博客：http://blog.csdn.net/lrita/article/details/51298227