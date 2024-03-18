# 写在前面
## 1、对于所有的操作并没有做严格性限制，所以请严格按照指南进行操作。因个人不当操作导致服务器宕机或配置出现问题，后果自己负责！
## 2、对于服务器显卡使用问题。
### `单卡`任务**禁止**分布到`多卡`——会影响其他人正常使用。
### `单卡`**禁止**部署`多个任务`。

## 3、docker容器问题。
### 单个用户容器数量**上限**为`2`
### **禁止**重复多次实行`容器创建`指令 `docker run -it xxxx`
### 单个容器**运行大小**大小上限为`20g`
### **定期清理**不满足要求的容器
# 用户使用指南
## 一、本地配置SSH
### 1、参考[知乎专栏](https://zhuanlan.zhihu.com/p/391373172)在windows中开启SSH服务。
### 2、推荐使用[MobaXterm](https://mobaxterm.mobatek.net/download.html)与服务器进行SSH连接。
### 3、登录过程中IP为：(xx.xxx.xxx.xxx)[jiatliu@163.com] ，端口号为：(xxxxx)[jiatliu@163.com]。用户名：(xxxxx)[jiatliu@163.com]，密码：(xxxxx)[jiatliu@163.com]。
#### IP地址，端口号，用户名与密码出于安全考虑隐藏，有需要请发邮件联系我。
#### &emsp;&emsp; 请联系管理员添加用户。
### 4、连接成功后，系统自动跳转至个人目录下，同时终端输出像素UI，表示正常登录
## 二、创建docker容器
### 1、查看服务器docker镜像
```
docker images # 查看服务器镜像

# DOCKER_IMAGES_NAME对应镜像名称
docker ps -a --filter ancestor=DOCKER_IMAGES_NAME # 查看DOCKER_IMAGES_NAME这个镜像创建的容器信息
```
### 2、选择合适镜像创建docker容器
#### 目前服务器配置有cuda 11.0 / 11.6 / 11.8 / 12.0，所有的镜像配有anaconda以及基本使用软件，自行选择使用。
#### 下面给出一般格式和常用的指令，大家可以参考修改使用
```
# 通用格式--不可直接使用，需要替换其中的变量
# memory_limit：内存限制 container_name：个人容器名，自行设置  
# host_directory：宿主机挂载目录 container_directory：容器挂载目录
# host_port：宿主机端口 contain_port：容器映射端口 ---主要用作jupyter使用，非必需不要占用
# shm-size：shm内存上限
# image_name：docker镜像名 例：nvi/cu:118


docker run -it --gpus all --memory <memory_limit> --name <container_name> -v <host_directory>:<container_directory>   -p <host_port>:<contain_port> --shm-size 4g <image_name>
```
#### 使用实例
```
# 这里我使用nvi/cu118创建一个需要所有gpu ，运行内存为8G，挂载到我个人目录，shm内存为4g，名字为ljt的容器
docker run -it --gpus "device=all" --memory 8g --name ljt -v /data/ljt:/data  --shm-size 4g nvi/cu:118
```
#### 注：docker run 指令只是完成容器创建指令，执行一次即可，不需要每次使用时执行。
### 3、docker容器参数
#### 参考[知乎专栏](https://zhuanlan.zhihu.com/p/671793715)

### 4、附录
## 三、使用dockers容器
### 1、进入容器
```
# 进入contain_name容器并开启bash终端
docker exec -it <contain_name> bash 
```
### 2、进入个人数据目录
#### 个人数据默认挂载在docker的 `/data` 目录下。
```
# 进入docker中的个人目录
cd /data
```
### 3、git
#### 服务器中每个镜像配置了git，可以直接使用。因为服务器机房中存在多个设备使用网络，有时VPN较慢导致git速度慢，正常现象。
```
git clone <project_url>

# clone submodule
git clone <project_url> --recursive

# git submodule
cd <project_dir> 
git submodules update --init --recursive
```
### 4、anaconda环境配置
#### 容器创建成功后默认配置anaconda，使用如下指令检验
```
# 查看conda版本
conda --version
# 查看conda环境
conda env list
# 进入conda环境
source activate <env_name> # 使用conda activate会报错
```
#### 创建环境和平时使用一致。下面给出几种方式。
```
# 创建空环境
conda create -n <env_name> python==PYTHON_VERSION


# 一般项目会给requirement.txt或environment.yml
# 因为服务器配置国内源后使用VPN下载软件包导致证书问题，所以在使用文件创建环境时请首先检查一下文件中的channels部分。更换为国内镜像或直接删除，如果能正常运行则忽略。
# environment.yml
conda env create --file environment.yml

# requirement.txt
conda create -n <env_name> python==PYTHON_VERSION
pip install -r requirement.txt


# 手动安装
conda create -n ENV_NAME python==PYTHON_VERSION
conda install <package_name>==PACKAGE_VERSION
```
#### 注：因为pip国内源未找到合适的替代，所以仍然使用tuna源，也可自行搜索替换。conda已经替换为ali源，不建议更换。
### 5、运行项目
#### 5.1 终端运行
#### &emsp;&emsp; 直接在个人终端执行指令——ssh断开后任务自动终止
```
# 对于短期任务/保证ssh不会断开的情况使用
python <command>
```
#### 5.2 终端挂载
#### &emsp;&emsp; 将任务挂载在容器后台——ssh断开后任务继续运行
```
# 适合长期任务/断开ssh连接

nohup command > /data/LOG/filename.log 2>&1 &
# 其中 command代表需要执行的python指令，/data/LOG/filename.log表示运行日志保存目录，其他请不要修改。

# 如何查看后台运行的任务
tail -f /data/LOG/filename.log

# 查看过程中不影响后台任务直接退出：ctrl + C
```
#### 5.3 服务器挂载
#### &emsp;&emsp; 将任务挂载在服务器后台——ssh断开后任务继续运行
#### &emsp;&emsp;① 更改配置文件
```
vi cfg/xxx.yaml
# 下面是一个使用实例，可以按照这个修改。其中LOG_DIR最后一级目录LOG不建议修改，它位于个人目录下。
# GPUS关系到提交任务的存放位置，请格外注意格式。从小到大排序
# GPU_NUM  GPUS两者关系要对应，否则任务会被删除

ENVIRONMENT:
  IMAGES:nvi/cu116
  CONTAIN:cuda116
  NAME:gaussian_splatting
TASK_CONFIG:
  PROJECT_DIR:gaussian-splatting
  GPU_NUM:3
  GPUS:012
  COMMAND:python render.py -s tandt_db/tandt/truck -m models_download/truck/
  LOG_DIR:/data/gaussian-splatting/LOG
```
#### &emsp;&emsp;&emsp;&emsp;如果想创建新的yaml文件，请使用`cp x.yaml y.yaml`指令
#### &emsp;&emsp; ② 使用`./.submit.sh .cfg/xxx.yaml`提交任务
#### &emsp;&emsp; ③ log输出日志位于yaml文件中的LOG_DIR中，文件以“时间_项目名.txt”存放。

### 6、退出/删除容器
```
# 删除容器
docker rm <contain_name>

# 退出
exit
```
## 四、附录
### docker常用指令
```
# 查看镜像
docker images / docker ps -a
# 查看某个镜像下的所有容器
docker ps -a --filter ancestor=<image_name>
# 查看某个镜像下正在运行的容器
docker ps --filter ancestor=<image_name>

# 启动某个容器
docker start <容器名>
# 退出某个容器
exit
# 停止运行某个镜像下的某个容器
docker stop <容器名>

# 删除某个镜像下的某个容器
docker rm <容器名>
# 删除镜像
docker rmi <镜像名>

# 进入某个正在运行容器
docker exec -it <容器名> bash
#进入某个停止运行的容器
docker start <容器名>
docker exec -it <容器名> bash

# 输出某个容器的详细信息
docker inspect <容器名>
```
### 常见问题汇总
```
1、Q：如何往服务器中传输文件
A： 1> 直接将文件拖至MobaXterm左侧目录栏
    2> `scp local_file     remote_username@remote_ip:remote_folder`

2、Q：如何从服务器中下载文件到本地
A： MobaXterm左侧目录栏对应文件右键`download`

3、Q：服务器权限问题
   A： 用户权限组为"docker sudo plugdev gh",容器中权限为"root"，如果存在上传文件显示权限不足错误，这是因为该目录是以容器中的root创建的。使用`chmod -R 777 <folder>`更改目录权限

4、Q：服务器中编辑代码
   A： MobaXterm右键文件`Open with ...`选择本地软件。

5、报错信息：
`
Error response from daemon: Cannot restart container hf: failed to create task for container: failed to create shim task: OCI runtime create failed: runc create failed: unable to start container process: error during container init: error running hook #0: error running hook: exit status 1, stdout: , stderr: Auto-detected mode as 'legacy'
nvidia-container-cli: initialization error: nvml error: driver/library version mismatch: unknown
`
   A:是因为显卡内核更新导致版本不一致而报错。
      ① 在宿主机上使用`nvidia-smi`指令查看宿主机上显卡内核版本是否正常。输出正常即可。一般情况宿主机显卡内核不会更新，已经手动禁止。
      ② 宿主机正常后，重新启动容器 docker restart <contain_name> 即可。
      ③ 宿主机驱动输出异常，联系管理员。
6、Q：opencv导入报错
   A： 使用如下方法安装opencv以及opencv-contrib
`
pip uninstall opencv-python opencv-contrib-python
`
`
pip install opencv-python-headless opencv-contrib-python-headless
`

7、Q:报错信息
`
CondaError: Downloaded bytes did not match Content-Length
  url: https://conda.anaconda.org/pytorch/linux-64/pytorch-2.2.1-py3.10_cuda11.8_cudnn8.7.0_0.tar.bz2
  target_path: /opt/conda/pkgs/pytorch-2.2.1-py3.10_cuda11.8_cudnn8.7.0_0.tar.bz2
  Content-Length: 1636041382
  downloaded bytes: 305994140
`
   A:重新执行安装指令，可能是conda官方的问题。该问题多出现在同时安装torch和torchvision时或第一次安装torch时
