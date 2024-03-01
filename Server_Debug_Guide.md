# 写在前面
## 1、使用VSCode进行远程调试
## 一、教程
### 1、本地VSCode需要安装如下拓展
- Remote - SSH
- Remote - SSH: Editing Configuration Files
- Remote - Development
- Remote - Remote Explorer
### 2、左侧任务栏选择`远程资源管理器` --> `远程(隧道/SSH)` --->  `SSH选项右侧➕`--->`ssh username@10.160.72.107 -p 420` ---> 输入密码后连接成功
### 3、左侧任务栏选择`资源管理器`或`ctrl+shift+e`选择打开文件夹打开对应项目文件夹
#### 注:VSCode将不同的文件夹归为不同的类,也就是说这里打开的文件夹A和B会在本地`SSH`选项中占据两个不同的选项位置
### 4、服务器VSCode需要安装如下拓展(安装方法和本地一致,选择左侧`拓展`或`ctrl+shift+x`快捷键打开)
- Docker
- Python
- Pylance
- Python Debugger
### 5、安装完成后左侧会出现`docker`图标(like this whale - 🐋)
### 6、再次打开`远程资源管理器`,在选择`远程(隧道/SSH)`位置将其更改为`开发容器`,在其中寻找自己的容器,点击右侧`➡`并输入密码进入对应容器
### 7、将需要调试的代码放置在当前窗口中
### 8、设置断点 ---> 点击`运行` ---> `启动调试` ---> `在弹出的命令行中输入对应的参数` ---> 开始Debug操作
