---
author: wschin
title: Convert existing ML models to ONNX
description: Code samples demonstrate how to use WinMLTools to convert existing models in scikit-learn and CoreML into ONNX.
ms.author: wechi
ms.date: 3/7/2018
ms.topic: article
ms.prod: windows
ms.technology: winmltools
keywords: windows 10, windows machine learning, WinML, WinMLTools, ONNX, ONNXMLTools, scikit-learn, CoreML
ms.localizationpriority: medium
---
# Convert existing ML models to ONNX
WinMLTools allows users to convert models trained in other frameworks to ONNX format. Here we demonstrate how to install WinMLTools and how to convert existing models in scikit-learn and CoreML into ONNX.

## Install WinMLTools
WinMLTools is a Python package (winmltools) that supports Python versions 2.7 and 3.6. If you are working on a data science project, we recommend installing a scientific Python distribution such as Anaconda.

WinMLTools follows the [standard python package installation process](https://packaging.python.org/installing/). From your python environment, run:

```
pip install winmltools
```
WinMLTools has the following dependencies
-	numpy v1.10.0+
-	onnxmltools 0.1.0.0+
-	protobuf v.3.1.0+

To update the dependent packages, please run the pip command with ‘-U’ argument.

```
pip install -U winmltools
```
Please follow [onnxmltools](https://github.com/onnx/onnxmltools) on GitHub for further information on onnxmltools dependencies.

Additional details on how to use WinMLTools can be found on the package specific documentation with the help function.
```
help(winmltools)
```

## Scikit-learn models
The following code snippet trains a linear support vector machine in scikit-learn and converts the model into ONNX.
~~~python
# First, we create a toy data set.
# The training matrix X contains three examples, with two features each.
# The label vector, y, stores the labels of all examples.
X = [[0.5, 1.], [-1., -1.5], [0., -2.]]
y = [1, -1, -1]

# Then, we create a linear classifier and train it.
from sklearn.svm import LinearSVC
linear_svc = LinearSVC()
linear_svc.fit(X, y)

# To convert scikit-learn models, we need to specify the input feature's name and type for our converter. 
# The following line means we have a 2-D float feature vector, and its name is "input" in ONNX.
from winmltools import convert_sklearn
linear_svc_onnx = convert_sklearn(linear_svc, input_features=[('input', 'float', 2)])

# Now, we save the ONNX model into binary format.
from winmltools.utils import save_model
save_model(linear_svc_onnx, 'linear_svc.onnx')

# If you'd like to load an ONNX binary file, our tool can also help.
from winmltools.utils import load_model
linear_svc_onnx = load_model('linear_svc.onnx')

# To see the produced ONNX model, we can print its contents or save it in text format.
print(linear_svc_onnx)
from winmltools.utils import save_text
save_text(linear_svc_onnx, 'linear_svc.txt')

# The conversion of linear regression is very similar. See the example below.
from sklearn.svm import LinearSVR
linear_svr = LinearSVR()
linear_svr.fit(X, y)
linear_svr_onnx = convert_sklearn(linear_svr, input_features=[('input', 'float', 2)])
~~~
Users can replace `LinearSVC` with other scikit-learn models such as `RandomForestClassifier`.

## Scikit-learn pipelines
Next, we show how scikit-learn pipelines can be converted into ONNX.
~~~python
# First, we create a toy data set.
# Notice that the first example's last feature value, 300, is large.
X = [[0.5, 1., 300.], [-1., -1.5, -4.], [0., -2., -1.]]
y = [1, -1, -1]

# Then, we declare a linear classifier.
from sklearn.svm import LinearSVC
linear_svc = LinearSVC()

# One common trick to improve a linear model's performance is to normalize the input data.
from sklearn.preprocessing import Normalizer
normalizer = Normalizer()

# Here, we compose our scikit-learn pipeline. 
# First, we apply our normalization. 
# Then we feed the normalized data into the linear model.
from sklearn.pipeline import make_pipeline
pipeline = make_pipeline(normalizer, linear_svc)
pipeline.fit(X, y)

# Now, we convert the scikit-learn pipeline into ONNX format. 
# Compared to the previous example, notice that the specified feature dimension becomes 3.
from winmltools import convert_sklearn
pipeline_onnx = convert_sklearn(linear_svc, name='my model', input_features=[('input', 'float', 3)])

# We can print the fresh ONNX model.
print(pipeline_onnx)

# We can also save the ONNX model into binary file for later use.
from winmltools.utils import save_model
save_model(pipeline_onnx, 'pipeline.onnx')

# Our conversion framework provides limited support of heterogeneous feature values.
# We cannot have numerical types and string type in one feature vector. 
# Let's assume that the first two features are floats and the last feature is integer (encoded a categorical attribute).
X_heter = [[0.5, 1., 300], [-1., -1.5, 400], [0., -2., 100]]

# One popular way to represent categorical is one-hot encoding.
from sklearn.preprocessing import OneHotEncoder
one_hot_encoder = OneHotEncoder(categorical_features=[2])

# Let's initialize a classifier. 
# It will be right after the one-hot encoder in our pipeline.
linear_svc = LinearSVC()

# Then, we form a two-stage pipeline.
another_pipeline = make_pipeline(one_hot_encoder, linear_svc)
another_pipeline.fit(X_heter, y)

# Now, we convert, save, and load the converted model. 
# For heterogeneous feature vectors, we need to seperately specify their types for all homogeneous segments.
another_pipeline_onnx = convert_sklearn(another_pipeline, name='my model', input_features=[('input', 'float', 2), ('another_input', 'int64', 1)])
save_model(another_pipeline_onnx, 'another_pipeline.onnx')
from winmltools.utils import load_model
loaded_onnx_model = load_model('another_pipeline.onnx')

# Of course, we can print the ONNX model to see if anything went wrong.
print(another_pipeline_onnx)
~~~

## CoreML models
Here, we assume that the path of an example CoreML model file is *example.mlmodel*. 
~~~python
from coremltools.models.utils import load_spec
# Load model file
model_coreml = load_spec('example.mlmodel')
from winmltools import convert_coreml
# Convert it!
model_onnx = convert_coreml(model_coreml)
~~~
The `model_onnx` is an ONNX [ModelProto](https://github.com/onnx/onnxmltools/blob/0f453c3f375c1ae928b83a4c7909c82c013a5bff/onnxmltools/proto/onnx-ml.proto#L176) object. We can save it in two different formats.
~~~python
from winmltools.utils import save_model
# Save the produced ONNX model in binary format
save_model(model_onnx, 'example.onnx')
# Save the produced ONNX model in text format
from winmltools.utils import save_text
save_text(model_onnx, 'example.txt')
~~~

## CoreML models with image inputs or outputs

Because of the lack of image types in ONNX, converting CoreML image models (i.e., models using images as inputs or outputs) requires some pre-processing and post-processing steps.

The objective of pre-processing is to make sure the input image is properly formatted as an ONNX tensor. Assume *X* is an image input with shape [C, H, W] in CoreML. In ONNX, the variable *X* would be a floating-point tensor with the same shape and *X*[0, :, :]/*X*[1, :, :]/*X*[2, :, :] stores the image's red/green/blue channel. For gray scale images in CoreML, their format are [1, H, W]-tensors in ONNX because we only have one channel.

If the original CoreML outputs an image, we may manually convert ONNX's floating-point output tensors back into images. There are two basic steps. The first step is to shrink values greater than 255 to 255 and change all negative values to 0. The second step is to round all pixel values to integers (by adding 0.5 and then truncating the decimals). The output channel order (e.g., RGB or BGR) is indicated in the CoreML model. To generate proper image output, we may need to transpose or shuffle to recover the desired format.

Here we consider a CoreML model, FNS-Candy, downloaded from a [model zoo](https://github.com/likedan/Awesome-CoreML-Models) as a concrete conversion example to demonstrate the difference between ONNX and CoreML formats. Note that all the subsequent commands are executed in a python enviroment.

First, we load the CoreML model and show its input and output formats.
~~~python
from coremltools.models.utils import load_spec
model_coreml = load_spec('FNS-Candy.mlmodel')
model_coreml.description # Print the content of CoreML model description
~~~
Screen output:
~~~
...
input {
    ... 
      imageType {
      width: 720
      height: 720
      colorSpace: BGR
    ...
}
...
output {
    ...
      imageType {
      width: 720
      height: 720
      colorSpace: BGR
    ...
}
...
~~~
Obviously, both of input and output are 720x720 BGR-image. Our next step is to convert the CoreML model with WinMLTools.
~~~python
from onnxmltools import convert_coreml
model_onnx = convert_coreml(model_coreml)
~~~
We can also observe the input and output formats in ONNX.
~~~python
model_onnx.graph.input # Print out the ONNX input tensor's information
~~~
Screen output:
~~~
...
  tensor_type {
    elem_type: FLOAT
    shape {
      dim {
        dim_param: "None"
      }
      dim {
        dim_value: 3
      }
      dim {
        dim_value: 720
      }
      dim {
        dim_value: 720
      }
    }
  }
...
~~~
The produced input (denoted by *X* again) in ONNX is a 4-D tensor. The last 3 axes are C-, H-, and W-axes, respectively. The first dimension is "None" which means that this ONNX model can be applied to any number of images. To apply this model to process a batch of 2 images, the first image may correspond to *X*[0, :, :, :] while *X*[1, :, :, :] is the second image. The blue/green/red channels of the first image is *X*[0, 0, :, :]/*X*[0, 1, :, :]/*X*[0, 2, :, :]. The channel order for the second image is similar.

~~~python
model_onnx.graph.output # Print out the ONNX output tensor's information
~~~
Screen output:
~~~
...
  tensor_type {
    elem_type: FLOAT
    shape {
      dim {
        dim_param: "None"
      }
      dim {
        dim_value: 3
      }
      dim {
        dim_value: 720
      }
      dim {
        dim_value: 720
      }
    }
  }
...
~~~
The output format is identical to the input format of the converted ONNX model. However, it's not an image because the pixel values are integers, not floating-point numbers. To convert it back to an image, change values greater than 255 to 255, change negative values to 0, and round off all decimals.