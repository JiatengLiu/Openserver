# TJDXT：
1. Remote debugging can take up resources for a long time during debugging, so stop the debugging process when you are finished debugging
2. This part of the functionality has not been updated after the server system update, for which the guidance is marked as deprecated
## 1、Using VSCode for remote debugging
## 一、GUIDE
### 1、Local VSCode requires the following extensions
- Remote - SSH
- Remote - SSH: Editing Configuration Files
- Remote - Development
- Remote - Remote Explorer
### 2、On the left side of the taskbar choose ` remote resource manager ` - > ` remote (tunnel/SSH) ` - > ` SSH option on the right side ➕ ` - > ` SSH username @ < IP > -p < PORT > ` - > enter the password after the connection is successful
### 3、Select 'Explorer' from the left taskbar or 'ctrl+shift+e' and select Open Folder to open the corresponding project folder
#### Note :VSCode classifies different folders into different classes, which means that the folders A and B opened here will occupy two different options in the local 'SSH' options
### 4、Server VSCode needs to install the following extensions (install the same as local, select 'Extensions' on the left or' ctrl+shift+x 'shortcut to open)
- Docker
- Python
- Pylance
- Python Debugger
### 5、After installation, the 'docker' icon will appear on the left (like this whale - 🐋)
### 6、Open 'Remote Explorer' again, select 'Remote (tunnel /SSH)' and change it to 'Development Container', look for your container, click '➡' on the right side and enter the password to enter the corresponding container
### 7、Place the code to be debugged in the current window
### 8、Set breakpoints - > click on run ` ` - > ` start debugging ` - > ` corresponding parameters on the command line input ` - > start the Debug operation
