# IMPALA with PopArt normalisation

This repository contains a re-implementation of Deepmind's [IMPALA with PopArt](https://arxiv.org/abs/1809.04474) agent. 
Most of the repository consist of a modification the original open-sourced [IMPALA](https://github.com/deepmind/scalable_agent).  Unfortunately, this code only supports python 2.7. 

## Installing Dependencies
1. Create a new virtual environment. E.g: 
```
conda create -n impala-popart python==2.7
```
2. Install the dependencies with: 
```
pip install requirements.txt
```
3. Install the dependencies for atari. Note the quotes '' surrounding the package:
```
pip install 'gym[atari]'
```
4.  Some weird old versioning of Deepmind's Sonnet library, we have to update IPython.
```
pip install --upgrade IPython
```
5. (Optional) Build the dynamic_batching module with: 
```
TF_INC="$(python -c 'import tensorflow as tf; print(tf.sysconfig.get_include())')" && \
TF_LIB="$(python -c 'import tensorflow as tf; print(tf.sysconfig.get_lib())')" && \
g++-4.8 -std=c++11 -shared batcher.cc -o batcher.so -fPIC -I $TF_INC -O2 -D_GLIBCXX_USE_CXX11_ABI=0 -L$TF_LIB -ltensorflow_framework
```
## Installation errors
You might encounter errors during the installation of the dependencies. Here are some common errors:
#### ImportError: No module named IPython.display
```
File "/.../lib/python2.7/site-packages/sonnet/graphs.py", line 11, in <module>
    from IPython.display import Image, SVG
ImportError: No module named IPython.display
```
Simply upgrade IPython should the the trick: 
```
pip install --upgrade IPython
```

#### 'module' object has no attribute 'AbstractModule'
```
class ImpalaFeedForward(snt.AbstractModule):
    AttributeError: 'module' object has no attribute 'AbstractModule'
```
Where re-installing sonnet should do the trick: 
```
pip uninstall dm-sonnet
pip install dm-sonnet==1.23
```


Afterwards you should be ready to go!
# Running the agent
You can choose to run the agent on your local machine or in a distributed setup. 
## Local single machine training on multiple atari games. 
You can run the agent with the following commands:
```
python train_impala.py --multi_task=1
```
or 
```
python train_popart.py --multi_task=1
```
There are some pre-defined settings and hyper-parameters in `popart/flags.py` and `impala/flags.py` respectively. Feel free to modify them to your needs. 
## Distributed setting 
Use a multiplexer to execute the learner and actors in separate terminal windows.  
#### Learner (for Atari)
For instance, if you wish to run the agent with 30 actors and 1 central learner:
```sh
python train_impala.py --job_name=learner --task=0 --num_actors=30 \   
    --batch_size=32 --entropy_cost=0.01 \
    --learning_rate=0.0006 \
    --total_environment_frames=2000000000 
```
#### Actor(s)

```sh
for i in $(seq 0 29); do
  python atari_experiment.py --job_name=actor --task=$i \
      --num_actors=30 &
done;
wait
```


# Issues
Feel free to open any issues or problems, and I will try to respond as soon as possible. 