## P3-利用FNN， DeepFM算法实现电商场景下商品精排

sort流程：request包括-用户id+召回物品id; 返回评分

tf 1.4.0

### 1.  相关模块划分

#### （1）模型平台

* 数据流服务：数据预处理(流式处理) -- 抽取收集数据，划分数据集离线训练；数据分批在线训练
* 离线训练/在线训练平台：离线训练平台 -> 离线模型文件； 在线训练，更新参数服务
* 模型部署平台：由离线模型文件 ；参数/在线服务 提供在线预估服务

#### （2）参数服务器

* 功能：

  * 将规模较大的部分参数存起来，如embedding部分参数
  * 支持分布式模型训练- 参数分布式存储与并行训练

  具体：键值对形式存储参数； 提供给worker参数；整合来自worker的计算结果+**更新para**

* 接口：push pull (worker node vs server node)

* 结构：

  * server group共同维护所有参数更新
  * server node; server manager node维持

* 经典ps-server原型：**ps-lite(待学习)**

### 2. 代码结构

* feature_server

  * feature_gen: 抽取特征保存至redis

* model

  * data_preprocessing-数据流处理

    * 读取数据，划分数据集；
    * 定义hash方法，将所有特征项进行hash(名+值)
    * 转换为tfrecords格式, 并分割为小文件保存

  * 模型训练相关

    * input.py - 定义输入服务InputFn类： 读取tfrecords类型转化为tf.dataset统一格式，分批等

    * model.py 模型定义文件：config; 定义结构+计算loss; setup_graph:建立图框架+梯度更新

    * main.py 模型训练文件:  初始化参数服务，train/test 评估对象，定义模型网络（连接输入和图）；

      写train函数-session.run；写valid函数; 保存模型为checkpoint+在线预估需要的pb格式

    * save_model.py保存模型
      * checkpoint: 网络模型结构，参数，梯度
      * pd: 较小,只保存网络结构和参数
      * PS.可以用tensorboard从pd读取网络结构
    * auc.py：定义评估对象类
    * ps.py 参数服务：定义PS单例模式
    * save_load_model.py

    

    

    