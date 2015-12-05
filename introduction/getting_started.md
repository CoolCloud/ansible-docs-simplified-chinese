## 前言

看完了安装文档，想必您已经装好了Ansible。现在让我们开始使用Ansible，就从一些命令开始吧。

我们不打算一上来就介绍Ansible的配置、部署、编排之类的牛逼功能。这些功能都是由playbook实现的，我们会在别的章节介绍。

这一章我们先来聊聊怎么起步。当您了解了本章介绍的一些概念后，再去看“一次性命令简介”这一章节。再然后就可以去学习playbook，研究点儿有意思的东西了！

### 远程连接的细节

聊别的之前，让我们先来看一下Ansible是如何通过SSH和远端服务器交互的。

默认情况下，Ansible 1.3之后的版本会尝试使用本机自带的OpenSSH来和远端通信。这样一来便能使用ControlPersist（一个提高性能的功能），Kerberos，以及在```~/.ssh/config```中的配置了（比如Jump Host的设置）。可是，如果您用RHEL 6（或者对应版本的CentOS）作为主控端，上面的OpenSSH版本太低，是没办法支持ControlPersist的。在这样的操作系统上，Ansible会转而使用一个Python版本的OpenSSH，叫做“paramiko”。如果您还想用支持Kerberos的SSH，您要么用Fedora，OS X或者Ubuntu之类的系统做主控端，要么就看看Ansible的“加速模式”吧。

1.2版本和再之前的Ansible都默认使用paramiko来连接SSH。如果想要使用原生的SSH，必须要再命令上加上```-c```参数，或者在配置文件里设置。

有时候，您可能还会碰见不支持SFTP的设备。这种事情不常有，但如果真让您碰上了，您可以在配置文件里把文件服务设置成SCP。

在和被控端通信时，Ansible会默认您用SSH密钥通信，我们推荐您使用SSH密钥，但是您也可以用密码，用```--ask-pass```参数即可。如果您要用到sudo命令，而且sudo时候也需要密码的话，用```--ask-sudo-pass```参数即可。

有件事情可能是众人皆知，但是还是值得拿出来说说：管理系统离被管理的机器越近越好。如果您要在云环境下使用Ansible，您最好在云内的一个机器上使用。很多时候，这样做好于您从公网连到云。

说点儿高级的：Ansible也并不是只能用SSH连到远端服务器上去才有用。远程传输是可以不用的，而且Ansible还有管理本地机器的功能，以及管理chroot，lxc，或者jail container等等。Ansible还有一个叫做“Ansible-pull”的模块，能把整个管理模式倒过来，让被管理的系统“像给家里打电话”一样通过git将配置信息从集中的配置库中提走。

### 您要敲第一条命令了！

装好了Ansible，让我们从基础开始做起。

创建或者编辑```/etc/ansible/hosts```文件，在里面写上被控端的IP或者域名。主控端的公钥应该在您写的这些被控端的```authorized_keys```文件夹中。

```
192.168.1.50
aserver.example.org
bserver.example.org
```

这便是我们的inventory文件，我们会在Inventory详细介绍。

我们假设您使用SSH密钥来完成认证。要设置SSH，您可以这样做：

```
$ ssh-agent bash
$ ssh-add ~/.ssh/id_rsa
```

（根据您的实际情况，您可能会需要用```--private-key```这个Ansible参数来指定用哪一个pem文件。）

现在让我们ping一下各个节点：

```
$ ansible all -m ping
```

Ansible会用您当前的用户来连接被控端，这和SSH的做法是一样的。如果想指定一个登录用户，使用```-u```参数。

```
# 以用户bruce登录
$ ansible all -m ping -u bruce
# 以用户bruce登录，还要sudo到root
$ ansible all -m ping -u bruce --sudo
# 以用户bruce登录，还要sudo到用户batman
$ ansible all -m ping -u bruce --sudo --sudo-user batman

# 在最新版的Ansible中我们不再支持`sudo`了，因此我们希望您用become参数
# 以用户bruce登录，还要sudo到root
$ ansible all -m ping -u bruce -b
# 以用户bruce登录，还要sudo到用户batman
$ ansible all -m ping -u bruce -b --become-user batman
```

（如果您想用别的版本的sudo，您可以在Ansible的配置文件中指定用哪一个```sudo```。```sudo```的参数在Ansible配置文件中也可设置，比如```-H```。）

现在让我们在所有被控端上运行一个即时命令：

```
$ ansible all -a "/bin/echo hello"
```

恭喜！您刚刚通过Ansible联系了所有的被控端。接下来您可以阅读Introduction To Ad-Hoc Commands来学习更实用的例子，研究不同的模块，并且学习用Ansible的Playbook了。Ansible可不只是用来跑命令的，它有强大的配置管理和部署功能。虽然要研究的还有很多，但是您要用到的Ansible基础已经成型了！

### 被控端密钥校验

从1.2.1版本开始，Ansible会默认启用被控端密钥校验。

如果我们给一个被控端重新安装了系统，它的密钥和主控端中的“known_hosts”里面的就不一样了，此时主控端这边便会报错。如果您第一次连接某个被控端，或者您主控端的“known_hosts”列表里没有这个被控端，系统会询问您是否相信该被控端。这就导致了一次交互行为，如果您是用cron来运行Ansible的，您肯定不希望这种交互行为出现。

如果您对此处有了解，同时希望关掉密钥校验，您可以修改Ansible的配置文件```/etc/ansible/ansible.cfg```或者```~/.ansible.cfg```：

```
[defaults]
host_key_checking = False
```

此外，配置以下环境变量也可达到同样目的：

```
$ export ANSIBLE_HOST_HOST_CHECKING=False
```

被控端密钥校验在paramiko模式下的速度略慢，因此如果您使用这个功能的话，还是推荐您使用原生的“SSH”。

Ansible还会在被控端的系统日志中留下一些和模块有关的信息。如果要禁用日志，在某项任务或者play上标注“```no_log: True```”即可。我们之后还会介绍。

如果要在主控端启用基本的日志功能，请参看配置文件章节，设置配置文件中的“log_path”这一项。企业级用户也可了解一下Ansible Tower。Tower提供了非常完备的日志数据库功能，使得用户可以分别从被控端，项目，或者不同inventory的视角下查看历史数据。这些数据可以用图形化界面查看，也可以用REST API来获取。