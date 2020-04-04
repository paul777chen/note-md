# Windows系统安装

##  1. 系统安装

主要来源于：[win10安装](https://www.zhihu.com/question/39207359)

> 其实直接参考：

1. 进入[windows10下载官网](https://www.microsoft.com/zh-cn/software-download/windows10)
2. 点击立即下载工具：就会下载一个`MediaCreationTool1909.exe`的安装工具
3. 双击，进去后：
   - 选择**“为另一台电脑创建安装介质”**
   - 然后选择一个至少8G以上的U盘，后续自动就会弄好了的
4. 在需要装系统的电脑上面插入U盘，并重启进入BIOS，启动的时候选择安装了系统的U盘即可；后续按照步骤进行就好了！

## 2. 激活

自己在网上搜索一些激活码

## 3. Office安装和激活

1. 在[MSDN-I tell you](https://msdn.itellyou.cn/)上面下载Office2019

2. 按照步骤安装即可

3. 个人激活按照[SoulVen的教程](https://zhuanlan.zhihu.com/p/91328420)完成的

   - 新建txt文档，粘贴下面的代码

     ```
     @echo off
     (cd /d "%~dp0")&&(NET FILE||(powershell start-process -FilePath '%0' -verb runas)&&(exit /B)) >NUL 2>&1
     title Office 2019 Activator r/Piracy
     echo Converting... & mode 40,25
     (if exist "%ProgramFiles%\Microsoft Office\Office16\ospp.vbs" cd /d "%ProgramFiles%\Microsoft Office\Office16")&(if exist "%ProgramFiles(x86)%\Microsoft Office\Office16\ospp.vbs" cd /d "%ProgramFiles(x86)%\Microsoft Office\Office16")&(for /f %%x in ('dir /b ..\root\Licenses16\ProPlus2019VL*.xrm-ms') do cscript ospp.vbs /inslic:"..\root\Licenses16\%%x" >nul)&(for /f %%x in ('dir /b ..\root\Licenses16\ProPlus2019VL*.xrm-ms') do cscript ospp.vbs /inslic:"..\root\Licenses16\%%x" >nul)
     cscript //nologo ospp.vbs /unpkey:6MWKP >nul&cscript //nologo ospp.vbs /inpkey:NMMKJ-6RK4F-KMJVX-8D9MJ-6MWKP >nul&set i=1
     :server
     if %i%==1 set KMS_Sev=kms7.MSGuides.com
     if %i%==2 set KMS_Sev=kms8.MSGuides.com
     if %i%==3 set KMS_Sev=kms9.MSGuides.com
     cscript //nologo ospp.vbs /sethst:%KMS_Sev% >nul
     echo %KMS_Sev% & echo Activating...
     cscript //nologo ospp.vbs /act | find /i "successful" && (echo Complete) || (echo Trying another KMS Server & set /a i+=1 & goto server)
     pause >nul
     exit
     ```

   - 将文档后缀.txt更改为.bat

   - 以管理员运行该文件， 等待即可

> 注：是否存在什么风险，不太确定