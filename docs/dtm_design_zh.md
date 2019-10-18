
## DTM设计思想

在微服务架构下，一次请求调用多个服务，每个服务独享数据库，通过接口隔离，因此，数据的一致性受到很大影响。
两阶段提交（2PC）将会导致性能大幅下降，增加数据库的压力，况且，数据库相对于业务服务来说，难以伸缩。
DTM采用了TCC的分布式事务设计模式，相对于两阶段提交来说，性能更高，不会对数据库的数据加锁。开发人员只需要通过
注解标示分布式事务的参数，无需自己控制事务的逻辑，DTM以服务的形式提供给开发者，降低了开发难度。

### 事务处理流程

![avatar](https://support.huaweicloud.com/devg-servicestage/zh-cn_image_0166738635.png)

步骤说明如下：

步骤1：进入到发起全局事务的方法内时，会先向DTM集群申请注册一个全局事务ID（Global Transaction ID），只有申请成功才可继续后续流程。

步骤2.1：事务发起者将申请到的全局事务ID透传到所调用的事务参与者中。

步骤2.2：事务参与者利用得到的全局事务ID，向DTM集群注册申请一个分支事务ID（Branch Transaction ID），只有申请成功才可继续此事务参与者的后续流程。

步骤2.3：事务参与者完成自身业务逻辑（即完成TCC中的Try阶段）。

步骤2.4：事务参与者将自身业务逻辑结果上传到DTM集群，并示意分支事务结束。

步骤3.1~3.4：与2.1~2.4类似，完成剩余事务参与者的业务逻辑。

步骤4：事务发起者发起TCC二阶段。

步骤5.1：DTM集群根据全局事务ID，找到事务参与者，发起TCC二阶段。

步骤5.2：与5.1类似，完成剩余事务参与者的TCC二阶段。

步骤6：DTM集群通告事务发起者全局事务结束。

[更多文档](https://support.huaweicloud.com/devg-servicestage/cse_dtm_0002.html)