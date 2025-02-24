---
title:  "开发工具常用配置"
---

## 一些常用的配置
### AndroidStudio修改终端为git的bash.sh后的中文问题解决
git的bash.sh终端中删除中文会出现乱码的解决方式：
在git安装目录的./etc/bash.bashrc文件最后添加：
```
export LANG="zh_CN.UTF-8"
export LC_ALL="zh_CN.UTF-8"
```

## gerrit提交报错
### missing Change-Id in message footer
遇到此报错，按提示信息里的建议命令进行执行。执行scp时可能还会报
```
subsystem request failed on channel 0
scp: Connection closed
```
那么将`scp -p -P ...`改为`scp -O -P ...`则可正常运行。之后提交使用`git push origin HEAD:refs/for/分支名`
