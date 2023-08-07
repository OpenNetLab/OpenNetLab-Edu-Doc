:标题: OpenNetLab for Education学生使用手册

:作者:
 - yangfan

:时间: 2023年8月6日

=========================
OpenNetLab Student Manual
=========================


1. Register an account
----------------------
在实验平台注册界面进行注册，输入用户名（如学号+姓名）、电子邮箱以及密码。

.. figure:: ./images/register1.1.png
    :name: register1.1
    :align: center
    :width: 700px
    :alt: image cannot be loaded

    步骤一：查找Login位置

.. figure:: ./images/register1.2.png
    :name: register1.2
    :align: center
    :width: 300px
    :alt: image cannot be loaded

    步骤二：注册账号

2. Login account
-----------------
在登陆界面使用用户名和密码登录实验教学平台。

.. figure:: ./images/login2.1.png
    :name: login2.1
    :align: center
    :width: 700px
    :alt: image cannot be loaded

    步骤三：查找Register位置

.. figure:: ./images/login2.2.png
    :name: login2.2
    :align: center
    :width: 300px
    :alt: image cannot be loaded

    步骤四：登录账号

3. Lab list
------------
登录成功后，在Labs中查看平台已发布的实验。

.. figure:: ./images/lablist3.1.png
    :name: lab_list3.1
    :align: center
    :width: 700px
    :alt: image cannot be loaded

    步骤五：查看所有发布在平台的实验信息


4. Choose Courses and Submit experiment code
---------------------------------------------

- 在Courses中查看所有的课程信息，并选择所需加入的课程。

.. figure:: ./images/courseslist4.1.png
    :name: courses_list4.1
    :align: center
    :width: 700px
    :alt: image cannot be loaded

    步骤六：查看所有课程信息

- 选择所需要加入的课程，查看课程的详细信息(课程持续时间、实验信息等)。

.. figure:: ./images/courseslist4.2.png
    :name: courses_list4.2
    :align: center
    :width: 700px
    :alt: image cannot be loaded

    步骤七：选择课程并了解课程信息

- 选择完课程之后，在页面右侧点击Labs选项，可以查看该课程下的实验情况。

.. figure:: ./images/courseslist4.3.png
    :name: courses_list4.3
    :align: center
    :width: 700px
    :alt: image cannot be loaded

    步骤八：查看课程下的实验情况

- 选择所需要完成的实验，进入代码提交页面，根据要求提交完整代码，并点击submit进行提交评测。

.. figure:: ./images/courseslist4.4.png
    :name: courses_list4.4
    :align: center
    :width: 700px
    :alt: image cannot be loaded

    步骤九：提交代码

- 提交代码后，页面跳转到代码评测界面，显示代码提交时间、代码状态、得分、用户名信息等.

.. figure:: ./images/courseslist4.5.png
    :name: courses_list4.5
    :align: center
    :width: 700px
    :alt: image cannot be loaded
    
    步骤十：代码评测状态

- 若代码状态显示不是ALL PASSED，可以通过ID，查看错误信息。

.. figure:: ./images/debug5.1.png
    :name: debug5.1
    :align: center
    :height: 80px
    :width: 700px
    :alt: image cannot be loaded
    
    步骤十一：查找错误信息

- 页面跳转到错误信息页面，根据错误的测试案例，查看日志信息，帮助修改代码

.. figure:: ./images/debug5.2.png
    :name: debug5.2
    :align: center
    :width: 700px
    :alt: image cannot be loaded
    
.. figure:: ./images/debug5.3.png
    :name: debug5.3
    :align: center
    :width: 700px
    :alt: image cannot be loaded
    
    步骤十二：查看错误案例日志

Tips
-----

其中代码状态分为：

.. py:attribute:: ALL PASSED
    :noindex:

    测试全部通过，代码完全正确！！！；

.. py:attribute:: SOME PASSED
    :noindex:

    测试部分通过；

.. py:attribute:: ALL FAILED
    :noindex:

    测试全部失败，仔细阅读实验要求，重新设计代码；

.. py:attribute:: TIMEOUT
    :noindex:

    测试超时，请重新设计代码逻辑；

