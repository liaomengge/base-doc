### Nepxion整理

1. 解析xml/json（配置中心/本地配置）

   - registry（注册）

     - 服务注册黑白名单
     - 限制单服务注册最大实例数
   - discovery（发现）

     - 服务发现黑白名单
     - 版本（consumer/provider对应的版本路由策略）
     - 区域（consumer/provider对应的区域路由策略）
     - 权重（包含consumer/provider对应的版本和区域的路由策略）- 在custom route使用
   - strategy（全局缺失策略）

     - 版本
     - 区域
     - 地址
     - 版本权重
     - 区域权重

   - strategy-release（自定义策略）

     - 蓝绿
       1. 版本
       2. 区域
       3. 地址
       4. 版本权重
       5. 区域权重
     - 灰度
       1. 版本权重
       2. 区域权重
       3. 地址权重
   - 无损下线 - 只是做了路由过滤，provider服务下线写到配置中心还需要自己实现

     - 基于ip:port - vm
     - 基于uuid - docker/k8s
   - 组件灰度（数据库，mq等）

     - 参数匹配

   整个流程：

   Rule流程：

   1. 配置中心配置group+serviceId(局部)或者group+group(全局)，查询不到，读本地配置文件，全局和局部规则合并统一

   服务查找流程：

   1. NacosServerListDecorator filter服务列表（主要是过滤配置在`<discovery>`元素节点下的黑白名单，版本，区域）

   2. DiscoveryEnabledBasePredicate过滤服务（核心流程见DefaultDiscoveryEnabledAdapter处理）

      > - id和address黑名单主要用于无损下线，见元素节点`<strategy-blacklist>`
      > - 版本，区域，地址过滤流程如下：
      >   1. 优先级：蓝绿[`<strategy-release>`] > 灰度[`<strategy-release>`] > 缺省[`<strategy>`]
      >   2. `<strategy-release>`元素节点下，分别找到蓝绿和灰度的路由route信息

   3. PredicateBasedRuleDecorator选择服务实例（核心见choose流程）

      > - 查找`strategy-release`元素节点下，蓝绿`<conditions type="blue-green">`元素下的version-weight-id或region-weight-id
      > - 查找全局缺省`strategy`元素节点下的`<version-weight>`或`<region-weight>`
      > - 如果以上2点没有配置，则查询`discovery`元素节点下的`<weight>`子节点

> **注：**
>
> 1. 支持header、params、cookie条件配置 ~
> 2. 外部传参和内置参数优先级

2. 配置中心（支持常用的开源配置中心）

   - 数据动态刷新通知，并通过事件总线发布相应事件（部分是动态配置并更刷新，部分对应其他事件）
   - 支持OpenApi操作

3. 注册中心（支持常用的开源注册中心）

   - 配置元数据以及配合上面解析rule规则的registry和route封装

4. 上下线文传递

   - 支持Feign、RestTemplate、WebClient等上下游传递
   - 支持多重异步传递 agent

5. 端点

   - 提供支持端点操作

6. 扩展插件

   - hystrix

   - sentinel

   - skywalking

     ......