# Ubuntu 装机步骤

* 安装 Ubuntu 18.04
* 配置 ssh
  * 安装 openssh-server
  * 使用 ssh-keygen 生成一个 ssh key
  * 在 Ubuntu 服务器上添加刚才生成的 ssh key
  * 禁止密码登录
* 配置 sudo 免密码
* 安装 oh my zsh 以及常用命令（可省略）
* 安装 NVIDIA 驱动
* 安装 CUDA、cuDNN（PyTorch 用户可以忽略）

## 如何安装 Ubuntu

* 准备系统 U 盘
* 修改 BIOS 选择 U 盘启动
* 安装 Ubuntu

## 配置 ssh

### 安装 openssh-server

刚装好 Ubuntu 以后，为了能够方便地在笔记本上远程连接安装各种软件，我一般会先装openssh-server。

```bash
sudo apt update
sudo apt install openssh-server
```

装好了以后，就可以在终端里通过下面的命令连接服务器。

```bash
ssh 192.168.8.65
```

使用到的命令：[apt](linux-command.md#apt)、[ssh](linux-command.md#ssh)

### 使用 ssh-keygen 生成一个 ssh key

目前使用密码上不如使用 key 安全的，因为 key 的长度通常是 2048 或 4096 位，远超普通的密码。这里建议按照 GitHub 官方网站上的教程[生成新 SSH 密钥并添加到 ssh-agent](https://help.github.com/cn/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)，这样不仅可以用于登录 Linux 机器，也可以用于 git。

使用 `ssh-keygen` 生成了密钥以后，你会得到两个文件：

* `~/.ssh/id_rsa`
* `~/.ssh/id_rsa.pub`

其中 `id_rsa` 是私钥，`id_rsa.pub` 是公钥。

​公钥通常上这样的文本：

> ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDcfQD4nnaQrgC003C0ReAYqrm6XKEchuM6DwkNtHzRs2YhLolH6fEohWAp0mlW9yzQl258lA+FdoH51ONmvGUx/2Y953A6ujHrt7dOIpOkYW5HT6Rmvlwk+3uRmVNQqZTJeRWdblcSHmYY1fi1miawONZiVoLW14xhyH5bOwgwY/GB2yTaffDby451RLT5kQ7LHqiDJtWiRQekR5tKDKZ7mBhKZIKYLOoqAayCHw2scJYsgtw2tPt0/y7BlxO+RuDwuD2WOnthe1ewbwAbbTN17HiLvBd/MIWtVDoGGq6jdNDjNDNbTs8H2VohTc2mNtOOlycN6yzf3ZKZTN6/hNtl ypw@yangpeiendeiMac

### 在 Ubuntu 服务器上添加刚才生成的 ssh key

首先使用密码登录刚才的机器：

```bash
ssh 192.168.8.65
```

然后创建 `~/.ssh` 目录，并且创建 `~/.ssh/authorized_keys` 文件，配置好相应的权限：

```bash
mkdir -p ~/.ssh
touch ~/.ssh/authorized_keys
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

把刚才生成的公钥 `id_rsa.pub` 添加到 `~/.ssh/authorized_keys` 里，如果文件已经有内容，就添加在最后一行。这样以后**每次使用 ssh 登录机器都不需要再输入密码了**。

使用到的命令：[mkdir](linux-command.md#mkdir)、[touch](linux-command.md#touch)、[chmod](linux-command.md#chmod)、[nano](linux-command.md#nano)

### 禁止密码登录

由于密码登录存在不安全因素，比如暴露在公网的 IP 会被扫描，而 key 是绝对安全的，所以我们可以禁止密码登录：

```bash
sudo nano /etc/ssh/sshd_config

PasswordAuthentication no # 添加在最后一行
```

## 配置 sudo 免密码

由于安装驱动等操作需要 sudo 权限，为了避免频繁输入密码，可以配置 sudo 免密码：

```bash
sudo nano /etc/sudoers

ypw ALL=(ALL) NOPASSWD:ALL # 添加在最后一行
```

## 安装 oh my zsh 以及常用命令

### 安装 oh my zsh

相比默认的 bash，zsh 有以下几个优点：

* 当你使用 tab 提示的时候，如果有多个匹配项，你可以用 tab 进行切换
* 当你想使用一个之前输入过的命令的时候，只需要输入首字母，然后按上方向键切换

安装 oh my zsh 的步骤如下：

* 安装 [zsh](https://github.com/robbyrussell/oh-my-zsh/wiki/Installing-ZSH)
* 安装 [oh my zsh](https://ohmyz.sh/)

完整命令如下：

```bash
sudo apt install -y git curl zsh
sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```

### 常用命令

这些是我眼中的[必备命令](ubuntu-environment.md#bi-bei-ming-ling)，你可以按照自己的需要安装：

```bash
sudo apt install git curl htop nload tmux screen aria2 graphviz aptitude tree
```

## 安装 NVIDIA 驱动

### 下载驱动

首先去官网下载驱动：[https://www.nvidia.com/Download/index.aspx?lang=cn](https://www.nvidia.com/Download/index.aspx?lang=cn)

有下面几种方法可以下载：

* 在 Ubuntu 机器上直接使用浏览器下载
* 在自己机器上下载，然后通过 [scp](linux-command.md#scp) 传到 Ubuntu 机器上
* 在自己机器上找到下载地址，然后在 Ubuntu 机器上使用 wget 命令下载

以上方法可以根据实际情况选择，这里我选择第三种方法：

```bash
wget https://cn.download.nvidia.cn/XFree86/Linux-x86_64/430.34/NVIDIA-Linux-x86_64-430.34.run
```

### 屏蔽 Nouveau

如果你没有屏蔽，就会出现以下提示：

> ERROR: The Nouveau kernel driver is currently in use by your system. This driver is incompatible with the NVIDIA driver, and must be disabled before proceeding. Please consult the NVIDIA driver README and your Linux distribution's documentation for details on how to correctly disable the Nouveau kernel driver.

命令：

```bash
sudo nano /etc/modprobe.d/blacklist-nouveau.conf

# 添加以下内容

blacklist nouveau
options nouveau modeset=0

# 保存

sudo update-initramfs -u
sudo reboot
```

### 安装必要的编译工具

如果没有安装 gcc 等编译工具，就会出现以下提示：

> ERROR: Unable to find the development tool `cc` in your path; please make sure that you have the package 'gcc' installed. If gcc is installed on your system, then please check that `cc` is in your PATH.

命令：

```bash
sudo apt install -y build-essential gcc g++ make binutils linux-headers-`uname -r`
```

### 安装显卡驱动

```bash
sudo bash NVIDIA-Linux-x86_64-430.34.run
```
