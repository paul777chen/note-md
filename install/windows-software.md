# Windows 常用软件

### 1. Notepad++

下载链接：[Notepad++](https://notepad-plus-plus.org/downloads/)

> 可以视为文本文档编辑器的替代品

### 2. Typora

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

### 3. shadows-ssr

下载地址：[shadowsocksr-windows](https://github.com/HMBSbige/ShadowsocksR-Windows/releases)

> 代理工具—与SwitchyOmega搭配使用效果更佳

### 4. github-desktop

下载地址：[Github Desktop](https://desktop.github.com/)

> Github桌面版本

### 5. CUDA安装



### 6. 包管理

> 类似利用linux下sudo apt install xxx的方式来按照一些库

利用管理员身份运行powershell

1. 输入：`Get-ExecutionPolicy`，如果显示`Restricted`，则输入`Set-ExecutionPolicy AllSigned`

2. 输入：

   ```
   Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))
   ```

3. 按回车等待安装完成即可

> 以后要安装什么包`choco install xxx`即可

