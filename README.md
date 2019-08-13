# 如何在Ubuntu 16.04安装Linux，Apache，MySQL和PHP（LAMP）


## 介绍
“LAMP”是一组开放源代码软件，通常安装在一起以使服务器能够托管动态网站和网络应用。 这个词其实是代表第linux下的操作系统，与Apache Web服务器的缩写。 站点数据存储在MySQL数据库，和动态内容用PHP处理。 
在本指南中，我们将得到一个安装在Ubuntu 16.04 上的LAMP。 Ubuntu将满足我们的第一个需求：一个Linux操作系统。

## 先决条件
你本指南开始之前，你应该有一个独立的，非root用户帐户sudo设置您的服务器上的权限。 你可以学习如何通过完成[1-4的步骤](https://www.howtoing.com/initial-server-setup-with-ubuntu-16-04/)做这个初始服务器设置为Ubuntu 16.04 。


## 第1步：在防火墙中安装Apache和允许
Apache Web服务器是世界上最流行的Web服务器之一。 它有良好的文档，并已被广泛使用的网络的大部分历史，这使它成为托管网站的一个伟大的默认选择。  
我们可以很容易地使用Ubuntu的包管理器，安装Apache apt 。 软件包管理器允许我们从由Ubuntu维护的软件库中安装大多数软件。 您可以了解更多关于如何使用apt在这里。 
为了我们的目的，我们可以通过键入以下命令开始：
```
sudo apt-get update
sudo apt-get install apache2
```
由于我们使用sudo命令，这些行动得到以root权限执行。 它将要求您提供常规用户的密码以验证您的意图。 
一旦你输入你的密码， apt会告诉你哪些包，计划安装多少他们会占用额外的磁盘空间。 按Y，然后按Enter继续，安装将继续进行。

## 将“全局服务器名称”设置为“禁止语法警告”
下一步，我们将一个行添加到/etc/apache2/apache2.conf的文件来抑制一个警告消息。 而无害的，如果你不设置ServerName全球范围内，你会检查语法错误Apache配置时收到以下警告：

```
sudo apache2ctl configtest
```
```
AH00558: apache2: Could not reliably determine the server's fully qualified domain name, using 127.0.1.1. Set the 'ServerName' directive globally to suppress this message
Syntax OK
```

使用文本编辑打开主配置文件：
```
sudo nano /etc/apache2/apache2.conf
```
在内部，在文件的底部，添加一个ServerName指令，指向您的主域名。 如果您没有与服务器关联的域名，则可以使用服务器的公共IP地址：
**NOTE:** 如果你不知道你的服务器的IP地址，跳过了一节**如何找到您的服务器的公网IP地址**找到它。
```
ServerName server_domain_or_IP
```
保存并在完成后关闭文件。 
接下来，通过键入以检查语法错误：
```
sudo apache2ctl configtest
```
由于我们增加了全球ServerName指令，你应该看到的是：
```
Syntax OK
```

重新启动Apache以实现您的更改：
```
service apache2 restart
```
您现在可以开始调整防火墙。

## 调整防火墙以允许Web流量
接下来，假设您已按照[初始服务器设置说明启用UFW防火墙](https://www.howtoing.com/initial-server-setup-with-ubuntu-16-04/)，请确保防火墙允许HTTP和HTTPS流量。 您可以确保UFW具有Apache的应用程序配置文件，如下所示：

```
sudo ufw app list
```
```
Available applications:
  Apache
  Apache Full
  Apache Secure
  OpenSSH
```
如果你看一下Apache Full轮廓，它应该显示它使交通到端口80和443：

```
sudo ufw app info "Apache Full"
```
```
Profile: Apache Full
Title: Web Server (HTTP,HTTPS)
Description: Apache v2 is the next generation of the omnipresent Apache web
server.

Ports:
  80,443/tcp
```

允许此配置文件的传入流量：
```
sudo ufw allow in "Apache Full"
```
您可以立即进行即时检查，以验证一切都按计划通过访问您的服务器的公共IP地址在您的网络浏览器（请参阅下一个标题下的说明，以了解您的公共IP地址是如果你没有这个信息已经）：

```
http://your_server_IP_address
```
您将看到默认的Ubuntu 16.04 Apache网页，这是为了信息和测试目的。 它应该看起来像这样

![apache home page](/media/apache2.PNG)  
如果您看到此页面，那么您的Web服务器现在可以正确安装并通过防火墙访问。

## 如何查找您的服务器的公共IP地址
如果你不知道你的服务器的公共IP地址是什么，有很多方法你可以找到它。 通常，这是您用于通过SSH连接到服务器的地址。   
从命令行，你可以找到这几种方法。 首先，你可以使用iproute2工具，通过键入这让你的地址：
```
ip addr show eth0 | grep inet | awk '{ print $2; }' | sed 's/\/.*$//'
```
这将给你两三线回来。 他们都是正确的地址，但您的计算机可能只能使用其中一个，所以请随意尝试每一个。 
另一种方法是使用curl工具联系外线来告诉你如何看待你的服务器。 您可以通过询问特定服务器您的IP地址是这样做的：
```
sudo apt-get install curl
curl http://icanhazip.com
```
无论使用哪种方法获取您的IP地址，您都可以在浏览器的地址栏中输入您的服务器。

## 第2步：安装MySQL
现在我们已经启动并运行了Web服务器，现在是安装MySQL的时候了。 MySQL是一个数据库管理系统。 基本上，它将组织并提供对我们的网站可以存储信息的数据库的访问。 
同样，我们可以使用apt获取并安装我们的软件。 这一次，我们还将安装一些其他“帮助”包，这将帮助我们使我们的组件相互通信：
```
sudo apt-get install mysql-server
```

**注**：在这种情况下，您不必运行sudo apt-get update前的命令。 这是因为我们最近在上面的命令中运行它来安装Apache。 我们计算机上的软件包索引应该是最新的。 
同样，您将看到一个将要安装的软件包列表，以及它们将占用的磁盘空间量。 输入y继续。 
在安装过程中，您的服务器将要求您选择并确认MySQL“root”用户的密码。 这是MySQL中的管理帐户，具有增加的权限。 可以认为它类似于服务器本身的根帐户（但是现在配置的是一个特定于MySQL的帐户）。 请确保这是一个强大的，唯一的密码，不要留空。 
当安装完成后，我们要运行一个简单的安全脚本，它将删除一些危险的默认值，并锁定对我们的数据库系统的访问一点。 通过运行以下命令来启动交互式脚本：

```
sudo mysql_secure_installation
```
将要求您输入为MySQL root帐户设置的密码。 接下来，如果你想配置你会被要求VALIDATE PASSWORD PLUGIN 。 
警告：启用此功能是一个主观判断的东西。
如果启用，那么不符合指定条件的密码将被MySQL拒绝并报错。
如果您使用弱密码与自动配置MySQL用户凭据的软件（例如phpMyAdmin的Ubuntu软件包）结合使用，将会导致问题。
可以安全地禁用验证，但是您应该始终对数据库凭据使用强的，唯一的密码。
回答y（是），或其他任何继续不启用。 
>>
  VALIDATE PASSWORD PLUGIN can be used to test passwords
  and improve security. It checks the strength of password
  and allows the users to set only those passwords which are
  secure enough. Would you like to setup VALIDATE PASSWORD plugin?

  Press y|Y for Yes, any other key for No:
>>
系统将要求您选择一个级别的密码验证。 请记住，如果你输入2，为最高水平，你会尝试设置不包含数字，大小写字母和特殊字符，或者它是基于通用的字典单词的密码时收到错误。 
>>
There are three levels of password validation policy:

LOW    Length >= 8
MEDIUM Length >= 8, numeric, mixed case, and special characters
STRONG Length >= 8, numeric, mixed case, special characters and dictionary                  file

Please enter 0 = LOW, 1 = MEDIUM and 2 = STRONG: 1
>>
如果启用了密码验证，则会显示现有root密码的密码强度，并询问您是否要更改该密码。 如果你很高兴与您当前密码，输入n为“无”的提示： 
>>
Using existing password for root.

Estimated strength of the password: 100
Change the password for root ? ((Press y|Y for Yes, any other key for No) : n
>>
对于剩余的问题，您应该按Y和命中每个提示Enter键。 这将删除一些匿名用户和测试数据库，禁用远程根登录，并加载这些新规则，以便MySQL立即尊重我们所做的更改。 
此时，您的数据库系统现已设置，我们可以继续。

## 第3步：安装PHP
PHP是我们的设置的组件，它将处理代码以显示动态内容。 它可以运行脚本，连接到我们的MySQL数据库以获取信息，并将处理的内容传递到我们的Web服务器以显示。 
我们可以再次利用apt系统中安装我们的组件。 我们还将包括一些帮助程序包，以便PHP代码可以在Apache服务器下运行，并与我们的MySQL数据库通信：
```
sudo apt-get install php5 libapache2-mod-php5 php5-mcrypt php5-mysql
```
这应该安装PHP没有任何问题。 我们稍后将测试一下。 
在大多数情况下，我们希望修改Apache在请求目录时提供文件的方式。 目前，如果用户从服务器请求一个目录，Apache将首先寻找一个名为index.html 。 我们要告诉我们的Web服务器更喜欢PHP文件，所以我们让Apache寻找一个index.php文件中的第一。 
要做到这一点，键入此命令打开dir.conf与root权限的文本编辑器文件中：
```
sudo nano /etc/apache2/mods-enabled/dir.conf
```
它将如下所示：
```/etc/apache2/mods-enabled/dir.conf
<IfModule mod_dir.c>
    DirectoryIndex index.html index.cgi index.pl index.php index.xhtml index.htm
</IfModule>
```
我们要移动的PHP索引文件上文所强调的之后的第一个位置DirectoryIndex规范，就像这样：

```/etc/apache2/mods-enabled/dir.conf
<IfModule mod_dir.c>
    DirectoryIndex index.php index.html index.cgi index.pl index.xhtml index.htm
</IfModule>
```
当你完成后，保存并按下Ctrl-X关闭文件。 你必须确认键入y保存，然后按Enter键确认该文件的保存位置。 
之后，我们需要重新启动Apache Web服务器，以便识别我们的更改。 您可以输入以下命令：
```
service apache2 restart
```
我们也可以检查的状态apache2使用服务systemctl ：
```
service apache2 status
```
```
apache2.service - LSB: Apache2 web server
   Loaded: loaded (/etc/init.d/apache2; bad; vendor preset: enabled)
  Drop-In: /lib/systemd/system/apache2.service.d
           └─apache2-systemd.conf
   Active: active (running) since Wed 2016-04-13 14:28:43 EDT; 45s ago
     Docs: man:systemd-sysv-generator(8)
  Process: 13581 ExecStop=/etc/init.d/apache2 stop (code=exited, status=0/SUCCESS)
  Process: 13605 ExecStart=/etc/init.d/apache2 start (code=exited, status=0/SUCCESS)
    Tasks: 6 (limit: 512)
   CGroup: /system.slice/apache2.service
           ├─13623 /usr/sbin/apache2 -k start
           ├─13626 /usr/sbin/apache2 -k start
           ├─13627 /usr/sbin/apache2 -k start
           ├─13628 /usr/sbin/apache2 -k start
           ├─13629 /usr/sbin/apache2 -k start
           └─13630 /usr/sbin/apache2 -k start

Apr 13 14:28:42 ubuntu-16-lamp systemd[1]: Stopped LSB: Apache2 web server.
Apr 13 14:28:42 ubuntu-16-lamp systemd[1]: Starting LSB: Apache2 web server...
Apr 13 14:28:42 ubuntu-16-lamp apache2[13605]:  * Starting Apache httpd web server apache2
Apr 13 14:28:42 ubuntu-16-lamp apache2[13605]: AH00558: apache2: Could not reliably determine the server's fully qualified domain name, using 127.0.1.1. Set the 'ServerNam
Apr 13 14:28:43 ubuntu-16-lamp apache2[13605]:  *
Apr 13 14:28:43 ubuntu-16-lamp systemd[1]: Started LSB: Apache2 web server.
```
## 第4步：在Web服务器上测试PHP处理
为了测试我们的系统是否正确配置为PHP，我们可以创建一个非常基本的PHP脚本。 
我们称这个脚本info.php 。 为了使Apache找到该文件并正确地提供它，它必须保存到一个非常特定的目录，这被称为“web根”。 
在Ubuntu 14.04，该目录位于/var/www/html/ 。 我们可以通过键入以下内容在该位置创建文件：
```
sudo nano /var/www/html/info.php
```
这将打开一个空白文件。 我们想把以下文本，这是有效的PHP代码，在文件内：
```
<?php
phpinfo();
?>
```
完成后，保存并关闭文件。 
现在我们可以测试我们的Web服务器是否可以正确显示由PHP脚本生成的内容。 要尝试这个，我们只需要在我们的网络浏览器访问此页面。 您将需要您的服务器的公共IP地址。 
您要访问的地址将是：
```
http://your_server_IP_address/info.php
```
你来的页面应该看起来像这样：
![php home page](/media/php5.PNG)
从PHP的角度来看，这个页面基本上提供了你的服务器的信息。 它对于调试和确保您的设置正确应用非常有用。 
如果这是成功的，那么你的PHP工作正常。 
您可能想要在此测试后删除此文件，因为它实际上可能会向未经授权的用户提供有关您的服务器的信息。 为此，您可以键入：
```
sudo rm /var/www/html/info.php
```

如果以后需要再次访问信息，您可以随时重新创建此页面。

## 第5步：安装phpMyAdmin
phpMyAdmin是一个MySQL数据库的Web管理界面，要方便使用MySQL，推荐安装它:
```
apt-get install phpmyadmin
```
如果遇到提示，这样选择：
```
Web server to reconfigure automatically: <-- apache2
Configure database for phpmyadmin with dbconfig-common? <-- No
```
然后，你可以访问phpMyAdmin: 
```
http://your_server_IP_address/phpmyadmin
```

如果遇到如下错误
![Error access phpmyadmin](/media/phpmyadminerror.PNG)

使用如下解决方案：
```
sudo ln -s /etc/phpmyadmin/apache.conf /etc/apache2/conf-available/phpmyadmin.conf
sudo ln -s /usr/share/phpmyadmin /var/www/html/phpmyadmin
sudo service apache2 restart
```
然后，重新访问phpMyAdmin


## 结论
现在你已经安装了一个LAMP，你有许多选择下一步做什么。 基本上，您已经安装了一个平台，将允许您在服务器上安装大多数类型的网站和Web软件。 
作为紧接着的下一步，您应该确保通过HTTPS提供与Web服务器的连接是安全的。 这里最简单的方法是用Let的加密与自由的TLS / SSL证书保护您的网站。

[Ubuntu 16.04初始服务器设置](https://www.howtoing.com/initial-server-setup-with-ubuntu-16-04/)  
[如何在Ubuntu 16.04安装的Git](https://www.howtoing.com/how-to-install-git-on-ubuntu-16-04)
