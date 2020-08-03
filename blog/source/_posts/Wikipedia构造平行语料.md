---
title: Wikipedia构造平行语料
date: 2019-08-28 17:06:42
tags:
  - Wikipieda
  - 平行语料
---

> 参考论文 [Learning To Split and Rephrase From Wikipedia Edit History](https://arxiv.org/abs/1808.09468)
<!--more-->
> 该论文主要分析的是如何通过维基百科的编辑历史文件一步步产生高质量的英文版平行语料库,论文中的方法主要是四步
  1. To construct the WikiSplit corpus, we identify edits that involve sentences being split. A list of sentences for each snapshot is obtained by stripping HTML tags and Wikipedia markup and running a sentence break detector.
  	+ 首先筛选出需要被分割的候选句子，根据xml文件中的时间快照字段以及维基百科本身的标记将其拆分成一些句子列表，然后用论文中给出的分句检测器工具将其拆成一个个句子。
  2. Temporally adjacent snapshots of a Wikipedia page are then compared to check for sentences that have undergone a split like that shown in Figure 1.
  	+ 然后通过维基百科提供的每个页面相邻的时间快照来对比核查该句子是否经历了正确的分割，其实论文图1中指的就是增加一些字段和删除一些字段。
  3. To extract a full sentence C and its candidate split into S = (S1,S2),we require that C and S1 have the same trigram prefix, C and S2 have the same trigram suffix, and S1 and S2 have different trigram suffixes. To filter out misaligned pairs, we use BLEU scores (Papineni et al., 2002) to ensure similarity between the original and the split versions.
     we discard pairs where BLEU(C,S1) or BLEU(C, S2) is less than δ (an empirically chosen threshold). If multiple candidates remain for a given sentence C, we retain argmaxS (BLEU(C, S1 ) + BLEU(C, S2 )).
	+ 为了保证源语句和被拆分成的两个子句之间达到一定的相似度，论文里的要求是保证C和S1的trigram 前缀相同，和S2的trigram 后缀相同，S1和S2的trigram后缀不同，还用到了BLEU指标来评定两个句子间的相似度，主要是设定阈值的方式来筛选的。
  4. Our extraction heuristic is imperfect, so we manually assess corpus quality using the same categorization schema proposed by Aharoni and Goldberg.
	+ 这一步主要是人为的操作来评定句子的质量。



### 严格按照论文在英文数据复现得到的结果如下
>A surge of popular interest in anarchism occurred during the [[1970's]] in [[Britain]] following the birth of the [[punk rock]] movement.
>A surge of popular interest in anarchism occurred during the [[1960's]] and [[1970's]].
>In [[Britain]] following the birth of the [[punk rock]] movement.

### 主要目的是想构造中文的平行语料,得到结果如下
>诗是 历史 最 悠久 的 文学 形式 ， 中国 是 世界 上 诗歌 最 发达 的 国度 之一 。
>诗是 历史 最 悠久 的 文学 形式 之一 ， 例如 荷马史诗 。
>中国 是 世界 上 诗歌 最 发达 的 国度 之一 。

>从事 定性分析 的 部分 社会学家 相信 这是 一种 更好 的 方法 ， 他们 认为 ， 这是 一种 可以 有助于 我们 对 一个 “ 离散 ” 性 的 社会 和 独特性 的 人文 的 了解 ， 这种 方法 从不 寻求 有 一致 观点 ， 但 它们 却 可以 互相 欣赏 各自 所 采取 的 独特 方式 并 互相 借鉴 。
>从事 定性分析 的 社会学家 相信 ， 这是 一种 更好 的 方法 ， 因为 这 可以 加强 理解 “ 离散 ” 性 的 社会 和 独特性 的 人文 。
>这种 方法 从不 寻求 有 一致 观点 ， 但 却 可以 互相 欣赏 各自 所 采取 的 独特 方式 并 互相 借鉴 。

>十月 殿试 ， 乾隆帝 亲 往 紫光阁 校阅 ， 见到 马全 ， 发现 相识 ， 问道 ： 「 尔马 瑔 耶 ？
>十月 殿试 ， 乾隆帝 亲 往 紫光阁 校阅 。
>马全 脱颖而出 ， 乾隆帝 竟 将 其 认出 ， 问道 ： 「 尔马 瑔 耶 ？

>洪武 年间 ， 明朝 大军 进攻 云南 ， 改平 缅为麓 川平 缅 军民 宣慰司 ， 才 首次 使用 “ 麓 川 ” 。
>洪武 年间 ， 明朝 大军 进攻 云南 ， 其 国君 思伦发 归顺 明朝 ， 授麓 川 宣慰 使 。
>改平 缅为麓 川平 缅 军民 宣慰司 ， 才 首次 使用 “ 麓 川 ” 。
