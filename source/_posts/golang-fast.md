---
title: golang快速安装
date: 2024-02-29 21:41:50
tags:
---

下载、解压，然后加入环境变量，即可。简单直接。

```bash
wget https://go.dev/dl/go1.22.0.linux-amd64.tar.gz
tar xvf go*.linux-amd64.tar.gz
mv go $HOME/goRoot

# bashrc
cat >>~/.bashrc<<EOF
export GOROOT=\$HOME/goRoot
export GOPATH=\$HOME/goPath
export PATH=\$GOROOT/bin:$GOPATH/bin:\$PATH
EOF

source ~/.bashrc
```