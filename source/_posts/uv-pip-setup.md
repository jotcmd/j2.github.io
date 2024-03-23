---
title: pyenv+uv快速setup开源项目
date: 2024-03-23 09:29:50
tags:
---

以`gpt-newspaper`项目为例

1. 先下载代码：
  ```bash
git clone https://github.com/rotemweiss57/gpt-newspaper
```
1. 如果没有安装pyenv/uv，则先安装：
  pyenv用于管理不同的python版本，如果macos自带的python3.9够用，也可以不安装。
  ```bash
curl https://pyenv.run | bash
### into .bashrc
export PYENV_ROOT="$HOME/.pyenv"
[[ -d $PYENV_ROOT/bin ]] && export PATH="$PYENV_ROOT/bin:$PATH"
eval "$(pyenv init -)"
# Load pyenv-virtualenv automatically by adding
eval "$(pyenv virtualenv-init -)"
```
  uv用于替代pip，其实也内置了venv，大部分情况可能uv足矣。
  ```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
### into .bashrc
source "$HOME/.cargo/env"
```
3. 切换到代码目录
  ```bash
uv venv
uv pip install -r requirements.txt
```
4. vscode配置关联当前目录的虚拟环境
  ```json
{
  "python.defaultInterpreterPath": "${workspaceFolder}/.venv"
}
```
  这个其实默认不配置也行，但两种情况需要；
  - 工作区，有多项目，这个可以设置到`文件夹`级别的setting，以区分不同的虚拟环境。
  - 项目中，混合了多语言，比如包含了前端代码，那么`requirements.txt`不在根目录，在子目录，比如：dify，这时候，需要显式指定，并增加目录名，比如：
    `"python.defaultInterpreterPath": "${workspaceFolder}/api/.venv"`
5. 查看下`import`的依赖库，就知道是否生效了。

前面也提过poetry工具，总结下：

1. 多python**版本管理**，一定是用`pyenv`；
2. 如果**不涉及cuda**的开源项目、只为了源码快速阅读，安装`uv`即可，python版本不合适，上pyenv；
3. 如果是一步步自己写起来的python项目，推荐`poetry`，信息更多、更规范，利于环境还原；
4. 如果涉及cude的项目，**虚拟环境**用`conda`来创建更好，涉及**环境换源**，考虑第2/3点；


