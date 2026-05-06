---
publish: true
tags:
  - 代码小记
date: 2025-12-08 00:16:00
---
﻿
突然想给vsc背景加张图片  
下载了 Custom CSS and JS Loader的插件  
这个插件专门用来加载自定义的css和js文件 
然后用ai写了个css文件   
```css
/* VS Code背景图：左下角局部显示 + 低存在感 */
body {
 /* ./images/3084836-20251207204854564-1083380884.png */
  background-image: url("./images/3084836-20251207201446453-1671085419.jpg");
  background-size: 30% auto; /* 图片宽度占窗口30%，高度自适应 */
  background-position: left bottom 30px; /* 左下角，距离底部30px */
  background-repeat: no-repeat; /* 禁止平铺 */
  background-attachment: fixed; /* 滚动编辑器时图片固定不动 */
  background-color: transparent; /* 不覆盖VS Code原有主题背景 */
  background-blend-mode: soft-light; /* 图片与背景柔和混合，降低存在感 */
  opacity: 0.98; /* 整体微透，几乎不影响界面 */
}

/* 确保代码编辑区不被遮挡，完美适配Solarized Light主题 */
.monaco-editor .editor-container {
  /* Solarized Light主题原生底色偏浅黄（#fdf6e3），匹配主题色+半透 */
  background-color: rgba(253, 246, 227, 0.9) !important; 
  opacity: 0.98; /* 微透保留背景图质感，不遮挡代码 */
  /* 额外适配：避免主题样式被覆盖，继承原生边框/阴影 */
  background-blend-mode: normal;
}

/* 补充：适配编辑器内容区的背景，和主题完全统一 */
.monaco-editor .editor-background {
  background-color: inherit !important;
}

/* 补充：保留配置的光标颜色，避免被样式覆盖 */
.monaco-editor .cursors-layer .cursor {
  border-color: #3ed130 !important; /* 和你settings.json里的editorCursor.foreground一致 */
}
```


在用户配置下增加如下区域  
```json
  //供给 给 vsc背景板css使用
  "vscode_custom_css.imports": [
  "file:///E:/apps/vscode/vsc_css/vsc_bg.css" 
  ],
```
- - -
ctrl+shift+p 打开命令面板  
执行Enable Custom CSS and JS  
重启vsc  

重启后发现vsc有“code安装似乎损坏，请重新安装”的error提示    
原因是往vsc注入自定义css样式后，修改了vsc的原生界面，  
vsc会定期检查自己的核心文件是否和官方发布的原始版本完全一致，一旦发现文件被修改，就会判定[文件完整性被破坏]  
- - -
上网查找下载了“fix vscode checksums next”插件后  
ctrl+shift+p 打开命令面板  
Fix Checksums: Apply  
重启vsc  

没有烦人error提示了    
- - -![img](3084836-20251208001302927-633161619.png))
好吧太淡了，调了好几遍，发现我的头像底色白色太明显就和主题冲突了，就这样吧，淡淡的就好  
（感觉太淡了）  
