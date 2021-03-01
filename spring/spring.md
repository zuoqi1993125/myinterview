1.springbean的生命周期

​      我理解不能只是创建单例bean的过程,而应该和上下文的处理过程联系在一起

- (1)preparerefresh 预处理,加载propertiesource,验证必要属性

- (2)获取beanfactory,默认是DefaultListableBeanFactory;

- (3)对beanfactory做预处理,会设置一些默认的属性比如classloader,beanPostProcessor

- (4)postProcessBeanFactory执行不同applicationContext对beanfactory的处理,比如annotationConfigServletwebServerApplicationContext会对scanner设置扫描包名

- (5)invokeBeanFactoryPostProcessors 执行beanfactoryPostProcessor

  这一步执行的时候会生成各种bean定义,甚至是beanFactory的bean,最重要的ConfigurationClassPostProcessor

  它会先执行postProcessBeanDefinitionRegistry获取解析各种configuration类,生成bean定义和工厂方法

  然后执行postProcessBeanFactory,对configuration类做cglib代理

- (6)registerBeanPostProcessors 实例化实现了beanPostProcessor的bean;这些bean处理器就包括了AutowiredAnnotationBeanPostProcessor,CommonAnnotationBeanPostProcessor,AnnotationAspectJAutoProxyCreator处理aop

- (7)initMessageSource

- (8)initApplicationEventMulticaster 初始化事件传播器

- (9)onRefresh为容器初始化一些特殊的bean(由各自容器实现)

- (10)注册监听器

- (11)finishBeanFactoryInitialization 实例化单例bean
	
	- DefaultListableBeanFactory#preInstantiateSingletons 会判断是否延迟初始化,是否是factoryBean
	
	- AbstractBeanFactory#doGetBean 会去一级缓存中获取bean
	
	- DefaultSingletonBeanRegistry#getSingleton 这里的参数ObjectFactory
	
	- AbstractAutowireCapableBeanFactory#createBean
	
	  - 会调用beanpostprocessor的postProcessBeforeInstantiation会获取代理后的bean
	
	  - 会调用beanpostprocessor的postProcessAfterInstantiation
	
	- AbstractAutowireCapableBeanFactory#doCreateBean
	
	  - 静态工厂方法instanceSupplier
	  
	  - 根据是否工厂方法创建实例
	  
	  - 构造函数创建
	  
	  - 存入三级缓存,封装成ObjectFactory
	  
	- AbstractAutowireCapableBeanFactory#populateBean为bean注入属性
	  
	  - InstantiationAwareBeanPostProcessor#postProcessAfterInstantiation 
	  
	  - InstantiationAwareBeanPostProcessor#postProcessProperties 属性注入
	  
	- AbstractAutowireCapableBeanFactory#initializeBean
	  
	  - 注入各种aware
	  - BeanPostProcessor#postProcessBeforeInitialization
	  - 调用initializebean#afterPropertiesSet
	  - 调用initMethod方法
	  - BeanPostProcessor#postProcessAfterInitialization 这里会对aop需要代理的bean做处理
	  
	  
	
- (12)finishRefresh 清理资源缓存(比如asm),初始化LifeCycleProcessor,发布容器以刷新完毕的的事件

## 2. aop

