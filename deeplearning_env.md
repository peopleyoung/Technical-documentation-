





# **环境配置方法**

## 设置固定IP

```
sudo vim /etc/netplan/01-netcfg.yam		#名字可能有变化
```

```
# Let NetworkManager manage all devices on this system
network:
  ethernets:
    enp11s0:
      addresses: [172.16.0.116/24]
      dhcp4: no
      optional: true
      gateway4: 172.16.0.255
      nameservers:
        addresses: [172.16.0.1,114.114.114.114]
    enp12s0:
      dhcp4: true
  version: 2
```

最后执行

```text
sudo netplan apply
```

        - 192.168.2.185/24
            gateway4: 192.168.2.255
            nameservers:
                addresses: [192.168.2.1,114.114.114.114]

## 安装openssh-server

```
sudo apt-get install openssh-server
```

开机自动启动ssh命令

```
sudo systemctl enable ssh
```

查看ssh是否启动，看到Active: active (running)即表示成功

```
sudo systemctl status ssh
```



## 关闭自动休眠

```
sudo systemctl mask sleep.target suspend.target hibernate.target hybrid-sleep.target
```

观察休眠状态，出现下面提示则关闭成功

```
$ systemctl status sleep.target
● sleep.target
  Loaded: masked (Reason: Unit sleep.target is masked.)
  Active: inactive (dead)
```

## 1.Ubuntu20.04下深度学习环境配置

参考博文：https://blog.csdn.net/m0_37412775/article/details/109355044

### (1)换国内源

```
 \\打开Ubuntu的终端
$sudo apt-get install vim
$sudo mv /etc/apt/sources.list /etc/apt/sources.list.bak
$sudo vim /etc/apt/sources.list
```

 将目前的源全部注释掉，即每行开头没有“#”的都加上#。

然后打开该网页[Ubuntu阿里源](https://developer.aliyun.com/mirror/ubuntu?spm=a2c6h.13651102.0.0.3e221b11YgAWt2)，选择自己对应的Ubuntu版本号将下面的源进行复制粘贴到刚刚打开的sources.list文件中。

```
$sudo apt-get update
$

```

清华源：https://mirrors.tuna.tsinghua.edu.cn/help/ubuntu/

### (2)gpu驱动

gpu驱动安装分为：1. 禁用nouveau, 2. 卸载旧的驱动（如果你是第一次安装驱动，这步可以省略），3.gcc安装，4，安装驱动

**禁用nouveau**

```
// 修改属性
$sudo chmod 666 /etc/modprobe.d/blacklist.conf  
$sudo vim /etc/modprobe.d/blacklist.conf
// 在最后一行加入以下几句，保存退出
blacklist vga16fb
blacklist nouveau
// 对刚才修改的文件进行更新
$sudo update-initramfs -u
```

记得重启计算机，打开终端检查nouveau是否被禁用

```
$lsmod | grep nouveau \\若执行完该句，没有任何输出，则nouveau被成功禁用
```

**卸载旧的驱动**

```
$sudo apt-get remove --purge nvidia*
```

安装gcc

必须先安装gcc, 才能安装gpu驱动。

```
$sudo apt-get  install  build-essential \\安装gcc,不要怀疑，就是build-essential，我也不知道为啥
$gcc --version \\检查gcc是否安装成功
```

这里我只安装了gcc，gpu驱动就可以成功安装了，如果你执行到下面，提示你make，g++没有安装的话，再回来执行以下语句进行安装即可。

```
$sudo apt-get install g++
$sudo apt-get install make
```

**下载对应版本的gpu驱动**

[gpu驱动下载链接](https://www.nvidia.cn/geforce/drivers/),推荐使用run文件安装。[官方驱动 | NVIDIA](https://www.nvidia.cn/Download/index.aspx?lang=cn)

**安装驱动**

先按Ctrl+Alt+F1，关闭图形界面（过程中出现星号则证明需要输入密码）。再在新出现的界面输入以下命令：

```
$sudo service lightdm stop
//以移动后的目录为/homg/xxx/为例
$cd /home/xxx/
//修改权限
$sudo chmod a+x NVIDIA-Linux-x86_64-xxx.run
//执行安装
$sudo ./NVIDIA-Linux-x86_64-xxx.run -no-x-check -no-nouveau-check -no-opengl-files
```

**完成安装并验证**

```
//启动图形界面
$sudo service lightdm start
//执行此语句，出现显卡信息则证明安装成功。
$nvidia-smi
```



![gpu驱动信息](https://raw.githubusercontent.com/daydreamerG/picstg/main/202112021402408.png)

### (3)miniconda安装

[miniconda下载链接](https://mirrors.tuna.tsinghua.edu.cn/anaconda/miniconda/)

**区分Ubuntu自带python和miniconda中的python**

```
$cd ~
$sudo vim .bashrc \\利用vim打开.bashrc文件
```

在文件最后加入以下命令：

```
alias python3="/usr/bin/python3.8"\\给系统自带的python起一个别名，就叫python3
```

然后按下esc键，再输入:wq!进行保存，再按esc键退出vim编辑器。

继续在终端中对刚才修改的. bashrc文件执行以下。

```
$source .bashrc
```

试试刚才进行区分的设置。

```
$python \\本条命令应该启动anaconda3中的python
$exit() \\退出
$python3 \\本条命令应该启动系统的python
$exit() \\退出
```

**创建Deeplearning的环境**

```
\\安装完成后需要重新打开命令行窗口
\\这步中Deeplearning可以换成任何你喜欢的名字
$conda create -n dev python=3.8
\\查看你创建的环境
$conda env list
\\激活创建的环境
$conda activate dev
\\关闭环境
$conda deactivate


conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/conda-forge
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/bioconda 
conda config --set show_channel_urls true
```

**pip源修改**

```
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
```

```
#需要新建文件夹
sudo mkdir ~/.config/
sudo mkdir ~/.config/pip
sudo vim ~/.config/pip/pip.conf
```

加入以下内容

    [global]
    index-url = https://pypi.tuna.tsinghua.edu.cn/simple
    [install]
    trusted-host=mirrors.aliyun.com

### (4)cuda安装

[选择合适的cuda版本进行安装](https://developer.nvidia.com/cuda-toolkit)

记得在你创建的虚拟环境中进行：

```
$ conda activate dev \\dev是我上面起的名字
```

各版本cuda下载:https://developer.nvidia.com/cuda-toolkit-archive.本项目选择离线下载cuda_11.3.0_465.19.01_linux.run

```
\\已经下载好cuda.run的镜像文件，先cd到对应目录
$cd /home/xxx/下载 \\默认下载目录

\\安装cuda
$sudo sh cuda_11.3.0_465.19.01_linux.run \\输入完语句后不要着急，稍微等一两分钟
```

"Do you accept the above EULA?"输入accept  

![cuda安装过程记录](https://raw.githubusercontent.com/daydreamerG/picstg/main/202112021401550.png)

然后将Driver前面的选项按回车，去掉；其他的保持不动，选择“Install”，稍等几分钟就装好了，会出现以下的界面。

![cuda安装过程](https://raw.githubusercontent.com/daydreamerG/picstg/main/202112021402868.png)

配置环境

```
$sudo vim ~/.bashrc \\进入vim界面。输入字母i，进入编辑模式
\\在bashrc文件中输入以下命令，注意修改你的cuda版本
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/cuda-11.3/lib64
export PATH=$PATH:/usr/local/cuda-11.3/bin
export CUDA_HOME=$CUDA_HOME:/usr/local/cuda-11.3
\\输入完成后，点击esc键并输入:wq!，再按esc键退出vim。
\\这时候返回终端了
$source ~/.bashrc \\运行.bashrc文件
```

安装完成检查

```
nvcc --version
```

若出现以下的界面，则表示cuda安装完成

![cuda安装完成](https://raw.githubusercontent.com/daydreamerG/picstg/main/202112021402661.png)

### (5)cudnn安装

[选择合适的cudnn版本进行安装](https://developer.nvidia.com/rdp/cudnn-download)

**本项目选择安装版本：cudnn-11.3-linux-x64-v8.2.1.32.tgz

```
$ conda activate deeplearning 
$sudo cp cuda/include/cudnn*.h /usr/local/cuda/include/
$sudo cp cuda/lib64/libcudnn* /usr/local/cuda/lib64/
$sudo chmod a+r /usr/local/cuda/include/cudnn*.h /usr/local/cuda/lib64/libcudnn*
```

最后用torch.cuda.is_available()测试是否成功

### (6)pytorch-gpu安装

离线下载pytorch和torchvision。版本：torch-1.7.0+cu110-cp38-cp38-linux_x86_64.whl，torchvision-0.8.0-cp38-cp38-linux_x86_64.whl

## 2.配置vscode

windows端：打开cmd,输入

```
ssh-keygen -t rsa -C "258415023@qq.com"
```

*其中-t代表密钥类型为rsa类型， -C 为注释。*

生成的公钥和私钥保存在 ***`c:/user/admin/.ssh/`*** 下
`id_rsa`为私钥
`id_rsa.pub`为公钥

在服务器上设置：

```
sudo mkdir ~/.ssh
cd .ssh
sudo vim authorized_keys
#把生成的id_rsa.pub文件中的内容复制进入保存
cd ../
sudo chmod 777 ~/.ssh 
sudo chmod 777 ~/.ssh/authorized_keys
```



最后打开vscode配置一下远程ssh目录和指定一下本地密钥就ok

我将配置放到了密钥的同级目录*C:\Users\admin.ssh\config*
加入配置：`IdentityFile "C:\Users\admin\.ssh\id_rsa"` 保存即可

## 3.配置github host

1、进入终端命令行模式，输入sudo vim /etc/hosts

2、用浏览器访问 IPAddress.com 使用 IP Lookup 工具获得github.com和github.global.ssl.fastly.net域名的ip地址

3、在vi打开的hosts文件中添加如下格式，在最后添加：

192.30.253.112 github.com （根据自己查到的ip地址改写）

151.101.44.249 github.global.ssl.fastly.net （根据自己查到的ip地址改写）后缀可能会有不一样，不用管

6、更新DNS缓存，输入`sudo /etc/init.d/networking restart`



# docker版环境配置方法

在docker容器内直接pip安装opencv-python，再import cv2时会报错：

```
ImportError: libgthread-2.0.so.0: cannot open shared object file: No such file or directory
```

【解决方法】
在容器内不能用上述方法安装，需要先卸载原有的cv2：

```
pip uninstall opencv-python
pip install opencv-python-headless
```

需要安装的环境

```
apt-get install build-essential git vim
```

pytorch启动命令：

```
docker run -it --gpus all --name pytorch1.7 pytorch/pytorch:1.7.0-cuda11.0-cudnn8-runtime

```



## 安装docker-ce

```
sudo apt-get update
sudo apt-get -y install apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
sudo apt-get update
sudo apt-get -y install docker-ce
```



## 安装nvidia-docker

```
curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add -
distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list
sudo apt-get update
sudo apt-get install -y nvidia-docker2
sudo systemctl restart docker
```



### 将docker账户给与权限

```
sudo gpasswd -a <你的用户名> docker
例如： sudo gpasswd -a ubuntu docker
newgrp docker  # 更新docker组
退出shell 重新进入
```