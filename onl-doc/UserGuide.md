# User Guide

## 1 ONL介绍
在本节中，我们将对ONL平台的一些概念进行说明，并介绍如何部署ONL平台。
### 1.1 用户
ONL平台账号有三种类型：实验开发者Lab Developer，教师Teacher与学生Student。
#### Lab Developer
可以创建新的实验模板（Lab Template）供教师使用。
#### Teacher
教师可以创建实验（Lab），并通过修改实验模板的配置文件来为实验添加问题（Problem），查看学生对问题的提交（Submission）。
#### Student
学生可以参加平台上的实验并提交解答。



### 1.2 实验

+ **ONL实验[ONL Lab]**：基于ONL协议设计的网络实验。ONL实验主要包含发送端Sender，接收端Receiver与配置Configuration。
  + ONL实验的配置与实验内容有关。发送端和接收端通过读取配置，改变实验运行时的网络环境。

+ **实验模板[Lab Template]**：由开发者设计的ONL实验。实验模板包含一个完整的实验需要的所有文件。


+ **实验[Lab]**：由教师基于某个实验模板创建。一个实验包含实验属性和问题列表。
  + 实验属性：包含对实验的介绍、实验是否公开等。
  + 问题列表：一个实验可以有若干个问题。任意一个完成创建的实验必然包含一个对应实验模板的初始问题。
  
+ **问题[Problem]**：一个问题包含配置文件和学生代码段。
  + 配置文件：由发布实验的教师创建。这个配置文件的内容将被用来重写实验模板提供的配置文件中的`lab_config`字段。
  + 学生代码段：由参与实验的学生创建。学生需要根据问题要求，完成实验中的某段代码（函数）。这段代码将被用来重写实验代码中的相应位置。

+ **提交[Submission]**：学生对实验中的一个问题提交的解答。ONL平台的控制节点将该提交分配到执行节点，对该提交进行测试并得出结果，反馈给用户。

### 1.3 部署ONL平台
#### 控制器Controller
```shell
安装脚本
```
#### 执行节点Execution Node
```shell
安装脚本
```
#### 客户端Client
```shell
安装脚本
```

## 2 实验开发者使用说明Lab Developer Guide
在本节中，我们将对ONL协议进行介绍，并说明实验开发者如何使用ONL协议设计新的实验模板。

### 2.1 ONL协议与实验
对ONL协议进行说明，并介绍如何设计ONL实验

#### 2.1.1 ONL节点
##### User-defined Sender
##### User-defined Receiver
##### Student Task
##### Ultilities

#### 2.1.2 配置Configuration
如何设计实验模板的可配置项。
例子：
```json
{
  "seqno_width": 4,
  "loss_rate": 0.1,
  "max_delay": 30,
  "timeout": 60,
  "testcases": [
    "this_is_some_text",
    "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ"
  ]
}
```

### 2.2 创建实验模板
实验模板需要三个文件：
+ 含有ONL实验代码及其运行环境的docker镜像
+ 启动ONL实验的shell脚本
+ 实验模板的配置文件（资源需求及可配置项）

![Lab Template](img/Lab_Template.png)

将实验模板上传到Azure cloud DB。

**client bash** 

```
>>> add_problem (config file path) -r (docker file path) (docker exucute script path)
```

**example**

```bash
# first login as a administrator
>>> please assign your login info
>>> username: root
>>> password: ********
Login success
sessionid :  wvtjkk9cdr13zful0imd931g5sseje3h

# add lab template 
root >>> add_problem GBN.json -r image.tar cmd.json
# IF success: 
add_problem success
# IF failed:
add_problem failed
# show lab templates
root >>> problems
id      title   description     tags
1        Basic-01        TCP_GBN_LAB     tcp transmission        ['tcp', 'reliable transmission', 'transport layer']
```
**配置文件 [例]GBN.json**

```bash
{
  "_id": "Basic-01",
  "languages": ["python"],
  "tags": ["tcp", "reliable transmission", "transport layer"],
  "is_public": true,
  "title": "TCP_GBN_LAB",
  "description": "tcp transmission",
  "lab_config": {
                  "seqno_width": 4,
                  "loss_rate": 0,
                  "timeout": 0.5,
                  "testcases": [
                    ...
                  ]
                },
  "hint": "",
  "vm_num": 2,
  "port_num": [1,1],
  "code_num": 2,
  "template": "",
  "total_score": 100
}
```

**docker镜像及docker启动脚本 [例]cmd.json**

```bash
image: image.tar
cmd: ["bash ../receiver.sh","bash ../sender.sh"]
```


### 2.3 管理实验模板
实验模板将被上传至Azure，并同步到运行实验的资源节点。
![Lab Template Sync](img/Lab_Template_Sync.png)

+ 修改：上传修改过的实验模板到Azure即可。
+ 删除：删除存储的实验模板。

**client bash**

```
>>> problem del (lab template id)
```

**example**

```bash
# show lab templates
root >>> problems
id      title   description     tags
1        Basic-01        TCP_GBN_LAB     tcp transmission        ['tcp', 'reliable transmission', 'transport layer']
# delete lab template
root >>> problem del 1
# IF success
delete success
# IF failed
delete failed
# show lab templates
root >>> problems 
id      title   description     tags
```



## 3 教师使用说明Teacher Guide
在本节中，我们将介绍教师如何使用ONL平台和已有的实验模板来发布实验。

### 3.1 使用ONL平台
#### 3.1.1 发布实验

在ONL平台上提供了不同的实验模板，教师可以根据现有的实验模板发布实验。创建实验时需要填写实验信息：
1. 实验属性。如实验标题，Tag，是否公开等。
2. 助教。为实验添加助教名单，成为助教的学生能在该实验下获得助教权限，可以查看该实验下的提交记录。

**client bash**

```bash
>>> contest create (lab_list config path)
```

**example**

```bash
# show lab_list
root >>> contests
id      title   description
# create lab_list
root >>> contest create Contest.json
contest create success
# show lab_list
root >>> contests
id      title   description
1 B_01   Test contest
```

**[例]Contest.json**

```bash
{
  "title": "B_01",
  "description": "Test contest",
  "password": "",
  "visible": true,
  "contest_admin": [],
  "allowed_ip_ranges": null
}
```

#### 3.1.2 管理实验
教师能够对实验进行编辑，比如修改实验属性，删除实验，管理问题等。

**client bash**

```bash
>>> contest modify (config path)
```

#### 3.1.3 管理问题
实验有一个包含若干个问题的问题列表。问题的添加、修改、删除均可以在列表中进行。
+ 实验初始状态只有对应实验模板的配置文件的一个问题。
+ 一个实验有若干个可配置项，教师可以根据[3.2 实验模板介绍](#32-实验模板介绍)中对应的实验模板的指导编写实验配置文件。通过“添加问题”上传该配置文件和填写问题属性（如问题的描述，解题要求等），即可在实验下添加问题。
+ 问题的修改与添加类似，上传新的配置文件或修改问题描述等属性。问题可在问题列表中删除。

**client bash**

```bash
# show contest problem list 
>>> contest show_problem (contest_id)
# add contest problem
>>> contest add_problem (contest_id) (template_id) (config_path)
```

**example**

```bash
# show problems in contest 1
root >>> contest show_problem 1
id      title   description     tags
# show problem template
root >>> problems            
id      title   description     tags
2        Basic-01        TCP_GBN_LAB     tcp transmission        ['tcp', 'reliable transmission', 'transport layer']
# add problem by overrided template 2
root >>> contest add_problem 1 2 contest_GBN.json
# problem add success
contest add_problem success
# show problems in contest 1
root >>> contest show_problem 1
id      title   description     tags
3        A-01    TCP_GBN_LAB     tcp transmission        ['tcp', 'reliable transmission', 'transport layer']
```

#### 3.1.4 查看提交
教师用户可以查看实验下的所有提交记录并导出。

**client bash**

```bash
>>> contest submissions (contest_id)
```

### 3.2 实验模板介绍
#### 3.2.1 GBN sender
##### Overview
* How does the experimental code work

##### Configuration
The test cases and lab specific configuration fields is stored in `lab_config.json` file. The configuration options are as follows (*default*):
```json
{
  "seqno_width": 4,
  "loss_rate": 0.1,
  "max_delay": 30,
  "timeout": 60,
  "testcases": [
    "this_is_some_text",
    "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ"
  ]
}
```
#### 3.2.2 ...

## 4 学生使用说明Student Guide

### 4.1 使用ONL平台
#### 4.1.1 参与实验
所有已被发布的实验都会被展示在实验列表中。
```shell
# 实验列表条目为如下形式：
|实验名称|属性|公开（Y/N）|
```
如果一个实验的实验属性中`password=''`（或`is_public=False`，那么这个实验将会被开放给所有用户；否则，该实验在实验列表中的相关条目会被设置为加锁（非公开）。学生可以参与实验列表中的某个实验，查看实验下的问题列表。
![实验列表与实验界面（等待补充）]()

**client bash**

```bash
# show contest problem list 
>>> contest show_problem (contest_id)
```

**example**

```bash
# log in as an ordinary user
>>> username: oduser
>>> password: **************
Login success
# show problem list in contest 1
oduser >>> contest show_problem 1
id      title   description     tags
2        A-01    TCP_GBN_LAB     tcp transmission        ['tcp', 'reliable transmission', 'transport layer']
```

#### 4.1.2 提交解答
学生可以查看实验下的问题列表。
![问题列表与问题详情（等待补充）]()
选择问题列表中的一项进入问题详情界面。学生需要提交对问题的解答（代码段），称为一次**提交Submission**，由平台生成任务进行评判。

**client bash**

```bash
# submit template problem
>>> problem submit (problem id) (src_path1) [src_path2]...
# submit contest problem
>>> contest submit_problem (contest id) (problem id) (src_path1) [src_path2]...
```

**example**

```bash
# show contest problem 1
oduser >>> contest show_problem 1
id      title   description     tags
2        A-01    TCP_GBN_LAB     tcp transmission        ['tcp', 'reliable transmission', 'transport layer']
# submit contest problem , src: code/gbn_receiver.py code/gbn_sender.py
oduser >>> contest submit_problem 1 2 code/gbn_receiver.py code/gbn_sender.py
code/gbn_receiver.py   code/gbn_sender.py
# submit success
submit success

# submit template problem
oduser >>> problem submit 1 code/gbn_receiver.py code/gbn_sender.py
code/gbn_receiver.py   code/gbn_sender.py
# submit success
submit success
```

#### 4.1.3 查看提交记录
学生可以查看自己的提交记录。提交记录会展示每一次提交的所属实验、提交状态、评判结果等信息。
```python
    # 提交状态分别为：
    COMPILE_ERROR = -2  # 编译错误
    WRONG_ANSWER = -1   # 解答错误
    ACCEPTED = 0        # 解答通过
    RUNTIME_ERROR = 1   # 运行时错误
    SYSTEM_ERROR = 2    # 系统错误
    NETWORK_TIMEOUT = 3 # 网络超时
    PENDING = 4         # 等待评判
    JUDGING = 5         # 正在评判
    PARTIALLY_ACCEPTED = 6  #部分通过

```
![提交记录（等待补充）]()

**client bash**

```bash
# get template problem submissions
>>> submissions
# get submissions of a contest
>>> contest submissions (contest_id)
```

**example**

```bash
# get template problem submissions
oduser >>> submissions
problem 		create time     			   result  		   JudgeServer                             port list
Basic-01         2022-05-26T06:39:59.269592Z     ACCEPTED        ['192.168.40.134', '192.168.40.133']    [[6001], [6001]]
# get submmissions of contest 1
oduser >>> contest submissions 1
problem  create time                     result           JudgeServer                             port list
A-01     2022-05-26T06:37:02.034758Z     ACCEPTED         ['192.168.40.134', '192.168.40.133']    [[6000], [6000]]
```



### 4.2 实验说明
在本节中，我们将介绍实验模板考察的计算机网络知识，并对实验考察方式与学生任务进行说明。
#### 4.2.1 GBN sender
##### Overview

* what is GBN
    * GBN concepts
    * recommended resources for students to learn and understand GBN
* what is your task in this lab

##### Get Started

Given skeleton code template, explain to the students:

* the usage of other usable functions in ONL
* what he should implement in each function

```python
class StudentGBNSender(GBNSender):
    async def setup(self):
        await super().setup()
        # TODO: setup your data

    async def student_task(self, message):
        # TODO: main routine

    async def teardown(self):
        await super().teardown()
        # TODO: post handling

    # TODO: add other fucntions yourself
```

##### Development and debugging advice

how to run the code on ONL platform and how to check the logging to find the problems in your code

##### Submission

how to submit the code to ONL platform

#### 4.2.2 ...




