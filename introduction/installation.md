## 安装Ansible

### 获取Ansible

如果你有Github账号，你可以关注Ansible在Github上的代码库。我们在Github上跟踪bug和有关功能的想法。

### 安装的基本信息/我们安装些什么

默认情况下，Ansible通过SSH协议来管理远端机器

安装Ansible是不会安装什么数据库的，并且后台也不会跑什么守护进程。只需在一台电脑上安装Ansible（笔记本都行），然后就能从这一台电脑上管理一整个服务器集群了。在Ansible操作被管理的机器时，是不会在被管理的机器上装任何软件的，所以也不必担心迁移到Ansible新版本的时候如何升级的问题。

### 我应该选哪个版本

因为Ansible可以用源码执行，也不需要在被控制的服务器上装软件，很多用户都会用开发版。

Ansible的更新周期大概是两个月。因为新版本的发布周期很短，小Bug一般会在下一个版本修复，而不是把修复Bug的代码移植到稳定版上。但是如果是影响比较大的Bug，我们还是会释放出一个修复更新来解决，虽然这种事情不常有。

如果你希望用到最近更新的Ansible版本，并且你用的是Red Hat Enterprise Linux(TM)，CentOS，Fedora，Debian或者是Ubuntu，我们建议你用操作系统上的程序包管理器。

如果你想用别的方法安装Ansible，我们建议你使用“```pip```”，“```pip```”是Python包管理器的（Python Package Manager）。除此之外还有其他安装方法，此处暂不赘述。

如果你乐意使用正在开发的版本，希望尝试最新的功能，我们会介绍如何用源码来运行Ansible。如果用源码执行的话，什么都不用装。

### 主控机的配置要求

只要你的被控端装了Python 2.6或者2.7就行了。（但不能是Window。）

这就包括了Red Hat，Debian，CentOS，OS X，任何版本的BSD，等等。

>注意：	
>2.0版本的Ansible需要更多的文件句柄来控制子进程，但是OS X的文件句柄设置非常的小。如果你要用超过15个子进程，或者你看见了诸如“```Too many open files```”的错误信息时，你就可能需要调大ulimit了，比如```sudo launchctl limit maxfiles 1024 2048```。

### 被控机的配置要求

在被控机上要有和主控机沟通的途径，一般是```SSH```。默认情况下```SSH```会用到```SFTP```，如果用不了的话你也可以在```ansible.cfg```下指定使用```SCP```。被控机上还要有Python 2.4以上版本。此外，如果你的被控机上的Python版本低于2.5，你还需要安装以下Python模块：

- ```python-simplejson```

>注意：	
>Ansible的“raw”模块（简单粗暴的执行命令工具）和脚本模块连```python-simplejson```都不需要。所以，理论上说，你可以用Ansible的“raw”模块先在被控机上安装```python-simplejson```，然后就能用别的各种功能了。(讲的可能有点儿超前了。)

>注意：	
>如果你再被控机上启用了SELinux，你可能还要在上面安装```libselinux-python```，然后才能用Ansible里面的复制、文件、模板等功能。用Ansible的yum模块可以在被控机上安装这个包。

>注意：
>Python 3是一个和Python 2 稍有区别的语言，绝大多数的Python程序还没有迁移到Python 3上，其中就包括Ansible。然而，有些Linux发行版（Gentoo，Arch）可能默认安装的不是Python 2.X版本。在这些系统里，你需要安装一个Python 2.X版本，然后再Ansible的Inventory中（参看*Inventory*章节）把```ansible_python_interpreter```变量设置成Python 2.X的安装路径。Red Hat Enterprise Linux，CentOS，Fedora和Ubuntu的默认Python版本都是2.X，因此不需要做上面的操作。几乎所有的Unix操作系统也都不用装2.X版本。	
>如果你需要对被控机做一些初始化操作，可以用"raw"模块。例如如下命令：```ansible myhost --sudo -m raw -a "yum install -y python2 python-simplejson"```，会帮你把Ansible和Ansible模块需要的Python 2.X和simplejson安装好。

### 在主控机上安装Ansible

#### 用源码运行

在代码库里提取出Ansible的代码，就已经可以运行Ansible了，就这么简单。不用root权限，也不用把Ansible看成需要装在系统中的软件，不用守护进程，也不用数据库。正因为如此，Ansible社区的很多用户一直以来用的都是Ansible的开发版本，以便用上最新的功能，同时为该社区做出贡献。因为什么都不需要装，用Ansible的开发版本要比用别的开源项目的开发版本要容易得多。

>注意：
>如果打算用Ansible Tower，不要用源码包。请使用操作系统提供的程序包管理器（比如apt或者yum），或者pip来安装稳定版。

获取源码：

```
$ git clone git://github.com/ansible/ansible.git --recursive
$ cd ./ansible
```

用Bash安装：

```
$ source ./hacking/env-setup
```

用Fish安装：

```
$ . ./hacking/env-setup.fish
```

如果想省略无关紧要的报错（warning/errors），可以使用下面的参数：

```
$ source ./hacking/env-setup -q
```

如果主控机上的Python没有pip，用下面的命令安装一个pip：

```
$ sudo easy_install pip
```

然后用pip安装一些Ansible需要的Python模块：

```
$ sudo pip install paramiko PyYAML Jinja2 httplib2 six
```

请注意，如果更新Ansible的话，请不要只更新代码树，还要更新git中的“子模块”，这个“子模块”指向了Ansible自己的模块（此模块非彼模块）：

```
$ git pull --rebase
$ git submodule update --init --recursive
```

一旦运行了```env-setup```脚本，你就可以在提取出的代码上运行Ansible了。默认的inventory文件是```/etc/ansible/hosts```，你还可以自己指定一个inventory文件（请参看*Inventory*章节）：

```
$ echo "127.0.0.1" > ~/ansible_hosts
$ export ANSIBLE_INVENTORY=~/ansible_hosts
```

>注意：
>```ANSIBLE_INVENTORY```在1.9版本被引入，代替了之前的```ANSIBLE_HOSTS```

有关inventory文件的细节可以参看后面的章节。

现在就可以用ansible的ping命令来测试一下了：

```
$ ansible all -m ping --ask-pass
```

当然，如果你愿意，你也可以用```sudo make install```把Ansible安装在主控机上。

#### 用YUM安装最新版

Ansible的rpm包现在已经收录在EPEL的6和7版本中，目前的Fedora里默认就有EPEL软件源。

Ansible可以管理安装了Python 2.4以上版本的操作系统（也包括RHEL 5）。

Fedora用户可以直接安装Ansible。如果你是RHEL或CentOS用户，没有添加EPEL数据源的话，需要提前添加进去。

```
# CentOS，RHEL或者Scientific Linux没装EPEL的要先装epel-release这个安装包
$ sudo yum install ansible
```

你也可以自己封装一个RPM包。在Ansible代码的根目录下使用```make rpm```命令封装RPM包，拿来分发给别人或者自己用它安装都可以，前提是系统中安装了```rpm-build```，```make```和```python2-devel```。

```
$ git clone git://github.com/ansible/ansible.git --recursive
$ cd ./ansible
$ make rpm
$ sudo rpm -Uvh ./rpm-build/ansible-*.noarch.rpm
```

#### 用APT（Ubuntu）安装最新版

为Ubuntu提供的安装包可以在这个PPA处下载到。

配置PPA，安装Ansible的命令如下：

```
$ sudo apt-get install software-properties-common
$ sudo apt-add-repository ppa:ansible/ansible
$ sudo apt-get update
$ sudo apt-get install ansible
```

>注意：     
>在旧版本的Ubuntu下，“```software-properties-common```”被称作“```python-software-properties```”。

Debian/Ubuntu下也可以通过如下方式把源码打包：

```
$ make deb
```

或许你还可能想用源码直接运行Ansible，具体方法我们已在本文上方介绍过，此处不再赘述。

#### 用Portage（Gentoo）安装最新版

```
$ emerge -av app-admin/ansible
```

要安装Ansible的最新版，你还要在emerge之前启用Ansible。

```
$ echo 'app-admin/ansible' >> /etc/portage/package.accept_keywords
```

>注意：     
>如果你的Gentoo系统上默认安装的是Python 3，则必须在组或者inventory的环境变量中设置以下内容：```ansible_python_interpreter = /usr/bin/python2```。

#### 用pkg（FreeBSD）安装最新版

```
$ sudo pkg install ansible
```

也可以在ports中安装：

```
$ sudo make -C /usr/ports/sysutils/ansible install
```

#### 在Mac OS X上安装最新版

在Mac系统里安装Ansible的最好方法就是用pip。具体可以参看用pip安装的章节。

#### 用OpenCSW（Solaris）安装最新版

OpenCSW为Solaris系统提供了一个用于SysV的Ansible安装包。

```
# pkgadd -d http://get.opencsw.org/now
# /opt/csw/bin/pkgutil -i ansible
```

#### 用Pacman（Arch Linux）安装最新版

在Pacman社区版本的软件库中可以下载到Ansible。

```
$ pacman -S ansible
```

在Arch Linux用户库中有一个专门用来从Github上拉取Ansible的软件包。

也请参阅ArchWiki上的Ansible介绍。

>注意：     
>如果你的Arch系统上默认安装的是Python 3，则必须在组或者inventory的环境变量中设置以下内容：```ansible_python_interpreter = /usr/bin/python2```。

#### 用pip安装最新版

Ansible can be installed via “pip”, the Python package manager. If ‘pip’ isn’t already available in your version of Python, you can get pip by:

也可以用pip来安装Ansible。pip的意思是“Python包管理器”。如果您的Python中尚未安装pip，您可以通过如下命令安装pip：

```
$ sudo easy_install pip
```
然后再安装Ansible
```
$ sudo pip install ansible
```
If you are installing on OS X Mavericks, you may encounter some noise from your compiler. A workaround is to do the following:

如果您要在OS X Mavericks上安装Ansible，您的编译器可能会误报一些错误信息。您可以通过如下变通的方法来解决这些报错：
```
$ sudo CFLAGS=-Qunused-arguments CPPFLAGS=-Qunused-arguments pip install ansible
```

Readers that use virtualenv can also install Ansible under virtualenv, though we’d recommend to not worry about it and just install Ansible globally. Do not use easy_install to install ansible directly.

习惯使用```virtualenv```的用户也可以在```virtualenv```下安装Ansible。不过我们不建议您这样做，直接将Ansible安装在真实环境中并没有什么大问题。对了，不要直接用```easy_install```安装Ansible。

#### Tarballs of Tagged Releases

如果您想把Ansible打成包自己留着用，但又不想用git checkout来下载代码，您可以在Ansible下载页下载到每个版本的压缩包。

在我们的Git里，除了最新版以外，Ansible的每个往期版本也有各自的标签。