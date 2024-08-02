---
date: 2019-09-03 20:07:00
status: public
categories:
 - deep learning
tags: 
 - 深度学习
title: Tensorflow example 和 TFRecord
---



## 生成tf.example

```python
import tensorflow as tf
import numpy as np

def generate_example(feature0, feature1, feature2, feature3):
  """
  Creates a tf.Example message ready to be written to a file.
  """

  # Create a dictionary mapping the feature name to the tf.Example-compatible
  # data type.

  feature = {
      'feature0': _int64_feature(feature0),
      'feature1': _int64_feature(feature1),
      'feature2': _bytes_feature(feature2),
      'feature3': _float_feature(feature3),
  }

  # Create a Features message using tf.train.Example.

  example_proto = tf.train.Example(features=tf.train.Features(feature=feature))
  return example_proto


def _bytes_feature(value):
  """Returns a bytes_list from a string / byte."""
  return tf.train.Feature(bytes_list=tf.train.BytesList(value=[value]))

def _float_feature(value):
  """Returns a float_list from a float / double."""
  return tf.train.Feature(float_list=tf.train.FloatList(value=[value]))

def _int64_feature(value):
  """Returns an int64_list from a bool / enum / int / uint."""
  return tf.train.Feature(int64_list=tf.train.Int64List(value=[value]))


# the number of observations in the dataset
n_observations = int(1e4)

# boolean feature, encoded as False or True
feature0 = np.random.choice([False, True], n_observations)

# integer feature, random from 0 .. 4
feature1 = np.random.randint(0, 5, n_observations)

# string feature
strings = np.array([b'cat', b'dog', b'chicken', b'horse', b'goat'])
feature2 = strings[feature1]

# float feature, from a standard normal distribution
feature3 = np.random.randn(n_observations)

example = generate_example(False, 4, b'goat', 0.9876)
example
```

```
features {
  feature {
    key: "feature0"
    value {
      int64_list {
        value: 0
      }
    }
  }
  feature {
    key: "feature1"
    value {
      int64_list {
        value: 4
      }
    }
  }
  feature {
    key: "feature2"
    value {
      bytes_list {
        value: "goat"
      }
    }
  }
  feature {
    key: "feature3"
    value {
      float_list {
        value: 0.9876000285148621
      }
    }
  }
}
```



## 序列化tf.example

```python
example_observation = []
serialized_example = example_proto.SerializeToString(example)
serialized_example
```



```
b'\nR\n\x11\n\x08feature0\x12\x05\x1a\x03\n\x01\x00\n\x11\n\x08feature1\x12\x05\x1a\x03\n\x01\x04\n\x14\n\x08feature2\x12\x08\n\x06\n\x04goat\n\x14\n\x08feature3\x12\x08\x12\x06\n\x04[\xd3|?'
```



## 反序列化tf.example

```
example_proto = tf.train.Example.FromString(serialized_example)
print(example_proto)
print(type(example_proto))
```



```
features {
  feature {
    key: "feature0"
    value {
      int64_list {
        value: 0
      }
    }
  }
  feature {
    key: "feature1"
    value {
      int64_list {
        value: 4
      }
    }
  }
  feature {
    key: "feature2"
    value {
      bytes_list {
        value: "goat"
      }
    }
  }
  feature {
    key: "feature3"
    value {
      float_list {
        value: 0.9876000285148621
      }
    }
  }
}

<class 'tensorflow.core.example.example_pb2.Example'>
```



## 使用TFRecord写

```python
with tf.python_io.TFRecordWriter('tfrecords.data') as writer:
  for i in range(3):
    writer.write(serialized_example)
```



## 使用TFRecord读

```python
iterator = raw_dataset.make_one_shot_iterator()
next_element = iterator.get_next()
with tf.Session() as sess:
    while True:
        try:
            value = sess.run(next_element)
            print(value)
        except tf.errors.OutOfRangeError:
            break
```



```
b'\nR\n\x11\n\x08feature0\x12\x05\x1a\x03\n\x01\x00\n\x11\n\x08feature1\x12\x05\x1a\x03\n\x01\x04\n\x14\n\x08feature2\x12\x08\n\x06\n\x04goat\n\x14\n\x08feature3\x12\x08\x12\x06\n\x04[\xd3|?'
b'\nR\n\x11\n\x08feature0\x12\x05\x1a\x03\n\x01\x00\n\x11\n\x08feature1\x12\x05\x1a\x03\n\x01\x04\n\x14\n\x08feature2\x12\x08\n\x06\n\x04goat\n\x14\n\x08feature3\x12\x08\x12\x06\n\x04[\xd3|?'
b'\nR\n\x11\n\x08feature0\x12\x05\x1a\x03\n\x01\x00\n\x11\n\x08feature1\x12\x05\x1a\x03\n\x01\x04\n\x14\n\x08feature2\x12\x08\n\x06\n\x04goat\n\x14\n\x08feature3\x12\x08\x12\x06\n\x04[\xd3|?'
```



### 解析成 tf.example

```python
iterator = raw_dataset.make_one_shot_iterator()
next_element = iterator.get_next()
with tf.Session() as sess:
    while True:
        try:
            value = sess.run(next_element)
            value = tf.train.Example.FromString(value)
            print(value)
        except tf.errors.OutOfRangeError:
            break
```

```
features {
  feature {
    key: "feature0"
    value {
      int64_list {
        value: 0
      }
    }
  }
  feature {
    key: "feature1"
    value {
      int64_list {
        value: 4
      }
    }
  }
  feature {
    key: "feature2"
    value {
      bytes_list {
        value: "goat"
      }
    }
  }
  feature {
    key: "feature3"
    value {
      float_list {
        value: 0.9876000285148621
      }
    }
  }
}

features {
  feature {
    key: "feature0"
    value {
      int64_list {
        value: 0
      }
    }
  }
  feature {
    key: "feature1"
    value {
      int64_list {
        value: 4
      }
    }
  }
  feature {
    key: "feature2"
    value {
      bytes_list {
        value: "goat"
      }
    }
  }
  feature {
    key: "feature3"
    value {
      float_list {
        value: 0.9876000285148621
      }
    }
  }
}

features {
  feature {
    key: "feature0"
    value {
      int64_list {
        value: 0
      }
    }
  }
  feature {
    key: "feature1"
    value {
      int64_list {
        value: 4
      }
    }
  }
  feature {
    key: "feature2"
    value {
      bytes_list {
        value: "goat"
      }
    }
  }
  feature {
    key: "feature3"
    value {
      float_list {
        value: 0.9876000285148621
      }
    }
  }
}
```

### 解析成tensor（特征列）

```python
def _parse_function(serialized_example):
  features = {"feature2":tf.FixedLenFeature((), tf.string)}
  parsed_features = tf.parse_single_example(serialized_example, features)
  return parsed_features

example = _parse_function(serialized_example)

with tf.Session() as sess:
    ret = sess.run(example)
    print(ret)
```

```
{'feature2': b'goat'}
```



