:标题: DNS Server 实验说明

:作者:
 - wangcenman

:时间: 2023年8月6日
 
==============
DNS Server Lab
==============

Overview
========


DNS（Domain Name System）是互联网中用于将域名（例如www.example.com）转换为对应IP地址的系统。在互联网上，设备之间通信需要使用IP地址进行定位，但IP地址不便于人们记忆和使用。DNS充当了一个类似电话簿的角色，将易于记忆的域名映射到相应的IP地址，从而使得用户可以通过域名来访问网站或者发送电子邮件，而无需记住复杂的IP地址。当用户在浏览器中输入一个域名时，系统会通过DNS查询将域名解析为对应的IP地址，然后进行通信。DNS起到了极为重要的转换和定位功能，是互联网通信的基础。

在这个实验中，你的任务是实现一个搭建在OpenNetLab上的DNS服务器，该服务器具备拦截特定域名，以及返回域名对应IP的功能。

Getting Started
---------------

1. :download:`下载实验资源 <./resources/dns.zip>` ，解压后进入dns文件夹，其中包含基础的实验代码模板。实验代码包含如下文件：

   - :file:`main.py` ：本地调试运行文件。
   - :file:`ipconf.txt` : 本地调试配置文件。
   - :file:`client.py` ：客户端文件，无需修改。
   - :file:`server.py` ：服务器文件， `TODO` 部分待编写。
   - :file:`dns_packet.py` : DNS数据包解析和编码文件，无需修改。
   - :file:`testcases.json` ：本地评测配置文件。
   - :file:`tester` ：本地评测运行文件，使用说明见相关文档。
   
2. 阅读实验任务，完成代码模板中 `TODO` 部分的代码片段。

3. 进行本地测试，完成本地测试后将代码提交到OpenNetLab在线平台进行远程评测。

4. 推荐实验环境：Linux操作系统，Python版本3.6以上。

Tasks
-----

在这个实验中，服务器从配置文件 :file:`ipconf.txt` 中读取 ``domain name --- IP address`` 的映射。你需要按照如下规则处理DNS请求：

- Intercept：如果请求的DNS域名在配置文件的映射中并且对应的IP为 ``0.0.0.0`` ，对该域名进行拦截，向客户端返回“域名不存在”的报错信息。

- Local Resolve：如果请求的DNS域名在配置文件的映射中并且对应的IP是合法的，返回对应IP地址给客户端。

- Relay：如果请求的DNS域名不在配置文件，将对应的DNS请求转发给公网上的DNS服务器，当收到该服务器的应答时，将应答转发给客户端。

该实验 `TODO` 部分的伪代码如下：

.. code-block:: text
 
    function recv_callback(data){
        # process DNS requests
        resolve data to DNS frame recvdp
        if recvdp is a query message{
            if recvdp.qname is in url_ip table{
                if url_ip[recvdp.qname]="0.0.0.0"{
                    generate a reply error data
                }
                else{
                    generate response data
                }
            }
            else{
                send query message to public DNS server
                receive data from public server
            }
            send data to client
        }
    }

------------

Tips
====

1.client.py和server.py中一些属性和方法的解释：


.. py:attribute:: self.env
   :noindex:

   程序运行的环境。

.. py:attribute:: self.debug
   :noindex:

   控制运行过程中日志信息输出，可以设置为True以方便调试。

.. py:attribute:: self.proc
   :noindex:

   客户端的进程；

.. py:function:: run(self, env) 
   :noindex:

   在给定的环境中运行客户端。

   @参数: env - 程序的运行环境

.. py:function:: recv_callback(self, data)
   :noindex:

   接收并保存服务器发送的DNS应答数据包。使用 ``self.finish_channel.put(True)`` 来终止客户端的发送进程。

   @参数: data - 接收到的数据


2.服务器通过socket与公共DNS服务器建立通信连接，以发送接收DNS数据包。以下代码片段表示服务器向上游DNS服务器（指定为”223.5.5.5”，port 53）发送DNS查询，并监听本地的1088端口得到查询结果。

.. code-block:: python

    self.name_server = ("223.5.5.5", 53)
    self.server_socket = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
    self.server_socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
    self.server_socket.bind(("", 1088))
    self.server_socket.setblocking(True)

3.DNSClient类和DNSServer类都继承了UDPDevice类，服务器如何向客户端发送数据包可以参考client.py中如何发送DNS请求数据包。


------------

Testing
=======

1. 进行本地测试

在本机运行 *main.py* 程序。 *main.py* 程序会使用本地的一个测试用例对DNS服务器的正确性进行评测并输出运行日志。

.. code-block:: shell

    python3 main.py


.. note::
    `main.py` 首先实例化类 `Environment` ，创建一个基于事件的网络模拟执行环境。然后 `main.py` 在模拟环境中实例化 DNSClient、DNSServer 类，并直接连接客户端、服务器的实例进行数据包传输。


完成程序基本功能的调试后，可以运行可执行文件 `tester` 进行多个测试用例评测，更详细的使用说明见 `相关文档 <./tester.html>`_


2. `提交代码进行远程评测 <http://20.247.32.90>`_




