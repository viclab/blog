title: 常见错误及处理方法汇总
date: 2016-03-31 03:07:08
tags:
---
经常在配置环境中出现各种莫名的错误，在此记录，以备以后查用

# 关于CentOS的Repo问题
CentOS自带库软件资源还不够丰富，比如clang这种比较常用的库就没有，需要安装额外的
库Repo, `yum install epel-release`，安装完后`yum update`发现`Error: Cannot 
retrieve metalink for repository: epel. Please verify its path and try again`的
错误，解决方法：
- 修改 `/etc/yum.repos.d/epel.repo`配置
> 编辑`[epel]`下的`baseurl`前的#号去掉，`mirrorlist`前添加#号,再运行`yum makecache`
[参考1](https://teddysun.com/153.html)
[参考2](http://www.rackspace.com/knowledge_center/article/installing-rhel-epel-repo-on-centos-5x-or-6x)

- 另一种根本解决办法（待验证）
> 根本解决办法：
```bash
yum upgrade ca-certificates –disablerepo=epel
#允许remi repository
yum install -y yum-utils
yum-config-manager –enable remi
```

# 关于devel库的问题
经常会出现在已经安装python或lua之类的库之后，依然出现错误提示
```
CMake Error at /usr/local/share/cmake-3.3/Modules/FindPackageHandleStandardArgs.cmake:148 (message):
  Could NOT find PythonLibs (missing: PYTHON_LIBRARIES PYTHON_INCLUDE_DIRS)
```
此时，就要考虑是缺乏开发库的问题了，其实可以理解为缺头文件，一条命令可以解决：
```bash
yum install python-devel  # CentOS
yum install python-dev    # Ubuntu
```
# 待续

