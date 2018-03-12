---
layout: experience
title: 用 Github 管理 Hexo 的博客项目
tags:
  - Tools
headerimg: images/comm-header/experience.jpg
abbrlink: 7e02fe5a
date: 2016-08-16 13:22:50
---
用hexo搭了这个静态博客，但是换台电脑就不可以写文章了。
尝试了[用两个分支管理博客代码](http://cnfeat.com/blog/2014/05/10/how-to-build-a-blog)之后，记录一下这个艰辛的过程。
<!-- more -->
**使用Hexo搭建静态博客流程**

1. 创建仓库，lusssn.github.io
2. 创建两个分支：master 与 hexo；
3. 设置hexo为默认分支，因为我们只需要手动管理这个分支上的Hexo网站文件
4. git clone拷贝仓库
5. 在本地 lusssn.github.io 文件夹下通过Git bash依次执行npm install hexo、hexo init、npm install 和 npm install hexo-deployer-git（此时当前分支应显示为hexo）
6. 修改_config.yml中的deploy参数，分支应为master
7. 依次执行git add .、git commit -m "..."、git push origin hexo提交网站相关的文件
8. 执行hexo g -d生成网站并部署到GitHub上

这样一来，在GitHub上的 lusssn.github.io 仓库就有两个分支，一个hexo分支用来存放网站的原始文件，一个master分支用来存放生成的静态网页。完美( •̀ ω •́ )y！

**按照上面的步骤尝试了无数次终于成功了，总结一下经验：**
* 原本在笔记本电脑上已经有一个lusssn项目了。
* 在git网页端建了两个分支后，将项目代码完整地放到了hexo分支。
* 换了一台电脑。开始没有设置默认分支为hexo。直接使用pull命令拉去代码。 `（这也可能是导致失败的一个点）`
* git push origin hexo 会报错，必须是git push origin HEAD:hexo。 `（应该是上面直接拉去分支代码的步骤导致的）`
* 失败后设置了默认分支将hexo的代码clone下来，添加了新文章，执行hexo deploy 总是会将hexo分支上的代码merge到master分支。

无论多少遍都是这样。想来想去应该是hexo配置的问题。

**于是决定重头来一遍**

* 新建文件夹 hexo init、npm install、copy原有的配置和文章之后成功deploy。`（直到这一步都没有执行git init，所以有怀疑是不是.git文件的原因）`
* 对比失败的文件目录，去掉.deploy_git文件夹、.gitmodules、.git，deploy成功。
* 执行git init，添加了.git之后deploy仍然成功。

![](mengb.jpg)     

失败的原因具体的没有找出来。可能是.deploy_git文件夹的问题。
