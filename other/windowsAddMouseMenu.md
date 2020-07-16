# windows给鼠标右键添加菜单

## 注册表

```
1·
win+R 弹出对话框

2·
regedit
```

### 注册表中选择路径

```
计算机\HKEY_CLASSES_ROOT\Directory\Background\shell
```

### 具体操作

![image-20200716111428685](.\1.png)

![image-20200716111837034](.\2.png)

### 添加内容

应用程序的Icon路径和应用程序命令行的路径是相同的

但以WindowsTerminal为例，需要加上一些额外的参数

Icon路径

```
C:\Program Files\WindowsApps\Microsoft.WindowsTerminal_1.0.1811.0_x64__8wekyb3d8bbwe\WindowsTerminal.exe
```

命令行路径

```
C:\Program Files\WindowsApps\Microsoft.WindowsTerminal_1.0.1811.0_x64__8wekyb3d8bbwe\WindowsTerminal.exe -d "%V"
```

## windowsApp打开姿势

实际上，对于大多数的软件，不存在这个问题但对于windows自己安装的软件（从应用商店）中，windowsApps是无法进行访问的。

所以很难获取到具体软件的位置.

所以要进行一些权限设置

### 第一次打开属性，设置用户

![image-20200716113231733](.\3.png)

![image-20200716113325112](.\4.png)

![image-20200716113409089](.\5.png)

![image-20200716113509893](.\6.png)

![image-20200716113629793](.\7.png)

![image-20200716113726597](.\8.png)

### 第二次打开属性，设置用户权限

![image-20200716113842432](.\9.png)