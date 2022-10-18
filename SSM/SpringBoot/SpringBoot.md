<h1 align="center" style="color: Navy">SpringBoot</h1>

# SpringBoot自动配置
## 依赖配置自动配置
- 自动配好Tomcat
  ○ 引入Tomcat依赖。
  ○ 配置Tomcat
- 自动配好SpringMVC
    ○ 引入SpringMVC全套组件
    ○ 自动配好SpringMVC常用组件（功能）
- 自动配好Web常见功能，如：字符编码问题
    ○ SpringBoot帮我们配置好了所有web开发的常见场景
- 默认的包结构
  ○ 主程序所在包及其下面的所有子包里面的组件都会被默认扫描进来
  ○ 无需以前的包扫描配置
  ○ 想要改变扫描路径，@SpringBootApplication(scanBasePackages="com.atguigu")，或者@ComponentScan 指定扫描路径
- 各种配置拥有默认值
  ○ 默认配置最终都是映射到某个类上，如：MultipartProperties
  ○ 配置文件的值最终会绑定每个类上，这个类会在容器中创建对象
- 按需加载所有自动配置项
  ○ 非常多的starter
  ○ 引入了哪些场景这个场景的自动配置才会开启
  ○ SpringBoot所有的自动配置功能都在 spring-boot-autoconfigure 包里面

## 组件添加
### Configuration
**Full**模式与**Lite**模式
- 配置类组件之间无依赖关系用Lite模式加速容器启动过程，减少判断
- 配置类组件之间有依赖关系，方法会被调用得到之前单实例组件，用Full模式

### Import
给容器中自动创建出这两个类型的组件、默认组件的名字就是全类名:
```java
@Import({User.class, DBHelper.class})
```

### Conditional
条件装配：满足Conditional指定的条件，则进行组件注入

### 总结
- SpringBoot先加载所有的自动配置类  xxxxxAutoConfiguration
- 每个自动配置类按照条件进行生效，默认都会绑定配置文件指定的值。xxxxProperties里面拿。xxxProperties和配置文件进行了绑定
- 生效的配置类就会给容器中装配很多组件
- 只要容器中有这些组件，相当于这些功能就有了
- 定制化配置
  ○ 用户直接自己@Bean替换底层的组件
  ○ 用户去看这个组件是获取的配置文件什么值就去修改。

`xxxxxAutoConfiguration` ---> 组件  ---> `xxxxProperties`里面拿值  ----> `application.properties`


# 定制化原理
## 定制化常见方式
1. 修改配置文件；
2. xxxxxCustomizer；
3. 编写自定义的配置类   xxxConfiguration；+ @Bean替换、增加容器中默认组件；视图解析器 
4. Web应用 编写一个配置类实现 WebMvcConfigurer 即可定制化web功能；+ @Bean给容器中再扩展一些组件
5. @EnableWebMvc + WebMvcConfigurer —— @Bean  可以全面接管SpringMVC，所有规则全部自己重新配置； 实现定制和扩展功能
     1. WebMvcAutoConfiguration  默认的SpringMVC的自动配置功能类。静态资源、欢迎页.....
     2. 一旦使用 @EnableWebMvc 、。会 @Import(DelegatingWebMvcConfiguration.class)
     3. DelegatingWebMvcConfiguration 的 作用，只保证SpringMVC最基本的使用
        - 把所有系统中的 WebMvcConfigurer 拿过来。所有功能的定制都是这些 WebMvcConfigurer  合起来一起生效
        -  自动配置了一些非常底层的组件。RequestMappingHandlerMapping、这些组件依赖的组件都是从容器中获取
        -  public class DelegatingWebMvcConfiguration extends WebMvcConfigurationSupport
     4. WebMvcAutoConfiguration 里面的配置要能生效 必须  @ConditionalOnMissingBean(WebMvcConfigurationSupport.class)
     5. @EnableWebMvc  导致了 WebMvcAutoConfiguration  没有生效。

## 原理分析套路
场景starter - xxxxAutoConfiguration - 导入xxx组件 - 绑定xxxProperties -- 绑定配置文件项

