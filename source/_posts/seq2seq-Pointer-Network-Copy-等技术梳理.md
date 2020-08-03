---
title: seq2seq Pointer-Network Copy 等技术梳理
date: 2020-06-03 15:17:07
tags: 
	- Pointer-Network 
	- Summarization
	- seq2seq
	- copy
---

> 之前或多或少都有去关注以及该方面的paper阅读，但是并没有去好好的整理该技术的整体发展，今天闲来无事，想从代码以及paper的核心思想梳理一遍。供自己后续方便查看吧。

## Pointer Networks

<!--more-->

**[paper](https://arxiv.org/abs/1506.03134)**

**[pytorch code](https://github.com/shirgur/PointerNet)**

传统的seq2seq模型是无法解决输出序列的词汇表会随着输入序列长度的改变而改变的问题，所以一个很简单的想法就是为什么输出不能直接从输入端直接拿过来呢？那直接从输入端拿过来，具体要怎么操作呢？这就是该篇文章给出的一个思路，该篇文章给出的形象解释是凸包问题，如图所示

{% asset_img 1.png 1.png %}

对上图的一个简单解释：给定p1到p4四个二维坐标，找到一个凸包。答案是p1->p4->p2->p1，图a就是传统的seq2seq做法，就是把四个点的坐标作为输入序列输入进去，然后提供一个词汇表：[start, 1, 2, 3, 4, end]，最后依据词汇表预测出序列[start, 1, 4, 2, 1, end]，缺点作者也提到过了，对于图a的传统seq2seq模型来说，它的输出词汇表已经限定，当输入序列的长度变化的时候（如变为10个点）它根本无法预测大于4的数字。因为你的词汇表限定了最大就是4。图b是作者提出的Pointer Networks，它预测的时候每一步都找当前输入序列中权重最大的那个元素，而由于输出序列完全来自输入序列，它可以适应输入序列的长度变化。那具体的是怎么处理的呢？下面就直接从代码实现层面来简单说一下。

**还是以解凸包问题说起**

每一个batch5个坐标点，那最开始的输入就是：(假设batch 256)

inputs.shape (256，5，2)

假设embedding是128，那inputs 经过embedding后的shape就是：

embedded_inputs（256，5，128）

然后进行encode，假设用了LSTM，（uints假设为512）那它会输出 encoder_outputs 和 encoder_hidden，shape分别是：

encoder_outputs（256, 5, 512） 

encoder_hidden（256, 512）

接下来我们就要开始decode了，重点就是decode端去实现如何直接拿输入的信息了，其实对于这种seq2seq现在都会做一个attention的操作，那该paper其实就是在attention上做了简化，通过attention的操作得到一个alpha，通过alpha间接去拿输入端embedded_inputs 的具体某一个坐标的embedding。下面看一下decode端的一个操作吧：decode的输入主要是这四个值 

embedded_inputs   （256, 5, 128）就是encode端的embedding

decoder_input0  （256, 128）因为是t0时刻，所以这个值最开始是随机初始化的

decoder_hidden0 （256, 512）就是拿了encode端最后一个时刻的隐状态作为decode端的开始状态

encoder_outputs （256, 5, 512） 

将 decoder_input0 和 decoder_hidden0 经过一个时刻的LSTM操作得到t1 的 h_t, 然后将 encoder_outputs 和 t_1 时刻的h_t 输入到attention，此时attention操作就是计算出一个alpha。具体如何计算呢，继续往下看：

我们知道上面操作得到的 h_t维度是（256，512）， encoder_outputs 维度是（256, 5, 512） ，我们将h_t 进行repeat操作，维度变成 （256, 5, 512），h_t 和 encoder_outputs做一下维度变换，变成（256，512，5），然后这个encoder_outputs进行一次Conv1d操作，其实做不做这个操作我觉得影响也不是很大，该操作是不改变维度的，所以经过Conv1d操作后维度还是（256, 5, 512），在attention这里呢我们会一开始初始化一个变量，假设是V吧，，他的维度呢就是（256，512）的矩阵。（该矩阵呢其实是为了后面计算得到alpha的一个中间变量吧）为了矩阵操作方便，我们会将V进行维度扩展，变成（256，1，512），然后做一个这样的操作 

```python
att = torch.bmm(V, self.tanh(h_t+ encoder_outputs)).squeeze(1)
```

所以此时得到的 att 的维度就是 （256，5），此时呢，直接对这个att 进行 softmax操作，得到alpha，然后将 alpha 和 encoder_outputs做一个计算，得到一个 hidden_state 

```python
hidden_state = torch.bmm(encoder_outputs, alpha.unsqueeze(2)).squeeze(2)  
# hidden_state shape is （256，512）
```

至此，attention操作就结束了，最后返回的就是 alpha 和 hidden_state

那到此呢，我们只是拿到了alpha而已，那怎么通过这个alpha直接到embedded_inputs去拿对应索引的embeding呢？接着往下看：

拿到的alpha会做一个max操作，如下：

```python
max_probs, indices = alpha.max(1)
# 因为是输出随着输入变化而变化，所以一开始呢，我们会初始化一个runner，
# 比如我们这里坐标的个数采用了5，所以runner呢就会初始化成 [0,1,2,3,4] 的列表，
# 然后将它repeat到batch_size 大小，所以 runner的shape就是（256，5），然后呢，做一个这样操作：

one_hot_pointers = (runner == indices.unsqueeze(1).expand(-1, alpha.size()[1])).float()
# 所以此时呢 one_hot_pointers 就是一个 0/1 矩阵，其实就是 5个坐标 对应取哪一个坐标的索引嘛
# 然后通过这个矩阵到 embedded_inputs去挑一些embedding来做decode端 t1 时刻的 输入啦，具体操作如下：

# Get embedded inputs by max indices             
embedding_mask = one_hot_pointers.unsqueeze(2).expand(-1, -1, self.embedding_dim).byte()             decoder_input = embedded_inputs[embedding_mask.data].view(batch_size, self.embedding_dim)
# 从这里我们可以看大下一个时刻的 decoder_input 其实是直接通过alpha取max后对应的索引到 
# embedded_inputs直接拿到的。t2、t3...时刻以此类推，都是这样的操作
```

到现在为止，已经解释了如何直接从输入端来取embedding来做为decode端的输入了，但是最终我们要拿到这个凸包的输出还没有解释，下面就简单来看一下吧，其实很简单了：

```python
outputs.append(alpha.unsqueeze(0))             
pointers.append(indices.unsqueeze(1))
# 我们的坐标个数取的是5，所以其实是会循环5次上面这个操作，每一步拿到一个，最后cat起来

outputs = torch.cat(outputs).permute(1, 0, 2)    # （256，5，5）     
pointers = torch.cat(pointers, 1)  # （256，5）
```

最终，返回的outputs 会参与模型计算loss，至此，整个 Pointer Network 的代码实现就解释完了。或许有点懵。直接阅读代码吧，配合这个解释会特别清晰
**阅读code**

**阅读code**

**阅读code**

重要的事情说三遍，Down



## Get To The Point: Summarization with Pointer-Generator Networks

[paper](https://arxiv.org/pdf/1704.04368.pdf)

[code](https://github.com/atulkum/pointer_summarizer)

在这篇论文中，作者认为，用于文本摘要的seq2seq模型往往存在两大缺陷：

+ 模型容易不准确地再现事实细节，也就是说模型生成的摘要不准确；
+ 往往会重复，也就是会重复生成一些词或者句子。而针对这两种缺陷，作者分别使用Pointer Networks和Coverage技术来解决
+ 作者给了一张效果图如下：
+ {% asset_img 2.png 2.png %}



在这张图中，基础的seq2seq模型的预测结果存在许多谬误的句子，同时如nigeria这样的单词反复出现（红色部分）。这也就印证了作者提出的基础seq2seq在文本摘要时存在的问题；Pointer-Generator模型，也就是在seq2seq基础上加上Pointer Networks的模型基本可以做到不出现事实性的错误，但是重复预测句子的问题仍然存在（绿色部分）；最后，在Pointer-Generator模型上增加Coverage机制，可以看出，这次模型预测出的摘要不仅做到了事实正确，同时避免了重复某些句子的问题（摘要结果来自原文中的蓝色部分）

> 那么，Pointer-Generator模型以及变体Pointer-Generator+Coverage模型是怎么做的呢，我们具体从代码层面来分析一下

既然是Pointer Network的进一步改进，那首先想到的就是，如何像Pointer Network那样输出能跟着输入的改变而改变吧？为什么要有这样的操作，其实就是为了解决OOV的问题嘛，假设词表是10000，当你输入的某一个词不在词表中的时候，是不是就要用UNK来代替了，而且这个词也出现在decode端，那么decode端也是UNK了。所以为了解决这个问题，就有了，词表随着输入的扩大而扩大，具体代码体现如下：

```python
def article2ids(article_words, vocab):
  ids = []
  oovs = []
  unk_id = vocab.word2id(UNKNOWN_TOKEN)
  for w in article_words:
    i = vocab.word2id(w)
    if i == unk_id: # If w is OOV
      if w not in oovs: # Add to list of OOVs
        oovs.append(w)
      oov_num = oovs.index(w) # This is 0 for the first article OOV, 1 for the second article OOV...
      ids.append(vocab.size() + oov_num) # This is e.g. 50000 for the first article OOV, 50001 for the second...
    else:
      ids.append(i)
  return ids, oovs


def abstract2ids(abstract_words, vocab, article_oovs):
  ids = []
  unk_id = vocab.word2id(UNKNOWN_TOKEN)
  for w in abstract_words:
    i = vocab.word2id(w)
    if i == unk_id: # If w is an OOV word
      if w in article_oovs: # If w is an in-article OOV
        vocab_idx = vocab.size() + article_oovs.index(w) # Map to its temporary article OOV number
        ids.append(vocab_idx)
      else: # If w is an out-of-article OOV
        ids.append(unk_id) # Map to the UNK token id
    else:
      ids.append(i)
  return ids
```

**从上面代码可以很清晰的看到，但你输入的词超过词表时，问题也不大，词表跟着扩大就行了。**

接着，现在是可以做到词表跟着输入的变化而变化了，但是接下来要怎么做呢？我们知道正常的seq2seq，encode端将词变成索引只要做这样一个操作就是了

```python
self.enc_input = [vocab.word2id(w) for w in article_words]
```

这种操作当出现一个词不在词表中时，就会出现UNK对应的索引了。如果是这样来实现的话：

```python
if config.pointer_gen:
      # Store a version of the enc_input where in-article OOVs are represented by their temporary OOV id; also store the in-article OOVs words themselves
      self.enc_input_extend_vocab, self.article_oovs = data.article2ids(article_words, vocab)

      # Get a verison of the reference summary where in-article OOVs are represented by their temporary article OOV id
      abs_ids_extend_vocab = data.abstract2ids(abstract_words, vocab, self.article_oovs)
```

因为词表跟着输入的扩充变化而变化，所以可以知道 **self.enc_input_extend_vocab** 列表里是不会出现UNK对应的索引的，至此输入端的情况就应该很清楚了。接着就是**encode**  **decode** 的一些操作了

encode端其实是很简单的一些操作，核心代码如下：

```python
def forward(self, input, seq_lens):
    # input.shape (batch_size, seq_length)
    embedded = self.embedding(input)
    packed = pack_padded_sequence(embedded, seq_lens, batch_first=True)
    output, hidden = self.lstm(packed)
    encoder_outputs, _ = pad_packed_sequence(output, batch_first=True)  # h dim = B x t_k x n
    encoder_outputs = encoder_outputs.contiguous()
    encoder_feature = encoder_outputs.view(-1, 2*config.hidden_dim)  # B * t_k x 2*hidden_dim
    # [8, 400, 512] [3200, 512] [[2, 8, 256],[2, 8, 256]]
    return encoder_outputs, encoder_feature, hidden

```

decode端稍微复杂一些，但是其实和Pointer Network 没什么的大的区别，也是在Attention操作的时候，直接拿到一个 （batch_size，seq_length)的概率分布矩阵，直接softmax操作，作为概率返回，看一下attention的核心代码吧

```python
def forward(self, s_t_hat, encoder_outputs, encoder_feature, enc_padding_mask, coverage):
        b, t_k, n = list(encoder_outputs.size())
        dec_fea = self.decode_proj(s_t_hat) # B x 2*hidden_dim
        dec_fea_expanded = dec_fea.unsqueeze(1).expand(b, t_k, n).contiguous() # B x t_k x 2*hidden_dim
        dec_fea_expanded = dec_fea_expanded.view(-1, n)  # B * t_k x 2*hidden_dim

        att_features = encoder_feature + dec_fea_expanded # B * t_k x 2*hidden_dim
        if config.is_coverage:
            coverage_input = coverage.view(-1, 1)  # B * t_k x 1
            coverage_feature = self.W_c(coverage_input)  # B * t_k x 2*hidden_dim
            att_features = att_features + coverage_feature

        e = F.tanh(att_features) # B * t_k x 2*hidden_dim
        scores = self.v(e)  # B * t_k x 1
        scores = scores.view(-1, t_k)  # B x t_k

        attn_dist_ = F.softmax(scores, dim=1)*enc_padding_mask # B x t_k
        normalization_factor = attn_dist_.sum(1, keepdim=True)
        attn_dist = attn_dist_ / normalization_factor

        attn_dist = attn_dist.unsqueeze(1)  # B x 1 x t_k
        c_t = torch.bmm(attn_dist, encoder_outputs)  # B x 1 x n
        c_t = c_t.view(-1, config.hidden_dim * 2)  # B x 2*hidden_dim

        attn_dist = attn_dist.view(-1, t_k)  # B x t_k

        if config.is_coverage:
            coverage = coverage.view(-1, t_k)
            coverage = coverage + attn_dist

        return c_t, attn_dist, coverage
```

可以看到**attn_dist**其实和Pointer Network的实现或者说操作是一样的，没有什么大的区别，coverage是为了惩罚重复出现问题而计算的一个矩阵

在attention计算完毕，decode端是如何来计算整体的概率分布呢？还是直接看代码吧

```python
p_gen = None
if config.pointer_gen:
  p_gen_input = torch.cat((c_t, s_t_hat, x), 1)  # B x (2*2*hidden_dim + emb_dim)
  p_gen = self.p_gen_linear(p_gen_input)
  p_gen = F.sigmoid(p_gen)

output = torch.cat((lstm_out.view(-1, config.hidden_dim), c_t), 1) # B x hidden_dim * 3
output = self.out1(output) # B x hidden_dim
output = self.out2(output) # B x vocab_size
vocab_dist = F.softmax(output, dim=1)

if config.pointer_gen:
  vocab_dist_ = p_gen * vocab_dist
  attn_dist_ = (1 - p_gen) * attn_dist

  if extra_zeros is not None:
    vocab_dist_ = torch.cat([vocab_dist_, extra_zeros], 1)
    final_dist = vocab_dist_.scatter_add(1, enc_batch_extend_vocab, attn_dist_)
else:
  final_dist = vocab_dist
```

至此，我觉得我自己应该大差不差的可以搞清楚了。如果你没有搞明白，还是那句话，代码面前没有秘密。看代码去吧（插一句，这份代码中有好几个地方我觉得是有待考虑的，在实现上，但是主体还是OK的）



## CopyNet

[paper](https://arxiv.org/pdf/1603.06393.pdf)

[code]()

该篇文章开篇作者提到要解决的问题就是赋予seq2seq复制的能力，如下所示：

{% asset_img 3.png 3.png %}

从这个例子中我们可以看到，针对绿色的这部分词汇其实是不需要去理解语意的，直接从输入端copy到输出端就可以了，那我们该如何去实现这个功能呢？，下面我们直接从代码上进行解释：

##### encode

```python
class CopyEncoder(nn.Module):
    def __init__(self, vocab_size, embed_size, hidden_size):
        super(CopyEncoder, self).__init__()

        self.embed = nn.Embedding(vocab_size, embed_size)

        self.gru = nn.GRU(input_size=embed_size,
            hidden_size=hidden_size, batch_first=True,
            bidirectional=True)

    def forward(self, x):
        # input: [b x seq]
        embedded = self.embed(x)
        out, h = self.gru(embedded) # out: [b x seq x hid*2] (biRNN)
        return out, h
```

encod部分代码其实不需要进行什么解释了，很容易就理解了

##### decode

```python
# 1. input_idx 就是decode端第一次的输入，weighted第一次也是初始化的
gru_input = torch.cat([self.embed(input_idx).unsqueeze(1), weighted],2) # [b x 1 x (h*2+emb)]
_, state = self.gru(gru_input, prev_state)
state = state.squeeze() # [b x h]

# 2.拿到state后直接做一个dense得到生成词汇表的概率分布
score_g = self.Wo(state) # [b x vocab_size]

# 3. 接着我们需要得到一个copy的概率分布，这个分布当然得根据encode的输出和state一起来计算啦，如下
score_c = F.tanh(self.Wc(encoded.contiguous().view(-1,hidden_size*2))) # [b*seq x hidden_size]
score_c = score_c.view(b,-1,hidden_size) # [b x seq x hidden_size]
score_c = torch.bmm(score_c, state.unsqueeze(2)).squeeze() # [b x seq]

# 4. 为了避免padding部分在计算softmax时带来的影响，做了一个mask如下：
encoded_mask = torch.Tensor(np.array(encoded_idx==0, dtype=float)*(-1000)) # [b x seq]
score_c = score_c + encoded_mask # padded parts will get close to 0 when applying softmax

# 5. 将score_g 和 score_c 一起计算softmax：
 score = torch.cat([score_g,score_c],1) # [b x (vocab+seq)]
 probs = F.softmax(score)
 prob_g = probs[:,:vocab_size] # [b x vocab]
 prob_c = probs[:,vocab_size:] # [b x seq]

# 6. 此时我们知道 prob_g 和 prob_c 肯定是不能直接相加的，而且 prob_g 也还是一个词汇表的概率分布，但oov时还是不能搞定，
# 所以这里和上一篇文章做法稍微有点不同，这里是固定oov的大小，和上一篇文章动态的变化大小有所区别，其实要改成动态变化也是一样的。
oovs = Variable(torch.Tensor(b,self.max_oovs).zero_())+1e-4
oovs = self.to_cuda(oovs)
prob_g = torch.cat([prob_g,oovs],1)

# 7. 而为了要实现可以将prob_g 和 prob_c 相加，做了下面一个操作，encoded_idx 是输入端的一些词汇表表示的数字， 
# 初始化一个prob_g大小的one_hot矩阵，根据 encoded_idx 的数字进行补1操作，最后将prob_c 的概率分布放到one_hot 
# 上对应的1的位置上，此时，prob_g 和 prob_c 大小是一样的，当然可以直接相加啦
en = torch.LongTensor(encoded_idx) # [b x in_seq]
en.unsqueeze_(2) # [b x in_seq x 1]
one_hot = torch.FloatTensor(en.size(0),en.size(1),prob_g.size(1)).zero_() # [b x in_seq x vocab+oov_nums]
one_hot.scatter_(2,en,1) # one hot tensor: [b x seq x vocab]
one_hot = self.to_cuda(one_hot)
prob_c_to_g = torch.bmm(prob_c.unsqueeze(1),Variable(one_hot, requires_grad=False)) # [b x 1 x vocab]
prob_c_to_g = prob_c_to_g.squeeze() # [b x vocab]

out = prob_g + prob_c_to_g
out = out.unsqueeze(1) # [b x 1 x vocab]
```

其实代码后面针对weighted 和 state 还做了一部分操作，这都是次要的，只要理解了out的全部计算过程，可以说CopyNet 的核心思想你也就掌握了，其实这里和上一篇文章没有什么大的差别，这里是直接相加，上一篇文章弄了个软概率来合并，还有一个覆盖操作，大差不差吧。重要的事情说三遍。

**阅读code**

**阅读code**

**阅读code**



### 结论

> Seq2seq 的copy机制可以暂时告一段落啦。。。