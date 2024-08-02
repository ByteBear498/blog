---
date: 2020-07-16 21:07:00
status: public
categories:
 - deep learning
tags: 
 - bert
 - transformers
 - tensorflow hub
title: 通过Tensorflow2使用Bert预训练模型的两种方式
---


以中文Bert为例



下面的例子均以中文Bert预训练模型为例

#### 方式1：使用Tensorflow Hub

```python
import tensorflow as tf
import tensorflow_hub as hub

hub_url_or_local_path = "https://tfhub.dev/tensorflow/bert_zh_L-12_H-768_A-12/2"

# 或者将模型下载到本地
# !wget "https://storage.googleapis.com/tfhub-modules/tensorflow/bert_zh_L-12_H-768_A-12/2.tar.gz"
# !tar -xzvf 2.tar.gz -C bert_zh_L-12_H-768_A-12
# hub_url_or_local_path = "./bert_zh_L-12_H-768_A-12"


def build_model():
  input_word_ids = tf.keras.layers.Input(shape=(max_seq_length,), dtype=tf.int32, name="input_word_ids")
  input_mask = tf.keras.layers.Input(shape=(max_seq_length,), dtype=tf.int32, name="input_mask")
  segment_ids = tf.keras.layers.Input(shape=(max_seq_length,), dtype=tf.int32, name="segment_ids")
  bert_layer = hub.KerasLayer(hub_url_or_local_path, name='bert', trainable=True)
  pooled_output, sequence_output = bert_layer([input_word_ids, input_mask, segment_ids])
  model = tf.keras.Model(inputs=[input_word_ids, input_mask, segment_ids], outputs=sequence_output)
  return model

model = build_model()
model.summary()
```

使用这种方式的时候在执行build_model时bert的参数已经被加载进来了。





#### 方式2：使用Transformers

通过transformers使用bert模型的方式不只这一种，下面的方式是我个人比较喜欢的一种方式（尽量使用Tensorflow原生方式）。
```python
import tensorflow as tf
from transformers import TFBertMainLayer

def build_model(cls_num):
    input_ids = tf.keras.layers.Input(shape=[None, ], dtype=tf.int32, name='input_ids')
    attention_mask = tf.keras.layers.Input(shape=[None, ], dtype=tf.int32, name='attention_mask')
    token_type_ids = tf.keras.layers.Input(shape=[None, ], dtype=tf.int32, name='token_type_ids')
    bert = TFBertMainLayer(bert_config, name='bert')
    bert.trainable = True
    outputs = bert(input_ids, attention_mask, token_type_ids)
    pool = outputs[1]
    model = tf.keras.Model(inputs=[input_ids, attention_mask, token_type_ids], outputs=pool)
    return model

bert_config = BertConfig.from_pretrained("bert-base-chinese")
model = build_model(bert_config=bert_config)
model.load_weights(pre_train_save_file, by_name=True)
```
使用这种方式的时候build_model时，bert的参数还没有加载，需要通过load_weigths方式将参数加载进来。