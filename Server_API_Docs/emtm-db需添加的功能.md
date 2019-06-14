## Emtm-DB 待添加的功能

1. 提供新接口：
	
	* 可根据不同任务类型，获取对应类型下的所有任务

	* 提供存储问卷调查内容，查询问卷调查内容的数据库接口

	* 提供存储闲置交易内容，查询闲置交易内容的数据库接口

	* 提供存储取寄快递内容，查询取寄快递内容的数据库接口

	* 为Mission数据表添加`task_request`字段，将表述任务要求的字段都包含到`task_request`中：

	```js
	{
		#[derive(Debug, Clone, Eq, PartialEq)]
		pub struct TaskRequest {
			pub min_grade: Option<i32>,
			pub max_grade: Option<i32>,
			pub major: Option<String>,
			pub task_expe: Option<i32>,
			pub credit_score: Option<int>,
			pub max_participants: Option<i32>
		}

		#[derive(Debug, Clone, Eq, PartialEq)]
		pub struct Mission {
		    pub mid: i32,
		    pub poster_uid: i32,
		    pub bounty: i32,
		    pub risk: i32,
		    pub name: String,
		    pub mission_type: MissionType,
		    pub content: String,
		    pub post_time: NaiveDateTime,
		    pub deadline: NaiveDateTime,
		    pub participants: Vec<Participant>,
		    pub task_request: TaskRequest
		}
	}
	```

	* 提供存储，查询对应问卷任务答案的接口


2. 提供新字段：
	
	* add_mission()函数调用成功之后，须返回对应新建任务的mid

