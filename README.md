# ann4brains

ann4brains implements filters for adjacency matrices that can be used within a deep neural network. These filters are designed specifically for brain network connectomes, but could be used with any adjacency matrix. 

ann4brains is a Python wrapper for [Caffe](https://github.com/BVLC/caffe) that implements the Edge-to-Edge, and Edge-to-Node filters as described in:

> Kawahara, J., Brown, C. J., Miller, S. P., Booth, B. G., Chau, V., Grunau, R. E., Zwicker, J. G., and Hamarneh, G. (2017). BrainNetCNN: Convolutional neural networks for brain networks; towards predicting neurodevelopment. NeuroImage, 146(July), 1038–1049. [[DOI]](https://doi.org/10.1016/j.neuroimage.2016.09.046) [[URL]](http://brainnetcnn.cs.sfu.ca/) [[PDF]](http://www.cs.sfu.ca/~hamarneh/ecopy/neuroimage2016.pdf)

------------------

## Hello World

Here's a fully working, minimal ["hello world" example](https://github.com/jeremykawahara/ann4brains/blob/master/examples/helloworld.py),

```python
import os, sys
import numpy as np
from scipy.stats.stats import pearsonr
import caffe
from ann4brains.synthetic.injury import ConnectomeInjury
from ann4brains.nets import BrainNetCNN
np.random.seed(seed=333) # To reproduce results.

injury = ConnectomeInjury() # Generate train/test synthetic data.
x_train, y_train = injury.generate_injury()
x_test, y_test = injury.generate_injury()
x_valid, y_valid = injury.generate_injury()

hello_arch = [ # We specify the architecture like this.
    ['e2n', {'num_output': 16,  # e2n layer with 16 filters.
             'kernel_h': x_train.shape[2], 
             'kernel_w': x_train.shape[3]}], # Same dimensions as spatial inputs.
    ['dropout', {'dropout_ratio': 0.5}], # Dropout at 0.5
    ['relu',    {'negative_slope': 0.33}], # For leaky-ReLU
    ['fc',      {'num_output': 30}],  # Fully connected (n2g) layer with 30 filters.
    ['relu',    {'negative_slope': 0.33}],
    ['out',     {'num_output': 1}]]  # Output layer with num_outs nodes as outputs.

hello_net = BrainNetCNN('hello_world', hello_arch) # Create BrainNetCNN model
hello_net.fit(x_train, y_train[:,0], x_valid, y_valid[:,0]) # Train (regress only on class 0)
preds = hello_net.predict(x_test) # Predict labels of test data
print("Correlation:", pearsonr(preds, y_test[:,0])[0])
```

More examples can be found in this [extended notebook](https://github.com/jeremykawahara/ann4brains/blob/master/examples/brainnetcnn.ipynb).

If you prefer to work directly with [Caffe](https://github.com/BVLC/caffe) and not use this wrapper, you can modify the [example prototxt files](https://github.com/jeremykawahara/ann4brains/tree/master/examples/proto) that implement the E2E and E2N filters.

------------------

## Installation

ann4brains uses the following dependencies:

- numpy
- scipy
- h5py
- matplotlib
- cPickle
- [Caffe](https://github.com/BVLC/caffe)

*You must already have [Caffe](https://github.com/BVLC/caffe) and pycaffe working on your system*

i.e., in Python, you should be able to run,
```python
import caffe
```
without errors.

To install ann4brains, download it, cd to the ann4brains root folder, and then run the install command:
```
git clone https://github.com/jeremykawahara/ann4brains.git
cd ann4brains
python setup.py install --user
```

You can run the ["hello world"](https://github.com/jeremykawahara/ann4brains/blob/master/examples/helloworld.py) example,
```
cd examples
python helloworld.py
```

which creates synthetic data, trains a small neural network, and should output the correlation of:
```
('Correlation:', 0.63340843)
```

