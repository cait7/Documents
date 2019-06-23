# 后端测试报告



## 服务端

### 测试用例

1. **Cow Account**

   - 1a. 注册

     |       test item        | test result |
     | :--------------------: | :---------: |
     |       微信ID重复       |      ✔      |
     |        邮箱重复        |      ✔      |
     |       手机号重复       |      ✔      |
     |       组织名重复       |      ✔      |
     | 字符长度不符或非法字符 |      ✔      |

   - 1b. 接收任务

     |     test item      | test result |
     | :----------------: | :---------: |
     | 尝试接收（不允许） |      ✔      |

   - 1c. 确认任务完成内容

     |   test item    | test result |
     | :------------: | :---------: |
     |   任务不存在   |      ✔      |
     | 完成者身份不符 |      ✔      |



2. **Student Account**

   - 2a. 注册

     |       test item        | test result |
     | :--------------------: | :---------: |
     |       微信ID重复       |      ✔      |
     |        邮箱重复        |      ✔      |
     |       手机号重复       |      ✔      |
     |        学号重复        |      ✔      |
     | 字符长度不符或非法字符 |      ✔      |

   - 2b. 信用积分检查

     |      test item      | test result |
     | :-----------------: | :---------: |
     |   信息类型：余额    |      ✔      |
     | 信息类型：wechat id |      ✔      |

   - 2c. 接收任务

     |        test item        | test result |
     | :---------------------: | :---------: |
     | 任务类型：questionnaire |      ✔      |
     |  任务类型：transaction  |      ✔      |
     |    任务类型：errand     |      ✔      |

   - 2c. 提交完成内容

     | test item | test result |
     | :-------: | :---------: |
     | 任务相符  |      ✔      |
     | 任务不符  |      ✔      |



3. **common part**

   - 3a. 登录

     |  test item   | test result |
     | :----------: | :---------: |
     | 登录信息正确 |      ✔      |
     | 登录信息错误 |      ✔      |

   - 3b. 修改信息

     |    test item     | test result |
     | :--------------: | :---------: |
     | 修改信息符合要求 |      ✔      |
     | 修改信息不符要求 |      ✔      |

   - 3c. 获取个人简略信息

     |     test item     | test result |
     | :---------------: | :---------: |
     |   账号身份为cow   |      ✔      |
     | 账号身份为student |      ✔      |

   - 3d. 获取个人详细信息

     |      test item      | test result |
     | :-----------------: | :---------: |
     |   信息类型：余额    |      ✔      |
     | 信息类型：wechat id |      ✔      |

   - 3e. 获取任务信息

     |        test item        | test result |
     | :---------------------: | :---------: |
     | 任务类型：questionnaire |      ✔      |
     |  任务类型：transaction  |      ✔      |
     |    任务类型：errand     |      ✔      |

   - 3f. 个人发布任务检查

     |        test item        | test result |
     | :---------------------: | :---------: |
     | 任务类型：questionnaire |      ✔      |
     |  任务类型：transaction  |      ✔      |
     |    任务类型：errand     |      ✔      |

   - 3g. 个人接受任务检查

     |        test item        | test result |
     | :---------------------: | :---------: |
     | 任务类型：questionnaire |      ✔      |
     |  任务类型：transaction  |      ✔      |
     |    任务类型：errand     |      ✔      |

   - 3h. 存款

     |    test item     | test result |
     | :--------------: | :---------: |
     | 存款信息符合要求 |      ✔      |
     | 存款信息不符要求 |      ✔      |

   - 3i. 取款

     |    test item     | test result |
     | :--------------: | :---------: |
     | 取款信息符合要求 |      ✔      |
     | 取款信息不符要求 |      ✔      |



## 数据库

### 测试环境

- mysql5.7
- win10



### 数据库初始设置

安装diesel CLI：

```
cargo install diesel_cli --no-default-features --features mysql
```

修改`.env`文件以配置数据库与用户信息，如：

```
DATABASE_URL=mysql://root:ROOT_PASSWORD@127.0.0.1:9877/EMTM
```

初始化数据库：

```
diesel setup
diesel migration run
```



### 测试项目

对数据库各项存储进行功能测试，分别调用各项controller中的添加数据与获取数据api，测试数据库存储是否正确无误。

在进行每项测试前，通过调用以下代码首先回滚数据库至初始状态：

```rust
let ctrl = Controller::test_new();

ctrl.revert_all();
ctrl.migrate();
```



**user controller**

|   test api   | test result |
| :----------: | :---------: |
|   add_cows   |      ✔      |
| add_students |      ✔      |
| update_users |      ✔      |

测试中使用到的获取数据的api：

- get_user_from_identifier



**school controller**

|    test api     | test result |
| :-------------: | :---------: |
| get_school_name |      ✔      |
|  get_school_id  |      ✔      |



**mission controller**

|      test api      | test result |
| :----------------: | :---------: |
|    add_mission     |      ✔      |
|   update_mission   |      ✔      |
|  add_participants  |      ✔      |
| update_participant |      ✔      |

测试中使用到的获取数据的api：

- get_poster_missions
- get_mission_participants



**errand_controller**

|  test api  | test result |
| :--------: | :---------: |
| add_errand |      ✔      |

测试中使用到的获取数据的api：

- get_errand



**trade_controller**

| test api  | test result |
| :-------: | :---------: |
| add_trade |      ✔      |

测试中使用到的获取数据的api：

- get_trade



**servey_controller**

|   test api    | test result |
| :-----------: | :---------: |
| add_questions |      ✔      |
|  add_answer   |      ✔      |

测试中使用到的获取数据的api：

- get_questionaire
- get_question_answers
- get_student_answered
- get_student_answer



