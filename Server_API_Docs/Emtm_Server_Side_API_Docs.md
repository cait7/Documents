# Emtm Server APIs



### 请求方式：POST

### 请求参数格式："Content-Type:application/json"


## Part Zero. 通用 APIs
1. 登录功能(公用功能)

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
         "user_type" : int, // 0 for cow registered, 1 for student, 2 for unregisterd user
         "err_message" : string
     }
     ```

> 通用发布任务部分

1. 发布任务功能 - 第一步(完成任务通用参数传递)(这里不进行用户发布模式的检验，前端需要自行规定奶牛/学生允许发布的任务类型)

   * 接口地址：服务器地址/release_task

   * 请求参数：(Option 字段可以不传输)

     ```js
     {
         "userid" : 用户id[string],
         "task_name" : 任务名称[string],
         "task_intro" : 任务介绍[string],
         "task_mode" : 发布任务类型[int],//系统目前提供三种：0.问卷调查，1.闲置交易，2.帮忙取件
         "task_request" : {
             // 该json参数是奶牛用户发布任务时专用，需要完善任务要求内容
             "min_grade" : 目标学生最低年级[int] - Option, 
             "max_grade" : 目标学生最高年级[int] - Option,
             "major" : 目标学生专业[string] - Option,
             "task_expe" : 目标学生任务经验下限[int] - Option, // 此处以学生历史完成的任务数来衡量
             "credit_score" : 目标学生的信誉积分下限[int] - Option,
             "max_participants": 最大参加人数[int] - 必须传输
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
         "err_message" : string,
         "mid" : int // 新建任务的ID
     }
     ```
执行完第一步之后，必须根据不同的任务种类，前端选择不同的接口进行调用，完成详细任务的参数补充

2. 发布任务 - 第二步(针对不同种类的任务进行信息填充)

    * 问卷调查 - 接口地址：服务器地址/release_task_question

    * 请求参数：(Option 字段可以不传输)

    ```js
    {
        "mid": 上一步新建的任务ID[int],
        // 问卷中所有的问题按照序号排序，存储到问题数组中，每个问题为一个json对象，严格按照以下结构存储
        "questions":[
            {
                "order": 当前题目序号[int],
                "q_type": 题目类型[int], // 0 为填空题, 1 为单选题， 2 为多选题
                "content": 题目内容[string],
                "choices": 题目提供的选项[array] - Option // 填空题不需要选项，选择题需要将选项按顺序放置
            }[json-object],
            ...
        ][array]
    }
    ```

    * 闲置交易 - 接口地址：服务器地址/release_task_transaction

    * 请求参数：

    ```js
    {
        "mid": 上一步新建的任务ID[int],
        "t_type": 交易商品类型[string],
        "info": 交易商品信息描述[string],
        "loss": 交易商品的损耗情况[int] // 0~5作为损耗范围，其中0为无损耗，5为完全损耗
        "address": 交易商品的地点
    }
    ```

    * 取寄快递 - 接口地址：服务器地址/release_task_errand

    * 请求参数：

    ```js
    {
        "mid": 上一步新建的任务ID[int],
        "address": 取寄快递地址[string],
        "phone_number": 发布任务者的电话号码[string],
        "pick_number": 快递号[string],
        "info": 快递的其余信息[string]
    }
    ```

    * 通用 - 返回格式：

    ```js
    {
        "code" : boolean, // false or true
        "err_message" : string
    }
    ```


> 查看任务部分

0. 查看当前系统的某一类型的所有任务

    * 接口地址：服务器地址/get_tasks

    * 请求参数：

    ```js
    {
        "task_type": 任务类型[int] //系统目前提供三种：0.问卷调查，1.闲置交易，2.帮忙取件
    }
    ```

    * 返回格式：

    ```js
    {
        "code": boolean, // true or false
        "err_message": string,
        "tasks":[
            {
                "mid": 该任务的唯一id[int],
                "poster_id": 任务发布者的唯一标识id[string],
                "poster_name": 任务发布者的用户昵称[string],
                "task_state" : 当前任务的状态[boolean], // true代表任务进行中，false代表任务已截止
                 // 当前任务的所有详细信息字段
                 "task_name" : 任务名称[string],
                 "task_intro" : 任务介绍[string],
                 "task_mode" : 发布任务类型[int],//系统目前提供三种：0.问卷调查，1.闲置交易，2.帮忙取件
                 "task_pay" : 任务薪酬[int],
                 "task_time_limit" : 任务最终deadline时间戳，格式为：yyyy-mm-dd:hh-mm[string]
            }[json-object],
            ...
        ][array]
    }
    ```


1. 查看自己接收的任务(用于任务列表显示，返回的都是简述)

	* 接口地址：服务器地址/check_task_self_receive

	* 请求参数：

	```js
	{
		"userid": 用户唯一标识id[string]
	}
	```

	* 返回格式：

	```js
	{
		"code": boolean, // false for failed, true for success
		"err_message": string,
		"tasks":[
			{
				"mid": 该任务的唯一id[int],
				"poster_id": 任务发布者的唯一标识id[string],
				"poster_name": 任务发布者的用户昵称[string],
				"task_state" : 当前任务的状态[boolean], // true代表任务进行中，false代表任务已截止
				"user_finish_state" : 当前用户是否完成该任务[boolean], // true已完成，false未完成
		         // 当前任务的所有详细信息字段
		         "task_name" : 任务名称[string],
		         "task_intro" : 任务介绍[string],
		         "task_mode" : 发布任务类型[int],//系统目前提供三种：0.问卷调查，1.闲置交易，2.帮忙取件
		         "task_pay" : 任务薪酬[int],
         		 "task_time_limit" : 任务最终deadline时间戳，格式为：yyyy-mm-dd:hh-mm[string]
			}[json-object]
			...
		][array]
	}
	```
   

2. 查看自己发布的任务(用于任务列表显示，返回的都是简述)

	* 接口地址：服务器地址/check_task_self_release

	* 请求参数：

	```js
	{
		"userid": 用户唯一标识id[string]
	}
	```

	* 返回格式：

	```js
	{
		"code": boolean, // false for failed, true for success
		"err_message": string,
		"tasks":[
			{
				"mid": 该任务的唯一id[int],
				"poster_id": 任务发布者的唯一标识id[string],
				"poster_name": 任务发布者的用户昵称[string],
				"task_state" : 当前任务的状态[boolean], // true代表任务进行中，false代表任务已截止
		         // 当前任务的所有详细信息字段
		         "task_name" : 任务名称[string],
		         "task_intro" : 任务介绍[string],
		         "task_mode" : 发布任务类型[int],//系统目前提供三种：0.问卷调查，1.闲置交易，2.帮忙取件
		         "task_pay" : 任务薪酬[int],
         		 "task_time_limit" : 任务最终deadline时间戳，格式为：yyyy-mm-dd:hh-mm[string]
			}[json-object]
			...
		][array]
	}
	```



3. 查看特定任务详细信息

	* 接口地址：服务器地址/check_task

   * 请求参数：

     ```js
     {
         "userid" : 当前用户id[string],
         "poster_id" : 发布任务者id[string],
         "task_mid" : 目标任务的id[string]
     }
     ```

   * 返回格式：

     ```js
     {
         "code" : boolean, // false or true
         "err_message" : string,
         "task_state" : boolean, // 任务状态，进行中为true, 或者已结束为false
         // 0 代表用户是任务发布者, 1 代表用户是任务接受者并已经完成任务， 2 为用户暂未完成任务， 3 代表用户未接受任务
         "task_user_state" : 用户对于任务的状态[int],
         // 当前任务的所有详细信息字段
         "poster_name" : 发布者的用户昵称[string],
         "task_name" : 任务名称[string],
         "task_intro" : 任务介绍[string],
         "task_mode" : 发布任务类型[int],//系统目前提供三种：0.问卷调查，1.闲置交易，2.帮忙取件
         "task_request" : {
             // 该json参数是奶牛用户发布任务时专用，需要完善任务要求内容
             "min_grade" : 目标学生最低年级[int] - Option, 
             "max_grade" : 目标学生最高年级[int] - Option,
             "major" : 目标学生专业[string] - Option,
             "task_expe" : 目标学生任务经验下限[int] - Option, // 此处以学生历史完成的任务数来衡量
             "credit_score" : 目标学生的信誉积分下限[int] - Option,
             "max_participants": 最大参加人数[int] - 必须传输
         }[json-object],
         "task_risk" : 任务未完成扣除积分数[int],
         "task_pay" : 任务薪酬[int],
         "task_time_limit" : 任务最终deadline时间戳，格式为：yyyy-mm-dd:hh-mm[string]

         // 以下是所有接受任务者的完成情况
         // 已接收任务的用户列表
         "accept_users": {
            "accept_user_num": 已接收任务的用户总数[int],
            "accept_user_names": 已接收任务的用户昵称数组[array] // 用户昵称[string]构成的数组
            "accept_user_id": 已接收任务的用户id数组[array]
         }[json-object],
         //已完成任务的用户列表
         "finish_users": {
            "finish_user_num": 已完成任务的用户总数[int],
            "finish_user_names": 已完成任务的用户昵称数组[array] // 用户昵称[string]构成的数组
            "finish_user_id": 已完成任务的用户id数组[array]
         }[json-object],
         //此处闲置交易任务，取寄快递任务不需要额外的任务信息，但问卷调查需要，具体获取用户已填好的问卷内容
         //，须根据不同参与者，调用额外的接口进行获取
     }
     ```

4. 查看某一学生用户的问卷调查任务完成情况

    * 接口地址：服务器地址/check_question_naire

    * 请求参数：

    ```js
    {
        "mid": 对应的问卷调查任务id[int],
        "userid": 作为查看目标的大学生id[int],
        "poster_id": 作为任务发起人的当前用户id[string] // 规定只有任务发起人才能看问卷调查结果
    }
    ```

    * 返回格式：

    ```js
    {
        "code": boolean, // ture for success, false for failed
        "err_message": string,
        "answers": [
            {
                "order": 问题序号[int],
                "q_type": 问题类型[int], 0 为填空题, 1 为单选题， 2 为多选题
                "content": 问题内容[string],
                "answer": 问题填空题回答[string] - Option, 
                "choices": 问题选择题回答[array] - Option,
                // 选择题的答案形式为：[0, 1, ...]
            }[json-object],
            ...
        ][array]
    }
    ```


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

> 奶牛用户和学生用户认证功能

1. 用户认证功能

	* 接口地址：服务器地址/user_verify

	* 请求参数：

	```js
	{
		"image_data": 用户拍照上传的证件照片数据流，需要转成base64编码字符串[base64-string], // 学生上传校园卡照片，奶牛上传名片
        "verify_mode": 认证模式[boolean], // 认证模式，false为奶牛认证，true为学生认证
        "user_id": 用户证件上的ID信息[string], // 学生认证时上传学号，奶牛认证时上传真实名字
        "organization": 用户所属组织名称[string] // 学生认证时上传学校名称，奶牛认证时上传组织机构名称
	}
	```

	* 返回格式：

	```js
	{
		"code": boolean,
		"err_message": string
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

3. 获取当前资金功能

    * 接口地址：服务器地址/get_balance

    * 请求参数：

    ```js
    {
        "userid" : 当前大学生用户id[string]
    }
    ```

    * 返回格式：

    * 返回格式：

     ```js
     {
         "code" : boolean, // false or true
         "balance" : int, // 用户账户余额
         "err_message" : string
     }
     ```

## Part One. 奶牛 APIs

> 注册部分

1. 奶牛注册功能

   * 接口地址：服务器地址/cow_logup

   * 请求参数：

     ```js
     {
     	 "username" : 奶牛用户昵称，可以考虑使用微信昵称[string],
         "userid" : 奶牛用户id[string], // 用户的唯一标识
         "wechat_ok" : 是否通过认证[boolean] // 使用verify接口认证，false or true
         "email" : 奶牛用户邮箱[string],
         "phone" : 奶牛用户手机号码[string],
         "infos" : 奶牛组织的相关介绍信息[string],
         "organization" : 奶牛用户机构名称[string]
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

> 个人信息部分

1. 获取奶牛用户个人信息功能

    * 接口地址：服务器地址/get_cow_info

    * 请求参数：

    ```js
    {
        "userid": 当前用户id[string]
    }
    ```


    * 返回格式：

    ```js
    {
        "code": boolean,
        "err_message": string,

        "username": 用户昵称[string],
        "email": 用户邮箱[string],
        "phone": 用户电话[string],
        "infos": 用户个人介绍[string],
        "organization": 用户组织[string]
    }
    ```


2. 编辑奶牛用户个人信息功能(注意：组织认证完成不得更改用户组织的信息)

    * 接口地址：服务器地址/edit_cow_info

    * 请求参数：

    ```js
    {
        "userid": 当前用户id[string],
        "new_email": 新邮箱，[string]
        "new_phone": 新电话，[string]
        "new_infos": 新个人介绍[string]
    }
    ```

    * 返回格式：

    ```js
    {
        "code": boolean,
        "err_message": string,
    }
    ```


## Part Two. 大学生 APIs

> 注册部分

1. 大学生注册功能

   * 接口地址：服务器地址/stu_logup

   * 请求参数：

     ```js
     {
     	 "username" : 大学生用户昵称，可以考虑设置为微信昵称[string], 
         "userid" : 大学生用户id[string], // 用户的唯一标识
         "wechat_ok" : 是否通过认证[boolean] // 使用verify接口认证，false or true
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

> 个人信息部分

1. 获取学生用户个人信息功能

    * 接口地址：服务器地址/get_cow_info

    * 请求参数：

    ```js
    {
        "userid": 当前用户id[string]
    }
    ```


    * 返回格式：

    ```js
    {
        "code": boolean,
        "err_message": string,

        "username": 用户昵称[string],
        "email": 用户邮箱[string],
        "phone": 用户电话[string],
        "infos": 用户个人介绍[string],
        "school_name": 用户组织[string],
        "student_id": 用户学生id[string],
        "major": 用户专业[string],
        "year": 用户年级[int]
    }
    ```


2. 编辑学生用户个人信息功能(注意：学生认证完成不得更改学生认证信息)

    * 接口地址：服务器地址/edit_cow_info

    * 请求参数：

    ```js
    {
        "userid": 当前用户id[string],
        "new_email": 用户邮箱[string],
        "new_phone": 用户电话[string],
        "new_infos": 用户个人介绍[string],
        "new_major": 用户专业[string],
        "new_year": 用户年级[int]
    }
    ```

    * 返回格式：

    ```js
    {
        "code": boolean,
        "err_message": string,
    }
    ```


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




> 接受与提交任务部分

1. 接受任务功能

   * 接口地址：服务器地址/receive_task

   * 请求参数：

     ```js
     {
         "userid" : 当前大学生用户id[string],
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


2. 提交任务功能

   * 接口地址：服务器地址/submit_task

   * 请求参数：

     ```js
     {
         "userid" : 当前大学生用户名称,
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