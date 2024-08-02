---
date: 2020-05-09 21:07:00
status: public
categories:
 - deep learning
tags: 
 - bert
 - transformers
title: Transformers的一些迷思
---



#### 通过from_pretrained缓存的模型在哪

 如果调用from_pretrained方法时指定了cache_dir 则保存到cache_dir，

```python
cache_dir = kwargs.pop("cache_dir", None)
```

如果没指定则去通过系统环境变量寻找（"PYTORCH_TRANSFORMERS_CACHE"", "PYTORCH_PRETRAINED_BERT_CACHE"）

```python
os.getenv("PYTORCH_TRANSFORMERS_CACHE", os.getenv("PYTORCH_PRETRAINED_BERT_CACHE", default_cache_path))
```

如果还没找到则设置为pytorch_home下的transformers目录下

```python
from torch.hub import _get_torch_home
torch_cache_home = _get_torch_home()
os.path.join(torch_cache_home, "transformers")
```



####  from_pretrained方法是如何加载模型的

首先判断是否在pretrained_model_archive_map中，然后判断是否为目录或文件，如果都不是则默认为hf_bucket_url

```python
https://s3.amazonaws.com/models.huggingface.co/bert/{pretrained_model_name_or_path}/{pytorch_model.bin/tf_model.h5}
```

pytorch_model.bin或tf_model.h5 通过from_tf判断



#### 不同模型实现from_pretrained的方式

from_pretrained 的根据不同 cls 来实现加载不同模型的差异， 以bert为例， cls -> BertPreTrainedModel；

```python
class BertPreTrainedModel(PreTrainedModel):
    """ An abstract class to handle weights initialization and
        a simple interface for downloading and loading pretrained models.
    """

    config_class = BertConfig
    pretrained_model_archive_map = BERT_PRETRAINED_MODEL_ARCHIVE_MAP
    load_tf_weights = load_tf_weights_in_bert
    base_model_prefix = "bert"

    def _init_weights(self, module):
        """ Initialize the weights """
        if isinstance(module, (nn.Linear, nn.Embedding)):
            # Slightly different from the TF version which uses truncated_normal for initialization
            # cf https://github.com/pytorch/pytorch/pull/5617
            module.weight.data.normal_(mean=0.0, std=self.config.initializer_range)
        elif isinstance(module, BertLayerNorm):
            module.bias.data.zero_()
            module.weight.data.fill_(1.0)
        if isinstance(module, nn.Linear) and module.bias is not None:
            module.bias.data.zero_()
```

