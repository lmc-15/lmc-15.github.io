### 自动装配

- 通过**@Import(AutoConfigurationImportSelector)**,实现配置导入，这里并不是传统的单个配置类装配
- **AutoConfigurationImportSelector**类实现了**ImportSelector**接口，重写了方法`selectImport`，他用于实现选择性批量配置类的装配。
- 通过**Spring**提供的**SpringFactoriesLoader**机制，扫描`classpath`路径下的*META-INF/spring.factores,*读取需要实现自动装配的配置类
- 通过条件筛选的方式，把不符合条件的类移除。

### Spring Boot 启动流程

- 调用**SpringApplication**的静态`run`方法
- 通过**SpringFactoriesLoader**查找并加载**SpringApplicationRunListener**，开始启动**Spring Boot**
- 创建当前**Spring Boot** 的一些**Environment**
- 遍历调用所有**SpringApplicationRunListener**的`environmentPrepared()`的方法，告诉他们：“当前**Spring Boot**应用使用的**Environment**准备好了咯！”。
- 创建**ApplicationContext**
- **SpringFactoriesLoader**加载可用的**ApplicationContextInitializer**的`initialize（applicationContext）`对**ApplicationContext**进一步处理。
-  遍历调用所有**SpringApplicationRunListener**的**contextPrepared()**方法。
- 最核心的一步，将之前通过**@EnableAutoConfiguration**获取的所有配置以及其他形式的IoC容器配置加载到已经准备完毕的**ApplicationContext**。
- 遍历调用所有**SpringApplicationRunListener**的`contextLoaded()`方法。
- 调用**ApplicationContext**的`refresh()`方法，完成IoC容器可用的最后一道工序
- 查找当前**ApplicationContext**中是否注册有**CommandLineRunner**，如果有，则遍历执行它们。
- 正常情况下，遍历执行**SpringApplicationRunListener**的`finished()`方法