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
model.py
```
# Define, load and configure data
(ds_train, ds_test), info = tfds.load('fashion_mnist', split=['train', 'test'], with_info=True, as_supervised=True)

# Values before normalization
image_batch, labels_batch = next(iter(ds_train))
print("Before normalization ->", np.min(image_batch[0]), np.max(image_batch[0]))
```
- data preprocessing
model.py
```
# Define batch size [number of training examples utilized in one iteration]
BATCH_SIZE = 32
```
- normalize and batch process the dataset
model.py
```
# Normalize and batch process the dataset
ds_train = ds_train.map(lambda x, y: (tf.cast(x, tf.float32)/255.0, y)).batch(BATCH_SIZE)
ds_test = ds_test.map(lambda x, y: (tf.cast(x, tf.float32)/255.0, y)).batch(BATCH_SIZE)

# Examine the min and max values of the batch after normalization
image_batch, labels_batch = next(iter(ds_train))
print("After normalization ->", np.min(image_batch[0]), np.max(image_batch[0]))
```

### Design your model
model.py
```
# Define the model
model = tf.keras.models.Sequential([tf.keras.layers.Flatten(),
                                    tf.keras.layers.Dense(64, activation=tf.nn.relu),
                                    tf.keras.layers.Dense(10, activation=tf.nn.softmax)])
```

### Compile and train the model
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

# Import TensorFlow
import tensorflow as tf
# Import tensorflow_datasets
import tensorflow_datasets as tfds
# Import numpy
import numpy as np

# Define, load and configure data
(ds_train, ds_test), info = tfds.load('fashion_mnist', split=['train', 'test'], with_info=True, as_supervised=True)

# Values before normalization
image_batch, labels_batch = next(iter(ds_train))
print("Before normalization ->", np.min(image_batch[0]), np.max(image_batch[0]))

# Define batch size
BATCH_SIZE = 32

# Normalize and batch process the dataset
ds_train = ds_train.map(lambda x, y: (tf.cast(x, tf.float32)/255.0, y)).batch(BATCH_SIZE)
ds_test = ds_test.map(lambda x, y: (tf.cast(x, tf.float32)/255.0, y)).batch(BATCH_SIZE)

# Examine the min and max values of the batch after normalization
image_batch, labels_batch = next(iter(ds_train))
print("After normalization ->", np.min(image_batch[0]), np.max(image_batch[0]))

# Define the model
model = tf.keras.models.Sequential([tf.keras.layers.Flatten(),
                                    tf.keras.layers.Dense(64, activation=tf.nn.relu),
                                    tf.keras.layers.Dense(10, activation=tf.nn.softmax)])

# Compile the model
model.compile(optimizer = tf.keras.optimizers.Adam(),
              loss = tf.keras.losses.SparseCategoricalCrossentropy(),
              metrics=[tf.keras.metrics.SparseCategoricalAccuracy()])
model.fit(ds_train, epochs=5)
```
### Evaluate model performance on unseen data
model.py
```
# to evaluate model, add this line to the end of the file
cloud_logger.info(model.evaluate(ds_test))
```
### Saving and loading the model

- save models by adding this code to the end of the file
model.py
```
# Save the entire model as a SavedModel.
model.save('saved_model')

# Reload a fresh Keras model from the saved model
new_model = tf.keras.models.load_model('saved_model')

# Summary of loaded SavedModel
new_model.summary()


# Save the entire model to a HDF5 file.
model.save('my_model.h5')

# Recreate the exact same model, including its weights and the optimizer
new_model_h5 = tf.keras.models.load_model('my_model.h5')

# Summary of loaded h5 model
new_model_h5.summary()
```
### Explore callbacks
callback_model.py
```
# Import and configure logging
import logging
import google.cloud.logging as cloud_logging
from google.cloud.logging.handlers import CloudLoggingHandler
from google.cloud.logging_v2.handlers import setup_logging
exp_logger = logging.getLogger('expLogger')
exp_logger.setLevel(logging.INFO)
exp_logger.addHandler(CloudLoggingHandler(cloud_logging.Client(), name="callback"))

# Import tensorflow_datasets
import tensorflow_datasets as tfds
# Import numpy
import numpy as np
# Import TensorFlow
import tensorflow as tf
# Define Callback
class myCallback(tf.keras.callbacks.Callback):
  def on_epoch_end(self, epoch, logs={}):
    if(logs.get('sparse_categorical_accuracy')>0.84):
      exp_logger.info("\nReached 84% accuracy so cancelling training!")
      self.model.stop_training = True
callbacks = myCallback()
# Define, load and configure data
(ds_train, ds_test), info = tfds.load('fashion_mnist', split=['train', 'test'], with_info=True, as_supervised=True)
# Define batch size
BATCH_SIZE = 32
# Normalizing and batch processing of data
ds_train = ds_train.map(lambda x, y: (tf.cast(x, tf.float32)/255.0, y)).batch(BATCH_SIZE)
ds_test = ds_test.map(lambda x, y: (tf.cast(x, tf.float32)/255.0, y)).batch(BATCH_SIZE)
# Define the model
model = tf.keras.models.Sequential([tf.keras.layers.Flatten(),
                                    tf.keras.layers.Dense(64, activation=tf.nn.relu),
                                    tf.keras.layers.Dense(10, activation=tf.nn.softmax)])
# Compile data
model.compile(optimizer = tf.keras.optimizers.Adam(),
              loss = tf.keras.losses.SparseCategoricalCrossentropy(),
              metrics=[tf.keras.metrics.SparseCategoricalAccuracy()])
model.fit(ds_train, epochs=5, callbacks=[callbacks])
```

### Exercise 1
explore the layers in your model. What happens when you change the number of neurons? 64 -> 128
update_model.py
```
# Import and configure logging
import logging
import google.cloud.logging as cloud_logging
from google.cloud.logging.handlers import CloudLoggingHandler
from google.cloud.logging_v2.handlers import setup_logging
up_logger = logging.getLogger('upLogger')
up_logger.setLevel(logging.INFO)
up_logger.addHandler(CloudLoggingHandler(cloud_logging.Client(), name="updated"))

# Import tensorflow_datasets
import tensorflow_datasets as tfds
# Import numpy
import numpy as np
# Import TensorFlow
import tensorflow as tf

# Define, load and configure data
(ds_train, ds_test), info = tfds.load('fashion_mnist', split=['train', 'test'], with_info=True, as_supervised=True)
# Define batch size
BATCH_SIZE = 32
# Normalizing and batch processing of data
ds_train = ds_train.map(lambda x, y: (tf.cast(x, tf.float32)/255.0, y)).batch(BATCH_SIZE)
ds_test = ds_test.map(lambda x, y: (tf.cast(x, tf.float32)/255.0, y)).batch(BATCH_SIZE)
# Define the model
model = tf.keras.models.Sequential([tf.keras.layers.Flatten(),
                                    tf.keras.layers.Dense(128, activation=tf.nn.relu),
                                    tf.keras.layers.Dense(10, activation=tf.nn.softmax)])
# Compile data
model.compile(optimizer = tf.keras.optimizers.Adam(),
              loss = tf.keras.losses.SparseCategoricalCrossentropy(),
              metrics=[tf.keras.metrics.SparseCategoricalAccuracy()])
model.fit(ds_train, epochs=5)
# Logs model summary
model.summary(print_fn=up_logger.info)
```
### Exercise 2
Consider the effects of additional layers in the network. What will happen if you add another layer between the two dense layers?
update_model.py
```
# Import and configure logging
import logging
import google.cloud.logging as cloud_logging
from google.cloud.logging.handlers import CloudLoggingHandler
from google.cloud.logging_v2.handlers import setup_logging
up_logger = logging.getLogger('upLogger')
up_logger.setLevel(logging.INFO)
up_logger.addHandler(CloudLoggingHandler(cloud_logging.Client(), name="updated"))

# Import tensorflow_datasets
import tensorflow_datasets as tfds
# Import numpy
import numpy as np
# Import TensorFlow
import tensorflow as tf

# Define, load and configure data
(ds_train, ds_test), info = tfds.load('fashion_mnist', split=['train', 'test'], with_info=True, as_supervised=True)
# Define batch size
BATCH_SIZE = 32
# Normalizing and batch processing of data
ds_train = ds_train.map(lambda x, y: (tf.cast(x, tf.float32)/255.0, y)).batch(BATCH_SIZE)
ds_test = ds_test.map(lambda x, y: (tf.cast(x, tf.float32)/255.0, y)).batch(BATCH_SIZE)
# Define the model
model = tf.keras.models.Sequential([tf.keras.layers.Flatten(),
                                    tf.keras.layers.Dense(64, activation=tf.nn.relu),
                                    tf.keras.layers.Dense(64, activation=tf.nn.relu),
                                    tf.keras.layers.Dense(10, activation=tf.nn.softmax)])
# Compile data
model.compile(optimizer = tf.keras.optimizers.Adam(),
              loss = tf.keras.losses.SparseCategoricalCrossentropy(),
              metrics=[tf.keras.metrics.SparseCategoricalAccuracy()])
model.fit(ds_train, epochs=5)
# Logs model summary
model.summary(print_fn=up_logger.info)
```
### Exercise 3
update_model.py
```
# Import and configure logging
import logging
import google.cloud.logging as cloud_logging
from google.cloud.logging.handlers import CloudLoggingHandler
from google.cloud.logging_v2.handlers import setup_logging
up_logger = logging.getLogger('upLogger')
up_logger.setLevel(logging.INFO)
up_logger.addHandler(CloudLoggingHandler(cloud_logging.Client(), name="updated"))

# Import tensorflow_datasets
import tensorflow_datasets as tfds
# Import numpy
import numpy as np
# Import TensorFlow
import tensorflow as tf

# Define, load and configure data
(ds_train, ds_test), info = tfds.load('fashion_mnist', split=['train', 'test'], with_info=True, as_supervised=True)
# Define batch size
BATCH_SIZE = 32
# Normalizing and batch processing of data
ds_train = ds_train.batch(BATCH_SIZE)
ds_test = ds_test.batch(BATCH_SIZE)
# Define the model
model = tf.keras.models.Sequential([tf.keras.layers.Flatten(),
                                    tf.keras.layers.Dense(64, activation=tf.nn.relu),
                                    tf.keras.layers.Dense(10, activation=tf.nn.softmax)])
# Compile data
model.compile(optimizer = tf.keras.optimizers.Adam(),
              loss = tf.keras.losses.SparseCategoricalCrossentropy(),
              metrics=[tf.keras.metrics.SparseCategoricalAccuracy()])
model.fit(ds_train, epochs=5)
# Logs model summary
model.summary(print_fn=up_logger.info)

# Print out max value to see the changes
image_batch, labels_batch = next(iter(ds_train))
t_image_batch, t_labels_batch = next(iter(ds_test))
up_logger.info("training images max " + str(np.max(image_batch[0])))
up_logger.info("test images max " + str(np.max(t_image_batch[0])))
```
### Exercise 4
What happens if you remove the Flatten() layer, and why?
update_model.py
```
# Import and configure logging
import logging
import google.cloud.logging as cloud_logging
from google.cloud.logging.handlers import CloudLoggingHandler
from google.cloud.logging_v2.handlers import setup_logging
up_logger = logging.getLogger('upLogger')
up_logger.setLevel(logging.INFO)
up_logger.addHandler(CloudLoggingHandler(cloud_logging.Client(), name="updated"))

# Import tensorflow_datasets
import tensorflow_datasets as tfds
# Import numpy
import numpy as np
# Import TensorFlow
import tensorflow as tf

# Define, load and configure data
(ds_train, ds_test), info = tfds.load('fashion_mnist', split=['train', 'test'], with_info=True, as_supervised=True)
# Define batch size
BATCH_SIZE = 32
# Normalizing and batch processing of data
ds_train = ds_train.batch(BATCH_SIZE)
ds_test = ds_test.batch(BATCH_SIZE)
# Define the model
# Define the model
model = tf.keras.models.Sequential([tf.keras.layers.Dense(64, activation=tf.nn.relu),
                                    tf.keras.layers.Dense(10, activation=tf.nn.softmax)])
# Compile data
model.compile(optimizer = tf.keras.optimizers.Adam(),
              loss = tf.keras.losses.SparseCategoricalCrossentropy(),
              metrics=[tf.keras.metrics.SparseCategoricalAccuracy()])
model.fit(ds_train, epochs=5)
# Logs model summary
model.summary(print_fn=up_logger.info)

# Print out max value to see the changes
image_batch, labels_batch = next(iter(ds_train))
t_image_batch, t_labels_batch = next(iter(ds_test))
up_logger.info("training images max " + str(np.max(image_batch[0])))
up_logger.info("test images max " + str(np.max(t_image_batch[0])))
```
### Exercise 5
Notice the final (output) layer. Why are there 10 neurons in the final layer? What happens if you have a different number than 10?
update_model.py
```
# Import and configure logging
import logging
import google.cloud.logging as cloud_logging
from google.cloud.logging.handlers import CloudLoggingHandler
from google.cloud.logging_v2.handlers import setup_logging
up_logger = logging.getLogger('upLogger')
up_logger.setLevel(logging.INFO)
up_logger.addHandler(CloudLoggingHandler(cloud_logging.Client(), name="updated"))

# Import tensorflow_datasets
import tensorflow_datasets as tfds
# Import numpy
import numpy as np
# Import TensorFlow
import tensorflow as tf

# Define, load and configure data
(ds_train, ds_test), info = tfds.load('fashion_mnist', split=['train', 'test'], with_info=True, as_supervised=True)
# Define batch size
BATCH_SIZE = 32
# Normalizing and batch processing of data
ds_train = ds_train.batch(BATCH_SIZE)
ds_test = ds_test.batch(BATCH_SIZE)
# Define the model
# Define the model
model = tf.keras.models.Sequential([tf.keras.layers.Dense(64, activation=tf.nn.relu),
                                    tf.keras.layers.Dense(10, activation=tf.nn.softmax)])
# Compile data
model.compile(optimizer = tf.keras.optimizers.Adam(),
              loss = tf.keras.losses.SparseCategoricalCrossentropy(),
              metrics=[tf.keras.metrics.SparseCategoricalAccuracy()])
model.fit(ds_train, epochs=5)
# Logs model summary
model.summary(print_fn=up_logger.info)

# Print out max value to see the changes
image_batch, labels_batch = next(iter(ds_train))
t_image_batch, t_labels_batch = next(iter(ds_test))
up_logger.info("training images max " + str(np.max(image_batch[0])))
up_logger.info("test images max " + str(np.max(t_image_batch[0])))
```
## Introduction to Convolutions with TensorFlow [GSP632]
### Open Vertex Notebook instance
Vertex AI
Workbench
User-managed-notebooks

### Navigate to lab notebook
training-data-analyst/self-paced-labs/learning-tensorflow/
CLS_Vertex_AI_Intro_to_CNN.ipynb

## Classify Images with TensorFlow Convolutional Neural Networks [GSP633]
### Enable Google Cloud services
gcloud services enable \
  compute.googleapis.com \
  monitoring.googleapis.com \
  logging.googleapis.com \
  notebooks.googleapis.com \
  aiplatform.googleapis.com \
  artifactregistry.googleapis.com \
  container.googleapis.com

### Open Vertex Notebook instance
Vertex AI
Workbench
User-managed-notebooks

### Navigate to lab notebook
training-data-analyst/self-paced-labs/learning-tensorflow/convolutional-neural-networks/
[CLS_Vertex_AI_CNN_fmnist.ipynb](https://github.com/GoogleCloudPlatform/training-data-analyst/blob/master/self-paced-labs/learning-tensorflow/convolutional-neural-networks/CLS_Vertex_AI_CNN_fmnist.ipynb)

- Setting up the environment
```
!pip install google-cloud-aiplatform
!pip install --user tensorflow-text
!pip install --user tensorflow-datasets
!pip install protobuf==3.20.1
```
- Restart the kernel
```
import os

if not os.getenv("IS_TESTING"):
    # Automatically restart kernel after installs
    import IPython

    app = IPython.Application.instance()
    app.kernel.do_shutdown(True)
```
- Set your project ID
```
import os

PROJECT_ID = ""

if not os.getenv("IS_TESTING"):
    # Get your Google Cloud project ID from gcloud
    shell_output=!gcloud config list --format 'value(core.project)' 2>/dev/null
    PROJECT_ID = shell_output[0]
    print("Project ID: ", PROJECT_ID)
```
- Timestamp
```
from datetime import datetime

TIMESTAMP = datetime.now().strftime("%Y%m%d%H%M%S")
```
- Create a Cloud Storage bucket
```
BUCKET_NAME = "gs://[your-bucket-name]"
REGION = "us-central1"  # @param {type:"string"}
```

```
PROJECT_ID
```

```
if BUCKET_NAME == "" or BUCKET_NAME is None or BUCKET_NAME == "gs://[your-bucket-name]":
    BUCKET_NAME = "gs://" + PROJECT_ID
```

```
BUCKET_NAME
```
- create bucket
```
gsutil mb -l $REGION $BUCKET_NAME
```
- Set up variables
```
import os
import sys

from google.cloud import aiplatform
from google.cloud.aiplatform import gapic as aip

aiplatform.init(project=PROJECT_ID, location=REGION, staging_bucket=BUCKET_NAME)
```
- Set hardware accelerators [no use GPU]
```
TRAIN_GPU, TRAIN_NGPU = (None, None)
DEPLOY_GPU, DEPLOY_NGPU = (None, None)
```
- Set pre-built containers
```
TRAIN_VERSION = "tf-cpu.2-8"
DEPLOY_VERSION = "tf2-cpu.2-8"

TRAIN_IMAGE = "us-docker.pkg.dev/vertex-ai/training/{}:latest".format(TRAIN_VERSION)
DEPLOY_IMAGE = "us-docker.pkg.dev/vertex-ai/prediction/{}:latest".format(DEPLOY_VERSION)

print("Training:", TRAIN_IMAGE, TRAIN_GPU, TRAIN_NGPU)
print("Deployment:", DEPLOY_IMAGE, DEPLOY_GPU, DEPLOY_NGPU)
```
- Set machine types
```
MACHINE_TYPE = "n1-standard"

VCPU = "4"
TRAIN_COMPUTE = MACHINE_TYPE + "-" + VCPU
print("Train machine type", TRAIN_COMPUTE)

MACHINE_TYPE = "n1-standard"

VCPU = "4"
DEPLOY_COMPUTE = MACHINE_TYPE + "-" + VCPU
print("Deploy machine type", DEPLOY_COMPUTE)
```
- Design and Train the Convolutional Neural Network [Training script]
  -Gets the directory to save the model artifacts from the environment variable AIP_MODEL_DIR. This variable is set by the training service.
  -Loads the Fashion MNIST dataset from TensorFlow Datasets (tfds).
  -Builds a CNN model using tf.keras model API. (Please see the code comments for details about the layers in the CNN).
  -Compiles the model (compile()).
  -Trains the model (fit()) for the number of epochs specified by the argument args.epochs.
  -Saves the trained model (save(MODEL_DIR)) to the specified model directory.
```
%%writefile task.py
# Training Fashion MNIST using CNN

import tensorflow_datasets as tfds
import tensorflow as tf
from tensorflow.python.client import device_lib
import argparse
import os
import sys
tfds.disable_progress_bar()

parser = argparse.ArgumentParser()

parser.add_argument('--epochs', dest='epochs',
                    default=10, type=int,
                    help='Number of epochs.')

args = parser.parse_args()

print('Python Version = {}'.format(sys.version))
print('TensorFlow Version = {}'.format(tf.__version__))
print('TF_CONFIG = {}'.format(os.environ.get('TF_CONFIG', 'Not found')))
print('DEVICES', device_lib.list_local_devices())

# Define batch size
BATCH_SIZE = 32

# Load the dataset
datasets, info = tfds.load('fashion_mnist', with_info=True, as_supervised=True)

# Normalize and batch process the dataset
ds_train = datasets['train'].map(lambda x, y: (tf.cast(x, tf.float32)/255.0, y)).batch(BATCH_SIZE)


# Build the Convolutional Neural Network
# As input, a CNN takes tensors of shape (image_height, image_width, color_channels), ignoring the batch size.
# Here, the CNN is configured to process inputs of shape (28, 28, 1), which is the format of Fashion MNIST images
# You start with a stack of Conv2D and MaxPooling2D layers.

# Conv2D layer https://www.tensorflow.org/api_docs/python/tf/keras/layers/Conv2D
# The number of output channels for each Conv2D layer is controlled by the first argument (e.g., 32 or 64). 
# The next argument specifies the size of the Convolution, in this case, a 3x3 grid.
# `relu` is used as the activation function, which you might recall is the equivalent of returning x when x>0, else returning 0.

# MaxPooling2D layer https://www.tensorflow.org/api_docs/python/tf/keras/layers/MaxPool2D
# By specifying (2,2) for the MaxPooling, the effect is to quarter the size of the image.

# Dense layers
# The output from the convolution stack is flattened and passed to a set of dense layers for classification.
model = tf.keras.models.Sequential([                               
      tf.keras.layers.Conv2D(64, (3,3), activation=tf.nn.relu, input_shape=(28, 28, 1)),
      tf.keras.layers.MaxPooling2D(2, 2),
      tf.keras.layers.Conv2D(64, (3,3), activation=tf.nn.relu),
      tf.keras.layers.MaxPooling2D(2,2),
      tf.keras.layers.Flatten(),
      tf.keras.layers.Dense(128, activation=tf.nn.relu),
      tf.keras.layers.Dense(10, activation=tf.nn.softmax)
    ])
model.compile(optimizer = tf.keras.optimizers.Adam(),
      loss = tf.keras.losses.SparseCategoricalCrossentropy(),
      metrics=[tf.keras.metrics.SparseCategoricalAccuracy()])



# Train and save the model

MODEL_DIR = os.getenv("AIP_MODEL_DIR")

model.fit(ds_train, epochs=args.epochs)
model.save(MODEL_DIR)
```
- Define the command args for the training script
```
JOB_NAME = "custom_job_" + TIMESTAMP
MODEL_DIR = "{}/{}".format(BUCKET_NAME, JOB_NAME)

EPOCHS = 5

CMDARGS = [
    "--epochs=" + str(EPOCHS),
]
```
- Train the model
Define your custom training job on Vertex AI.

Use the CustomTrainingJob class to define the job, which takes the following parameters:

  - display_name: The user-defined name of this training pipeline.
  - script_path: The local path to the training script.
  - container_uri: The URI of the training container image.
  - requirements: The list of Python package dependencies of the script.
  - model_serving_container_image_uri: The URI of a container that can serve predictions for your model — either a prebuilt container or a custom container.

  Use the run function to start training, which takes the following parameters:

  - args: The command line arguments to be passed to the Python script.
  - replica_count: The number of worker replicas.
  - model_display_name: The display name of the Model if the script produces a managed Model.
  - machine_type: The type of machine to use for training.
  - accelerator_type: The hardware accelerator type.
  - accelerator_count: The number of accelerators to attach to a worker replica.
The run function creates a training pipeline that trains and creates a Model object. After the training pipeline completes, the run function returns the Model object.
```
job = aiplatform.CustomTrainingJob(
    display_name=JOB_NAME,
    script_path="task.py",
    container_uri=TRAIN_IMAGE,
    requirements=["tensorflow_datasets==4.6.0"],
    model_serving_container_image_uri=DEPLOY_IMAGE,
)

MODEL_DISPLAY_NAME = "fashionmnist-" + TIMESTAMP

# Start the training
if TRAIN_GPU:
    model = job.run(
        model_display_name=MODEL_DISPLAY_NAME,
        args=CMDARGS,
        replica_count=1,
        machine_type=TRAIN_COMPUTE,
        accelerator_type=TRAIN_GPU.name,
        accelerator_count=TRAIN_NGPU,
    )
else:
    model = job.run(
        model_display_name=MODEL_DISPLAY_NAME,
        args=CMDARGS,
        replica_count=1,
        machine_type=TRAIN_COMPUTE,
        accelerator_count=0,
    )
```
- Deploy the model
Before you use your model to make predictions, you need to deploy it to an Endpoint. You can do this by calling the deploy function on the Model resource. This will do two things:

  - Create an Endpoint resource for deploying the Model resource to.
  - Deploy the Model resource to the Endpoint resource.
The function takes the following parameters:

  - deployed_model_display_name: A human readable name for the deployed model.
  - traffic_split: Percent of traffic at the endpoint that goes to this model, which is specified as a dictionary of one or more key/value pairs.
    - If only one model, then specify as { "0": 100 }, where "0" refers to this model being uploaded and 100 means 100% of the traffic.
    - If there are existing models on the endpoint, for which the traffic will be split, then use model_id to specify as { "0": percent, model_id: percent, ... }, where model_id is the model id of an existing model to the deployed endpoint. The percents must add up to 100.
  - machine_type: The type of machine to use for training.
  - accelerator_type: The hardware accelerator type.
  - accelerator_count: The number of accelerators to attach to a worker replica.
  - starting_replica_count: The number of compute instances to initially provision.
  - max_replica_count: The maximum number of compute instances to scale to. In this tutorial, only one instance is provisioned.
- Traffic split
The traffic_split parameter is specified as a Python dictionary. You can deploy more than one instance of your model to an endpoint, and then set the percentage of traffic that goes to each instance.

You can use a traffic split to introduce a new model gradually into production. For example, if you had one existing model in production with 100% of the traffic, you could deploy a new model to the same endpoint, direct 10% of traffic to it, and reduce the original model's traffic to 90%. This allows you to monitor the new model's performance while minimizing the distruption to the majority of users.

- Compute instance scaling
You can specify a single instance (or node) to serve your online prediction requests. This tutorial uses a single node, so the variables MIN_NODES and MAX_NODES are both set to 1.

If you want to use multiple nodes to serve your online prediction requests, set MAX_NODES to the maximum number of nodes you want to use. Vertex AI autoscales the number of nodes used to serve your predictions, up to the maximum number you set. Refer to the pricing page to understand the costs of autoscaling with multiple nodes.

- Endpoint
The method will block until the model is deployed and eventually return an Endpoint object. If this is the first time a model is deployed to the endpoint, it may take a few additional minutes to complete provisioning of resources.
```
DEPLOYED_NAME = "fashionmnist_deployed-" + TIMESTAMP

TRAFFIC_SPLIT = {"0": 100}

MIN_NODES = 1
MAX_NODES = 1

if DEPLOY_GPU:
    endpoint = model.deploy(
        deployed_model_display_name=DEPLOYED_NAME,
        traffic_split=TRAFFIC_SPLIT,
        machine_type=DEPLOY_COMPUTE,
        accelerator_type=DEPLOY_GPU.name,
        accelerator_count=DEPLOY_NGPU,
        min_replica_count=MIN_NODES,
        max_replica_count=MAX_NODES,
    )
else:
    endpoint = model.deploy(
        deployed_model_display_name=DEPLOYED_NAME,
        traffic_split=TRAFFIC_SPLIT,
        machine_type=DEPLOY_COMPUTE,
        accelerator_type=None,
        accelerator_count=0,
        min_replica_count=MIN_NODES,
        max_replica_count=MAX_NODES,
    )
```
- Make an online prediction request
```
import tensorflow_datasets as tfds
import numpy as np

tfds.disable_progress_bar()
```

```
datasets, info = tfds.load('fashion_mnist', batch_size=-1, with_info=True, as_supervised=True)

test_dataset = datasets['test']
```
- convert the dataset to NumPy arrays
```
x_test, y_test = tfds.as_numpy(test_dataset)

# Normalize (rescale) the pixel data by dividing each pixel by 255. 
x_test = x_test.astype('float32') / 255.
```
- Ensure the shapes are correct.
```
x_test.shape, y_test.shape
```

```
#@title Pick the number of test images
NUM_TEST_IMAGES = 20 #@param {type:"slider", min:1, max:20, step:1}
x_test, y_test = x_test[:NUM_TEST_IMAGES], y_test[:NUM_TEST_IMAGES]
```
- Send the prediction request
Now that you have prepared the test images, you can use them to send a prediction request. Use the Endpoint object's predict function, which takes the following parameters:
  - instances: A list of image instances. According to your custom model, each image instance should be a 3-dimensional matrix of floats. This was prepared in the previous step.

The predict function returns a list, where each element in the list corresponds to the corresponding image in the request. The ouput for each prediction will contain:
  - Confidence level for the prediction (predictions), between 0 and 1, for each of the ten classes.

Now you can run a quick evaluation on the prediction results:
  - np.argmax: Convert each list of confidence levels to a label
  - Compare the predicted labels to the actual labels
  - Calculate accuracy as no: of correct predictions/total no: of predictions
```
predictions = endpoint.predict(instances=x_test.tolist())
y_predicted = np.argmax(predictions.predictions, axis=1)

correct = sum(y_predicted == np.array(y_test.tolist()))
total = len(y_predicted)
print(
    f"Correct predictions = {correct}, Total predictions = {total}, Accuracy = {correct/total}"
)
```

## Identify Horses or Humans with TensorFlow and Vertex AI [GSP634]
### Enable Google Cloud services
gcloud services enable \
  compute.googleapis.com \
  monitoring.googleapis.com \
  logging.googleapis.com \
  notebooks.googleapis.com \
  aiplatform.googleapis.com \
  artifactregistry.googleapis.com \
  container.googleapis.com

### Deploy Vertex Notebook instance
Vertex AI
Workbench
User-managed-notebooks

### Navigate to lab notebook
training-data-analyst/self-paced-labs/learning-tensorflow/convolutions-with-complex-images/
[CLS_Vertex_AI_CNN_horse_or_human.ipynb](https://github.com/GoogleCloudPlatform/training-data-analyst/blob/master/self-paced-labs/learning-tensorflow/convolutions-with-complex-images/CLS_Vertex_AI_CNN_horse_or_human.ipynb)

This tutorial demonstrates how to

Train a CNN model on Vertex AI using the SDK for Python
Deploy a custom image classification model for online prediction using Vertex AI
Dataset
You will train a neural network to classify images from a dataset called [Horses or Humans](https://www.tensorflow.org/datasets/catalog/horses_or_humans).

Objective
In this notebook, you will create a custom-trained CNN model from a Python script in a Docker container using the Vertex SDK for Python, and then do a prediction on the deployed model by sending data. Alternatively, you can create custom-trained models using gcloud command-line tool, or online using the Cloud Console.

The steps performed include:

Create a Vertex AI custom job and train a CNN model.
Deploy the Model resource to a serving Endpoint resource.
Make a prediction.
Undeploy the Model resource.

- Setting up the environment [installation]
```
!pip install --user google-cloud-aiplatform 
!pip install --user tensorflow-text 
!pip install --user tensorflow-datasets

!pip install --user protobuf==3.20.1
```
- Restart the kernel
```
import os

if not os.getenv("IS_TESTING"):
    # Automatically restart kernel after installs
    import IPython

    app = IPython.Application.instance()
    app.kernel.do_shutdown(True)
```
- Set your project ID
```
import os

PROJECT_ID = ""

if not os.getenv("IS_TESTING"):
    # Get your Google Cloud project ID from gcloud
    shell_output=!gcloud config list --format 'value(core.project)' 2>/dev/null
    PROJECT_ID = shell_output[0]
    print("Project ID: ", PROJECT_ID)
```
- Timestamp
```
from datetime import datetime

TIMESTAMP = datetime.now().strftime("%Y%m%d%H%M%S")
```
- Create a Cloud Storage bucket
```
BUCKET_NAME = "gs://[your-bucket-name]"
REGION = "us-central1"  # @param {type:"string"}
```

```
PROJECT_ID
```

```
if BUCKET_NAME == "" or BUCKET_NAME is None or BUCKET_NAME == "gs://[your-bucket-name]":
    BUCKET_NAME = "gs://" + PROJECT_ID
```

```
BUCKET_NAME
```

```
gsutil mb -l $REGION $BUCKET_NAME
```
- Import Vertex SDK for Python
```
import os
import sys

from google.cloud import aiplatform
from google.cloud.aiplatform import gapic as aip

aiplatform.init(project=PROJECT_ID, location=REGION, staging_bucket=BUCKET_NAME)
```
- Set hardware accelerators
```
TRAIN_GPU, TRAIN_NGPU = (None, None)
DEPLOY_GPU, DEPLOY_NGPU = (None, None)
```
- Set pre-built containers
```
TRAIN_VERSION = "tf-cpu.2-8"
DEPLOY_VERSION = "tf2-cpu.2-8"

TRAIN_IMAGE = "us-docker.pkg.dev/vertex-ai/training/{}:latest".format(TRAIN_VERSION)
DEPLOY_IMAGE = "us-docker.pkg.dev/vertex-ai/prediction/{}:latest".format(DEPLOY_VERSION)

print("Training:", TRAIN_IMAGE, TRAIN_GPU, TRAIN_NGPU)
print("Deployment:", DEPLOY_IMAGE, DEPLOY_GPU, DEPLOY_NGPU)
```
- Set machine types
```
MACHINE_TYPE = "e2-standard"

VCPU = "4"
TRAIN_COMPUTE = MACHINE_TYPE + "-" + VCPU
print("Train machine type", TRAIN_COMPUTE)

MACHINE_TYPE = "e2-standard"

VCPU = "4"
DEPLOY_COMPUTE = MACHINE_TYPE + "-" + VCPU
print("Deploy machine type", DEPLOY_COMPUTE)
```
- Design and Train the Convolutional Neural Network[Training script]
```
%%writefile task.py
# Training Horses vs Humans using CNN

import tensorflow_datasets as tfds
import tensorflow as tf
from tensorflow.python.client import device_lib
import argparse
import os
import sys
tfds.disable_progress_bar()

parser = argparse.ArgumentParser()

parser.add_argument('--epochs', dest='epochs',
                    default=10, type=int,
                    help='Number of epochs.')

args = parser.parse_args()

print('Python Version = {}'.format(sys.version))
print('TensorFlow Version = {}'.format(tf.__version__))
print('TF_CONFIG = {}'.format(os.environ.get('TF_CONFIG', 'Not found')))
print('DEVICES', device_lib.list_local_devices())

# Define batch size
BATCH_SIZE = 32

# Load the dataset
datasets, info = tfds.load('horses_or_humans', with_info=True, as_supervised=True)

# Normalize and batch process the dataset
ds_train = datasets['train'].map(lambda x, y: (tf.cast(x, tf.float32)/255.0, y)).batch(BATCH_SIZE)


# Build the Convolutional Neural Network
model = tf.keras.models.Sequential([
    # Note the input shape is the desired size of the image 300x300 with 3 bytes color
    tf.keras.layers.Conv2D(16, (3,3), activation='relu', input_shape=(300, 300, 3)),
    tf.keras.layers.MaxPooling2D(2, 2),
    tf.keras.layers.Conv2D(32, (3,3), activation='relu'),
    tf.keras.layers.MaxPooling2D(2,2),
    tf.keras.layers.Conv2D(64, (3,3), activation='relu'),
    tf.keras.layers.MaxPooling2D(2,2),
    tf.keras.layers.Conv2D(64, (3,3), activation='relu'),
    tf.keras.layers.MaxPooling2D(2,2),
    tf.keras.layers.Conv2D(64, (3,3), activation='relu'),
    tf.keras.layers.MaxPooling2D(2,2),
    tf.keras.layers.Flatten(),
    tf.keras.layers.Dense(512, activation='relu'),
    # Only 1 output neuron. It will contain a value from 0-1 where 0 for 1 class ('horses') and 1 for the other ('humans')
    tf.keras.layers.Dense(1, activation='sigmoid')
])

model.compile(optimizer = tf.keras.optimizers.RMSprop(),
      loss = tf.keras.losses.BinaryCrossentropy(),
      metrics=[tf.keras.metrics.BinaryAccuracy()])



# Train and save the model
MODEL_DIR = os.getenv("AIP_MODEL_DIR")

model.fit(ds_train, epochs=args.epochs)
model.save(MODEL_DIR)
```
- Define the command args for the training script
```
JOB_NAME = "custom_job_" + TIMESTAMP
MODEL_DIR = "{}/{}".format(BUCKET_NAME, JOB_NAME)

EPOCHS = 8

CMDARGS = [
    "--epochs=" + str(EPOCHS),
]
```
- Train the model
```
job = aiplatform.CustomTrainingJob(
    display_name=JOB_NAME,
    script_path="task.py",
    container_uri=TRAIN_IMAGE,
    requirements=["tensorflow_datasets==4.6.0"],
    model_serving_container_image_uri=DEPLOY_IMAGE,
)

MODEL_DISPLAY_NAME = "horsesvshumans-" + TIMESTAMP

# Start the training
if TRAIN_GPU:
    model = job.run(
        model_display_name=MODEL_DISPLAY_NAME,
        args=CMDARGS,
        replica_count=1,
        machine_type=TRAIN_COMPUTE,
        accelerator_type=TRAIN_GPU.name,
        accelerator_count=TRAIN_NGPU,
    )
else:
    model = job.run(
        model_display_name=MODEL_DISPLAY_NAME,
        args=CMDARGS,
        replica_count=1,
        machine_type=TRAIN_COMPUTE,
        accelerator_count=0,
    )
```
- Testing
```
import tensorflow_datasets as tfds
import numpy as np

tfds.disable_progress_bar()
```

```
datasets, info = tfds.load('horses_or_humans', batch_size=-1, with_info=True, as_supervised=True)

test_dataset = datasets['test']
```

```
x_test, y_test = tfds.as_numpy(test_dataset)

# Normalize (rescale) the pixel data by dividing each pixel by 255. 
x_test = x_test.astype('float32') / 255.
```

```
x_test.shape, y_test.shape
```

```
#@title Pick the number of test images
NUM_TEST_IMAGES = 20 #@param {type:"slider", min:1, max:20, step:1}
x_test, y_test = x_test[:NUM_TEST_IMAGES], y_test[:NUM_TEST_IMAGES]
```

```
# Convert to python list
list_x_test = x_test.tolist()
```
- Batch prediction request
```
import json

PUBLIC_BUCKET_NAME = 'gs://spls'
PUBLIC_DESTINATION_FOLDER = 'gsp634'

RESULTS_DIRECTORY = "prediction_results"

# Create destination directory for downloaded results.
os.makedirs(RESULTS_DIRECTORY, exist_ok=True)

BATCH_PREDICTION_GCS_DEST_PREFIX = PUBLIC_BUCKET_NAME + "/" + PUBLIC_DESTINATION_FOLDER

# Download the results.
! gsutil -m cp -r $BATCH_PREDICTION_GCS_DEST_PREFIX $RESULTS_DIRECTORY

# Store the downloaded prediction file paths into a list
results_files = []
for dirpath, subdirs, files in os.walk(RESULTS_DIRECTORY):
    for file in files:
        if file.startswith("prediction.results"):
            results_files.append(os.path.join(dirpath, file))

            
def consolidate_results():          
    # Consolidate all the results into a list
    results = []
    for results_file in results_files:
        # Download each result
        with open(results_file, "r") as file:
            results.extend([json.loads(line) for line in file.readlines()])
    return results

results = consolidate_results()
```
- Rearrange the Batch prediction result
```
def rearrange_results():
    # Empty array to hold the new ordered result
    new_results = [None]*20

    for result in results:
        instance_img = np.array(result["instance"])
        for i in range(len(x_test)):
            # If the result image and x_test image matches,
            # then get the index of the image in x_test.
            # Insert the result into that index
            # in the new_results array
            if (instance_img == x_test[i]).all():
                new_results[i] = result
    return new_results
                
new_results = rearrange_results()
```
- Evaluate results
```
def evaluate_results():
    # If the prediction value of result is less than 0.5 then it means model predicted this image to belong to class 0, i.e, horse,
    # else the model predicted the image to belong to class 1, i.e, human.
    y_predicted = [0 if result["prediction"][0] <= 0.5 else 1  for result in new_results]

    correct_ = sum(y_predicted == y_test)
    accuracy = len(y_predicted)
    print(
        f"Correct predictions = {correct_}, Total predictions = {accuracy}, Accuracy = {correct_/accuracy}"
    )
    return y_predicted
    
y_predicted = evaluate_results()
```
- Plot the result
```
import matplotlib.pyplot as plt

label = {0: 'horse', 1: 'human'}

def display_instance_images(rows=64, cols=4):
  """Display given images and their labels in a grid."""
  fig = plt.figure()
  fig.set_size_inches(cols * 3, rows * 3)
  count = 0
  i = 0
  for result in new_results:
      plt.subplot(rows, cols, i + 1)
      plt.axis('off')
      plt.imshow(result['instance'])
      plt.title(label[y_predicted[count]])
      i += 1
      count += 1

display_instance_images()
```
- [Optional] Make a batch prediction request
```
if BUCKET_NAME == "" or BUCKET_NAME is None or BUCKET_NAME == "gs://[your-bucket-name]":
    BUCKET_NAME = "gs://" + PROJECT_ID
```

```
import json

BATCH_PREDICTION_INSTANCES_FILE = "batch_prediction_instances.jsonl"

BATCH_PREDICTION_GCS_SOURCE = (
    BUCKET_NAME + "/batch_prediction_instances/" + BATCH_PREDICTION_INSTANCES_FILE
)

# Write instances at JSONL
with open(BATCH_PREDICTION_INSTANCES_FILE, "w") as f:
    for x in list_x_test:
        f.write(json.dumps(x) + "\n")

# Upload to Cloud Storage bucket
! gsutil cp $BATCH_PREDICTION_INSTANCES_FILE $BATCH_PREDICTION_GCS_SOURCE

print("Uploaded instances to: ", BATCH_PREDICTION_GCS_SOURCE)
```
- Send the prediction request
```
MIN_NODES = 1
MAX_NODES = 1

# The name of the job
BATCH_PREDICTION_JOB_NAME = "horsesorhuman_batch-" + TIMESTAMP

# Folder in the bucket to write results to
DESTINATION_FOLDER = "batch_prediction_results"

# The Cloud Storage bucket to upload results to
BATCH_PREDICTION_GCS_DEST_PREFIX = BUCKET_NAME + "/" + DESTINATION_FOLDER

# Make SDK batch_predict method call
batch_prediction_job = model.batch_predict(
    instances_format="jsonl",
    predictions_format="jsonl",
    job_display_name=BATCH_PREDICTION_JOB_NAME,
    gcs_source=BATCH_PREDICTION_GCS_SOURCE,
    gcs_destination_prefix=BATCH_PREDICTION_GCS_DEST_PREFIX,
    model_parameters=None,
    machine_type=DEPLOY_COMPUTE,
    accelerator_type=None,
    accelerator_count=0,
    starting_replica_count=MIN_NODES,
    max_replica_count=MAX_NODES,
    sync=True,
)
```
- Retrieve batch prediction results
```
RESULTS_DIRECTORY = "prediction_results"
RESULTS_DIRECTORY_FULL = RESULTS_DIRECTORY + "/" + DESTINATION_FOLDER

# Create missing directories
os.makedirs(RESULTS_DIRECTORY, exist_ok=True)

# Get the Cloud Storage paths for each result
! gsutil -m cp -r $BATCH_PREDICTION_GCS_DEST_PREFIX $RESULTS_DIRECTORY

# Get most recently modified directory
latest_directory = max(
    [
        os.path.join(RESULTS_DIRECTORY_FULL, d)
        for d in os.listdir(RESULTS_DIRECTORY_FULL)
    ],
    key=os.path.getmtime,
)

# Get downloaded results in directory
results_files = []
for dirpath, subdirs, files in os.walk(latest_directory):
    for file in files:
        if file.startswith("prediction.results"):
            results_files.append(os.path.join(dirpath, file))

# Consolidate all the results into a list
def consolidate_results():          
    # Consolidate all the results into a list
    results = []
    for results_file in results_files:
        # Download each result
        with open(results_file, "r") as file:
            results.extend([json.loads(line) for line in file.readlines()])
    return results
results = consolidate_results()
```
- Rearrange the Batch prediction result
```
# Rearrange results
def rearrange_results():
    # Empty array to hold the new ordered result
    new_results = [None]*20

    for result in results:
        instance_img = np.array(result["instance"])
        for i in range(len(x_test)):
            # If the result image and x_test image matches,
            # then get the index of the image in x_test.
            # Insert the result into that index
            # in the new_results array
            if (instance_img == x_test[i]).all():
                new_results[i] = result
    return new_results
new_results = rearrange_results()
```
- Evaluate results
```
# Evaluate results
def evaluate_results():
    # If the prediction value of result is less than 0.5 then it means model predicted this image to belong to class 0, i.e, horse,
    # else the model predicted the image to belong to class 1, i.e, human.
    y_predicted = [0 if result["prediction"][0] <= 0.5 else 1  for result in new_results]

    correct_ = sum(y_predicted == y_test)
    accuracy = len(y_predicted)
    print(
        f"Correct predictions = {correct_}, Total predictions = {accuracy}, Accuracy = {correct_/accuracy}"
    )
    return y_predicted
y_predicted = evaluate_results()
```
- Plot the result
```
import matplotlib.pyplot as plt

label = {0: 'horse', 1: 'human'}

def display_instance_images(rows=64, cols=4):
  """Display given images and their labels in a grid."""
  fig = plt.figure()
  fig.set_size_inches(cols * 3, rows * 3)
  count = 0
  i = 0
  for result in new_results:
      plt.subplot(rows, cols, i + 1)
      plt.axis('off')
      plt.imshow(result['instance'])
      plt.title(label[y_predicted[count]])
      i += 1
      count += 1

display_instance_images()
```

## Classify Images with TensorFlow on Google Cloud: Challenge Lab [GSP398]
### Enable google cloud service
gcloud services enable \
  compute.googleapis.com \
  monitoring.googleapis.com \
  logging.googleapis.com \
  notebooks.googleapis.com \
  aiplatform.googleapis.com \
  artifactregistry.googleapis.com \
  container.googleapis.com

### Challenge scenario
You were recently hired as a Machine Learning Engineer for an Optical Character Recognition app development team. Your manager has tasked you with building a machine learning model to recognize Hiragana alphabets. The challenge: your business requirements are that you have just 6 weeks to produce a model that achieves greater than 90% accuracy to improve upon an existing bootstrapped solution. Furthermore, after doing some exploratory analysis in your startup's data warehouse, you found that you only have a small dataset of 60k images of alphabets to build a higher-performing solution.

To build and deploy a high-performance machine learning model with limited data quickly, you will walk through training and deploying a CNN classifier for online predictions on Google Cloud's Vertex AI platform. Vertex AI is Google Cloud's next-generation machine learning development platform where you can leverage the latest ML pre-built components to significantly enhance your development productivity, scale your workflow and decision-making with your data, and accelerate time to value.

First, you will progress through a typical experimentation workflow where you will write a script that trains your custom CNN model using tf.keras classification layers. You will then send the model code to a custom training job and run the custom training job using pre-built Docker containers provided by Vertex AI to run training and prediction. Lastly, you will deploy the model to an endpoint so that you can use your model for predictions.

### Create a Vertex Notebooks instance
Vertex AI 
Workbench 
User-Managed Notebooks
Create a Notebook instance
TensorFlow Enterprise 2.11 Without GPUs
Name your notebook cnn-challenge

### Download the Challenge Notebook
git clone https://github.com/GoogleCloudPlatform/training-data-analyst

training-data-analyst/self-paced-labs/learning-tensorflow/cnn-challenge-lab/cnn-challenge-lab.ipynb

- Installation

```
import os
! pip3 install --user --upgrade google-cloud-aiplatform google-cloud-storage google-cloud-logging protobuf==3.19.* pillow numpy
```
- restart kernel
```

import os

if not os.getenv("IS_TESTING"):
    # Automatically restart kernel after installs
    import IPython

    app = IPython.Application.instance()
    app.kernel.do_shutdown(True)
```
- set project id
```
import os

# Retrieve and set PROJECT_ID environment variables.
# TODO: fill in PROJECT_ID.

PROJECT_ID = ""

if not os.getenv("IS_TESTING"):
    # Get your Google Cloud project ID from gcloud
    shell_output=!gcloud config list --format 'value(core.project)' 2>/dev/null
    PROJECT_ID = shell_output[0]
```

```
PROJECT_ID
```

```
from datetime import datetime

TIMESTAMP = datetime.now().strftime("%Y%m%d%H%M%S")
```

```
REGION = "us-central1"  # @param {type:"string"}
BUCKET_NAME = "gs://[your-bucket-name]"
```

```
# TODO: Create a globally unique Google Cloud Storage bucket name for artifact storage.
# HINT: Start the name with gs://
if BUCKET_NAME == "" or BUCKET_NAME is None or BUCKET_NAME == "gs://[your-bucket-name]":
    BUCKET_NAME = "gs://" + PROJECT_ID
    print(BUCKET_NAME)
```
- create bucket
```
! gsutil mb -l $REGION $BUCKET_NAME
```
- Import Vertex SDK for Python
```
import os
import sys

from google.cloud import aiplatform
from google.cloud.aiplatform import gapic as aip

aiplatform.init(project=PROJECT_ID, location=REGION, staging_bucket=BUCKET_NAME)
```

```
TRAIN_GPU, TRAIN_NGPU = (None, None)
DEPLOY_GPU, DEPLOY_NGPU = (None, None)
```

```
TRAIN_VERSION = "tf-cpu.2-8"
DEPLOY_VERSION = "tf2-cpu.2-8"

TRAIN_IMAGE = "us-docker.pkg.dev/vertex-ai/training/{}:latest".format(TRAIN_VERSION)
DEPLOY_IMAGE = "us-docker.pkg.dev/vertex-ai/prediction/{}:latest".format(DEPLOY_VERSION)

print("Training:", TRAIN_IMAGE, TRAIN_GPU, TRAIN_NGPU)
print("Deployment:", DEPLOY_IMAGE, DEPLOY_GPU, DEPLOY_NGPU)
```

```
MACHINE_TYPE = "n1-standard"

VCPU = "4"
TRAIN_COMPUTE = MACHINE_TYPE + "-" + VCPU
print("Train machine type", TRAIN_COMPUTE)

MACHINE_TYPE = "n1-standard"

VCPU = "4"
DEPLOY_COMPUTE = MACHINE_TYPE + "-" + VCPU
print("Deploy machine type", DEPLOY_COMPUTE)
```

### Create a training script
- training script
```
%%writefile task.py
# Training kmnist using CNN

import tensorflow_datasets as tfds
import tensorflow as tf
from tensorflow.python.client import device_lib
import argparse
import os
import sys
tfds.disable_progress_bar()

parser = argparse.ArgumentParser()

parser.add_argument('--epochs', dest='epochs',
                    default=10, type=int,
                    help='Number of epochs.')

args = parser.parse_args()

print('Python Version = {}'.format(sys.version))
print('TensorFlow Version = {}'.format(tf.__version__))
print('TF_CONFIG = {}'.format(os.environ.get('TF_CONFIG', 'Not found')))
print('DEVICES', device_lib.list_local_devices())

# Define batch size
BATCH_SIZE = 32

# Load the dataset
datasets, info = tfds.load('kmnist', with_info=True, as_supervised=True)

# Normalize and batch process the dataset
ds_train = datasets['train'].map(lambda x, y: (tf.cast(x, tf.float32)/255.0, y)).batch(BATCH_SIZE)


# Build the Convolutional Neural Network
model = tf.keras.models.Sequential([                               
      tf.keras.layers.Conv2D(16, (3,3), activation=tf.nn.relu, input_shape=(28, 28, 1), padding = "same"),
      tf.keras.layers.MaxPooling2D(2,2),
      tf.keras.layers.Conv2D(16, (3,3), activation=tf.nn.relu, padding = "same"),
      tf.keras.layers.MaxPooling2D(2,2),
      tf.keras.layers.Flatten(),
      tf.keras.layers.Dense(128, activation=tf.nn.relu),
      # TODO: Write the last layer.
      # Hint: KMNIST has 10 output classes.
      tf.keras.layers.Dense(10, activation=tf.nn.softmax)
      
    ])

model.compile(optimizer = tf.keras.optimizers.Adam(),
      loss = tf.keras.losses.SparseCategoricalCrossentropy(),
      metrics=[tf.keras.metrics.SparseCategoricalAccuracy()])



# Train and save the model

MODEL_DIR = os.getenv("AIP_MODEL_DIR")

model.fit(ds_train, epochs=args.epochs)

# TODO: Save your CNN classifier. 
# Hint: Save it to MODEL_DIR.
model.save(MODEL_DIR)
```
- define the command args for training script
```
JOB_NAME = "custom_job_" + TIMESTAMP
MODEL_DIR = "{}/{}".format(BUCKET_NAME, JOB_NAME)

EPOCHS = 5

CMDARGS = [
    "--epochs=" + str(EPOCHS),
]
```

### Train the model
- train the model
```
job = aiplatform.CustomTrainingJob(
    display_name=JOB_NAME,
    requirements=["tensorflow_datasets==4.6.0"],
    # TODO: fill in the remaining arguments for the CustomTrainingJob function.
    script_path="task.py",
    container_uri=TRAIN_IMAGE,
    model_serving_container_image_uri=DEPLOY_IMAGE,
)

MODEL_DISPLAY_NAME = "kmnist-" + TIMESTAMP

# Start the training
model = job.run(
    model_display_name=MODEL_DISPLAY_NAME,
    replica_count=1,
    accelerator_count=0,
    # TODO: fill in the remaining arguments to run the custom training job function.
    args=CMDARGS,
    machine_type=TRAIN_COMPUTE,
)
```

### Deploy the model to a Vertex Online Prediction Endpoint

```
DEPLOYED_NAME = "kmnist_deployed-" + TIMESTAMP

TRAFFIC_SPLIT = {"0": 100}

MIN_NODES = 1
MAX_NODES = 1

endpoint = model.deploy(
        deployed_model_display_name=DEPLOYED_NAME,
        accelerator_type=None,
        accelerator_count=0,
        # TODO: fill in the remaining arguments to deploy the model to an endpoint.
        traffic_split=TRAFFIC_SPLIT,
        machine_type=DEPLOY_COMPUTE,
        min_replica_count=MIN_NODES,
        max_replica_count=MAX_NODES,
    )
```

### Query deployed model on Vertex Online Prediction Endpoint

```
import tensorflow_datasets as tfds
import numpy as np

tfds.disable_progress_bar()
```

```
datasets, info = tfds.load('kmnist', batch_size=-1, with_info=True, as_supervised=True)

test_dataset = datasets['test']
```

```
x_test, y_test = tfds.as_numpy(test_dataset)

# Normalize (rescale) the pixel data by dividing each pixel by 255. 
x_test = x_test.astype('float32') / 255.
x_test.shape, y_test.shape
```

```
#@title Pick the number of test images
NUM_TEST_IMAGES = 20 #@param {type:"slider", min:1, max:20, step:1}
x_test, y_test = x_test[:NUM_TEST_IMAGES], y_test[:NUM_TEST_IMAGES]
```
- send the prediction request
```
# Import and configure logging
from google.cloud import logging
logging_client = logging.Client()
logger = logging_client.logger('challenge-notebook')
```

```
# TODO: use your Endpoint to return prediction for your x_test.

predictions = endpoint.predict(instances=x_test.tolist())
```

```
y_predicted = np.argmax(predictions.predictions, axis=1)

correct = sum(y_predicted == np.array(y_test.tolist()))
total = len(y_predicted)

logger.log_text(str(correct/total))

print(
    f"Correct predictions = {correct}, Total predictions = {total}, Accuracy = {correct/total}"
)
```