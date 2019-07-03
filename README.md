# Caffe

&emsp;This version of Caffe is forked from [weiliu89](https://github.com/weiliu89/caffe), and some extra files are added:

## ReLU6

&emsp;In order to support ReLU6, some files from [chuanqi305](https://github.com/chuanqi305/ssd) are added, details can be found in my [blog](https://cjjohn.top/2019/06/25/%E4%B8%BAcaffe%E6%B7%BB%E5%8A%A0ReLU6%E6%94%AF%E6%8C%81.html).

# Installation
--A Chinese version can be found in my [blog](https://cjjohn.top/2019/06/19/Ubuntu-16.04%E4%B8%8B%E5%88%A9%E7%94%A8Anaconda%E7%9A%84%E6%B2%99%E7%9B%92%E7%8E%AF%E5%A2%83%E5%AE%89%E8%A3%85GPU%E7%89%88%E6%9C%ACcaffe.html).

## GPU Environment
&emsp;In order to install the GPU version of caffe, CUDA and cudnn support is in need. Here we use CUDA 8.0.61 + cudnn 5.1.1.

## Creating a Virtual Environment
&emsp;By creating a virtual environment to avoid the impact of packages installed during Caffe installation on the base environment, since Python 2.7 is used, create a caffe-specific virtual environment using the following commands:

```
conda create --name caffe-py27 python=2.7
```

## Download caffe
&emsp;Enter the virtual environment created in the previous step and download caffe:

```
cd ~/anaconda3/envs/caffe-py27/
git clone https://github.com/ShowLo/caffe.git
```

&emsp;If SSD is to be used later, you need to use the following commands here to switch to the SSD branch:

```
git checkout ssd
```

&emsp;At this point, you should be able to find a folder named `ssd` in the `examples` folder.

## Activate the Environment, Install opencv

&emsp;It's better to specify the installation of version 3.1.0 opencv from the beginning:

```
conda activate caffe-py27
conda install opencv=3.1.0
```

## Make Makefile.config

&emsp;Because the `make` instruction can only make Makefile.config file, and Makefile.config.example is an example of makefile given by caffe, first copy the content of Makefile.config.example to Makefile.config:

```
cd caffe
cp Makefile.config.example Makefile.config
```
&emsp;Next we need to make the following changes to Makefile.config:

&emsp;a.If using cudnn, the corresponding comment is removed:

```
# cuDNN acceleration switch (uncomment to build with cuDNN).
USE_CUDNN := 1
```

&emsp;b.Because the opencv version used is 3, the corresponding comments are removed:

```
# Uncomment if you're using OpenCV 3
OPENCV_VERSION := 3
```

&emsp;c.Delete/comment out the corresponding line in `CUDA_ARCH :=` according to the CUDA version

&emsp;According to the CUDA version I used, two lines have been deleted here:

```
-gencode arch=compute_20,code=sm_20 \
-gencode arch=compute_20,code=sm_21 \
```

&emsp;d.In order to use anaconda, you need to comment out the default Python 2.7, then remove the corresponding annotation about anaconda, and change ANACONDA_HOME to your Anaconda address.

```
# NOTE: this is required only if you will compile the python interface.
# We need to be able to find Python.h and numpy/arrayobject.h.
# PYTHON_INCLUDE := /usr/include/python2.7 \
#		/usr/lib/python2.7/dist-packages/numpy/core/include
# Anaconda Python distribution is quite popular. Include path:
# Verify anaconda location, sometimes it's in root.
ANACONDA_HOME := /home/chenjiarong/anaconda3/envs/caffe-py27
PYTHON_INCLUDE := $(ANACONDA_HOME)/include \
		  $(ANACONDA_HOME)/include/python2.7 \
		  $(ANACONDA_HOME)/lib/python2.7/site-packages/numpy/core/include
```

&emsp;In the same time, you need to modify:

```
# We need to be able to find libpythonX.X.so or .dylib.
# PYTHON_LIB := /usr/lib
PYTHON_LIB := $(ANACONDA_HOME)/lib
```

&emsp;e.To use Python to write layers, you can remove the corresponding comments:

```
# Uncomment to support layers written in Python (will link against Python libs)
WITH_PYTHON_LAYER := 1
```

&emsp;f.If using Ubuntu, you need to modify it to tell caffe your hdf5's address:
```
# Whatever else you find you need goes here.
INCLUDE_DIRS := $(PYTHON_INCLUDE) /usr/local/include /usr/include/hdf5/serial
LIBRARY_DIRS := $(PYTHON_LIB) /usr/local/lib /usr/lib /usr/lib/x86_64-linux-gnu /usr/lib/x86_64-linux-gnu/hdf5/serial
```

&emsp;g.If a `try using-rpath or-rpath-link' prompt appears in the error, you can add:

```
LINKFLAGS := -Wl,-rpath,$(ANACONDA_HOME)/lib
```

## Modify Makefile

```
NVCCFLAGS +=-ccbin=$(CXX) -Xcompiler-fPIC $(COMMON_FLAGS)
change to：
NVCCFLAGS += -D_FORCE_INLINES -ccbin=$(CXX) -Xcompiler -fPIC $(COMMON_FLAGS)
```

```
LIBRARIES += glog gflags protobuf boost_system boost_filesystem m hdf5_hl hdf5
change to：
LIBRARIES += glog gflags protobuf boost_system boost_filesystem m hdf5_serial_hl hdf5_serial
```

## Compile caffe

&emsp;Now that we have finished the preliminary work, we begin compiling caffe:

```
make all -j8
make test -j8
make runtest -j8
```

&emsp;If successful, you can finally see output such as the following:

```
[ RUN      ] ArgMaxLayerTest/0.TestCPUMaxValTopK
[       OK ] ArgMaxLayerTest/0.TestCPUMaxValTopK (1 ms)
[----------] 12 tests from ArgMaxLayerTest/0 (45 ms total)

[----------] Global test environment tear-down
[==========] 2101 tests from 277 test cases ran. (992402 ms total)
[  PASSED  ] 2101 tests.
```

## Configuration Environment

&emsp;Compile pycaffe

```
make pycaffe -j8
```

&emsp;Add to environmental variables

```
echo export PYTHONPATH="/home/chenjiarong/anaconda3/envs/caffe-py27/caffe/python" >> ~/.bashrc
```

&emsp;Save and exit

```
source ~/.bashrc
```

## Test

&emsp;Activate caffe environment

```
conda activate caffe-py27
```

&emsp;Opeb python

```
python
```

&emsp;Import caffe

```
>>>import caffe
```

&emsp;If no error is reported, it means that the python interface of Caffe has been compiled correctly. Of course, it's usually not so lucky. More oftern there will be `ImportError`.

&emsp;So using:

```
conda install cython scikit-image protobuf scikit-learn ipython pandas jupyter tqdm lxml pillow
```

&emsp;to install all dependency libraries.

&emsp;Finally, make a test:

```
python -c "import caffe;print caffe.__version__"
```

&emsp;if output 1.0.0, congratulation!
