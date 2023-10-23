---
title:  "Repo详解"
---
待完善，暂记录几个常用的点

### 切换manifest
场景：repo init和repo sync拉取了一套代码（manifest_a.xml, branch_old），项目建立了另一个分支(branch_new)，使用了另一个manifest文件(manifest_b.xml)。怎么把代码全部安全的切到新分支上？manifest中管理的仓库不是每个仓库都有branch_new分支，有的仓库还是用branch_old。使用`repo forall -c "git checkout -b branch_new origin/branch_new"`就会有问题。  
解决：  
```
// 更改默认manifest，使用init -m之后，".repo/manifest.xml"文件的内容就会变成指定的manifest_b.xml的内容。
repo init -m manifest_b.xml
// 此时同步会按manifest_b.xml的配置同步，毕竟manifext.xml已经替换成了manifest_b.xml，-d参数使分支回到游离态。
repo sync -d
// 本地attach分支即建立本地分支，该本地分支会与manifest_b.xml中配置的远端分支同步。
repo start branch_new --all
```
此时进入一个git仓库使用`git branch -vv`就可以看到本地所关联的远端分支是想要的分支，与manifest_b.xml一致。
> 注意：repo init -m之后进行repo sync -d时可能遇到"different xxx.git vs .repo/xxx/xxx.git"的错误，只要将检出的xxx.git（非.repo下的）删除再执行repo sync -d即可。


### repo branches
这个命令都是操作本地的，与远端没有关系
### repo checkout
这个命令都是操作本地的，与远端没有关系，没有`git checkout -b branch origin/branch`这种创建本地与远端关联分支的能力。