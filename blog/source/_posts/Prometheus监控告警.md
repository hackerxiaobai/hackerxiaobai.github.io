---
title: Prometheus监控告警
date: 2019-09-02 09:43:22
tags:
  - Prometheus
  - Alert
---

# 算法部门接入Prometheus，自定义报警监控步骤

## 本文档不解释相关API的使用，自行查询

> 总体分3个步骤
> 1. 在自己的服务模型中写入需要监控的代码，代码部署，运行起来，可以在[监控页面](普罗米修斯提供的网页)搜索到自己的服务，可以看到你自己监控的一些数据，说明代码这一步就成功啦
> 2. 配合Grafana跳转到图表显示面，进行配置。首先需要登陆，然后右上角会有一个add panel按钮，增加视图。
>  点击add panel ，然后在刚创建的 New Panel 中选择 Choose Visualization 按钮，默认是Graph，接着点击左侧的Queries，在查询输入框中填入要查询的 metric, 创建即可完成 
> 3. 接入Alert报警配置
<!--more-->
### 接下来具体说明上述3个步骤

#### step1 代码层

```python 
'''
在自己的模型中写入相关代码，参考以下代码
该代码监控的是用户输入不合法句子长度的个数
'''
from prometheus_client import Metric, push_to_gateway, CollectorRegistry, Gauge, Counter

class UserCommentRank(SimplexBaseModel):
    def __init__(self, *args, **kwargs):
        super(UserCommentRank, self).__init__(*args, **kwargs)
        
        # 生成 registry 实例
        self.registry = CollectorRegistry()
        
        # 统计不符合句子长度的句子数量
        # 创建 counter 类型对象
        # metric_name 名称在当前 register 中必须唯一, 建议使用 service 名称作为前缀
        # 描述信息可使用中英文
        # 指定 registry 收集该对象的监控数据
        # label_name 列表
        self.SENT_LEN_COUNTS = Counter('usercommentrank_sent_len_num','不符合长度句子个数',registry=self.registry,labelnames=['labels1'])
        
        # 别的一些代码
        
    def predict(self, data, **kwargs):
    
        # 处理你自己业务的一些代码
        
        illegal_sent_len_count = 0
        for comment, score in zip(raw_tmp, list(tmp_pred)):
            if len(comment) < 15:
                illegal_sent_len_count += 1
        
        # 计数统计加, 并添加 label valu
        self.SENT_LEN_COUNTS.labels(labels1='illegal_sent_len_value').inc(illegal_sent_len_count)
        
        # 将监控数据推送至 pushgateway, 设置 job 的名称为 service 的名称, 指定 registry
        # job 可以理解为该监控指标所属的集合, 名称必须全局唯一
        push_to_gateway('pgw.monitor.aipp.io', job="usercommentrank_wl", registry=self.registry)
```
> 部署，运行起来,在[监控页面](http://ddprom.monitor.aipp.io/graph?g0.range_input=130m&g0.moment_input=2019-08-30%2006%3A32%3A08&g0.expr=&g0.tab=0)可以看到如下信息就说明代码层没有问题啦

{% asset_img 1.png 1.png  %}

#### step2 Grafana层
> 1. 右下角登陆
> 2. 点击add panel {% asset_img 2.png 2.png  %}
> 3. 看到下图 {% asset_img 3.png 3.png  %}
> 4. 点击Choose Visualization看到下图{% asset_img 4.png 4.png  %}
> 5. 直接点击{% asset_img 5.png 5.png %}看到如下图，填写红线部分就OK啦（按自己的需求填写），右上角保存、刷新，此时就会看到你定的图
> {% asset_img 6.png 6.png  %}
#### step3 Alert报警接入层
> 1. 第二步完成后，按需配置告警，按照下面两图配置就好啦。（按自己的需求设置）
> {% asset_img 7.png 7.png  %}
> {% asset_img 8.png 8.png  %}
> 2. **记得保存**，最后你就可以将这个panel拖到你的你想放的地方中去啦。


**自此，整个流程就打通啦**



