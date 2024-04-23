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


交换机工作在数据链路层，在交换机收到数据帧后，根据帧的目的MAC地址和交换机内部的帧交换表对帧进行处理，包括：明确的转发，即交换机知道应当从哪个/哪些端口转发该帧（单播，多播，广播）；盲目的转发，即交换机不知道应当从哪个端口转发帧，只能将其通过除进入交换机的接口外的其他所有接口转发（也称为泛洪）；明确的丢弃，即交换机知道不应该转发该帧，将其丢弃。交换机是一种即插即用的设备，其内部的帧交换表是通过自学习算法自动地逐渐建立起来的。

在这个实验中，你的任务是实现一个搭建在OpenNetLab上的交换机，该交换机具有自学习功能，可以逐渐学习MAC地址对应的端口以建立帧交换表。


Getting Started
---------------

1. :download:`下载实验资源 <./resources/switch.zip>` ，解压后进入文件夹，其中包含基础的实验代码模板。实验代码包含如下文件：

   - :file:`main.py` ：本地调试运行文件。
   - :file:`framegenerator.py` ：帧发送方文件，无需修改。
   - :file:`endpoint.py` ：帧接收方文件， `TODO` 部分待编写。
   - :file:`switch.py` : 自学习的MAC交换机文件，TODO部分待编写。
   - :file:`testcases.json` ：本地评测配置文件。
   - :file:`tester` ：本地评测运行文件，使用说明见相关文档。
   
2. 阅读实验任务，完成代码模板中 `TODO` 部分的代码片段。

3. 进行本地测试，完成本地测试后将代码提交到OpenNetLab在线平台进行远程评测。

4. 推荐实验环境：Linux操作系统，Python版本3.6以上。

Tasks
-----

在这个实验中，framegenerator从待发送的帧列表中获取endpoint的MAC地址，并向其发送帧，你需要按照如下规则实现endpoint.py 和switch.py中的put方法的设计：

- 在endpoint.py中，put方法表示接收来自switch的帧，并根据帧的frame_type进行相应处理。
  
  - 若frame_type为DATA,打印收到的数据帧Id
  - 若frame_type为BROADCAST
  
    - 检查目的mac地址是否与endpoint的mac地址相同，若相同则返回一个ACK帧给switch
    - 否则，不做处理

- 在switch.py中，put方法表示接收来自framegenerator的数据帧和endpoint确认帧，并根据帧的frame_type进行相应处理。
  
    - 首先，你需要根据实际需求,选择合适的数据结构先实现switch的forward_table（转发表）,要求至少包含mac地址和对应的端口号
    - 若frame_type为DATA

        - 检查forward_table中是否存在mac地址和对应的端口号
        - 若存在，则将帧转发到对应的端口
        - 否则，将广播帧转发到除源端口外的所有端口

    - 若frame_type为ACK
  
        - 将帧的mac地址和对应的端口号添加到forward_table中
  

该实验 `TODO` 部分的伪代码如下：

.. code-block:: text

    # endpoint
    function put(frame){
        # process the reception of frames
        receive frame from switch
        if frame_type is DATA{
            print frame_id
        }
        else if frame_type is BROADCAST{
            check the destination mac_addr
            if mac_addr is the same as endpoint's mac_addr{
                return an ACK frame to switch
            }
            else{
                pass
            }
        }


    # switch
    forward_table to be define #根据需求选择合适的数据结构实现转发表
    function put(frame){
        # process the reception of frames
        receive frame from framegenerator or endpoint
        if frame_type is DATA{
            check forward_table
            if mac_addr and it's port in forward_table{
                forward the frame to the port
            }
            else{
                forward BROADCAST frame to all the port except the source port
            }
        }
        else if frame_type is ACK{
            add the entry into forward_table
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

.. py:attribute:: self.outs
   :noindex:

   交换机转发帧的端口日志记录；

.. py:attribute:: self.outs
   :noindex:

   交换机的输出端口；

.. py:attribute:: self.frame_type
   :noindex:

   数据帧的类型，有DATA，BROADCAST，ACK三种；

.. py:function:: run(self, env) 
   :noindex:

   在给定的环境中运行客户端。

   @参数: env - 程序的运行环境

.. py:function:: dprint(self, msg)
   :noindex:

   打印运行日志信息。

   @参数: msg - 需要打印的信息

.. py:function:: forward(self, port_num, frame):
   :noindex:

   向给定端口转发帧，并记录转发的端口信息。

   @参数: port_num - 转发端口号
   @参数: frame - 数据帧

 


2.framegenerator、switch、endpoint之间均通过Wire完成帧的传输，其中framegenerator和switch之间有一条Wire，endpoint和switch之间各有两条Wire。

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




