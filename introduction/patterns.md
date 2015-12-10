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

这个例子的意思是：所有在“webservers”和“dbservers”中的主机，只要在“staging”中的就会被管理，但是在“phoenix”中的主机就不会被管理了……哎呀我的天呐！

如果您希望用“-e”参数将组的标示符传递给`ansible-playbook`，您也可以在匹配式中使用变量。不过很少有人这么做：

```
webservers:!{{excluded}}:&{{required}}
```

您也不必拘泥于完整的组名。主机名，IP地址和组名中都可以用通配符：

```
*.example.com
*.com
```

而且，在匹配式中，把通配符形式和组名混着用也一样没问题。

您可以根据一个主机，或者几个主机在组的位置编号来选择操作哪些主机。比如，我们有这样一个组：

```
[webservers]
cobweb
webbing
weber
```

您可以通过给组添加序号的方式来选定主机。

```
webservers[0]	# == cobweb
webservers[-1]	# == weber
webservers[0:1]	# == webservers[0],webservers[1]
				# == cobweb,webbing
webservers[1:]	# == webbing,weber
```

很少有人会用正则表达式来写匹配式，但这样做也是可以的。只要在匹配式的前面加上“~”即可：

```
~(web|db).*\.example\.com
```

下面的内容可能稍微有点儿超前：您可以在使用`ansible`或者`ansible-playbook`时，通过添加`--limit`参数来添加排除规则：

```
ansible-playbook site.yml --limit datacenter2
```

如果您想从文件中读取主机列表作为参数传递，从Ansible 1.2开始在文件名前加上“@”即可。

```
ansible-playbook site.yml --limit @retry_hosts.txt
```

就这么简单。要想了解如何应用我们在这个地方学到的知识，请参阅Introduction To Ad-Hoc Commands和Playbooks。

>注意：
>主机列表的分隔符不光可以是“:”，也可以是“,”。我们尤其推荐您在处理范围，或者是IPv6的时候使用“,”。这个特性在Ansible 1.9中不适用。

>注意：
>到了Ansible 2.0，“;”不能再作为主机列表的分隔符使用了。