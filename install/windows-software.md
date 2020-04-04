# Windows 常用软件

### ① Notepad++

下载链接：[Notepad++](https://notepad-plus-plus.org/downloads/)

> 可以视为文本文档编辑器的替代品

### ② Typora

下载链接：[typora-windows](https://typora.io/#windows)

> markdown编辑器

右键直接新建markdown文件快捷方式（内容来自：[keavnn的博客](https://stepneverstop.github.io/win-rightclick-create-md.html)）：

1. 先安装typora

2. 新建一个文本文件（最好用notepad打开），在里面写入下述内容

   ```
   Windows Registry Editor Version 5.00
   [HKEY_CLASSES_ROOT\.md]
   @="Typora.exe"
   [HKEY_CLASSES_ROOT\.md\ShellNew]
   "NullFile"=""
   [HKEY_CLASSES_ROOT\Typora.exe]
   @="Markdown"
   ```

3. 然后另存为`xxx.reg`（文本文档）

4. 之后双击即可

### ③ shadows-ssr

下载地址：[shadowsocksr-windows](https://github.com/HMBSbige/ShadowsocksR-Windows/releases)

> 代理工具—与SwitchyOmega搭配使用效果更佳

### ④ github-desktop

下载地址：[Github Desktop](https://desktop.github.com/)

> Github桌面版本

### ⑤ CUDA安装

