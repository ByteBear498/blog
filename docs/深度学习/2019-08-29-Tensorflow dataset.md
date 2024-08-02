---
date: 2019-08-29 20:07:00
status: public
categories:
 - deep learning
tags: 
 - 深度学习
title: Tensorflow dataset
---



tf.data API 在 TensorFlow 中引入了两个新的抽象类：
* tf.data.Dataset 表示一系列元素，其中每个元素包含一个或多个 Tensor 对象。例如，在图像管道中，元素可能是单个训练样本，具有一对表示图像数据和标签的张量。可以通过两种不同的方式来创建数据集：
	* 创建来源（例如 Dataset.from_tensor_slices()），以通过一个或多个 tf.Tensor 对象构建数据集。
	* 应用转换（例如 Dataset.batch()），以通过一个或多个 tf.data.Dataset 对象构建数据集。
* tf.data.Iterator 提供了从数据集中提取元素的主要方法。Iterator.get_next() 返回的操作会在执行时生成 Dataset 的下一个元素，并且此操作通常充当输入管道代码和模型之间的接口。最简单的迭代器是“单次迭代器”，它与特定的 Dataset 相关联，并对其进行一次迭代。要实现更复杂的用途，您可以通过 Iterator.initializer 操作使用不同的数据集重新初始化和参数化迭代器，这样一来，您就可以在同一个程序中对训练和验证数据进行多次迭代（举例而言）。



### 1、从dataset中获取数据

### 1.1 one_shot_iterator

```python
import tensorflow as tf
features = [[1,1,2,2], [3,3,4,4], [5,5,6,6]] # [1,1,2,2] 是一个sample ，1 1 2 2 是四个特征值
labels = [0, 1, 1] # 每个sample的标签值
dataset = tf.data.Dataset.from_tensor_slices((features, labels))

iterator = dataset.make_one_shot_iterator()
next_element = iterator.get_next()
with tf.Session() as sess:
    while True:
        try:
            value = sess.run(next_element)
            print(value)
        except tf.errors.OutOfRangeError:
            break
```

output:

```
(array([1, 1, 2, 2], dtype=int32), 0)
(array([3, 3, 4, 4], dtype=int32), 1)
(array([5, 5, 6, 6], dtype=int32), 1)
```



配合模型代替feed

```python
import tensorflow as tf
features = [[1,1,2,2], [3,3,4,4], [5,5,6,6]] # [1,1,2,2] 是一个sample ，1 1 2 2 是四个特征值
labels = [0, 1, 1] # 每个sample的标签值
dataset = tf.data.Dataset.from_tensor_slices((features, labels))

iterator = dataset.make_one_shot_iterator()
next_element = iterator.get_next()

result = next_element[0] + 2

with tf.Session() as sess:
    while True:
        try:
            value = sess.run(result)
            print(value)
        except tf.errors.OutOfRangeError:
            break
```



```
[3 3 4 4]
[5 5 6 6]
[7 7 8 8]
```



### 1.2 initializable_iterator

多个epoch读取数据

```python
iterator = dataset.make_initializable_iterator()
next_element = iterator.get_next()

with tf.Session() as sess:
    for epoch in range(3):
        sess.run(iterator.initializer)
        print('epoch : %d' % epoch)
        while True:
            try:
                value = sess.run(next_element)
                print(value)
            except tf.errors.OutOfRangeError:
                break
```

output:

```
epoch : 0
(array([1, 1, 2, 2], dtype=int32), 0)
(array([5, 5, 6, 6], dtype=int32), 1)
(array([3, 3, 4, 4], dtype=int32), 1)
epoch : 1
(array([5, 5, 6, 6], dtype=int32), 1)
(array([1, 1, 2, 2], dtype=int32), 0)
(array([3, 3, 4, 4], dtype=int32), 1)
epoch : 2
(array([1, 1, 2, 2], dtype=int32), 0)
(array([5, 5, 6, 6], dtype=int32), 1)
(array([3, 3, 4, 4], dtype=int32), 1)
```



### 1.3 feedale iterator

```python
import tensorflow as tf
features_data = {}
features_data['input_ids'] =  [[1,1,2,2], [3,3,4,4], [5,5,6,6], [7,7,8,8]]
features_data['input_mask'] =  [[1,1,2,2], [3,3,4,4], [5,5,6,6], [7,7,8,8]]
labels_data = [0, 1, 1, 0] # 每个sample的标签值
data_set = tf.data.Dataset.from_tensor_slices((features_data, labels_data))

handle = tf.placeholder(tf.string, shape=[])
features, labels = tf.data.Iterator.from_string_handle(
    handle, data_set.output_types, data_set.output_shapes).get_next()

print(data_set.output_types)
print(data_set.output_shapes)

iterator = data_set.make_one_shot_iterator()
with tf.Session() as sess:
  training_handle = sess.run(iterator.string_handle())
  for _ in range(2):
    ret = sess.run(features, feed_dict={handle: training_handle})
    print('TRAIN:', ret['input_ids'])
```

output:

```
({'input_ids': tf.int32, 'input_mask': tf.int32}, tf.int32)
({'input_ids': TensorShape([Dimension(4)]), 'input_mask': TensorShape([Dimension(4)])}, TensorShape([]))
TRAIN: [1 1 2 2]
TRAIN: [3 3 4 4]
```



### 2、转成成dataset

#### 2.1 从numpy转换成dataset

```python
features = [[1,1,2,2], [3,3,4,4], [5,5,6,6]] # [1,1,2,2] 是一个sample ，1 1 2 2 是四个特征值
labels = [0, 1, 1] # 每个sample的标签值
dataset = tf.data.Dataset.from_tensor_slices((features, labels))
```



#### 2.2 从tfrecor文件转换成dataset

```python
filenames = ["/var/data/file1.tfrecord", "/var/data/file2.tfrecord"]
dataset = tf.data.TFRecordDataset(filenames)
```



#### 2.3 还可以通过dict生成

```python
features = {'feature_1': [[1, 1, 2, 2], [3, 3, 4, 4], [5, 5, 6, 7], [8, 8, 9, 9]],
            'feature_2': [[0, 0, 0, 0], [0, 0, 0, 0], [0, 0, 0, 0], [0, 0, 0, 0]],
            'feature_3': [[1, 1, 1, 1], [1, 1, 1, 1], [1, 1, 1, 1], [1, 1, 1, 1]]}
labels = [0, 1, 1, 0]  # 每个sample的标签值
dataset = tf.data.Dataset.from_tensor_slices((features, labels))

iterator = dataset.make_one_shot_iterator()
next_element = iterator.get_next()

with tf.Session() as sess:
    while True:
        try:
            value = sess.run(next_element)
            print(value)
        except tf.errors.OutOfRangeError:
            break
```

output:

```
({'feature_1': array([1, 1, 2, 2], dtype=int32), 'feature_2': array([0, 0, 0, 0], dtype=int32), 'feature_3': array([1, 1, 1, 1], dtype=int32)}, 0)
({'feature_1': array([3, 3, 4, 4], dtype=int32), 'feature_2': array([0, 0, 0, 0], dtype=int32), 'feature_3': array([1, 1, 1, 1], dtype=int32)}, 1)
({'feature_1': array([5, 5, 6, 7], dtype=int32), 'feature_2': array([0, 0, 0, 0], dtype=int32), 'feature_3': array([1, 1, 1, 1], dtype=int32)}, 1)
({'feature_1': array([8, 8, 9, 9], dtype=int32), 'feature_2': array([0, 0, 0, 0], dtype=int32), 'feature_3': array([1, 1, 1, 1], dtype=int32)}, 0)
```



### 3、dataset 相关方法

#### 3.1 batch

```python
features = [[1,1,2,2], [3,3,4,4], [5,5,6,6], [7,7,8,8]] # [1,1,2,2] 是一个sample ，1 1 2 2 是四个特征值
labels = [0, 1, 1, 0] # 每个sample的标签值
dataset = tf.data.Dataset.from_tensor_slices((features, labels))

batched_dataset = dataset.batch(3)
iterator = batched_dataset.make_one_shot_iterator()
next_element = iterator.get_next()

# 如果不够 batchsize 就只返回剩余的
with tf.Session() as sess:
    while True:
        try:
            value = sess.run(next_element)
            print(value)
        except tf.errors.OutOfRangeError:
            break
```

output:

```
(array([[1, 1, 2, 2],
       [3, 3, 4, 4],
       [5, 5, 6, 6]], dtype=int32), array([0, 1, 1], dtype=int32))
(array([[7, 7, 8, 8]], dtype=int32), array([0], dtype=int32))
```





#### 3.2 repeat

```python
features = [[1,1,2,2], [3,3,4,4], [5,5,6,6], [7,7,8,8]] # [1,1,2,2] 是一个sample ，1 1 2 2 是四个特征值
labels = [0, 1, 1, 0] # 每个sample的标签值
dataset = tf.data.Dataset.from_tensor_slices((features, labels))
dataset = dataset.repeat(2)

iterator = dataset.make_one_shot_iterator()
next_element = iterator.get_next()


with tf.Session() as sess:
    while True:
        try:
            value = sess.run(next_element)
            print(value)
        except tf.errors.OutOfRangeError:
            break
```

output:

```
(array([1, 1, 2, 2], dtype=int32), 0)
(array([3, 3, 4, 4], dtype=int32), 1)
(array([5, 5, 6, 6], dtype=int32), 1)
(array([7, 7, 8, 8], dtype=int32), 0)
(array([1, 1, 2, 2], dtype=int32), 0)
(array([3, 3, 4, 4], dtype=int32), 1)
(array([5, 5, 6, 6], dtype=int32), 1)
(array([7, 7, 8, 8], dtype=int32), 0)
```



#### 3.3 repeat + batch

```python
features = [[1,1,2,2], [3,3,4,4], [5,5,6,6], [7,7,8,8]] # [1,1,2,2] 是一个sample ，1 1 2 2 是四个特征值
labels = [0, 1, 1, 0] # 每个sample的标签值
dataset = tf.data.Dataset.from_tensor_slices((features, labels))
dataset = dataset.repeat(2)


batched_dataset = dataset.batch(3)
iterator = batched_dataset.make_one_shot_iterator()
next_element = iterator.get_next()

with tf.Session() as sess:
    while True:
        try:
            value = sess.run(next_element)
            print(value)
        except tf.errors.OutOfRangeError:
            break
```

output:

```
(array([[1, 1, 2, 2],
       [3, 3, 4, 4],
       [5, 5, 6, 6]], dtype=int32), array([0, 1, 1], dtype=int32))
(array([[7, 7, 8, 8],
       [1, 1, 2, 2],
       [3, 3, 4, 4]], dtype=int32), array([0, 0, 1], dtype=int32))
(array([[5, 5, 6, 6],
       [7, 7, 8, 8]], dtype=int32), array([1, 0], dtype=int32))
```





#### 3.4 shuffle

> Dataset.shuffle() 转换会使用类似于 tf.RandomShuffleQueue 的算法随机重排输入数据集：它会维持一个固定大小的缓冲区，并从该缓冲区统一地随机选择下一个元素。

```python
features = [[1,1,2,2], [3,3,4,4], [5,5,6,6], [7,7,8,8]] # [1,1,2,2] 是一个sample ，1 1 2 2 是四个特征值
labels = [0, 1, 1, 0] # 每个sample的标签值
dataset = tf.data.Dataset.from_tensor_slices((features, labels))
dataset = dataset.shuffle(buffer_size=100)

iterator = dataset.make_one_shot_iterator()
next_element = iterator.get_next()


with tf.Session() as sess:
    while True:
        try:
            value = sess.run(next_element)
            print(value)
        except tf.errors.OutOfRangeError:
            break
```

output:

```
(array([7, 7, 8, 8], dtype=int32), 0)
(array([3, 3, 4, 4], dtype=int32), 1)
(array([5, 5, 6, 6], dtype=int32), 1)
(array([1, 1, 2, 2], dtype=int32), 0)
```

