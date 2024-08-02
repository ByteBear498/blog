---
date: 2019-08-20 20:07:00
status: public
categories:
 - deep learning
tags: 
 - 深度学习
title: Tensorflow 模型的保存和恢复
---

tensorflow模型的保存和恢复方法基本上可以参考其官方文档： https://www.tensorflow.org/guide/saved_model?hl=zh-cn
本文讨论 自定义estimato保存恢复方法r 和 fine-tune 的参数恢复方法



#### 自定义estimator的保存和恢复

```python
run_config = tf.estimator.RunConfig(model_dir=args.output_dir, save_summary_steps=500, save_checkpoints_steps=500, session_config=session_config)
estimator = tf.estimator.Estimator(
    model_fn,
    params=params,
    config=run_config)
```

在train时会根据参数自动读取和保存模型
如果checkpoint 中的状态与描述的模型不兼容，因此重新训练失败并出现以下错误：

```
...
InvalidArgumentError (see above for traceback): tensor_name =
dnn/hiddenlayer_1/bias/t_0/Adagrad; shape in shape_and_slice spec [10]
does not match the shape stored in checkpoint: [20]
```



#### fine-tune 参数恢复方法

```python
tvars = tf.trainable_variables()
(assignment_map, initialized_variable_names) = modeling.get_assignment_map_from_checkpoint(tvars, init_checkpoint)
tf.train.init_from_checkpoint(init_checkpoint, assignment_map)
graph = tf.get_default_graph()
test_varibal = graph.get_tensor_by_name('bert/encoder/layer_9/output/dense/bias:0')
with tf.Session() as sess:
    # 最后初始化变量
    sess.run(tf.global_variables_initializer())
    print(sess.run(test_varibal))
```

在session run tf.global_variables_initializer() 时 会将tf.train.init_from_checkpoint处理的参数进行恢复，这种方法应该只会恢复指定名称的参数，不强制要求模型结构一致。