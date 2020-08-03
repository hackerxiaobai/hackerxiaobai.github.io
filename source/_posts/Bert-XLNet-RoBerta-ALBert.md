---
title: Bert XLNet RoBerta ALBert
date: 2019-10-10 11:36:09
tags:
  Bert 
  XLNet 
  RoBerta
  ALBert
---
### Bert
> 自编码语言模型。mask语言模型， next sentence   mask预训练和finetune两阶段不一致，mask之间相互独立
<!--more-->

### XLNet
> 自回归语言模型，但是不采用mask机制， 句子固定其中摸一个词，剩下的词随机排列组合，但是在finetune阶段是不能将句子排列组合原始输入的，所以，就必须让预训练阶段的输入部分，看上去仍然是x1,x2,x3,x4这个输入顺序，但是可以在Transformer部分做些工作，来达成我们希望的目标。具体而言，XLNet采取了Attention掩码的机制，你可以理解为，当前的输入句子是X，要预测的单词Ti是第i个单词，前面1到i-1个单词，在输入部分观察，并没发生变化，该是谁还是谁。但是在Transformer内部，通过Attention掩码，从X的输入单词里面，也就是Ti的上文和下文单词中，随机选择i-1个，放到Ti的上文位置中，把其它单词的输入通过Attention掩码隐藏掉，于是就能够达成我们期望的目标（当然这个所谓放到Ti的上文位置，只是一种形象的说法，其实在内部，就是通过Attention Mask，把其它没有被选到的单词Mask掉，不让它们在预测单词Ti的时候发生作用，如此而已。

> GPT2.0的核心其实是更多更高质量的预训练数据，这个明显也被XLNet吸收进来了；再然后，Transformer XL的主要思想也被吸收进来，它的主要目标是解决Transformer对于长文档NLP应用不够友好的问题。

### RoBerta
**静态Masking vs 动态Masking**
> 原来Bert对每一个序列随机选择15%的Tokens替换成[MASK]，为了消除与下游任务的不匹配，还对这15%的Tokens进行（1）80%的时间替换成[MASK]；（2）10%的时间不变；（3）10%的时间替换成其他词。但整个训练过程，这15%的Tokens一旦被选择就不再改变，也就是说从一开始随机选择了这15%的Tokens，之后的N个epoch里都不再改变了。这就叫做静态Masking。

> 而RoBERTa一开始把预训练的数据复制10份，每一份都随机选择15%的Tokens进行Masking，也就是说，同样的一句话有10种不同的mask方式。然后每份数据都训练N/10个epoch。这就相当于在这N个epoch的训练中，每个序列的被mask的tokens是会变化的。这就叫做动态Masking。

**with NSP vs without NSP**
> 原本的Bert为了捕捉句子之间的关系，使用了NSP任务进行预训练，就是输入一对句子A和B，判断这两个句子是否是连续的。在训练的数据中，50%的B是A的下一个句子，50%的B是随机抽取的。而RoBERTa去除了NSP，而是每次输入连续的多个句子，直到最大长度512（可以跨文章）。这种训练方式叫做（FULL - SENTENCES），而原来的Bert每次只输入两个句子。实验表明在MNLI这种推断句子关系的任务上RoBERTa也能有更好性能。

**更大的mini-batch**
> 原本的BERTbase 的batch size是256，训练1M个steps。RoBERTa的batch size为8k。为什么要用更大的batch size呢？（除了因为他们有钱玩得起外）作者借鉴了在机器翻译中，用更大的batch size配合更大学习率能提升模型优化速率和模型性能的现象，并且也用实验证明了确实Bert还能用更大的batch size

**更多的数据，更长时间的训练**

### ALBert
**词嵌入向量参数的因式分解 Factorized embedding parameterization**
```python
O(V * H) to O(V * E + E * H)

如以ALBert_xxlarge为例，V=30000, H=4096, E=128

那么原先参数为V * H= 30000 * 4096 = 1.23亿个参数，现在则为V * E + E * H = 30000*128+128*4096 = 384万 + 52万 = 436万，

词嵌入相关的参数变化前是变换后的28倍。
```

**跨层参数共享 Cross-Layer Parameter Sharing**
> 参数共享能显著减少参数。共享可以分为全连接层、注意力层的参数共享；注意力层的参数对效果的减弱影响小一点。

**段落连续性任务 Inter-sentence coherence loss.**
> 使用段落连续性任务。正例，使用从一个文档中连续的两个文本段落；负例，使用从一个文档中连续的两个文本段落，但位置调换了。避免使用原有的NSP任务，原有的任务包含隐含了预测主题这类过于简单的任务。

**其他**
```python
1）去掉了dropout  Remove dropout to enlarge capacity of model.
    最大的模型，训练了1百万步后，还是没有过拟合训练数据。说明模型的容量还可以更大，就移除了dropout
    （dropout可以认为是随机的去掉网络中的一部分，同时使网络变小一些）
    We also note that, even after training for 1M steps, our largest models still do not overfit to their training data. 
    As a result, we decide to remove dropout to further increase our model capacity.
    其他型号的模型，在我们的实现中我们还是会保留原始的dropout的比例，防止模型对训练数据的过拟合。
    
2）为加快训练速度，使用LAMB做为优化器 Use LAMB as optimizer, to train with big batch size
  使用了大的batch_size来训练(4096)。 LAMB优化器使得我们可以训练，特别大的批次batch_size，如高达6万。

3）使用n-gram(uni-gram,bi-gram, tri-gram）来做遮蔽语言模型 Use n-gram as make language model
   即以不同的概率使用n-gram,uni-gram的概率最大，bi-gram其次，tri-gram概率最小。
   本项目中目前使用的是在中文上做whole word mask，稍后会更新一下与n-gram mask的效果对比。n-gram从spanBERT中来。

```
