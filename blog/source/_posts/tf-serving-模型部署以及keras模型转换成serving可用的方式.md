---
title: tf-serving 模型部署以及keras模型转换成serving可用的方式
date: 2020-07-31 20:11:12
tags:
	- tf-serving
	- keras 
	- model 转成 serving
---

### tf-serving 模型部署

+ 由于之前一直都是使用pytorch,部署就一直也没去关心tf-serving那一套鬼东西,近期使用了一段时间tf,keras,涉及到部署问题,当然这就离不开serving了,所以就做一个使用总结,供自己后续参考吧,别又忘记了
+ 网上资料一大堆,如果你是熟悉这些东西的,没有必要往下看了
+ 那就先从一般的 tf模型代码说起吧

<!-- more -->

> 简单定义一个tf模型

```python
import os
import shutil
 
import numpy as np
import tensorflow as tf
 
 
def add_layer(inputs, input_size, output_size, activation_function=None):
    weights = tf.Variable(tf.random_normal([input_size, output_size]))
    biases = tf.Variable(tf.zeros([1, output_size]) + 0.1)
    wx_plus_b = tf.matmul(inputs, weights) + biases  # WX + b
    if activation_function is None:
        outputs = wx_plus_b
    else:
        outputs = activation_function(wx_plus_b)
    return outputs
 
 
# 造一些随机输入数据
num_points = 30000  # 总数据条数
feature_number = 100  # 每条输入数据有100个feature
# num_points个输入数据,每个有feature_number个feature,即输入数据的维度是(num_points,feature_number)
x_data = np.random.rand(num_points, feature_number)
y_data = np.random.randint(0, 2, (num_points, 1))  # nx1的数组, 每一行为1个数(0或1)
 
# 用于接收输入的Tensor
x_actual = tf.placeholder(tf.float32, [None, feature_number], name="myInput")
y_actual = tf.placeholder(tf.float32, [None, 1], name="myOutput")
 
# 隐层1
l1 = add_layer(x_actual, feature_number, 32, activation_function=tf.nn.relu)
# 隐层2
l2 = add_layer(l1, 32, 64, activation_function=tf.nn.tanh)
# 隐层3
l3 = add_layer(l2, 64, 32, activation_function=tf.nn.relu)

# 输出层
y_predict = add_layer(l3, 32, 1, activation_function=tf.nn.sigmoid)
# 损失函数
loss = -tf.reduce_mean(y_actual * tf.log(tf.clip_by_value(y_predict, 1e-10, 1.0)))
# 优化器
train_step = tf.train.AdamOptimizer(0.001).minimize(loss)
 
init = tf.global_variables_initializer()
# 迭代次数
num_iterations = 10000
with tf.Session() as sess:
    sess.run(init)
    for i in range(num_iterations):
        # 训练模型
        sess.run(train_step, feed_dict={x_actual: x_data, y_actual: y_data})
        if i % 500 == 0:
            prediction_value = sess.run(y_predict, feed_dict={x_actual: x_data})
            print(sess.run(loss, feed_dict={x_actual: x_data, y_actual: y_data}))
    # 训练完成后,以SavedModel格式保存模型文件
    model_output_dir = "./model/201908070001"
    if os.path.exists(model_output_dir):  # 目录存在
        shutil.rmtree(model_output_dir)  # 删除原目录
    tf.saved_model.simple_save(
        sess, model_output_dir, inputs={"myInput": x_actual}, outputs={"myOutput": y_predict})
```

+ 保存模型的目录大概是有这几个文件

{% asset_img 1.jpg 说明 %}

+ 接下来拉一个TensorFlow-Serving的Docker镜像
  + **docker pull tensorflow/serving**

+ 下载完后，再 cd 到上面输出的“model”目录的上一级目录,运行下面的命令

  + TESTDATA="$(pwd)/model"

    docker run -t --rm -p 8501:8501 \

      -v "$TESTDATA:/models/simple_fc_nn" \

      -e MODEL_NAME=simple_fc_nn \

      tensorflow/serving 

+ 这样就可以把模型serve起来了。其中，端口号可以自己改，simple_fc_nn是我自己起的模型名称，在后面使用REST API来访问TF-Serving服务的时候，会用到这个名称。
+ 通过REST API查看服务状态
  + **curl http://localhost:8501/v1/models/simple_fc_nn**
+ 这里可以看到，URL里的simple_fc_nn就是在启动TF-Serving服务的时候指定的那个名字。
  另外，这里使用的是localhost，所以必须在TF-Serving运行的同一台机器上执行该命令。
  返回：
  + {% asset_img 2.jpg 说明 %}
+ 通过REST API查看模型的元数据
  + **curl http://localhost:8501/v1/models/simple_fc_nn/metadata**
+ 通过Apache ab对TF-Serving进行性能测试 post.txt 里放的是输入数据,(转成json的格式)
  + **ab -n 100000 -c 50 -T 'Content-Type:application/json' -p ./post.txt http://localhost:8501/v1/models/simple_fc_nn:predict**



### keras 模型 转到 tf-serving 可用的pb格式

+ 直接看代码吧

```python
def keras_model_2_tf_serving():
    '''
	    keras model会自动保存为pb格式
	'''
    def export_model(model,export_model_dir,model_version):
        """
        :param export_model_dir: type string, save dir for exported model    url
        :param model_version: type int best
        :return:no return
        """
        with tf.get_default_graph().as_default():
            # prediction_signature
            tensor_info_input_0 = tf.saved_model.utils.build_tensor_info(model.input[0])
            tensor_info_input_1 = tf.saved_model.utils.build_tensor_info(model.input[1])
            tensor_info_output = tf.saved_model.utils.build_tensor_info(model.output)
            prediction_signature = (
                tf.saved_model.signature_def_utils.build_signature_def(
                    inputs ={'input_0': tensor_info_input_0,'input_1': tensor_info_input_1}, # Tensorflow.TensorInfo
                    outputs={'result': tensor_info_output},
                    #method_name=tf.saved_model.signature_constants.PREDICT_METHOD_NAME)
                    method_name= "tensorflow/serving/predict")
                
            )
            print('step1 => prediction_signature created successfully')
            # set-up a builder
            os.mkdir(export_model_dir)
            export_path_base = export_model_dir
            export_path = os.path.join(
                tf.compat.as_bytes(export_path_base),
                tf.compat.as_bytes(str(model_version)))
            builder = tf.saved_model.builder.SavedModelBuilder(export_path)
            builder.add_meta_graph_and_variables(
                # tags:SERVING,TRAINING,EVAL,GPU,TPU
                sess=K.get_session(),
                tags=[tf.saved_model.tag_constants.SERVING],
                clear_devices=True,
                signature_def_map={
                    'predict':prediction_signature,
                    tf.saved_model.signature_constants.DEFAULT_SERVING_SIGNATURE_DEF_KEY: prediction_signature,

                },
                )
            print('step2 => Export path(%s) ready to export trained model' % export_path, '\n starting to export model...')
            #builder.save(as_text=True)
            builder.save()
            print('Done exporting!')


    with open('model_config_classification.json') as json_file:
        json_config = json_file.read()
    model = keras.models.model_from_json(json_config)
    model.load_weights('model_weights_classification.h5')

    # 导出tf-serving的方式
    export_model(model,"tf-serving-model",1)

# 通过上面的转换,可以用tf-serving启起来
'''
TESTDATA="$(pwd)/tf-serving-model"
docker run -t --rm -p 8503:8501 \
    -v "$TESTDATA:/models/bert" \
    -e MODEL_NAME=bert \
    tensorflow/serving
'''
    
def predict_tf_serving():
    sent = "来玩英雄联盟"
    tokenid_train =  tokenizer.encode(sent,maxlen=200)[0] 
    sen_id_train =   tokenizer.encode(sent,maxlen=200)[1]
    SERVER_URL = "http://localhost:8503/v1/models/bert:predict"
    predict_request='{"signature_name": "predict", "instances":[{"input_0":%s,"input_1":%s}] }' %(tokenid_train,sen_id_train)
    response = requests.post(SERVER_URL, data=predict_request)
    res = json.loads(response.content)
    print(res)
```

+ 大概就是这么一个样子吧,看懂就👌

