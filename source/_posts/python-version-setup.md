---
title: 使用pyenv方式在debian系安装指定版本的python
date: 2024-02-24 11:11:50
tags:
---

首先，我们需要了解`pyenv`的主要功能和特点：`pyenv`是一个用于管理多个Python版本的工具。它允许用户在同一台机器上安装和使用多个版本的Python，而不会互相干扰。基于这一点，我们可以考虑一些具有相似功能的工具：

1. `virtualenv`：
   - 功能：创建隔离的Python环境。
   - 特点：每个`virtualenv`环境都可以有自己的Python版本和库，但它不提供管理和安装多个Python版本的功能。

2. `conda`：
   - 功能：一个开源的包管理器和环境管理器。
   - 特点：`conda`可以用来安装、运行和升级包和其依赖项。与`pyenv`不同的是，`conda`可以用于安装多种语言的包，例如Python、R。

3. `docker`：
   - 功能：提供虚拟化的容器环境。
   - 特点：虽然它不是专门为Python设计的，但可以用于创建包含特定版本Python的容器，从而实现类似于`pyenv`的环境隔离和版本管理。

4. `asdf`：
   - 功能：一个多版本语言版本管理器。
   - 特点：支持多种语言，包括Python。可以用来安装和管理不同版本的Python。

5. `pipenv`：
   - 功能：旨在将`pip`和`virtualenv`的特点结合起来。
   - 特点：自动创建和管理一个虚拟环境，同时也可以在`Pipfile`中管理项目的依赖。

这些工具都有各自的优势和特点，选择哪一个取决于你的具体需求和使用场景。例如，如果你需要一个更加通用的环境管理器，`conda`可能是一个不错的选择；如果你需要一个可以支持多种语言的版本管理器，那么`asdf`可能更适合你。

---

轻量化的，我选`pyenv`


- 安装`pyenv`
```bash
curl https://pyenv.run | bash
export PATH="$HOME/.pyenv/bin:$PATH"
eval "$(pyenv init --path)"        # 初始化 pyenv 的环境变量
eval "$(pyenv virtualenv-init -)"  # 初始化 pyenv-virtualenv 插件
```

- 安装`python3.10`
```bash
apt install -y build-essential libbz2-dev libncurses-dev libffi-dev libreadline-dev libsqlite3-dev xz-utils zlib1g-dev libssl-dev liblzma-dev
pyenv install 3.10   # 本质还是本地编译 😂  
# Downloading Python-3.10.13.tar.xz...
# -> https://www.python.org/ftp/python/3.10.13/Python-3.10.13.tar.xz
# Installing Python-3.10.13...
# Installed Python-3.10.13 to /home/user/.pyenv/versions/3.10.13
pyenv global 3.10.13  # 设置生效
pyenv versions # 验证生效
python3 --version
```

- 其他用到的命令
```bash
curl -LsSf https://astral.sh/uv/install.sh | sh  # 安装uv
uv venv
uv pip install poetry
eval "$(pyenv init --path)"
pyenv global 3.10.13
poetry env use 3.10.13
poetry config virtualenvs.create false
cat requirements.txt | xargs poetry add
```