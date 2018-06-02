# Caffe-Installation-Process

Ubuntu version - 18
OpenCV version - 3.4.0
Python version – 2.7.4(Any Version)

The other versions of Ubuntu should work as well, because it is the Caffe and the necessary dependencies matter, not the Ubuntu version.
Caffe does require a lot of necessary dependencies to make it work. And errors occur mostly because of dependencies incompatibility.

If you have both Python and Anaconda installed, it caused some kind of a mess, and Caffe struggled with finding the appropriate libraries it needed. Note that Anaconda itself is a completely seperate Python distribution, which means we had two version of Python installed in our machine.
So we have to remove Anaconda using following command 

$ sudo rm -rf ~/anaconda2

(Change path according to your Anaconda path) 

Now we will install Boost:

$ sudo apt-get install -y --no-install-recommends libboost-all-dev

In the next step, we will install all the necessary packages for Caffe:

$ sudo apt-get install libprotobuf-dev libleveldb-dev libsnappy-dev libopencv-dev libboost-all-dev libhdf5-serial-dev libgflags-dev libgoogle-glog-dev liblmdb-dev protobuf-compiler

clone Caffe repository, and make a copy of file Makefile.config.example:

$ git clone https://github.com/BVLC/caffe
$ cd caffe
$ cp Makefile.config.example Makefile.config

Next, we will have to install all the necessary Python packages, using pip. Navigate to python folder, and type the line below:

$ sudo pip install scikit-image protobuf
$ cd python
$ for req in $(cat requirements.txt); do sudo pip install $req; done

If you are using any other version of python, then instead of using requirement.txt use mannual commands to install python packages 
follows these commands to mannually install packages -

for(python3)
$ sudo pip3 install scikit-image protobuf
$ sudo pip3 install Cpython
$ sudo pip3 install numpy
$ sudo pip3 install scipy
$ sudo pip3 install scikit-image
$ sudo pip3 install matplotlib
$ sudo pip3 install ipython3
$ sudo pip3 install h5py
$ sudo pip3 install leveldb
$ sudo pip3 install networkx
$ sudo pip3 install nose
$ sudo pip3 install pandas
$ sudo pip3 install python-dateutil
$ sudo pip3 install protobuf
$ sudo pip3 install python-gflags
$ sudo pip3 install pyyaml
$ sudo pip3 install Pillow
$ sudo pip3 install six

If you get error in any of these installation go to corresponding package site and install it from there.
Next, we have to modify the Makefile.config. 
Uncomment the line CPU_ONLY := 1, and the line OPENCV_VERSION := 3 , USE_PKG_CONFIG := 1
Also commented out the lines which indicate using Anaconda just to make sure that Anaconda doesn’t mess up with the installation and USE_CUDNN := 1 (because we are using cpu)
NOTE - 
If you are using python 3, then you need to comment path for python 2.7 and uncomment path for python 3 and in the path update your python version accordingly. I will suggest that you read the Makefile.config file and according to your setup update that file.

$ cd ..
$ sudo vim Makefile.config
 
open using vim and now change accordingly (If vim is not installed on your pc you can directly open it and update it)

Now let’s make it:
$ make all

It will take a while to finish. But you would probably get some error message like below:

CXX src/caffe/net.cpp
src/caffe/net.cpp:8:18: fatal error: hdf5.h: No such file or directory
compilation terminated.
Makefile:575: recipe for target '.build_release/src/caffe/net.o' failed
make: *** [.build_release/src/caffe/net.o] Error 1

It is because hdf5 library and hdf5_hl library actually have a postfix serial in their names, the compiler cannot find them. To fix this, we just have to make a link to the actual files. Remember, we are not changing their names!
But first, let’s check out the actual name of the libraries. It may vary on your machines, though.

$ cd /usr/lib/x86_64-linux-gnu/
$ ls -al
...
libhdf5_serial.so.10.1.0
libhdf5_serial_hl.so.10.0.2
...
You may find the two files like above. Note again that the version may be different. Just take note the ones you saw. Then we will make a link to them:

$ sudo ln -s /usr/lib/x86_64-linux-gnu/libhdf5_serial.so.10.1.0/usr/lib/x86_64-linux-gnu/libhdf5.so
$ sudo ln -s /usr/lib/x86_64-linux-gnu/libhdf5_serial_hl.so.10.0.2 /usr/lib/x86_64-linux-gnu/libhdf5_hl.so

And note that the postfixes of hdf5 and hdf5_hl are not always the same.
After doing that, try make all again, this time there should be no more errors

$ make test
$ make runtest

if you are still getting error just follow these commands

$ make clean
$ cd caffe
$ mkdir build
$ cd build
$ make -j4 or make – j8
$ make test 
$ make runtest
$ cd ..

And these two should be working as well. And you will likely see the result like below:

Now your caffe is installed.
Next step is optional but I highly recommend because we are using Python for our works. We will compile the Python layer so that we can use caffe directly in our Python source code.

$ make pycaffe

Here, most of your machines will compile without error. But someone may see some error like below.

CXX/LD -o python/caffe/_caffe.so python/caffe/_caffe.cpp
python/caffe/_caffe.cpp:10:31: fatal error: numpy/arrayobject.h: No such file or directory
compilation terminated.
Makefile:501: recipe for target 'python/caffe/_caffe.so' failed
make: *** [python/caffe/_caffe.so] Error 1

The error indicates that it can not find a header file named arrayobject.h. It was caused because numpy was installed in a different path, and we must manually point to it. Actually, this problem was solved at the time of writing, but the installation path varies, so not everyone will get through it. For ones who encountered the error above, all you have to do is to make a small change to your Makefile.config from this:

PYTHON_INCLUDE := /usr/include/python2.7 \
/usr/lib/python2.7/dist-packages/numpy/core/include

to this 

PYTHON_INCLUDE := /usr/include/python2.7 \
/usr/local/lib/python2.7/dist-packages/numpy/core/include

For python3 (my version is 3.6.0)

PYTHON_INCLUDE := /usr/include/python3.6 \
/usr/lib/python3.6/dist-packages/numpy/core/include

to this

PYTHON_INCLUDE := /usr/include/python3.6 \
/usr/local/lib/python3.6/dist-packages/numpy/core/include

After that, let’s do make pycaffe again, and it should work now.
If still makr pycaffe is not working then follow this

$ cd 
$ make clean
$ cd caffe
$ mkdir build 
$ cd build
$ make pycaffe

Next, we will have to add the module directory to our $PYTHONPATH by adding this line to the end of ~/.bashrc file.

$ sudo vim ~/.bashrc
export PYTHONPATH=$HOME/Downloads/caffe/python:$PYTHONPATH

Note that you have to change your caffe directory accordingly. We are nearly there, next execute the command below to make things take effect:

$ source ~/.bashrc

At this time, you can import caffe in Python code without any error

$ python
>>> import caffe
>>>
Now Your Caffe is completely Installed
