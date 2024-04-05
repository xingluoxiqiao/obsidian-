# 项目结构
## 枚举和常量
系统级公共常量：用户注册可复用用户名分片数1024
Redis Key 定义常量类
用户注册错误码枚举
用户注册错误码枚举（审核）
用户相关责任链 Mark 枚举：用户注册过滤器（三个实现）
- 用户注册检查证件号是否多次注销UserRegisterCheckDeletionChainHandler
- 用户注册参数必填检验UserRegisterParamNotNullChainHandler
- 用户注册用户名唯一检验UserRegisterHasUsernameChainHandler
## 配置
用户注册布隆过滤器属性配置UserRegisterBloomFilterProperties
布隆过滤器配置RBloomFilterConfiguration
## 控制层
## DAO
用户注销表实体
用户手机号实体对象
用户名复用表实体
用户信息实体
用户邮箱表实体对象
乘车人实体
## DTO
### req
乘车人移除请求参数
乘车人添加&修改请求参数
用户注册请求参数
用户注销请求参数
用户修改请求参数
用户登录请求参数
### resp
用户查询返回无脱敏参数
乘车人返回参数
用户登录返回参数
用户查询返回参数
乘车人真实返回参数，不包含脱敏信息
用户注册返回参数
## 序列化
身份证号脱敏反序列化IdCardDesensitizationSerializer
手机号脱敏反序列化PhoneDesensitizationSerializer
## Service及实现

## 工具类
用户名可复用工具类UserReuseUtil

# 实现细节（按业务）

## 检查用户是否已经存在
先判断布隆过滤器，如果布隆过滤器中已经存在，再判断redis缓存中
布隆过滤器中不存在则一定不存在

## 注册用户
1.注册时前端收集用户信息通过请求体传递给后端
2.责任链模式校验：检查证件号是否多次注销、注册参数必填检验、用户名唯一检验（布隆过滤器）
3.加分布式锁
4.检查用户名重复，手机号重复，邮箱重复
5.复用库删除（注册成功不可复用）
6.布隆过滤器添加
7.返回UserRegisterRespDTO

## 用户登录（token校验）
1.前端传递用户名和密码，其中用户名可能是用户名，邮箱，手机号中的一种
2.判断是不是邮箱（是否含有@），是邮箱走邮箱拿到username，或者走手机号拿到username或者直接从请求体拿到
3.查询userDO，构建userinfodto通过jwt工具类获取token
4.构建UserLoginRespDTO并存入缓存（键token，值UserLoginRespDTO），设置30分钟有效期（后续校验token就是查缓存中有没有对应的token）
5.上述流程走完返回UserLoginRespDTO，否则抛异常

## 退出登录
在缓存中删除对应的token



