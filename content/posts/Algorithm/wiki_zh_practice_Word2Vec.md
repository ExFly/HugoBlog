---
title: "使用gensim训练word2vec模型--中文维基百科语料"
author: "Exfly"
cover: "/media/img/Algorithm/wiki_zh_practice_Word2Vec/avatar.jpg"
tags: []
date: 2018-02-14T11:04:36+08:00
---

文章简介：为了写论文，使用gensim训练word2vec模型，如下记录了进行训练的过程

<!--more--> 

# 准备
* 中文维基百科预料：[zhwiki-latest-pages-articles.xml.bz2](https://dumps.wikimedia.org/zhwiki/latest/zhwiki-latest-pages-articles.xml.bz2)
* [python3](#)
* [wiki_zh_word2vec](https://github.com/ExFly/wiki_zh_word2vec.git)
* 繁体转简体：[opencc](https://bintray.com/package/files/byvoid/opencc/OpenCC)一定要下\*-win32.7z,win64的在我电脑上无法运行。如果使用我的[wiki_zh_word2vec](https://github.com/ExFly/wiki_zh_word2vec.git),则项目中包含可以直接使用的opencc

# TODO

## 依赖准备
* 下载[中文维基百科预料](https://dumps.wikimedia.org/zhwiki/latest/zhwiki-latest-pages-articles.xml.bz2)
* git clone https://github.com/ExFly/wiki_zh_word2vec.git
* 将zhwiki-latest-pages-articles.xml.bz2放到build文件夹下
* cd path/to/wiki_zh_word2vec
* pip install pipenv
* pipenv install --dev

## 将XML的Wiki数据转换为text格式
* pipenv run python 1_process.py build/zhwiki-latest-pages-articles.xml.bz2 build/wiki.zh.txt

31分钟运行完成282855篇文章，得到一个931M的txt文件

## 中文繁体替换成简体
* opencc-1.0.1-win32/opencc -i build/wiki.zh.txt -o build/wiki.zh.simp.txt -c opencc-1.0.1-win32/t2s.json

大约使用了15分钟

## 结巴分词
* pipenv run python 2_jieba_participle.py

大约使用了30分钟

## Word2Vec模型训练
* pipenv run python 3_train_word2vec_model.py

大约使用了30分钟，且全程cpu使用率达到90%+

## 模型测试
* pipenv run python 4_model_match.py

```cmd
d:\Project\wiki_zh_word2vec (develop)
λ pipenv run python 4_model_match.py
国际足球 0.5256255865097046
足球队 0.5234458446502686
篮球 0.5108680725097656
足球运动 0.5033905506134033
国家足球队 0.494105726480484
足球比赛 0.4919792115688324
男子篮球 0.48382389545440674
足球联赛 0.4837716817855835
体育 0.4835757911205292
football 0.47945135831832886
```

## 查看结果
可以使用linux的head或者tail命令查看运行的结果。
* head -n 100 wiki.zh.simp.txt > wiki.zh.simp_head_100.txt,直接查看wiki.zh.simp_head_100.txt即可
* 没有head命令，可以安装[gow](https://github.com/bmatzelle/gow/releases)，或者直接下载[cmder](http://cmder.net/),进入就可以使用head命令了

# 结果
* 至此，使用python对中文wiki语料的词向量建模就全部结束了，wiki.zh.text.vector中是每个词对应的词向量，可以在此基础上作文本特征的提取以及分类。所有代码都已上传至[本人GitHub](https://github.com/ExFly/wiki_zh_word2vec)中，欢迎指教！
* 感谢[AimeeLee77](https://github.com/AimeeLee77/wiki_zh_word2vec),其代码为Python2，我的项目[exfly/wiki_zh_word2vec](https://github.com/ExFly/wiki_zh_word2vec.git)已经完全迁移到python3,并向[AimeeLee77](https://github.com/AimeeLee77/wiki_zh_word2vec)提交了pull request
* [wiki_zh_word2vec](https://github.com/ExFly/wiki_zh_word2vec.git)
