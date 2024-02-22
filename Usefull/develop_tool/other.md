## 一些常用的配置
### AndroidStudio修改终端为git的bash.sh后的中文问题解决
git的bash.sh终端中删除中文会出现乱码的解决方式：
在git安装目录的./etc/bash.bashrc文件最后添加：
```
export LANG="zh_CN.UTF-8"
export LC_ALL="zh_CN.UTF-8"
```
