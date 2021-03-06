# 测试方案



## 小程序端

使用微信开发者工具中，由微信官方提供的的真机测试系统进行测试（以高页面覆盖率为目标穷举随机测试），生成包括缺陷、性能以及机型覆盖等方面的测试报告。



## 服务端与数据库

> 功能性测试

使用rust语言自带的单元测试、集成测试框架，可直接通过`cargo test [查找名]`来运行测试。运行`cargo test --release`可指定按release版本编译测试，能够测试编译优化后的结果，也可拿来做压力测试。

**单元测试**

在每个文件中创建包含测试函数的`tests`模块，并使用`cfg(test)`标注模块，在与其他部分隔离的环境中测试每一个单元的代码，针对各个用例的源代码编写测试用例。

**集成测试**

创建tests目录，与src目录同级。同其他使用库的代码一样测试库文件。用以测试库的多个部分能否一同正常工作。

**系统测试**

包括随机测试、功能测试、压力测试等，验证单元测试和集成测试的正确性，检查产品的各个功能，并测试产品的健壮性、安全性、可维护性等。初步可使用rust编写高强度的调用，以对调用率高的数据库api进行压力测试，后期可采用测试工具进行测试。



 