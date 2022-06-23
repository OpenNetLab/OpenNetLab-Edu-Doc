# System Design

## 1 Overview
![](./img/System.png)
`Controller`: controller node of the ONL platform, which manages all functions of the ONL platform
`Execution Node`: the executor of the ONL experiment, who judges the submission of the experiment
`Client`: interact with users

## 2 Data Objects
Use the database to save various objects, and operate the database through Django's ORM framework.
### 2.1 User
`User` inherits from Django's `AbstractBaseUser` class as a user of the ONL platform.

```python
# account/model.py
class AdminType(object):
    REGULAR_USER = "Regular User" #student
    ADMIN = "Admin" # tutor
    SUPER_ADMIN = "Super Admin" #teacher
    
class User(AbstractBaseUser):
    id = models.UUIDField(primary_key=True, default=uuid.uuid4, editable=False)
    username = models.TextField(unique=True)
    email = models.TextField(null=True, unique=True)
    create_time = models.DateTimeField(auto_now_add=True, null=True)
    #UserType
    admin_type = models.TextField(default=AdminType.REGULAR_USER)
    problem_permission = models.TextField(default=ProblemPermission.NONE)
    reset_password_token = models.TextField(null=True)
    reset_password_token_expire_time = models.DateTimeField(null=True)
    #SSO auth token
    auth_token = models.TextField(null=True)
    two_factor_auth = models.BooleanField(default=False)
    tfa_token = models.TextField(null=True)
    session_keys = JSONField(default=list)
    # open api key
    open_ap0i = models.BooleanField(default=False)
    open_api_appkey = models.TextField(null=True)
    is_disabled = models.BooleanField(default=False)

    USERNAME_FIELD = "username"
    REQUIRED_FIELDS = []

    def is_admin(self):
        ...

    def is_super_admin(self):
        ...

    def is_admin_role(self):
        ...

    def can_mgmt_all_problem(self):
        ...

    def is_contest_admin(self, contest):
        ...
    class Meta:
        db_table = "user"

class UserProfile(models.Model):
    user = models.OneToOneField(User, on_delete=models.CASCADE)

    problems_status = JSONField(default=dict)
    real_name = models.TextField(null=True)
    blog = models.URLField(null=True)
    github = models.TextField(null=True)
    school = models.TextField(null=True)
    major = models.TextField(null=True)
    language = models.TextField(null=True)
    #for Contest
    accepted_number = models.IntegerField(default=0)
    total_score = models.IntegerField(default=0)

    def add_accepted_problem_number(self):
        ...

    def add_score(self, this_time_score, last_time_score=None):
        ...
    class Meta:
        db_table = "user_profile"
```
The `User` class contains all parameters related to the ONL platform (including experiments), and functions to check user information and permissions.

The `UserProfile` class is mainly personal information that can be customized.

### 2.2 实验

The `Contest` class represents ONL labs.

```python
# /contest/model.py
class Contest(models.Model):
    title = models.TextField()
    description = RichTextField()
    # show real time rank or cached rank
    password = models.TextField(null=True)
    # enum of ContestRuleType
    start_time = models.DateTimeField(timezone.now())
    end_time = models.DateTimeField(null=True)
    create_time = models.DateTimeField(auto_now_add=True)
    last_update_time = models.DateTimeField(auto_now=True)
    created_by = models.ForeignKey(User, on_delete=models.CASCADE)
    # is visible. If false, it is equivalent to delete
    visible = models.BooleanField(default=True)
    contest_admin = models.JSONField(default=list)
    allowed_ip_ranges = JSONField(default=list)

    @property
    def status(self):
        ...

    @property
    def contest_type(self):
        ...

    def problem_details_permission(self, user):
        ...

    class Meta:
        db_table = "contest"
        ordering = ("-start_time",)
```

### 2.3 Problem
`Problem` is the content of the experiment. There may be multiple Problems in one lab.
```python
class Problem(models.Model):
    # display ID
    _id = models.TextField(db_index=True)
    contest = models.ForeignKey(Contest, null=True, on_delete=models.CASCADE)
    # for contest problem
    lab_id = models.IntegerField(null=True)
    lab_config = models.JSONField(default=dict)
    is_public = models.BooleanField(default=False)
    title = models.TextField()
    # HTML
    description = RichTextField()
    hint = RichTextField(null=True)
    languages = JSONField()
    # number of nodes required
    vm_num = models.IntegerField()
    # number of port
    port_num = models.JSONField(default=list)
    # Number of code snippets students need to write
    code_num = models.IntegerField()
    template = JSONField(null=True)
    create_time = models.DateTimeField(auto_now_add=True)
    # we can not use auto_now here
    last_update_time = models.DateTimeField(null=True)
    created_by = models.ForeignKey(User, on_delete=models.CASCADE)

    # judge related
    visible = models.BooleanField(default=True)
    tags = models.ManyToManyField(ProblemTag)
    total_score = models.IntegerField(default=0)
    submission_number = models.BigIntegerField(default=0)
    accepted_number = models.BigIntegerField(default=0)
    # {JudgeStatus.ACCEPTED: 3, JudgeStaus.WRONG_ANSWER: 11}, the number means count
    statistic_info = JSONField(default=dict)
    share_submission = models.BooleanField(default=False)

    class Meta:
        db_table = "problem"
        unique_together = (("_id", "contest"),)
        ordering = ("create_time",)

    def add_submission_number(self):
        ...

    def add_ac_number(self):
        ...
```
### 2.4 Submission
`Submission` is a student's submission, and then an execution node is assigned to judge the submitted code.
```python
class Submission(models.Model):
    id = models.TextField(default=rand_str, primary_key=True, db_index=True)
    contest = models.ForeignKey(Contest, null=True, on_delete=models.CASCADE)
    problem = models.ForeignKey(Problem, on_delete=models.CASCADE)
    create_time = models.DateTimeField(auto_now_add=True)
    user_id = models.CharField(db_index=True, max_length=50)
    username = models.TextField()
    code_list = models.JSONField(default=list)
    server_list = models.JSONField(default=list)
    ports_list = models.JSONField(default=dict)
    result = models.IntegerField(db_index=True, default=JudgeStatus.PENDING)
    # Judgment details returned from JudgeServer
    info = JSONField(default=dict)
    language = models.TextField()
    shared = models.BooleanField(default=False)
    # Store the time and memory values ​​used by the commit for easy display of the commit list
    # {time_cost: "", memory_cost: "", err_info: "", score: 0}
    ip = models.TextField(null=True)

    def check_user_permission(self, user, check_share=True):
        ...

    def modify_permission(self, user, check_share=True):
        ...

    class Meta:
        db_table = "submission"
        ordering = ("-create_time",)

```
### 2.5 Others
`Announcement`: Used to publish an announcement.
```python
class Announcement(models.Model):
    title = models.TextField()
    # HTML
    content = RichTextField()
    create_time = models.DateTimeField(auto_now_add=True)
    created_by = models.ForeignKey(User, on_delete=models.CASCADE)
    last_update_time = models.DateTimeField(auto_now=True)
    visible = models.BooleanField(default=True)

    class Meta:
        db_table = "announcement"
        ordering = ("-create_time",)
```
`JudgeServer`: Indicates the status of the execution node.
```python
class JudgeServer(models.Model):
    hostname = models.JSONField(null=True)
    ip = models.TextField(null=True)
    available_ports = models.JSONField(default=list)
    using_ports = models.JSONField(default=list)
    available_ports_num = models.IntegerField(default=0)
    location = models.TextField(null=True)
    is_public = models.BooleanField(default=True)
    cpu_core = models.IntegerField(null=True)
    memory_usage = models.FloatField(null=True)
    cpu_usage = models.FloatField(null=True)
    last_heartbeat = models.DateTimeField(timezone.now())
    create_time = models.DateTimeField(auto_now_add=True)
    expired_time = models.DateTimeField(null=True)
    task_number = models.IntegerField(default=0)
    service_url = models.TextField(null=True)
    ca_pem = models.TextField(null=True)
    c_cert = models.TextField(null=True)
    c_key = models.TextField(null=True)
    is_ready = models.BooleanField(default=False)
    is_disabled = models.BooleanField(default=False)

    @property
    def status(self):
        ...

    class Meta:
        db_table = "judge_server"
```

## 3 Node introduction

There are two types of nodes in the system: the controller `Controller` and the execution node `Execution Node`
### 3.1 Controller
#### account

Account module
```python
# account.views.admin
class UserAdminAPI(APIView):
    ...
class GenerateUserAPI(APIView):
    ...
```
`UserAdminAPI`: User management (POST/GET/PUT/DELETE).
`GenerateUserAPI`: Get the user table and generate users in batches.
```python
# account.view.user
class UserProfileAPI(APIView):
    ...
class AvatarUploadAPI(APIView):
    ...
class TwoFactorAuthAPI(APIView):
    ...
class CheckTFARequiredAPI(APIView):
    ...
class UserLoginAPI(APIView):
    ...
class UserLogoutAPI(APIView):
    ...
class UsernameOrEmailCheck(APIView):
    ...
class UserRegisterAPI(APIView):
    ...
class UserChangeEmailAPI(APIView):
    ...
class UserChangePasswordAPI(APIView):
    ...
class ApplyResetPasswordAPI(APIView):
    ...
class ResetPasswordAPI(APIView):
    ...
class SessionManagementAPI(APIView):
    ...
class ProfileProblemDisplayIDRefreshAPI(APIView):
    ...
class OpenAPIAppkeyAPI(APIView):
    ...
class SSOAPI(CSRFExemptAPIView):
    ...
```
`UserProfileAPI`, `UserChangeEmailAPI`, `UserChangePasswordAPI`, `ApplyResetPasswordAPI` and `ResetPasswordAPI`: Get and modify user information.
`UserRegisterAPI`, `UserLoginAPI` and `UserLogoutAPI`: register, log in and log out.
Others: Account security detection.

```python
# decorators
class BasePermissionDecorator(object):
    ...
```
`BasePermissionDecorator`: The base class of decorators, there are multiple derived classes for status checking, such as whether you are logged in, whether you have administrative permissions, etc.

#### announcement
Announcement module for website announcements and experiment notifications.
```python
# announcement.views.admin
class AnnouncementAdminAPI(APIView):
    ...

```
`AnnouncementAdminAPI`: Announcement management (POST/PUT/GET/DELETE).

```python
# announcement.views.user
class AnnouncementAPI(APIView):
```
`AnnouncementAPI`: get the announcement

#### config
The configuration module manages the global configuration information of the platform.

```python
# conf.views
class JudgeServerAPI(APIView):
    ...
class JudgeServerHeartbeatAPI(CSRFExemptAPIView):
    ...

```
`JudgeServerAPI`: Manage the state of resource nodes (GET/PUT).
`JudgeServerHeartbeatAPI`: Heartbeat detection, automatic resource registration when a new resource node is detected.
#### contest
```python
# contest.views.admin
class ContestAPI(APIView):
    ...

```
`ContestAPI`: Experimental module, methods for operating experiments (POST/PUT/GET)
```python
# contest.views.user
class ContestAPI(APIView):
    ...
```
`ContestAPI`: Get the test object (GET).

#### problem

```python
# problem.views.admin
class ProblemAPI(ProblemBase):
    ...
class ContestProblemAPI(ProblemBase):
    ...
class MakeContestProblemPublicAPIView(APIView):
    ...
class AddContestProblemAPI(APIView):
    ...
class ExportProblemAPI(APIView):
    ...
```
`ContestProblemAPI`: The problem whose configuration has been modified by the teacher.
Others: How to operate the Problem.
```python
# problem.views.user
class ProblemTagAPI(APIView):
    ...
class PickOneAPI(APIView):
    ...
class ProblemAPI(APIView):
    ...
class ContestProblemAPI(APIView):
    ...
```
`ContestProblemAPI`: Get the details of a single Problem, or filter the status of the Problem obtained by condition.
#### submission
```python
# submission.views.admin
class SubmissionAPI(APIView):
    ...
class SubmissionRejudgeAPI(APIView):
    ...
```
Manage submissions, such as obtaining status, re-judging, modifying scores, etc.
```python
# submission.views.user
class SubmissionAPI(APIView):
    ...
class SubmissionListAPI(APIView):
    ...
class ContestSubmissionListAPI(APIView):
    ...
class SubmissionUpdateAPI(CSRFExemptAPIView):
    ...
```
Get the status change of the submission on the resource node.

#### judge
```python
# judge.dispatcher
class ChooseJudgeServer:
    ...
class JudgeDispatcher(DispatcherBase):
    ...
def process_pending_task():
```
`ChooseJudgeServer`: Allocate resource nodes as execution nodes for experiments.
`JudgeDispatcher`: The task is dispatched.
`process_pending_task`: Use queues to manage tasks that need to be delivered.
#### deploy
Required files and `entrypoint.sh` for the deployment platform

### 3.2 Execution Node
#### service
```python
class HeartBeatService(BasicService):
    ...
class ResultPushService(BasicService):
    ...
class RsyncService(object):
    ...
```
`HeartBeatService`: Heartbeat detection, interacts with the controller regularly, and updates the server status.
`ResultPushService`: Pushes the status of the submission on this execution node to the controller.
`RsyncService`: Sync experiment templates to this node.

#### server
```python
class JudgeServer:
    ...
def server(path):
    ...
```
`JudgeServer`: Implements methods for executing various functions of the node (ping/judge/fetch/sync).
`server`: According to the type of Controller access request, call the method of `JudgeServer` to reply
## 4 Node communication
The ONL platform nodes mainly use the Python requests library to communicate based on HTTP.
![](img/Communication.drawio.png)
### Controller and Execution Node
+ Heartbeat detection to detect the availability of execution nodes.
```python
class HeartBeatService(BasicService):
    def heartbeat(self):
        data = info.server_info()
        data.update(info.net_info())
        if tlsStatus.TLS in [tlsStatus.Status.INITIAL, tlsStatus.Status.RENEW]:
            data.update(info.tls_info())
            tlsStatus.TLS = tlsStatus.Status.NORMAL
        data["action"] = "heartBeat"
        print(self.backend_url)
        return self._request(data=data, route="/judge_server_heartbeat", action="HeatBeat")
```
+ Update the evaluation status of submissions, and push the status of submissions running on the execution node to the Controller.
```python
class ResultPushService(BasicService):
    def result_push(self, submission_id):
        data: dict = info.submission_info(submission_id)
        self._request(data=data, route="/submission/update", action="Submission update")
```

### Execution Nodes
The data transmission of the experimental run is based on the ONL protocol.
### User interaction with the platform
Use ONL Client to connect with the control node.
`Login`: login and registration module
`main.py`: Operates from the command line.


## 5 Deploy ONL platform
The controller node, the execution node and the client are deployed using the installation script.
`Controller`:
`Execution node`:
`Client`:
## 6 API description

#### account/admin POST api/admin/user/?
``request``
```json

// POST /user
{
    "header":{

    },
    //request.data
    "json":{
        //data
        "user":[
            //user_data
            [   
                username,   //string,
                password,   //string
                email,      //string
                real_name,
            ],
            ...

        ],

    }
}
```
``response``
```json
//APIView.success:
{
    "error": None, 
    "data": None,
}

//APIView.error:
{
    "error": "error",      
    "data": "Error occurred while processing data '{user_data}'" / IntegrityError,//user_data in "request"
}
```
#### account/admin PUT api/admin/user/?
``request``
```json
{
    "header":{

    },
    //request.data
    "json":{
        "id":id,                                    //UUIDField
        "username":username,                        //TextField
        "password":password,
        "email":email,                              //TextField
        "admin_type":admin_type,                    //AdminType { REGULAR_USER = "Regular User", ADMIN = "Admin", SUPER_ADMIN = "Super Admin" }
        "id_disabled":is_disabled,                  //BooleanField
        "problem_permission":problem_permission,    //TextField, ProblemPermission{ NONE = "None",OWN = "Own", ALL = "All"}
        "open_api": ,                               //BooleanField
        "two_factor_auth": ,                        //BooleanField
        "real_name": ,
    }
}
```
``response``
```json
//APIView.error
{
    "msg":"User does not exist" / "Username already exists" / "Email already exists",
    "error":"error",
}
//APIView.success:
{
    "error": None, 
    "data": data   //data: UserAdminSerializer(user).data       model = User; fields = ["id", "username", "email", "admin_type", "problem_permission", "real_name", "create_time", "last_login", "two_factor_auth", "open_api", "is_disabled"]
}

```
#### account/admin GET api/admin/user/?
``request``
```json
{
    "id": id,
    "keyword":keyword,          //optional
}
```

``response``
```json
//APIView.error
{
    "msg":"User does not exist" ,
    "error":"error",
}
//APIView.success:
{
    "error": None, 
    "data": data   //data: UserAdminSerializer(user).data       
    //model = User; fields = ["id", "username", "email", "admin_type", "problem_permission", "real_name", "create_time", "last_login", "two_factor_auth", "open_api", "is_disabled"]
    // if "keyword" : data = paginate_data
}
```
#### account/admin DELETE api/admin/user/?
``request``
```json
{
    "id": id,

}
```
``response``
```json
//APIView.error
{
    "msg":"Invalid Parameter, id is required" / "Current user can not be deleted" ,
    "error":"error",
}
//APIView.success:
{
    "error": None, 
    "data": None,
}
```


#### account/admin GET api/admin/generate_user/?
下载user excel  
``request``
```json
parameter: "file_id"
```
``response``
```json
//APIView.error
{
    "msg":"Invalid Parameter, file_id is required" / "Illegal file_id" / "File does not exist",
    "error":"error",
}
//HttpResponse
{
    "content": data,   //xlsx file
    "Content-Disposition":"attachment; filename=users.xlsx",
    "Content-Type":"application/xlsx"
}
```

#### account/admin POST api/admin/generate_user/?
批量添加用户  
``request``
```json
{
    "header":{

    },
    "json":{
        "number_from": number_from,
        "number_to": number_to,
        "prefix": prefix,
        "suffix": suffix,
        "password_length": password_length,
    }
}
```
``response``
```json
//APIView.error
{
    "msg":"Username should not more than 32 characters" / "Start number must be lower than end number" / IntegrityError,
    "error":"error",
}
//APIView.success
{
    "msg":{"file_id": file_id,},
    "error":"error",
}
```


#### account/user POST api/login/?
用户登录  
``request``
```json
{
    "header":{

    },
    "json":{
        "username":username,
        "password":password,
        "tfa_code":tfa_code,
    }
}
```
``response``
```json
//APIView.success
{
    "msg":"Succeeded",
    "error":None,
}
//APIView,error
{
    "msg":"Your account has been disabled" / "tfa_required" / "Invalid two factor verification code" / "Invalid username or password",
    "error":"error",
}
```

#### account/user GET api/logout/?
登出  
``response``
```json
//APIView
{
    "msg":None,
    "error":None,
}
```
#### account/user POST api/register/?
注册  
``request``
```json
{
    "header":{

    },
    "json":{
        "username":username,
        "email":email,
        "password":password,

    }
}
```
``response``
```json
//APIView.error
{
    "msg":"Register function has been disabled by admin" / "Username already exists" / "Email already exists" / ,
    "error":"error",
}
//APIView.success
{
    "msg":"Succeeded" ,
    "error":None,
}
```
#### account/user POST api/change_password/?
改密码  
``request``
```json
{
    "header":{

    },
    "json":{
        "old_password":old_password,
        "tfa_code":tfa_code,        //"tfa_required" error
        "new_password":new_password,

    }
}
```
``response``
```json
//APIView.error
{
    "msg":"Invalid two factor verification code" / "tfa_required" / "Invalid old password",
    "error":"error",
}
//APIView.success
{
    "msg":"Succeed",
    "error":"error",
}
```
#### account/user POST api/change_email/?
``request``
```json
{
    "header":{

    },
    "json":{
        "password":password,
        "tfa_code":tfa_code,
        "new_email":new_email,

    }
}
```
``response``
```json
//APIView.error
{
    "msg":"Wrong password" / "tfa_required"/ "Invalid two factor verification code" / "The email is owned by other account",
    "error":"error",
}
//APIView.success
{
    "msg":"Succeeded" ,
    "error":None,
}
```
#### account/user POST api/apply_reset_password/?
根据邮箱重置密码（验证邮箱？）  
``request``
```json
{
    "header":{

    },
    "json":{
        "email":email,

    }
}
```
``response``
```json
//APIView.error
{
    "msg":"You have already logged in, are you kidding me? " / "User does not exist" / "You can only reset password once per 20 minutes",
    "error":"error",
}
//APIView.success
{
    "msg":"Succeeded" ,
    "error":None,
}
```
#### account/user POST api/reset_password/?
配合apply_reset_password（重置密码）  
``request``
```json
{
    "header":{

    },
    "json":{
        "token":token,
        "password":password,
    }
}
```
``response``
```json
//APIView.error
{
    "msg":"Token does not exist" / "Token has expired",
    "error":"error",
}
//APIView.success
{
    "msg":"Succeeded" ,
    "error":None,
}
```
#### account/user POST api/check_username_or_email
check username or email is duplicate  
``request``
```json
{
    "header":{

    },
    "json":{
        "username": username,
        "email": email,
    }
}
```
``response``
```json
//APIView.success
{
    "msg":{
        "username": Boolean,
        "email": Boolean,
    } ,
    "error":None,
}
```

#### account/user GET api/profile/?
判断是否登录，成功就返回用户信息    
``request``
```json
parameter:username
```
``response``
```json
//APIView.error
{
    "msg":"User does not exist",
    "error":"error",
}
//APIView.success
{
    "msg":None / request.user.userprofile,
    "error":None,
}
```
#### account/user PUT api/profile/?
``request``
```json
//request.data, 
/*
class UserProfile(models.Model):
    user = models.OneToOneField(User, on_delete=models.CASCADE)
    problems_status = JSONField(default=dict)
    real_name = models.TextField(null=True)
    blog = models.URLField(null=True)
    github = models.TextField(null=True)
    school = models.TextField(null=True)
    major = models.TextField(null=True)
    language = models.TextField(null=True)
    #for Contest
    accepted_number = models.IntegerField(default=0)
    total_score = models.IntegerField(default=0)
*/

```
``response``
```json
//APIView.success
{
    "msg":request.user.userprofile, // user_profile <-- request.data,
    "error":None,
}
```

#### account/user GET api/profile/fresh_display_id
``request``
```json
//profile = request.user.userprofile

```
``response``
```json
//APIView.success
{
    "msg":None,
    "error":None,
}
```

#### account/user POST api/upload_avatar/?    
``request``
```json
文件上传，request.FILES。request.FILES 只有当请求方法是 POST，至少有一个文件字段被实际发布，并且发布请求的 <form> 有 enctype="multipart/form-data" 属性时，才会包含数据。否则 request.FILES 将为空。
```
``response``
```json
//APIView.error
{
    "msg":"Invalid file content" / "Picture is too large" / "Unsupported file format" / ,
    "error":"error",
}
//APIView.success
{
    "msg":"Succeed" / request.user.userprofile,
    "error":None,
}
```
#### account/user POST api/tfa_required/?
Check TFA is required  
``request``
```json
{
    "header":{

    },
    "json":{
        "username":username,

    }
}
```
``response``
```json
//APIView.success
{
    "msg":{
        "result":Boolean,
    },
    "error":None,
}
```



#### account/user GET api/two_factor_auth/?
GET QR code       
``request``
```json
//request.user
```
``response``
```json
//APIView.error
{
    "msg":"2FA is already turned on",
    "error":"error",
}
//APIView.success
{
    "msg": base64,
    "error":None,
}
```
#### account/user POST api/two_factor_auth/?
Open 2FA        
``request``
```json
//request.user
{
    "header":{

    },
    "json":{
        "code":code,

    }
}
```
``response``
```json
//APIView.error
{
    "msg":"Invalid code",
    "error":"error",
}
//APIView.success
{
    "msg": "Succeed",
    "error":None,
}
```
#### account/user PUT api/two_factor_auth/?      
``request``
```json
//request.user
{
    "header":{

    },
    "json":{
        "code":code,

    }
}
```
``response``
```json
//APIView.error
{
    "msg":"2FA is already turned off" / "Invalid code",
    "error":"error",
}
//APIView.success
{
    "msg": "Succeed",
    "error":None,
}
```
#### account/user GET api/sessions/?      
``request``
```json
//request.session.session_key, request.user.session_keys
{
    "header":{

    },
    "json":{
        "code":code,

    }
}
```
``response``
```json

//APIView.success
{
    "msg": [
        [
            "current_session":Boolean,
            "ip":ip,
            "user_agent":user_agent,
            "last_activity":string,
            "session_key":session_key,
        ],
        ...
    ]:,
    "error":None,
}
```

#### account/user DELETE api/two_factor_auth/?      
``request``
```json
parameter:session_key
{
    "header":{

    },
    "json":{
        "code":code,

    }
}
```
``response``
```json
//APIView.error
{
    "msg":"Parameter Error" / "Invalid session_key",
    "error":"error",
}
//APIView.success
{
    "msg": "Succeed",
    "error":None,
}
```

#### account/user POST api/open_api_appkey/?      
``request``
```json
//request.user

```
``response``
```json
//APIView.error
{
    "msg":"OpenAPI function is truned off for you",
    "error":"error",
}
//APIView.success
{
    "msg": {
        
        "appkey":string,
    },
    "error":None,
}
```

#### account/user GET api/sso?      
``request``
```json
//request.user

```
``response``
```json
//APIView.success
{
    "msg": {
        
        "token":string,
    },
    "error":None,
}
```

#### account/user POST api/sso?      
``request``
```json
{
    "header":{

    },
    "json":{
        "token":string
    }
}

```
``response``
```json
//APIView.error
{
    "msg": "User does not exist",
    "error":None,
}
//APIView.success
{
    "msg": {
        "username": user.username, 
        "avatar": user.userprofile.avatar, 
        "admin_type": user.admin_type,
    },
    "error":None,
}
```

#### announcement/admin POST api//admin/announcement/?    
publish announcement  
``request``
```json
{
    "header":{

    },
    "json":{
    //data:
        "title":title,
        "content":content,
        "visible":Boolean,
        
    }
}
//request.user
```
``response``
```json
//APIView.success
{
    "msg":{
    //data
        "title":title,
        "create_time":Datetime,
        "created_by":User,  //ForeignKey
        "content":content,  //RichTextField
        "visible":Boolean,

    },
    "error":None,
}    
//request.user
```

#### announcement/admin PUT api/admin/announcement/?    
edit announcement  
``request``
```json
{
    "header":{

    },
    "json":{
        "id":id,
    }
}

```
``response``
```json
//APIView.error
{
    "msg":"Announcement does not exist" / ,
    "error":"error",
}
//APIView.success
{
    "msg":{    
    //announcement     , announcement <--request.data
        "title":title,
        "create_time":Datetime,
        "created_by":User,  //ForeignKey
        "content":content,  //RichTextField
        "last_update_time":Datetime,
        "visible":Boolean,
    },

    /*
    class Announcement(models.Model):
        title = models.TextField()
        # HTML
        content = RichTextField()
        create_time = models.DateTimeField(auto_now_add=True)
        created_by = models.ForeignKey(User, on_delete=models.CASCADE)
        last_update_time = models.DateTimeField(auto_now=True)
        visible = models.BooleanField(default=True)
    */
    "error":None,
}
```
#### announcement/admin GET api/admin/announcement/?    
get announcements / an announcement  
``request``
```json
parameter: id

```
``response``
```json
//APIView.error
{
    "msg":"Announcement does not exist" ,
    "error":"error",
}
//APIView.success
{
    "msg":
    {    
        "title":title,
        "create_time":Datetime,
        "created_by":User,  //ForeignKey
        "content":content,  //RichTextField
        "last_update_time":Datetime,
        "visible":Boolean,
    }   OR   
    {
        "result":[
            //maybe 10 announcements in one Page
            announcement 1,
            announcement 2,
            ...
        ],
        "total":number of announcement,
    },
    //announcement,     
    /*
    class Announcement(models.Model):
        title = models.TextField()
        # HTML
        content = RichTextField()
        create_time = models.DateTimeField(auto_now_add=True)
        created_by = models.ForeignKey(User, on_delete=models.CASCADE)
        last_update_time = models.DateTimeField(auto_now=True)
        visible = models.BooleanField(default=True)
    */
    "error":None,
}
```

#### announcement/admin DELETE api/admin/announcement/?    
``request``
```json
parameter: id

```
``response``
```json
//APIView.success
{
    "msg":None,
    "error":None,
}
```

#### announcement/user GET api/announcement/?    
``response``
```json
//APIView.success
{
    "msg": {
        "result":[
            //maybe 10 announcements in one Page
            announcement 1,
            announcement 2,
            ...
        ],
        "total":number of announcement,
    },
    "error":None,
}    

```

#### contest/admin GET api/admin/contest/?

``request``
```json
parameter: id & keyword
```
``response``
```json
//APIView.success
{
    "msg":Contest
        OR
    {
        "result":[
            //maybe 10 in one page
            Contest 1,
            Contest 2,
            ...
        ],
        "total":number of Contest
    },
    "error":None,
}
//APIView.error
{
    "msg":"Contest does not exist",
    "error":"error",
}
```
#### contest/admin POST api/admin/contest/?
``request``
```json
{
    "header":{

    },
    "json":{
        "title":title,
        "start_time":start_time,
        "end_time":end_time,
        "created_by":created_by,
        "contest_admin":contest_admin,
        "start_time":start_time,
        "end_time":end_time,
        "password":password,
        "allowed_ip_ranges":[
            ip_range,
            ...
        ],
        /*
        class Contest(models.Model):
            title = models.TextField()
            description = RichTextField()
            password = models.TextField(null=True)
            start_time = models.DateTimeField(timezone.now())
            end_time = models.DateTimeField(null=True)
            create_time = models.DateTimeField(auto_now_add=True)
            last_update_time = models.DateTimeField(auto_now=True)
            created_by = models.ForeignKey(User, on_delete=models.CASCADE)
            visible = models.BooleanField(default=True)
            contest_admin = models.JSONField(default=list)
            allowed_ip_ranges = JSONField(default=list)
        */
    }
}
```
``response``
```json
//APIView.success
{
    "msg":Contest,
    "error":None,
}
//APIView.error
{
    "msg":"Start time must occur earlier than end time" / "{ip_range} is not a valid cidr network",
    "error":"error",
}
```

#### contest/admin PUT api/admin/contest/?
``request``
```json
{
    "header":{

    },
    "json":{
        "id":id,
        //"title":title,
        "start_time":start_time,
        "end_time":end_time,
        //"created_by":created_by,
        "contest_admin":[
            contest_admin,
            ...
        ],
        "start_time":start_time,
        "end_time":end_time,
        "password":password,
        "allowed_ip_ranges":[
            ip_range,
            ...
        ],
        /*
        class Contest(models.Model):
            title = models.TextField()
            description = RichTextField()
            password = models.TextField(null=True)
            start_time = models.DateTimeField(timezone.now())
            end_time = models.DateTimeField(null=True)
            create_time = models.DateTimeField(auto_now_add=True)
            last_update_time = models.DateTimeField(auto_now=True)
            created_by = models.ForeignKey(User, on_delete=models.CASCADE)
            visible = models.BooleanField(default=True)
            contest_admin = models.JSONField(default=list)
            allowed_ip_ranges = JSONField(default=list)
        */
    }
}
```
``response``
```json
//APIView.success
{
    "msg":Contest,      //class Contest:
    "error":None,
}
//APIView.error
{
    "msg":"Contest does not exist" / "User:{user_id} not exist" / "Start time must occur earlier than end time" / "{ip_range} is not a valid cidr network" / ,
    "error":"error",
}
```


#### contest/admin POST api/admin/contest/announcement/?
Create one contest_announcement.
``request``
```json
{
    "header":{

    },
    "json":{
        "contest_id":contest_id,
        "title":title,
        /*
        class ContestAnnouncement(models.Model):
            contest = models.ForeignKey(Contest, on_delete=models.CASCADE)
            title = models.TextField()
            content = RichTextField()
            created_by = models.ForeignKey(User, on_delete=models.CASCADE)
            visible = models.BooleanField(default=True)
            create_time = models.DateTimeField(auto_now_add=True)
        */
    }
}
```
``response``
```json
//APIView.success
{
    "msg":ContestAnnouncement,      //class ContestAnnouncement:
    "error":None,
}
//APIView.error
{
    "msg":"Contest does not exist" /  ,
    "error":"error",
}
```


#### contest/admin PUT api/admin/contest/announcement/?
update contest_announcement.
``request``
```json
{
    "header":{

    },
    "json":{
        "id":id,
        [Optional]
        /*
        class ContestAnnouncement(models.Model):
            contest = models.ForeignKey(Contest, on_delete=models.CASCADE)
            title = models.TextField()
            content = RichTextField()
            created_by = models.ForeignKey(User, on_delete=models.CASCADE)
            visible = models.BooleanField(default=True)
            create_time = models.DateTimeField(auto_now_add=True)
        */
    }
}
```
``response``
```json
//APIView.success
{
    "msg":None,      //class ContestAnnouncement:
    "error":None,
}
//APIView.error
{
    "msg":"Contest announcement does not exist" /  ,
    "error":"error",
}
```

#### contest/admin DELETE api/admin/contest/announcement/?
delete one contest_announcement.
``request``
```json
parameter: id
```
``response``
```json
//APIView.success
{
    "msg":None,      //class ContestAnnouncement:
    "error":None,
}
```

#### contest/admin GET api/admin/contest/announcement/?
Get one contest_announcement or contest_announcement list.
``request``
```json
parameter: id & contest_id & keyword
```
``response``
```json
//APIView.success
{
    "msg":ContestAnnouncements,      //class ContestAnnouncement:
    "error":None,
}
//APIView.error
{
    "msg":"Contest announcement does not exist" /  "Parameter error",
    "error":"error",
}
```

#### contest/admin GET api/admin/download_submissions/?
``request``
```json
parameter: contest_id & exclude_admin 
```
``response``
```json
//APIView.error
{
    "msg":"Contest does not exist" /  "Parameter error",
    "error":"error",
}
//download submission
FileResponse(open_file,``kwargs)
/*
    kwargs = {
        "Content-Type":"application/zip",
        "Content-Disposition":"attachment;filename={os.path.basename(zip_path)}",
    }
*/
```

#### contest/user GET api/contests/?
``request``
```json
parameter: keyword & rule_type & status
```
``response``
```json
//APIView.success
{
    "msg":
    {
        "result":[
            //maybe 10 in one page
            Contest 1,
            Contest 2,
            ...
        ],
        "total":number of Contest
    },
    "error":None,
}
```
#### contest/user GET api/contest/?
``request``
```json
parameter: id
```
``response``
```json
//APIView.error
{
    "msg":"Invalid parameter, id is required" / "Contest does not exist" ,
    "error": "error",
}
//APIView.success
{
    "msg":
    {
        "title" : title,
        "description" : description,  //RichTextField
        "start_time" :Datetime,
        "end_time" :Datetime,
        "create_time Datetime,
        last_update_time" : Datetime,
        "created_by" :User,
        "contest_admin" :[
                //JSONField(default=list)
        ],
        "now":Datetime,
    },
    "error":None,
}
```

#### contest/user POST api/contest/password/?
``request``
```json
{
    "header":{

    },
    "json":{
        "contest_id":contest_id,
        "password":password,
    }
}
```
``response``
```json
//APIView.error
{
    "msg":"Invalid parameter, id is required" / "Contest does not exist" / "Wrong password or password expired",
    "error": "error",
}
//APIView.success
{
    "msg":True,
    "error":None,
}
```

#### contest/user GET api/contest/announcement/?
``request``
```json
parameter: contest_id & max_id &
```
``response``
```json
//APIView.error
{
    "msg":"Invalid parameter, contest_id is required",
    "error": "error",
}
//APIView.success
{
    "msg":
    {
        "result":[
            //maybe 10 in one page
            Contest 1,
            Contest 2,
            ...
        ],
        "total":number of Contest
    },
    "error":None,
}
```

#### contest/user GET api/contest/access/?
``request``
```json
parameter: contest_id 
```
``response``
```json
//APIView.error
{
    "msg":"Contest does not exist",
    "error": "error",
}
//APIView.success
{
    "msg":
    {
        "access":Boolean,
    },
    "error":None,
}
```

#### problem/admin POST api/admin/problem/?
``request``
```json
{
    "header":{

    },
    "json":{
        "_id":_id,
        "languages":[
            language 1,
            ...
        ],
        "tags":tags,
        "created_by":User,

    }
}
```
``response``
```json
//APIView.error
{
    "msg":"Display ID is required" / "Display ID already exists" / list() error,
    "error": "error",
}
//APIView.success
{
    "msg":Problem,  //class Problem(Model)
    "error":None,
}
/*
class Problem(models.Model):
    # display ID
    _id = models.TextField(db_index=True)
    contest = models.ForeignKey(Contest, null=True, on_delete=models.CASCADE)
    # for contest problem
    lab_id = models.IntegerField(null=True)
    lab_config = models.JSONField(default=dict)
    is_public = models.BooleanField(default=False)
    title = models.TextField()
    # HTML
    description = RichTextField()
    hint = RichTextField(null=True)
    languages = JSONField()
    #需要的节点数量
    vm_num = models.IntegerField()
    #各个节点所需要的端口数量
    port_num = models.JSONField(default=list)
    #学生需要编写的代码段数量
    code_num = models.IntegerField()
    template = JSONField(null=True)
    create_time = models.DateTimeField(auto_now_add=True)
    # we can not use auto_now here
    last_update_time = models.DateTimeField(null=True)
    created_by = models.ForeignKey(User, on_delete=models.CASCADE)

    # judge related
    visible = models.BooleanField(default=True)
    tags = models.ManyToManyField(ProblemTag)
    total_score = models.IntegerField(default=0)
    submission_number = models.BigIntegerField(default=0)
    accepted_number = models.BigIntegerField(default=0)
    # {JudgeStatus.ACCEPTED: 3, JudgeStaus.WRONG_ANSWER: 11}, the number means count
    statistic_info = JSONField(default=dict)
    share_submission = models.BooleanField(default=False)

*/
```
#### problem/admin GET api/admin/problem/?
``request``
```json
parameter: id & keyword
```
``response``
```json
//APIView.error
{
    "msg":"Problem does not exist" / ,
    "error": "error",
}
//APIView.success
{
    "msg":Problem
    OR
    {
        "result":[
            //maybe 10 in one page
            Problem 1,
            Problem 2,
            ...
        ],
        "total":number of Problem
    },
    //class Problem(Model)
    "error":None,
}
```
#### problem/admin PUT api/admin/problem/?
``request``
```json
{
    "header":{

    },
    "json":{
        "id":id,
        "tags":
        [
            tag 1,
            ...
        ],
        "language":[
            language 1,
            ...
        ]
    }
}
```
``response``
```json
//APIView.error
{
    "msg":"Problem does not exist"/ "Display ID is required" / "Display ID already exists" / list() error,
    "error":"error",
}
//APIView.success
{
    "msg":None,
    "error":None:
}
```
#### problem/admin DELETE api/admin/problem/?
``request``
```json
parameter: id
```
``response``
```json
//APIView.error
{
    "msg":"Invalid parameter, id is required" / "Problem does not exists",
    "error":"error",
}
//APIView.success
{
    "msg":None,
    "error":None:
}
```

#### problem/admin POST api/admin/contest/problem/?
``request``
```json
{
    "header":{

    },
    "json":
    {
        "contest_id":contest_id,
        "problem_id":problem_id,
        "lab_id":lab_id,
        "contest":Contest,
        "created_by":User,
        "display_id":display_id,
        "title":title,
        "description":description,
        //"visible"
        "language":[],
        "hint":hint,
        "lab_config":lab_config,
        "vm_num":vm_num,
        "port_num":port_num,
        "total_score":total_score,
        //"share_submission"
        "code_num":code_num,
        //"_id"
        //"id_public"
        //"accepted_number"
        //"submission_number"
    }
}
```
``response``
```json
//APIView.error
{
    "msg":"Contest or Problem does not exist" / "Contest has ended" / "Duplicate display id in this contest" / ,
    "error":"error",
}
//APIView.success
{
    "msg":Problem,
    "error":None:
}
```
#### problem/admin GET api/admin/contest/problem/?
``request``
```json
parameter: if & contest_id & keyword
```
``response``
```json
//APIView.error
{
    "msg":"Problem does not exist" / "Contest id is required" / "Contest does not exist" / ,
    "error":"error",
}
//APIView.success
{
    "msg":Problem
    OR
    {
        "result":[
            //maybe 10 in one page
            Problem 1,
            Problem 2,
            ...
        ],
        "total":number of Problem

    },
    "error":None:
}
```
#### problem/admin PUT api/admin/contest/problem/?
``request``
```json
{
    "header":{

    },
    "json":{
        "contest_id":contest_id,
        "id":id,
        "_id":_id,  //display_id
        "tags":[],
        "language":[]
    }
}
```
``response``
```json
//APIView.error
{
    "msg":"Contest does not exist" / "Problem does not exist" / "Display ID is required" / "Display ID already exists" / list() error,
    "error":"error",
}
//APIView.success
{
    "msg":None,
    "error":None:
}
```
#### problem/admin DELETE api/admin/contest/problem/?
``request``
```json
parameter: id 
```
``response``
```json
//APIView.error
{
    "msg":"Invalid parameter, id is required" / "Problem does not exists" / "Can't delete the problem as it has submissions",
    "error":"error",
}
//APIView.success
{
    "msg":None,
    "error":None:
}
```

#### problem/admin POST api/admin/contest_problem/make_public/?
``request``
```json
{
    "header":{

    },
    "json":{
        "display_id":display_id,
        "id":id,
    }
}
```
``response``
```json
//APIView.error
{
    "msg":"Duplicate display ID" / "Problem does not exist" / "Already be a public problem" ,
    "error":"error",
}
//APIView.success
{
    "msg":None,
    "error":None:
}
```

#### problem/admin POST api/admin/contest/add_problem_from_public/?
``request``
```json
{
    "header":{

    },
    "json":{
        "contest_id":contest_id,
        "problem_id":problem_id,
        "lab_id":lab_id,
        "created_by":User,
        "display_id":display_id,
        //"title"
        //"description"
        //"hint"
        "lab_config":lab_config,

    }
}
```
``response``
```json
//APIView.error
{
    "msg":"Contest or Problem does not exist" / "Contest has ended" / "Duplicate display id in this contest" /  ,
    "error":"error",
}
//APIView.success
{
    "msg":None,
    "error":None:
}
```

#### problem/admin GET api/admin/export_problem/?
``request``
```json
{
    "header":{

    },
    "json":{
        "problem_id":problem_id, 
    }
}
```
``response``
```json
FileResponse(open_file,``kwargs)
/*
    kwargs = {
        "Content-Type":"application/zip",
        "Content-Disposition":"attachment;filename=problem-export.zip",
    }
*/
```

#### problem/user GET api/problem/tags/?
``request``
```json
parameter: keyword
```
``response``
```json

//APIView.success
{
    "msg":
    {
        "result":[
            //maybe 10 in one page
            Tag 1,
            Tag 2,
            ...
        ],
        "total":number of Tag
    },
    "error":None:
}
```

#### problem/user GET api/problem/?
``request``
```json
parameter: problem_id & limit & tag & keyword
```
``response``
```json
//APIView.error
{
    "msg":"Problem does not exist"/"Limit is needed",
    "error":"error",
}

//APIView.success
{
    "msg":Problem
    OR
    {
        "result":[
            //maybe 10 in one page
            Problem 1,
            Problem 2,
            ...
        ],
        "total":number of Problem
    },
    "error":None:
}
```
#### problem/user GET api/pickone/?
``request``
```json
parameter: problem_id & limit & tag & keyword
```
``response``
```json
//APIView.error
{
    "msg":"No problem to pick",
    "error":"error",
}

//APIView.success
{
    "msg":Problem._id,
    "error":None:
}
```

#### problem/user GET api/contest/problem/?
``request``
```json
parameter: problem_id
```
``response``
```json
//APIView.error
{
    "msg":"Problem does not exist." / ,
    "error":"error",
}

//APIView.success
{
    "msg":{
    //class  ProblemSafeSerializer(BaseProblemSerializer):
        "_id" // models.TextField(db_index=True)
        "contest" // models.ForeignKey(Contest, null=True, on_delete=models.CASCADE)
        # for contest problem
        "lab_id" // models.IntegerField(null=True)
        "lab_config" // models.JSONField(default=dict)
        "title" // models.TextField()
        # HTML
        "description" //RichTextField()
        "hint" // RichTextField(null=True)
        "languages" // JSONField()
        #需要的节点数量
        "vm_num" // models.IntegerField()
        #各个节点所需要的端口数量
        "port_num" // models.JSONField(default=list)
        #学生需要编写的代码段数量
        "code_num" // models.IntegerField()
        "template" // JSONField(null=True)
        "create_time" // models.DateTimeField(auto_now_add=True)
        # we can not use auto_now here
        "last_update_time" // models.DateTimeField(null=True)
        "created_by" // models.ForeignKey(User, on_delete=models.CASCADE)
        # judge related
        "tags" // models.ManyToManyField(ProblemTag)
        "total_score" // models.IntegerField(default=0)
        # {JudgeStatus.ACCEPTED: 3, JudgeStaus.WRONG_ANSWER: 11}, the number means count
        "share_submission" // models.BooleanField(default=False)
    },
    "error":None:
}
```


#### submission/user POST api/submission/?
``request``
```json
{
    "header":{

    },
    "json":{
        "contest_id":contest_id,
        "auth_method":auth_method,
        "problem_id":problem_id,
        "language":language,
        "code_list":[],
    }
}
```
``response``
```json
//APIView.error
{
    "msg":"The contest have ended" / "Your IP is not allowed in this contest" / "Please wait %d seconds" / "Problem not exist" / "{language} is now allowed in the problem"/"Code segment can't meet the requirement",
    "error":"error",
}

//APIView.success
{
    "msg":Submission
    OR
    None,
    /*
    class Submission(models.Model):
        id = models.TextField(default=rand_str, primary_key=True, db_index=True)
        contest = models.ForeignKey(Contest, null=True, on_delete=models.CASCADE)
        problem = models.ForeignKey(Problem, on_delete=models.CASCADE)
        create_time = models.DateTimeField(auto_now_add=True)
        user_id = models.CharField(db_index=True, max_length=50)
        username = models.TextField()
        code_list = models.JSONField(default=list)
        server_list = models.JSONField(default=list)
        ports_list = models.JSONField(default=dict)
        result = models.IntegerField(db_index=True, default=JudgeStatus.PENDING)
        # 从JudgeServer返回的判题详情
        info = JSONField(default=dict)
        language = models.TextField()
        shared = models.BooleanField(default=False)
        # 存储该提交所用时间和内存值，方便提交列表显示
        # {time_cost: "", memory_cost: "", err_info: "", score: 0}
        ip = models.TextField(null=True)
    */
    "error":None:
}
```
#### submission/user GET api/submission/?
``request``
```json
parameter: id 
```
``response``
```json
//APIView.error
{
    "msg":"Parameter id doesn't exist" / "Submission doesn't exist"/ "No permission for this submission",
    "error":"error",
}

//APIView.success
{
    "msg":Submission,
    "error":None:
}
```
#### submission/user PUT api/submission/?
``request``
```json
{
    "header":{

    },
    "json":
    {
        "id":id,
        "shared":shared,
    }
}
```
``response``
```json
//APIView.error
{
    "msg":"Submission doesn't exist" / "No permission to share the submission" / "Can not share submission now",
    "error":"error",
}

//APIView.success
{
    "msg":None,
    "error":None:
}
```
#### submission/user GET api/submissions/?
``request``
```json
parameter: limit & contest_id & problem_id & result
```
``response``
```json
//APIView.error
{
    "msg":"Problem doesn't exist" / "Parameter error" / "Limit is needed",
    "error":"error",
}

//APIView.success
{
    "msg":
    {
        "result":[
            //maybe 10 in one page
            Submission 1,   //Submission exclude ("info", "contest", "code_list", "ip")
            Submission 2,
            ...
        ],
        "total":number of Problem
    },
    "error":None:
}
```
#### submission/user GET api/submission_exists/?
``request``
```json
parameter: problem_id
```
``response``
```json
//APIView.error
{
    "msg":"Parameter error, problem_id is required",
    "error":"error",
}

//APIView.success
{
    "msg":Boolean,
    "error":None:
}
```
#### submission/user GET api/contest_submissions/?
``request``
```json
parameter: limit & contest_id & problem_id & result
```
``response``
```json
//APIView.error
{
    "msg":"Limit is needed"/"Problem doesn't exist",
    "error":"error",
}

//APIView.success
{
    "msg":
    {
        "result":[
            //maybe 10 in one page
            Submission 1,   //Submission exclude ("info", "contest", "code_list", "ip")
            Submission 2,
            ...
        ],
        "total":number of Problem,
    },
    "error":None:
}
```


#### submission/admin GET api/admin/submission/rejudge/?
``request``
```json
parameter: id
```
``response``
```json
//APIView.error
{
    "msg":"Parameter error, id is required" / "Submission does not exists",
    "error":"error",
}

//APIView.success
{
    "msg":None,
    "error":None:
}
```


#### submission/admin POST api/admin/submission/excution/?
``request``
```json
{
    "header":{
        "HTTP_X_JUDGE_SERVER_TOKEN":token,
    },
    "json":{
        "submission_id":id,
        "result":result,
        "info":submission_info,
    }
}
```
``response``
```json
//APIView.error
{
    "msg":"HTTP_X_JUDGE_SERVER_TOKEN" / "illegal server submission put" / "submission_id is needed" / "submmission not exist or expired" /"submission result is needed" / "resource Fetch error",
    "error":"error",
}

//APIView.success
{
    "msg":Submission,
    "error":None:
}
```

#### submission/admin GET api/admin/submission/?
``request``
```json
parameter: id & find_user_id & problem_id & contest_id
```
``response``
```json
//APIView.error
{
    "msg":"Submission not exist" / "Contest not exist" /"Problem not exist"/"User not exist",
    "error":"error",
}

//APIView.success
{
    "msg":    
    {
        "result":[
            //maybe 10 in one page
            Submission 1,   //Submission exclude ("info", "contest", "code_list", "ip")
            Submission 2,
            ...
        ],
        "total":number of Problem,
    },
    "error":None:
}
```
#### submission/admin PUT api/admin/submission/?
``request``
```json
{
    "header":{

    },
    "json":{
        "submission_id":submission_id,

    }
}
```
``response``
```json
//APIView.error
{
    "msg":"submission_id is needed" / "Submission not exist",
    "error":"error",
}

//APIView.success
{
    "msg":  None,
    "error":None:
}
```

#### utils/views POST api/admin/upload_image/?
``request``
```json
    request.POST & request.FILES
```
``response``
```json
HttpResponse:
{
    "success": False,
    "msg": "Unsupported file format",
    "file_path": ""
}
OR
{
    "success": False,
    "msg": "Upload Error",
    "file_path": ""}
OR
{
    "success": True,
    "msg": "Success",
    "file_path": f"{settings.UPLOAD_PREFIX}/{img_name}"
}
```

#### utils/views POST api/admin/upload_file/?
``request``
```json
    request.POST & request.FILES
```
``response``
```json
HttpResponse:
{
    "success": False,
    "msg": "Upload failed"
}
OR
{
    "success": False,
    "msg": "Upload Error"
}
OR
{
    "success": True,
    "msg": "Success",
    "file_path": f"{settings.UPLOAD_PREFIX}/{file_name}",
    "file_name": file.name
}
```

#### conf/admin GET api/admin/smtp/?
``response``
```json
//APIView.success
{
    "msg":None  OR  SysOptions.value,   //JSONField()
    "error":None,
}
```


#### conf/admin POST api/admin/smtp/?
``request``
```json
{
    "header":{

    },
    "json":
    {
        "smtp_config":smtp_config
    }
}
```
``response``
```json
//APIView.success
{
    "msg":None,
    "error":None,
}
```

#### conf/admin PUT api/admin/smtp/?
``request``
```json
{
    "header":{

    },
    "json":
    {
        "server":server,
        "port":port,
        "email":email,
        "tls":tls,
        "password":password,
    }
}
```
``response``
```json
//APIView.success
{
    "msg":None,
    "error":None,
}
```

#### conf/admin POST api/admin/smtp_test/?
``request``
```json
{
    "header":{

    },
    "json":
    {
        "server":server,
        "port":port,
        "email":email,
        "tls":tls,
        "password":password,
    }
}
```
``response``
```json
//APIView.success
{
    "msg":None,
    "error":None,
}
//APIView.error
{
    "msg":"Please setup SMTP config at first" / STMP Error,
    "error":"error",
}
```

#### conf/admin GET api/admin/website/?
``request``
```json
{
    "header":{

    },
    "json":
    {
    }
}
```
``response``
```json
//APIView.success
{
    "msg":{
        "website_base_url", 
        "website_name", 
        "website_name_shortcut",
        "website_footer", 
        "allow_register", 
        "submission_list_show_all"
    },
    "error":None,
}

```
#### conf/admin POST api/admin/website/?
``request``
```json
{
    "header":{

    },
    "json":
    {
        "website_base_url":, 
        "website_name":, 
        "website_name_shortcut":,
        "website_footer":, 
        "allow_register":, 
        "submission_list_show_all":,
    }
}
```
``response``
```json
//APIView.success
{
    "msg":None,
    "error":None,
}
```
#### conf/admin GET api/admin/judge_server/?

``response``
```json
//APIView.success
{
    "msg":{
        "token": judge_server_token,
        "servers": JudgeServer
    },
    "error":None,
}
```
#### conf/admin DELETE api/admin/website/?
``request``
```json
parameter: hostname
```
``response``
```json
//APIView.success
{
    "msg":None,
    "error":None,
}
```
#### conf/admin PUT api/admin/website/?
``request``
```json
{
    "header":{

    },
    "json":{
        "is_disabled":Boolean,
        "id":id,
    }
}
```
``response``
```json
//APIView.success
{
    "msg":None,
    "error":None,
}
```

#### conf/admin GET api/admin/prune_test_case/?

``response``
```json
//APIView.success
{
    "msg":[
        {
            "id":name,
            "create_time":Datetime,
        }
    ],
    "error":None,
}
```

#### conf/admin DELETE api/admin/prune_test_case/?
``request``
```json
parameter: id
```
``response``
```json
//APIView.success
{
    "msg":None,
    "error":None,
}
```

#### conf/admin GET api/admin/versions/?
``response``
```json
//APIView.success
{
    "msg":releases,     // releases info from github, add "local_version"
    "error":None,
}
```

#### conf/admin GET api/admin/dashboard_info/?
``response``
```json
//APIView.success
{
    "msg":{
            "user_count": User.objects.count(),
            "recent_contest_count": recent_contest_count,
            "today_submission_count": today_submission_count,
            "judge_server_count": judge_server_count,
            "env": {
                "FORCE_HTTPS": get_env("FORCE_HTTPS", default=False),
                "STATIC_CDN_HOST": get_env("STATIC_CDN_HOST", default="")
            },
    "error":None,
    }
}
```




### JudgeServer

#### 任务下发 POST /judge

``request``

```json
// POST /judge
{
    //认证 token为控制节点与实验运行节点之间的认证令牌
	"headers":{
        "X-Judge-Server-Token": token
    },
    "json":{
        //submission字段ID
        "submission_id": submission_id,
        //语言字段 暂时默认为python
        "language_config": lang,
        //实验ID
        "lab_id": lab_id,
        //实验相关参数配置
        "lab_config":{
            "loss_rate": 0.01,
            "packet_size": 64KB,
            //...
        },
        //学生提交的代码段
        "src":["code_segment1",
               "code_segment2",
               "code_segment3",
               //...
        ],
        //在实验中的Node编号。一般编号为0的节点负责反馈实验状态
        "vm_index": 0,
        //实验使用的端口号
        "ports": [8000,8080,...],
    }
} 
```

``response``

```json
//normal 正确
{"status":success}

//error 错误
//主要是在Token认证不通过，已经实验配置不合法的情况下报错
{
    //err字段记录错误原因
    "err": ("Token illegal"/"lab resource not found"
    	   "lab config error"/ "port blocked"),
   //info记录原请求信息Debug
    "info": {
        "X-Judge-Server-Token": token,
        "submission_id": submission_id,
    	"language_config": lang,
        "lab_id": lab_id,
        "lab_config":{
            "loss_rate": 0.01,
            "packet_size": 64KB,
            //...
        },
        "src":["code_segment1",
               "code_segment2",
               "code_segment3",
               //...
        ],
        "vm_index": 0,
        "ports": [8000,8080,...],
    }
}
```



#### 实验状态查询请求 POST /fetch

``request``

```json
// POST /fetch
{
    //认证 token为控制节点与实验运行节点之间的认证令牌
	"headers":{
        "X-Judge-Server-Token": token,...
    },
    "json":{
        //同上
        "submission_id": submission_id,
        "lab_id": lab_id,
        "vm_index": 0,
    }
}
```

``response``

```json
//正确
{
    "status":success,
    //提交状态
    "result": 0
}

//error 错误
//主要是在Token认证不通过，已经实验配置不合法的情况下报错
{
    //err字段记录错误原因
    "err": ("Token illegal"/"submission not found"/"vm index error"),
   //info记录原请求信息Debug
    "info": {
        "X-Judge-Server-Token": token,
        "submission_id": submission_id,
        "lab_id": lab_id,
        "vm_index": 0,
    }
}
```

除了简单在response中返回对应状态，JudgeServer会主动进行一次状态推送。以更新详细信息



### ONL Server

#### 提交状态更新 PUT admin/submission/excution

```json
{
    //认证 token为控制节点与实验运行节点之间的认证令牌
	"headers":{
        "X-Judge-Server-Token": token,...
    },
    "json":{
        //同上
        "submission_id": submission_id,
        //实验ID
        "lab_id": lab_id,
        //实验结果
        "result": 0,
        //实验信息记录
        "info":{
            "test_case1": true,
            "test_case2": true,
            "lab_start": XXXX/XX/XX,
            "lab_end": XXXX/XX/XX,
        },
    }
}
```


