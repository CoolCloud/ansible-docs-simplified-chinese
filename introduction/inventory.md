## Inventory

Ansible可以同时对您治下的多个系统做操作。它完成这项工作时，要读取您存储在Ansible的Inventory文件中的系统信息。默认情况下，Inventory文件指的是```/etc/ansible/hosts```这个文件。

您不仅能编辑Inventory文件，还能同时使用多个Inventory文件。更为强大的是，您可以动态的获取Inventory文件，或者从云资源中下载到它，我们在Dynamic Inventory这一章会有更详细的介绍。

### Hosts和Groups

```/etc/ansible/hosts```文件的格式和INI文件的格式很像，如下所示：

```
mail.example.com

[webservers]
foo.example.com
bar.example.com

[dbservers]
one.example.com
two.example.com
three.example.com
```

方括号中的是组名，我们可以用组名来给治下的不同系统分组，这样方便在操作时选择要操作哪些组，什么时候操作，以及为什么操作。

一个系统可以在不同的组中，因为一个服务器很可能既是web服务器又是数据库服务器。如果您的确将一个系统安排在了多个组中，请注意：施加在该系统的变量也同样会来自多个组，变量的优先级会在后面的章节讲到。

如果您治下的某个主机没有用标准的SSH端口，您可以将端口号写在主机名后面，用冒号隔开。在您SSH配置中列出的端口号是不会用在Paramiko上的，而是会用在OpenSSH上。

为了让配置文件更清楚明了，如果您管理的服务器没有用默认端口，我们建议您将它写在Inentory中：

```
badwolf.example.com:5309
```

如果您只有被控端的固定IP，并且想要在您主控端的inventory文件中给它们设置别名，或者您是用隧道连接的。您也可以用如下的方式来描述这些被控端：

```
jumper ansible_port=5555 ansible_host=192.168.1.50
```

在上面的例子中，如果您对“jumper”这一主机别名（这可能干脆不是主机名）执行ansible命令，ansible会尝试连接192.168.1.50的5555端口。我们在本例中使用了inventory的一些功能来定义特殊变量。不过，如果您打算通过定义变量的方式来描述系统策略，本例的做法并不是最好的，我们稍后再来进行改进，现在先学会起步就好。

要添加很多主机么？如果您这些主机名都遵循大致一样的格式，您可以用下面的方法，就不用一行一行写了。

```
[webserver]
ww[01:50].example.com
```

对于数字格式，前置0可以加上，也可以不加，您说了算。所列范围的开头和结尾也在这个组中。您还可以使用字母表范围：

```
[database]
db-[a:f].example.com
```

