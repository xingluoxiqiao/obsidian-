**1.实现分布式唯一雪花算法 ID 生成器。**
自定义雪花算法见技术文档
存储机器 ID 的位置默认是通过 Redis 缓存存储，但是当项目中没有 Redis 时，采用随机数方式获取机器 ID
1）定义雪花算法获取机器 ID 模板抽象类。
采用模板方法模式，获取 Redis 或者随机数提供的机器 ID
2）Redis 获取机器 ID 处理器。底层通过 Redis Lua 保障原子性。
脚本返回的 `resultWorkId, resultDataCenterId` 就是雪花算法组成本分的机器 ID 标识部分
3）随机数生成机器 ID 处理器。
整体比较简单，调用 Random 随机函数获取两个值。

**2.封装分布式唯一雪花算法 ID 工具类。**
对象属性 `SNOWFLAKE` 是在 `AbstractWorkIdChooseTemplate` 获取到机器 ID 标识位后初始化的。