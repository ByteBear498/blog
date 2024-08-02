---
date: 2019-12-12 08:07:00
status: public
categories:
 - deep learning
tags: 
 - 深度学习 
 - tensorflow2.0 
 - bert
title: Tensorflow2.0 基于BERT的开发实战（huggingface-transformers）
---



tensorflow2.0 刚刚发布正式版，网上关于tf2.0使用bert的文章较少。之前想tf2.0做基于bert的NER任务，想了不少资料踩了许多坑，因此将期间的过程总结成文。

在tf2.0上使用bert，主要的难点还是：自己写tf2.0代码适配google的官方源码的方案工作量大对tf熟练度的要求比较高。因此选择了第三方的实现，这里使用的是huggingface的transformers。

除了transformers，其它兼容tf2.0的bert项目还有：

1. keras-bert（Star:1.3k, Kashgari在使用）, 现在也开始兼容tf2.0了，但它只支持bert一种预训练模型
   
2. bert-for-tf2（Star:280），缺点是不是很正规，只给了tf2.0 pipeline示例


在tf2.0正式版发布前后huggingface的transformers也发布了transformers2.0，开始支持tf.2.0的各个预训练模型，虽然没有对pytorch支持的那么全面但在我们的场景已经足够适用了。



### 环境

> tensorflow版本：2.0.0

> transformers版本：2.2.1



### 构建模型

```python
class BertNerModel(TFBertPreTrainedModel):
    def __init__(self, config, *inputs, **kwargs):
      super(BERT_NER, self).__init__(config, *inputs, **kwargs)
      self.bert_layer = TFBertMainLayer(config, name='bert')
      self.bert_layer.trainable = False
      self.concat_layer = tf.keras.layers.Concatenate(name='concat_bert')
    
    def call(self, inputs):
      outputs = self.bert_layer(inputs)
      #将后n层的结果相连
      tensor = self.concat_layer(list(outputs[2][-4:]))
```

这里给出的是简要的代码，可以自行根据任务在`bert_layer`之后加入`RNN`等

自定义模型的写法可以参考官方源码里的`TFBertForSequenceClassification`， 继承`TFBertPreTrainedModel`

`self.bert_layer(inputs)`的返回值为`tuple`类型：

1. 最后1层隐藏层的输出值，`shape=(batch_size, max_length, hidden_dimention)`
2. `[CLS]` 对应的输出值，`shape=(batch_size, hidden_dimention)`
3. 只有设置了`config.output_hidden_states = True`，才有该值，所有隐藏层的输出值，返回值类型是`list` 每个`list`里的值的`shape`是`(batch_size, max_length, hidden_dimention)``



### 模型的初始化

```python
bert_ner_model = BertNerModel.from_pretrained("bert-base-chinese", output_hidden_states=True)
```

因为是模型继承的`TFBertPreTrainedModel`因此这里初始化使用的父类的方式。第一个参数是要加载预训练好的模型参数



### 踩过的坑
1. 通过设置：`self.bert.trainable = False`， 模型可以更快收敛，减少训练时间
2. 预测的时候，输出的数据一定要与`max_length`一致，否则效果完全不可用，猜测可能是我们只给了， 没有给`input_mask`，有看到transformers的源码，如果不给`attention_mask `，默认全是1的
3. 通过设置 `output_hidden_states=True `， 可以得到隐藏层的结果



### 使用huggingface transformers 的一些技巧

#### 使用google原始预训练模型

1、先通过`transformers `命令将原始google预训练的模型文件转换成pytorch格式。这个命令在安装transformers时会回到环境变量中。

```shell
transformers bert \
  chinese_L-12_H-768_A-12/bert_model.ckpt \
  chinese_L-12_H-768_A-12/bert_config.json \
  chinese_L-12_H-768_A-12/pytorch_model.bin
```

output:

```
...
Save PyTorch model to chinese_L-12_H-768_A-12/pytorch_model.bin
```

2、加载转换后的模型

```python
# 加载模型
config = BertConfig.from_json_file('chinese_L-12_H-768_A-12/bert_config.json')
model = TFBertModel.from_pretrained('chinese_L-12_H-768_A-12/',from_pt=True, config=config)
```



在开源代码库下面有好多有关转换的py文件，应该是该功能的源码。例如：`convert_bert_original_tf_checkpoint_to_pytorch.py`、`convert_pytorch_checkpoint_to_tf2.py`

那么，确认下该命令的小源码，在`setup.py`中可以找到如下：

```json
entry_points={
    'console_scripts': [
        "transformers=transformers.__main__:main",
    ]
}
```

可以看到实际执行的`__main__`文件的`main`方法，可以找到源码：

```python
if sys.argv[1] == "bert":
	try:
		from .convert_bert_original_tf_checkpoint_to_pytorch import convert_tf_checkpoint_to_pytorch
	except ImportError:
		print("transformers can only be used from the commandline to convert TensorFlow models in PyTorch, "
			"In that case, it requires TensorFlow to be installed. Please see "
			"https://www.tensorflow.org/install/ for installation instructions.")
		raise

	if len(sys.argv) != 5:
		# pylint: disable=line-too-long
		print("Should be used as `transformers bert TF_CHECKPOINT TF_CONFIG PYTORCH_DUMP_OUTPUT`")
	else:
		PYTORCH_DUMP_OUTPUT = sys.argv.pop()
		TF_CONFIG = sys.argv.pop()
		TF_CHECKPOINT = sys.argv.pop()
		convert_tf_checkpoint_to_pytorch(TF_CHECKPOINT, TF_CONFIG, PYTORCH_DUMP_OUTPUT)
```



 

---



更新：

* 2020-01-17

  huggingface 已经将之前的`transformers` 命令改为 `transformers-cli` 根据 `setup.py` 中 `scripts=["transformers-cli"]` 找到`transformers-cli`文件；修改后转换模型的命令为：

  ```
  # 将tensorflow checkpoint 转换成 pytorch
  !transformers-cli convert --model_type bert \
    --tf_checkpoint  chinese_L-12_H-768_A-12/bert_model.ckpt \
    --config  chinese_L-12_H-768_A-12/bert_config.json \
    --pytorch_dump_output chinese_L-12_H-768_A-12/pytorch_model.bin
  ```

  ps : 话说这改的有点勤啊， 还不如直接找源码执行（`transformers/commands/convert.py ` ）：
  
  ```python
  from transformers.convert_bert_original_tf_checkpoint_to_pytorch import (
                      convert_tf_checkpoint_to_pytorch,
                  )
  convert_tf_checkpoint_to_pytorch(tf_checkpoint, config, pytorch_dump_output)
  ```
  
  

