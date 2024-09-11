# 私有云远程桌面配置说明

 **By 蒋旭**

> 由于远程桌面的配置繁琐以及体验一般，如果您不是强需求，推荐您对代码做相应调整(比如opencv的imshow改为imsave保存图片后浏览)

VNC（Virtual Network Computing ）是一种图形化的桌面共享协议，它使用远程帧缓冲协议 (RFB) 来远程控制另一台计算机，它将键盘和鼠标事件从一台计算机传输到另一台计算机，通过网络向另一个方向转发图形屏幕更新。我们可以在实例中手动安装图形化界面，并安装vncserver，从而实现连接容器实例的图形化界面。

## 配置实例

### 更新源

```shell
apt update
```

### 安装图形化桌面

```shell
apt install xorg xdm xfce4
```

### 安装基本的依赖包

```shell
apt update && apt install -y libglu1-mesa-dev mesa-utils xterm xauth x11-xkb-utils xfonts-base xkb-data libxtst6 libxv1
```

### 安装libjpeg-turbo和turbovnc

```shell
# 安装libjpeg-turbo和turbovnc
export TURBOVNC_VERSION=2.2.5
export LIBJPEG_VERSION=2.0.90
wget http://aivc.ks3-cn-beijing.ksyun.com/packages/libjpeg-turbo/libjpeg-turbo-official_${LIBJPEG_VERSION}_amd64.deb
wget http://aivc.ks3-cn-beijing.ksyun.com/packages/turbovnc/turbovnc_${TURBOVNC_VERSION}_amd64.deb
dpkg -i libjpeg-turbo-official_${LIBJPEG_VERSION}_amd64.deb
dpkg -i turbovnc_${TURBOVNC_VERSION}_amd64.deb
rm -rf *.deb
```

### 使用自带torch的镜像时，需要做如下改动，miniconda镜像不需要

```shell
rm -rf ~/.vnc
mkdir ~/.vnc
touch ~/.vnc/xstartup.turbovnc
chmod +x ~/.vnc/xstartup.turbovnc
vim ~/.vnc/xstartup.turbovnc
```

内容如下：

```shell
#!/bin/sh
xrdb $HOME/.Xresources
unset SESSION_MANAGER

unset DBUS_SESSION_BUS_ADDRESS

[ -x /etc/vnc/xstartup ] && exec /etc/vnc/xstartup

[ -r $HOME/.Xresources ] && xrdb $HOME/.Xresources

vncconfig -iconic &

xfce4-session & startxfce4 & 
```


### 启动VNC服务端

这一步可能涉及vnc密码配置（注意不是实例的账户密码）。另外如果出现报错xauth未找到，那么使用apt install xauth再安装一次

```shell
rm -rf /tmp/.X1*  # 如果再次启动，删除上一次的临时文件，否则无法正常启动
USER=root /opt/TurboVNC/bin/vncserver :1 -desktop X -auth /root/.Xauthority -geometry 1920x1080 -depth 24 -rfbwait 120000 -rfbauth /root/.vnc/passwd -fp /usr/share/fonts/X11/misc/,/usr/share/fonts -rfbport 6006
```

### 检查是否启动

```shell
# 如果有vncserver的进程，证明已经启动
ps -ef | grep vnc
```

## 使用VNC客户端连接

### 获取连接地址

点击对应容器实例的“端口与服务”，复制自定义服务中6006端口服务访问地址

<img src=".\image-20240911133314709.png" alt="image-20240911133314709" style="zoom:50%;" />

### 使用VNC客户端连接

#### 选项1：在realvnc viewer中连接

在realvnc viewer中输入地址，按回车。

<img src=".\image-20240911133517990.png" alt="image-20240911133517990" style="zoom:50%;" />

点击Continue

<img src=".\image-20240911133601811.png" alt="image-20240911133601811" style="zoom: 50%;" />

并输入首次启动VNC服务端时设置的密码即可连接

<img src=".\image-20240911133617929.png" alt="image-20240911133617929" style="zoom:50%;" />

#### 选项2：在MobaXTerm中连接

选择VNC选项，分别输入相应的IP和端口号，点击ok，并输入密码即可连接

<img src=".\image-20240911133742589.png" alt="image-20240911133742589" style="zoom:80%;" />

### 简单验证

可以使用以下python代码进行简单验证：

```python
import numpy as np
import cv2

h = 500
w = 500
img = 255 * np.ones((h ,w , 3), dtype=np.uint8)
cv2.imshow("", img)
cv2.waitKey(0)

```

如果在本地的vnc client显示图片，证明安装和启动过程无误

<img src=".\image-20240911133933585.png" alt="image-20240911133933585" style="zoom:50%;" />
