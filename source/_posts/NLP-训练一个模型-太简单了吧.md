---
title: 'NLP 训练一个模型,太简单了吧'
date: 2020-07-31 19:23:25
tags:
	- NLP
	- huggingface
	- transformer
	- 训练模型
---

### 太简单了吧!!!

​        **现在随着深度学习各种框架以及高阶API的封装,自己从头搭一个模型真的是成本太低了,说白了,也就是现在入门或者简单跑跑demo的门槛几乎为零,接下来就简单的聊聊两个库的使用吧**

+ [transformers](https://github.com/huggingface/transformers)
+ [nlp](https://github.com/huggingface/nlp)

<!-- more -->

> 对于NLPer 的同学来说, **[transformres](https://github.com/huggingface/transformers)** 这个包应该是在熟悉不过了,如果你还没用过这个东西,说明你已经落后啦(其实也不能这样说,应该说如果你还没fine-tuning过bert相关模型,那你就落后主流NLP技术啦)

> **[nlp](https://github.com/huggingface/nlp)** 这个包其实是类似 Tensorflow Datasets 的一个东西,反正就是来对数据预处理转成tensor的一个东西

> 废话少说, talk is cheaper, show me the code.



### 以文本分类为demo说起吧

+ 如果单纯的使用 transformers来做文本分类,大概是长下面这个样子的

  1. 下载bert预训练文件

  2. 加载两个类  BertForSequenceClassification  , BertTokenizer

  3. 定义 optimizer

  4. 加载数据转到batch-tensor, 数据大概长下面这个样子(anyway, 随你自己怎么定吧)

      - ```python
        {
          'version':'0.0.1',
          'train':[
            {
              'sent':'我是一个句子',
              'label':'pos'
            },
            {
              'sent':'我是一个句子',
              'label':'pos'
            },
            ...
          ]
        }
        ```

  5. 训练模型

```python
import torch
import json
from tqdm import tqdm
from transformers import BertForSequenceClassification,BertTokenizer,AdamW, set_seed

device = 'cuda' if torch.cuda.is_available() else 'cpu'
pretrain_file = '/nfs/users/wanglei/github/BERT-NER-Pytorch-master/prev_trained_model/bert-base'
model = BertForSequenceClassification.from_pretrained(pretrain_file,num_labels=3).to(device)
tokenizer = BertTokenizer.from_pretrained(pretrain_file)

no_decay = ['bias', 'LayerNorm.weight']
optimizer_grouped_parameters = [
    {'params': [p for n, p in model.named_parameters() if not any(nd in n for nd in no_decay)], 'weight_decay': 0.01},
    {'params': [p for n, p in model.named_parameters() if any(nd in n for nd in no_decay)], 'weight_decay': 0.0}
]
optimizer = AdamW(optimizer_grouped_parameters, lr=5e-5)

def train():
    with open('./data/train.json') as f:
        train = json.loads(f.read())
        D,L = [],[]
        Dd,Ld = [],[]
        for epoch in tqdm(range(5)):
            model.train()
            tbar = tqdm(train['train'])
            for line in tbar:
                sent = line['sent']
                label = int(line['label'])
                D.append(sent)
                L.append(label)
                if len(D)%64==0:
                    encoding = tokenizer(D, return_tensors='pt', padding=True, truncation=True, max_length=50)
                    input_ids = encoding['input_ids'].to(device)
                    attention_mask = encoding['attention_mask'].to(device)
                    labels = torch.tensor(L).unsqueeze(1).to(device)
                    optimizer.zero_grad()
                    outputs = model(input_ids, attention_mask=attention_mask, labels=labels)
                    loss = outputs[0]
                    loss.backward()
                    optimizer.step()
                    tbar.set_description('Loss is {}'.format(loss.item()))
                    D,L = [],[]

if __name__ == "__main__":
    set_seed(1234)
    train()
```

> 没啥废话好说的,一个文本分类任务就是这么快,不费任何脑子



> 上面demo还是有一点烦, 需要自己构造一个batch,以及model.forward()  loss.backward() optimizer.step() 等一系列操作,那能不能不要自己弄呢,答案是,可以,看下面 

+ 如果在加上 nlp 这个数据预处理模块,那真的是更加酸爽,直接看代码吧,处理的方式大致是一样的

  1. 下载bert预训练文件

  2. 加载两个类  BertForSequenceClassification  , BertTokenizer

  3. 通过 nlp 来对数据做预处理,转成batch-tensor, 数据大概就是一个json 一行存储,如下:

     1. ```python
        {'sent''我是一个句子', 'label':'1'}
        {'sent''我是一个句子', 'label':'1'}
        ```

  4. 定义 TrainingArguments
  5. 通过 Trainer 开启 训练,验证,推理等一系列操作

```python
from nlp import load_dataset
from transformers import BertForSequenceClassification,Trainer, TrainingArguments
from transformers import BertTokenizer
import torch
from sklearn.metrics import accuracy_score, precision_recall_fscore_support


device = 'cuda' if torch.cuda.is_available() else 'cpu'
pretrain_file = '/nfs/users/wanglei/github/BERT-NER-Pytorch-master/prev_trained_model/bert-base'
model = BertForSequenceClassification.from_pretrained(pretrain_file, num_labels=3).to(device)
tokenizer = BertTokenizer.from_pretrained(pretrain_file)

def tokenize(batch):
    return tokenizer(batch['sent'], padding=True, truncation=True)

train_dataset, test_dataset = load_dataset('json', data_files={'train': '/nfs/users/wanglei/transformers/nlp/data/train.json',
                                              'test': '/nfs/users/wanglei/transformers/nlp/data/dev.json'},
                                              split=['train', 'test'])
# 分词
train_dataset = train_dataset.map(tokenize, batched=True, batch_size=len(train_dataset))
test_dataset = test_dataset.map(tokenize, batched=True, batch_size=len(test_dataset))

# 标签转整形
train_dataset = train_dataset.map(lambda examples: {'labels': [int(_) for _ in examples['label']]}, batched=True)
test_dataset = test_dataset.map(lambda examples: {'labels': [int(_) for _ in examples['label']]}, batched=True)

# 返回pytorch 的数据格式
train_dataset.set_format('torch', columns=['input_ids', 'attention_mask', 'labels'])
test_dataset.set_format('torch', columns=['input_ids', 'attention_mask', 'labels'])

# 使用loader的方式也是可以的,自己loss.backward() optimizer.step() 等操作吧
# dataloader = torch.utils.data.DataLoader(train_dataset, batch_size=32)

training_args = TrainingArguments(
    output_dir='./results',
    num_train_epochs=1,
    per_device_train_batch_size=4,
    per_device_eval_batch_size=4,
    warmup_steps=500,
    weight_decay=0.01,
    evaluate_during_training=True,
    logging_dir='./logs',
)

trainer = Trainer(
    model=model,
    args=training_args,
    train_dataset=train_dataset,
    eval_dataset=test_dataset
)

trainer.train()

trainer.evaluate()
```



**就是这么简单 !!!!!!!**