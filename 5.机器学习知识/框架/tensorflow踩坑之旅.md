坑1：报错： ***OP_REQUIRES failed at example_parsing_ops.cc:240 : Invalid argument: Key: std_baidu_tag_features.  Can't parse serialized Example.***

原因1： 如果config中配置的特征维数不对，会导致出这个错。

原因2 :   如果tfrecord中有些字段为null值，也会导致出这个错，需要设置default value，参照如下代码

```python
fea_len = 1000
feature_name = "std_baidu_tag_features"
self._column_dict[feature_name] = tf.FixedLenFeature([fea_len], tf.float32, default_value=[0.0] * fea_len)
```

