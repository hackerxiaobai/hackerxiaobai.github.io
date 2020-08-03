---
title: wordpiece和sentencepiece
date: 2019-08-27 16:07:58
tags: 分词
---
### wordpiece
1. wordpiece的主要实现方式有BPE,BPE的大概训练过程：首先将词分成一个一个的字符，然后在词的范围内统计字符对出现的次数，每次将次数最多的字符对保存起来，直到循环次数结束。
<!-- more -->
### sentencepiece
1. 原理是重复出现次数多的片断，就认为是一个意群（词）
```python
# 安装
sudo pip install SentencePiece

# 命令行训练
spm_train --input=/tmp/a.txt --model_prefix=/tmp/test
# --input指定需要训练的文本文件，--model_prefix指定训练好的模型名，本例中生成/tmp/test.model 
# 和/tmp/test.vocab两个文件，vocab是词典信息。

# 命令行调用
echo "食材上不会有这样的纠结" | spm_encode --model=/tmp/test.model


# python调用
import sentencepiece as spm
sp = spm.SentencePieceProcessor()
text = "食材上不会有这样的纠结" 
sp.Load("/tmp/test.model") 
print(sp.EncodeAsPieces(text))

```
