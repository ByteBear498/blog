---
date: 2019-11-22 11:07:00
status: public
categories:
 - deep learning
tags: 
 - 深度学习 
 - tensorflow2.0 
 - bert 
 - tra
title: Tensorflow2.0使用bert：transformers与kashgaria
---

**背景：**最近打算做NER任务，打算使用kashgaria当baseline，然后使用transformers来做模型开发。

**问题：**使用kashgaria来做Baseline，使用人民日报语料可以得到99.99%的准确率，但是使用transformers的TFBertForTokenClassification来做的话只能得到75%左右的效果， 看了两个项目的源码开始怀疑是模型结构的问题，因此使用transformers模拟kashgaria的结果又做了一次，效果还是在75%左右。相关代码如下：

kashgaria代码：
```python
from kashgari.corpus import ChineseDailyNerCorpus
from kashgari.tasks.labeling import BiLSTM_Model
from kashgari.embeddings import BERTEmbedding
import kashgari

train_x, train_y = ChineseDailyNerCorpus.load_data('train')
test_x, test_y = ChineseDailyNerCorpus.load_data('test')
valid_x, valid_y = ChineseDailyNerCorpus.load_data('valid')

bert_embed = BERTEmbedding('chinese_L-12_H-768_A-12',
                           task=kashgari.LABELING,
                           sequence_length=100)
model = BiLSTM_Model(bert_embed)
model.fit(train_x, train_y, valid_x, valid_y, epochs=1)

```

transformers代码：
```python
# 数据的处理省略，但使用的同样语料
model = TFBertForTokenClassification.from_pretrained('bert-base-chinese')
model.compile(loss='categorical_crossentropy', metrics=['accuracy'], optimizer='adam')
model.fit(all_input_ids, tf.constant(all_label_ids_ca, dtype=tf.float32), epochs=2, batch_size=32)
```



transformers 模拟kashgaria代码：

```python
class BERT_NER(TFBertPreTrainedModel):
  def __init__(self, config, *inputs, **kwargs):
      super(BERT_NER, self).__init__(config, *inputs, **kwargs)
      self.num_labels = config.num_labels
      self.bert = TFBertMainLayer(config, name='bert')      
      self.layer_blstm = tf.keras.layers.Bidirectional(tf.keras.layers.LSTM(units=128, return_sequences=True),name='layer_blstm')

      self.layer_dropout = tf.keras.layers.Dropout(0.4,name='layer_dropout')

      self.layer_time_distributed = tf.keras.layers.TimeDistributed(tf.keras.layers.Dense(8), name='layer_time_distributed')
      self.layer_activation = tf.keras.layers.Activation('softmax')

  def call(self, inputs, **kwargs):
      output_dim = 8    
      outputs = self.bert(inputs)
      sequence_output = outputs[0]
      tensor = self.layer_blstm(sequence_output)
      tensor = self.layer_dropout(tensor)
      tensor = self.layer_time_distributed(tensor)
      output_tensor = self.layer_activation(tensor)

      return output_tensor

model = BERT_NER.from_pretrained("bert-base-chinese")
model.compile(loss='categorical_crossentropy', metrics=['accuracy'], optimizer='adam')
model.fit(all_input_ids, tf.constant(all_label_ids_ca, dtype=tf.float32), epochs=2, batch_size=32)
```



**直接上结论：**

kashgari的BERTEmbedding默认是非fine-tune，而transformers默认是fine-tune， 也就是说kashgari默认bert的参数是不参加训练的。但效果却比bert参数参加训练的transformers，猜测可能是2个原因：bert模型参数参加训练那么训练的参数过多，在语料比较小的时候并且训练时间不足的情况下反而无法微调到比较过合适的值；或者是NER任务类别不均衡，导致学习偏了。



**解决方案：**

在BERT_NER中，`self.bert = TFBertMainLayer(config, name='bert')`下面加上`self.bert.trainable = False`不让bert参数参加训练。另外如果想让kashgari中的bert参数参加训练的话可以在BERTEmbedding初始化时添加参数`trainable=True`



**关于调试：**

在tensorflow2.0中，可以通过`tf_model.trainable_variables`打印可训练的一些变量。

```
<tf.Variable 'layer_blstm_1/forward_lstm_1/kernel:0' shape=(3072, 512) dtype=float32>
<tf.Variable 'layer_blstm_1/forward_lstm_1/recurrent_kernel:0' shape=(128, 512) dtype=float32>
<tf.Variable 'layer_blstm_1/forward_lstm_1/bias:0' shape=(512,) dtype=float32>
<tf.Variable 'layer_blstm_1/backward_lstm_1/kernel:0' shape=(3072, 512) dtype=float32>
<tf.Variable 'layer_blstm_1/backward_lstm_1/recurrent_kernel:0' shape=(128, 512) dtype=float32>
<tf.Variable 'layer_blstm_1/backward_lstm_1/bias:0' shape=(512,) dtype=float32>
<tf.Variable 'layer_time_distributed_1/kernel:0' shape=(256, 8) dtype=float32>
<tf.Variable 'layer_time_distributed_1/bias:0' shape=(8,) dtype=float32>
```

也可以看到里面具体值，可以观察bert参数是否有加载成功。

```
<tf.Variable 'bert_ner/bert/embeddings/word_embeddings/weight:0' shape=(21128, 768) dtype=float32, numpy=
array([[ 0.01929518,  0.00411201, -0.02079543, ...,  0.09468532,
         0.00853808,  0.00178786],
       [ 0.00211436,  0.02164099,  0.00108996, ...,  0.08090564,
         0.00178312,  0.02494784],
       [ 0.01467745,  0.00050856,  0.00283794, ...,  0.08360939,
         0.01208044,  0.02821462],
       ...,
       [ 0.03456404,  0.00210567,  0.00852101, ...,  0.00853979,
         0.03371229,  0.00985317],
       [ 0.05406349,  0.02890619,  0.02626012, ...,  0.0525924 ,
         0.06508742,  0.03532186],
       [ 0.02002425,  0.00229523, -0.00892451, ...,  0.07987329,
        -0.05615233,  0.02471835]], dtype=float32)>
```





