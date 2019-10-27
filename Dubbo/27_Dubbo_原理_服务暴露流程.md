---
title: 27_Dubbo_原理_服务暴露流程
date: 2019-09-22 18:41:12
tags: 
 - Dubbo
categories:
 - Java
 - Dubbo
---

# 27_Dubbo\_原理_服务暴露流程

ServiceBean实现了两个Spring重要的接口，InitializingBean接口以及ApplicationListener接口：

- InitializingBean接口：afterPropertiesSet()方法在属性设置完以后执行。
- ApplicationListener接口：应用监听器，这里监听的事件是ContextRefreshedEvent，即实现ApplicationListener\<ContextRefreshedEvent>，即整个IOC容器刷新完成之后，会回调onApplicationEvent()方法；

也就是说ServiceBean会在Spring容器创建完对象以后调用afterPropertiesSet()方法，还会在IOC启动完成以后调用onApplicationEvent()方法；

![/dev-guide/images/dubbo-export.jpg](27_Dubbo_%E5%8E%9F%E7%90%86_%E6%9C%8D%E5%8A%A1%E6%9A%B4%E9%9C%B2%E6%B5%81%E7%A8%8B/dubbo-export.jpg)

这两个方法具体做了一些什么事？

1. afterPropertiesSet()方法，首先获取Provider，这里就是获取标签里的信息，然后setProvider(providerConfig)方法，把Provider信息塞入到ServiceBean当中去，以及后面的ApplicationConfig、ModuleConfig、RegistryConfig、Monitor、Protocol；
2. onApplicationEvent()方法，当IOC容器刷新了以后，如果不是延迟的，而且未暴露，以及不是不暴露的服务，就执行export()方法将其暴露出去。
3. 这个export()方法其实就是再把标签里相关的信息解析出来，比如timeout属性等，然后调用doExport()方法执行暴露。
4. doExport()方法里也会做一些检查，到最后doExportUrls()，这里就暴露Url地址出去了；
5. doExportUrls()方法里，把注册中心的信息读取出来，读取协议的信息，端口以及协议名，然后再调用doExportUrlsFor1Protocol()方法暴露，并把信息放到注册中心去。
6. doExportUrlsFor1Protocol()方法把服务的信息进行包装，比如方法的返回值，参数等信息，最后用代理工厂获取执行器，把执行者的信息（服务提供者类）以及url地址包装。最后会调用protocol.export()方法，在指定的端口进行暴露；
7. DubboProtocol类的export方法，以及RegistryProtocol类的export方法；
8. 首先是注册中心暴露【RegistryProtocol】，把执行者作为参数传入，在export()方法中调用doLocalExport()方法本地暴露，然后ProviderConsumerRegTable【提供者和消费者的注册表】调用registerProvider()方法，记录提供者的信息，传入提供者、注册中心的地址、以及提供者的Url地址信息，包装成一个暴露器进行暴露。
9. 最后发现RegistryProtocol中还会调用其他Protocol，也就是DubboProtocol，这里面对url地址进行各种转化，包装成DubboExporter，这里会做一件事，openServer()方法打开服务，这里拿到Url地址，如本地的20880地址，然后创建一个ExchangeServer信息交换的服务器对象，然后在ExchangeServer会调用bind方法，传入url信息进行绑定服务器，以及请求处理器的对象，然后就是调用netty的底层了，Transporter以及channel等。让netty去监听我们的20880端口；
10. 然后回到注册表对象【ProviderConsumerRegTable】，里面的两个静态对象，就是我们的注册信息。

