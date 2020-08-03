---
title: Cython与Spacy加速NLP项目
date: 2019-08-28 10:38:23
tags: 
  - Cython
  - Spacy
  - NLP加速
---
# Cython与Spacy合用加速NLP项目
> 本次报告主要内容参考这篇[博客](https://medium.com/huggingface/100-times-faster-natural-language-processing-in-python-ee32033bdced),重点在spacy与自定义数据结构的分析

### 主要内容
+ 如何用 Python 设计一个高速模块
+ 如何利用 spaCy 的内部数据结构来有效地设计超高速 NLP 函数。
<!--more-->
### 何时需要加速NLP项目
+ 你正在使用 Python 开发一个 NLP 的生产模块
+ 你正在使用 Python 计算分析大型 NLP 数据集
+ 你正在为深度学习框架，如 PyTorch / TensorFlow，预处理大型训练集，或者你的深度学习批处理加载器中的处理逻辑过于繁重，这会降低训练速度

### 预热代码
> 假设我们有一大堆矩形，并将它们存储进一个 Python 对象列表，例如 Rectangle 类的实例。我们的模块的主要工作是迭代这个列表，以便计算有多少矩形的面积大于特定的阈值。

```python 
from random import random

class Rectangle:
    def __init__(self, w, h):
        self.w = w
        self.h = h
    def area(self):
        return self.w * self.h

def check_rectangles_py(rectangles, threshold):
    n_out = 0
    for rectangle in rectangles:
        if rectangle.area() > threshold:
            n_out += 1
    return n_out

def main_rectangles_slow():
    n_rectangles = 10000000
    rectangles = list(Rectangle(random(), random()) for i in range(n_rectangles))
    n_out = check_rectangles_py(rectangles, threshold=0.25)
    print(n_out)


%%time
main_rectangles_slow()

'''
4033950
CPU times: user 15.5 s, sys: 1.06 s, total: 16.5 s
Wall time: 16.9 s
'''

```
> check_rectangles_py 函数是瓶颈部分！它对大量的 Python 对象进行循环，这可能会很慢，因为 Python 解释器在每次迭代时都会做大量工作（寻找类中的求面积方法、打包和解包参数、调用 Python API 
> 接下来用Cython进行改写
+ Cython 语言是 Python 的超集，它包含两种对象：
  1. Python 对象是我们在常规 Python 中操作的对象，如数字、字符串、列表、类实例
  2. Cython C 对象是 C 或 C ++ 对象，比如 double、int、float、struct、vectors。这些可以由 Cython 在超快速的底层代码中编译
> 快速循环只是 Cython 程序（只能访问 Cython C 对象）中的一个循环
> 设计这样一个循环的直接方法是定义 C 结构，它将包含我们在计算过程中需要的所有要素：在我们的例子中，就是矩形的长度和宽度
> 然后，我们可以将矩形列表存储在这种结构的 C 数组中，并将这个数组传递给我们的 check_rectangle_cy 函数。此函数现在接受一个 C 数组作为输入，因此通过 cdef 关键字而不是 def 将其定义为 Cython 函数

**Python 模块的快速 Cython 版**

```python 
%%cython
from cymem.cymem cimport Pool
from random import random

cdef struct Rectangle:
    float w
    float h

cdef int check_rectangles_cy(Rectangle* rectangles, int n_rectangles, float threshold):
    cdef int n_out = 0
    # C arrays contain no size information => we need to state it explicitly
    for rectangle in rectangles[:n_rectangles]:
        if rectangle.w * rectangle.h > threshold:
            n_out += 1
    return n_out

def main_rectangles_fast():
    cdef int n_rectangles = 10000000
    cdef float threshold = 0.25
    cdef Pool mem = Pool()
    cdef Rectangle* rectangles = <Rectangle*>mem.alloc(n_rectangles, sizeof(Rectangle))
    for i in range(n_rectangles):
        rectangles[i].w = random()
        rectangles[i].h = random()
    n_out = check_rectangles_cy(rectangles, n_rectangles, threshold)
    print(n_out)


%%time
main_rectangles_fast()

'''
4032951
CPU times: user 704 ms, sys: 168 ms, total: 872 ms
Wall time: 894 ms
'''
```

> 我们在这里使用了原生 C 指针数组，但你也可以选择其他选项，特别是 C ++ 结构，如向量、对、队列等。在这个片段中，我还使用了 cymem 的便利的 Pool（）内存管理对象，以避免必须手动释放分配的 C 数组。当 Pool 由 Python 当做垃圾回收时，它会自动释放我们使用它分配的内存。

**以上代码事例主要是简单分析了如何自定义数据结构来改写一般的python代码，以实现加速功能，但还没有很好的涉及到NLP中大量的字符串操作**

### 使用 Cython 与 spaCy 来加速 NLP
> 官方的 Cython 文档甚至建议不要使用 C 字符串.
一般来说,除非你知道自己在做什么，否则应尽可能避免使用 C 字符串，而应使用 Python 字符串对象。
那么我们如何在使用字符串时在 Cython 中设计快速循环？
spaCy 会帮我们.
spaCy 解决这个问题的方式非常聪明。

**将所有字符串转换为 64 位哈希码**

> spaCy 中的所有 unicode 字符串（token 的文本、其小写文本、引理形式、POS 键标签、解析树依赖关系标签、命名实体标签...）都存储在叫 StringStore 的单数据结构中，它们在里面由 64 位散列索引，即 C uint64_t
StringStore 对象实现了 Python unicode 字符串和 64 位哈希码之间的查找表。

{% asset_img spacy.png spacy  %}

> 它可以通过 spaCy 任意处及任意对象访问（请参阅上图），例如 nlp.vocab.strings、doc.vocab.strings 或 span.doc.vocab.string

**当某个模块需要对某些 token 执行快速处理时，仅使用 C 级别的 64 位哈希码而不是字符串。调用 StringStore 查找表将返回与哈希码相关联的 Python unicode 字符串**

> 但是，spaCy 做的远不止这些，它使我们能够访问文档和词汇表的完全覆盖的 C 结构，我们可以在 Cython 循环中使用这些结构，而不必自定义结构

**spaCy 的内部数据结构**

> 与 spaCy Doc 对象关联的主要数据结构是 Doc 对象，该对象拥有已处理字符串的 token 序列（「单词」）以及 C 对象中的所有称为 doc.c 的标注，它是一个 TokenC 结构数组。
TokenC 结构包含我们需要的关于每个 token 的所有信息。这些信息以 64 位哈希码的形式存储，可以重新关联到 unicode 字符串，就像我们刚刚看到的那样,要深入了解这些 C 结构中的内容，只需查看SpaCy 的 [Cython API doc](https://spacy.io/api)

**使用 spaCy 和 Cython 进行快速 NLP 处理**

> 假设我们有一个需要分析的文本数据集

```python 
import urllib.request
import spacy
# Build a dataset of 10 parsed document extracted from the Wikitext-2 dataset
url='https://raw.githubusercontent.com/pytorch/examples/master/word_language_model/data/wikitext-2/valid.txt'
with urllib.request.urlopen(url) as response:
   text = response.read()
nlp = spacy.load('en')
doc_list = list(nlp(text[:800000].decode('utf8')) for i in range(10))
```

> 该脚本生成用于 spaCy 解析的 10 份文档的列表，每个文档大约 170k 字。我们也可以生成每个文档 10 个单词的 170k 份文档（比如对话数据集）
我们想要在这个数据集上执行一些 NLP 任务。例如，我们想要统计数据集中单词run作为名词的次数（即用 spaCy 标记为NN词性）
一个简单明了的 Python 循环就可以做到

```python
def slow_loop(doc_list, word, tag):
    n_out = 0
    for doc in doc_list:
        for tok in doc:
            if tok.lower_ == word and tok.tag_ == tag:
                n_out += 1
    return n_out

def main_nlp_slow(doc_list):
    n_out = slow_loop(doc_list, 'run', 'NN')
    print(n_out)


%%time
main_nlp_slow(doc_list)

'''
90
CPU times: user 1.34 s, sys: 4 ms, total: 1.34 s
Wall time: 1.38 s
'''
```

**Cython进行改写**

```python 
%%cython -+
import numpy # Sometime we have a fail to import numpy compilation error if we don't import numpy
from cymem.cymem cimport Pool
from spacy.tokens.doc cimport Doc
from spacy.typedefs cimport hash_t
from spacy.structs cimport TokenC

cdef struct DocElement:
    TokenC* c
    int length

cdef int fast_loop(DocElement* docs, int n_docs, hash_t word, hash_t tag):
    cdef int n_out = 0
    for doc in docs[:n_docs]:
        for c in doc.c[:doc.length]:
            if c.lex.lower == word and c.tag == tag:
                n_out += 1
    return n_out

cpdef main_nlp_fast(doc_list):
    cdef int i, n_out, n_docs = len(doc_list)
    cdef Pool mem = Pool()
    cdef DocElement* docs = <DocElement*>mem.alloc(n_docs, sizeof(DocElement))
    cdef Doc doc
    for i, doc in enumerate(doc_list): # Populate our database structure
        docs[i].c = doc.c
        docs[i].length = (<Doc>doc).length
    word_hash = doc.vocab.strings.add('run')
    tag_hash = doc.vocab.strings.add('NN')
    n_out = fast_loop(docs, n_docs, word_hash, tag_hash)
    print(n_out)


%%time
main_nlp_fast(doc_list)

'''
90
CPU times: user 16 ms, sys: 4 ms, total: 20 ms
Wall time: 20 ms
'''
```

> 代码有点长，因为我们必须在调用 Cython 函数之前在 main_nlp_fast 中声明并填充 C 结构。（如果你在代码中多次使用低级结构，使用 C 结构包装的 Cython 扩展类型来设计我们的 Python 代码是比每次填充 C 结构更优雅的选择。这就是大多数 spaCy 的结构，它是一种结合了快速，低内存以及与外部 Python 库和函数接口的简便性的非常优雅的方法。

**对上述代码的一个解释**

> 主要是以下6类,详细介绍还是看官方文档吧
  + Doc
  + Token
  + Span
  + Lexeme
  + Vocab
  + StringStore


```python 
import spacy
nlp = spacy.load('en')
sents = 'i like eat apple,this is a test python file.'
doc = nlp(sents) 
'''
doc就成了一个Doc对象，并且是一个经过分词后的可迭代对象
doc.c 指向TokenC，TokenC是一个spacy定义好的结构体指针
doc.vocab 指向Vocab对象，该对象中有一个strings属性
doc.vocab.strings 指向StringStore对象，存储着所有的词汇
StringStore对象中有一个keys属性，包含所有的hash值，该属性与doc.vocab.strings包含的所有词汇有着一一对应的关系
doc.vocab.strings.add('run')的意思就是将run添加到词汇表并返回字符串'run'的hash值
c.lex.lower 得到的是hash值，如想得到它对应的string，则可以用doc.vocab.strings[c.lex.lower]方式得到

上述代码总结一句话就是尽量的将字符串操作转到hash上的操作

'''

print('所有的词汇')
for vocab in doc.vocab.strings:
  print(vocab)
```

**结束**
