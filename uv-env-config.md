# Notes for UV Environment Configuration

## I. Guide
### 1. uv安装
让机器上所有用户都可用uv：
```Shell
sudo pip install uv
```

### 2. 路径配置
这个时候的uv应该是装到了系统默认python的目录下面，在我这里路径是在`/usr/local/python3.11/bin`下面，但是这个路径并不在系统环境路径`PATH`下面。我们可以在系统环境的路径下对uv和 uvx做个软链接，这样就不用添加额外的系统环境路径变量了
  
已知`/usr/local/bin`肯定在`PATH`里面，那么我们只需要这样就可以了：
```Shell
ln -s /usr/local/bin/uv /usr/local/python3.11/bin/uv
ln -s /usr/local/bin/uvx /usr/local/python3.11/bin/uvx
```
运行uv和uvx命令查看是否可以有相关输出
### 3. 创建UV环境
先安装`python`【强烈推荐！！！】：
```Bash
uv python install {option:version}
```
接下来就可以构建一个带uv python环境的工作目录了
1. 初始化uv目录
```Shell
uv init myWorkDir
```
2. 安装Python包到当前环境：
   （如果在国内，需要配置镜像源加速，参考本文档部分II不然下载很慢）
   像这里，比如我要安装django和requests：
   ```
   uv add django  # 若要指定版本： uv add django==x.xx.xx
   uv add requests
   ```
   在添加完之后，同步到当前环境：
   ```Shell
   uv sync
   ```
4. 运行Python文件
比如目录下有个main.py,我只需要运行：
```Shell
uv run main.py
```
即可

## II. Mirror Config
装Python的镜像和Python包的镜像是两回事，所以需要分别设置
### UV Python的镜像
目前`uv python install`国内的镜像源，只有南京大学的镜像：
```Shell
uv python install 3.13 -v --mirror https://mirror.nju.edu.cn/github-release/astral-sh/python-build-standalone
  # 安装了3.13的python standalone版本
```

### Python包的镜像
用于`uv add`时加速包的下载：
因为这个命令经常会用到，所以可以设置一个环境变量一劳永逸，在`~/.bashrc`文件的最后一行加上：
```Shell
export UV_DEFAULT_INDEX=http://mirrors.aliyun.com/pypi/simple/
```
然后：
```Shell
source ~/.bashrc
echo $UV_DEFAULT_INDEX
# 输出 http://mirrors.aliyun.com/pypi/simple/
```


## III. Issues
如果没有安装standalone的包，而是使用系统默认python环境，uv使用venv在当前目录下就会根据系统默认环境创建python环境。
但是，如果在/usr/local/bin中python可能在某个路径存在相同版本的python时，因为linux环境里的python是个软链接，venv的环境创建会出问题/冲突。

具体的来说，应该是python可执行文件的路径和python库的路径在venv生成的时候会对不上，导致会在引用struct这个包的时候出现异常：
```Shell
error: Querying Python at `/root/workspace/test/.venv/bin/python3` failed with exit status exit status: 1

[stderr]
Traceback (most recent call last):
  File "<string>", line 1, in <module>
  File "/root/.cache/uv/.tmpuEEAtC/python/get_interpreter_info.py", line 13, in <module>
    import struct
  File "/usr/lib/python3.11/struct.py", line 13, in <module>
    from _struct import *
ModuleNotFoundError: No module named '_struct'
```
### 解决方法1：
先创建文件夹，然后`uv venv -p <python的真实路径，非软链接>`这样venv里的环境就是对的
### 解决方法2： 
使用 `python-build-standalone`，uv的开发团队专门处理了这个问题，并推荐开发者专门使用这些standalone的python release版本： 使用 `uv python install`
    
