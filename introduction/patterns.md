## 匹配式（Pattern）

匹配式（Pattern）是我们用来告诉Ansible要管哪些服务器的样式。匹配式代表了我们要和哪些主机做交互，但是在用playbook时它的实际意义是哪个主机匹配哪一个配置，或者哪一个主机匹配哪一个IT流程。

我们会在Introduction To Ad-Hoc Commands这一节介绍如何使用命令行，但是我们现在先略微窥探一下。命令行的格式大概是这样子的：

```
ansible <pattern_goes_here> -m <module_name> -a <arguments>
```

举一个更具体的例子：

```
ansible webservers -m service -a "name=httpd state=restarted"
```

一个匹配式，一般都能匹配到几个组（每个组是若干服务器的集合）——在上例中，匹配到的是在“webservers”组中的机器。

总之，如果要用Ansible，您首先就要学会告诉Ansible您要和哪些服务器交流。这件事就是通过制定特定的主机名，或者组来完成的。

下面这两个匹配式是等价的，指向了主机清单中的所有主机：

```
all
*
```

您也可以指定某个主机，或者由名字决定的一组主机：

```
one.example.com
one.example.com:two.example.com
192.168.1.50
192.168.1.*
```

下面这种匹配式匹配到了一个或多个组。由冒号分隔的组意思是“或”。这意味着想要管理的主机要么在前面的组，要么在后面的组：

```
webservers
webservers:dbservers
```

您也可以排除组，比如，所有在webserver组，但是不在phoenix组的主机：

```
webservers:!phoenix
```

您还可以指定两个组的交集。这就意味着要操作的主机必须同时在`webserver`和`staging`这两个组中。

```
webserver:&staging
```

您还可以把上面的用法结合使用：

```
webservers:dbservers:&staging:!phoenix
```

上面的