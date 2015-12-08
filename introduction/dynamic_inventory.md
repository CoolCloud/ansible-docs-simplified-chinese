## 动态主机清单

在用配置管理系统的用户很可能想用一套独立的系统来存储主机清单。在上一章，我们介绍了Ansible提供的文本格式的主机清单，但如果您想用存在别处的主机清单，该怎么办呢？

例如，您的主机清单可能存在云主机提供商那里，可能存在LDAP里，Cobbler里，或者那些企业级的CMDB里。

其实，Ansible能通过一套对外的主机清单系统，轻易地支持上面提到的这几样东西。清单文件夹中早已支持了一些云系统——包括EC2/Eucalyptus，Rackspace Cloud和OpenStack，这些中的某些例子会在下面详细介绍。

Ansible Tower产品还提供了一个能存储主机清单的数据库服务，这份清单用Web和REST API都可以访问到。Tower同步您所有的动态主机清单来源，甚至还提供了一个图形化的主机清单编辑器。借助数据库存储您的所有主机信息，您可以方便地关联历史事件，查看到哪些主机在上次运行playbook动作时失败了。

如果您想了解如何编写自己的动态主机清单源，请查阅Developing Dynamic Inventory Sources。

### 案例：来自Cobbler的动态主机清单

想必很多Ansible的用户都是Cobbler的用户，尤其是您管理的物理设备非常多得时候更有可能。（注：Cobbler最开始的作者时Michael Dehaan，如今是由James Cammarata主导的，后者还是Ansible公司的雇员。）

Cobbler的核心目的是操作系统安装的自动化，以及DHCP和DNS的管理。然而它还能为多个配置管理系统提供数据（甚至还能同时提供），有些系统管理员甚至已经把Cobbler看成是“小型的CMDB”了。

如果您要将Ansible的主机清单文件和Cobbler绑定，请复制这个脚本到`/etc/ansible/`，然后对该文件做`chmod +x`操作。您使用Ansible时，要保证`cobblerd`在运行，您还需要在Ansible的命令行工具上添加`-i`参数（例如：`-i /etc/ansible/cobbler.py`）。这份脚本会调用Cobbler的XMLRPC API和Cobbler进行通信。

首先，让我们先直接运行这个`/etc/ansible/cobbler.py`脚本来直接测试一下。您应该能看见JSON格式的输出结果，但是里面可能还什么都没有。

让我们看看它都做了什么。在Cobbler中，假设有如下的配置设定：

```
cobbler profile add --name=webserver --distro=CentOS6-x86_64
cobbler profile edit --name=webserver --mgmt-classes="webserver" --ksmeta="a=2 b=3"
cobbler system edit --name=foo --dns-name="foo.example.com" --mgmt-classes="atlanta" --ksmeta="c=4"
cobbler system edit --name=bar --dns-name="bar.example.com" --mgmt-classes="atlanta" --ksmeta="c=5"
```

在这个例子中，Ansible可以直接通过“foo.example.com”来连接到这个系统，但是也可以通过组名“webserver”或“atlanta”来连接。因为Ansible用的是SSH，我们只会用“foo.example.com”来尝试连接foo这个服务器，而不会用“foo”。类似地，如果您尝试用`ansible foo`，也一样找不到服务器。但是`ansible 'foo*'`是可以的，因为这个系统的DNS名称是以'foo'开头的。

这个脚本不仅仅只提供主机和组的信息。此外，如果运行了`setup`模块（使用playbooks的时候会触发），变量“a”、“b”、“c”会自动填充进模板中：

```
# file: /srv/motd.j2
Welcome, I am templated with a value of a={{ a }}, b={{ b }}, and c={{ c }}
```

然后您便可以像下面这样执行了：

```
ansible webserver -m setup
ansible webserver -m template -a "src=/tmp/motd.j2 dest=/etc/motd"
```

>注意：
>“webserver”这个名字是从cobbler来的，配置文件中的变量也是。不过您还是可以用正常的方式传参数给Ansible。外部清单脚本中的同名参数会覆盖掉手动指定的参数。

这样，使用上面的模板（motd.j2），对于“foo”这个系统来说，下面的数据便会写进`/etc/motd`中：

```
Welcome, I am templated with a value of a=2, b=3, and c=4
```

对于“bar”这个系统（bar.example.com）来说则是：

```
Welcome, I am templated with a value of a=2, b=3, and c=5
```

还有，理论上说，下面这样做也是有效地，虽然没什么实际意义：

```
ansible webserver -m shell -a "echo {{ a }}"
```

也就是说，您在传参数，或者执行一些动作时，也可以用这些变量。

### 案例：来自AWS EC2的动态主机清单

（因为目前我们不怎么用AWS，因此本节内容先略了。回头再补。）