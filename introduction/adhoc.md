## 即时命令（Ad-Hoc Commands）简介

本章我们用例子来聊一聊如何用`/usr/bin/ansible`来运行即时命令。

那什么是即时命令呢？

即时命令是您为了做一些很快就能做完的事情而编写的命令。因为只是为了完成一个临时的任务，因此您可能不会把这些命令保存下来。

在学习playbook的语法之前先来了解一下即时命令，是理解Ansible的一个非常好的起点——而且即时命令也适用于那些不需要写出整个playbook就能完成的任务。

总的来说，Ansible的真正力量体现在playbook上。那我们何必还要有即时任务这个东西？

举个例子：如果您要为了过圣诞节假期关掉您实验室中的所有服务器，您完全不用写一个playbook，一行命令就能搞定了。

如果要做配置管理和部署，您可能就需要用`/usr/bin/ansible-playbook`了——您在本章学到的概念可以直接用在playbook语法中。

（想要了解更多内容，请参看Playbook的章节。）

注意了，如果您还没看过主机清单那一章，请先大概看一下，然后我们就要开始了。

### “并行”与Shell命令

我们随便举一个例子。

假如我们现在要用Ansible命令行工具重启所有位于Atlanta的web服务器，一次十个。首先，让我们设置SSH-agent，以便它能记住我们的登录信息。

```
$ ssh-agent bash
$ ssh-add ~/.ssh/id_rsa
```

如果您不想用ssh-agent，不想用密钥来登录，而是想用密码，您可以在命令中加入`--ask-pass`（或者是`-k`）参数，但其实用ssh-agent更好。

现在，让我们在“atlanta”组内的所有服务器上执行重启命令，一次十个。

```
$ ansible atlanta -a "/sbin/reboot" -f 10
```

`/usr/bin/ansible`默认情况下会用您当前的账户执行命令，如果您不希望它这样做，您可以用`-u username`来指定一个用户。如果您希望用另一个用户来运行命令，那这个命令大概会是下面这个样子：

```
$ ansible atlanta -a "/usr/bin/foo" -u username
```

一般情况下您可能还不仅仅想用普通用户来做操作。如果您想用sudo来执行命令，可以这样：

```
$ ansible atlanta -a "/usr/bin/foo" -u username --sudo [--ask-sudo-pass]
```

如果您用的不是免密码的sudo，您需要加上`--ask-sudo-pass`（`-K`）参数。这个参数会和您做交互，询问您密码。免密码的sudo会让自动化来得更容易，但我们不强制要求您使用免密码的sudo。

如果您乐意，您还可以sudo到非root的用户上去，用`--sudo-user`（`-U`）即可：

```
$ ansible atlanta -a "/usr/bin/foo" -u username -U otheruser [--ask-sudo-pass]
```

