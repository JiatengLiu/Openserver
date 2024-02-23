# 写在前面
### 1、服务器使用docker进行环境管理，禁止在宿主机中使用anaconda运行代码，统一使用docker创建个人容器进行操作。原系统中的conda虚拟环境请尽快迁移到docker中，或在docker中重新配置环境。环境迁移见[环境迁移](#transfer)。
### 2、docker中配置有多个版本的cuda-toolkit镜像，日常使用中将nvi/cu118作为主镜像，默认使用nvi/cu118镜像创建容器。若项目对于cuda版本有严格要求，请使用其他合适镜像，详情请参考[附属镜像使用](#subsidiary)[未完成]。
### 3、对于docker的解释可以参考[官方文档](https://docs.docker.com/get-started/)。简而言之，docker可以分为三层：宿主机，镜像，容器。宿主机是真实存在的物理主机；镜像相当于封装了一定依赖的软件系统，容器是对镜像的使用，进入使用某个镜像创建的容器之后就可以像在宿主机上操作一样了，但是因为镜像的原因，在容器内进行任何操作均不会影响宿主机的运行，这就是为什么选择使用docker而非anaconda管理环境的原因。
### 4、服务器使用期间需要大家共同努力维护，出现的bug请及时联系我(jiatliu@163.com)
### 5、如果不想了解服务器的搭建思路和流程，可直接跳转至[用户指南](#guide)
# 一、服务器配置综述
## 1、概述
### 服务器上将存储空间分为2部分：docker区（dockerzone）、数据区（datazone）。docker区中存放镜像和容器（dockerzone/image）（dockerzone/contains）。数据区分为两类：个人数据区（selfdatazone）和数据集区（datasetzone），个人python项目和配置文件放在个人数据区，训练使用的数据集放在数据集区（只读）。
### 服务器配置的docker镜像有：cuda118 cuda116 cuda113 cuda110
# 二、服务器SSH配置
### 安装openssh-server 和 openssh-client
```
sudo apt-get install openssh-server
sudo apt-get install openssh-client
```
### 启动SSH并设置SSH开机自启
```
sudo /etc/init.d/ssh start
sudo systemctl enable ssh
```
### 添加用户并进入分组
```
su root
useradd -m username
# or
# useradd  username
# chown username:username -R /home/user_dir
# 期间会设置密码，请牢记密码
usermod -a -G groupname username
# 将所有用户权限设置为跟 gh 一致
```
### 配置SSH配置文件 -->末尾添加内容如下
```
AllowUsers username1 username2 ...
```
### 获取服务器IP地址
```
ifconfig or hostname -I
```
### 更改用户默认主目录
```
# 注意给予bash而不是sh
usermod -d /new/home/directory  -s /bin/bash john
```
### 界面UI
```
# in .bash_profile
clear
figlet -c -f standard "WELCOME" | lolcat
figlet -c -f standard "NJUPT" | lolcat
figlet -c -f standard "AI & AUTO LAB 420" | lolcat
```

### 完整流程 
```
username=xxx
useradd -m $username
usermod -a -G gh $username
usermod -a -G sudo $username
usermod -a -G plugdev $username
usermod -a -G docker $username
usermod -d /data/$username  -s /bin/bash $username
mkdir /data/$username
chmod 777 /data/$username
cp /data/ljt/.get.sh /data/$username/.get.sh
cp /data/ljt/.submit.sh /data/$username/.submit.sh
cp /data/ljt/.bash_profile /data/$username/.bash_profile
cd /data/$username && mkdir LOG && mkdir cfg

chmod 777 LOG
chmod 777 cfg
cp /data/ljt/cfg/default.yaml /data/$username/cfg/default.yaml
passwd $username


ip 10.160.72.107 username dxy password XKL420dxy port 420
```

# 二、<span id=user>客户端SSH配置
## windows系统安装MobaXterm
### 下载地址 
```https://mobaxterm.mobatek.net/download.html```
### 下载free即可。安装完成后点击`Session`,选择`SSH`链接
#### Remote host中输入服务器ip地址；username输入创建的用户名。输入完成后点击OK。
#### 注：期间需要输入密码等操作，按照提示进行。
#### 校园局域网互联，不需要服务器和用户连接在同一个局域网下
#### 至此，完成windows客户端与服务器的连接。
### 文件传输
#### 直接将需要传输的文件拖入左侧的目录即可。

# 三、服务器docker配置
## 1、配置docker
### 首先，更新软件包索引，并且安装必要的依赖软件，来添加一个新的 HTTPS 软件源：
```
sudo apt update
sudo apt install apt-transport-https ca-certificates curl gnupg-agent software-properties-common
```
### 使用下面的 curl 导入源仓库的 GPG key：
```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```
### 将 Docker APT 软件源添加到你的系统：
```
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
```
### 现在，Docker 软件源被启用了，你可以安装软件源中任何可用的 Docker 版本。
### 安装 Docker 最新版本，运行下面的命令。
```
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io
```
### 一旦安装完成，Docker 服务将会自动启动。你可以输入下面的命令，验证它：
```
sudo systemctl status docker
```
### 输出将会类似下面这样：
```
● docker.service - Docker Application Container Engine
     Loaded: loaded (/lib/systemd/system/docker.service; enabled; vendor preset: enabled)
     Active: active (running) since Thu 2020-05-21 14:47:34 UTC; 42s ago
...
```
### 当一个新的 Docker 发布时，你可以使用标准的`sudo apt update` && `sudo apt upgrade`流程来升级 Docker 软件包。
### 如果你想阻止 Docker 自动更新，锁住它的版本：
```
sudo apt-mark hold docker-ce
```
### 注：默认情况下，只有 root 或者 有 sudo 权限的用户可以执行 Docker 命令。想要以非 root 用户执行 Docker 命令，你需要将你的用户添加到 Docker 用户组，该用户组在 Docker CE 软件包安装过程中被创建。想要这么做，输入：
```
# $USER是一个环境变量，代表当前用户名。
sudo usermod -aG docker $USER
# 登出，并且重新登录，以便用户组会员信息刷新。
```
### 但上述方法需要确定所有的用户，较为麻烦。较好的替代方法是所有关于`docker`的指令使用`sudo`指令提权。
### 想要验证 Docker 是否已经成功被安装，你可以执行docker命令，前面不需要加`sudo, 我们将会运行一个测试容器:
```
docker container run hello-world
```
## 2、卸载docker
### 运行下面的命令停止所有正在运行的容器，并且移除所有的 docker 对象：
```
docker container stop $(docker container ls -aq)
docker system prune -a --volumes
```
### 卸载docker
```
sudo apt purge docker-ce
sudo apt autoremove
```
## 3、docker的使用
### 查看docker的镜像
```
docker images
# 下列指令可以查看当前宿主机内的所有镜像
docker ps -a
```
### 下载镜像
```
# 常用的docker镜像可以从如下地址下载 docker hub
https://hub.docker.com/
# 如果下载速度慢，可以选择如下地址
        "https://docker.m.daocloud.io",
        "https://dockerproxy.com",
        "https://docker.mirrors.ustc.edu.cn",
        "https://docker.nju.edu.cn"


# 宿主机将如上镜像地址写入系统
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
    "registry-mirrors": [
        "https://docker.m.daocloud.io",
        "https://dockerproxy.com",
        "https://docker.mirrors.ustc.edu.cn",
        "https://docker.nju.edu.cn"
    ]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker

```
### 对于nvidia镜像的显卡支持
#### 创建需要gpu的容器时，默认并不能创建成功，需要进行如下配置
```
# 1、新建脚本
# nvidia-container-runtime-script.sh

sudo curl -s -L https://nvidia.github.io/nvidia-container-runtime/gpgkey | \
  sudo apt-key add -
distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
sudo curl -s -L https://nvidia.github.io/nvidia-container-runtime/$distribution/nvidia-container-runtime.list | \
  sudo tee /etc/apt/sources.list.d/nvidia-container-runtime.list
sudo apt-get update
# ----------------------------------------------------------------

# 2、执行脚本
sudo sh nvidia-container-runtime-script.sh

# 3、安装
sudo apt-get install nvidia-container-runtime

# 4、验证
which nvidia-container-runtime
#理论输出为：/usr/bin/nvidia-container-runtime

# 5、重启docker
sudo systemctl restart docker
```
### 在容器中重新安装cuda sdk
#### 一般不建议直接进行cuda版本的更改，这是一个效率很低而且很累的方法。
#### 如果当前cu118版本没办法正确使用，请参考[附属镜像使用](#subsidiary)。



### 容器默认保存地址
#### 镜像默认保存地址：/var/lib/docker/image
#### 容器默认保存地址：/var/lib/docker/containers
### 默认保存地址的更改
```
# 停止 Docker 服务
sudo systemctl stop docker
sudo systemctl stop docker.socket
sudo systemctl stop containerd

# 移动 Docker 默认目录的所有内容
## 创建一个新目录：
sudo mkdir -p /new_dir
## 移动之前的数据：
sudo mv /var/lib/docker /new_dir

# 编辑配置文件
##/etc/docker/daemon.json 保存了 Docker 的配置：
sudo vim /etc/docker/daemon.json
## 更改 Docker 默认的数据存储位置，将以下内容添加到该文件中：
## 注意不要添加空格
{
  "data-root":"/new_dir/docker"
}
# 保存并关闭/etc/docker/daemon.json文件后，重新启动 Docker 服务:
sudo systemctl start docker
# 验证新的 Docker 根位置：
docker info -f '{{ .DockerRootDir}}'
# 完成之后，Docker 的 images 和 ps 命令应该一切正常。
```

### 使用已有的镜像创建容器并进入
```
# 使用选定的镜像创建一个新的容器。可以使用以下命令：
docker run -it --gpus all --memory <memory_limit> --name <container_name> -v <host_directory>:<container_directory> <image_name>

     -  --gpus all ：指定使用所有可用的GPU设备。 如：--gpus "device=1,2"[禁止创建容器使用的GPU数量超过宿主机GPU数量；或者使用宿主机未存在的GPU]
     -  --memory <memory_limit> ：指定容器的内存限制。例如，可以使用 --memory 8g 来限制容器使用的内存为8GB。 
     -  --name <container_name> ：为容器指定一个名称。 
     -  -v <host_directory>:<container_directory> ：指定要挂载的主机目录和容器内目录的映射关系。这允许您在容器中访问主机上的文件和目录。 

# 使用以下命令进入容器的交互式终端：
docker exec -it <container_name> bash
```
#### 上述指令使用实例
```
# 调用所有的gpus 8g的空间 my_container的容器名 
# 将/media/gh/data1/ljt这个目录映射到容器中的data目录（容器中的data的操作相当于对/media/gh/data1/ljt这个目录进行操作）
# 指明使用的镜像名为cuda118_torch1.8.1
docker run -it --gpus all --memory 8g --name my_container -v $/media/gh/data1/ljt:$/data cuda118_torch1.8.1
```
### 进入容器之后可以像在主系统中一样进行操作
### 进入一个之前已经创建好的容器中
```
# 运行指定的容器
docker start <容器名>
docker exec -it <容器名> bash
```

### 附：常用指令查询 提示权限不足加sudo
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
### 附录：镜像配置详情请参考
```
https://blog.csdn.net/m0_46680603/article/details/129338895
# 注意其中镜像源的修改和报错的rm指令的注释
```
# 四、<span id=transfer>环境迁移
#### 对于conda环境迁移，网络上比较常用的方法是将conda环境文件夹传输到容器对应位置，但这个方法很多时候并不能奏效，给出对应指令
```
# docker cp 宿主机文件/路径 容器名：容器内路径
docker cp /conda/file/in/host contain:/path/to/anaconda
```
#### 比较和兴的方法是使用conda生成环境配置文件，然后在docker中重新配置环境，这个方法通常不会出错。
```
# in host 
conda activate env / source activate env
conda env export > environment.yml
# in contain
conda create --file environemnt.yml / conda create -f environment.yml
```
#### 对于更好的解决办法会及时更新。 
# 五、服务器任务管理配置
## 服务器端任务执行内容包括：
### -任务扫描
### -任务执行
### -任务反馈
### -空间清理
### 主函数如下
```python
'''
该文件用于检测任务、执行任务、管理GPUs、存储日志、下放输出结果等。
任何人请不要对该文件进行更改！！！
'''
import time
from py3nvml.py3nvml import *
import os


def detection_of_GPUs():
    nvmlInit()
    freeNgpus,free_gpus = 0 , []
    device_count = nvmlDeviceGetCount()
    for i in range(device_count):
        handle = nvmlDeviceGetHandleByIndex(i)
        info = nvmlDeviceGetMemoryInfo(handle)
        # print(f'Device{i}:')
        # print(f'Total memory: {info.total/1024**2} MB')
        # print(f'Free  memory: {info.free/1024**2} MB')
        # print(f'Used  memory: {info.used/1024**2} MB')
        if info.free/info.total >= 0.85:
            freeNgpus += 1
            free_gpus.append(i)
    nvmlShutdown()
    return freeNgpus,free_gpus

def detection_of_tasks():
    pass

def get_runn_tasks():
    pass

def get_task(task_direc:list):
    ROOT = '/home/gh/tasks'
    for t in task_direc:
        path = os.path.join(ROOT,t)
        tasks = os.listdir(path)
        if len(tasks) > 0 :
            return os.path.join(path,tasks[0])
    return 'empty'


# config

task_direc = ['0123',
        '012','013','023','123',
        '01','02','03','12','13','23',
        '0','1','2','3']

def find_fill_task_direc(free_gpulist,task_direc):
# free  012
    def check_direc(tdirec):
        return all(char in free_gpulist for char in tdirec)
    return [tdirec for tdirec in task_direc if check_direc(tdirec)]



NUMBER_OF_GPUS = 4

if __name__ == '__main__':

    while(1):
        time.sleep(3)
        N,free_gpus = detection_of_GPUs()
        if N <= 0:
            print('NO FREE GPUS')
            continue
        print('THE NUMBER OF FREE GPUS IS {},AND THEY ARE: {}'.format(N,free_gpus))
        freegpulist = ''.join(map(str,free_gpus))
        task_direc_list = find_fill_task_direc(freegpulist,task_direc)
        
        ftask = get_task(task_direc_list)
        if ftask == 'empty':
            print("NO MATCHED TASKS")
            continue
        
        print("START THE TASK : {}".format(ftask))

        os.system("gnome-terminal -- "+"sh "+ftask)
        print("FINISH THE TASK : {}".format(ftask))
	
        os.remove(ftask)


```
# 六、客户端任务提交配置
## 配置工作
### 用户使用个人账户登录服务器后，编辑selfdatazone/cfg/xxxxx.yaml文件。yaml文件格式如下：
```yaml
# jfile
ENVIRONMENT:
  # 指定需要使用的docker镜像以及容器
  IMAGES:
  CONTAIN:
  # 容器中需要进入的conda环境名称
  NAME:
TASK_CONFIG:
  # 项目名称
  PROJECT_DIR:
  # 需要的gpu数量    
  GPU_NUM:
  # 需要的gpus 如：0 01 012 0123等
  GPUS:
  # 需要执行的python指令
  COMMAND:
  # 输出日志保存地址
  LOG_DIR:
```
### 编辑完成后的文件将作为本次任务的执行配置文件。允许存在多个执行配置文件，统一放置在`selfdatazone/cfg/`目录下
### 下面给出一个使用cuda:0,1,2共3个gpus的实例
```yaml
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
## 提交任务
### 编辑完成后执行配置文件后，可以执行如下脚本提交任务
```commandline
sh submit.sh xxxxx.yaml
```
### submit.sh脚本内容如下：
```commandline
#!/bin/bash

# Check if the filename is provided as an argument
if [ -z "$1" ]; then
  echo "Please provide the filename as an argument."
  exit 1
fi

# Load the configuration from the YAML file
ENVIRONMENT_IMAGES=$(awk '/IMAGES:/{print}' "$1")
ENVIRONMENT_IMAGES="${ENVIRONMENT_IMAGES#*:}"

ENVIRONMENT_CONTAIN=$(awk '/CONTAIN:/{print}' "$1")
ENVIRONMENT_CONTAIN="${ENVIRONMENT_CONTAIN#*:}"

ENVIRONMENT_NAME=$(awk '/NAME:/{print}' "$1")
ENVIRONMENT_NAME="${ENVIRONMENT_NAME#*:}"

TASK_CONFIG_PROJECT_DIR=$(awk '/PROJECT_DIR:/{print}' "$1")
TASK_CONFIG_PROJECT_DIR="${TASK_CONFIG_PROJECT_DIR#*:}"

TASK_CONFIG_GPU_NUM=$(awk '/GPU_NUM:/{print}' "$1")
TASK_CONFIG_GPU_NUM="${TASK_CONFIG_GPU_NUM#*:}"

TASK_CONFIG_GPUS=$(awk '/GPUS:/{print}' "$1")
TASK_CONFIG_GPUS="${TASK_CONFIG_GPUS#*:}"

TASK_CONFIG_COMMAND=$(awk '/COMMAND:/{print}' "$1")
TASK_CONFIG_COMMAND="${TASK_CONFIG_COMMAND#*:}"

TASK_CONFIG_LOG_DIR=$(awk '/LOG_DIR:/{print}' "$1")
TASK_CONFIG_LOG_DIR="${TASK_CONFIG_LOG_DIR#*:}"


time=$(date +"%Y_%m_%d_%H_%M")

logname="/data/LOG/${time}_${TASK_CONFIG_PROJECT_DIR}.txt"

# Activate the conda environment in the specified container
#docker exec -it $ENVIRONMENT_CONTAIN bash -c "source activate $ENVIRONMENT_NAME;cd /data/$TASK_CONFIG_PROJECT_DIR;$TASK_CONFIG_COMMAND > $logname"

# generate final bash for host
directory="/home/gh/tasks"

fname="${directory}/${TASK_CONFIG_GPUS}/${time}_N_${TASK_CONFIG_GPU_NUM}_GPUs_${TASK_CONFIG_GPUS}.sh"
echo $fname

echo 'docker exec -it $ENVIRONMENT_CONTAIN bash -c \"source activate $ENVIRONMENT_NAME;cd /data/$TASK_CONFIG_PROJECT_DIR;$TASK_CONFIG_COMMAND > $logname\"' > $fname
```
### 对于服务器的实时监视，提供了`.gettask.sh`脚本。脚本内容如下：
#### 需要注意的是，该脚本中关于指令的捕捉采用的是关键字匹配和按列选取的方法，所有有些时候输出内容并不完整，或者输出多余内容，只要不影响正常判断容器中运行情况即可。
```
#!/bin/bash

# Get the list of running containers
container_list=$(docker ps --format "{{.Names}}")

# Declare an empty array to store container information
container_info=()

# Loop through each container
for container_name in $container_list; do
  # Get the running Python command in the container
  python_command=$(docker exec -it "$container_name" bash -c "ps -ef | grep '[p]ython' | awk '{print \$8}'")

  # If no Python command is running, set default value to "NONE"
  if [ -z "$python_command" ]; then
    python_command="NONE"
  fi

  # Get the container's runtime
  container_runtime=$(docker inspect --format='{{.State.StartedAt}}' "$container_name")

  # Store the container information in the array
  container_info+=("Container: $container_name, Running Python Command: $python_command, Container Runtime: $container_runtime")
done

# Output all container information with box borders
for info in "${container_info[@]}"; do
  # Calculate the length of the longest line in the information
  max_length=$(echo "$info" | awk '{ if (length > max) { max = length } } END { print max }')

  # Add box borders above and below the information
  border=$(printf '=%.0s' $(seq 1 "$max_length"))
  echo "+$border+"
  echo "| $info |"
  echo "+$border+"
  echo
done


current_dir="/home/gh/tasks/"
echo "+==================================================+"
echo "| tasks waitting to be executed"
for dir in $(ls $current_dir); do


  dir="${current_dir}/${dir}"
  cd $dir

  for file in $(ls ); do
    echo "| $file"
  done 

  cd ..

done
echo "+===================================================+"
```
# 七、<span id=guide>用户使用指南
### 使用前请先来找我添加用户，因为考试周的原因，没有时间给所有人添加用户，请谅解。
### 关于本节内容，我录制了一个视频，有需要可以查看。
### 查阅该章节之前请先参考[客户端SSH配置](#user)完成本地SSH环境的安装。接下来开始从零开始使用服务器。
#### 1、在本地电脑上git项目，同时完成其中涉及到的附属子项目的git。
#### 2、使用MobaXterm登录服务器，登录过程中IP为：[10.160.72.107] ，端口号为：[420]。用户名为名字首字母缩写，密码为XKL+420+姓名首字母缩写（小写）     —— +不是密码内容，仅作连接表示。
#### 3、连接成功后，系统自动跳转至个人目录下，同时终端输出像素UI，表示正常登录（如非如此则代表配置文件错误，虽不影响正常使用，但建议跟我说）
#### 4、如果你不想了解系统整个的框架以及docker的镜像部署情况，请直接跳转到6，否则跳转到5。
#### 5、查看docker镜像部署以及已有容器部署
```
docker images
# DOCKER_IMAGES_NAME对应镜像名称
docker ps -a --filter ancestor=DOCKER_IMAGES_NAME
```
#### 6、使用默认的主镜像创建个人容器
```
# 如果提示无权限等问题，请使用sudo提权，后续不再赘述
# 这里我使用nvi/cu118创建一个需要gpu 0 1 ，运行内存为8G，挂载到我个人目录和数据集目录，名字为ljt的容器
docker run -it --gpus "device=0,1" --memory 8g --name ljt -v /data/ljt:/data  nvi/cu:118
```
##### 下面给出一般格式和常用的指令，大家可以参考修改使用
```
# 通用格式--不可直接使用，需要替换其中的变量
docker run -it --gpus all --memory <memory_limit> --name <container_name> -v <host_directory>:<container_directory>   -p <host_port>:<contain_port> --shm-size 4g <image_name>
# 高内存要求
docker run -it --gpus "device=0,1" --memory 16g --name NAME -v /selfdatazone/ljt:/data /datasetzone:/dataset  -p 42005:8888 --shm-size 4g   nvi/cu:118
# 无gpu需求
docker run -it --memory 8g --name NAME -v /selfdatazone/ljt:/data /datasetzone:/dataset nvi/cu:118
# cu116需求 ---- 不可使用 -因为nvidia还没有提供对于ubuntu22.04的cuda11.6.x的支持镜像，使用可能会报错
docker run -it --gpus "device=0,1" --memory 8g --name NAME -v /selfdatazone/ljt:/data /datasetzone:/dataset nvi/cu:116
```
#### 7、通过6我已经创建好了属于我自己的容器，默认会直接进入容器中，接下来我需要进入容器中使用从而避免别人打扰和我打扰别人。
#### 容器中已经封装好了anaconda和cuda toolkit环境，不需要再安装。
#### 8、接下来我需要创建适合我代码运行的conda环境
```
# 创建空环境
conda create -n ENV_NAME python==PYTHON_VERSION


# 一般项目会给requirement.txt或environment.yml
# 因为服务器配置国内源后使用VPN下载软件包导致证书问题，所以在使用文件创建环境时请首先检查一下文件中的channels部分
# environment.yml
conda env create --file environment.yml

# requirement.txt
conda create -n ENV_NAME python==PYTHON_VERSION
pip install -r requirement.txt
# 注：因为pip国内源未找到合适的替代，所以仍然使用清华源，但清华源可能会存在一些问题，可以尝试使用，可自行搜索替换。conda已经替换为ali源。


# 手动安装
conda create -n ENV_NAME python==PYTHON_VERSION
conda install PACKAGE==PACKAGE_VERSION
```
#### 9、通过8，我已经创建好了适合的conda环境，接下来可以提交任务。提交任务分为两种方案：将任务挂载在个人用户下；将任务挂载在服务器主用户下。
##### 两者区别：挂载在主用户下无法查看python报错信息，但任务权限高，适合已经确定没有运行错误的任务；挂载在个人目录下可以查看所有输出信息，但任务权限较低，适合进行初次代码调试任务。

---

##### 9.1、挂载在服务器主用户目录下：首先编辑个人目录下的`.cfg/xxx.yaml`文件.如何修改请参考 六
```
sudo vi cfg/xxx.yaml
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
#### 配置完成之后执行提交任务脚本
```
./.submit.sh .cfg/xxx.yaml
```

##### 9.2、挂载在个人用户下：使用如下指令后台实行任务
```bash
//其中 your_command代表需要执行的python指令，/data/LOG/filename.log表示运行日志保存目录，其他请不要修改。
nohup your_command > /data/LOG/filename.log 2>&1 &
//如何查看后台运行的任务
tail -f /data/LOG/filename.log
//查看过程中如何不影响后台任务直接退出
ctrl + C
```

---

#### 10、通过上面的9步，我已经将任务提交给了服务器，那么我应该如何查看服务器的运行情况呢？请使用如下脚本
```
./.gettask.sh
```
#### 需要注意的是，该脚本中关于指令的捕捉采用的是关键字匹配和按列选取的方法，所有有些时候输出内容并不完整，或者输出多余内容，只要不影响正常判断容器中运行情况即可。
#### 11、对于代码运行过程的输出内容，全部保存在LOG文件夹中，文件以”时间_项目名.txt“存放。



#### 12、中途我如果不想等待代码运行，我希望直接退出，这是被允许的。直接使用`exit`指令即可退出容器返回宿主终端。如果这个过程中我希望再次进入容器，请使用以下指令。
```
docker startt CONTAIN_NAME
docker exec -it CONTAIN_NAME bash
# 不进入容器但希望容器执行一条指令
docker exec -it CONTAIN_NAME bash -c COMMAND
# 不进入容器但希望容器执行多条指令
docker exec -it CONTAIN_NAME bash -c "COMMAND1;COMMAND2;COMMAND3;..."
```
#### 13、如果我发现我已经不使用这个容器或者其中已经不适合再使用了。请使用如下指令删除容器。
#### 删除过程中不要害怕对服务器由什么影响，大胆删除！也不要对这个容器心存不舍，服务器的内存希望、也希望你这么做。
```
# 停止容器
docker stop CONTAIN_NAME
# 删除容器
docker rm CONTAIN_NAME
```

### 附：jupyterlab使用
#### jupyter lab虽然写代码体验不如pycharm vscode等众多IDE，但是其功能很强大，可以完成数据分析、学习模型可视化等众多高端功能，所以我后续更新了jupyter lab 的使用过程


### 后记：
#### 1、对于未使用过docker的同僚来说确实比较陌生和难以下手，但请放心按照上面的13步一步一步来，遇到问题及时沟通，旧的容器不好用了直接换新的容器。多多使用才是关键！
#### 2、对于服务器的使用来说，服务器代码的运行会直接挂载在主用户下运行，所以一般不会存在问题。比较头疼的问题可能就是环境的配置，因为证书问题，不可能同时使用国内源和挂载VPN，所以如果在环境配置的过程中（尤其是下载软件包）遇到网络问题，例如HTTP 000错误，可以重复执行几次试试。或者进入~/.condarc文件中将“-default”那一行删除。
#### 3、还存在一个潜在的问题是关于docker的镜像版本问题。nvidia并没有发布对于Ubuntu22.04的cuda 11.6.x版本的支持镜像；而对于使用Ubuntu20.04而言，可以使用cuda 11.0.x ~ 11.6.x，但当安装高于510.x的显卡驱动（CUDA 11.6+）时，会更新系统内核而导致较低版本的网卡驱动无法使用，也就是说Ubuntu20.04的服务器无法使用cuda 11.6+版本。出于对cuda的向下兼容性和版本持续更新，以及Ubuntu20.04LTS剩余支持时间的考虑，服务器使用了Ubuntu22.04。对于上面的问题，如果大家有好的解决办法的话请务必跟我联系，这是一个至今没有任何解决办法的问题。
#### 4、个人精力和水平有限，如果大家有更好的解决方案可以一起讨论实施。