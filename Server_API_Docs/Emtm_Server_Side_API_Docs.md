# Emtm Server APIs



### 统一请求参数格式："Content-Type:application/json"


## Part Zero. 通用 APIs
1. 登录功能(公用功能)

   * 接口地址：服务器地址/login

   * 请求方式：POST

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

   * 接口地址：服务器地址/task/release

   * 请求方式：POST

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

    * 问卷调查 - 接口地址：服务器地址/task/release-question

    * 请求方式：POST

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

    * 闲置交易 - 接口地址：服务器地址/task/release-transaction

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

    * 取寄快递 - 接口地址：服务器地址/task/release-errand

    * 请求参数：

    ```js
    {
        "mid": 上一步新建的任务ID[int],
        "pickup_address": 取寄快递地址[string], (接收任务者拿取快递的地点)
        "deliver_address": 发起任务者的所在地址[string], (接受任务者在拿取快递之后，须到该地址递交快递)
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

    * 接口地址：服务器地址/task/type

    * 请求方式：GET

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
                "poster_id": 任务发布者的唯一标识id[int],
                "poster_name": 任务发布者的用户昵称[string],
                "task_state" : 当前任务的状态[boolean], // true代表任务已截止，false代表任务进行中
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


0. 获取数据库内最新的前N个任务简述信息

	* 接口地址：服务器地址/task/top

    * 请求方式：GET

	* 请求参数：

	```js
	{
		"number": 需要获取的任务数量[int]
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
				"poster_id": 任务发布者的唯一标识id[int],
				"poster_name": 任务发布者的用户昵称[string],
				"task_state" : 当前任务的状态[boolean], // true代表任务已截止，false代表任务进行中
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



1. 查看自己接收的任务(用于任务列表显示，返回的都是简述)

	* 接口地址：服务器地址/task/self-receive

    * 请求方式：GET

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
				"poster_id": 任务发布者的唯一标识id[int],
				"poster_name": 任务发布者的用户昵称[string],
				"task_state" : 当前任务的状态[boolean], // true代表任务已截止，false代表任务进行中
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

	* 接口地址：服务器地址/task/self-release

    * 请求方式：GET

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
				"poster_id": 任务发布者的唯一标识id[int],
				"poster_name": 任务发布者的用户昵称[string],
				"task_state" : 当前任务的状态[boolean], // true代表任务已截止，false代表任务进行中
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

	* 接口地址：服务器地址/task/specific

    * 请求方式：GET

   * 请求参数：

     ```js
     {
         "userid" : 当前用户id[string],
         "poster_id" : 发布任务者数据库id[int],
         "task_mid" : 目标任务的数据库id[int]
     }
     ```

   * 返回格式：

     ```js
     {
         "code" : boolean, // false or true
         "err_message" : string,
         "mid": 任务id[int],
         "poster_id": 任务发布者id[int],
         "task_state" : boolean, // true代表任务已截止，false代表任务进行中
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
         // 问卷调查需要，具体获取用户已填好的问卷内容, 须根据不同参与者，调用额外的接口进行获取
         // 闲置交易任务，取寄快递任务，只需要查看固定的额外任务需求即可
     }
     ```

4. 查看特定问卷调查任务的问卷内容

    * 接口地址：服务器地址/task/question-naire

    * 请求方式：GET

    * 请求参数：

    ```js
    {
        "task_mid": 对应的问卷调查任务id[int],
        "userid": 当前用户微信id[string],
        "poster_id": 该问卷调查任务的发起人id[int]
    }
    ```

    * 返回格式：

    ```js
    {
        "code": boolean, // ture for success, false for failed
        "err_message": string,
        "questions": [
            {
                "order": 当前题目序号[int],
                "q_type": 题目类型[int], // 0 为填空题, 1 为单选题， 2 为多选题
                "content": 题目内容[string],
                "choices": 题目提供的选项[array] - Option // 填空题没有选项，选择题的选项按顺序放置
            }[json-object],
            ...
        ]
    ```


4. 查看某一学生用户的问卷调查任务完成情况

    * 接口地址：服务器地址/task/question-naire-answer

    * 请求方式：GET

    * 请求参数：

    ```js
    {
        "task_mid": 对应的问卷调查任务id[int],
        "userid": 当前用户微信id[string], // 规定只有任务发起人才能看问卷调查的详细结果
        "student_id": 查看的目标学生的数据库id[int] 
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


5. 查看闲置交易任务详情

	* 接口地址：服务器地址/task/transaction

    * 请求方式：GET

	* 请求参数：

	```js
	{
		"task_mid": 任务mid[int],
		"userid": 当前查看该任务的用户微信id[string],
		"poster_id": 作为任务发起人的当前用户数据库id[int]
	}
	```

	* 返回格式：

	```js
	{
		"code": boolean,
		"err_message": string,
		"t_type": 交易商品类型[string],
        "info": 交易商品信息描述[string],
        "loss": 交易商品的损耗情况[int] // 0~5作为损耗范围，其中0为无损耗，5为完全损耗
        "address": 交易商品的地点[string]
	}
	```


6. 查看取寄快递任务详情

	* 接口地址：服务器地址/task/errand

    * 请求方式：GET

	* 请求参数：

	```js
	{
		"task_mid": 任务mid[int],
		"userid": 当前查看该任务的用户微信id[string],
		"poster_id": 作为任务发起人的当前用户数据库id[int]
	}
	```

	* 返回格式：

	```js
	{
		"code": boolean,
		"err_message": string,
		"pickup_address": 取寄快递地址[string], (接收任务者拿取快递的地点)
        "deliver_address": 发起任务者的所在地址[string], (接受任务者在拿取快递之后，须到该地址递交快递)
        "phone_number": 发布任务者的电话号码[string],
        "pick_number": 快递号[string],
        "info": 快递的其余信息[string]
	}
	```



> 获取用户自身的微信id

1. 用户微信ID获取功能

	* 接口地址：服务器地址/wechatid

    * 请求方式：GET

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

	* 接口地址：服务器地址/user/verify

    * 请求方式：POST

	* 请求参数：

	```js
	{
		"image_data": 用户拍照上传的证件照片数据流，需要转成base64编码字符串[base64-string], // 学生上传校园卡照片，奶牛上传名片
        "verify_mode": 认证模式[boolean], // 认证模式，false为奶牛认证，true为学生认证
        "user_id": 用户证件上的ID信息[string], // 学生认证时上传学号，奶牛认证时上传真实名字
        "organization": 用户所属组织名称[string] // 学生认证时上传学校名称，奶牛认证时上传组织机构名称
        "wechat_id": 当前操作用户的微信openid[string]
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

   * 接口地址：服务器地址/task/search

   * 请求方式：GET

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
                 "mid": 该任务的唯一id[int],
				"poster_id": 任务发布者的唯一标识id[int],
				"poster_name": 任务发布者的用户昵称[string],
				"task_state" : 当前任务的状态[boolean], // true代表任务已截止，false代表任务进行中
		         // 当前任务的所有详细信息字段
		         "task_name" : 任务名称[string],
		         "task_intro" : 任务介绍[string],
		         "task_mode" : 发布任务类型[int],//系统目前提供三种：0.问卷调查，1.闲置交易，2.帮忙取件
		         "task_pay" : 任务薪酬[int],
         		 "task_time_limit" : 任务最终deadline时间戳，格式为：yyyy-mm-dd:hh-mm[string],
                 "score" : 搜索关键词与该任务关联分数，分数越高关联度越高，前端按score顺序展示[float]
             }
             ...
             ...
         ]
     }
     ```


> 资金管理部分

1. 充值功能

   * 接口地址：服务器地址/balance/recharge

   * 请求方式：POST

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

   * 接口地址：服务器地址/balance/withdraw

   * 请求方式：POST

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

    * 接口地址：服务器地址/balance

    * 请求方式：GET

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

   * 接口地址：服务器地址/logup/cow

   * 请求方式：POST

   * 请求参数：

     ```js
     {
     	 "username" : 奶牛用户昵称，可以考虑使用微信昵称[string],
         "userid" : 奶牛用户id[string], // 用户的唯一标识
         "wechat_ok" : 是否通过认证[boolean] // 使用verify接口认证，false or true
         "email" : 奶牛用户邮箱[string],
         "phone" : 奶牛用户手机号码[string],
         "infos" : 奶牛组织的相关介绍信息[string]
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

    * 接口地址：服务器地址/info/cow

    * 请求方式：GET

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
        "verified": 是否已认证[bool],
        "organization": 用户组织[string] - Option选项，如果已经认证则包含，否则为null
    }
    ```


2. 编辑奶牛用户个人信息功能(注意：组织认证完成不得更改用户组织的信息)

    * 接口地址：服务器地址/info/cow/edit

    * 请求方式：POST

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

   * 接口地址：服务器地址/logup/stu

   * 请求方式：POST

   * 请求参数：

     ```js
     {
     	 "username" : 大学生用户昵称，可以考虑设置为微信昵称[string], 
         "userid" : 大学生用户id[string], // 用户的唯一标识
         "wechat_ok" : 是否通过认证[boolean] // 使用verify接口认证，false or true
         "email" : 大学生用户邮箱[string],
         "phone" : 大学生用户手机号[string],
         "infos" : 大学生兴趣爱好，自我介绍[string],
         "major" : 大学生专业[string],
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

    * 接口地址：服务器地址/info/stu

    * 请求方式：GET

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
        "major": 用户专业[string],
        "year": 用户年级[int],
        "verified": 是否已经认证[bool],
        "school_name": 用户组织[string], // Option - 如果没有认证则为null
        "student_id": 用户学生id[string], // Option - 如果没有认证则为null

        "credit": 学生信用积分[int],
        "accepted": 学生已接收任务数量[int],
        "finished": 学生已完成任务数量[int]
    }
    ```


2. 编辑学生用户个人信息功能(注意：学生认证完成不得更改学生认证信息)

    * 接口地址：服务器地址/info/stu/edit

    * 请求方式：POST

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


> 接受与提交任务部分

1. 接受任务功能

   * 接口地址：服务器地址/task/receive

   * 请求方式：POST

   * 请求参数：

     ```js
     {
         "userid" : 当前大学生用户id[string],
         "poster_id" : 发布任务的目标用户id[int],
         "task_mid" : 目标任务id[int]
     }
     ```

   * 返回格式：

     ```js
     {
         "code" : boolean, // false or true
         "err_message" : string
     }
     ```


1. 奶牛, 大学生用户手动确认学生完成情况(适用于闲置交易、取寄快递任务)

    * 接口地址：服务器地址/task/submit

    * 请求方式：POST

    * 请求参数：

    ```js
    {
        "userid": 当前奶牛用户id[string], //只有是当前任务发布者才可以确认完成
        "student_id": 待确认的学生用户微信id[int],
        "task_mid": 对应的任务id[int]
    }
    ```

    * 返回格式：

    ```js
    {
        "code" : boolean, // false or true
         "err_message" : string
    }
    ```

2. 学生用户提交任务信息(适用于问卷调查任务)

    * 接口地址：服务器地址/task/submit-stu

    * 请求方式：POST

    * 请求参数：

    ```js
    {
        "userid": 当前学生用户id[string],
        "poster_id": 任务发布者的id[int],
        "task_mid": 对应的任务id[int],
        "answers": [
        	// 此处前端必须按order进行数组内部成员的排序，即order与该条项的下标相同
            {
               "order": 问题序号[int],
               // 若为填空题，则将答案放置在answer字段
               "answer": 问题回答[string] - Option,
               // 若为选择题，将答案放置在choices字段，格式为[0, 1, 2, ....]等等
               "choices": 问题选项[array] - Option
            }[json-object],
            ...
        ][array]
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

   * 接口地址：服务器地址/credit

   * 请求方式：GET

   * 请求参数：

     ```js
     {
         "userid" : 当前用户id[string]
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