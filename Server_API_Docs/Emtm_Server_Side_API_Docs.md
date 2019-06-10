# Emtm Server APIs



### 请求方式：POST

### 请求参数格式："Content-Type:application/json"



## Part One. 奶牛 APIs

> 注册登录部分

1. 奶牛认证信息功能

   * 接口地址：服务器地址/cow_logup

   * 请求参数：

     ```js
     {
     	 "username" : 奶牛用户昵称，可以考虑使用微信昵称[string],
         "userid" : 奶牛用户id[string],
         "wechat_ok" : 是否通过微信账号认证[boolean] // false or true
         "email" : 奶牛用户邮箱[string],
         "phone" : 奶牛用户手机号码[string],
         "infos" : 奶牛组织的相关介绍信息[string],
         "organization" : 奶牛用户机构名称[string]
         // 目前还在思考如何验证奶牛用户身份
     }
     ```

   * 返回格式：

     ```js
     {
         "code" : boolean, // false or true
         "err_message" : string
     }
     ```

     ​

2. 登录功能(公用功能，在用户使用微信账号登录小程序之后调用查看是否注册)

   * 接口地址：服务器地址/login

   * 请求参数：

     ```js
     {
         "userid" : 奶牛用户id[string],
         "wechat_ok" : 是否通过微信账号认证[boolean] // false or true
     }
     ```

   * 返回格式：

     ```js
     {
         "code" : boolean, // false or true
         "err_message" : string
     }
     ```

     ​


> 发布任务部分

1. 奶牛发布任务功能

   * 接口地址：服务器地址/release_task

   * 请求参数：

     ```js
     {
         "userid" : 奶牛用户id[string],
         "release_mode" : false, // 奶牛用户发布任务模式与大学生区分开
         "task_name" : 任务名称[string],
         "task_intro" : 任务介绍[string],
         "task_mode" : 发布任务类型[int],//系统目前提供三种：0.问卷调查，1.文档翻译，2.帮忙取件
         "task_request" : {
             // 该json参数是奶牛用户发布任务时专用，需要完善任务要求内容
             // 学生用户发布时，同样需要将本参数的每一项完善，由于Rust静态读取的特性，可以填入缺省值
             "grade" : 目标学生年级[int],
             "major" : 目标学生专业[string],
             "task_expe" : 目标学生任务经验下限[int], // 此处以学生历史完成的任务数来衡量
             "credit_score" : 目标学生的信誉积分下限[int],
             "max_participants": 最大参加人数[int]
         }[json-object],
         "task_risk" : 任务未完成扣除积分数[int],
         "task_pay" : 任务薪酬[int],
         "task_time_limit" : 任务最终deadline时间戳，格式为：yyyy-mm-dd:hh-mm[string]
     }
     ```

   * 返回格式：

     ```js
     {
         "code" : boolean, // false or true
         "err_message" : string
     }
     ```

     ​


> 查看任务部分

1. 奶牛查看任务功能

   * 接口地址：服务器地址/check_task

   * 请求参数：

     ```js
     {
         "userid" : 奶牛用户id[string],
         "check_mode" : false, // 奶牛查看任务模式，须与大学生查看任务区分开
         "task_name" : 目标任务的名称[string]
     }
     ```

   * 返回格式：

     ```js
     {
         "code" : boolean, // false or true
         "err_message" : string,
         "task_state" : string, // 任务状态，进行中或者已结束
         // 当code为false时，task_status字段为空对象
         "task_status" : [
             // 存储所有已接受任务的学生id，与完成与否信息
             {
                 "student_userid" : string, // 参与活动的学生微信ID
                 "is_finish" : boolean, // false for not finished, 1 for finished
             }
             
             ...
         ]
         // 目前还在考虑如何支持奶牛查看大学生任务完成的结果
     }
     ```

     ​


## Part Two. 大学生 APIs

> 注册登录部分

1. 大学生信息认证功能

   * 接口地址：服务器地址/stu_logup

   * 请求参数：

     ```js
     {
     	 "username" : 大学生用户昵称，可以考虑设置为微信昵称[string], 
         "userid" : 大学生用户id[string],
         "wechat_ok" : 是否通过微信账号认证[boolean] // false or true
         "email" : 大学生用户邮箱[string],
         "phone" : 大学生用户手机号[string],
         "infos" : 大学生兴趣爱好，自我介绍[string],
         // 下面是大学生身份认证所需信息
         "school_name" : 大学生所在学校[string],
         "student_id" : 大学生学号[int],
         "major" : 大学生真实姓名[string],
         "year" : 大学生在校就读年长[int]
     }
     ```

   * 返回格式：

     ```js
     {
         "code" : boolean, // false or true
         "err_message" : string
     }
     ```

     ​

   ​

2. 登录功能(公用功能)

   * 接口地址：服务器地址/login

   * 请求参数：

     ```js
     {
         "userid" : 用户id[string],
         "wechat_ok" : 是否通过微信账号认证[boolean] // false or true
     }
     ```

   * 返回格式：

     ```js
     {
         "code" : boolean, // false or true
         "type" : int, // -1 for not registered, 0 for cow registered, 1 for student
         "err_message" : string
     }
     ```

     ​


> 创建和加入群组部分

1. 创建群聊功能

   * 接口地址：服务器地址/create_group

   * 请求参数：

     ```js
     {
         "userid" : 创建群聊的大学生id，作为群主[string],
         "group_name" : 群聊名称[string],
         "max_limit" : 群聊人数上限[int]
     }
     ```

   * 返回格式：

     ```js
     {
         "code" : boolean, // false or true
         "err_message" : string
     }
     ```

     ​

2. 加入群聊功能

   * 接口地址：服务器地址/join_group

   * 请求参数：

     ```js
     {
         "userid" : 加入群聊的大学生id[string],
         "group_name" : 目标群聊的名称[string]
     }
     ```

   * 返回格式：

     ```js
     {
         "code" : boolean, // false or true
         "err_message" : string
     }
     ```

     ​

3. 添加个人好友功能

   * 接口地址：服务器地址/add_friend

   * 请求参数：

     ```js
     {
         "uername" : 当前大学生用户的id[string],
         "friend_name" : 目标大学生好友id[string]
     }
     ```

   * 返回格式：

     ```js
     {
         "code" : boolean, // false or true
         "err_message" : string
     }
     ```




> 接受任务部分

1. 接受奶牛任务功能

   * 接口地址：服务器地址/receive_task

   * 请求参数：

     ```js
     {
         "userid" : 当前大学生用户id[string],
         "receive_mode" : false, // 接收奶牛发布的任务，须与接收大学生私人任务区分
         "target_userid" : 发布任务的目标奶牛用户id[string],
         "target_task" : 目标任务名称[string]
     }
     ```

   * 返回格式：

     ```js
     {
         "code" : boolean, // false or true
         "err_message" : string
     }
     ```





2. 提交奶牛任务功能

   * 接口地址：服务器地址/submit_task

   * 请求参数：

     ```js
     {
         "userid" : 当前大学生用户名称,
         "submit_mode" : false, // 提交奶牛发起的任务应与提交私人任务分开
         "target_userid" : 发布任务的目标用户id[string],
         "target_task" : 目标任务名称[string],
         // 目前还在考虑大学生用户以何种形式提交任务结果
     }
     ```

   * 返回格式：

     ```js
     {
         "code" : boolean, // false or true
         "err_message" : string
     }
     ```

     ​

> 发布私人任务部分

1. 发布私人任务功能

   * 接口地址：服务器地址/release_task

   * 请求参数：

     ```js
     {
         "userid" : 奶牛用户id[string],
         "release_mode" : true, // 私人任务发布模式与奶牛任务区分开
         "task_name" : 任务名称[string],
         "task_content" : 私人任务要求[string],
         "task_pay" : 任务薪酬[int],
         "task_time_limit" : 任务最终deadline时间戳，格式为：yyyy-mm-dd:hh-mm[string]
     }
     ```

   * 返回格式：

     ```js
     {
         "code" : boolean, // false or true
         "err_message" : string
     }
     ```

     ​

2. 查看自己发布的私人任务状态

   * 接口地址：服务器地址/check_task

   * 请求参数：

     ```js
     {
         "userid" : 大学生用户id[string],
         "check_mode" : true, // 大学生查看任务模式，须与奶牛查看任务区分开
         "task_name" : 目标任务的名称[string]
     }
     ```

   * 返回格式：

     ```js
     {
         "code" : boolean, // false or true
         "err_message" : string,
         // 当code为false时，task_status字段为空对象
         "task_status" : {
             // 存储所有已接受任务的学生id，与完成与否信息
             "student_name_1" : boolean, // false for not finished, true for finished
             ...
         }[js-object]
     }
     ```

     ​


> 接受私人任务部分

1. 接受私人任务功能

   * 接口地址：服务器地址/receive_task

   * 请求参数：

     ```js
     {
         "userid" : 当前大学生用户id[string],
         "receive_mode" : true, // 接收私人发布的任务，须与接受奶牛任务区分
         "target_userid" : 发布任务的目标大学生用户id[string],
         "target_task" : 目标任务名称[string]
     }
     ```

   * 返回格式：

     ```js
     {
         "code" : boolean, // false or true
         "err_message" : string
     }
     ```

     ​

2. 提交私人任务功能

   * 接口地址：服务器地址/submit_task

   * 请求参数：

      ```js
      {
          "userid" : 当前大学生用户名称,
          "submit_mode" : true, // 提交私人任务应与提交奶牛任务分开
          "target_userid" : 发布任务的目标用户id[string],
          "target_task" : 目标任务名称[string],
          // 目前还在考虑大学生用户以何种形式提交任务结果
      }
      ```

   * 返回格式：

      ```js
      {
          "code" : boolean, // false or true
          "err_message" : string
      }
      ```

   ​


> 信誉积分部分

1. 查看信誉积分功能

   * 接口地址：服务器地址/check_credit

   * 请求参数：

     ```js
     {
         "userid" : 当前大学生用户id[string]
     }
     ```

   * 返回格式：

     ```js
     {
         "code" : boolean, // false or true
         "err_message" : string,
         "credit_score" : int
     }
     ```

     ​


> 资金管理部分

1. 充值功能

   * 接口地址：服务器地址/recharge

   * 请求参数：

     ```js
     {
         "userid" : 当前大学生用户id[string],
         "recharge_amount" : 充值金额[int] // 以闲钱币作为充值单位
     }
     ```

   * 返回格式：

     ```js
     {
         "code" : boolean, // false or true
         "err_message" : string
     }
     ```

     ​

2. 提现功能

   * 接口地址：服务器地址/withdraw

   * 请求参数：

     ```js
     {
         "userid" : 当前大学生用户id[string],
         "withdraw_amount" : 提现金额[int] // 以闲钱币作为提现单位
     }
     ```

   * 返回格式：

     ```js
     {
         "code" : boolean, // false or true
         "err_message" : string
     }
     ```

     ​

## Part Three. 公用APIs

> 获取用户自身的微信id

1. 用户微信ID获取功能

	* 接口地址：服务器地址/get_wechatid

	* 请求参数：

	```js
	{
		"appid": 小程序appid[string],
		"secret": 小程序appsecret[string], 
		"code": 前端登录时获取的code[string]
	}
	```

	* 返回格式：

	```js
	{
		"openid": 用户的微信id[string],
		"errcode": 错误码[int],
		"errmsg": 错误信息[string]
	}
	```


> 任务展示与搜索部分

1. 任务搜索功能

   * 接口地址：服务器地址/search_mission

   * 请求参数：

     ```js
     {
         "keyword" : 搜索框内输入的关键词[string]
     }
     ```

   * 返回格式：

     ```js
     {
         "code" : boolean, // false or true
         "err_message" : string,
         "search_result" : [
             {
                 "mid" : 任务数据库id[int], "name" : 任务名称[string], 
                 "content" : 任务描述[string], "poster_userid" : 发布任务者的微信id[string],
                 "time_limit" : 任务截止时间[string],
                 "score" : 搜索关键词与该任务关联分数，分数越高关联度越高，前端按score顺序展示[float]
             }
             ...
             ...
         ]
     }
     ```

     ​



> 个人信息部分





