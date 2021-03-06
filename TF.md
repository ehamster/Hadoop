1.Tensorboard
---------------
```python
from __future__ import print_function
import tensorflow as tf
import os

# The default path for saving event files is the same folder of this python file.
tf.app.flags.DEFINE_string(
'log_dir', os.path.dirname(os.path.abspath(__file__)) + '/logs',
'Directory where event logs are written to.')

# Store all elements in FLAG structure!
FLAGS = tf.app.flags.FLAGS
welcome = tf.constant('Welcome to TensorFlow world!')

# Run the session
with tf.Session() as sess:
    writer = tf.summary.FileWriter(os.path.expanduser(FLAGS.log_dir), sess.graph)
    print("output: ", sess.run(welcome))

# Closing the writer.
writer.close()
sess.close()
```

The Tensorboard can be run as follows in the terminal:
--------------------

tensorboard --logdir="absolute/path/to/log_dir"

2.
----------------
```python
  x = tf.add(a,b,name = 'add')
  with tf.Session() as sess:
    print(sess.run(x))
```
    
3.Variables
----------------
先define再initialize才行
```python
import tensorflow as tf
from tensorflow.python.framework import ops

weights = tf.Variable(tf.random_normal([2,3], stddev = 0.1), name = "weights")
biases = tf.Variable(tf.zeros([3]),name = "biases")

  #1）initial特定的variable：
       variable_c = [weights, biases]
       init_all_op = tf.variables_initializer(var_list = variable_c)
       
  #2)initial所有variables:
       init_all_op = tf.global_variables_initializer()
       
       
  #3)利用已经存在的变量
  wNew = tf.Variable(weights.initialized_value(), name = "wNew")
  init_weightsNew+op = tf.variables_initializer(var_list = [wNew])
```

