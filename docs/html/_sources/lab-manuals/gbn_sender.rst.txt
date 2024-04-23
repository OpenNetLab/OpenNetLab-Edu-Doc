:标题: GBN Sender实验说明

:作者:
 - yangfan
 - dingzf
 - zhengbw

:时间: 2023年8月6日

==============
GBN Sender Lab
==============

Overview
========


Go-Back-N是自动重复请求(ARQ)协议的一个特定实例，在该协议中，即使没有收到来自接收方的确认(ACK)包，发送过程也会继续发送由窗口大小指定的分组数。它是一般滑动窗口协议的一种特殊情况，发送窗口大小为N，接收窗口大小为1。当收到接收方的确认时，发送方将窗口向后移动，并且发送其他可以发送的分组。

在这个实验中，你的任务是实现一个搭建在OpenNetLab上的使用GBN协议的发送端。

Getting Started
---------------

1. :download:`下载实验资源 <./resources/gbn.zip>` ，解压后进入gbn文件夹，其中包含基础的实验代码模板。实验代码包含如下文件：

   - :file:`main.py` ：本地调试运行文件；
   - :file:`receiver.py` ：接收方文件，无需修改；
   - :file:`sender.py` ：发送方文件， `TODO` 部分待编写。
   - :file:`testcases.json` ：本地评测配置文件。
   - :file:`tester` ：本地评测运行文件，使用说明见相关文档。

2. 阅读实验任务，完成代码模板中 `TODO` 部分的代码片段。

3. 进行本地测试，完成本地测试后将代码提交到OpenNetLab在线平台进行远程评测。

Tasks
-----

在这个实验中，你需要将一个字符串中的每个字符封装为分组发送给接收方，并且遵循GBN协议：

- 开始时发送滑动窗口内所有可以发送的分组。

- 每发送一个分组，保存该分组在缓冲区中，表示已发送但还未被确认。

- 缓冲区使用一个定时器，当定时器超时的时候，重新发送缓冲区中的所有分组。

- 当收到接收方的确认后，判断该确认是否有效。如果无效的话，什么也不做；如果有效的话，采取累计确认，移动滑动窗口，将已经被确认的分组从缓冲区中删除，并且发送接下来可以发送的分组，重置定时器。

该实验 `TODO` 部分的伪代码如下：

.. code-block:: text

    function run() {
        # send available packets, add these packets to buffer
        while (sequence number is available) {
            encapsulate messages into new packets 
            send new packets
            add these packets to buffer
        }
        reset timer
        Check whether to terminate the sending process
    }

    function put(packet) {
        # Process packets received from the receiver
        if ackno is valid {
            remove acked packets from the buffer
        }
        send the following available packets
        if all packets are sent and acked {
            Inform to terminate the sending process
        }
    }

    function timeout_callback() {
        for all packets in the buffer {
            send packet
        }
    }

------------

Tips
====

你可以通过 `GBN交互演示 <https://www2.tkn.tu-berlin.de/teaching/rn/animations/gbn_sr/>`_ 来帮助自己更好地理解GBN的过程。


sender.py和receiver.py中一些属性和方法的解释：


.. py:attribute:: self.seqno_range
   :noindex:

   序号空间大小。请注意序号空间的序号是从0开始的；

.. py:attribute:: self.window_size
   :noindex:

   滑动窗口大小；

.. py:attribute:: self.timer
   :noindex:

   发送方的定时器；

.. py:attribute:: self.env
   :noindex:

   程序运行的环境；

.. py:attribute:: self.debug
   :noindex:

   控制运行过程中日志信息输出，可以设置为True以方便调试；

.. py:attribute:: self.proc
   :noindex:

   发送方的进程；

.. py:function:: new_packet(self, seqno, data) -> Packet
   :noindex:

   将需要传输的信息内容封装为一个数据包；

   @参数: seqno - 数据包的序列号

   @参数: data - 数据包所携带的数据内容

   @返回类型: 已封装好的一个数据包实例


.. py:function:: timeout_callback(self)
   :noindex:

   超时处理的回调函数，当gbn sender中的超时器超时会自动调用该函数，你需要在该函数中发送所有未收到确认的数据包；

.. py:function:: send_packet(self, packet)
   :noindex:

   发送数据包packet到receiver；

   @参数: packet - 需要发送的一个数据包实例

.. py:function:: run(self, env) 
   :noindex:

   在给定的环境中运行发送端；

   @参数: env - 程序的运行环境；

.. py:function:: put(self, packet)
   :noindex:

   处理确认数据包并且发送余下的数据；

   @参数: packet - sender接受到的来自receiver的确认数据包

.. py:function:: dprint(self, msg)
   :noindex:

   打印运行日志信息。

   @参数: msg - 需要打印的信息

------------

Testing
=======

1. 进行本地测试

在本机运行 *main.py* 程序。 *main.py* 程序会使用本地的一个测试用例对GBN发送端的正确性进行评测并输出运行日志。

.. code-block:: shell

    python3 main.py


.. note::
    `main.py` 首先实例化类 `Environment` ，创建一个基于事件的网络模拟执行环境。然后 `main.py` 在模拟环境中实例化 GBNsender、GBNreceiver、Wire 类，创建由 `sender` , `receiver` , `wire1` , `wire2` 构成的网络回路进行数据包传输，`Wire` 类主要目的是实现 `sender` 与 `receiver` 之间不可靠的数据传输（模拟数据包的发送时延、丢失和乱序的情况），流程主要分为四步：

    1. `sender` 通过 `wire1` 实现数据包的发送
    2. `receiver` 通过 `wire1` 中的方法，获取 `sender` 发送的数据包
    3. `receiver` 通过 `wire2` 实现数据包的发送
    4. `sender` 通过 `wire2` 中的方法，获取 `receiver` 发送的确认数据包

.. figure:: ./images/main.png
   :name: gbn_main_process
   :align: center
   :width: 600px
   :alt: image cannot be loaded

   main.py 模拟流程



完成程序基本功能的调试后，可以运行可执行文件 `tester` 进行多个测试用例评测，更详细的使用说明见 `相关文档 <./tester.html>`_

2. `提交代码进行远程评测 <http://20.247.32.90>`_
