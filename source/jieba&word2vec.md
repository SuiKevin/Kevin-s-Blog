# Ubuntu/Python 结巴分词 + Word2Vec利用维基百科训练词向量

[结巴分词](https://github.com/fxsjy/jieba)是一个跨语言的中文分词器，整体效果还算不错，功能也够用，这里直接用`Python`了，其他主流语言版本均有提供。

`Word2Vec`，起源于谷歌的一个项目，在我刚开始接触的时候就关注到了他的神奇，大致是通过深度神经网络把词映射到N维空间，处理成向量之后我们终于可以在自然语言处理上方便的使用它进行一些后续处理。（具体的方法忘了）

Python的`gensim`库中有`word2vec`包，我们使用这个就可以了，接下来我们就对维基百科进行处理，作为训练集去训练。（包地址：[http://radimrehurek.com/gensim/models/word2vec.html](http://radimrehurek.com/gensim/models/word2vec.html)）

本文参考：

[中英文维基百科语料上的word2vec实验](http://www.52nlp.cn/%E4%B8%AD%E8%8B%B1%E6%96%87%E7%BB%B4%E5%9F%BA%E7%99%BE%E7%A7%91%E8%AF%AD%E6%96%99%E4%B8%8A%E7%9A%84word2vec%E5%AE%9E%E9%AA%8C)

[Ubuntu/Python 结巴分词 + Word2Vec利用维基百科训练词向量](https://codesky.me/archives/ubuntu-python-jieba-word2vec-wiki-tutol.wind)

## 安装
---
依赖库是Numpy和SciPy，在Scipy的说明里有：
```
pip install numpy scipy matplotlib ipython ipython-notebook pandas sympy nose
```
安装期间如果遇到问题，可以使用：
```
yum install gcc g++
```
这样安装完成之后，继续
```
pip install gensim
```
下载不动，换源再试，依旧安装失败，似乎是在编译时出现了问题，具体查了一下也没有查出什么问题，只是有人说手动安装成功了，那么我就试试手动安装吧。

手动安装就是在pip官网上把对应的包下载下来，然后sudo python setup.py install，结果似乎没什么问题。总算是安装上了。

打开python终端尝试import也能用，换言之我们总算可以用了。

## 处理
---
使用维基百科的数据很方便，一是Wiki给我们提供了现成的语料库（听说是实时更新的），虽然中文体积不大，但比起自己爬来方便了不少。

如果使用英文那就更棒了，非常适合作为语料库。

当然只是说在通用的情况下，在专业词汇上，经过测试效果比较一般（考虑到专业词库有专业wiki，以及中文词条本身也不太多）。

首先，我们把Wiki处理成Text格式待处理的文本，这一步在本文参考中有现成的代码。
``` python
#!/usr/bin/env python
# -*- coding: utf-8 -*-
import logging
import os.path
import sys
from gensim.corpora import WikiCorpus
if __name__=='__main__':
    program = os.path.basename(sys.argv[0])
    logger = logging.getLogger(program)
    logging.basicConfig(format='%(asctime)s: %(levelname)s: %(message)s')
    logging.root.setLevel(logging.INFO)
    console = logging.StreamHandler()
    console.setLevel(logging.INFO)
    logging.getLogger('').addHandler(console)
    logger.info("running %s" % ' '.join(sys.argv))
    #check and process input arguments
    if len(sys.argv) < 3:
        print globals()['__doc__'] % locals()
        sys.exit(1)
    inp, outp = sys.argv[1:3]
    space = " "
    i = 0
    output = open(outp, 'w')
    wiki = WikiCorpus(inp, lemmatize=False, dictionary={})
    for text in wiki.get_texts():
        output.write(space.join(text) + "\n")
        i = i + 1
        if (i % 10000 == 0):
            logger.info("Saved " + str(i) + " articles")
    output.close()
    logger.info("Finished Saved " + str(i) + " articles")
```
logger比print更规范，过去没有用过相关的，不太会用，其实用起来还是蛮方便的，这里暂时就先不介绍了。

Wiki的处理函数在gensim库中有，通过处理我们可以发现，最终效果是变成一行一篇文章并且空格分隔一些关键词，去掉了标点符号。

执行：
```
python process_wiki.py zhwiki-latest-pages-articles.xml.bz2 wiki.zh.text
```
等待处理结果，比较漫长，基本上接下来你可以随便做点什么了。

处理完成之后我们会发现，简体和繁体并不统一，所以我们需要用opencc进行简繁体的转换，这里不得不说BYVoid是个非常牛逼的同学。这个被官方收录了，我们可以直接用
```
yum install opencc
```
来安装，github开源：[https://github.com/BYVoid/OpenCC](https://github.com/BYVoid/OpenCC)

**如果安装不成功，参考我的另一篇文章 [Compile and Install OpenCC on Minimal CentOS 7](http://www.kevinsui.com/posts/opencc_install_installation/)**

然后执行：
```
opencc -i wiki.zh.text -o wiki.zh.text.jian -c zht2zhs.ini
```

得到简体中文的版本，这一步的速度还可以。

## 分词
---

下一步，分词，原文中用的似乎有些复杂，结巴分词的效果其实已经不错了，而且很好用，这里就用结巴分词处理一下。本身而言结巴分词是不去掉标点的，但是由于上一步帮我们去掉了，所以这里我们比较省力（不然的话原本准备遍历去掉，根据词性标注标点为`x`）。

我的Python还是不太6，所以写的代码比较难看OTZ，不过效果是实现了，处理起来比较慢，我觉得`readlines`里的参数可以更多一点。

这里下面处理完了之后用`map`处理，拼接list并且使用`utf-8`编码，此外，保证一行一个文章，空格分隔（这是后续处理函数的规定）。

这里分词没开多线程，不过后来发现瓶颈似乎在读取的IO上。
``` python
#!/usr/bin/env python
#-*- coding:utf-8 -*-
import jieba
import jieba.analyse
import jieba.posseg as pseg
def cut_words(sentence):
    #print sentence
    return " ".join(jieba.cut(sentence)).encode('utf-8') #如果writelines报错，删除.encode('utf-8')
f = open("wiki.zh.text.jian")
target = open("wiki.zh.text.jian.seg", 'a+')
print('open files')
line = f.readlines(100000)
while line:
    curr = []
    for oneline in line:
        #print(oneline)
        curr.append(oneline)
    '''
    seg_list = jieba.cut_for_search(s)
    words = pseg.cut(s)
    for word, flag in words:
        if flag != 'x':
            print(word)
    for x, w in jieba.analyse.extract_tags(s, withWeight=True):
        print('%s %s' % (x, w))
    '''
    after_cut = map(cut_words, curr)
    # print lin,
    #for words in after_cut:
        #print words
    target.writelines(after_cut)
    print('saved 100000 articles')
    line = f.readlines(100000)
f.close()
target.close()
print("end")
```

## 训练
---
最后就能愉快的训练了，训练函数还是参考了原文：
``` python
#!/usr/bin/env python
#-*- coding:utf-8 -*-
import logging
import os.path
import sys
import multiprocessing
from gensim.corpora import WikiCorpus
from gensim.models import Word2Vec
from gensim.models.word2vec import LineSentence
if __name__ == '__main__':
    program = os.path.basename(sys.argv[0])
    logger = logging.getLogger(program)
    logging.basicConfig(format='%(asctime)s: %(levelname)s: %(message)s')
    logging.root.setLevel(level=logging.INFO)
    logger.info("running %s" % ' '.join(sys.argv))
    # check and process input arguments
    if len(sys.argv) < 4:
        print globals()['__doc__'] % locals()
        sys.exit(1)
    inp, outp1, outp2 = sys.argv[1:4]
    model = Word2Vec(LineSentence(inp), size=400, window=5, min_count=5, workers=multiprocessing.cpu_count())
    model.save(outp1)
    model.save_word2vec_format(outp2, binary=False)
```
这里用了一个个`LineSentence`函数，官方文档：[http://radimrehurek.com/gensim/models/word2vec.html](http://radimrehurek.com/gensim/models/word2vec.html)

文档这么说：

>Simple format: one sentence = one line; words already preprocessed and separated by whitespace.
简单的格式：一句话 = 一行，预处理过并且用空白符分隔。

这里我们一篇文章等于一行。

执行训练：
```
python train_word2vec_model.py wiki.zh.text.jian.seg wiki.zh.text.model wiki.zh.text.vector
```
训练速度也还可以。

之后我们就可以根据这个进行Word2Vec相关操作了：
``` python
import gensim

model = gensim.models.Word2Vec.load("wiki.zh.text.model")

#model.most_similar(u"足球")

"""
[(u'\u8054\u8d5b', 0.6553816199302673),
 (u'\u7532\u7ea7', 0.6530429720878601),
 (u'\u7bee\u7403', 0.5967546701431274),
 (u'\u4ff1\u4e50\u90e8', 0.5872289538383484),
 (u'\u4e59\u7ea7', 0.5840631723403931),
 (u'\u8db3\u7403\u961f', 0.5560152530670166),
 (u'\u4e9a\u8db3\u8054', 0.5308005809783936),
 (u'allsvenskan', 0.5249762535095215),
 (u'\u4ee3\u8868\u961f', 0.5214947462081909),
 (u'\u7532\u7ec4', 0.5177896022796631)]
"""

result = model.most_similar(u"足球")

for e in result:
    print e[0], e[1]
"""
联赛 0.65538161993
甲级 0.653042972088
篮球 0.596754670143
俱乐部 0.587228953838
乙级 0.58406317234
足球队 0.556015253067
亚足联 0.530800580978
allsvenskan 0.52497625351
代表队 0.521494746208
甲组 0.51778960228
"""
```
搞完这一波，一天也就差不多过去了……至于训练效果，取决于语料库以及我们的分词效果两点，可以针对这两点进行处理。

## 附：Notebook版
``` python
import logging
import os.path
import sys
from gensim.corpora import WikiCorpus
# if __name__=='__main__':

sys.argv[0] = "process_wiki"
sys.argv[1] = "/root/zhwiki-latest-pages-articles.xml.bz2"
sys.argv[2] = "wiki.zh.text"

program = os.path.basename(sys.argv[0])
logger = logging.getLogger(program)
logging.basicConfig(format='%(asctime)s: %(levelname)s: %(message)s')
logging.root.setLevel(logging.INFO)
console = logging.StreamHandler()
console.setLevel(logging.INFO)
logging.getLogger('').addHandler(console)
logger.info("running %s" % ' '.join(sys.argv))
#check and process input arguments
if len(sys.argv) < 3:
    print(globals()['__doc__'] % locals())
    sys.exit(1)
inp, outp = sys.argv[1:3]
space = " "
i = 0
output = open(outp, 'w')
wiki = WikiCorpus(inp, lemmatize=False, dictionary={})
for text in wiki.get_texts():
    output.write(space.join(text) + "\n")
    i = i + 1
    if (i % 10000 == 0):
        logger.info("Saved " + str(i) + " articles")
output.close()
logger.info("Finished Saved " + str(i) + " articles")
```
``` python
import jieba
import jieba.analyse
import jieba.posseg as pseg
import json
def cut_words(sentence):
    #print sentence
    return " ".join(jieba.cut(sentence))
f = open("/root/wiki.zh.text.jian")
target = open("/root/wiki.zh.text.jian.seg", 'a+')
print('open files')
line = f.readlines(100000)
while line:
    curr = []
    for oneline in line:
        #print(oneline)
        curr.append(oneline)
    '''
    seg_list = jieba.cut_for_search(s)
    words = pseg.cut(s)
    for word, flag in words:
        if flag != 'x':
            print(word)
    for x, w in jieba.analyse.extract_tags(s, withWeight=True):
        print('%s %s' % (x, w))
    '''
    after_cut = map(cut_words, curr)
    # print lin,
#     for words in after_cut:
#         print(words)
#     print(json.dumps(after_cut))
    target.writelines(after_cut)
    print('saved 100000 articles')
    line = f.readlines(100000)
f.close()
target.close()
print("end")
```
``` python
import logging
import os.path
import sys
import multiprocessing
from gensim.corpora import WikiCorpus
from gensim.models import Word2Vec
from gensim.models.word2vec import LineSentence
# if __name__ == '__main__':

sys.argv[0] = "train_word2vec_model.py"
inp = "/root/wiki.zh.text.jian.seg"
outp1 = "/root/wiki.zh.text.model"
outp2 = "/root/wiki.zh.text.vector"

program = os.path.basename(sys.argv[0])
logger = logging.getLogger(program)
logging.basicConfig(format='%(asctime)s: %(levelname)s: %(message)s')
logging.root.setLevel(level=logging.INFO)
logger.info("running %s" % ' '.join(sys.argv))
# check and process input arguments
# if len(sys.argv) < 4:
#     print(globals()['__doc__'] % locals())
#     sys.exit(1)
# inp, outp1, outp2 = sys.argv[1:4]
model = Word2Vec(LineSentence(inp), size=400, window=5, min_count=5, workers=multiprocessing.cpu_count())
model.save(outp1)
model.save_word2vec_format(outp2, binary=False)
```
``` python
import gensim

model = gensim.models.Word2Vec.load("/root/wiki.zh.text.model")
# print(model)
# model.most_similar(u"男人")

result = model.most_similar(u"男人")

for e in result:
    print(e[0], e[1])
```
## 扩展阅读：
---
[深度学习word2vec笔记之基础篇](http://blog.csdn.net/mytestmy/article/details/26961315)

[Deep Learning实战之word2vec](http://blog.csdn.net/mytestmy/article/details/26961315)
