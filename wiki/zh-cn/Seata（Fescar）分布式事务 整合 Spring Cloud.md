# 1.前言
针对Fescar 相信很多开发者已经对他并不陌生，当然Fescar 已经成为了过去时，为什么说它是过去时，因为Fescar 已经华丽的变身为Seata。如果还不知道Seata 的朋友，请登录下面网址查看。

 
  SEATA GITHUB:[https://github.com/seata/seata]

对于阿里各位同学的前仆后继，给我们广大开发者带来很多开源软件，在这里对他们表示真挚的感谢与问候。

今天在这里和大家分享下Spring Cloud 整合Seata 的相关心得。也让更多的朋友在搭建的道路上少走一些弯路，少踩一些坑。

# 2.工程内容

本次搭建流程为：client->网关->服务消费者->服务提供者.

```
                        技术框架：spring cloud gateway

                                spring cloud fegin

                                nacos1.0.RC2

                                fescar-server0.4.1(Seata)
```
关于nacos的启动方式请参考：[Nacos启动参考](https://nacos.io/zh-cn/docs/quick-start.html)       

首先seata支持很多种注册服务方式，在 fescar-server-0.4.1\conf 目录下

```
    file.conf
    logback.xml
    nacos-config.sh
    nacos-config.text
    registry.conf
``` 

总共包含五个文件，其中 file.conf和 registry.conf 分别是我们在 服务消费者 & 服务提供者 代码段需要用到的文件。
注：file.conf和 registry.conf 必须在当前使用的应用程序中，即： 服务消费者 & 服务提供者 两个应用在都需要包含。
    如果你采用了配置中心 是nacos 、zk ，file.cnf 是可以忽略的。但是type=“file” 如果是为file  就必须得用file.cnf

下面是registry.conf 文件中的配置信息，其中 registry 是注册服务中心配置。config为配置中心的配置地方。

从下面可知道目前seata支持nacos，file eureka redis zookeeper 等注册配置方式，默认下载的type=“file” 文件方式，当然这里选用什么方式，取决于

每个人项目的实际情况，这里我选用的是nacos，eureka的也是可以的，我这边分别对这两个版本进行整合测试均可以通过。

注：如果整合eureka请选用官方最新版本。

# 3.核心配置

```java
registry {
  # file 、nacos 、eureka、redis、zk
  type = "nacos"

  nacos {
    serverAddr = "localhost"
    namespace = "public"
    cluster = "default"
  }
  eureka {
    serviceUrl = "http://localhost:1001/eureka"
    application = "default"
    weight = "1"
  }
  redis {
    serverAddr = "localhost:6379"
    db = "0"
  }
  zk {
    cluster = "default"
    serverAddr = "127.0.0.1:2181"
    session.timeout = 6000
    connect.timeout = 2000
  }
  file {
    name = "file.conf"
  }
}

config {
  # file、nacos 、apollo、zk
  type = "nacos"

  nacos {
    serverAddr = "localhost"
    namespace = "public"
    cluster = "default"
  }
  apollo {
    app.id = "fescar-server"
    apollo.meta = "http://192.168.1.204:8801"
  }
  zk {
    serverAddr = "127.0.0.1:2181"
    session.timeout = 6000
    connect.timeout = 2000
  }
  file {
    name = "file.conf"
  }
}
``` 
这里要说明的是nacos-config.sh 是针对采用nacos配置中心的话，需要执行的一些默认初始化针对nacos的脚本。

SEATA的启动方式参考官方： 注意，这里需要说明下，命令启动官方是通过 空格区分参数，所以要注意。这里的IP 是可选参数，因为涉及到DNS解析，在部分情况下，有的时候在注册中心fescar 注入nacos的时候会通过获取地址，如果启动报错注册发现是计算机名称，需要指定IP。或者host配置IP指向。不过这个问题，在最新的SEATA中已经进行了修复。

```shell
sh fescar-server.sh 8091 /home/admin/fescar/data/ IP（可选）
```


上面提到过，在我们的代码中也是需要file.conf 和registry.conf 这里着重的地方要说的是file.conf,file.conf只有当registry中 配置file的时候才会进行加载，如果采用ZK、nacos、作为配置中心，可以忽略。因为type指定其他是不加载file.conf的，但是对应的 service.localRgroup.grouplist  和 service.vgroup_mapping  需要在支持配置中心 进行指定，这样你的client 在启动后会通过自动从配置中心获取对应的 SEATA 服务 和地址。如果不配置会出现无法连接server的错误。当然如果你采用的eureka在config的地方就需要采用type="file" 目前SEATA config暂时不支持eureka的形势

```java
transport {
  # tcp udt unix-domain-socket
  type = "TCP"
  #NIO NATIVE
  server = "NIO"
  #enable heartbeat
  heartbeat = true
  #thread factory for netty
  thread-factory {
    boss-thread-prefix = "NettyBoss"
    worker-thread-prefix = "NettyServerNIOWorker"
    server-executor-thread-prefix = "NettyServerBizHandler"
    share-boss-worker = false
    client-selector-thread-prefix = "NettyClientSelector"
    client-selector-thread-size = 1
    client-worker-thread-prefix = "NettyClientWorkerThread"
    # netty boss thread size,will not be used for UDT
    boss-thread-size = 1
    #auto default pin or 8
    worker-thread-size = 8
  }
}
service {
  #vgroup->rgroup
  vgroup_mapping.service-provider-fescar-service-group = "default"
  #only support single node
  localRgroup.grouplist = "127.0.0.1:8091"
  #degrade current not support
  enableDegrade = false
  #disable
  disable = false
}

client {
  async.commit.buffer.limit = 10000
  lock {
    retry.internal = 10
    retry.times = 30
  }
}
```

# 4.服务相关
 这里有两个地方需要注意

```java
    grouplist IP，这里是当前fescar-sever的IP端口，
    vgroup_mapping的配置。
```
 vgroup_mapping.服务名称-fescar-service-group,这里 要说下服务名称其实是你当前的consumer 或者provider application.properties的配置的应用名称：spring.application.name=service-provider，源代码中是 获取应用名称与 fescar-service-group 进行拼接，做key值。同理value是当前fescar的服务名称，  cluster = "default"  / application = "default"

```java
     vgroup_mapping.service-provider-fescar-service-group = "default"
      #only support single node
      localRgroup.grouplist = "127.0.0.1:8091"
```
同理无论是provider 还是consumer 都需要这两个文件进行配置。

如果你采用nacos做配置中心，需要在nacos通过添加配置方式进行配置添加。
# 5.事务使用
我这里的代码逻辑是请求通过网关进行负载转发到我的consumer上，在consumer 中通过fegin进行provider请求。官方的例子中是通过fegin进行的，而我们这边直接通过网关转发，所以全局事务同官方的demo一样 也都是在controller层。

```java
@RestController
public class DemoController {
	@Autowired
	private DemoFeignClient demoFeignClient;
	
	@Autowired
	private DemoFeignClient2 demoFeignClient2;
	@GlobalTransactional(timeoutMills = 300000, name = "spring-cloud-demo-tx")
	@GetMapping("/getdemo")
	public String demo() {
		
		// 调用A 服务  简单save
		ResponseData<Integer> result = demoFeignClient.insertService("test",1);
		if(result.getStatus()==400) {
			System.out.println(result+"+++++++++++++++++++++++++++++++++++++++");
			throw new RuntimeException("this is error1");
		}
	
		// 调用B 服务。报错测试A 服务回滚
		ResponseData<Integer>  result2 = demoFeignClient2.saveService();
	
		if(result2.getStatus()==400) {
			System.out.println(result2+"+++++++++++++++++++++++++++++++++++++++");
			throw new RuntimeException("this is error2");
		}

		return "SUCCESS";
	}
}
```
到此为止核心的事务整合基本到此结束了，我这里是针对A,B 两个provider进行调用，当B发生报错后，进行全局事务回滚。当然每个事务内部都可以通过自己的独立本地事务去处理自己本地事务方式。

SEATA是通过全局的XID方式进行事务统一标识方式。这里就不列出SEATA需要用的数据库表。具体参考：[spring-cloud-fescar 官方DEMO](https://github.com/spring-cloud-incubator/spring-cloud-alibaba/tree/master/spring-cloud-alibaba-examples/fescar-example)
# 5.数据代理

这里还有一个重要的说明就是，在分库服务的情况下，每一个数据库内都需要有一个undo_log的数据库表进行XID统一存储处理。

同事针对每个提供服务的项目，需要进行数据库连接池的代理。也就是：

目前只支持Druid连接池，后续会继续支持。

```java
@Configuration
public class DatabaseConfiguration {

	
	@Bean(destroyMethod = "close", initMethod = "init")
	@ConfigurationProperties(prefix="spring.datasource")
	public DruidDataSource druidDataSource() {

		return new DruidDataSource();
	}
	
	
	@Bean
	public DataSourceProxy dataSourceProxy(DruidDataSource druidDataSource) {
	
		return new DataSourceProxy(druidDataSource);
	}
	

    @Bean
    public SqlSessionFactory sqlSessionFactory(DataSourceProxy dataSourceProxy) throws Exception {
        SqlSessionFactoryBean factoryBean = new SqlSessionFactoryBean();
        factoryBean.setDataSource(dataSourceProxy);    
        return factoryBean.getObject();
    }
}
```
大家要注意的就是配置文件和数据代理。如果没有进行数据源代理，undo_log是无数据的，也就是没办进行XID的管理。

 
本文作者：大菲.Fei

