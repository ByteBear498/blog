---
date: 2020-10-16 21:07:00
status: public
markup: mmark
categories:
 - deep learning
tags: 
 - CRF
 - Tensorflow
title: DeepLearning中CRF的Tensorflow代码实现
---

### 主方法
```python
def crf_log_likelihood(
    inputs: TensorLike,
    tag_indices: TensorLike,
    sequence_lengths: TensorLike,
    transition_params: Optional[TensorLike] = None,
) -> tf.Tensor:
    """Computes the log-likelihood of tag sequences in a CRF.
    Args:
      inputs: A [batch_size, max_seq_len, num_tags] tensor of unary potentials
          to use as input to the CRF layer.
      tag_indices: A [batch_size, max_seq_len] matrix of tag indices for which
          we compute the log-likelihood.
      sequence_lengths: A [batch_size] vector of true sequence lengths.
      transition_params: A [num_tags, num_tags] transition matrix,
          if available.
    Returns:
      log_likelihood: A [batch_size] `Tensor` containing the log-likelihood of
        each example, given the sequence of tag indices.
      transition_params: A [num_tags, num_tags] transition matrix. This is
          either provided by the caller or created in this function.
    """
    num_tags = inputs.shape[2]

    # cast type to handle different types
    tag_indices = tf.cast(tag_indices, dtype=tf.int32)
    sequence_lengths = tf.cast(sequence_lengths, dtype=tf.int32)

    if transition_params is None:
        initializer = tf.keras.initializers.GlorotUniform()
        transition_params = tf.Variable(
            initializer([num_tags, num_tags]), "transitions"
        )

    sequence_scores = crf_sequence_score(
        inputs, tag_indices, sequence_lengths, transition_params
    )
    log_norm = crf_log_norm(inputs, sequence_lengths, transition_params)

    # Normalize the scores to get the log-likelihood per example.
    log_likelihood = sequence_scores - log_norm
    return log_likelihood, transition_params
```



`log_likelihood = sequence_scores - log_norm` 中`sequence_scores `表示**单路径分数**，`log_norm` 表示**所有路径总分数**。在`-log_likelihood `才是我们想要最小化的值（损失函数）。

### `sequence_scores `的计算：

```python
def crf_sequence_score(
    inputs: TensorLike,
    tag_indices: TensorLike,
    sequence_lengths: TensorLike,
    transition_params: TensorLike,
) -> tf.Tensor:
    """Computes the unnormalized score for a tag sequence.
    Args:
      inputs: A [batch_size, max_seq_len, num_tags] tensor of unary potentials
          to use as input to the CRF layer.
      tag_indices: A [batch_size, max_seq_len] matrix of tag indices for which
          we compute the unnormalized score.
      sequence_lengths: A [batch_size] vector of true sequence lengths.
      transition_params: A [num_tags, num_tags] transition matrix.
    Returns:
      sequence_scores: A [batch_size] vector of unnormalized sequence scores.
    """
    tag_indices = tf.cast(tag_indices, dtype=tf.int32)
    sequence_lengths = tf.cast(sequence_lengths, dtype=tf.int32)

    # If max_seq_len is 1, we skip the score calculation and simply gather the
    # unary potentials of the single tag.
    def _single_seq_fn():
        batch_size = tf.shape(inputs, out_type=tf.int32)[0]
        batch_inds = tf.reshape(tf.range(batch_size), [-1, 1])
        indices = tf.concat([batch_inds, tf.zeros_like(batch_inds)], axis=1)

        tag_inds = tf.gather_nd(tag_indices, indices)
        tag_inds = tf.reshape(tag_inds, [-1, 1])
        indices = tf.concat([indices, tag_inds], axis=1)

        sequence_scores = tf.gather_nd(inputs, indices)

        sequence_scores = tf.where(
            tf.less_equal(sequence_lengths, 0),
            tf.zeros_like(sequence_scores),
            sequence_scores,
        )
        return sequence_scores

    def _multi_seq_fn():
        # Compute the scores of the given tag sequence.
        unary_scores = crf_unary_score(tag_indices, sequence_lengths, inputs)
        binary_scores = crf_binary_score(
            tag_indices, sequence_lengths, transition_params
        )
        sequence_scores = unary_scores + binary_scores
        return sequence_scores

    return tf.cond(tf.equal(tf.shape(inputs)[1], 1), _single_seq_fn, _multi_seq_fn)
```



如果`inputs`的lengths是1（即只一个字或词），则使用`_single_seq_fn`计算，否则使用`_multi_seq_fn`，这里主要看`_multi_seq_fn`。

`_multi_seq_fn`是两个部分组成`unary_scores`和`binary_scores`。

`unary_scores`：计算的是**EmissionScore**

`binary_scores`：计算的是**TransitionScore**



```python
def crf_unary_score(
    tag_indices: TensorLike, sequence_lengths: TensorLike, inputs: TensorLike
) -> tf.Tensor:
    """Computes the unary scores of tag sequences.
    Args:
      tag_indices: A [batch_size, max_seq_len] matrix of tag indices.
      sequence_lengths: A [batch_size] vector of true sequence lengths.
      inputs: A [batch_size, max_seq_len, num_tags] tensor of unary potentials.
    Returns:
      unary_scores: A [batch_size] vector of unary scores.
    """
    tag_indices = tf.cast(tag_indices, dtype=tf.int32)
    sequence_lengths = tf.cast(sequence_lengths, dtype=tf.int32)

    batch_size = tf.shape(inputs)[0]
    max_seq_len = tf.shape(inputs)[1]
    num_tags = tf.shape(inputs)[2]

    flattened_inputs = tf.reshape(inputs, [-1])

    offsets = tf.expand_dims(tf.range(batch_size) * max_seq_len * num_tags, 1)
    offsets += tf.expand_dims(tf.range(max_seq_len) * num_tags, 0)
    # Use int32 or int64 based on tag_indices' dtype.
    if tag_indices.dtype == tf.int64:
        offsets = tf.cast(offsets, tf.int64)
    flattened_tag_indices = tf.reshape(offsets + tag_indices, [-1])

    unary_scores = tf.reshape(
        tf.gather(flattened_inputs, flattened_tag_indices), [batch_size, max_seq_len]
    )

    masks = tf.sequence_mask(
        sequence_lengths, maxlen=tf.shape(tag_indices)[1], dtype=tf.float32
    )

    unary_scores = tf.reduce_sum(unary_scores * masks, 1)
    return unary_scores
```

`crf_unary_score`的计算方法是先将`inputs`拉平，然后计算`tag_index`的`flattened_tag_indices`，然后根据`flattened_tag_indices`取到`inputs`中的内容。

```
inputs:
 tf.Tensor(
[[[0.3 0.7]
  [0.2 0.8]
  [0.4 0.6]
  [0.4 0.6]]], shape=(1, 4, 2), dtype=float32)
tag_indices:
 tf.Tensor([[1 0 1 1]], shape=(1, 4), dtype=int32)
offset: 
 tf.Tensor([[0]], shape=(1, 1), dtype=int32)
offset: 
 tf.Tensor([[0 2 4 6]], shape=(1, 4), dtype=int32)
flattened_tag_indices: 
 tf.Tensor([1 2 5 7], shape=(4,), dtype=int32)
unary_scores: 
 tf.Tensor([[0.7 0.2 0.6 0.6]], shape=(1, 4), dtype=float32)
masks: 
 tf.Tensor([[1. 1. 1. 0.]], shape=(1, 4), dtype=float32)
<tf.Tensor: shape=(1,), dtype=float32, numpy=array([1.5], dtype=float32)>
```



```python
def crf_binary_score(
    tag_indices: TensorLike, sequence_lengths: TensorLike, transition_params: TensorLike
) -> tf.Tensor:
    """Computes the binary scores of tag sequences.
    Args:
      tag_indices: A [batch_size, max_seq_len] matrix of tag indices.
      sequence_lengths: A [batch_size] vector of true sequence lengths.
      transition_params: A [num_tags, num_tags] matrix of binary potentials.
    Returns:
      binary_scores: A [batch_size] vector of binary scores.
    """
    tag_indices = tf.cast(tag_indices, dtype=tf.int32)
    sequence_lengths = tf.cast(sequence_lengths, dtype=tf.int32)

    num_tags = tf.shape(transition_params)[0]
    num_transitions = tf.shape(tag_indices)[1] - 1

    # Truncate by one on each side of the sequence to get the start and end
    # indices of each transition.
    start_tag_indices = tf.slice(tag_indices, [0, 0], [-1, num_transitions])
    end_tag_indices = tf.slice(tag_indices, [0, 1], [-1, num_transitions])

    # Encode the indices in a flattened representation.
    flattened_transition_indices = start_tag_indices * num_tags + end_tag_indices
    flattened_transition_params = tf.reshape(transition_params, [-1])

    # Get the binary scores based on the flattened representation.
    binary_scores = tf.gather(flattened_transition_params, flattened_transition_indices)

    masks = tf.sequence_mask(
        sequence_lengths, maxlen=tf.shape(tag_indices)[1], dtype=tf.float32
    )
    truncated_masks = tf.slice(masks, [0, 1], [-1, -1])
    binary_scores = tf.reduce_sum(binary_scores * truncated_masks, 1)
    return binary_scores
```

与`crf_unary_score`类似，`crf_unary_score`是要在`inputs`这个二维矩阵中选值，而`crf_binary_score`要做的是要在同样是二维矩阵的`transition_params`中选值。`start_tag_indices * num_tags`计算的就是`offsets`，即横坐标。



### `crf_log_norm`计算



```python
def crf_log_norm(
    inputs: TensorLike, sequence_lengths: TensorLike, transition_params: TensorLike
) -> tf.Tensor:
    """Computes the normalization for a CRF.
    Args:
      inputs: A [batch_size, max_seq_len, num_tags] tensor of unary potentials
          to use as input to the CRF layer.
      sequence_lengths: A [batch_size] vector of true sequence lengths.
      transition_params: A [num_tags, num_tags] transition matrix.
    Returns:
      log_norm: A [batch_size] vector of normalizers for a CRF.
    """
    sequence_lengths = tf.cast(sequence_lengths, dtype=tf.int32)
    # Split up the first and rest of the inputs in preparation for the forward
    # algorithm.
    first_input = tf.slice(inputs, [0, 0, 0], [-1, 1, -1])
    first_input = tf.squeeze(first_input, [1])

    # If max_seq_len is 1, we skip the algorithm and simply reduce_logsumexp
    # over the "initial state" (the unary potentials).
    def _single_seq_fn():
        log_norm = tf.reduce_logsumexp(first_input, [1])
        # Mask `log_norm` of the sequences with length <= zero.
        log_norm = tf.where(
            tf.less_equal(sequence_lengths, 0), tf.zeros_like(log_norm), log_norm
        )
        return log_norm

    def _multi_seq_fn():
        """Forward computation of alpha values."""
        rest_of_input = tf.slice(inputs, [0, 1, 0], [-1, -1, -1])
        # Compute the alpha values in the forward algorithm in order to get the
        # partition function.

        alphas = crf_forward(
            rest_of_input, first_input, transition_params, sequence_lengths
        )
        log_norm = tf.reduce_logsumexp(alphas, [1])
        # Mask `log_norm` of the sequences with length <= zero.
        log_norm = tf.where(
            tf.less_equal(sequence_lengths, 0), tf.zeros_like(log_norm), log_norm
        )
        return log_norm

    return tf.cond(tf.equal(tf.shape(inputs)[1], 1), _single_seq_fn, _multi_seq_fn)
```



计算是当每个字拿出来当初始值，即第一次的**previous**。



```python
def crf_forward(
    inputs: TensorLike,
    state: TensorLike,
    transition_params: TensorLike,
    sequence_lengths: TensorLike,
) -> tf.Tensor:
    """Computes the alpha values in a linear-chain CRF.
    See http://www.cs.columbia.edu/~mcollins/fb.pdf for reference.
    Args:
      inputs: A [batch_size, num_tags] matrix of unary potentials.
      state: A [batch_size, num_tags] matrix containing the previous alpha
         values.
      transition_params: A [num_tags, num_tags] matrix of binary potentials.
          This matrix is expanded into a [1, num_tags, num_tags] in preparation
          for the broadcast summation occurring within the cell.
      sequence_lengths: A [batch_size] vector of true sequence lengths.
    Returns:
      new_alphas: A [batch_size, num_tags] matrix containing the
          new alpha values.
    """
    sequence_lengths = tf.cast(sequence_lengths, dtype=tf.int32)

    last_index = tf.maximum(
        tf.constant(0, dtype=sequence_lengths.dtype), sequence_lengths - 1
    )
    inputs = tf.transpose(inputs, [1, 0, 2])
    transition_params = tf.expand_dims(transition_params, 0)

    def _scan_fn(_state, _inputs):
        _state = tf.expand_dims(_state, 2)
        transition_scores = _state + transition_params
        new_alphas = _inputs + tf.reduce_logsumexp(transition_scores, [1])
        return new_alphas

    all_alphas = tf.transpose(tf.scan(_scan_fn, inputs, state), [1, 0, 2])
    # add first state for sequences of length 1
    all_alphas = tf.concat([tf.expand_dims(state, 1), all_alphas], 1)

    idxs = tf.stack([tf.range(tf.shape(last_index)[0]), last_index], axis=1)
    return tf.gather_nd(all_alphas, idxs)
```



这里要注意的是`tf.scan`负责遍历， `tf.reduce_logsumexp`表示`log(sum(exp(元素值))`

