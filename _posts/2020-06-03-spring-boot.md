### 自动装配

- 通过**@Import(AutoConfigurationImportSelector)**,实现配置导入，这里并不是传统的单个配置类装配
- **AutoConfigurationImportSelector**类实现了**ImportSelector**接口，重写了方法`selectImport`，他用于实现选择性批量配置类的装配。
- 通过**Spring**提供的**SpringFactoriesLoader**机制，扫描`classpath`路径下的*META-INF/spring.factores,*读取需要实现自动装配的配置类
- 通过条件筛选的方式，把不符合条件的类移除。

### Spring Boot 启动流程

- 根据classpath里是否存在某个特征类(javax.servlet.Servlet,org.springframework.web.context.ConfigurableWebApplicationContext)来判断是否需要创建一个为Web应用使用的ApplicationContext。
- 使用SpringFactoriesLoader在应用的classpath下的所有META-INF/spring.factories中查找并加载所有可用的ApplicationContextInitializer。
- 使用SpringFactoriesLoader在应用的classpath下的所有META-INF/spring.factories中查找并加载所有可用的ApplicationListener。
- 设置main方法的定义类
- 通过SpringFactoriesLoader加载SpringApplicationRunListeners并实例化
- 配置当前Spring boot 需要的环境变量