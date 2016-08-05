title: YCM重生
date: 2016-03-25 14:43:59
tags:
---

# 困扰
很早就开始用VIM，VIM称为编辑器之神。确实如此。一方面，大量的快捷操作，编辑效率
极高。但如果作为IDE,最薄弱也为人所诟病的是它的补全功能太弱，好在开源世界贡献了
丰富的插件可以选择，从一开始用omnicomplete，觉得还可以，但仅限于字面补全，
而且使用前需要借助ctag生成tag文件，文件稍有改动又要重新生成tag文件，相比于IDE
还是太弱。直到后来接触了YCM，折腾了很久，总算是在公司离线的开发机上装完
（PS:llvm+clang,gcc等库在离线安装简直是噩梦，好在后来摸索掌握了直接借助
编好的库来安装，就事半功倍了）,在KL项目组时体验还很棒，后来调到Kihan之后完全不能
够忍受, 大量分散的头文件很多无法补全，效率大减。一段时间，这一直是一个很大的困扰。

# 寻找历程
找了一段时间后，想想干嘛那么费劲呢，直接用成熟的IDE不就结了，便考虑放弃YCM,
转用eclipse，用了段时间，感觉补全功能还是挺强大的，但速度真心比VIM慢多了，
而且更重要的是eclipse的快捷键完全用不顺手，虽然可以重新设置，但终究还是
感觉很别扭，就这样，VIM和eclipse尴尬而别扭的用了段时间后，发现自己还是
离不开VIM，编辑的快捷键实在太熟练了。僵持了短时间后，考虑在eclipse装了
个vrapper（模拟vim的插件），又设置折腾了一段时间，发现还是有那么一丝欠缺。
我承认其实已经做的很好了，大部分VIM能有的命令它都有，只是eclipse跟开发机的同步
很蛋疼，必须借助samba，而且还不太稳定，这更加深了想回归VIM的想法。但苦于YCM的
补全问题没有完全结局，还是在用着eclipse+vrapper.本打算就这样用着，但一次偶然的发现居然让事情有了新的转机。

# 重生
终于，功夫不负有心人，大量的Google爬贴之后，偶然在一个英文[帖子](http://www.iauns.com/b?summary=false&articles=2013_08_Vim_YouCompleteMe)
上看到一篇关于.ycm_extra_conf.py配置的文章。作者估计遇到了跟我一样头痛的问题。
文章大致的内容就是说.ycm_extra_conf中flag的配置大概有中方式:
## *硬编码*
所谓的硬编码就是说直接在.ycm_extra_conf文件中设置flags[]数组。
> 由于是事先根据源码路径手工设置录入头文件路径，所以不免有很多遗漏，就出现部分
YCM无法找到补全的问题，在KL就存在这样的问题，但好在KL的头文件相对比较集中，
配置几个目录基本够用，虽然也有一些无法补全，但基本不影响体验。后来调到火影之后，
情况就完全不同了，头文件不仅分散，还特别多,补全效果大打折扣。

```bash
flags = [
'-Wall',
'-Wextra',
'-Werror,
...
'-isystem',             # 系统路径
'/usr/include/c++'
'-I',                   # 用户路径
'./libsrc',
```

## *读取compile_commands.json文件*
> 这种是通过cmake生成compile_commands.json文件，然后.ycm_extra_conf读取该json文件，
准确的分析每一个源文件所在目录，因而补全十分精准，理论上补全率100%，基本上不会出现
之前的部分补全的情况了。而且刚好kihan项目用cmake编译,天助我也。具体操作如下：
### cmake设置
cmake设置有2中方式:
- 设置临时参数

```bash
cd <ProjectRoot>/build
cmake .. -DCMAKE_EXPORT_COMPILE_COMMANDS=1
```
- 写CMakeList.txt文件
在CMakeList配置文件中加一条配置
`set (DCMAKE_EXPORT_COMPILE_COMMANDS 1)`
这2中方式都能生成compile_commands.json文件

### 设置compilation_database_folder
`compilaion_database_folder = './build'`
最后，打开cpp文件，执行:YcmRestartServer命令，生成.ycm_extra_conf.pyc文件，便大功告成了，
哈哈，好好享受YCM快捷的语义补全吧