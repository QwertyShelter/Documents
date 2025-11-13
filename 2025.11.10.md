# 在 conda 环境中手动升级 cudatool 版本

参考博客：https://blog.csdn.net/2401_84277762/article/details/147098323

先手动在官网下载对应版本的 cuda 包，然后如下手动指定并配置环境
```
./cuda_12.4.0_550.54.14_linux.run --silent --toolkit --override --installpath=$CONDA_PREFIX
```
上面就是只安装 toolkit，同时指定安装路径为 CONDA_PREFIX。之后再按如下方法配置环境变量
```
export PATH=$CONDA_PREFIX/bin:$PATH
 
export LD_LIBRARY_PATH=$CONDA_PREFIX/lib64:$LD_LIBRARY_PATH
 
export CUDA_HOME=$CONDA_PREFIX
```
