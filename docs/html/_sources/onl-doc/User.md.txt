# User Manual
## 1 OpenNetLab

### 1.1 What is OpenNetLab
OpenNetLab(ONL) is an online network experiment platform. ONL is dedicated to provide a near-real network experiment environment for teaching computer network courses. In the ONL project, device resources contributed by different institutions are managed in a unified manner, and experiments will be assigned to run on free devices according to resource requirements.

To develop a uniform style of experimentation, the ONL project provides the ONL protocol to build experiment templates. The ONL protocol uses a client-server architecture to provide a common approach to the design of network experiments.

A description of the terminology specific to the ONL platform is given here：
+ `ONL Lab`: network experiments designed based on the ONL protocol.

+ `Lab Template`: ONL lab implementations based on Docker image. The lab template is the complete flow of the experiment.

+ `Lab`: experiments that users can participate in. The content of the experiment is a number of questions that require the user to submit answers.

+ `Problem`: contents that the experiment will examine. The user submits the answer to this question and waits for the review.

+ `Submission`: experimental task to evaluate the answers submitted by users. A submission is a single answer from the user to a problem in the lab.

+ `Achievement`: evaluation results obtained in one submission

### 1.2 What you can do
The ONL platform provides a variety of lab templates, allowing users with special permissions to add lab templates or publish web labs based on them. 

ONL users can participate in labs and submit solutions, and the platform allocates resources to experimental tasks and judges solutions.

Based on the actions (permissions) that users can perform on the ONL platform, users can be divided into three categories：
`Developer`: provide lab templates
`Teacher`: publish web labs
`Student`: participate in labs and submit answers

In addition to the above 3 capacities, there is another capacity related to experiment: Teaching assistant (called `Tutor`). A tutors is a student who has been granted special privileges by the teacher for the experiment only.

The following table shows the description of permissions for different users:
*权限的表*


## 2 ONL Instruction

In this section, you will learn how to use the ONL platform.

### 2.1 Experimental Process

The core function of the ONL platform is undoubtedly "experimentation". On the ONL platform, you can see a "lab list" and participate in the lab you are interested in.
+ You will not always be able to participate in the experiments you are interested in, and some experiments will have restrictions on participant status.

Each entry in the "lab list" represents a lab published by an advanced user (teacher or developer). In these labs, depending on the specific content of the lab, there are several problems corresponding to different configurations of the lab.

Participating in a lab means that you need to select a problem  (assuming the problem belongs to a lab you have access to) and submit a snippet of code based on the problem description. This code snippet is usually an implementation of some function in the lab, related to the experiment goal.

After the code snippet is successfully submitted, the controller of the platform will generate the experimental task and add it to the waiting queue. When computing resources are obtained, the task is distributed to the resource nodes for the network experiment. At the end of the experimental task (i.e., a submission), the achievement is reported back to the user. You can view it in the submission record.


### 2.2 Participate in a lab

The entries in the experiment list have the following format:
```bash
| lab title | lab property | lab status |

```
In an entry, lab property includes lab description, publisher, tag, and so on, while `lab status` includes whether the lab is open and ongoing.

Once you enter the experiment (click in the UI or use the 'contest' in the command line), you see a list of problems that belong to this experiment.

**client bash**

```bash
# show contest problem list 
>>> contest show_problem (contest_id)
```

**example (take the TCP GBN Sender experiment as an example)**

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

### 2.3 Submit an answer

Select a problem and you can see the details. Analyze the description of the problem and submit the code that meets the requirements.

In order to help you better understand the experiment content and complete the code, more instructions are given in the lab manual section of OpenNetLab for education website.

**client bash**

```bash
# submit template problem
>>> problem submit (problem id) (src_path1) [src_path2]...
# submit contest problem
>>> contest submit_problem (contest id) (problem id) (src_path1) [src_path2]...
```

**example (take the TCP GBN Sender experiment as an example)**

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

### 2.4 View Submissions
#### User Submissions Records
Users can view the history of submissions. The submission history shows the status of the experiment task corresponding to each submission.

```python
# the status of the experiment task:
    COMPILE_ERROR = -2  
    WRONG_ANSWER = -1   
    ACCEPTED = 0        
    RUNTIME_ERROR = 1   
    SYSTEM_ERROR = 2    
    NETWORK_TIMEOUT = 3 
    PENDING = 4         
    JUDGING = 5         
    PARTIALLY_ACCEPTED = 6  
# a submission:
problem 		create time     			   result  		   JudgeServer                             port list
Basic-01         2022-05-26T06:39:59.269592Z     ACCEPTED        ['192.168.40.134', '192.168.40.133']    [[6001], [6001]]
```
**client bash**

```bash
# get template problem submissions
>>> submissions
# get submissions of a contest
>>> contest submissions (contest_id)
```

**example (take the TCP GBN Sender experiment as an example)**

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
#### Lab Submissions Records

Users with tutor status can view submissions for that lab. The submission record of the lab will show the submission of all users participating in the lab (including the corresponding problem, evaluation status and result of the submission).
