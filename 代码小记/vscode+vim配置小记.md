---
publish: true
tags:
  - 代码小记
date: 2025-01-03 14:16:00
---
﻿# 引入
在windows系统下使用vscode+vim编写代码时会遇到一个令人略有不爽的小麻烦。  
在vim的normal模式下，首先需要进入insert模式才能正常编写。这里一般是在英文输入法键入相应字母才能进入，比如“i”和“o”  
我们进入insert模式之后，在敲代码的过程中难免会需要增加些中文注释，这个时候我们的输入法就从英文切换到中文了。  
按esc退回normal模式时，输入法却保留在了中文输入法，这时想再进入insert模式就需要再次切换到英文输入法，我经常忘记这回事就经常被输入法卡住。  

# 解决
终于有意上网查找解决这个不大不小的麻烦的方法： 
以下是我自己的基于windows系统下对于vscode+vim插件关于自动切换输入法的解决方案小结   
>使用im-select插件  
>github链接：<https://github.com/daipeihust/im-select>  
* 在github上下载项目到本地，将项目内的exe执行文件移到特定路径下，可自定，最好是不会被轻易清理的路径。  ![img](3084836-20241230223937960-1920501147.png))![img](3084836-20241230224008167-743238921.png))
* 插件作者建议windows用户使用git的git-bash来查看不同输入法的句柄值，由于我的电脑下载了git，这里就没有额外配置了
* 打开git-bash,进入我刚刚放置im-select.exe的路径，在不同的输入法状态下分别执行bash命令
```bash
./im-select.exe
```
即可查看当前输入法的句柄值，我这里分别是2051和1033,分别对应微软拼音和美式键盘  ![img](3084836-20241230224706688-2070171022.png))  
* 接下来在vscode的settings.json文件中添加以下配置
```json
   // 自动切换输入法
  "vim.autoSwitchInputMethod.enable": true,
  "vim.autoSwitchInputMethod.defaultIM": "1033",  // 这里输入刚刚获得的英文输入法名称
  "vim.autoSwitchInputMethod.obtainIMCmd": "D:\\apps\\tools\\im-select.exe",
  "vim.autoSwitchInputMethod.switchIMCmd": "D:\\apps\\tools\\im-select.exe {im}&& D:\\apps\\tools\\im-select.exe 2052",

```
其中`"vim.autoSwitchInputMethod.switchIMCmd": "D:\\apps\\tools\\im-select.exe {im}",`即表示按esc退出insert模式时，自动切换到句柄为1033的英文输入法   但是来回切换不同输入法不如在一个输入法内切换中英文来的方便。  
所以我在代码后增加了`&& D:\\apps\\tools\\im-select.exe 2052`，表示同时切换到微软拼音，在系统设置里面将微软拼音的默认语言改为英语，这样在切换微软拼音时，微软拼音会自动切换为英文。  
虽然有点曲折，不过实现效果是达到了的。  
