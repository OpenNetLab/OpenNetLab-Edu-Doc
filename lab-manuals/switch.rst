:标题: DNS Server 实验说明

:作者:
 - dingzf
 - zhangxinyv

:时间: 2024年4月23日
 
==============
Switch Lab
==============

Overview
========


交换机工作在数据链路层，在交换机收到数据帧后，根据帧的目的MAC地址和交换机内部的转发表对帧进行处理，包括：明确的转发，即交换机知道应当从哪个/哪些端口转发该帧（单播，多播，广播）；盲目的转发，即交换机不知道应当从哪个端口转发帧，只能将其通过除进入交换机的接口外的其他所有接口转发（也称为泛洪）；明确的丢弃，即交换机知道不应该转发该帧，将其丢弃。交换机是一种即插即用的设备，其内部的帧交换表是通过自学习算法自动地逐渐建立起来的。

在这个实验中，你的任务是实现一个搭建在OpenNetLab上的交换机，该交换机具有自学习功能，可以逐渐学习MAC地址对应的端口以建立转发表。


Getting Started
---------------

1. :download:`下载实验资源 <./resources/switch.zip>` ，解压后进入文件夹，其中包含基础的实验代码模板。实验代码包含如下文件：

   - :file:`main.py` ：本地调试运行文件。
   - :file:`framegenerator.py` ：帧发送方文件，无需修改。
   - :file:`switch.py` : 自学习的MAC交换机文件，TODO部分待编写。
   - :file:`testcases.json` ：本地评测配置文件。
   - :file:`tester` ：本地评测运行文件，使用说明见相关文档。
   
2. 阅读实验任务，完成代码模板中 `TODO` 部分的代码片段。

3. 进行本地测试，完成本地测试后将代码提交到OpenNetLab在线平台进行远程评测。

4. 推荐实验环境：Linux操作系统，Python版本3.6以上。

Tasks
-----

在这个实验中，framegenerator负责向switch发送帧，switch负责根据转发表完成帧的转发，你需要按照如下规则实现switch.py中的recv方法及相关细节的设计：

- 首先，你需要根据实际需求,选择合适的数据结构实现switch的forward_table（转发表）,要求表项包含TTL字段，即用于记录表项的超时时间
- recv方法表示交换机接收到帧后的处理过程，具体包括：
  
    - 在交换机表中添加（或刷新）发送者的MAC地址和收到该帧接口的表项
    - 使用该帧的目的MAC地址在转发表中检索
    - 如果存在该地址的表项

        - 如果表项的接口是收到帧的接口，丢弃该帧，不转发
        - 否则，按表项中的接口转发帧

    - 如果不存在该地址的表项，则泛洪，即将该帧从除了到达接口外的所有接口转发
  

该实验 `TODO` 部分的伪代码如下：

.. code-block:: text

    # switch.py
    forward_table to be define #根据需求选择合适的数据结构实现转发表
    function recv(frame,port_in) {
        # process the reception of frames
        add or refresh the entry in forward_table according to the frame's source MAC address and port_in
        if the frame's destination MAC address is in forward_table {
            if the port_in is the same as the port in forward_table {
                drop the frame
            }
            else {
                forward the frame according to the port in forward_table
            }
        else {
            flood the frame to all ports except port_in
        }
    
    function timeout_callback(mac_addr) {
        # process the timeout of the entry in forward_table
        remove the entry in forward_table according to the mac_addr
    }


------------

Tips
====

1.switch.py和framegenerator.py中一些属性和方法的解释：


.. py:attribute:: self.env
   :noindex:

   程序运行的环境。

.. py:attribute:: self.debug
   :noindex:

   控制运行过程中日志信息输出，可以设置为True以方便调试。

.. py:attribute:: self.proc
   :noindex:

   客户端的进程；

.. py:attribute:: self.log
   :noindex:

   交换机转发帧的端口日志记录；

.. py:attribute:: self.outs
   :noindex:

   交换机的输出端口；

.. py:function:: run(self, env) 
   :noindex:

   在给定的环境中运行客户端。

   @参数: env - 程序的运行环境

.. py:function:: dprint(self, msg)
   :noindex:

   打印运行日志信息。

   @参数: msg - 需要打印的信息

.. py:function:: forward(self, port_id frame)
   :noindex:

   向给定端口转发帧，并记录转发的端口信息。

   @参数: port_id - 转发端口号

   @参数: frame - 数据帧

 


2.framegenerator、switch之间通过Cable完成帧的传输,switch的每个端口都连接一条Cable(可以进行双向传输)。

3.若switch.log与expected_res一致，则说明设计成功。


------------

Testing
=======

1. 进行本地测试

在本机运行 *main.py* 程序。 *main.py* 程序会使用本地的一个测试用例对switch及endpoint的正确性进行评测并输出运行日志。

.. code-block:: shell

    python3 main.py


.. note::
    `main.py` 首先实例化类 `Environment` ，创建一个基于事件的网络模拟执行环境。然后 `main.py` 在模拟环境中实例化 FrameGenerator、EndPoint 、Switch类，并将三者连接进行帧传输。


完成程序基本功能的调试后，可以运行可执行文件 `tester` 进行多个测试用例评测，更详细的使用说明见 `相关文档 <./tester.html>`_


1. `提交代码进行远程评测 <http://20.247.32.90>`_




