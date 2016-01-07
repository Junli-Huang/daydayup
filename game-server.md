# 游戏服务器开发 - 记录/总结

### 技术选择:
- 纯C, 或C/C++混搭(better C), 或纯现代C++开发方式.
- 现代新选择 [[golang]], [[erlang]], [[node.js]](netease: [[pomelo]]) ...
- 成熟框架 [[smart-fox]], RakNet, [[skynet]], [[pomelo]] ...

### 规范:
- 代码规范, 风格, 提前约定.
- 版本管理, [[Git]], ~~SVN 使用分支注意! 上线后相关重大, 平时保持良好习惯~~.
- 注释可文档化, 方便新人了解及旧人交接. 考虑doxygen

### 网络模型:
- 异步网络库, 使用通用C/C++技术, 方便跨平台移植, 如: boost::asio, libuv 等.
- 自实现网络库(win:IOCP  linux:epoll).  或包装 libuv.
- [[golang]] - [CSP(Communicating Sequential Processes)模式](https://en.wikipedia.org/wiki/Communicating_sequential_processes)
- [[erlang]] - [Actor模式](https://en.wikipedia.org/wiki/Actor_model)

### 通讯协议:
- JSON通用简单，但数据包过于庞大. 
  单个登录协议包内容不宜过大, 按内容分批次发送.
- 考虑使用 msgpack, protobuffer, flatbuffer 等成熟通讯库.

### 游戏逻辑:
- 数据缓存服务器 DBServer 过于复杂, 逻辑添加过于繁冗.  
  因排行榜, 擂台挑战等功能, 需要在玩家离线的状态下仍要对其信息进行读取与更新.  
  由于DBServer的原因, 为满足这些功能, 进行了对部分表进行了预加载.  
- 注意数据的划分,  
  方便逻辑异步执行, 便于多线程化
- 模块耦合度注意,   
  降低关联, 减少历史负担, 减少编译时间(C/C++ 头文件依赖)... 
- 提前考虑用户ID设计:  
  为运维需求添加了相同GS下唯一的id, 其功能从代码逻辑来看完全可以替代角色id.

### 资源文件(策划数值excel等...):  
- 手游改动频繁, 尤其是字段的稍有改动牵扯较大. 如何简化?  
- 建议, 资源配置文件类型尽可能简单, 只需: 整型,浮点,字符串. 枚举值按整数对待.  

### 数据库:
- 部分 游戏角色数据库 所承担功能, 考虑同个 GS 下的所有大区公用一个.  
  无需与其他 游戏角色数据库 相配对, 手游频繁更新会造成负担.
- 考虑单个实例数据库连接过多问题.  
  划分好单个 GS 下所承受大区数量, 或按其他方式划分.  
- 数据库表的设计谨慎, 
  尽可能多的进行时间记录(物品获得,装备变更...)以方便逻辑扩展, 以及上线后问题的校对检验.  
  库表的属性列变化, 非必要不做删减, 只做添加, 做到兼容旧表结构.
- 做好行为日志记录, 在数据库表设计的基础上再全面考虑.  
- 尽量不对数据库表的字段进行后续变动, 如必须修改, 确定好影响范围, 以防有存储过程或代码上的遗漏.
- 存储特殊字符的字段, 在C++侧一定要调用转义函数. 
- 考虑使用 [[mongodb]], [[redis]] 等 [[NoSQL]] 

### 日志:
- 详细的日志有助于错误排查, 日志打印等级需要提前注意约定, 慎重考虑.  
- 实时写入file的日志对效率有影响, 引入缓冲队列多线程定时刷入file.  
- 参考或者引入成熟开源日志库,   
  如: log4j, syslog...

### GM:
- 简单且必要的功能需要提前做好准备, 
  如: 封号, 禁言, 踢人...
- 系统功能的管理, 需要仔细考虑.  
  如: 功能的状态, 不仅仅用于关闭, 应该有默认(可能会周期性开启), 关闭, 开启.

### 测试:  
- 引入自动化测试工具, 类似 printf 函数的可变参数对应与否可以提前检测出来.

### 渠道及第三方接入:  
- 通讯方式通用, 简单简洁, 可扩展.  HTTP  
- 接入sdk一定注意账号取只有数字+英文的ID，不要取中文或特殊字符的名字（oppo先例）.  

### 充值:
- App Store 的沙盒测试环境与正式环境有差异.  
  如: 某JSON字段内容测试环境看上去只有 int32 正式环境却需要 int64 

### 运营:
- 做好服务器划分:  
  依照游戏规模大小, 注册人数, 同时在线人数, CPU占用率, 峰值内存消耗 ...  
  单个物理机开多区? 多个物理机开多区? 
- 重要货币的发放设置上限, 以免引发类似钻石与金币发送数目交错的问题.  

### 版本更新:
- 服务器打包需要根据大区类型分类:  (**注意考虑分区以及编号问题**)  
   test_XXXX表示更新技术封测区(100区)的包;  
   android_XXXX表示更新非appstore区(101以上)的包;  
   appstore_XXXX表示更新appstore区(300以上)的包;  
   apptest_XXXX表示更新appstore审核区(112区)的包;  
   global_XXXX表示更新区LS,GS,HTTP的包;  
   misc_XXXX表示更新混服区(204区以上)的包;  
   misc_global_XXXX表示更新混服去LS,GS,HTTP的包;  
   其中XXXX表示服务器版本号,要求是比上一个同类型的包的版本号高.  
- 服务器的更新是用脚本分开更新的    
  区多的时候需要时间比较多. 启动也依然.  
- 配置文件也是每个区一份.  
  配置文件生成工具 /GMScript/config_script(运维用,使用时需要更新最新的Server_Template)
- 数据库的更新:   
  必须保证语句正确   
  有条件需要测试确保语义正确,不然回退很麻烦  
  不要出现drop语句,除非必要  
  对于update,delete等语句需要小心.对于交叉查询避免出现笛卡尔乘积(用left join, join等)  
- 更新的时间尽量预估多一些, 提前通知到相关人员.  
- appstore审核区和正式区由于提审包不能热更新的关系,  
  需要每次提审的包内设备编码和正式区不同(正式区是2,则提审包是1;正式区是1,则提审包是2).  
- 建议下个项目配置文件只保留一个数据库的配置, 其他的配置信息从该数据库的某个表内读出来.  

### 远程连接管理:
- windows server 本地与远程无法复制粘贴:   
  重新创建 rdpclip 进程 (任务管理器中利用管理员特权创建).

### 注意:
- 合区需求，考虑与逻辑的交融（邀请功能）
- 效率优化:  
  战斗,合成 --> 多线程  
- 功能优化:  
  巨型数据包（策划需求膨胀导致）拆分  
  任务系统优化  
  完善的定时器系统  
  带成长的机器人系统 (人数极少区需要, 部分需求可以用其他方式实现, 如:随机虚拟数据...)
- 逻辑优化:  
  极限数量角色限制？
- 维护优化:  
  WS -> DBS 指定玩家信息重加载  
  友情点日志, 方便校对玩家反馈.  
