### 单体应用

1. 单一代码库 -> 难以维护，陈旧技术与架构很难替换
2. 统一部署 -> 耗时

### 微服务

1. 独立服务 -> 独立架构 -> 不限语言，数据库
           -> 独立部署
2. api 网关 -> 负载均衡
            -> 服务发现 -> 服务注册表 -> 第三方注册服务 (如 consul 生成配置模板，自动替换 nginx 文件)

总结:

1. 微服务 = 多个单体应用 + api 网关