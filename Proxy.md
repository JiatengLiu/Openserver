# 服务器学术资源加速使用方式

**声明：限于学术使用github和huggingface等地址网络速度慢的问题，以下为方便用户学术用途使用相关资源提供的加速代理，不承诺稳定性保证。请勿恶意攻击或借助该服务访问非法网站。此外如遭遇恶意攻击、用户访问非法网站等行为，将随时停止该加速服务且不提前通知。**
**以下操作均在个人docker容器中完成**

### 设置方法

1. 请编辑“~/network_turbo_on”文件为如下内容：

   ```shell
   export http_proxy=http://192.168.229.15:7890
   export https_proxy=http://192.168.229.15:7890
   export no_proxy=127.0.0.1,localhost
   export HTTP_PROXY=http://192.168.229.15:7890
   export HTTPS_PROXY=http://192.168.229.15:7890
   export NO_PROXY=127.0.0.1,localhost
   echo -e "\033[32m[INFO] Proxy Enabled\033[0m"
   ```

2. 请编辑“~/network_turbo_off”文件为如下内容：

   ```shell
   unset http_proxy
   unset https_proxy
   unset no_proxy
   unset HTTP_PROXY
   unset HTTPS_PROXY
   unset NO_PROXY
   echo -e "\033[31m[INFO] Proxy Disabled\033[0m"
   ```
3. 若不了解Ubuntu编辑文件指令参考如下指令
   ```shell
   # 创建文件
   touch ~/network_turbo_on
   touch ~/network_turbo_off
   # vim修改文件
   vi ~/network_turbo_on
   vi ~/network_turbo_off
   # 保存并退出
   Esc ; Shift + Z + Z
   ```
### 在终端中使用

1. 请执行以下命令开启系统代理:

   ```shell
   source ~/network_turbo_on
   ```

2. **取消学术加速**，如果不再需要建议关闭学术加速，因为该加速可能对正常网络造成一定影响. 若要关闭系统代理，请执行以下命令：

   ```shell
   source ~/network_turbo_off
   ```


### 在Notebook中使用

```python
import subprocess
import os

result = subprocess.run('bash -c "source ~/network_turbo_on && env | grep proxy"', shell=True, capture_output=True, text=True)
output = result.stdout
for line in output.splitlines():
    if '=' in line:
        var, value = line.split('=', 1)
        os.environ[var] = value
```

### Test
```shell
source ~/network_turbo_on
curl google.com
source ~/network_turbo_off
```
