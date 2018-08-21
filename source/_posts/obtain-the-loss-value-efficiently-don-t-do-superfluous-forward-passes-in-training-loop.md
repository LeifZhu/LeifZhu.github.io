---
title: >-
  obtain the loss value efficiently: don't do superfluous forward passes in
  training loop
date: 2018-08-21 14:50:40
tags: [deep learning]
---

在利用Tensorflow或CNTK训练DNN的时候，我们可能需要在一次迭代后获得当前的loss或者其他metric。一种常见的方式是：通过一次forward pass计算loss(比如tensorflow的loss.eval())。但是这样带来的额外消耗是没有必要的，因为在训练过程中已经进行过forward pass。
<!--more-->
## 在迭代完成后打印当前的损失值
我有一份用CNTK实现的DNN，它的training loop长这样
```python
...
ce = C.cross_entropy_with_softmax(z, label_var)
pe = C.classification_error(z, label_var)
...
...
# training toop
for epoch in range(1, max_epochs+1):
  sample_count = 0
  while sample_count < epoch_size:
      data = reader_train.next_minibatch(min(minibatch_size, epoch_size - sample_count),
                                          input_map=input_map)
      trainer.train_minibatch(data)
      sample_count += data[label_var].num_samples
  trainer.summarize_training_progress()
...
```
但是我希望每完成一个mini batch的训练我都能及时获得当前的loss，因为我希望：
1. 能够在每次迭代后打印当前的loss值
2. 当loss达到某个给定的target_loss的时候就停止训练

一种很自然的想法就是像下面这样：
```python
...
ce = C.cross_entropy_with_softmax(z, label_var)
pe = C.classification_error(z, label_var)
...
...
# training toop
for epoch in range(1, max_epochs+1):
  sample_count = 0
  while sample_count < epoch_size:
      data = reader_train.next_minibatch(min(minibatch_size, epoch_size - sample_count),
                                          input_map=input_map)
      trainer.train_minibatch(data)
-->   curr_loss = np.asscalar(np.mean(ce.eval(data)))
-->   print("current loss: %f" % (curr_loss))
      sample_count += data[label_var].num_samples
...
```
然而我发现这样会拖慢迭代的速度，在加上这两行之前，训练一个256个examples的mini batch大约是13s， 加上后则需要17~18s，显然问题出在`curr_loss = np.asscalar(np.mean(ce.eval(data)))`这里，进行一次forward pass还是挺耗时的。

## 正确的方式
我们知道在`trainer.train_minibatch(data)`中是进行过forward pass的，那么看看`trainer.train_minibatch()`有没有提供这样的接口，从[API文档[1]](https://www.cntk.ai/pythondocs/cntk.train.trainer.html)中我们可以看到：

{% asset_img api.png cntk.trainer.train_minibatch api %}

注意红色方框标记的内容。简单地说，反正可以通过传入outputs参数输出制定的令`trainer.train_minibatch()`以字典的方式返回请求获取的值。最终代码如下：
```python
...
ce = C.cross_entropy_with_softmax(z, label_var)
pe = C.classification_error(z, label_var)
...
...
# train loop
finished = False
for epoch in range(1, max_epochs+1):
    if finished:
        logging.info("Training finished!")
        break
    sample_count = 0
    batch_cnt = 1
    while sample_count < epoch_size:
        batch_begin_time = time.time()
        data = reader_train.next_minibatch(min(minibatch_size, epoch_size - sample_count),
                                            input_map=input_map)
-->     _, dict_out = trainer.train_minibatch(data, outputs=[ce, pe])
-->     curr_loss = np.asscalar(np.mean(dict_out[ce]))
-->     curr_err_rate = np.asscalar(np.mean(dict_out[pe]))
        logging.info("epoch[%d of %d] - batch[%d] - training loss=%f - training err_rate = %f %% - %f exampls/s"%
                      (epoch, max_epochs, batch_cnt, curr_loss, curr_err_rate*100,
                      data[label_var].num_samples/(time.time()-batch_begin_time)
                      )
                      )
        sample_count += data[label_var].num_samples
        batch_cnt += 1
        if curr_loss < target_loss:
            finished = True
            break
...
```
这样更改之后，我实现了打印loss和用loss控制训练结束，并且训练速度恢复到了约13 s/batch。

**注意** : outputs参数需要传入一个iterable的对象，因此即使你只希望取loss一个值，也应该使用`trainer.train_minibatch(data, outputs=[loss])`而不是`trainer.train_minibatch(data, outputs=loss)`

## Tensorflow中该如何做
同样的问题，在tensorflow中你应该使用[[2]](https://stackoverflow.com/questions/33833818/printing-the-loss-during-tensorflow-training)

```python
for i in range(100):
    batch_xs, batch_ys = mnist.train.next_batch(100)
    cross_entropy = -tf.reduce_sum(y_ * tf.log(y))
    _, loss_val = sess.run([train_step, cross_entropy],
                           feed_dict={x: batch_xs, y_: batch_ys})
    print 'loss = ' + loss_val
```
而不是
```python
for i in range(100):
    batch_xs, batch_ys = mnist.train.next_batch(100)
    cross_entropy = -tf.reduce_sum(y_ * tf.log(y))
    sess.run(train_step,
             feed_dict={x: batch_xs, y_: batch_ys})
    loss_val = sess.run(cross_entropy)
    print 'loss = ' + loss_val
```
下面的例子可以看出两种方式运行时间的差别
{% asset_img run_time.png 运行时间对比 %}
## 参考
[1]https://www.cntk.ai/pythondocs/cntk.train.trainer.html

[2] https://stackoverflow.com/questions/33833818/printing-the-loss-during-tensorflow-training