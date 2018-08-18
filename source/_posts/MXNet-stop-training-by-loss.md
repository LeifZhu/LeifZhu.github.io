
使用MXNet训练Alexnet，如何将训练停止条件设置为loss?

官方提供的example[[1]](https://github.com/apache/incubator-mxnet/tree/master/example/image-classification)里面使用mx.mod.Model.fit()来训练网络，但是检查api介绍[[2]](https://mxnet.apache.org/api/python/module/module.html#mxnet.module.BaseModule.fit)我们可以发现，该方法只能通过num_epochs控制训练终止。[[3]](https://discuss.mxnet.io/t/stopping-crioterion/249)推荐使用gluon API自定义训练循环。






