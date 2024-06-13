# Classify Images with TensorFlow on Google Cloud
## TensorFlow: Qwik Start [GSP637]
### Access Cloud Code
- check required packages in IDE
pip install google-cloud-logging
pip install --upgrade protobuf
pip install --upgrade tensorflow

- check if tensorflow is installed
python -c "import tensorflow;print(tensorflow.__version__)"

### Create your first machine learning model

### Start coding
Application Menu
File
New File

- import necessary packages
```
import logging
import google.cloud.logging as cloud_logging
from google.cloud.logging.handlers import CloudLoggingHandler
from google.cloud.logging_v2.handlers import setup_logging

cloud_logger = logging.getLogger('cloudLogger')
cloud_logger.setLevel(logging.INFO)
cloud_logger.addHandler(CloudLoggingHandler(cloud_logging.Client()))
cloud_logger.addHandler(logging.StreamHandler())

# import TensorFlow
import tensorflow as tf

# import numpy
import numpy as np
```
- prepare the data
```
xs = np.array([-1.0, 0.0, 1.0, 2.0, 3.0, 4.0], dtype=float)
ys = np.array([-2.0, 1.0, 4.0, 7.0, 10.0, 13.0], dtype=float)
```
- design the model
```
model = tf.keras.Sequential([tf.keras.layers.Dense(units=1, input_shape=[1])])
```
- compile the model
```
model.compile(optimizer=tf.keras.optimizers.SGD(), loss=tf.keras.losses.MeanSquaredError())
```
- train the neural network
```
model.fit(xs, ys, epochs=500)
```
- final code
model.py
```
import logging
import google.cloud.logging as cloud_logging
from google.cloud.logging.handlers import CloudLoggingHandler
from google.cloud.logging_v2.handlers import setup_logging

cloud_logger = logging.getLogger('cloudLogger')
cloud_logger.setLevel(logging.INFO)
cloud_logger.addHandler(CloudLoggingHandler(cloud_logging.Client()))
cloud_logger.addHandler(logging.StreamHandler())

import tensorflow as tf
import numpy as np

xs = np.array([-1.0, 0.0, 1.0, 2.0, 3.0, 4.0], dtype=float)
ys = np.array([-2.0, 1.0, 4.0, 7.0, 10.0, 13.0], dtype=float)

model = tf.keras.Sequential([tf.keras.layers.Dense(units=1, input_shape=[1])])

model.compile(optimizer=tf.keras.optimizers.SGD(), loss=tf.keras.losses.MeanSquaredError())

model.fit(xs, ys, epochs=500)
```
- run script
python model.py
- using the model
```
# add this line to the end of model.py
cloud_logger.info(str(model.predict([10.0])))
```
## Introduction to Computer Vision with TensorFlow [GSP631]
### Access Cloud Code
python --version
python -c "import tensorflow;print(tensorflow.__version__)"
pip3 install --upgrade pip

- install google-cloud-logging
/usr/bin/python3 -m pip install -U google-cloud-logging --user

- install pylint
/usr/bin/python3 -m pip install -U pylint --user

- upgrade tensorflow
pip install --upgrade tensorflow

### Start Coding
### Import necessary packages
model.py
```
# Import and configure logging
import logging
import google.cloud.logging as cloud_logging
from google.cloud.logging.handlers import CloudLoggingHandler
from google.cloud.logging_v2.handlers import setup_logging

cloud_logger = logging.getLogger('cloudLogger')
cloud_logger.setLevel(logging.INFO)
cloud_logger.addHandler(CloudLoggingHandler(cloud_logging.Client()))
cloud_logger.addHandler(logging.StreamHandler())

import tensorflow as tf

import numpy as np

import tensorflow_datasets as tfds

```

### Loading and preprocessing the dataset
- loading the dataset via putting this code at the end of the file
```
# Define, load and configure data
(ds_train, ds_test), info = tfds.load('fashion_mnist', split=['train', 'test'], with_info=True, as_supervised=True)

# Values before normalization
image_batch, labels_batch = next(iter(ds_train))
print("Before normalization ->", np.min(image_batch[0]), np.max(image_batch[0]))
```
- data preprocessing
```
# Define batch size [number of training examples utilized in one iteration]
BATCH_SIZE = 32
```
- normalize and batch process the dataset
```
# Normalize and batch process the dataset
ds_train = ds_train.map(lambda x, y: (tf.cast(x, tf.float32)/255.0, y)).batch(BATCH_SIZE)
ds_test = ds_test.map(lambda x, y: (tf.cast(x, tf.float32)/255.0, y)).batch(BATCH_SIZE)

# Examine the min and max values of the batch after normalization
image_batch, labels_batch = next(iter(ds_train))
print("After normalization ->", np.min(image_batch[0]), np.max(image_batch[0]))
```

### Design your model

### Compile and train the model

### Evaluate model performance on unseen data

### Saving and loading the model

### Explore callbacks


## Introduction to Convolutions with TensorFlow []
### 

## Classify Images with TensorFlow Convolutional Neural Networks []
### 

## Identify Horses or Humans with TensorFlow and Vertex AI []
### 

## Classify Images with TensorFlow on Google Cloud: Challenge Lab []
### 
