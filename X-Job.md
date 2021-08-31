### xxl-job总结

1. ##### xxl-job优化

   > - 每个logId新增一个日志文件 - 使用ELK上日志替代
   > - kill logId会导致整个队列清空 - 只kill单个
   > - ring thread轮询耗cpu - wait/notify处理
   > - 秒级拉入拉出
   > - 告警支持掌控，新增超时，无可用机器，异常等告警
   > - 用户管理对接ldap以及权限管理对接devops
   > - 开放OpenApi重新设计
   > - 优化`重试`和`重跑`逻辑
   > - 支持子环境和灰度
   > - 新增参数metadata设计
   > - 修复压测场景下，调度器OOM情况下，RegistryThread退出while，不再注册（恢复后也不再注册）
   > - 修复压测场景下，线程池拒绝RejectedExecutionException，但是有部分任务提交给线程池了，导致部分任务重复执行（后面又轮询扫描）
   > - 压测issue优化（logger.debug）
   > - github其他上问题修复以及性能优化
   > - 参数调优&代码优化
   > - ......

2. ##### xxl-job压测情况（3台1C4G，Myql5.7）

   > - 同一服务，在同一时刻，同时跑20w任务 - cpu 60%，Mem 60%
   >
   >   任务无丢失，最后一条实际触发时间(trigger_time)比理想触发的时间(corn_trigger_time)晚大约7分钟（也就是执行完所有任务，无丢失，大约需要7分钟完成）
   >
   > - 同一服务，每20s执行，每次执行1w条任务（一直稳定运行1天）- cpu 10%
   >
   >   任务无丢失，最后一条实际触发时间(trigger_time)比理想触发的时间(corn_trigger_time)晚大约16秒（也就是每批次执行完所有任务，无丢失，大约需要16秒完成）
   >
   > - 内存以及CPU情况
   >
   >   内存60%，CPU60%
   >
   >   <img src="https://tva1.sinaimg.cn/large/008i3skNgy1gu03xkzinbj61bc0ce0tx02.jpg" alt="image-20210610151520832"  />
   >
   >   <img src="https://tva1.sinaimg.cn/large/008i3skNgy1gu03xoaef6j60rw0kajs002.jpg" alt="image-20210610151608553"  />
   >
   >   <img src="https://tva1.sinaimg.cn/large/008i3skNgy1gu03yrqin6j60vo0akdgp02.jpg" alt="image-20210610151746948"  />
   >
   >   <img src="https://tva1.sinaimg.cn/large/008i3skNgy1gu03yqvnknj60vw0ao76102.jpg" alt="image-20210610151819209"  />
   >
   >

   ###### 总结：任务无丢失情况下，大约每天调度大约能处理4kw~6kw左右的调度~

3. ##### xxl-job问题分析

   > - 所有的任务都是依赖mysql数据库，尤其**xxl_job_log**表，数据量很大，导致查询插入更新很慢
   > - 很多冗余的多次查询（JobScheduleHelper#start），每批次实际查询的次数
   > - 调度器伪集群模式，实际是通过mysql行锁实现，只有其中一台获取到锁，执行
   > - 时间环RingThread轮询扫描 - wait/notify通知，避免无效的空轮训
   > - JobThread执行任务，超时中断，每次创建和中断线程
   > - ......

4. ##### xxl-job问题分析优化

   > - 定期清理**xxl_job_log**表数据（备份冷数据）
   > - 去掉冗余的查询，部分查询可以走Redis等缓存
   > - 可以引入ZK等，实现真正的调度器集群模式
   > - 在设置超时时间，使用线程池，避免频繁的创建线程
   > - 其他优化点，见上面第一条