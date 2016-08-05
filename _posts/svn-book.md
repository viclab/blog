title: svn用法总结
date: 2016-07-29 12:29:01
tags:
---
> 公司版本管理一般比较常用svn，但一直仅限于很简单的svn用法，浅尝辄止。后来在
项目开发中逐渐用到了merge，觉得还是实践中也还是积累了不少经验需要记录下来备忘。
尤其是网上搜出的svn教程完全没法儿读，乱七八糟，很费时间。

## 状态查询
在提交前，先查询一下本地的修改状态是比较好的习惯，因为有些场景并不需要提交本地
所有修改。这个时候可以考虑先查询一下所有的本地修改，然后按需提交。
```bash
svn st          # 本地所有文件的修改状态，包括非版本库中的文件及目录
svn st -q       # 本地所有版本库中的文件及目录的修改状态
```

## 提交
按需提交是比较好的习惯，而且提交前再次确认一下某个重点关注的文件的修改内容
也是好习惯。
```bash
svn diff [要关注的文件]     # svn会打印出本地修改与版本库中的差异
svn ci -m "提交说明" [要提交的文件]     # 默认提交当前目录所有版本库修改
```
## 合并
合并是个比较复杂的操作，通常最好在操作前有比较明确的认识，比如先确定是全量
合并还是部分合并，是branch-》trunk，还是trunk-》branch，以及合并之后的冲突处理。
一般全量和并操作起来比较方便，只是冲突比较多，需要花点儿时间去处理冲突。全量
合并按方向又细分为2种。
### branch->trunk
这种合并svn直接提供了一种合并选项，合并起来比较方便。
```bash
cd <trunk目录>
svn merge $SVR/branches/KiHan04 --reintegrate       # reintegrate分支到trunk专用
```
但这种合并并不是任何时候都奏效的，只能全量合并，部分合并会提示错误如下：
```
svn: E195016: 'http://tc-svn.tencent.com/ied/ied_kihan_rep/server_proj/branches/
KiHan04/server/zonesvr@112931' must be ancestrally related to 'http://tc-svn.tencent.com/ied/ied_kihan_rep/server_proj/trunk@112831'
```
提示找不到依赖的问题，这个时候 就需要另外一种手动合并的方式。
```bash
cd <trunk目录>
svn merge -r r1:r2 $SVR/branches/KiHan04 [project_dir]  # 合并r1~r2所有提交
```
其中project_dir指的是跟KiHan04对应的项目主目录，一般就是trunk。r2一般就是HEAD，
而r1则是分支从主干刚拉出的那个版本号。这个一般是很难去查和记录的，例如一个大
版本项目组中多人提交，直接去查太低效。幸亏有个好方法可以直接查询出来。
```bash
svn log -q $SVR/branches/KiHan04 --stop-on-copy
```
最底下一条log的版本号就是拉出分支时版本号，用这个作为手动合并的其实版本号r1就是了。
> 注意: 一般全量合并冲突会比较多，可以先加参数`--dry-run`来只探测合并的冲突数，而
不真正合并, 可以用这个提前检测有多少冲突。
```bash
svn merge -r r1:HEAD $SVR/branches/KiHan04/XXX/XXX [trunk/XXX/XXX] --dry-run
```

### trunk->branch
一般trunk到branch属于反向合并。在图形化界面反向合并需要注意的一点就是从在合并并解决完冲突后，在提交列表中
要去掉所有的目录，就能避免反向提交失败的问题。
命令行下是否能规避反向提交目录失败的问题还有待验证。

## 解决冲突
一般在合并完后肯定会有冲突，如何高效解决多个冲突。
- 首先在产生冲突的时候一直按p
- vi 所有冲突的文件列表
- 识别并去掉<<<< ==== >>>> 等标志
- 然后svn resolved 所有冲突文件列表
- svn st -q 确认并提交

eg：
```cpp
<<<<<<< .working                                                                                                                                            
      role_b ? role_b->IsLose(GetOfflineTimeout()) : 0,
=======
      role_b ? role_b->IsLose() : 0,
>>>>>>> .merge-right.r105441
```
`<<<<`表示本地，也即合并的目标
`>>>>`表示合并的源头，也是远程分支
`====`分割本地与源分支，跟<<<<和>>>>成对出现。

## 撤销修改
有时候会觉得修改不合适，svn提供了撤销的机制，分为提交前撤销与提交后撤销：
- 提交前
提交前撤销比较方便，如下:
```bash
svn revert <待撤销的文件或目录>     # 撤销文件
svn revert -R <目录>                # 递归撤销目录下所有修改
```
- 提交后
提交后就麻烦一点，也是借助merge的功能
```
svn merge R1:R2 <文件或目录>
svn ci -m "提交说明"
```
R1为插销文件想要回退到的版本号，R2为撤销文件最新版本号.


