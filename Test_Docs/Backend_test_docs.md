# 后端测试报告



## 服务端





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



