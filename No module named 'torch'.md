# ModuleNotFoundError: No module named 'torch'

出现时机：安装需要编译的 pip 库时出现。例如：在安装 GroundingSAM 的 grounding_dino 库以及配置 SegMAN 的 natten 库时发生。

原因：多半是编译依赖 torch 库而当前 torch 的版本不满足编译要求导致出错。

解决方案：降级 torch 或者需要编译的 pip 库。例如：GroundingSAM 将 torch 降级为所支持的最低版本的 torch==2.3.1；natten 按照官网下载特定版本的 natten
