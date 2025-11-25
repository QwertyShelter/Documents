1. `root, dirs, files` 的含义：

`root`：当前正在遍历的目录路径

`dirs`：当前目录下的直接子目录列表（不包括子目录的内容）

`files`：当前目录下的直接文件列表

2. 示例：
```cmd
dataset/
├── raw/
│   ├── eeg/
│   │   ├── sub1.set
│   │   └── sub2.set
│   └── meg/
│       └── data.fif
└── processed/
    └── results.mat
```
```cmd
# 第一次迭代
root = "dataset"
dirs = ["raw", "processed"]      # dataset的直接子目录
files = []                       # dataset根目录下的文件

# 第二次迭代（进入raw）
root = "dataset/raw"
dirs = ["eeg", "meg"]           # raw的直接子目录  
files = []                       # raw目录下的文件

# 第三次迭代（进入raw/eeg）
root = "dataset/raw/eeg"
dirs = []                        # eeg没有子目录
files = ["sub1.set", "sub2.set"] # eeg目录下的文件

# 第四次迭代（进入raw/meg）
root = "dataset/raw/meg"
dirs = []                        # meg没有子目录
files = ["data.fif"]             # meg目录下的文件

# 第五次迭代（进入processed）
root = "dataset/processed"
dirs = []                        # processed没有子目录
files = ["results.mat"]          # processed目录下的文件
```

3. 注意事项

os.walk() does not guarantee sorted order of directories or files. Order depends on underlying file system.

两次返回的文件顺序可能不一致，保险起见是用
```python
for root, dirs, filenames in os.walk(filepath):
    for filename in sorted(filenames):   # 必须排序
        ······
```
