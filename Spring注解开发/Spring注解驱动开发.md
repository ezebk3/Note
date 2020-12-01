# Spring注解驱动开发

![](Spring%E6%B3%A8%E8%A7%A3%E9%A9%B1%E5%8A%A8%E5%BC%80%E5%8F%91.assets/OG-Spring.png)

本文整理于csdn博主 [让优秀成为你的习惯](https://blog.csdn.net/weixin_37778801)的笔记

[来源](https://blog.csdn.net/weixin_37778801/)

## IOC

### 组件注册

#### @Configuration&@Bean给容器类注册组件

创建一个Maven项目：spring-annotation

导入 spring-context jar包 – 这个就是Spring核心环境所有依赖的jar包

------

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.ldc</groupId>
    <artifactId>spring-annotation</artifactId>
    <version>1.0-SNAPSHOT</version>

    <dependencies>
        <!-- https://mvnrepository.com/artifact/org.springframework/spring-context -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>4.3.12.RELEASE</version>
        </dependency>
    </dependencies>

</project>
```

------

- **xml文件配置的方式**
  先按照我们以前配置的方式来使用Spring：
  首先有一个Person类：

```java
public class Person {
    private String name;
    private Integer age;

    public Person() {
    }

    public Person(String name, Integer age) {
        this.name = name;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return "Person{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}

```

我们再写上一个Spring的xml配置文件：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	   xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

	<bean id="person" class="com.ldc.bean.Person">
		<property name="age" value="18"></property>
		<property name="name" value="张三"></property>
	</bean>
</beans>
```

测试类来测试：

```java
public class MainTest {
    public static void main(String[]args){
        ApplicationContext applicationContext = new ClassPathXmlApplicationContext("beans.xml");
        Person person = (Person) applicationContext.getBean("person");
        System.out.println(person);
    }
}
```

输出结果为：

> Person{name=‘张三’, age=18}

------

- **注解的方式**：

首先我们先写一个配置类：等同于xml配置文件

```java
/**
 * @Description 配置类就等同以前的配置文件
 */
@Configuration //告诉Spring这是一个配置类
public class MainConfig {

    //相当于xml配置文件中的<bean>标签，告诉容器注册一个bean
    //之前xml文件中<bean>标签有bean的class类型，那么现在注解方式的类型当然也就是返回值的类型
    //之前xml文件中<bean>标签有bean的id，现在注解的方式默认用的是方法名来作为bean的id
    @Bean
    public Person person() {
        return new Person("lisi",20);
    }

}
```

现在，我们来测试一下：

```java
public class MainTest {
    public static void main(String[]args){
        /**
         * 这里是new了一个AnnotationConfigApplicationContext对象，以前new的ClassPathXmlApplicationContext对象
         * 的构造函数里面传的是配置文件的位置，而现在AnnotationConfigApplicationContext对象的构造函数里面传的是
         * 配置类的类型
         */
        ApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfig.class);
        Person person = applicationContext.getBean(Person.class);
        System.out.println(person);
    }
}
```

测试结果如下：

> Person{name=‘lisi’, age=20}

------

我们也可以通过ApplicationContext的一些方法来获取容器里面bean的一些信息，

比如我们可以获取Person这个bean在IOC容器里面的名字，

也是相当于是xml配置文件里面标签里面的id属性；

```java
public class MainTest {
    public static void main(String[]args){
        /**
         * 这里是new了一个AnnotationConfigApplicationContext对象，以前new的ClassPathXmlApplicationContext对象
         * 的构造函数里面传的是配置文件的位置，而现在AnnotationConfigApplicationContext对象的构造函数里面传的是
         * 配置类的类型
         */
        ApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfig.class);
        Person person = applicationContext.getBean(Person.class);
        System.out.println(person);

        //我们可以来获取bean的定义信息
        String[] namesForType = applicationContext.getBeanNamesForType(Person.class);
        for (String name : namesForType) {
            System.out.println(name);
        }
    }
}
```

测试结果如下：

> Person{name=‘lisi’, age=20}
> person

------

**Spring注解的方式默认是以配置的方法名来作为这个bean的默认id，如果我们不想要方法名来作为bean的id，我们可以在`@Bean`这个注解的value属性来进行指定：**

```java
@Configuration //告诉Spring这是一个配置类
public class MainConfig {

    //相当于xml配置文件中的<bean>标签，告诉容器注册一个bean
    //之前xml文件中<bean>标签有bean的class类型，那么现在注解方式的类型当然也就是返回值的类型
    //之前xml文件中<bean>标签有bean的id，现在注解的方式默认用的是方法名来作为bean的id
    @Bean(value = "person")//通过这个value属性可以指定bean在IOC容器的id
    public Person person01() {
        return new Person("lisi",20);
    }

}
```

我们再来运行这个测试类：

```java
public class MainTest {
    public static void main(String[]args){
        /**
         * 这里是new了一个AnnotationConfigApplicationContext对象，以前new的ClassPathXmlApplicationContext对象
         * 的构造函数里面传的是配置文件的位置，而现在AnnotationConfigApplicationContext对象的构造函数里面传的是
         * 配置类的类型
         */
        ApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfig.class);
        Person person = applicationContext.getBean(Person.class);
        System.out.println(person);

        //我们可以来获取bean的定义信息
        String[] namesForType = applicationContext.getBeanNamesForType(Person.class);
        for (String name : namesForType) {
            System.out.println(name);
        }
    }
}
```

那么现在的测试结果如下：bean在IOC容器的名字就是@Bean这个注解的value属性的值，而不是默认的id是方法名person01

> Person{name=‘lisi’, age=20}
> person

#### @ComponentScan-自动扫描组件&指定扫描规则 

在xml文件配置的方式，我们可以这样来进行配置：

```xml
    <!-- 包扫描、只要标注了@Controller、@Service、@Repository，@Component -->
    <context:component-scan base-package="com.ldc"/>
```

------

以前是在xml配置文件里面写包扫描，现在我们可以在配置类里面写包扫描：

```java
/**
 * @Description 配置类就等同以前的配置文件
 */
@Configuration //告诉Spring这是一个配置类
@ComponentScan(value = "com.ldc")//相当于是xml配置文件里面的<context:component-scan base-package="com.ldc"/>
public class MainConfig {

    //相当于xml配置文件中的<bean>标签，告诉容器注册一个bean
    //之前xml文件中<bean>标签有bean的class类型，那么现在注解方式的类型当然也就是返回值的类型
    //之前xml文件中<bean>标签有bean的id，现在注解的方式默认用的是方法名来作为bean的id
    @Bean(value = "person")//通过这个value属性可以指定bean在IOC容器的id
    public Person person01() {
        return new Person("lisi",20);
    }

}
```

------

我们创建BookController、BookService、BookDao这几个类，分别添加了`@Controller`、`@Service`、`@Repository`注解：

```java
@Controller
public class BookController {
}

@Service
public class BookService {
}

@Repository
public class BookDao {
}

```

我们可以引入junit的jar包来进行测试：

```xml
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.12</version>
        <scope>test</scope>
    </dependency>
```

我们来进行单元测试：

```java
    @Test
    public void test01() {
        ApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfig.class);
        String[] definitionNames = applicationContext.getBeanDefinitionNames();
        for (String name : definitionNames) {
            System.out.println(name);
        }
    }
```

测试结果如下：除开IOC容器自己要装配的一些组件外，还有是我们自己装配的组件

> org.springframework.context.annotation.internalConfigurationAnnotationProcessor
> org.springframework.context.annotation.internalAutowiredAnnotationProcessor
> org.springframework.context.annotation.internalRequiredAnnotationProcessor
> org.springframework.context.annotation.internalCommonAnnotationProcessor
> org.springframework.context.event.internalEventListenerProcessor
> org.springframework.context.event.internalEventListenerFactory
> mainConfig
> bookController
> bookDao
> bookService
> person

从上面的测试结果我们可以发现主配置类 MainConfig 也是IOC容器里面的组件，也被纳入了IOC容器的管理：

```java
/**
 * @Description 配置类就等同以前的配置文件
 */
@Configuration //告诉Spring这是一个配置类
@ComponentScan(value = "com.ldc")//相当于是xml配置文件里面的<context:component-scan base-package="com.ldc"/>
public class MainConfig {

    //相当于xml配置文件中的<bean>标签，告诉容器注册一个bean
    //之前xml文件中<bean>标签有bean的class类型，那么现在注解方式的类型当然也就是返回值的类型
    //之前xml文件中<bean>标签有bean的id，现在注解的方式默认用的是方法名来作为bean的id
    @Bean(value = "person")//通过这个value属性可以指定bean在IOC容器的id
    public Person person01() {
        return new Person("lisi",20);
    }

}
```

我们从`@Configuration` 这个注解点进去就可以发现这个注解上也标注了 `@Component` 的这个注解，也纳入到IOC容器中作为一个组件：

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
public @interface Configuration {

	/**
	 * Explicitly specify the name of the Spring bean definition associated
	 * with this Configuration class. If left unspecified (the common case),
	 * a bean name will be automatically generated.
	 * <p>The custom name applies only if the Configuration class is picked up via
	 * component scanning or supplied directly to a {@link AnnotationConfigApplicationContext}.
	 * If the Configuration class is registered as a traditional XML bean definition,
	 * the name/id of the bean element will take precedence.
	 * @return the specified bean name, if any
	 * @see org.springframework.beans.factory.support.DefaultBeanNameGenerator
	 */
	String value() default "";

}
```

------

我们在 `@ComponentScan` 这个注解上，也是可以指定要排除哪些包或者是只包含哪些包来进行管理：里面传是一个Filter[]数组
![在这里插入图片描述](Spring%E6%B3%A8%E8%A7%A3%E9%A9%B1%E5%8A%A8%E5%BC%80%E5%8F%91.assets/20190110194446276-1606114165821.png)
我们从这个excludeFilters方法点过去，就到了`@Filter`这个注解：

```java
	@Retention(RetentionPolicy.RUNTIME)
	@Target({})
	@interface Filter {


		//这个是要排除的规则：是按注解来进行排除还是按照类来进行排除还是按照正则表达式来来进行排除
		FilterType type() default FilterType.ANNOTATION;

		/**
		 * Alias for {@link #classes}.
		 * @see #classes
		 */
		@AliasFor("classes")
		Class<?>[] value() default {};

		@AliasFor("value")
		Class<?>[] classes() default {};

		String[] pattern() default {};

	}
```

------

这个时候，我们就可以这样来配置：

```java
@Configuration
@ComponentScan(value = "com.ldc",excludeFilters = {
        //这里面是一个@Filter注解数组，FilterType.ANNOTATION表示的排除的规则 ：按照注解的方式来进行排除
        //classes = {Controller.class,Service.class}表示的是标有这些注解的类给排除掉
        @Filter(type = FilterType.ANNOTATION,classes = {Controller.class,Service.class})
})
public class MainConfig {

    @Bean(value = "person")
    public Person person01() {
        return new Person("lisi",20);
    }

}
```

我们再来测试一下：

```java
public class IOCTest {

    @Test
    public void test01() {
        ApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfig.class);
        String[] definitionNames = applicationContext.getBeanDefinitionNames();
        for (String name : definitionNames) {
            System.out.println(name);
        }
    }

}
```

这个时候的测试结果如下：这个时候，bookService、bookController这两个组件就已经被排除掉了，不再被IOC容器给管理：

> org.springframework.context.annotation.internalConfigurationAnnotationProcessor
> org.springframework.context.annotation.internalAutowiredAnnotationProcessor
> org.springframework.context.annotation.internalRequiredAnnotationProcessor
> org.springframework.context.annotation.internalCommonAnnotationProcessor
> org.springframework.context.event.internalEventListenerProcessor
> org.springframework.context.event.internalEventListenerFactory
> mainConfig
> bookDao
> person

------

我们也可以来配置**includeFilters**：指定在扫描的时候，只需要包含哪些组件
在用xml文件配置的方式来进行配置的时候，还要禁用掉默认的配置规则，只包含哪些组件的配置才能生效

> <context:component-scan base-package=“com.ldc” use-default-filters=“false”/>

![在这里插入图片描述](Spring%E6%B3%A8%E8%A7%A3%E9%A9%B1%E5%8A%A8%E5%BC%80%E5%8F%91.assets/20190110203227264-1606114166040.png)

------

这个时候，我们就可以这样来写：

```java
@Configuration
//excludeFilters = Filter[];指定在扫描的时候按照什么规则来排除脑哪些组件
//includeFilters = Filter[];指定在扫描的时候，只需要包含哪些组件
@ComponentScan(value = "com.ldc",includeFilters = {
        //这里面是一个@Filter注解数组，FilterType.ANNOTATION表示的排除的规则 ：按照注解的方式来进行排除
        //classes = {Controller.class}表示的是标有这些注解的类给纳入到IOC容器中
        @Filter(type = FilterType.ANNOTATION, classes = {Controller.class})
},useDefaultFilters = false)
public class MainConfig {

    @Bean(value = "person")
    public Person person01() {
        return new Person("lisi",20);
    }

}
```

测试类：

```java
public class IOCTest {

    @Test
    public void test01() {
        ApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfig.class);
        String[] definitionNames = applicationContext.getBeanDefinitionNames();
        for (String name : definitionNames) {
            System.out.println(name);
        }
    }

}
```

这个时候，测试结果如下：这个时候是按照标有注解来进行包含的，现在就只有一个bookController被纳入到IOC容器进行管理

> org.springframework.context.annotation.internalConfigurationAnnotationProcessor
> org.springframework.context.annotation.internalAutowiredAnnotationProcessor
> org.springframework.context.annotation.internalRequiredAnnotationProcessor
> org.springframework.context.annotation.internalCommonAnnotationProcessor
> org.springframework.context.event.internalEventListenerProcessor
> org.springframework.context.event.internalEventListenerFactory
> mainConfig
> bookController
> person

------

`@ComponentScan`这个注解是可以重复定义的：来指定不同的扫描策略
![在这里插入图片描述](Spring%E6%B3%A8%E8%A7%A3%E9%A9%B1%E5%8A%A8%E5%BC%80%E5%8F%91.assets/2019011020395423-1606114166304.png)

------

我们还可以用 `@ComponentScans`来定义多个扫描规则：里面是`@ComponentScan`规则的数组

```java
@Configuration
//excludeFilters = Filter[];指定在扫描的时候按照什么规则来排除脑哪些组件
//includeFilters = Filter[];指定在扫描的时候，只需要包含哪些组件
@ComponentScans(value = {
    @ComponentScan(value = "com.ldc",includeFilters = {
            //这里面是一个@Filter注解数组，FilterType.ANNOTATION表示的排除的规则 ：按照注解的方式来进行排除
            //classes = {Controller.class}表示的是标有这些注解的类给纳入到IOC容器中
            @Filter(type = FilterType.ANNOTATION, classes = {Controller.class})
    },useDefaultFilters = false),
    @ComponentScan(value = "com.ldc")
})
public class MainConfig {

    @Bean(value = "person")
    public Person person01() {
        return new Person("lisi",20);
    }

}
```

------

也可以直接这样来配置多个`@ComponentScan`注解：**但是这样写的话，就必须要java8及以上的支持**

```java
@Configuration
//excludeFilters = Filter[];指定在扫描的时候按照什么规则来排除脑哪些组件
//includeFilters = Filter[];指定在扫描的时候，只需要包含哪些组件
@ComponentScan(value = "com.ldc",includeFilters = {
        //这里面是一个@Filter注解数组，FilterType.ANNOTATION表示的排除的规则 ：按照注解的方式来进行匹配
        //classes = {Controller.class}表示的是标有这些注解的类给纳入到IOC容器中
        @Filter(type = FilterType.ANNOTATION, classes = {Controller.class})
},useDefaultFilters = false)

@ComponentScan(value = "com.ldc")
public class MainConfig {

    @Bean(value = "person")
    public Person person01() {
        return new Person("lisi",20);
    }

}
```

##### 自定义TypeFilter指定过滤规则 

我们可以来看看有哪几种过滤规则：

```java
public enum FilterType {

	/**
	 * Filter candidates marked with a given annotation.
	 * @see org.springframework.core.type.filter.AnnotationTypeFilter
	 */
	ANNOTATION,

	/**
	 * Filter candidates assignable to a given type.
	 * @see org.springframework.core.type.filter.AssignableTypeFilter
	 */
	ASSIGNABLE_TYPE,

	/**
	 * Filter candidates matching a given AspectJ type pattern expression.
	 * @see org.springframework.core.type.filter.AspectJTypeFilter
	 */
	ASPECTJ,

	/**
	 * Filter candidates matching a given regex pattern.
	 * @see org.springframework.core.type.filter.RegexPatternTypeFilter
	 */
	REGEX,

	/** Filter candidates using a given custom
	 * {@link org.springframework.core.type.filter.TypeFilter} implementation.
	 */
	CUSTOM

}
```

我们可以这样来匹配，来指定不同的匹配规则：

```java
@Configuration
//excludeFilters = Filter[];指定在扫描的时候按照什么规则来排除脑哪些组件
//includeFilters = Filter[];指定在扫描的时候，只需要包含哪些组件
@ComponentScans(value = {
        @ComponentScan(value = "com.ldc",includeFilters = {
                //这里面是一个@Filter注解数组，FilterType.ANNOTATION表示的排除的规则 ：
                //按照注解的方式来进行匹配
                //classes = {Controller.class}表示的是标有这些注解的类给纳入到IOC容器中

                // FilterType.ANNOTATION 按照注解来进行匹配
                // FilterType.ASSIGNABLE_TYPE 按照给定的类型来进行匹配
                @Filter(type = FilterType.ANNOTATION, classes = {Controller.class}),
                //按照给定的类型来进行匹配
                @Filter(type = FilterType.ASSIGNABLE_TYPE, classes = {BookService.class})
        },useDefaultFilters = false)
})

public class MainConfig {

    @Bean(value = "person")
    public Person person01() {
        return new Person("lisi",20);
    }

}
```

测试结果如下：

> org.springframework.context.annotation.internalConfigurationAnnotationProcessor
> org.springframework.context.annotation.internalAutowiredAnnotationProcessor
> org.springframework.context.annotation.internalRequiredAnnotationProcessor
> org.springframework.context.annotation.internalCommonAnnotationProcessor
> org.springframework.context.event.internalEventListenerProcessor
> org.springframework.context.event.internalEventListenerFactory
> mainConfig
> bookController
> bookService
> person

bookService组件又重新的被IOC容器给管理了；

下面的这两种是我们最常用的匹配规则：

> FilterType.ANNOTATION 按照注解来进行匹配
> FilterType.ASSIGNABLE_TYPE 按照给定的类型来进行匹配

------

我们还可以来写上一个 `FilterType.ASPECTJ`表达式来进行匹配，这个不常用；
我们也可以按照正则表达式`FilterType.REGEX`的方式来进行匹配：

------

我们来说说最后一种：自定义匹配规则`FilterType.CUSTOM`

我们可以自己来写一个匹配规则的类：MyTypeFilter，这个类要实现TypeFilter这个接口

```java
public class MyTypeFilter implements TypeFilter {
    /**
     *
     * @param metadataReader  the metadata reader for the target class 读取到当前正在扫描的类的信息
     * @param metadataReaderFactory a factory for obtaining metadata readers 可以获取到其他任何类的信息
     * @return
     * @throws IOException
     */
    @Override
    public boolean match(MetadataReader metadataReader, MetadataReaderFactory metadataReaderFactory) throws IOException {
        //获取到当前类注解的信息
        AnnotationMetadata annotationMetadata = metadataReader.getAnnotationMetadata();
        //获取当前类的资源的信息（比如类的路径）
        Resource resource = metadataReader.getResource();

        //获取到当前正在扫描的类的信息
        ClassMetadata classMetadata = metadataReader.getClassMetadata();
        String className = classMetadata.getClassName();
        System.out.println("通过自定义的匹配规则--->"+className);
        return false;
    }
}
```

这个时候，我们就可以这样来用了：使用`FilterType.CUSTOM`

```java
@Configuration
//excludeFilters = Filter[];指定在扫描的时候按照什么规则来排除脑哪些组件
//includeFilters = Filter[];指定在扫描的时候，只需要包含哪些组件
@ComponentScans(value = {
        @ComponentScan(value = "com.ldc",includeFilters = {
                //自定义匹配的规则
                @Filter(type = FilterType.CUSTOM, classes = {MyTypeFilter.class})
        },useDefaultFilters = false)
})

public class MainConfig {

    @Bean(value = "person")
    public Person person01() {
        return new Person("lisi",20);
    }

}
```

------

现在的测试结果如下:

> 通过自定义的匹配规则—>com.ldc.test.IOCTest
> 通过自定义的匹配规则—>com.ldc.bean.Person
> 通过自定义的匹配规则—>com.ldc.config.MyTypeFilter
> 通过自定义的匹配规则—>com.ldc.controller.BookController
> 通过自定义的匹配规则—>com.ldc.dao.BookDao
> 通过自定义的匹配规则—>com.ldc.MainTest
> 通过自定义的匹配规则—>com.ldc.service.BookService
> org.springframework.context.annotation.internalConfigurationAnnotationProcessor
> org.springframework.context.annotation.internalAutowiredAnnotationProcessor
> org.springframework.context.annotation.internalRequiredAnnotationProcessor
> org.springframework.context.annotation.internalCommonAnnotationProcessor
> org.springframework.context.event.internalEventListenerProcessor
> org.springframework.context.event.internalEventListenerFactory
> mainConfig
> person

------

由于，我的自定义的规则类里面返回的是false，所有一个都没有匹配到；
我们可以这样来修改一下，让clsssName里面包含“er”的组件给匹配到：

```java
public class MyTypeFilter implements TypeFilter {
    /**
     *
     * @param metadataReader  the metadata reader for the target class 读取到当前正在扫描的类的信息
     * @param metadataReaderFactory a factory for obtaining metadata readers 可以获取到其他任何类的信息
     * @return
     * @throws IOException
     */
    @Override
    public boolean match(MetadataReader metadataReader, MetadataReaderFactory metadataReaderFactory) throws IOException {
        //获取到当前类注解的信息
        AnnotationMetadata annotationMetadata = metadataReader.getAnnotationMetadata();
        //获取当前类的资源的信息（比如类的路径）
        Resource resource = metadataReader.getResource();

        //获取到当前正在扫描的类的信息
        ClassMetadata classMetadata = metadataReader.getClassMetadata();
        String className = classMetadata.getClassName();
        System.out.println("通过自定义的匹配规则--->"+className);

        if (className.contains("er")) {
            return true;
        }
        return false;
    }
}
```

> 通过自定义的匹配规则—>com.ldc.test.IOCTest
> 通过自定义的匹配规则—>com.ldc.bean.Person
> 通过自定义的匹配规则—>com.ldc.config.MyTypeFilter
> 通过自定义的匹配规则—>com.ldc.controller.BookController
> 通过自定义的匹配规则—>com.ldc.dao.BookDao
> 通过自定义的匹配规则—>com.ldc.MainTest
> 通过自定义的匹配规则—>com.ldc.service.BookService
> org.springframework.context.annotation.internalConfigurationAnnotationProcessor
> org.springframework.context.annotation.internalAutowiredAnnotationProcessor
> org.springframework.context.annotation.internalRequiredAnnotationProcessor
> org.springframework.context.annotation.internalCommonAnnotationProcessor
> org.springframework.context.event.internalEventListenerProcessor
> org.springframework.context.event.internalEventListenerFactory
> mainConfig
> person
> myTypeFilter
> bookController
> bookService

这个时候，包含“er”的组件就给添加到了IOC容器中了；只要在包扫描里面的包里面的每一个类都会进入到这个自定义的匹配规则进行匹配；

------

#### @Scope-设置组件作用域

 首先有一个配置类： 

```java
@Configuration
public class MainConfig2 {

    //默认是单实例的
    @Bean("person")
    public Person person() {
        return new Person();
    }

}
```

测试方法：

```java
    @Test
    public void test02() {
        ApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfig2.class);
        String[] definitionNames = applicationContext.getBeanDefinitionNames();
        for (String name : definitionNames) {
            System.out.println(name);
        }
        //默认是单实例的
        Person person1 = (Person) applicationContext.getBean("person");
        Person person2 = (Person) applicationContext.getBean("person");
        System.out.println(person1==person2);
    }
```

测试结果如下：

> org.springframework.context.annotation.internalConfigurationAnnotationProcessor
> org.springframework.context.annotation.internalAutowiredAnnotationProcessor
> org.springframework.context.annotation.internalRequiredAnnotationProcessor
> org.springframework.context.annotation.internalCommonAnnotationProcessor
> org.springframework.context.event.internalEventListenerProcessor
> org.springframework.context.event.internalEventListenerFactory
> mainConfig2
> person
> true

说明这个bean的实例是单例的；

------

我们可以用`@Scope`这个注解来指定作用域的范围：这个就相当于在xml文件中配置的`bean`标签里面指定scope=“prototype” 属性；

```java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Scope {

	/**
	 * Alias for {@link #scopeName}.
	 * @see #scopeName
	 */
	@AliasFor("scopeName")
	String value() default "";

	/**
	 * Specifies the name of the scope to use for the annotated component/bean.
	 * <p>Defaults to an empty string ({@code ""}) which implies
	 * {@link ConfigurableBeanFactory#SCOPE_SINGLETON SCOPE_SINGLETON}.
	 * @since 4.2
	 * @see ConfigurableBeanFactory#SCOPE_PROTOTYPE
	 * @see ConfigurableBeanFactory#SCOPE_SINGLETON
	 * @see org.springframework.web.context.WebApplicationContext#SCOPE_REQUEST
	 * @see org.springframework.web.context.WebApplicationContext#SCOPE_SESSION
	 * @see #value
	 */
	@AliasFor("value")
	String scopeName() default "";

	/**
	 * Specifies whether a component should be configured as a scoped proxy
	 * and if so, whether the proxy should be interface-based or subclass-based.
	 * <p>Defaults to {@link ScopedProxyMode#DEFAULT}, which typically indicates
	 * that no scoped proxy should be created unless a different default
	 * has been configured at the component-scan instruction level.
	 * <p>Analogous to {@code <aop:scoped-proxy/>} support in Spring XML.
	 * @see ScopedProxyMode
	 */
	ScopedProxyMode proxyMode() default ScopedProxyMode.DEFAULT;

}
```

从源码的注释上，我们可以知道scopeName可以取下面这些值：

**前两个用的比较多，我们就来看看前面两个可以取的值**

> ConfigurableBeanFactory#SCOPE_PROTOTYPE
> ConfigurableBeanFactory#SCOPE_SINGLETON
> org.springframework.web.context.WebApplicationContext#SCOPE_REQUEST
> org.springframework.web.context.WebApplicationContext#SCOPE_SESSION

我们可以点到ConfigurableBeanFactory接口去看一下：

```java
public interface ConfigurableBeanFactory extends HierarchicalBeanFactory, SingletonBeanRegistry {
    String SCOPE_SINGLETON = "singleton";
    String SCOPE_PROTOTYPE = "prototype";

    void setParentBeanFactory(BeanFactory var1) throws IllegalStateException;

    void setBeanClassLoader(ClassLoader var1);

    ClassLoader getBeanClassLoader();

    void setTempClassLoader(ClassLoader var1);

    ClassLoader getTempClassLoader();

    void setCacheBeanMetadata(boolean var1);

    boolean isCacheBeanMetadata();

    void setBeanExpressionResolver(BeanExpressionResolver var1);

    BeanExpressionResolver getBeanExpressionResolver();

    void setConversionService(ConversionService var1);

    ConversionService getConversionService();

    void addPropertyEditorRegistrar(PropertyEditorRegistrar var1);

    void registerCustomEditor(Class<?> var1, Class<? extends PropertyEditor> var2);

    void copyRegisteredEditorsTo(PropertyEditorRegistry var1);

    void setTypeConverter(TypeConverter var1);

    TypeConverter getTypeConverter();

    void addEmbeddedValueResolver(StringValueResolver var1);

    boolean hasEmbeddedValueResolver();

    String resolveEmbeddedValue(String var1);

    void addBeanPostProcessor(BeanPostProcessor var1);

    int getBeanPostProcessorCount();

    void registerScope(String var1, Scope var2);

    String[] getRegisteredScopeNames();

    Scope getRegisteredScope(String var1);

    AccessControlContext getAccessControlContext();

    void copyConfigurationFrom(ConfigurableBeanFactory var1);

    void registerAlias(String var1, String var2) throws BeanDefinitionStoreException;

    void resolveAliases(StringValueResolver var1);

    BeanDefinition getMergedBeanDefinition(String var1) throws NoSuchBeanDefinitionException;

    boolean isFactoryBean(String var1) throws NoSuchBeanDefinitionException;

    void setCurrentlyInCreation(String var1, boolean var2);

    boolean isCurrentlyInCreation(String var1);

    void registerDependentBean(String var1, String var2);

    String[] getDependentBeans(String var1);

    String[] getDependenciesForBean(String var1);

    void destroyBean(String var1, Object var2);

    void destroyScopedBean(String var1);

    void destroySingletons();
}
```

我们来指定一个多实例的：

```java
@Configuration
public class MainConfig2 {
    //singleton:单实例的
    //prototype:多实例的
    //request:同一次请求创建一个实例
    //session:同一个session创建的一个实例
    @Scope("prototype")
    //默认是单实例的
    @Bean("person")
    public Person person() {
        return new Person();
    }

}
```

现在，我们再来测试一次：

```java
    @Test
    public void test02() {
        ApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfig2.class);
        String[] definitionNames = applicationContext.getBeanDefinitionNames();
        for (String name : definitionNames) {
            System.out.println(name);
        }
        //默认是单实例的
        Person person1 = (Person) applicationContext.getBean("person");
        Person person2 = (Person) applicationContext.getBean("person");
        System.out.println(person1==person2);
    }
```

这个时候的测试结果如下：

> org.springframework.context.annotation.internalConfigurationAnnotationProcessor
> org.springframework.context.annotation.internalAutowiredAnnotationProcessor
> org.springframework.context.annotation.internalRequiredAnnotationProcessor
> org.springframework.context.annotation.internalCommonAnnotationProcessor
> org.springframework.context.event.internalEventListenerProcessor
> org.springframework.context.event.internalEventListenerFactory
> mainConfig2
> person
> false

这个时候，bean的实例就多实例的，每调用一次getBean()方法就会创建一个实例；

------

我们来看看当bean的作用域为单例的时候，它在IOC容器中是何时创建的：

```java
@Configuration
public class MainConfig2 {
    
    @Scope
    @Bean("person")
    public Person person() {
        System.out.println("给IOC容器中添加Person...");
        return new Person();
    }

}
```

首先，我们先启动IOC容器，但是不调用getBean方法来获取Person实例：

```java
    @Test
    public void test02() {
        ApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfig2.class);
    }
```

测试结果如下：

> 给IOC容器中添加Person…

这个时候，我们就可以发现，当作用域为单例的时候，IOC容器在启动的时候，就会将容器中所有作用域为单例的bean的实例给创建出来；以后的每次获取，就直接从IOC容器中来获取，相当于是从map.get()的一个过程；

------

然而，当我们的bean的作用域改成多实例的时候，我们再看看结果：

```java
@Configuration
public class MainConfig2 {

    @Scope("prototype")
    @Bean("person")
    public Person person() {
        System.out.println("给IOC容器中添加Person...");
        return new Person();
    }

}
```

当我们再运行的时候：

```java
    @Test
    public void test02() {
        ApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfig2.class);
    }
```

我们可以发现，控制台没有任何的输出结果；在IOC容器创建的时候，没有去创建这个作用域为多实例的bean；

这个时候，我们来调用getBean()方法来获取一下：

```java
    @Test
    public void test02() {
        ApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfig2.class);
        System.out.println("IOC容器创建完成...");
        Person person = (Person) applicationContext.getBean("person");
    }
```

这个时候，控制台打印了：

> IOC容器创建完成…
> 给IOC容器中添加Person…

同时， 如果我多次获取：

```java
    @Test
    public void test02() {
        ApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfig2.class);
        System.out.println("IOC容器创建完成...");
        Person person = (Person) applicationContext.getBean("person");
        Person person1 = (Person) applicationContext.getBean("person");
    }
```

测试结果如下：

> IOC容器创建完成…
> 给IOC容器中添加Person…
> 给IOC容器中添加Person…

我们可以发现，我们用getBean方法获取几次，就创建几次bean的实例；

也就是说当bean是作用域为多例的时候，IOC容器启动的时候，就不会去创建bean的实例的，而是当我们调用getBean()获取的时候去创建bean的实例；而且每次调用的时候，都会创建bean的实例；



#### @Lazy-bean懒加载

- 懒加载：是专门针对于单实例的bean的
  - 单实例的bean：默认是在容器启动的时候创建对象；
  - 懒加载：容器启动的时候，不创建对象，而是在第一次使用（获取）Bean的时候来创建对象，并进行初始化

------

当我们还没有配置懒加载的时候：作用域为单例的bean，默认是在容器启动的时候创建实例对象

```java
@Configuration
public class MainConfig2 {

    /**
     * 懒加载：是专门针对于单实例的bean的
     *       1.单实例的bean：默认是在容器启动的时候创建对象；
     *       2.懒加载：容器启动的时候，不创建对象，而是在第一次使用（获取）Bean的时候来创建对象，并进行初始化
     * @return
     */
    @Bean("person")
    public Person person() {
        System.out.println("给IOC容器中添加Person...");
        return new Person();
    }

}
```

测试：

```java
    @Test
    public void test02() {
        ApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfig2.class);
        System.out.println("IOC容器创建完成...");
    }
```

测试结果：

> 给IOC容器中添加Person…
> IOC容器创建完成…

从上面的测试结果，我们可以看出：**作用域为单例的bean，默认是在容器启动的时候创建实例对象**

------

而现在，我加上懒加载的注解：`@Lazy`

```java
@Configuration
public class MainConfig2 {

    /**
     * 懒加载：是专门针对于单实例的bean的
     *       1.单实例的bean：默认是在容器启动的时候创建对象；
     *       2.懒加载：容器启动的时候，不创建对象，而是在第一次使用（获取）Bean的时候来创建对象，并进行初始化
     * @return
     */
    @Lazy
    @Bean("person")
    public Person person() {
        System.out.println("给IOC容器中添加Person...");
        return new Person();
    }

}
```

我们再来运行以下的测试方法：

```java
    @Test
    public void test02() {
        ApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfig2.class);
        System.out.println("IOC容器创建完成...");
    }
```

这个时候，运行结果如下：

> IOC容器创建完成…

我们可以看到，这个时候，只有IOC容器启动了，而作用域为单例的bean并没有被创建；

而当我们第一次获取的时候，我们再来看看运行结果：

```java
    @Test
    public void test02() {
        ApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfig2.class);
        System.out.println("IOC容器创建完成...");
        Person person = (Person) applicationContext.getBean("person");
    }
```

运行结果：

> IOC容器创建完成…
> 给IOC容器中添加Person…

从运行结果，我们可以看到，当我们调用了getBean()方法获取的时候，bean的实例对象才会被创建；而且只会被创建一次；作用域还是单实例的！

------

#### @Conditional-按照条件注册bean

现在有两个bean： bill 和 linus ，现在我们想按照操作系统的不同来选择是否在IOC容器里面注册bean：

> 现在下面的两个bean注册到IOC容器是要条件的：
> 1.如果系统是windows，给容器注册(“bill”)
> 1.如果系统是linux，给容器注册(“linus”)

```java
@Configuration
public class MainConfig2 {

    /**
     * @Conditional:是SpringBoot底层大量使用的注解，按照一定的条件来进行判断，满足条件 给容器注册bean
     */

    /**
     *  现在下面的两个bean注册到IOC容器是要条件的：
     *  1.如果系统是windows，给容器注册("bill")
     *  1.如果系统是linux，给容器注册("linus")
     * @return
     */
    @Bean("bill")
    public Person person01() {
        return new Person("Bill Gates",62);
    }

    @Bean("linus")
    public Person person02() {
        return new Person("linus",48);
    }

}
```

我们可以看到这个注解：value值传的是实现了Condition这个接口的类的数组

```java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Conditional {

	/**
	 * All {@link Condition}s that must {@linkplain Condition#matches match}
	 * in order for the component to be registered.
	 */
	Class<? extends Condition>[] value();

}
```

这是我们写的两个条件：

```java
/**
 * 判断操作系统是否为Linux系统
 */
public class LinuxCondition implements Condition {
    /**
     *  ConditionContext : 判断条件能使用的上下文(环境)
     *  AnnotatedTypeMetadata : 注释信息
     * @return
     */
    @Override
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        //判断是否为Linux系统
        //1.能获取到IOC容器里面的BeanFactory
        ConfigurableListableBeanFactory beanFactory = context.getBeanFactory();

        //2.获取类加载器
        ClassLoader classLoader = context.getClassLoader();
        //3.能获取当前环境的信息
        Environment environment = context.getEnvironment();
        //4.获取到bean定义的注册类
        BeanDefinitionRegistry registry = context.getRegistry();

        //获取操作系统
        String property = environment.getProperty("os.name");
        if (property.contains("linux")) {
            return true;
        }
        return false;
    }
}
```

------

```java
/**
 * 判断操作系统是否为Windows系统
 */
public class WindowsCondition implements Condition {
    @Override
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        //3.能获取当前环境的信息
        Environment environment = context.getEnvironment();
        //获取操作系统
        String property = environment.getProperty("os.name");
        if (property.contains("Windows")) {
            return true;
        }
        return false;
    }
}
```

------

当两个条件写好了之后，我们给容器注册bean就可以写了：

```java
@Configuration
public class MainConfig2 {

    /**
     * @Conditional:是SpringBoot底层大量使用的注解，按照一定的条件来进行判断，满足条件 给容器注册bean
     */

    /**
     *  现在下面的两个bean注册到IOC容器是要条件的：
     *  1.如果系统是windows，给容器注册("bill")
     *  1.如果系统是linux，给容器注册("linus")
     * @return
     */
    @Conditional({WindowsCondition.class})
    @Bean("bill")
    public Person person01() {
        return new Person("Bill Gates",62);
    }

    @Conditional({LinuxCondition.class})
    @Bean("linus")
    public Person person02() {
        return new Person("linus",48);
    }

}
```

这个时候，我们就可以来测试一下：因为当前使用的环境是Windows 7 系统，所以只会出现bill这一个bean会被注册到IOC容器当中：

```java
@Test
    public void test03() {
        ApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfig2.class);

        //我们可以获取当前的操作系统是什么：
        Environment environment = applicationContext.getEnvironment();
        //动态的获取环境变量的值：Windows 7
        String property = environment.getProperty("os.name");
        System.out.println(property);

        //获取IOC容器类型为Person类型的bean的名字一共有哪些
        String[] definitionNames = applicationContext.getBeanNamesForType(Person.class);
        for (String name : definitionNames) {
            System.out.println(name);
        }
        //我们也可以来获取类型为Person类型的对象
        Map<String, Person> persons = applicationContext.getBeansOfType(Person.class);
        persons.values().forEach(System.out::println);
    }
```

测试结果如下：

> Windows 7
> bill
> Person{name=‘Bill Gates’, age=62}

------

而对于Linux系统，我们也可以在运行环境参数里面进行模拟：
![在这里插入图片描述](Spring%E6%B3%A8%E8%A7%A3%E9%A9%B1%E5%8A%A8%E5%BC%80%E5%8F%91.assets/20190112144729279.png)

------

这个时候，我们再来运行测试类方法，结果如下：

> linux
> linus
> Person{name=‘linus’, age=48}

实际上，在这个条件类里面，我们还可以做很多的事情：比如说可以判断容器中bean的注册情况，也可以给容器中注册bean

```java
/**
 * 判断操作系统是否为Linux系统
 */
public class LinuxCondition implements Condition {
    /**
     *  ConditionContext : 判断条件能使用的上下文(环境)
     *  AnnotatedTypeMetadata : 注释信息
     * @return
     */
    @Override
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        //判断是否为Linux系统
        //1.能获取到IOC容器里面的BeanFactory
        ConfigurableListableBeanFactory beanFactory = context.getBeanFactory();

        //2.获取类加载器
        ClassLoader classLoader = context.getClassLoader();
        //3.能获取当前环境的信息
        Environment environment = context.getEnvironment();
        //4.获取到bean定义的注册类
        BeanDefinitionRegistry registry = context.getRegistry();

        //获取操作系统
        String property = environment.getProperty("os.name");

        //可以判断容器中bean的注册情况，也可以给容器中注册bean
        boolean definition = registry.containsBeanDefinition("person");
        
        if (property.contains("linux")) {
            return true;
        }
        return false;
    }
}
```

------

`@Conditional` 的这个注解还可以标注在类上，对容器中的组件进行统一设置：满足当前条件，这个类中配置的所有bean注册才能生效

```java
@Configuration
//满足当前条件，这个类中配置的所有bean注册才能生效
@Conditional({WindowsCondition.class})
public class MainConfig2 {

    /**
     * @Conditional:是SpringBoot底层大量使用的注解，按照一定的条件来进行判断，满足条件 给容器注册bean
     */

    /**
     *  现在下面的两个bean注册到IOC容器是要条件的：
     *  1.如果系统是windows，给容器注册("bill")
     *  1.如果系统是linux，给容器注册("linus")
     * @return
     */

    @Bean("bill")
    public Person person01() {
        return new Person("Bill Gates",62);
    }

    @Conditional({LinuxCondition.class})
    @Bean("linus")
    public Person person02() {
        return new Person("linus",48);
    }

}
```

现在的运行结果如下：一个bean都没有， 只打印的一个操作系统的值

> linux

------

#### @Import-给容器中快速导入一个组件

我们先写上一个类：

```java
public class Color {
}

```

当我们没有添加`@Import`注解的时候：我们打印容器里面所有的组件

```java
    @Test
    public void test4() {
        ApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfig2.class);
        printBeans(applicationContext);
    }

    private void printBeans(ApplicationContext applicationContext) {
        String[] definitionNames = applicationContext.getBeanDefinitionNames();
        for (String name : definitionNames) {
            System.out.println(name);
        }
    }
```

------

这个时候，是没有Color 的这个bean在IOC容器里面

> org.springframework.context.annotation.internalConfigurationAnnotationProcessor
> org.springframework.context.annotation.internalAutowiredAnnotationProcessor
> org.springframework.context.annotation.internalRequiredAnnotationProcessor
> org.springframework.context.annotation.internalCommonAnnotationProcessor
> org.springframework.context.event.internalEventListenerProcessor
> org.springframework.context.event.internalEventListenerFactory
> mainConfig2
> bill

------

而当我们使用了 `@Import(Color.class)`注解之后，我们可以看到IOC容器里面就有这个组件了，id默认就是这个组件的全类名：

```java
@Configuration
//满足当前条件，这个类中配置的所有bean注册才能生效
@Conditional({WindowsCondition.class})
//快速导入组件，id默认是组件的全类名
@Import(Color.class)
public class MainConfig2 {

    /**
     * @Conditional:是SpringBoot底层大量使用的注解，按照一定的条件来进行判断，满足条件 给容器注册bean
     */

    /**
     *  现在下面的两个bean注册到IOC容器是要条件的：
     *  1.如果系统是windows，给容器注册("bill")
     *  1.如果系统是linux，给容器注册("linus")
     * @return
     */

    @Bean("bill")
    public Person person01() {
        return new Person("Bill Gates",62);
    }

    @Conditional({LinuxCondition.class})
    @Bean("linus")
    public Person person02() {
        return new Person("linus",48);
    }

    /**
     * 给容器中注册组件：
     * 1）、扫描+组件标注注解（@Controller/@Service/@Repository/@Component）
     * 【局限于要求是自己写的类，如果导入的第三方没有添加这些注解，那么就注册不上了】
     *
     * 2）、@Bean[导入的第三方包里面的组件]
     * 3）、@Import[快速的给容器中导入一个组件]
     */
}
```

我们可以看到现在 IOC 容器有这个com.ldc.bean.Color的这个bean的实例了：

> org.springframework.context.annotation.internalConfigurationAnnotationProcessor
> org.springframework.context.annotation.internalAutowiredAnnotationProcessor
> org.springframework.context.annotation.internalRequiredAnnotationProcessor
> org.springframework.context.annotation.internalCommonAnnotationProcessor
> org.springframework.context.event.internalEventListenerProcessor
> org.springframework.context.event.internalEventListenerFactory
> mainConfig2
> com.ldc.bean.Color
> bill

------

`@Import`这个注解也是可以导入多个组件的：

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Import {

	/**
	 * {@link Configuration}, {@link ImportSelector}, {@link ImportBeanDefinitionRegistrar}
	 * or regular component classes to import.
	 */
	Class<?>[] value();

}
```

这个时候，我们可以再来写上一个Red类：

```java
public class Red {
}
```

我们就可以这样用一个数组来导入多个组件：

```java
@Configuration
//满足当前条件，这个类中配置的所有bean注册才能生效
@Conditional({WindowsCondition.class})
//快速导入组件，id默认是组件的全类名
@Import({Color.class,Red.class})
public class MainConfig2 {

    /**
     * @Conditional:是SpringBoot底层大量使用的注解，按照一定的条件来进行判断，满足条件 给容器注册bean
     */

    /**
     *  现在下面的两个bean注册到IOC容器是要条件的：
     *  1.如果系统是windows，给容器注册("bill")
     *  1.如果系统是linux，给容器注册("linus")
     * @return
     */


    @Bean("bill")
    public Person person01() {
        return new Person("Bill Gates",62);
    }

    @Conditional({LinuxCondition.class})
    @Bean("linus")
    public Person person02() {
        return new Person("linus",48);
    }

    /**
     * 给容器中注册组件：
     * 1）、扫描+组件标注注解（@Controller/@Service/@Repository/@Component）
     * 【局限于要求是自己写的类，如果导入的第三方没有添加这些注解，那么就注册不上了】
     *
     * 2）、@Bean[导入的第三方包里面的组件]
     * 3）、@Import[快速的给容器中导入一个组件]
     */
}
```

测试结果：这个时候IOC容器里面就有Color和Red这两个bean的实例了：

> org.springframework.context.annotation.internalConfigurationAnnotationProcessor
> org.springframework.context.annotation.internalAutowiredAnnotationProcessor
> org.springframework.context.annotation.internalRequiredAnnotationProcessor
> org.springframework.context.annotation.internalCommonAnnotationProcessor
> org.springframework.context.event.internalEventListenerProcessor
> org.springframework.context.event.internalEventListenerFactory
> mainConfig2
> com.ldc.bean.Color
> com.ldc.bean.Red
> bill

------

##### @Import-使用ImportSelector

ImportSelector是一个接口：**返回需要的组件的全类名的数组；**

```java
public interface ImportSelector {

	/**
	 * Select and return the names of which class(es) should be imported based on
	 * the {@link AnnotationMetadata} of the importing @{@link Configuration} class.
	 */
	String[] selectImports(AnnotationMetadata importingClassMetadata);

}
```

现在，我们就来自定义逻辑返回需要导入的组件，需要实现 ImportSelector 这个接口：

```java
//自定义逻辑返回需要导入的组件
public class MyImportSelector implements ImportSelector {
    @Override
    public String[] selectImports(AnnotationMetadata importingClassMetadata) {
        return new String[0];
    }
}
```

------

现在，我们就可以这样来写：@Import({Color.class,Red.class,MyImportSelector.class})

```java
@Configuration
//满足当前条件，这个类中配置的所有bean注册才能生效
@Conditional({WindowsCondition.class})
//快速导入组件，id默认是组件的全类名
@Import({Color.class,Red.class,MyImportSelector.class})
public class MainConfig2 {

    /**
     * @Conditional:是SpringBoot底层大量使用的注解，按照一定的条件来进行判断，满足条件 给容器注册bean
     */

    /**
     *  现在下面的两个bean注册到IOC容器是要条件的：
     *  1.如果系统是windows，给容器注册("bill")
     *  1.如果系统是linux，给容器注册("linus")
     * @return
     */


    @Bean("bill")
    public Person person01() {
        return new Person("Bill Gates",62);
    }

    @Conditional({LinuxCondition.class})
    @Bean("linus")
    public Person person02() {
        return new Person("linus",48);
    }

    /**
     * 给容器中注册组件：
     * 1）、扫描+组件标注注解（@Controller/@Service/@Repository/@Component）
     * 【局限于要求是自己写的类，如果导入的第三方没有添加这些注解，那么就注册不上了】
     *
     * 2）、@Bean[导入的第三方包里面的组件]
     * 3）、@Import[快速的给容器中导入一个组件]
     *      （1）、 @Import(要导入容器中的组件);容器中就会自动的注册这个组件，id默认是全类名
     *      （2）、 ImportSelector ：返回需要的组件的全类名的数组；
     */
}
```

我们打断点运行这个测试方法：

```java
    @Test
    public void test4() {
        ApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfig2.class);
        printBeans(applicationContext);
    }

    private void printBeans(ApplicationContext applicationContext) {
        String[] definitionNames = applicationContext.getBeanDefinitionNames();
        for (String name : definitionNames) {
            System.out.println(name);
        }
    }
```

![在这里插入图片描述](Spring%E6%B3%A8%E8%A7%A3%E9%A9%B1%E5%8A%A8%E5%BC%80%E5%8F%91.assets/20190114132636260.png)
也就是对应着这个类里面所有注解的信息：
![在这里插入图片描述](Spring%E6%B3%A8%E8%A7%A3%E9%A9%B1%E5%8A%A8%E5%BC%80%E5%8F%91.assets/20190114132804192.png)

------

![在这里插入图片描述](Spring%E6%B3%A8%E8%A7%A3%E9%A9%B1%E5%8A%A8%E5%BC%80%E5%8F%91.assets/20190114133108206.png)

------

![在这里插入图片描述](Spring%E6%B3%A8%E8%A7%A3%E9%A9%B1%E5%8A%A8%E5%BC%80%E5%8F%91.assets/20190114133344141.png)

------

![在这里插入图片描述](Spring%E6%B3%A8%E8%A7%A3%E9%A9%B1%E5%8A%A8%E5%BC%80%E5%8F%91.assets/20190114133447234.png)

------

现在，我们再写两个类：

```java
public class Blue {
}

public class Yellow {
}

```

------

这个时候，我们就可以这样来在IOC容器里面加入bean,纳入IOC容器的管理：

```java
//自定义逻辑返回需要导入的组件
public class MyImportSelector implements ImportSelector {
    //返回值就是要导入到容器中的组件的全类名
    //AnnotationMetadata ：当前标注@Import注解的类的所有注解信息
    @Override
    public String[] selectImports(AnnotationMetadata importingClassMetadata) {
        //方法不要返回null值
        return new String[]{"com.ldc.bean.Blue","com.ldc.bean.Yellow"};
    }
}
```

------

现在，我们再来运行测试方法，看看IOC容器里面有哪些组件：这个时候Blue和Yellow也就进来了

> org.springframework.context.annotation.internalConfigurationAnnotationProcessor
> org.springframework.context.annotation.internalAutowiredAnnotationProcessor
> org.springframework.context.annotation.internalRequiredAnnotationProcessor
> org.springframework.context.annotation.internalCommonAnnotationProcessor
> org.springframework.context.event.internalEventListenerProcessor
> org.springframework.context.event.internalEventListenerFactory
> mainConfig2
> com.ldc.bean.Color
> com.ldc.bean.Red
> com.ldc.bean.Blue
> com.ldc.bean.Yellow
> bill

------

##### @Import-使用ImportBeanDefinitionRegistrar

ImportBeanDefinitionRegistrar是一个接口：

```java
public interface ImportBeanDefinitionRegistrar {

	/**
	 * Register bean definitions as necessary based on the given annotation metadata of
	 * the importing {@code @Configuration} class.
	 * <p>Note that {@link BeanDefinitionRegistryPostProcessor} types may <em>not</em> be
	 * registered here, due to lifecycle constraints related to {@code @Configuration}
	 * class processing.
	 * @param importingClassMetadata annotation metadata of the importing class
	 * @param registry current bean definition registry
	 */
	public void registerBeanDefinitions(
			AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry);

}
```

我们可以再定义一个类：

```java
public class RainBow {
}

```

这个时候，我们就可以自定义一个MyImportBeanDefinitionRegistrar类并且实现ImportBeanDefinitionRegistrar这个接口：

```java
public class MyImportBeanDefinitionRegistrar implements ImportBeanDefinitionRegistrar {
    /**
     *
     * AnnotationMetadata 当前类的注解信息
     * BeanDefinitionRegistry BeanDefinition注册类
     *
     * 我们把所有需要添加到容器中的bean通过BeanDefinitionRegistry里面的registerBeanDefinition方法来手动的进行注册
     */
    @Override
    public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
        //判断IOC容器里面是否含有这两个组件
        boolean definition = registry.containsBeanDefinition("com.ldc.bean.Red");
        boolean definition2 = registry.containsBeanDefinition("com.ldc.bean.Blue");
        //如果有的话，我就把RainBow的bean的实例给注册到IOC容器中
        if (definition && definition2) {
            //指定bean的定义信息，参数里面指定要注册的bean的类型：RainBow.class
            RootBeanDefinition rootBeanDefinition = new RootBeanDefinition(RainBow.class);
            //注册一个bean，并且指定bean名
            registry.registerBeanDefinition("rainBow", rootBeanDefinition );
        }
    }
}
```

![在这里插入图片描述](Spring%E6%B3%A8%E8%A7%A3%E9%A9%B1%E5%8A%A8%E5%BC%80%E5%8F%91.assets/20190114141008125.png)

同样，我们在`@Import`的这个注解里面添加我们自定义的MyImportBeanDefinitionRegistrar

> @Import({Color.class,Red.class,MyImportSelector.class,MyImportBeanDefinitionRegistrar.class})

```java
@Configuration
//满足当前条件，这个类中配置的所有bean注册才能生效
@Conditional({WindowsCondition.class})
//快速导入组件，id默认是组件的全类名
@Import({Color.class,Red.class,MyImportSelector.class,MyImportBeanDefinitionRegistrar.class})
public class MainConfig2 {

    /**
     * @Conditional:是SpringBoot底层大量使用的注解，按照一定的条件来进行判断，满足条件 给容器注册bean
     */

    /**
     *  现在下面的两个bean注册到IOC容器是要条件的：
     *  1.如果系统是windows，给容器注册("bill")
     *  1.如果系统是linux，给容器注册("linus")
     * @return
     */


    @Bean("bill")
    public Person person01() {
        return new Person("Bill Gates",62);
    }

    @Conditional({LinuxCondition.class})
    @Bean("linus")
    public Person person02() {
        return new Person("linus",48);
    }

    /**
     * 给容器中注册组件：
     * 1）、扫描+组件标注注解（@Controller/@Service/@Repository/@Component）
     * 【局限于要求是自己写的类，如果导入的第三方没有添加这些注解，那么就注册不上了】
     *
     * 2）、@Bean[导入的第三方包里面的组件]
     * 3）、@Import[快速的给容器中导入一个组件]
     *      （1）、 @Import(要导入容器中的组件);容器中就会自动的注册这个组件，id默认是全类名
     *      （2）、 ImportSelector ：返回需要的组件的全类名的数组；
     *      （3）、 ImportBeanDefinitionRegistrar : 手动注册bean到容器中
     */
}
```

现在，我们来进行测试：

```java
    @Test
    public void test4() {
        ApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfig2.class);
        printBeans(applicationContext);
    }

    private void printBeans(ApplicationContext applicationContext) {
        String[] definitionNames = applicationContext.getBeanDefinitionNames();
        for (String name : definitionNames) {
            System.out.println(name);
        }
    }
```

测试结果如下：这个时候，我们发现rainBow的这个组件也进来了：

> org.springframework.context.annotation.internalConfigurationAnnotationProcessor
> org.springframework.context.annotation.internalAutowiredAnnotationProcessor
> org.springframework.context.annotation.internalRequiredAnnotationProcessor
> org.springframework.context.annotation.internalCommonAnnotationProcessor
> org.springframework.context.event.internalEventListenerProcessor
> org.springframework.context.event.internalEventListenerFactory
> mainConfig2
> com.ldc.bean.Color
> com.ldc.bean.Red
> com.ldc.bean.Blue
> com.ldc.bean.Yellow
> bill
> rainBow

以上就是`@Import`的三种用法：

- @Import[快速的给容器中导入一个组件]
  - （1）、 @Import(要导入容器中的组件);容器中就会自动的注册这个组件，id默认是全类名
  - （2）、 ImportSelector ：返回需要的组件的全类名的数组；
  - （3）、 ImportBeanDefinitionRegistrar : 手动注册bean到容器中

**ImportSelector 这种用法在SpringBoot里面用的非常的多；**

------

#### 使用FactoryBean注册组件

FactoryBean：

```java
public interface FactoryBean<T> {

	/**
	 * Return an instance (possibly shared or independent) of the object
	 * managed by this factory.
	 * <p>As with a {@link BeanFactory}, this allows support for both the
	 * Singleton and Prototype design pattern.
	 * <p>If this FactoryBean is not fully initialized yet at the time of
	 * the call (for example because it is involved in a circular reference),
	 * throw a corresponding {@link FactoryBeanNotInitializedException}.
	 * <p>As of Spring 2.0, FactoryBeans are allowed to return {@code null}
	 * objects. The factory will consider this as normal value to be used; it
	 * will not throw a FactoryBeanNotInitializedException in this case anymore.
	 * FactoryBean implementations are encouraged to throw
	 * FactoryBeanNotInitializedException themselves now, as appropriate.
	 * @return an instance of the bean (can be {@code null})
	 * @throws Exception in case of creation errors
	 * @see FactoryBeanNotInitializedException
	 */
	T getObject() throws Exception;

	/**
	 * Return the type of object that this FactoryBean creates,
	 * or {@code null} if not known in advance.
	 * <p>This allows one to check for specific types of beans without
	 * instantiating objects, for example on autowiring.
	 * <p>In the case of implementations that are creating a singleton object,
	 * this method should try to avoid singleton creation as far as possible;
	 * it should rather estimate the type in advance.
	 * For prototypes, returning a meaningful type here is advisable too.
	 * <p>This method can be called <i>before</i> this FactoryBean has
	 * been fully initialized. It must not rely on state created during
	 * initialization; of course, it can still use such state if available.
	 * <p><b>NOTE:</b> Autowiring will simply ignore FactoryBeans that return
	 * {@code null} here. Therefore it is highly recommended to implement
	 * this method properly, using the current state of the FactoryBean.
	 * @return the type of object that this FactoryBean creates,
	 * or {@code null} if not known at the time of the call
	 * @see ListableBeanFactory#getBeansOfType
	 */
	Class<?> getObjectType();

	/**
	 * Is the object managed by this factory a singleton? That is,
	 * will {@link #getObject()} always return the same object
	 * (a reference that can be cached)?
	 * <p><b>NOTE:</b> If a FactoryBean indicates to hold a singleton object,
	 * the object returned from {@code getObject()} might get cached
	 * by the owning BeanFactory. Hence, do not return {@code true}
	 * unless the FactoryBean always exposes the same reference.
	 * <p>The singleton status of the FactoryBean itself will generally
	 * be provided by the owning BeanFactory; usually, it has to be
	 * defined as singleton there.
	 * <p><b>NOTE:</b> This method returning {@code false} does not
	 * necessarily indicate that returned objects are independent instances.
	 * An implementation of the extended {@link SmartFactoryBean} interface
	 * may explicitly indicate independent instances through its
	 * {@link SmartFactoryBean#isPrototype()} method. Plain {@link FactoryBean}
	 * implementations which do not implement this extended interface are
	 * simply assumed to always return independent instances if the
	 * {@code isSingleton()} implementation returns {@code false}.
	 * @return whether the exposed object is a singleton
	 * @see #getObject()
	 * @see SmartFactoryBean#isPrototype()
	 */
	boolean isSingleton();

}
```

------

首先，我们创建一个类，并且实现 FactoryBean 接口：

```java
//创建一个Spring定义的FactoryBean
public class ColorFactoryBean implements FactoryBean<Color> {

    //返回一个Color对象，这个对象会添加到容器中
    @Override
    public Color getObject() throws Exception {
        System.out.println("ColorFactoryBean...getBean...");
        return new Color();
    }

    //返回的类型
    @Override
    public Class<?> getObjectType() {
        return Color.class;
    }

    //控制是否为单例
    // true：表示的就是一个单实例，在容器中保存一份
    // false:多实例，每次获取都会创建一个新的bean
    @Override
    public boolean isSingleton() {
        return true;
    }
}
```

这个时候，我们在配置类里面进行配置：可以看到表面上我们装配的是ColorFactoryBean这个类型，但是实际上我们装配的是Color这个bean的实例：

```java
@Configuration
//满足当前条件，这个类中配置的所有bean注册才能生效
@Conditional({WindowsCondition.class})
//快速导入组件，id默认是组件的全类名
@Import({Color.class,Red.class,MyImportSelector.class,MyImportBeanDefinitionRegistrar.class})
public class MainConfig2 {

    /**
     * @Conditional:是SpringBoot底层大量使用的注解，按照一定的条件来进行判断，满足条件 给容器注册bean
     */

    /**
     *  现在下面的两个bean注册到IOC容器是要条件的：
     *  1.如果系统是windows，给容器注册("bill")
     *  1.如果系统是linux，给容器注册("linus")
     * @return
     */


    @Bean("bill")
    public Person person01() {
        return new Person("Bill Gates",62);
    }

    @Conditional({LinuxCondition.class})
    @Bean("linus")
    public Person person02() {
        return new Person("linus",48);
    }

    /**
     * 给容器中注册组件：
     * 1）、扫描+组件标注注解（@Controller/@Service/@Repository/@Component）
     * 【局限于要求是自己写的类，如果导入的第三方没有添加这些注解，那么就注册不上了】
     *
     * 2）、@Bean[导入的第三方包里面的组件]
     * 3）、@Import[快速的给容器中导入一个组件]
     *      （1）、 @Import(要导入容器中的组件);容器中就会自动的注册这个组件，id默认是全类名
     *      （2）、 ImportSelector ：返回需要的组件的全类名的数组；
     *      （3）、 ImportBeanDefinitionRegistrar : 手动注册bean到容器中
     *
     * 4）、使用Spring提供的FactoryBean（工厂bean）
     *      （1）、默认获取到的是工厂bean调用getObject创建的对象
     *      （2）、要获取工厂bean本身，我们需要给id前面加上一个“&”符号：&colorFactoryBean
     */

    @Bean
    public ColorFactoryBean colorFactoryBean() {
        return new ColorFactoryBean();
    }
}

```

最后，我们来进行测试：

```java
    @Test
    public void test4() {
        ApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfig2.class);
        printBeans(applicationContext);
        Object colorFactoryBean = applicationContext.getBean("colorFactoryBean");
        System.out.println("bean的类型："+colorFactoryBean.getClass());
    }

    private void printBeans(ApplicationContext applicationContext) {
        String[] definitionNames = applicationContext.getBeanDefinitionNames();
        for (String name : definitionNames) {
            System.out.println(name);
        }
    }
```

测试结果为：

> org.springframework.context.annotation.internalConfigurationAnnotationProcessor
> org.springframework.context.annotation.internalAutowiredAnnotationProcessor
> org.springframework.context.annotation.internalRequiredAnnotationProcessor
> org.springframework.context.annotation.internalCommonAnnotationProcessor
> org.springframework.context.event.internalEventListenerProcessor
> org.springframework.context.event.internalEventListenerFactory
> mainConfig2
> com.ldc.bean.Color
> com.ldc.bean.Red
> com.ldc.bean.Blue
> com.ldc.bean.Yellow
> bill
> colorFactoryBean
> rainBow
> ColorFactoryBean…getBean…
> bean的类型：class com.ldc.bean.Color

------

如果我们就想要获取这个工厂bean，我们就可以在id的前面加上一个"&"符号

```java
    @Test
    public void test4() {
        ApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfig2.class);
        printBeans(applicationContext);

        //工厂bean获取的是调用getObject方法创建的对象
        Object colorFactoryBean = applicationContext.getBean("colorFactoryBean");
        System.out.println("bean的类型："+colorFactoryBean.getClass());

        //如果我们就想要获取这个工厂bean，我们就可以在id的前面加上一个"&"符号
        Object colorFactoryBean2 = applicationContext.getBean("&colorFactoryBean");
        System.out.println("bean的类型："+colorFactoryBean2.getClass());
    }

    private void printBeans(ApplicationContext applicationContext) {
        String[] definitionNames = applicationContext.getBeanDefinitionNames();
        for (String name : definitionNames) {
            System.out.println(name);
        }
    }
```

测试结果：

> org.springframework.context.annotation.internalConfigurationAnnotationProcessor
> org.springframework.context.annotation.internalAutowiredAnnotationProcessor
> org.springframework.context.annotation.internalRequiredAnnotationProcessor
> org.springframework.context.annotation.internalCommonAnnotationProcessor
> org.springframework.context.event.internalEventListenerProcessor
> org.springframework.context.event.internalEventListenerFactory
> mainConfig2
> com.ldc.bean.Color
> com.ldc.bean.Red
> com.ldc.bean.Blue
> com.ldc.bean.Yellow
> bill
> colorFactoryBean
> rainBow
> ColorFactoryBean…getBean…
> colorFactoryBean的类型：class com.ldc.bean.Color
> colorFactoryBean2的类型：class com.ldc.bean.ColorFactoryBean

### 生命周期

------

#### 生命周期-`@Bean`指定初始化和销毁方法

首先，我们有一个Car类：

```java
public class Car {
    public Car() {
        System.out.println("car constructor...");
    }

    public void init() {
        System.out.println("car...init...");
    }

    public void destroy() {
        System.out.println("car...destroy...");
    }

}
```

有一个配置类：

```java
/**
 * bean的生命周期：bean的创建->初始化->销毁的过程
 * 容器管理bean的生命周期：
 * 我们可以自定义初始化方法和销毁的方法：容器在bean进行到当前的生命周期的时候，来调用我们自定义的初始化方法和销毁方法
 * 构造（对象创建）：
 *      单实例：在容器启动的时候创建对象
 *      多实例：在每次获取的时候来创建对象
 * 初始化方法：
 *      对象创建完成，并赋值好，调用初始化方法
 * 销毁方法：
 *      单实例的bean:在容器关闭的时候进行销毁
 *      多实例的bean:容器不会管理这个bean,容器不会调用销毁的方法
 *
 * 1)指定初始化方法和销毁方法；
 *          -我们可以通过@Bean(initMethod = "init",destroyMethod = "destroy")来指定初始化方法和销毁方法
 *          相当于xml配置文件bean标签里面的 init-method="" 和 destroy-method="" 属性
 *
 *
 *
 */
@Configuration
public class MainConfigOfLifeCycle {

    @Bean(initMethod = "init",destroyMethod = "destroy")
    public Car car() {
        return new Car();
    }
}
```

我们再来写一个测试方法：

```java
    @Test
    public void test01() {
        //1.创建IOC容器
        AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfigOfLifeCycle.class);
        System.out.println("容器创建完成");
    }

```

此时，还没有关闭IOC容器的时候，运行结果为：

> car constructor…
> car…init…
> 容器创建完成

而当我们调用了容器的close方法关闭容器的时候：

```java
    @Test
    public void test01() {
        //1.创建IOC容器
        AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfigOfLifeCycle.class);
        System.out.println("容器创建完成");
        applicationContext.close();
    }

```

测试的运行结果为：

> car constructor…
> car…init…
> 容器创建完成
> car…destroy…

而当bean的作用域为多例的时候：

```java
@Configuration
public class MainConfigOfLifeCycle {

    @Scope("prototype")
    @Bean(initMethod = "init",destroyMethod = "destroy")
    public Car car() {
        return new Car();
    }
}

```

这个时候，我们再来运行这个测试方法：

```java
    @Test
    public void test01() {
        //1.创建IOC容器
        AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfigOfLifeCycle.class);
        System.out.println("容器创建完成");
        applicationContext.close();
    }

```

运行结果为：

> 容器创建完成

当bean的作用域为多例的时候，只有在获取的时候，才会创建对象，而且在IOC容器关闭的时候，是不进行销毁的 ：

```java
    @Test
    public void test01() {
        //1.创建IOC容器
        AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfigOfLifeCycle.class);
        System.out.println("容器创建完成");
        applicationContext.getBean("car");
        applicationContext.close();
    }

```

此时，运行结果为：

> 容器创建完成
> car constructor…
> car…init…

------

#### 生命周期-`InitializingBean`和`DisposableBean`

InitializingBean接口：在bean的初始化方法调用之后进行调用

```java
public interface InitializingBean {

	/**
	 * Invoked by a BeanFactory after it has set all bean properties supplied
	 * (and satisfied BeanFactoryAware and ApplicationContextAware).
	 * <p>This method allows the bean instance to perform initialization only
	 * possible when all bean properties have been set and to throw an
	 * exception in the event of misconfiguration.
	 * @throws Exception in the event of misconfiguration (such
	 * as failure to set an essential property) or if initialization fails.
	 */
	void afterPropertiesSet() throws Exception;

}

```

DisposableBean接口：

```java
public interface DisposableBean {

	/**
	 * Invoked by a BeanFactory on destruction of a singleton.
	 * @throws Exception in case of shutdown errors.
	 * Exceptions will get logged but not rethrown to allow
	 * other beans to release their resources too.
	 */
	void destroy() throws Exception;

}

```

------

有一个cat类：实现这两个接口

```java
@Component
public class Cat implements InitializingBean, DisposableBean {
    public Cat() {
        System.out.println("Cat...Contrustor...");
    }

    @Override
    public void destroy() throws Exception {
        System.out.println("Cat...destroy...");
    }

    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("Cat...afterPropertiesSet...");
    }
}

```

这次，我们就用包扫描的方式来进行装配：

```java
/**
 * bean的生命周期：bean的创建->初始化->销毁的过程
 * 容器管理bean的生命周期：
 * 我们可以自定义初始化方法和销毁的方法：容器在bean进行到当前的生命周期的时候，来调用我们自定义的初始化方法和销毁方法
 * 构造（对象创建）：
 *      单实例：在容器启动的时候创建对象
 *      多实例：在每次获取的时候来创建对象
 * 初始化方法：
 *      对象创建完成，并赋值好，调用初始化方法
 * 销毁方法：
 *      单实例的bean:在容器关闭的时候进行销毁
 *      多实例的bean:容器不会管理这个bean,容器不会调用销毁的方法
 *
 * 1)指定初始化方法和销毁方法；
 *          -我们可以通过@Bean(initMethod = "init",destroyMethod = "destroy")来指定初始化方法和销毁方法
 *          相当于xml配置文件bean标签里面的 init-method="" 和 destroy-method="" 属性
 * 2)通过bean实现InitializingBean（定义初始化逻辑）
 *               DisposableBean（定义销毁逻辑）；
 *
 *
 */
@Configuration
@ComponentScan("com.ldc.bean")
public class MainConfigOfLifeCycle {

    @Bean(initMethod = "init",destroyMethod = "destroy")
    public Car car() {
        return new Car();
    }
}

```

测试：

```java
    @Test
    public void test01() {
        //1.创建IOC容器
        AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfigOfLifeCycle.class);
        System.out.println("容器创建完成");
        applicationContext.close();
    }

```

运行结果：

> Cat…Contrustor…
> Cat…afterPropertiesSet…
> car constructor…
> car…init…
> 容器创建完成
> car…destroy…
> Cat…destroy…

#### 生命周期- `@PostConstruct`&`@PreDestroy`

- 3）可以使用JSR250规范里面定义的两个注解：
  - @PostConstruct :在bean创建完成并且属性赋值完成，来执行初始化方法
  - @PreDestroy ：在容器销毁bean之前通知我们来进行清理工作

------

`@PostConstruct`：在bean创建完成并且属性赋值完成，来执行初始化方法

```java
@Documented
@Retention (RUNTIME)
@Target(METHOD)
public @interface PostConstruct {
}

```

`@PreDestroy`：在容器销毁bean之前通知我们来进行清理工作

```java
@Documented
@Retention (RUNTIME)
@Target(METHOD)
public @interface PreDestroy {
}

```

我们再来定义一个Dog类：

```java
@Component
public class Dog {
    public Dog() {
        System.out.println("Dog...Contructor...");
    }

    //在对象创建并赋值之后调用
    @PostConstruct
    public void init() {
        System.out.println("Dog...@PostConstruct...");
    }

    //在对象创建并赋值之后调用
    @PreDestroy
    public void destroy() {
        System.out.println("Dog...@PreDestroy...");
    }
}

```

我们再来运行这个测试方法：

```java
@Test
    public void test01() {
        //1.创建IOC容器
        AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfigOfLifeCycle.class);
        System.out.println("容器创建完成");
        applicationContext.close();
    }

```

测试结果如下：

> Cat…Contrustor…
> Cat…afterPropertiesSet…
> Dog…Contructor…
> Dog…@PostConstruct…
> car constructor…
> car…init…
> 容器创建完成
> car…destroy…
> Dog…@PreDestroy…
> Cat…destroy…

------

#### 生命周期-`BeanPostProcessor`-后置处理器

- 4）BeanPostProcessor接口：bean的后置处理器，在bean初始化前后做一些处理工作，这个接口有两个方法：
  - postProcessBeforeInitialization：在初始化之前工作
  - postProcessAfterInitialization：在初始化之后工作

------

BeanPostProcessor接口：

```java
public interface BeanPostProcessor {

	/**
	 * Apply this BeanPostProcessor to the given new bean instance <i>before</i> any bean
	 * initialization callbacks (like InitializingBean's {@code afterPropertiesSet}
	 * or a custom init-method). The bean will already be populated with property values.
	 * The returned bean instance may be a wrapper around the original.
	 * @param bean the new bean instance
	 * @param beanName the name of the bean
	 * @return the bean instance to use, either the original or a wrapped one;
	 * if {@code null}, no subsequent BeanPostProcessors will be invoked
	 * @throws org.springframework.beans.BeansException in case of errors
	 * @see org.springframework.beans.factory.InitializingBean#afterPropertiesSet
	 */
	Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException;

	/**
	 * Apply this BeanPostProcessor to the given new bean instance <i>after</i> any bean
	 * initialization callbacks (like InitializingBean's {@code afterPropertiesSet}
	 * or a custom init-method). The bean will already be populated with property values.
	 * The returned bean instance may be a wrapper around the original.
	 * <p>In case of a FactoryBean, this callback will be invoked for both the FactoryBean
	 * instance and the objects created by the FactoryBean (as of Spring 2.0). The
	 * post-processor can decide whether to apply to either the FactoryBean or created
	 * objects or both through corresponding {@code bean instanceof FactoryBean} checks.
	 * <p>This callback will also be invoked after a short-circuiting triggered by a
	 * {@link InstantiationAwareBeanPostProcessor#postProcessBeforeInstantiation} method,
	 * in contrast to all other BeanPostProcessor callbacks.
	 * @param bean the new bean instance
	 * @param beanName the name of the bean
	 * @return the bean instance to use, either the original or a wrapped one;
	 * if {@code null}, no subsequent BeanPostProcessors will be invoked
	 * @throws org.springframework.beans.BeansException in case of errors
	 * @see org.springframework.beans.factory.InitializingBean#afterPropertiesSet
	 * @see org.springframework.beans.factory.FactoryBean
	 */
	Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException;

}

```

同样，还是使用包扫描的方式来进行装配：

```java
/**
 * bean的生命周期：bean的创建->初始化->销毁的过程
 * 容器管理bean的生命周期：
 * 我们可以自定义初始化方法和销毁的方法：容器在bean进行到当前的生命周期的时候，来调用我们自定义的初始化方法和销毁方法
 * 构造（对象创建）：
 *      单实例：在容器启动的时候创建对象
 *      多实例：在每次获取的时候来创建对象
 * 初始化方法：
 *      对象创建完成，并赋值好，调用初始化方法
 * 销毁方法：
 *      单实例的bean:在容器关闭的时候进行销毁
 *      多实例的bean:容器不会管理这个bean,容器不会调用销毁的方法
 *
 * 1)指定初始化方法和销毁方法；
 *          -我们可以通过@Bean(initMethod = "init",destroyMethod = "destroy")来指定初始化方法和销毁方法
 *          相当于xml配置文件bean标签里面的 init-method="" 和 destroy-method="" 属性
 * 2)通过bean实现InitializingBean（定义初始化逻辑）
 *               DisposableBean（定义销毁逻辑）；
 *
 * 3）可以使用JSR250规范里面定义的两个注解：
 *      @PostConstruct :在bean创建完成并且属性赋值完成，来执行初始化方法
 *      @PreDestroy ：在容器销毁bean之前通知我们来进行清理工作
 * 4）BeanPostProcessor接口：bean的后置处理器
 * 在bean初始化前后做一些处理工作，这个接口有两个方法：
 *      - postProcessBeforeInitialization：在初始化之前工作
 *      - postProcessAfterInitialization：在初始化之后工作
 */
@Configuration
@ComponentScan("com.ldc.bean")
public class MainConfigOfLifeCycle {

    @Bean(initMethod = "init",destroyMethod = "destroy")
    public Car car() {
        return new Car();
    }
}

```

我们来写一个类并且实现这个后置处理器接口 BeanPostProcessor ：

```java
@Component
public class MyBeanPostProcessor implements BeanPostProcessor {
    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("postProcessBeforeInitialization..."+beanName+"=>"+bean);
        return bean;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("postProcessAfterInitialization..."+beanName+"=>"+bean);
        return bean;
    }
}

```

此时，我们再来运行这个测试方法：

```java
    @Test
    public void test01() {
        //1.创建IOC容器
        AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfigOfLifeCycle.class);
        System.out.println("容器创建完成");
        applicationContext.close();
    }

```

运行结果如下：

> postProcessBeforeInitialization…org.springframework.context.event.internalEventListenerProcessor=>org.springframework.context.event.EventListenerMethodProcessor@36f0f1be
> postProcessAfterInitialization…org.springframework.context.event.internalEventListenerProcessor=>org.springframework.context.event.EventListenerMethodProcessor@36f0f1be
> postProcessBeforeInitialization…org.springframework.context.event.internalEventListenerFactory=>org.springframework.context.event.DefaultEventListenerFactory@6ee12bac
> postProcessAfterInitialization…org.springframework.context.event.internalEventListenerFactory=>org.springframework.context.event.DefaultEventListenerFactory@6ee12bac
> postProcessBeforeInitialization…mainConfigOfLifeCycle=>com.ldc.config.MainConfigOfLifeCycleE n h a n c e r B y S p r i n g C G L I B EnhancerBySpringCGLIB*E**n**h**a**n**c**e**r**B**y**S**p**r**i**n**g**C**G**L**I**B*27d6c7d3@64c87930
> postProcessAfterInitialization…mainConfigOfLifeCycle=>com.ldc.config.MainConfigOfLifeCycleE n h a n c e r B y S p r i n g C G L I B EnhancerBySpringCGLIB*E**n**h**a**n**c**e**r**B**y**S**p**r**i**n**g**C**G**L**I**B*27d6c7d3@64c87930
> Cat…Contrustor…
> postProcessBeforeInitialization…cat=>com.ldc.bean.Cat@525f1e4e
> Cat…afterPropertiesSet…
> postProcessAfterInitialization…cat=>com.ldc.bean.Cat@525f1e4e
> Dog…Contructor…
> postProcessBeforeInitialization…dog=>com.ldc.bean.Dog@5ea434c8
> Dog…@PostConstruct…
> postProcessAfterInitialization…dog=>com.ldc.bean.Dog@5ea434c8
> car constructor…
> postProcessBeforeInitialization…car=>com.ldc.bean.Car@1d548a08
> car…init…
> postProcessAfterInitialization…car=>com.ldc.bean.Car@1d548a08
> 容器创建完成
> car…destroy…
> Dog…@PreDestroy…
> Cat…destroy…

------

#### 生命周期-`BeanPostProcessor`原理

我们在这个后置处理器的两个方法上面打上断点：
![在这里插入图片描述](Spring%E6%B3%A8%E8%A7%A3%E9%A9%B1%E5%8A%A8%E5%BC%80%E5%8F%91.assets/20190114202731852.png)

------

然后，我们以debug的方式来运行这个测试方法：

```java
    @Test
    public void test01() {
        //1.创建IOC容器
        AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfigOfLifeCycle.class);
        System.out.println("容器创建完成");
        applicationContext.close();
    }

```

我们大概走一下这个方法调用栈：
![在这里插入图片描述](Spring%E6%B3%A8%E8%A7%A3%E9%A9%B1%E5%8A%A8%E5%BC%80%E5%8F%91.assets/2019011420305010.png)

------

![在这里插入图片描述](Spring%E6%B3%A8%E8%A7%A3%E9%A9%B1%E5%8A%A8%E5%BC%80%E5%8F%91.assets/20190114203449285.png)

------

![在这里插入图片描述](Spring%E6%B3%A8%E8%A7%A3%E9%A9%B1%E5%8A%A8%E5%BC%80%E5%8F%91.assets/20190114203822859.png)

------

前置处理器调用的方法：调用getBeanPostProcessors()方法找到容器里面的所有的BeanPostProcessor，挨个遍历，调用BeanPostProcessor的postProcessBeforeInitialization方法，一旦调用postProcessBeforeInitialization方法的返回值为null的时候，就直接跳出遍历 ，后面的BeanPostProcessor 的postProcessBeforeInitialization也就不会执行了：

```java
	@Override
	public Object applyBeanPostProcessorsBeforeInitialization(Object existingBean, String beanName)
			throws BeansException {

		Object result = existingBean;
		for (BeanPostProcessor beanProcessor : getBeanPostProcessors()) {
			result = beanProcessor.postProcessBeforeInitialization(result, beanName);
			if (result == null) {
				return result;
			}
		}
		return result;
	}

```

后置处理器调用的方法：调用getBeanPostProcessors()方法找到容器里面的所有的BeanPostProcessor，挨个遍历，调用BeanPostProcessor的postProcessAfterInitialization方法，一旦调用postProcessAfterInitialization方法的返回值为null的时候，就直接跳出遍历 ，后面的BeanPostProcessor 的postProcessAfterInitialization也就不会执行了：

```java
	@Override
	public Object applyBeanPostProcessorsAfterInitialization(Object existingBean, String beanName)
			throws BeansException {

		Object result = existingBean;
		for (BeanPostProcessor beanProcessor : getBeanPostProcessors()) {
			result = beanProcessor.postProcessAfterInitialization(result, beanName);
			if (result == null) {
				return result;
			}
		}
		return result;
	}

```

------

> //前置处理器执行
> wrappedBean = applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);
> //初始化方法执行
> invokeInitMethods(beanName, wrappedBean, mbd);
> //后置处理器执行
> wrappedBean = applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);

在看源码的时候，我们可以发现：
在执行exposedObject = initializeBean(beanName, exposedObject, mbd);方法之前先调用了populateBean(beanName, mbd, instanceWrapper);这个方法：

populateBean(beanName, mbd, instanceWrapper);–给bean进行属性赋值
exposedObject = initializeBean(beanName, exposedObject, mbd);–初始化方法，在这个初始化方法内部包括了前置处理器、初始化方法、后置处理器的调用：

![在这里插入图片描述](Spring%E6%B3%A8%E8%A7%A3%E9%A9%B1%E5%8A%A8%E5%BC%80%E5%8F%91.assets/20190114205251946.png)

------

#### 生命周期- `BeanPostProcessor`在Spring底层的使用

- Spring底层对 BeanPostProcessor 的使用：
  - bean赋值，注入其他组件，`@Autowired`,生命周期注解等功能,`@Async`等等都是使用`BeanPostProcessor`来完成的

BeanPostProcessor 这个接口有很多的实现类：
![在这里插入图片描述](Spring%E6%B3%A8%E8%A7%A3%E9%A9%B1%E5%8A%A8%E5%BC%80%E5%8F%91.assets/20190114210907990.png)

------

如果我们想要获取IOC容器，我们可以这样做：

```java
@Component
public class Dog implements ApplicationContextAware {
                 //↑↑↑↑↑↑↑↑
    private ApplicationContext applicationContext;

    public Dog() {
        System.out.println("Dog...Contructor...");
    }

    //在对象创建并赋值之后调用
    @PostConstruct
    public void init() {
        System.out.println("Dog...@PostConstruct...");
    }

    //在对象创建并赋值之后调用
    @PreDestroy
    public void destroy() {
        System.out.println("Dog...@PreDestroy...");
    }

    //↓↓↓↓↓↓↓↓↓↓
    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        this.applicationContext = applicationContext;
    }
}

```

我们可以debug来调试：
![在这里插入图片描述](Spring%E6%B3%A8%E8%A7%A3%E9%A9%B1%E5%8A%A8%E5%BC%80%E5%8F%91.assets/20190114211329555.png)

------

我们还是debug运行这个测试方法：

```java
    @Test
    public void test01() {
        //1.创建IOC容器
        AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfigOfLifeCycle.class);
        System.out.println("容器创建完成");
        applicationContext.close();
    }

```

------

实际上BeanPostProcessor接口还有很多的实现类，比如说BeanValidationPostProcessor，这个是用来做数据校验的：
![在这里插入图片描述](Spring%E6%B3%A8%E8%A7%A3%E9%A9%B1%E5%8A%A8%E5%BC%80%E5%8F%91.assets/20190114211752340.png)

------

像InitDestroyAnnotationBeanPostProcessor这个实现类就是实现了@PostConstruct和@PreDestroy这两个注解的功能：
![在这里插入图片描述](Spring%E6%B3%A8%E8%A7%A3%E9%A9%B1%E5%8A%A8%E5%BC%80%E5%8F%91.assets/20190114212227938.png)

------

我们在这个地方打个断点：
![在这里插入图片描述](Spring%E6%B3%A8%E8%A7%A3%E9%A9%B1%E5%8A%A8%E5%BC%80%E5%8F%91.assets/20190114212657257.png)

------

![在这里插入图片描述](Spring%E6%B3%A8%E8%A7%A3%E9%A9%B1%E5%8A%A8%E5%BC%80%E5%8F%91.assets/20190114213229199.png)

### 属性赋值

#### 属性赋值-`@Value`赋值

有一个Person类：

```java
public class Person {
    private String name;
    private Integer age;

    public Person() {
    }

    public Person(String name, Integer age) {
        this.name = name;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return "Person{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}
```

还有一个配置类：

```java
@Configuration
public class MainConfigOfPropertyValues {

    @Bean
    public Person person() {
        return new Person();
    }
}
```

我们来运行这个测试方法：

```java
    @Test
    public void test01() {
        //1.创建IOC容器
        ApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfigOfPropertyValues.class);
        System.out.println("====================");

        Person person = (Person) applicationContext.getBean("person");
        System.out.println(person);
        printBeans(applicationContext);
    }

    private void printBeans(ApplicationContext applicationContext) {
        String[] definitionNames = applicationContext.getBeanDefinitionNames();
        for (String name : definitionNames) {
            System.out.println(name);
        }
    }
```

运行结果如下：

> ====================
> Person{name=‘null’, age=null}
> org.springframework.context.annotation.internalConfigurationAnnotationProcessor
> org.springframework.context.annotation.internalAutowiredAnnotationProcessor
> org.springframework.context.annotation.internalRequiredAnnotationProcessor
> org.springframework.context.annotation.internalCommonAnnotationProcessor
> org.springframework.context.event.internalEventListenerProcessor
> org.springframework.context.event.internalEventListenerFactory
> mainConfigOfPropertyValues
> person

------

我们可以看容器中的Person{name=‘null’, age=null}值都会默认值，

现在，我们就可以这样来写：

```java
public class Person {

    //使用@Value赋值
    //1.基本的数值
    //2.可以写SpEL: #{}
    //3.可以写${}：取出配置文件中的值（在运行环境变量里面的值）
    @Value("张三")
    private String name;
    
    @Value("#{20-2}")
    private Integer age;

    public Person() {
    }

    public Person(String name, Integer age) {
        this.name = name;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return "Person{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}
```

这个时候，我们再来运行以上的测试方法，测试结果如下：这个时候，Person的属性就是可以获取到值了：

> ====================
> Person{name=‘张三’, age=18}
> org.springframework.context.annotation.internalConfigurationAnnotationProcessor
> org.springframework.context.annotation.internalAutowiredAnnotationProcessor
> org.springframework.context.annotation.internalRequiredAnnotationProcessor
> org.springframework.context.annotation.internalCommonAnnotationProcessor
> org.springframework.context.event.internalEventListenerProcessor
> org.springframework.context.event.internalEventListenerFactory
> mainConfigOfPropertyValues
> person

------

#### 属性赋值-`@PropertySource`加载外部配置文件

我们为Person这个类再添加一个属性nickName:

```java
public class Person {

    //使用@Value赋值
    //1.基本的数值
    //2.可以写SpEL: #{}
    //3.可以写${}：取出配置文件【properties】中的值（在运行环境变量里面的值）
    @Value("张三")
    private String name;

    @Value("#{20-2}")
    private Integer age;

    private String nickName;

    public Person() {
    }

    public Person(String name, Integer age) {
        this.name = name;
        this.age = age;
    }


    public String getNickName() {
        return nickName;
    }

    public void setNickName(String nickName) {
        this.nickName = nickName;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return "Person{" +
                "name='" + name + '\'' +
                ", age=" + age +
                ", nickName='" + nickName + '\'' +
                '}';
    }
}
```

我们再来写上一个person.properties文件：

```properties
person.nickName=小张三
```

在用xml文件配置的时候，我们是这样做的：

```xml
<context:property-placeholder location="classpath:person.properties"/>
```

现在，我们用注解的方式就可以这样来做：
我们要添加这样的一个注解：`@PropertySource`，**查看源码的时候，我们可以发现，这个注解是一个可重复标注的注解，可多次标注，也可以在一个注解内添加外部配置文件位置的数组，我们也可以用PropertySources内部包含多个PropertySource ：**

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Repeatable(PropertySources.class)
public @interface PropertySource {

	/**
	 * Indicate the name of this property source. If omitted, a name will
	 * be generated based on the description of the underlying resource.
	 * @see org.springframework.core.env.PropertySource#getName()
	 * @see org.springframework.core.io.Resource#getDescription()
	 */
	String name() default "";

	/**
	 * Indicate the resource location(s) of the properties file to be loaded.
	 * For example, {@code "classpath:/com/myco/app.properties"} or
	 * {@code "file:/path/to/file"}.
	 * <p>Resource location wildcards (e.g. *&#42;/*.properties) are not permitted;
	 * each location must evaluate to exactly one {@code .properties} resource.
	 * <p>${...} placeholders will be resolved against any/all property sources already
	 * registered with the {@code Environment}. See {@linkplain PropertySource above}
	 * for examples.
	 * <p>Each location will be added to the enclosing {@code Environment} as its own
	 * property source, and in the order declared.
	 */
	String[] value();

	/**
	 * Indicate if failure to find the a {@link #value() property resource} should be
	 * ignored.
	 * <p>{@code true} is appropriate if the properties file is completely optional.
	 * Default is {@code false}.
	 * @since 4.0
	 */
	boolean ignoreResourceNotFound() default false;

	/**
	 * A specific character encoding for the given resources, e.g. "UTF-8".
	 * @since 4.3
	 */
	String encoding() default "";

	/**
	 * Specify a custom {@link PropertySourceFactory}, if any.
	 * <p>By default, a default factory for standard resource files will be used.
	 * @since 4.3
	 * @see org.springframework.core.io.support.DefaultPropertySourceFactory
	 * @see org.springframework.core.io.support.ResourcePropertySource
	 */
	Class<? extends PropertySourceFactory> factory() default PropertySourceFactory.class;

}
```

------

```java
@PropertySources ：内部可以指定多个@PropertySource
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface PropertySources {

	PropertySource[] value();

}
```

------

类似于xml文件配置的这一步：
我们要用`@PropertySource`这个注解来指定外部文件的位置：`@PropertySource(value = {"classpath:/person.properties"})`

```java
//使用@PropertySource读取外部配置文件中的key/value保存到运行的环境变量中
//加载完外部配置文件以后使用${}取出配置文件的值
@PropertySource(value = {"classpath:/person.properties"})
@Configuration
public class MainConfigOfPropertyValues {

    @Bean
    public Person person() {
        return new Person();
    }
}
```

然后，我们就可以用`@Value`，里面用${}就可以引用配置文件中的值了：

```java
public class Person {

    //使用@Value赋值
    //1.基本的数值
    //2.可以写SpEL: #{}
    //3.可以写${}：取出配置文件【properties】中的值（在运行环境变量里面的值）
    @Value("张三")
    private String name;

    @Value("#{20-2}")
    private Integer age;

    @Value("${person.nickName}")
    private String nickName;

    public Person() {
    }

    public Person(String name, Integer age) {
        this.name = name;
        this.age = age;
    }


    public String getNickName() {
        return nickName;
    }

    public void setNickName(String nickName) {
        this.nickName = nickName;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return "Person{" +
                "name='" + name + '\'' +
                ", age=" + age +
                ", nickName='" + nickName + '\'' +
                '}';
    }
}
```

我们再来运行测试方法，运行结果如下：这个时候，nickName就有值了：

> ====================
> Person{name=‘张三’, age=18, nickName=‘小张三’}
> org.springframework.context.annotation.internalConfigurationAnnotationProcessor
> org.springframework.context.annotation.internalAutowiredAnnotationProcessor
> org.springframework.context.annotation.internalRequiredAnnotationProcessor
> org.springframework.context.annotation.internalCommonAnnotationProcessor
> org.springframework.context.event.internalEventListenerProcessor
> org.springframework.context.event.internalEventListenerFactory
> mainConfigOfPropertyValues
> person

------

我们还可以用Environment里面的getProperty()方法来获取：

```java
@Test
    public void test01() {
        //1.创建IOC容器
        ApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfigOfPropertyValues.class);
        System.out.println("====================");

        Person person = (Person) applicationContext.getBean("person");
        System.out.println(person);

        Environment environment = applicationContext.getEnvironment();
        String property = environment.getProperty("person.nickName");
        System.out.println(property);

        printBeans(applicationContext);
    }

    private void printBeans(ApplicationContext applicationContext) {
        String[] definitionNames = applicationContext.getBeanDefinitionNames();
        for (String name : definitionNames) {
            System.out.println(name);
        }
    }
```

测试结果如下：

> ====================
> Person{name=‘张三’, age=18, nickName=‘小张三’}
> 小张三
> org.springframework.context.annotation.internalConfigurationAnnotationProcessor
> org.springframework.context.annotation.internalAutowiredAnnotationProcessor
> org.springframework.context.annotation.internalRequiredAnnotationProcessor
> org.springframework.context.annotation.internalCommonAnnotationProcessor
> org.springframework.context.event.internalEventListenerProcessor
> org.springframework.context.event.internalEventListenerFactory
> mainConfigOfPropertyValues
> person

------

### 自动装配

#### 自动装配-`@Autowired`&`@Qualifier`&`@Primary`

在原来，我们就是使用`@Autowired`的这个注解来进行自动装配；

现在，我们有一个BookController 类：

```java
@Controller
public class BookController {

    @Autowired
    private BookService bookService;

}
```

还有一个BookService：

```java
@Service
public class BookService {
    @Autowired
    private BookDao bookDao;

    public void print() {
        System.out.println(bookDao);
    }

    @Override
    public String toString() {
        return "BookService{" +
                "bookDao=" + bookDao +
                '}';
    }
}
```

还有一个BookDao：为了在自动装配的是哪一个，我们给这个BookDao加上一个标识属性：lable ，如果是通过包扫描到IOC容器中，标识为1，如果是在配置类里面通过`@Bean`装配的标识为2：

```java
//在IOC容器里面默认就是类名的首字母小写
@Repository
public class BookDao {

    private String lable = "1";

    public String getLable() {
        return lable;
    }

    public void setLable(String lable) {
        this.lable = lable;
    }

    @Override
    public String toString() {
        return "BookDao{" +
                "lable='" + lable + '\'' +
                '}';
    }
}
```

现在，我们有一个配置类：

```java
/**
 * 自动装配：
 *      Spring利用依赖注入（DI）完成对IOC容器中各个组件的依赖关系赋值
 * 1) @Autowired：自动注入
 *      (1)默认优先按照类型去容器中去找对应的组件：applicationContext.getBean(BookDao.class);如果找到了则进行赋值；
 *      public class BookService {
 *          @Autowired
 *          BookDao bookDao;
 *      }
 */
@Configuration
@ComponentScan({"com.ldc.service","com.ldc.dao","com.ldc.controller"})
public class MainConfigOfAutowired {

    @Bean("bookDao2")
    public BookDao bookDao() {
        BookDao bookDao = new BookDao();
        bookDao.setLable("2");
        return bookDao;
    }

}
```

我们再来写上一个测试类：为了区分是装配的是通过包扫描的方式还是通过在配置类里面进行装配的：

```java
    @Test
    public void test01() {
        //1.创建IOC容器
        AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfigOfAutowired.class);
        BookService bookService = applicationContext.getBean(BookService.class);
        System.out.println(bookService);
    }
```

测试结果如下：

> BookService{bookDao=BookDao{lable=‘1’}}

**自动装配**：
Spring利用依赖注入（DI）完成对IOC容器中各个组件的依赖关系赋值
1）`@Autowired`：自动注入
（1）默认优先按照类型去容器中去找对应的组件：applicationContext.getBean(BookDao.class);如果找到了则进行赋值；
（2）如果找到了多个相同类型的组件，再将属性的名称作为组件的id去容器中查找
applicationContext.getBean(“bookDao”)
（3）`@Qualifier("bookDao")`：使用 `@Qualifier` 指定需要装配的组件的id，而不是使用属性名
（4）自动装配默认一定要将属性赋值好，没有就会报错，可以使用`@Autowired(required = false)`;来设置为非必须的
（5）可以利用`@Primary`：让Spring在进行自动装配的时候，默认使用首选的bean
也可以继续使用`@Qualifier("bookDao")`来明确指定需要装配的bean的名字

```java
      public class BookService {
          @Autowired
          BookDao bookDao;
      }
```

如果我们想要装配bookDao2：我们就把属性名改成bookDao2就可以了：

```java
      public class BookService {
          @Autowired
          BookDao bookDao2;
      }
```

我们就可以这样来写：

```java
@Service
public class BookService {
    @Autowired
    private BookDao bookDao2;

    public void print() {
        System.out.println(bookDao2);
    }

    @Override
    public String toString() {
        return "BookService{" +
                "bookDao=" + bookDao2 +
                '}';
    }
}
```

这个时候，我们再来进行测试：这个时候，如果找找到了多个相同类型的bean，那么就是装配的bookDao2了

> BookService{bookDao=BookDao{lable=‘2’}}

------

虽然，我们在属性名写了bookDao2，但是，我就想要装配bookDao;实际上也是可以的：我们可以使用 `@Qualifier`这个注解
`@Qualifier("bookDao")`：使用 `@Qualifier` 指定需要装配的组件的id，而不是使用属性名

```java
@Service
public class BookService {

    @Qualifier("bookDao")
    @Autowired
    private BookDao bookDao2;

    public void print() {
        System.out.println(bookDao2);
    }

    @Override
    public String toString() {
        return "BookService{" +
                "bookDao=" + bookDao2 +
                '}';
    }
}
```

这个时候，我们再来测试：发现又装配了bookDao了

> BookService{bookDao=BookDao{lable=‘1’}}

------

而当我们的容器里面没有一个对应的bean的时候，这个时候，就是会报一个错 ：

> org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name ‘bookService’: Unsatisfied dependency expressed through field ‘bookDao2’; nested exception is org.springframework.beans.factory.NoSuchBeanDefinitionException: No qualifying bean of type ‘com.ldc.dao.BookDao’ available: expected at least 1 bean which qualifies as autowire candidate. Dependency annotations: {@org.springframework.beans.factory.annotation.Qualifier(value=bookDao), @org.springframework.beans.factory.annotation.Autowired(required=true)}

那可不可以在使用自动装配的时候，这个bean不是必须的呢？如果容器里面没有对应的bean，我就不装配，实际上也是可以的：我们要`@Autowired`注解里面添加`required = false`这个属性`@Autowired(required = false)`

```java
@Service
public class BookService {

    @Qualifier("bookDao")
    @Autowired(required = false)
    private BookDao bookDao2;

    public void print() {
        System.out.println(bookDao2);
    }

    @Override
    public String toString() {
        return "BookService{" +
                "bookDao=" + bookDao2 +
                '}';
    }
}
```

这个时候，我们再来运行测试方法，测试结果为：

> BookService{bookDao=null}

我们还可以利用一个注解来让Spring在自动装配的时候，首选装配哪个bean：`@Primary`

```java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
public @interface Primary {

}

@Configuration
@ComponentScan({"com.ldc.service","com.ldc.dao","com.ldc.controller"})
public class MainConfigOfAutowired {

    @Primary
    @Bean("bookDao2")
    public BookDao bookDao() {
        BookDao bookDao = new BookDao();
        bookDao.setLable("2");
        return bookDao;
    }

}
```

当然明确指定的注解也是不能用了：`@Qualifier("bookDao")`

```java
@Service
public class BookService {

    //@Qualifier("bookDao")
    @Autowired(required = false)
    private BookDao bookDao2;

    public void print() {
        System.out.println(bookDao2);
    }

    @Override
    public String toString() {
        return "BookService{" +
                "bookDao=" + bookDao2 +
                '}';
    }
}
```

这个时候，我们再来运行测试方法，测试结果如下：这个时候，Spring就首选装配了标注了`@Primary`注解的bean：

> BookService{bookDao=BookDao{lable=‘2’}}

------

我们把装配的时候的属性名也变一下：

```java
@Service
public class BookService {

    //@Qualifier("bookDao")
    @Autowired(required = false)
    private BookDao bookDao;

    public void print() {
        System.out.println(bookDao);
    }

    @Override
    public String toString() {
        return "BookService{" +
                "bookDao=" + bookDao +
                '}';
    }
}

```

我们再来看看测试结果：还是装配了标注了`@Primary`注解的bean

> BookService{bookDao=BookDao{lable=‘2’}}

如果是使用了`@Qualifier("bookDao")`明确指定了的：那还是按照明确指定的bean来进行装配

```java
@Service
public class BookService {

    @Qualifier("bookDao")
    @Autowired(required = false)
    private BookDao bookDao;

    public void print() {
        System.out.println(bookDao);
    }

    @Override
    public String toString() {
        return "BookService{" +
                "bookDao=" + bookDao +
                '}';
    }
}

```

测试结果如下：

> BookService{bookDao=BookDao{lable=‘1’}}

------

#### 自动装配-`@Resource`&`@Inject`

2）Spring还支持使用`@Resource`(JSR250)和`@Inject`(JSR330)
\- （1）`@Resource`：可以和`@Autowired`一样实现自动的装配，默认是按照组件的名称来进行装配,没有支持`@Primary`
也没有支持和`@Autowired(required = false)`一样的功能
\- （2）`@Inject`：需要导入javax.inject的包,和`@Autowired`的功能一样,没有支持和`@Autowired(required = false)`一样的功能

`AutowiredAnnotationBeanPostProcessor`是用来解析完成自动装配的功能的

`@Autowired`：是Spring定义的
`@Resource` 和 `@Inject`都是java的规范

------

- `@Resource` 的使用

```java
@Service
public class BookService {

    @Resource
    private BookDao bookDao;

    public void print() {
        System.out.println(bookDao);
    }

    @Override
    public String toString() {
        return "BookService{" +
                "bookDao=" + bookDao +
                '}';
    }
}

```

我们来运行测试方法:

```java
    @Test
    public void test01() {
        //1.创建IOC容器
        AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfigOfAutowired.class);
        BookService bookService = applicationContext.getBean(BookService.class);
        System.out.println(bookService);
    }

```

运行结果如下：

> BookService{bookDao=BookDao{lable=‘1’}}

我们也可以用`@Resource`注解里面的name属性来指定装配哪一个：

```java
@Service
public class BookService {

    @Resource(name = "bookDao2")
    private BookDao bookDao;

    public void print() {
        System.out.println(bookDao);
    }

    @Override
    public String toString() {
        return "BookService{" +
                "bookDao=" + bookDao +
                '}';
    }
}

```

这个时候的测试结果如下：

> BookService{bookDao=BookDao{lable=‘2’}}

------

- `@Inject`的使用
  导入jar包：

```xml
     <!-- https://mvnrepository.com/artifact/javax.inject/javax.inject -->
     <dependency>
         <groupId>javax.inject</groupId>
         <artifactId>javax.inject</artifactId>
         <version>1</version>
     </dependency>

```

我们就可以这样来使用：

```java
@Service
public class BookService {

    @Inject
    private BookDao bookDao;

    public void print() {
        System.out.println(bookDao);
    }

    @Override
    public String toString() {
        return "BookService{" +
                "bookDao=" + bookDao +
                '}';
    }
}

```

我们可以来进行测试：发现也是可以支持`@Primary`的功能的

> BookService{bookDao=BookDao{lable=‘2’}}

#### 自动装配-方法、构造器位置的自动装配

我们从`@Autowired`这个注解点进去看一下源码：
我们可以发现这个注解可以标注的位置有：

- 构造器，
- 参数，
- 方法，
- 属性；
- 缺省
  - 构造器只有有参数构造器时
  - @Bean的参数

都是从容器中来获取参数组件的值

```java
@Target({ElementType.CONSTRUCTOR, ElementType.METHOD, ElementType.PARAMETER, ElementType.FIELD, ElementType.ANNOTATION_TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Autowired {

	/**
	 * Declares whether the annotated dependency is required.
	 * <p>Defaults to {@code true}.
	 */
	boolean required() default true;

}

```

1. `@Autowired`注解标注在方法上：用的最多的方式就是在`@Bean`注解标注的方法的参数，这个参数就是会从容器中获取，在这个方法的参数前面可以加上`@Autowired`注解，也可以省略，默认是不写`@Autowired`，都能自动装配；

```java
@Component
public class Boss {

    private Car car;

    public Car getCar() {
        return car;
    }

    @Autowired //标注在方法上，Spring容器在创建当前对象的时候，就会调用当前方法完成赋值；
    //方法使用的参数，自定义类型的值从IOC容器里面进行获取
    public void setCar(Car car) {
        this.car = car;
    }

    @Override
    public String toString() {
        return "Boss{" +
                "car=" + car +
                '}';
    }
}

```

我们来进行测试：

```java
    @Test
    public void test01() {
        //1.创建IOC容器
        AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfigOfAutowired.class);
        Boss boss = applicationContext.getBean(Boss.class);
        Car car = applicationContext.getBean(Car.class);
        System.out.println(car);
        System.out.println(boss);
    }

```

测试结果如下：

> com.ldc.bean.Car@69930714
> Boss{car=com.ldc.bean.Car@69930714}

1. `@Autowired`注解标注在构造器上，默认加在IOC容器中的组件，容器启动的时候会调用无参构造器创建对象，再进行初始化赋值操作，构造器要用的组件，也都是从容器中来获取：
   注意：如果组件只有一个有参的构造器，这个有参的构造器的 `@Autowired`注解可以省略，参数位置的组件还是可以自动从容器中获取

```java
//默认加在IOC容器中的组件，容器启动的时候会调用无参构造器创建对象，再进行初始化赋值操作
@Component
public class Boss {

    private Car car;

    //构造器要用的组件，也都是从容器中来获取
    @Autowired
    public Boss(Car car) {
        this.car = car;
        System.out.println("Boss的有参构造器"+car);
    }

    public Car getCar() {
        return car;
    }

    @Autowired //标注在方法上，Spring容器在创建当前对象的时候，就会调用当前方法完成赋值；
    //方法使用的参数，自定义类型的值从IOC容器里面进行获取
    public void setCar(Car car) {
        this.car = car;
    }

    @Override
    public String toString() {
        return "Boss{" +
                "car=" + car +
                '}';
    }
}

```

测试结果：

> Boss的有参构造器com.ldc.bean.Car@1460a8c0
> com.ldc.bean.Car@1460a8c0
> Boss{car=com.ldc.bean.Car@1460a8c0}

1. `@Autowired`注解标注在参数上：效果是一样的

```java
//默认加在IOC容器中的组件，容器启动的时候会调用无参构造器创建对象，再进行初始化赋值操作
@Component
public class Boss {

    private Car car;

    //构造器要用的组件，也都是从容器中来获取

    //我们也可以标注在参数上效果是一样的
    public Boss(@Autowired Car car) {
        this.car = car;
        System.out.println("Boss的有参构造器"+car);
    }

    public Car getCar() {
        return car;
    }

    @Autowired //标注在方法上，Spring容器在创建当前对象的时候，就会调用当前方法完成赋值；
    //方法使用的参数，自定义类型的值从IOC容器里面进行获取
    public void setCar(Car car) {
        this.car = car;
    }

    @Override
    public String toString() {
        return "Boss{" +
                "car=" + car +
                '}';
    }
}

```

------

还一种情况，如果Boss这个类里面只有一个有参构造器，在构造器里面不加`@Autowired`注解也是可以的：

```java
//默认加在IOC容器中的组件，容器启动的时候会调用无参构造器创建对象，再进行初始化赋值操作
@Component
public class Boss {

    private Car car;

    //构造器要用的组件，也都是从容器中来获取

    //我们也可以标注在参数上效果是一样的
    public Boss(Car car) {
        this.car = car;
        System.out.println("Boss的有参构造器"+car);
    }

    public Car getCar() {
        return car;
    }

    @Autowired //标注在方法上，Spring容器在创建当前对象的时候，就会调用当前方法完成赋值；
    //方法使用的参数，自定义类型的值从IOC容器里面进行获取
    public void setCar(Car car) {
        this.car = car;
    }

    @Override
    public String toString() {
        return "Boss{" +
                "car=" + car +
                '}';
    }
}

```

------

还有一种缺省用法：
现在有一个Color类，里面有一个Car属性：

```java
public class Color {
    private Car car;

    public Car getCar() {
        return car;
    }

    public void setCar(Car car) {
        this.car = car;
    }

    @Override
    public String toString() {
        return "Color{" +
                "car=" + car +
                '}';
    }
}

```

我们在配置类里面来进行配置：

```java
@ComponentScan({"com.ldc.service","com.ldc.dao","com.ldc.controller","com.ldc.bean"})
public class MainConfigOfAutowired {
    @Primary
    @Bean("bookDao2")
    public BookDao bookDao() {
        BookDao bookDao = new BookDao();
        bookDao.setLable("2");
        return bookDao;
    }

    //@Bean标注的方法创建对象的时候，方法参数的值从容器中获取
    @Bean
    public Color color(Car car) {
        Color color = new Color();
        color.setCar(car);
        return color;
    }

}

```

我们来测试一下:

```java
@Test
    public void test01() {
        //1.创建IOC容器
        AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfigOfAutowired.class);
        Boss boss = applicationContext.getBean(Boss.class);
        Car car = applicationContext.getBean(Car.class);
        System.out.println(car);
        System.out.println(boss);
        Color color = applicationContext.getBean(Color.class);
        System.out.println(color);
    }

```

测试结果：

> com.ldc.bean.Car@6e75aa0d
> Boss{car=com.ldc.bean.Car@6e75aa0d}
> Color{car=com.ldc.bean.Car@6e75aa0d}

#### 自动装配-Aware注入Spring底层组件&原理

4）自定义组件想要使用Spring容器底层的一些组件（ApplicationContext、BeanFactory…）
自定义组件实现xxxAware接口就可以实现，在创建对象的时候，会调用接口规定的方法注入相关的组件;
把Spring底层的一些组件注入到自定义的bean中；

xxxAware等这些都是利用后置处理器的机制，比如ApplicationContextAware 是通过ApplicationContextAwareProcessor来进行处理的；

例如之前写的这个,实现ApplicationContextAware 接口，里面有一个setApplicationContext方法：

```java
@Component
public class Dog implements ApplicationContextAware {

    private ApplicationContext applicationContext;

    public Dog() {
        System.out.println("Dog...Contructor...");
    }

    //在对象创建并赋值之后调用
    @PostConstruct
    public void init() {
        System.out.println("Dog...@PostConstruct...");
    }

    //在对象创建并赋值之后调用
    @PreDestroy
    public void destroy() {
        System.out.println("Dog...@PreDestroy...");
    }

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        this.applicationContext = applicationContext;
    }
}

```

------

Aware 是一个总接口：

```java
/**
 * Marker superinterface indicating that a bean is eligible to be
 * notified by the Spring container of a particular framework object
 * through a callback-style method. Actual method signature is
 * determined by individual subinterfaces, but should typically
 * consist of just one void-returning method that accepts a single
 * argument.
 *
 * <p>Note that merely implementing {@link Aware} provides no default
 * functionality. Rather, processing must be done explicitly, for example
 * in a {@link org.springframework.beans.factory.config.BeanPostProcessor BeanPostProcessor}.
 * Refer to {@link org.springframework.context.support.ApplicationContextAwareProcessor}
 * and {@link org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory}
 * for examples of processing {@code *Aware} interface callbacks.
 *
 * @author Chris Beams
 * @since 3.1
 */
public interface Aware {

}

```

它有这么多的子接口：
![在这里插入图片描述](Spring%E6%B3%A8%E8%A7%A3%E9%A9%B1%E5%8A%A8%E5%BC%80%E5%8F%91.assets/20190115151732737.png)

------

我们来挑几个看看：

```java
@Component
public class Red implements ApplicationContextAware, BeanNameAware , EmbeddedValueResolverAware {

    private ApplicationContext applicationContext;

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        //如果我们后来要用，我们就用一个变量来存起来
        System.out.println("传入的IOC容器："+applicationContext);
        this.applicationContext = applicationContext;
    }

    @Override
    public void setBeanName(String name) {
        System.out.println("当前bean的名字："+name);
    }

    @Override
    public void setEmbeddedValueResolver(StringValueResolver resolver) {
        String resolveStringValue = resolver.resolveStringValue("你好${os.name} 我是#{20*18}");
        System.out.println("解析的字符串"+resolveStringValue);
    }
}

```

这个时候，我们再来测试：

```java
    @Test
    public void test01() {
        //1.创建IOC容器
        AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfigOfAutowired.class);
    }

```

测试结果：

> 当前bean的名字：red
> 解析的字符串你好Windows 7 我是360
> 传入的IOC容器：org.springframework.context.annotation.AnnotationConfigApplicationContext@4141d797: startup date [Tue Jan 15 15:29:08 CST 2019]; root of context hierarchy

------

#### 自动装配-`@Profile`环境搭建

`@Profile`注解源码：

```java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Conditional(ProfileCondition.class)
public @interface Profile {

	/**
	 * The set of profiles for which the annotated component should be registered.
	 */
	String[] value();

}

```

引入数据源和mysql驱动：

```xml
      <!--数据源-->
      <!-- https://mvnrepository.com/artifact/c3p0/c3p0 -->
      <dependency>
          <groupId>c3p0</groupId>
          <artifactId>c3p0</artifactId>
          <version>0.9.1.2</version>
      </dependency>
      <!--数据库驱动-->
      <!-- https://mvnrepository.com/artifact/mysql/mysql-connector-java -->
      <dependency>
          <groupId>mysql</groupId>
          <artifactId>mysql-connector-java</artifactId>
          <version>5.1.44</version>
      </dependency>

```

再写一个dbconfig.properties

```properties
db.user=root
db.password=12358
db.driverClass=com.mysql.jdbc.Driver

```

这个时候，我们就可以这样来进行配置：记得加上`@PropertySource("classpath:/dbconfig.properties")`告诉配置文件的位置

```java
/**
 * Profile:
 *      Spring为我们提供的可以根据当前的环境，动态的激活和切换一系列组件的功能；
 * 开发环境，测试环境，生产环境
 * 我们以切换数据源为例：
 * 数据源：开发环境中(用的是A数据库)、测试环境(用的是B数据库)、而生产环境（用的又是C数据库）
 */
@Configuration
@PropertySource("classpath:/dbconfig.properties")
public class MainConfigOfProfile implements EmbeddedValueResolverAware {

    @Value("${db.user}")
    private String user;

    private StringValueResolver resolver;

    private String driverClass;

    @Bean("testDataSource")
    public DataSource dataSourceTest(@Value("${db.password}") String pwd) throws PropertyVetoException {
        ComboPooledDataSource dataSource = new ComboPooledDataSource();
        dataSource.setUser(user);
        dataSource.setPassword(pwd);
        dataSource.setJdbcUrl("jdbc:mysql://localhost:3306/test");
        dataSource.setDriverClass(driverClass);
        return dataSource;
    }
    @Bean("devDataSource")
    public DataSource dataSourceDev(@Value("${db.password}") String pwd) throws PropertyVetoException {
        ComboPooledDataSource dataSource = new ComboPooledDataSource();
        dataSource.setUser(user);
        dataSource.setPassword(pwd);
        dataSource.setJdbcUrl("jdbc:mysql://localhost:3306/dev");
        dataSource.setDriverClass(driverClass);
        return dataSource;
    }
    @Bean("prodDataSource")
    public DataSource dataSourceProd(@Value("${db.password}") String pwd) throws PropertyVetoException {
        ComboPooledDataSource dataSource = new ComboPooledDataSource();
        dataSource.setUser(user);
        dataSource.setPassword(pwd);
        dataSource.setJdbcUrl("jdbc:mysql://localhost:3306/prod");
        dataSource.setDriverClass(driverClass);
        return dataSource;
    }

    @Override
    public void setEmbeddedValueResolver(StringValueResolver resolver) {
        this.resolver = resolver;
        driverClass = resolver.resolveStringValue("${db.driverClass}");
    }
}

```

上面涉及到三种获取配置文件中的值：

1. 直接通过属性上面加上`@Value("${db.user}")`

```java
    @Value("${db.user}")
    private String user;

```

1. 在参数上面使用`@Value("${db.password}")` public DataSource dataSourceTest(@Value("${db.password}") String pwd)

```java
 	@Bean("testDataSource")
    public DataSource dataSourceTest(@Value("${db.password}") String pwd) throws PropertyVetoException {
        ComboPooledDataSource dataSource = new ComboPooledDataSource();
        dataSource.setUser(user);
        dataSource.setPassword(pwd);
        dataSource.setJdbcUrl("jdbc:mysql://localhost:3306/test");
        dataSource.setDriverClass(driverClass);
        return dataSource;
    }

```

1. 实现EmbeddedValueResolverAware接口，在setEmbeddedValueResolver(StringValueResolver resolver)方法里面进行获取：里面使用 resolver.resolveStringValue("${db.driverClass}")方法解析，返回赋值给成员属性private StringValueResolver resolver;

```
    @Override
    public void setEmbeddedValueResolver(StringValueResolver resolver) {
        this.resolver = resolver;
        driverClass = resolver.resolveStringValue("${db.driverClass}");
    }

```

这个时候，我们来测试一下，看看容器里面DataSource类型的bean有哪些：

```java
    @Test
    public void test01() {
        ApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfigOfProfile.class);
        String[] beanNamesForType = applicationContext.getBeanNamesForType(DataSource.class);
        Stream.of(beanNamesForType).forEach(System.out::println);
    }

```

测试结果：

> testDataSource
> devDataSource
> prodDataSource

#### 自动装配-`@Profile`根据环境注册bean

`@Profile`:
Spring为我们提供的可以根据当前的环境，动态的激活和切换一系列组件的功能；
开发环境，测试环境，生产环境
我们以切换数据源为例：
数据源：开发环境中(用的是A数据库)、测试环境(用的是B数据库)、而生产环境（用的又是C数据库）

`@Profile`: 指定组件在哪一个环境的情况下才能被注册到容器中，不指定任何环境都能被注册这个组件
1）加了环境标识的bean，只有这个环境被激活的时候才能注册到容器中，默认是default环境，如果指定了default，那么这个bean默认会被注册到容器中
2）`@Profile` 写在配置类上，只有是指定的环境，整个配置类里面的所有配置才能开始生效
3）没有标注环境标识的bean，在任何环境都是加载的

现在，我来用`@Profile`注解只激活哪一个数据源：

```java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Conditional(ProfileCondition.class)
public @interface Profile {

	/**
	 * The set of profiles for which the annotated component should be registered.
	 */
	String[] value();

}

```

------

现在，我们来给数据源加上标识：

```java
/**
 * @Profile:
 *      Spring为我们提供的可以根据当前的环境，动态的激活和切换一系列组件的功能；
 * 开发环境，测试环境，生产环境
 * 我们以切换数据源为例：
 * 数据源：开发环境中(用的是A数据库)、测试环境(用的是B数据库)、而生产环境（用的又是C数据库）
 * @Profile: 指定组件在哪一个环境的情况下才能被注册到容器中，不指定任何环境都能被注册这个组件
 * 1）加了环境标识的bean，只有这个环境被激活的时候才能注册到容器中，默认是default环境，如果指定了
 * default，那么这个bean默认会被注册到容器中
 * 2）@Profile 写在配置类上，只有是指定的环境，整个配置类里面的所有配置才能开始生效
 * 3）没有标注环境标识的bean，在任何环境都是加载的
 */
@Configuration
@PropertySource("classpath:/dbconfig.properties")
public class MainConfigOfProfile implements EmbeddedValueResolverAware {

    @Value("${db.user}")
    private String user;

    private StringValueResolver resolver;

    private String driverClass;

    @Profile("test")
    @Bean
    public Yellow yellow() {
        return new Yellow();
    }

    @Profile("test")
    @Bean("testDataSource")
    public DataSource dataSourceTest(@Value("${db.password}") String pwd) throws PropertyVetoException {
        ComboPooledDataSource dataSource = new ComboPooledDataSource();
        dataSource.setUser(user);
        dataSource.setPassword(pwd);
        dataSource.setJdbcUrl("jdbc:mysql://localhost:3306/test");
        dataSource.setDriverClass(driverClass);
        return dataSource;
    }

    @Profile("dev")
    @Bean("devDataSource")
    public DataSource dataSourceDev(@Value("${db.password}") String pwd) throws PropertyVetoException {
        ComboPooledDataSource dataSource = new ComboPooledDataSource();
        dataSource.setUser(user);
        dataSource.setPassword(pwd);
        dataSource.setJdbcUrl("jdbc:mysql://localhost:3306/dev");
        dataSource.setDriverClass(driverClass);
        return dataSource;
    }

    @Profile("prod")
    @Bean("prodDataSource")
    public DataSource dataSourceProd(@Value("${db.password}") String pwd) throws PropertyVetoException {
        ComboPooledDataSource dataSource = new ComboPooledDataSource();
        dataSource.setUser(user);
        dataSource.setPassword(pwd);
        dataSource.setJdbcUrl("jdbc:mysql://localhost:3306/prod");
        dataSource.setDriverClass(driverClass);
        return dataSource;
    }

    @Override
    public void setEmbeddedValueResolver(StringValueResolver resolver) {
        this.resolver = resolver;
        driverClass = resolver.resolveStringValue("${db.driverClass}");
    }
}

```

现在，我们去测试方法里面去指定用哪一个：

1. 使用命令行动态参数的方式：在虚拟机参数的位置加载-Dspring.profile.active=test
   ![在这里插入图片描述](Spring%E6%B3%A8%E8%A7%A3%E9%A9%B1%E5%8A%A8%E5%BC%80%E5%8F%91.assets/20190115175422414.png)

------

这个时候，我们再来运行这个测试方法：

```java
    //1.使用命令行动态参数的方式
    @Test
    public void test01() {
        ApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfigOfProfile.class);
        String[] beanNamesForType = applicationContext.getBeanNamesForType(DataSource.class);
        Stream.of(beanNamesForType).forEach(System.out::println);
    }

```

运行结果：

> testDataSource

------

我们再切换到开发的环境来试试：
![在这里插入图片描述](Spring%E6%B3%A8%E8%A7%A3%E9%A9%B1%E5%8A%A8%E5%BC%80%E5%8F%91.assets/20190115175601330.png)

------

这个时候的运行结果为：

> devDataSource

1. 利用代码的方式来实现激活某种环境，这个时候，我们在创建IOC容器的时候，必须要用无参构造器：

```java
    @Test
    public void test01() {
        AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext();
        //1)使用无参构造器来创建applicationContext对象
        //2)设置需要激活的环境
        applicationContext.getEnvironment().setActiveProfiles("dev");
        //3)加载主配置类
        applicationContext.register(MainConfigOfProfile.class);
        //4)启动刷新容器
        applicationContext.refresh();

        String[] beanNamesForType = applicationContext.getBeanNamesForType(DataSource.class);
        Stream.of(beanNamesForType).forEach(System.out::println);
    }

```

运行结果：

> devDataSource

------

`@Profile`写在配置类上，只有是指定的环境，整个配置类里面的所有配置才能开始生效:

```java
@Profile("test")
@Configuration
@PropertySource("classpath:/dbconfig.properties")
public class MainConfigOfProfile implements EmbeddedValueResolverAware {

    @Value("${db.user}")
    private String user;

    private StringValueResolver resolver;

    private String driverClass;

    @Profile("test")
    @Bean
    public Yellow yellow() {
        return new Yellow();
    }


    @Bean("testDataSource")
    public DataSource dataSourceTest(@Value("${db.password}") String pwd) throws PropertyVetoException {
        ComboPooledDataSource dataSource = new ComboPooledDataSource();
        dataSource.setUser(user);
        dataSource.setPassword(pwd);
        dataSource.setJdbcUrl("jdbc:mysql://localhost:3306/test");
        dataSource.setDriverClass(driverClass);
        return dataSource;
    }

    @Profile("dev")
    @Bean("devDataSource")
    public DataSource dataSourceDev(@Value("${db.password}") String pwd) throws PropertyVetoException {
        ComboPooledDataSource dataSource = new ComboPooledDataSource();
        dataSource.setUser(user);
        dataSource.setPassword(pwd);
        dataSource.setJdbcUrl("jdbc:mysql://localhost:3306/dev");
        dataSource.setDriverClass(driverClass);
        return dataSource;
    }

    @Profile("prod")
    @Bean("prodDataSource")
    public DataSource dataSourceProd(@Value("${db.password}") String pwd) throws PropertyVetoException {
        ComboPooledDataSource dataSource = new ComboPooledDataSource();
        dataSource.setUser(user);
        dataSource.setPassword(pwd);
        dataSource.setJdbcUrl("jdbc:mysql://localhost:3306/prod");
        dataSource.setDriverClass(driverClass);
        return dataSource;
    }

    @Override
    public void setEmbeddedValueResolver(StringValueResolver resolver) {
        this.resolver = resolver;
        driverClass = resolver.resolveStringValue("${db.driverClass}");
    }
}

```

我们再来运行这个测试方法：

```java
    @Test
    public void test01() {
        AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext();
        //1)使用无参构造器来创建applicationContext对象
        //2)设置需要激活的环境
        applicationContext.getEnvironment().setActiveProfiles("dev");
        //3)加载主配置类
        applicationContext.register(MainConfigOfProfile.class);
        //4)启动刷新容器
        applicationContext.refresh();

        String[] beanNamesForType = applicationContext.getBeanNamesForType(DataSource.class);
        Stream.of(beanNamesForType).forEach(System.out::println);
    }

```

这个时候，就没有一个配置是生效的；

------

### IOC小结

**容器**：

- AnnotationConfigApplicationContext：

  - 配置类
  - 包扫描

- 组件添加：

  - @ComponentScan
  - @Bean

  1. 指定初始化销毁
  2. 初始化其他方式
     （1）InitializingBean（初始化设置值之后）
     （2）InitializingBean（初始化设置值之后）
     （3）JSR250：@PostConstruct、@PreDestroy
  3. BeanPostProcessor

  - @Configuration
  - @Component
  - @Service
  - @Controller
  - @Repository
  - @Conditional
  - @Primary
  - @Lazy
  - @Scope
  - @Import
  - ImportSelector
  - 工厂模式
    FactoryBean：&beanName获取Factory本身

- 组件赋值

  - @Value
  - @Autowired
    （1）@Qualifier
    （2）@Resources（JSR250）
    （3）@Inject（JSR330，需要导入javax.inject）
  - @PropertySource
  - @PropertySources
  - @Profile
    （1）Environment
    （2）-Dspring.profiles.active=test

- 组件注入

  - 方法参数
  - 构造器注入
  - xxxAware
  - ApplicationContextAware
    ApplicationContextAwareProcessor

```java
if (bean instanceof Aware) {
			if (bean instanceof EnvironmentAware) {
				((EnvironmentAware) bean).setEnvironment(this.applicationContext.getEnvironment());
			}
			if (bean instanceof EmbeddedValueResolverAware) {
				((EmbeddedValueResolverAware) bean).setEmbeddedValueResolver(this.embeddedValueResolver);
			}
			if (bean instanceof ResourceLoaderAware) {
				((ResourceLoaderAware) bean).setResourceLoader(this.applicationContext);
			}
			if (bean instanceof ApplicationEventPublisherAware) {
				((ApplicationEventPublisherAware) bean).setApplicationEventPublisher(this.applicationContext);
			}
			if (bean instanceof MessageSourceAware) {
				((MessageSourceAware) bean).setMessageSource(this.applicationContext);
			}
			if (bean instanceof ApplicationContextAware) {
				((ApplicationContextAware) bean).setApplicationContext(this.applicationContext);
			}
		}
```





## AOP

### AOP-AOP功能测试

AOP:【动态代理】
能在程序运行期间动态的将某段代码片段切入到指定的方法指定位置进行运行的编程方式；

- 1、导入aop模块，Spring AOP：(spring-aspects)
- 2、定义一个业务逻辑类（MathCalculator），在业务逻辑运行的时候将日志进行打印（方法之前、方法运行结束、包括方法出现异常等等）
- 3、定义一个日志切面类（LogAspects）：切面类里面的方法需要动态感知MathCalculator.div运行到哪里了然后执行
  切面类里面的方法就是通知方法：
  （1）前置通知（`@Before`）：logStart，在目标方法（div）运行之前运行
  （2）后置通知（`@After`）：logEnd，在目标方法（div）运行之前运行
  （3）返回通知（`@AfterReturning`）：logReturn，在目标方法（div）执行返回（无论是正常返回还是异常返回）之后运行
  （4）异常通知（`@AfterThrowing`）：logException，在目标方法（div）出现异常之后运行
  （5）环绕通知（`@Around`）：动态代理，手动推进目标方法运行（joinPoint.procced()）
- 4、给切面类的目标方法标注何时何地运行（通知注解）
- 5、将切面类和业务逻辑类（目标方法所在的类）都加入到容器中；
- 6、必须要告诉Spring哪一个类是切面类（只要给切面类上加上一个注解：`@Aspect`）
- 【7】、给配置类中加入`@EnableAspectJAutoProxy`：开启基于注解的aop模式

加入AOP核心jar包：

```xml
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-aspects</artifactId>
        <version>4.3.12.RELEASE</version>
    </dependency>

```

首先，我们要写一个目标类：

```java
public class MathCalculator {
    //除法
    public int div(int i, int j) {
        System.out.println("MathCalculator...div...");
        return i / j;
    }
}

```

再来一个切面类：

```java
//告诉Spring当前类是一个切面类
@Aspect
public class LogAspects {

    //抽取公共的切入点表达式
    //本类引用：pointCut()
    //其他的切面类要引用
    @Pointcut("execution(public int com.ldc.aop.MathCalculator.*(..))")
    public void pointCut() {

    }

    //@Before在目标方法之前切入：切入点表达式（指定在哪个方法切入）
    @Before("pointCut()")
    public void logStart() {
        System.out.println("除法运行...参数列表是：{}");
    }

    @After("pointCut()")
    public void logEnd() {
        System.out.println("除法结束...");
    }

    @AfterReturning("pointCut()")
    public void logReturn() {
        System.out.println("除法正常返回...运行结果为：{}");
    }
    @AfterThrowing("pointCut()")
    public void logException() {
        System.out.println("除法异常...异常信息为：{}");
    }
}

```

我们再来写一个配置类 ，把上面这两个类加入到IOC容器中，并且通过`@EnableAspectJAutoProxy`注解开启基于注解的aop模式

```java
/**
 *  AOP:【动态代理】
 *      能在程序运行期间动态的将某段代码片段切入到指定的方法指定位置进行运行的编程方式；
 *
 *  1、导入aop模块，Spring AOP：(spring-aspects)
 *  2、定义一个业务逻辑类（MathCalculator），在业务逻辑运行的时候将日志进行打印（方法之前、方法运行结束、包括方法出现异常等等）
 *  3、定义一个日志切面类（LogAspects）：切面类里面的方法需要动态感知MathCalculator.div运行到哪里了然后执行
 *      切面类里面的方法就是通知方法：
 *      （1）前置通知（@Before）：logStart，在目标方法（div）运行之前运行
 *      （2）后置通知（@After）：logEnd，在目标方法（div）运行之前运行
 *      （3）返回通知（@AfterReturning）：logReturn，在目标方法（div）执行返回（无论是正常返回还是异常返回）之后运行
 *      （4）异常通知（@AfterThrowing）：logException，在目标方法（div）出现异常之后运行
 *      （5）环绕通知（@Around）：动态代理，手动推进目标方法运行（joinPoint.procced()）
 *  4、给切面类的目标方法标注何时何地运行（通知注解）
 *  5、将切面类和业务逻辑类（目标方法所在的类）都加入到容器中；
 *  6、必须要告诉Spring哪一个类是切面类（只要给切面类上加上一个注解：@Aspect）
 *  【7】、给配置类中加入@EnableAspectJAutoProxy：开启基于注解的aop模式
 */
@EnableAspectJAutoProxy
@Configuration
public class MainConfigOfAOP {

    //将业务逻辑类加入到容器中
    @Bean
    public MathCalculator mathCalculator() {
        return new MathCalculator();
    }

    //将切面类加入到容器中
    @Bean
    public LogAspects logAspects() {
        return new LogAspects();
    }
}

```

测试方法：

```java
@Test
    public void test01() {
        AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfigOfAOP.class);
        //1.不要自己创建对象
        //MathCalculator mathCalculator = new MathCalculator();
        //mathCalculator.div(1, 1);
        //我们要中容器中获取组件
        MathCalculator mathCalculator = applicationContext.getBean(MathCalculator.class);
        mathCalculator.div(1, 1);
    }

```

运行结果为：

> 除法运行…参数列表是：{}
> MathCalculator…div…
> 除法结束…
> 除法正常返回…运行结果为：{}

------

我们可以在切面里面的通知方法里面获取方法名、参数、返回值以及异常等等：

```java
//告诉Spring当前类是一个切面类
@Aspect
public class LogAspects {

    //抽取公共的切入点表达式
    //本类引用：pointCut()
    //其他的切面类要引用
    @Pointcut("execution(public int com.ldc.aop.MathCalculator.*(..))")
    public void pointCut() {

    }

    //@Before在目标方法之前切入：切入点表达式（指定在哪个方法切入）
    @Before("pointCut()")
    public void logStart(JoinPoint joinPoint) {
        System.out.println(joinPoint.getSignature().getName()+"除法运行...参数列表是：{"+ Arrays.asList(joinPoint.getArgs()) +"}");
    }

    @After("pointCut()")
    public void logEnd(JoinPoint joinPoint) {
        System.out.println(joinPoint.getSignature().getName()+"除法结束...");
    }

    @AfterReturning(value = "pointCut()",returning = "result")
    public void logReturn(Object result) {
        System.out.println("除法正常返回...运行结果为：{"+result+"}");
    }
	
	//JoinPoint这个参数一定要出现在参数列表的第一位
    @AfterThrowing(value = "pointCut()",throwing = "exception")
    public void logException(JoinPoint joinPoint,Exception exception) {
        System.out.println(joinPoint.getSignature().getName()+"除法异常...异常信息为：{}");
    }
}

```

------

**我们要注意：如果我们要写JoinPoint这个参数，那么这个参数一定要写在参数列表的第一位；**

------

主要把握三步：
一、将业务逻辑组件和切面类都加入到IOC容器中，并且告诉Spring哪一个是切面类（`@Aspect`）
二、在切面类上的每一个通知方法标注通知注解：告诉Spring何时何地运行（写好切入点表达式）
三、开启基于注解的AOP模式：`@EnableAspectJAutoProxy`

### AOP-源码原理

##### [源码]-AOP原理-`@EnableAspectJAutoProxy`

```java
  AOP：【动态代理】
  		指在程序运行期间动态的将某段代码切入到指定方法指定位置进行运行的编程方式；

  1、导入aop模块；Spring AOP：(spring-aspects)
  2、定义一个业务逻辑类（MathCalculator）；在业务逻辑运行的时候将日志进行打印（方法之前、方法运行结束、方法出现异常，xxx）
  3、定义一个日志切面类（LogAspects）：切面类里面的方法需要动态感知MathCalculator.div运行到哪里然后执行；
  		通知方法：
  			前置通知(@Before)：logStart：在目标方法(div)运行之前运行
  			后置通知(@After)：logEnd：在目标方法(div)运行结束之后运行（无论方法正常结束还是异常结束）
  			返回通知(@AfterReturning)：logReturn：在目标方法(div)正常返回之后运行
  			异常通知(@AfterThrowing)：logException：在目标方法(div)出现异常以后运行
  			环绕通知(@Around)：动态代理，手动推进目标方法运行（joinPoint.procced()）
  4、给切面类的目标方法标注何时何地运行（通知注解）；
  5、将切面类和业务逻辑类（目标方法所在类）都加入到容器中;
  6、必须告诉Spring哪个类是切面类(给切面类上加一个注解：@Aspect)
  [7]、给配置类中加 @EnableAspectJAutoProxy 【开启基于注解的aop模式】
  		在Spring中很多的 @EnableXXX;

  三步：
  	1）、将业务逻辑组件和切面类都加入到容器中；告诉Spring哪个是切面类（@Aspect）
  	2）、在切面类上的每一个通知方法上标注通知注解，告诉Spring何时何地运行（切入点表达式）
   3）、开启基于注解的aop模式；@EnableAspectJAutoProxy

  AOP原理：【看给容器中注册了什么组件，这个组件什么时候工作，这个组件的功能是什么？】
  		@EnableAspectJAutoProxy；
  1、@EnableAspectJAutoProxy是什么？
  		@Import(AspectJAutoProxyRegistrar.class)：给容器中导入AspectJAutoProxyRegistrar
  			利用AspectJAutoProxyRegistrar自定义给容器中注册bean；BeanDefinetion
  			internalAutoProxyCreator=AnnotationAwareAspectJAutoProxyCreator

  		给容器中注册一个AnnotationAwareAspectJAutoProxyCreator；

  2、 AnnotationAwareAspectJAutoProxyCreator：
  		AnnotationAwareAspectJAutoProxyCreator
  			->AspectJAwareAdvisorAutoProxyCreator
  				->AbstractAdvisorAutoProxyCreator
  					->AbstractAutoProxyCreator
  							implements SmartInstantiationAwareBeanPostProcessor, BeanFactoryAware
  						关注后置处理器（在bean初始化完成前后做事情）、自动装配BeanFactory

  AbstractAutoProxyCreator.setBeanFactory()
  AbstractAutoProxyCreator.有后置处理器的逻辑；

  AbstractAdvisorAutoProxyCreator.setBeanFactory()-》initBeanFactory()

  AnnotationAwareAspectJAutoProxyCreator.initBeanFactory()


  流程：
  		1）、传入配置类，创建ioc容器
  		2）、注册配置类，调用refresh（）刷新容器；
  		3）、registerBeanPostProcessors(beanFactory);注册bean的后置处理器来方便拦截bean的创建；
  			1）、先获取ioc容器已经定义了的需要创建对象的所有BeanPostProcessor
  			2）、给容器中加别的BeanPostProcessor
  			3）、优先注册实现了PriorityOrdered接口的BeanPostProcessor；
  			4）、再给容器中注册实现了Ordered接口的BeanPostProcessor；
  			5）、注册没实现优先级接口的BeanPostProcessor；
  			6）、注册BeanPostProcessor，实际上就是创建BeanPostProcessor对象，保存在容器中；
  				创建internalAutoProxyCreator的BeanPostProcessor【AnnotationAwareAspectJAutoProxyCreator】
  				1）、创建Bean的实例
  				2）、populateBean；给bean的各种属性赋值
  				3）、initializeBean：初始化bean；
  						1）、invokeAwareMethods()：处理Aware接口的方法回调
  						2）、applyBeanPostProcessorsBeforeInitialization()：应用后置处理器的postProcessBeforeInitialization（）
  						3）、invokeInitMethods()；执行自定义的初始化方法
  						4）、applyBeanPostProcessorsAfterInitialization()；执行后置处理器的postProcessAfterInitialization（）；
  				4）、BeanPostProcessor(AnnotationAwareAspectJAutoProxyCreator)创建成功；--》aspectJAdvisorsBuilder
  			7）、把BeanPostProcessor注册到BeanFactory中；
  				beanFactory.addBeanPostProcessor(postProcessor);
  =======以上是创建和注册AnnotationAwareAspectJAutoProxyCreator的过程========

  			AnnotationAwareAspectJAutoProxyCreator => InstantiationAwareBeanPostProcessor
  		4）、finishBeanFactoryInitialization(beanFactory);完成BeanFactory初始化工作；创建剩下的单实例bean
  			1）、遍历获取容器中所有的Bean，依次创建对象getBean(beanName);
  				getBean->doGetBean()->getSingleton()->
  			2）、创建bean
  				【AnnotationAwareAspectJAutoProxyCreator在所有bean创建之前会有一个拦截，它属于InstantiationAwareBeanPostProcessor后置处理器，会调用postProcessBeforeInstantiation()】
  				1）、先从缓存中获取当前bean，如果能获取到，说明bean是之前被创建过的，直接使用，否则再创建；
  					只要创建好的Bean都会被缓存起来
  				2）、createBean（）;创建bean；
  					AnnotationAwareAspectJAutoProxyCreator 会在任何bean创建之前先尝试返回bean的实例
  					【BeanPostProcessor是在Bean对象创建完成初始化前后调用的】
  					【InstantiationAwareBeanPostProcessor是在创建Bean实例之前先尝试用后置处理器返回代理对象的】
  					1）、resolveBeforeInstantiation(beanName, mbdToUse);解析BeforeInstantiation
  						希望后置处理器在此能返回一个代理对象；如果能返回代理对象就使用，如果不能就继续
  						1）、后置处理器先尝试返回对象；
  							bean = applyBeanPostProcessorsBeforeInstantiation（）：
  								拿到所有后置处理器，如果是
                    			InstantiationAwareBeanPostProcessor;
  								就执行postProcessBeforeInstantiation
                                    
  							if (bean != null) {
                               bean = applyBeanPostProcessorsAfterInitialization(bean, beanName);
                           }

  					2）、doCreateBean(beanName, mbdToUse, args);真正的去创建一个bean实例；和3.6流程一样；
  					3）、


  AnnotationAwareAspectJAutoProxyCreator【InstantiationAwareBeanPostProcessor】	的作用：
  1）、每一个bean创建之前，调用postProcessBeforeInstantiation()；
  		关心MathCalculator和LogAspect的创建
  		1）、判断当前bean是否在advisedBeans中（保存了所有需要增强bean）
  		2）、判断当前bean是否是基础类型的Advice、Pointcut、Advisor、AopInfrastructureBean，
  			或者是否是切面（@Aspect）
  		3）、是否需要跳过
  			1）、获取候选的增强器（切面里面的通知方法）【List<Advisor> candidateAdvisors】
  				每一个封装的通知方法的增强器是 InstantiationModelAwarePointcutAdvisor；
  				判断每一个增强器是否是 AspectJPointcutAdvisor 类型的；返回true
  			2）、永远返回false

  2）、创建对象
  postProcessAfterInitialization；
  		return wrapIfNecessary(bean, beanName, cacheKey);//包装如果需要的情况下
  		1）、获取当前bean的所有增强器（通知方法）  Object[]  specificInterceptors
  			1、找到候选的所有的增强器（找哪些通知方法是需要切入当前bean方法的）
  			2、获取到能在bean使用的增强器。
  			3、给增强器排序
  		2）、保存当前bean在advisedBeans中；（advisedBean已增强的对象）
  		3）、如果当前bean需要增强，创建当前bean的代理对象；
  			1）、获取所有增强器（通知方法）
  			2）、保存到proxyFactory
  			3）、创建代理对象：Spring自动决定
  				JdkDynamicAopProxy(config);jdk动态代理；
  				ObjenesisCglibAopProxy(config);cglib的动态代理；
  		4）、给容器中返回当前组件使用cglib增强了的代理对象；
  		5）、以后容器中获取到的就是这个组件的代理对象，执行目标方法的时候，代理对象就会执行通知方法的流程；


  	3）、目标方法执行	；
  		容器中保存了组件的代理对象（cglib增强后的对象），这个对象里面保存了详细信息（比如增强器，目标对象，xxx）；
  		1）、CglibAopProxy.intercept();拦截目标方法的执行
  		2）、根据ProxyFactory对象获取将要执行的目标方法拦截器链；
  			List<Object> chain = this.advised.getInterceptorsAndDynamicInterceptionAdvice(method, targetClass);
  			1）、List<Object> interceptorList保存所有拦截器 5
  				一个默认的ExposeInvocationInterceptor 和 4个增强器；
  			2）、遍历所有的增强器，将其转为Interceptor；
  				registry.getInterceptors(advisor);
  			3）、将增强器转为List<MethodInterceptor>；
  				如果是MethodInterceptor，直接加入到集合中
  				如果不是，使用AdvisorAdapter将增强器转为MethodInterceptor；
  				转换完成返回MethodInterceptor数组；

  		3）、如果没有拦截器链，直接执行目标方法;
  			拦截器链（每一个通知方法又被包装为方法拦截器，利用MethodInterceptor机制）
  		4）、如果有拦截器链，把需要执行的目标对象，目标方法，
  			拦截器链等信息传入创建一个 CglibMethodInvocation 对象，
  			并调用 Object retVal =  mi.proceed();
  		5）、拦截器链的触发过程;
  			1)、如果没有拦截器执行执行目标方法，或者拦截器的索引和拦截器数组-1大小一样（指定到了最后一个拦截器）执行目标方法；
  			2)、链式获取每一个拦截器，拦截器执行invoke方法，每一个拦截器等待下一个拦截器执行完成返回以后再来执行；
  				拦截器链的机制，保证通知方法与目标方法的执行顺序；

  	总结：
  		1）、  @EnableAspectJAutoProxy 开启AOP功能
  		2）、 @EnableAspectJAutoProxy 会给容器中注册一个组件 AnnotationAwareAspectJAutoProxyCreator
  		3）、AnnotationAwareAspectJAutoProxyCreator是一个后置处理器；
  		4）、容器的创建流程：
  			1）、registerBeanPostProcessors（）注册后置处理器；创建AnnotationAwareAspectJAutoProxyCreator对象
  			2）、finishBeanFactoryInitialization（）初始化剩下的单实例bean
  				1）、创建业务逻辑组件和切面组件
  				2）、AnnotationAwareAspectJAutoProxyCreator拦截组件的创建过程
  				3）、组件创建完之后，判断组件是否需要增强
  					是：切面的通知方法，包装成增强器（Advisor）;给业务逻辑组件创建一个代理对象（cglib）；
  		5）、执行目标方法：
  			1）、代理对象执行目标方法
  			2）、CglibAopProxy.intercept()；
  				1）、得到目标方法的拦截器链（增强器包装成拦截器MethodInterceptor）
  				2）、利用拦截器的链式机制，依次进入每一个拦截器进行执行；
  				3）、效果：
  					正常执行：前置通知-》目标方法-》后置通知-》返回通知
  					出现异常：前置通知-》目标方法-》后置通知-》异常通知

```

------

AOP原理：看给容器中注册了哪些组件，这个组件什么时候工作，这个组件的功能是什么？
我们从`@EnableAspectJAutoProxy`这个注解入手：

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import(AspectJAutoProxyRegistrar.class)
public @interface EnableAspectJAutoProxy {

	/**
	 * Indicate whether subclass-based (CGLIB) proxies are to be created as opposed
	 * to standard Java interface-based proxies. The default is {@code false}.
	 */
	boolean proxyTargetClass() default false;

	/**
	 * Indicate that the proxy should be exposed by the AOP framework as a {@code ThreadLocal}
	 * for retrieval via the {@link org.springframework.aop.framework.AopContext} class.
	 * Off by default, i.e. no guarantees that {@code AopContext} access will work.
	 * @since 4.3.1
	 */
	boolean exposeProxy() default false;

}

```

- 在这个注解源码类里面标注了`@Import(AspectJAutoProxyRegistrar.class)`给容器中导入AspectJAutoProxyRegistrar
  利用了AspectJAutoProxyRegistrar自定义给容器中注册bean，给容器中注册一个AnnotationAwareAspectJAutoProxyCreator这个组件，翻译过来就是【注解装配模式的AspectJ切面自动代理创建器】
- AnnotationAwareAspectJAutoProxyCreator：

```java
class AspectJAutoProxyRegistrar implements ImportBeanDefinitionRegistrar {

	/**
	 * Register, escalate, and configure the AspectJ auto proxy creator based on the value
	 * of the @{@link EnableAspectJAutoProxy#proxyTargetClass()} attribute on the importing
	 * {@code @Configuration} class.
	 */
	@Override
	public void registerBeanDefinitions(
			AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {

		AopConfigUtils.registerAspectJAnnotationAutoProxyCreatorIfNecessary(registry);

		AnnotationAttributes enableAspectJAutoProxy =
				AnnotationConfigUtils.attributesFor(importingClassMetadata, EnableAspectJAutoProxy.class);
		if (enableAspectJAutoProxy.getBoolean("proxyTargetClass")) {
			AopConfigUtils.forceAutoProxyCreatorToUseClassProxying(registry);
		}
		if (enableAspectJAutoProxy.getBoolean("exposeProxy")) {
			AopConfigUtils.forceAutoProxyCreatorToExposeProxy(registry);
		}
	}

}

```

而上面的类实现了这个接口ImportBeanDefinitionRegistrar : 手动注册bean到容器中

```java
public interface ImportBeanDefinitionRegistrar {

	/**
	 * Register bean definitions as necessary based on the given annotation metadata of
	 * the importing {@code @Configuration} class.
	 * <p>Note that {@link BeanDefinitionRegistryPostProcessor} types may <em>not</em> be
	 * registered here, due to lifecycle constraints related to {@code @Configuration}
	 * class processing.
	 * @param importingClassMetadata annotation metadata of the importing class
	 * @param registry current bean definition registry
	 */
	public void registerBeanDefinitions(
			AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry);

}

```

之前我们也过一个这样的类：

```java
public class MyImportBeanDefinitionRegistrar implements ImportBeanDefinitionRegistrar {
    /**
     *
     * AnnotationMetadata 当前类的注解信息
     * BeanDefinitionRegistry BeanDefinition注册类
     *
     * 我们把所有需要添加到容器中的bean通过BeanDefinitionRegistry里面的registerBeanDefinition方法来手动的进行注册
     */
    @Override
    public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
        //判断IOC容器里面是否含有这两个组件
        boolean definition = registry.containsBeanDefinition("com.ldc.bean.Red");
        boolean definition2 = registry.containsBeanDefinition("com.ldc.bean.Blue");
        //如果有的话，我就把RainBow的bean的实例给注册到IOC容器中
        if (definition && definition2) {
            //指定bean的定义信息，参数里面指定要注册的bean的类型：RainBow.class
            RootBeanDefinition rootBeanDefinition = new RootBeanDefinition(RainBow.class);
            //注册一个bean，并且指定bean名
            registry.registerBeanDefinition("rainBow", rootBeanDefinition );
        }
    }
}

```

------

我们在这里打一个断点来debug：
![在这里插入图片描述](Spring%E6%B3%A8%E8%A7%A3%E9%A9%B1%E5%8A%A8%E5%BC%80%E5%8F%91.assets/20190115205817149.png)
我们来debug运行这个测试方法：

```java
    @Test
    public void test01() {
        AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfigOfAOP.class);
        //1.不要自己创建对象
        //MathCalculator mathCalculator = new MathCalculator();
        //mathCalculator.div(1, 1);
        //我们要中容器中获取组件
        MathCalculator mathCalculator = applicationContext.getBean(MathCalculator.class);
        mathCalculator.div(1, 1);
    }

```

------

经过上面的AspectJAutoProxyRegistrar这个类给容器中手动的给容器注册了AnnotationAwareAspectJAutoProxyCreator这个组件，现在我们就来看看这个组件：

```java
public class AnnotationAwareAspectJAutoProxyCreator extends AspectJAwareAdvisorAutoProxyCreator {

	private List<Pattern> includePatterns;

	private AspectJAdvisorFactory aspectJAdvisorFactory;

	private BeanFactoryAspectJAdvisorsBuilder aspectJAdvisorsBuilder;


	/**
	 * Set a list of regex patterns, matching eligible @AspectJ bean names.
	 * <p>Default is to consider all @AspectJ beans as eligible.
	 */
	public void setIncludePatterns(List<String> patterns) {
		this.includePatterns = new ArrayList<Pattern>(patterns.size());
		for (String patternText : patterns) {
			this.includePatterns.add(Pattern.compile(patternText));
		}
	}

	public void setAspectJAdvisorFactory(AspectJAdvisorFactory aspectJAdvisorFactory) {
		Assert.notNull(aspectJAdvisorFactory, "AspectJAdvisorFactory must not be null");
		this.aspectJAdvisorFactory = aspectJAdvisorFactory;
	}

	@Override
	protected void initBeanFactory(ConfigurableListableBeanFactory beanFactory) {
		super.initBeanFactory(beanFactory);
		if (this.aspectJAdvisorFactory == null) {
			this.aspectJAdvisorFactory = new ReflectiveAspectJAdvisorFactory(beanFactory);
		}
		this.aspectJAdvisorsBuilder =
				new BeanFactoryAspectJAdvisorsBuilderAdapter(beanFactory, this.aspectJAdvisorFactory);
	}


	@Override
	protected List<Advisor> findCandidateAdvisors() {
		// Add all the Spring advisors found according to superclass rules.
		List<Advisor> advisors = super.findCandidateAdvisors();
		// Build Advisors for all AspectJ aspects in the bean factory.
		advisors.addAll(this.aspectJAdvisorsBuilder.buildAspectJAdvisors());
		return advisors;
	}

	@Override
	protected boolean isInfrastructureClass(Class<?> beanClass) {
		// Previously we setProxyTargetClass(true) in the constructor, but that has too
		// broad an impact. Instead we now override isInfrastructureClass to avoid proxying
		// aspects. I'm not entirely happy with that as there is no good reason not
		// to advise aspects, except that it causes advice invocation to go through a
		// proxy, and if the aspect implements e.g the Ordered interface it will be
		// proxied by that interface and fail at runtime as the advice method is not
		// defined on the interface. We could potentially relax the restriction about
		// not advising aspects in the future.
		return (super.isInfrastructureClass(beanClass) || this.aspectJAdvisorFactory.isAspect(beanClass));
	}

	/**
	 * Check whether the given aspect bean is eligible for auto-proxying.
	 * <p>If no &lt;aop:include&gt; elements were used then "includePatterns" will be
	 * {@code null} and all beans are included. If "includePatterns" is non-null,
	 * then one of the patterns must match.
	 */
	protected boolean isEligibleAspectBean(String beanName) {
		if (this.includePatterns == null) {
			return true;
		}
		else {
			for (Pattern pattern : this.includePatterns) {
				if (pattern.matcher(beanName).matches()) {
					return true;
				}
			}
			return false;
		}
	}


	/**
	 * Subclass of BeanFactoryAspectJAdvisorsBuilderAdapter that delegates to
	 * surrounding AnnotationAwareAspectJAutoProxyCreator facilities.
	 */
	private class BeanFactoryAspectJAdvisorsBuilderAdapter extends BeanFactoryAspectJAdvisorsBuilder {

		public BeanFactoryAspectJAdvisorsBuilderAdapter(
				ListableBeanFactory beanFactory, AspectJAdvisorFactory advisorFactory) {

			super(beanFactory, advisorFactory);
		}

		@Override
		protected boolean isEligibleBean(String beanName) {
			return AnnotationAwareAspectJAutoProxyCreator.this.isEligibleAspectBean(beanName);
		}
	}

}

```

这个就是这个类的一个继承关系：
![在这里插入图片描述](Spring%E6%B3%A8%E8%A7%A3%E9%A9%B1%E5%8A%A8%E5%BC%80%E5%8F%91.assets/20190115213146734.png)

------

在AspectJAwareAdvisorAutoProxyCreator的父类AbstractAutoProxyCreator上面实现了这个接口：SmartInstantiationAwareBeanPostProcessor还实现了BeanFactoryAware接口（能把BeanFactory工厂传进来的）

------

##### [源码]-AOP原理-AnnotationAwareAspectJAutoProxyCreator分析

我们从AnnotationAwareAspectJAutoProxyCreator的父类AbstractAutoProxyCreator入手：
![在这里插入图片描述](Spring%E6%B3%A8%E8%A7%A3%E9%A9%B1%E5%8A%A8%E5%BC%80%E5%8F%91.assets/20190116095445864.png)

------

我们可以在关于上面两个接口的方法打上断点：
![在这里插入图片描述](Spring%E6%B3%A8%E8%A7%A3%E9%A9%B1%E5%8A%A8%E5%BC%80%E5%8F%91.assets/20190116095520838.png)

------

![在这里插入图片描述](Spring%E6%B3%A8%E8%A7%A3%E9%A9%B1%E5%8A%A8%E5%BC%80%E5%8F%91.assets/20190116095613449.png)

------

AbstractAdvisorAutoProxyCreator类里面：
![在这里插入图片描述](Spring%E6%B3%A8%E8%A7%A3%E9%A9%B1%E5%8A%A8%E5%BC%80%E5%8F%91.assets/20190116100419142.png)

------

AnnotationAwareAspectJAutoProxyCreator类里面：
![在这里插入图片描述](Spring%E6%B3%A8%E8%A7%A3%E9%A9%B1%E5%8A%A8%E5%BC%80%E5%8F%91.assets/20190116100512284.png)

------

我们再给配置类里面的这两个方法添加断点：
![在这里插入图片描述](Spring%E6%B3%A8%E8%A7%A3%E9%A9%B1%E5%8A%A8%E5%BC%80%E5%8F%91.assets/20190116100640250.png)

------

##### [源码]-AOP原理-注册AnnotationAwareAspectJAutoProxyCreator

这个就是一个调用流程：

```java
 流程：
  		1）、传入配置类，创建ioc容器
  		2）、注册配置类，调用refresh（）刷新容器；
  		3）、registerBeanPostProcessors(beanFactory);注册bean的后置处理器来方便拦截bean的创建；
  			1）、先获取ioc容器已经定义了的需要创建对象的所有BeanPostProcessor
  			2）、给容器中加别的BeanPostProcessor
  			3）、优先注册实现了PriorityOrdered接口的BeanPostProcessor；
  			4）、再给容器中注册实现了Ordered接口的BeanPostProcessor；
  			5）、注册没实现优先级接口的BeanPostProcessor；
  			6）、注册BeanPostProcessor，实际上就是创建BeanPostProcessor对象，保存在容器中；
  				创建internalAutoProxyCreator的BeanPostProcessor【AnnotationAwareAspectJAutoProxyCreator】
  				1）、创建Bean的实例
  				2）、populateBean；给bean的各种属性赋值
  				3）、initializeBean：初始化bean；
  						1）、invokeAwareMethods()：处理Aware接口的方法回调
  						2）、applyBeanPostProcessorsBeforeInitialization()：应用后置处理器的postProcessBeforeInitialization（）
  						3）、invokeInitMethods()；执行自定义的初始化方法
  						4）、applyBeanPostProcessorsAfterInitialization()；执行后置处理器的postProcessAfterInitialization（）；
  				4）、BeanPostProcessor(AnnotationAwareAspectJAutoProxyCreator)创建成功；--》aspectJAdvisorsBuilder
  			7）、把BeanPostProcessor注册到BeanFactory中；
  				beanFactory.addBeanPostProcessor(postProcessor);
  =======以上是创建和注册AnnotationAwareAspectJAutoProxyCreator的过程========

```

接着上面，我们来启动测试方法：

```java
    @Test
    public void test01() {
        AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfigOfAOP.class);
        //1.不要自己创建对象
        //MathCalculator mathCalculator = new MathCalculator();
        //mathCalculator.div(1, 1);
        //我们要中容器中获取组件
        MathCalculator mathCalculator = applicationContext.getBean(MathCalculator.class);
        mathCalculator.div(1, 1);
    }

```

第一步：先来到了setBeanFactory的这个方法：
![在这里插入图片描述](Spring%E6%B3%A8%E8%A7%A3%E9%A9%B1%E5%8A%A8%E5%BC%80%E5%8F%91.assets/2019011610092734.png)

------

![在这里插入图片描述](Spring%E6%B3%A8%E8%A7%A3%E9%A9%B1%E5%8A%A8%E5%BC%80%E5%8F%91.assets/20190116101037988.png)

------

##### [源码]-AOP原理-AnnotationAwareAspectJAutoProxyCreator执行时机

```java
  			AnnotationAwareAspectJAutoProxyCreator => InstantiationAwareBeanPostProcessor
  		4）、finishBeanFactoryInitialization(beanFactory);完成BeanFactory初始化工作；创建剩下的单实例bean
  			1）、遍历获取容器中所有的Bean，依次创建对象getBean(beanName);
  				getBean->doGetBean()->getSingleton()->
  			2）、创建bean
  				【AnnotationAwareAspectJAutoProxyCreator在所有bean创建之前会有一个拦截，InstantiationAwareBeanPostProcessor，会调用postProcessBeforeInstantiation()】
  				1）、先从缓存中获取当前bean，如果能获取到，说明bean是之前被创建过的，直接使用，否则再创建；
  					只要创建好的Bean都会被缓存起来
  				2）、createBean（）;创建bean；
  					AnnotationAwareAspectJAutoProxyCreator 会在任何bean创建之前先尝试返回bean的实例
  					【BeanPostProcessor是在Bean对象创建完成初始化前后调用的】
  					【InstantiationAwareBeanPostProcessor是在创建Bean实例之前先尝试用后置处理器返回对象的】
  					1）、resolveBeforeInstantiation(beanName, mbdToUse);解析BeforeInstantiation
  						希望后置处理器在此能返回一个代理对象；如果能返回代理对象就使用，如果不能就继续
  						1）、后置处理器先尝试返回对象；
  							bean = applyBeanPostProcessorsBeforeInstantiation（）：
  								拿到所有后置处理器，如果是
                    			InstantiationAwareBeanPostProcessor;
  								就执行postProcessBeforeInstantiation
  							if (bean != null) {
                               bean = applyBeanPostProcessorsAfterInitialization(bean, beanName);
                           }

  					2）、doCreateBean(beanName, mbdToUse, args);真正的去创建一个bean实例；和3.6流程一样；
  					3）、

```

##### [源码]-AOP原理-创建AOP代理

```java
AnnotationAwareAspectJAutoProxyCreator【InstantiationAwareBeanPostProcessor】	的作用：
  1）、每一个bean创建之前，调用postProcessBeforeInstantiation()；
  		关心MathCalculator和LogAspect的创建
  		1）、判断当前bean是否在advisedBeans中（保存了所有需要增强bean）
  		2）、判断当前bean是否是基础类型的Advice、Pointcut、Advisor、AopInfrastructureBean，
  			或者是否是切面（@Aspect）
  		3）、是否需要跳过
  			1）、获取候选的增强器（切面里面的通知方法）【List<Advisor> candidateAdvisors】
  				每一个封装的通知方法的增强器是 InstantiationModelAwarePointcutAdvisor；
  				判断每一个增强器是否是 AspectJPointcutAdvisor 类型的；返回true
  			2）、永远返回false

  2）、创建对象
  postProcessAfterInitialization；
  		return wrapIfNecessary(bean, beanName, cacheKey);//包装如果需要的情况下
  		1）、获取当前bean的所有增强器（通知方法）  Object[]  specificInterceptors
  			1、找到候选的所有的增强器（找哪些通知方法是需要切入当前bean方法的）
  			2、获取到能在bean使用的增强器。
  			3、给增强器排序
  		2）、保存当前bean在advisedBeans中；
  		3）、如果当前bean需要增强，创建当前bean的代理对象；
  			1）、获取所有增强器（通知方法）
  			2）、保存到proxyFactory
  			3）、创建代理对象：Spring自动决定
  				JdkDynamicAopProxy(config);jdk动态代理；
  				ObjenesisCglibAopProxy(config);cglib的动态代理；
  		4）、给容器中返回当前组件使用cglib增强了的代理对象；
  		5）、以后容器中获取到的就是这个组件的代理对象，执行目标方法的时候，代理对象就会执行通知方法的流程；

```

------

##### [源码]-AOP原理-获取拦截器链-MethodInterceptor

```java
	3）、目标方法执行	；
  		容器中保存了组件的代理对象（cglib增强后的对象），这个对象里面保存了详细信息（比如增强器，目标对象，xxx）；
  		1）、CglibAopProxy.intercept();拦截目标方法的执行
  		2）、根据ProxyFactory对象获取将要执行的目标方法拦截器链；
  			List<Object> chain = this.advised.getInterceptorsAndDynamicInterceptionAdvice(method, targetClass);
  			1）、List<Object> interceptorList保存所有拦截器 5
  				一个默认的ExposeInvocationInterceptor 和 4个增强器；
  			2）、遍历所有的增强器，将其转为Interceptor；
  				registry.getInterceptors(advisor);
  			3）、将增强器转为List<MethodInterceptor>；
  				如果是MethodInterceptor，直接加入到集合中
  				如果不是，使用AdvisorAdapter将增强器转为MethodInterceptor；
  				转换完成返回MethodInterceptor数组；

  		3）、如果没有拦截器链，直接执行目标方法;
  			拦截器链（每一个通知方法又被包装为方法拦截器，利用MethodInterceptor机制）
  		4）、如果有拦截器链，把需要执行的目标对象，目标方法，
  			拦截器链等信息传入创建一个 CglibMethodInvocation 对象，
  			并调用 Object retVal =  mi.proceed();
  		5）、拦截器链的触发过程;
  			1)、如果没有拦截器执行执行目标方法，或者拦截器的索引和拦截器数组-1大小一样（指定到了最后一个拦截器）执行目标方法；
  			2)、链式获取每一个拦截器，拦截器执行invoke方法，每一个拦截器等待下一个拦截器执行完成返回以后再来执行；
  				拦截器链的机制，保证通知方法与目标方法的执行顺序；

```

------

##### [源码]-AOP原理-链式调用通知方法

```java
  		5）、拦截器链的触发过程;
  			1)、如果没有拦截器执行执行目标方法，或者拦截器的索引和拦截器数组-1大小一样（指定到了最后一个拦截器）执行目标方法；
  			2)、链式获取每一个拦截器，拦截器执行invoke方法，每一个拦截器等待下一个拦截器执行完成返回以后再来执行；
  				拦截器链的机制，保证通知方法与目标方法的执行顺序；

```

------

![在这里插入图片描述](Spring%E6%B3%A8%E8%A7%A3%E9%A9%B1%E5%8A%A8%E5%BC%80%E5%8F%91.assets/20190116184854230.png)

------

##### [源码]-AOP-原理总结

```java
	总结：
  		1）、 @EnableAspectJAutoProxy 开启AOP功能
  		2）、 @EnableAspectJAutoProxy 会给容器中注册一个组件 AnnotationAwareAspectJAutoProxyCreator
  		3）、AnnotationAwareAspectJAutoProxyCreator是一个后置处理器；
  		4）、容器的创建流程：
  			1）、registerBeanPostProcessors（）注册后置处理器；创建AnnotationAwareAspectJAutoProxyCreator对象
  			2）、finishBeanFactoryInitialization（）初始化剩下的单实例bean
  				1）、创建业务逻辑组件和切面组件
  				2）、AnnotationAwareAspectJAutoProxyCreator拦截组件的创建过程
  				3）、组件创建完之后，判断组件是否需要增强
  					是：切面的通知方法，包装成增强器（Advisor）;给业务逻辑组件创建一个代理对象（cglib）；
  		5）、执行目标方法：
  			1）、代理对象执行目标方法
  			2）、CglibAopProxy.intercept()；
  				1）、得到目标方法的拦截器链（增强器包装成拦截器MethodInterceptor）
  				2）、利用拦截器的链式机制，依次进入每一个拦截器进行执行；
  				3）、效果：
  					正常执行：前置通知-》目标方法-》后置通知-》返回通知
  					出现异常：前置通知-》目标方法-》后置通知-》异常通知
```



## 声明式事物

### 声明式事务-环境搭建

1. 导入相关依赖：数据源、数据库驱动、Spring-jdbc模块

```xml
        <!-- https://mvnrepository.com/artifact/org.springframework/spring-context -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>4.3.12.RELEASE</version>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-aspects</artifactId>
            <version>4.3.12.RELEASE</version>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-jdbc</artifactId>
            <version>4.3.12.RELEASE</version>
        </dependency>

```

1. 配置数据源、JdbcTemplate（Spring提供简化数据库操作的工具）操作数据

```java
@PropertySource({"classpath:dbconfig.properties"})
@Configuration
public class TxConfig {

    //数据源
    @Bean
    public DataSource dataSource() throws PropertyVetoException {
        ComboPooledDataSource dataSource = new ComboPooledDataSource();
        dataSource.setUser("root");
        dataSource.setPassword("12358");
        dataSource.setDriverClass("com.mysql.jdbc.Driver");
        dataSource.setJdbcUrl("jdbc:mysql://localhost:3306/test");
        return dataSource;
    }

    @Bean
    public JdbcTemplate jdbcTemplate() throws PropertyVetoException {
        //Spring会的@Configuration类会做特殊的处理：给容器中添加组件，多次调用都是从容器中找组件
        JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource());
        return jdbcTemplate;
    }

}

```

1. 在mysql数据库中创建一张表
   ![在这里插入图片描述](Spring%E6%B3%A8%E8%A7%A3%E9%A9%B1%E5%8A%A8%E5%BC%80%E5%8F%91.assets/20190116192620573.png)

------

1. 创建配置类：

```java
/**
     声明式事务：

    环境搭建：
        1.导入相关依赖：数据源、数据库驱动、Spring-jdbc模块
        2.配置数据源、JdbcTemplate（Spring提供简化数据库操作的工具）操作数据
 */
@ComponentScan({"com.ldc.tx"})
@PropertySource({"classpath:dbconfig.properties"})
@Configuration
public class TxConfig {

    //数据源
    @Bean
    public DataSource dataSource() throws PropertyVetoException {
        ComboPooledDataSource dataSource = new ComboPooledDataSource();
        dataSource.setUser("root");
        dataSource.setPassword("12358");
        dataSource.setDriverClass("com.mysql.jdbc.Driver");
        dataSource.setJdbcUrl("jdbc:mysql://localhost:3306/test");
        return dataSource;
    }

    @Bean
    public JdbcTemplate jdbcTemplate() throws PropertyVetoException {
        //Spring会的@Configuration类会做特殊的处理：给容器中添加组件，多次调用都是从容器中找组件
        JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource());
        return jdbcTemplate;
    }

}

```

1. 创建UserService类：

```java
@Service
public class UserService {

    @Autowired
    private UserDao userDao;

    public void insertUser() {
        userDao.insert();
        System.out.println("插入完成...");
    }

}

```

1. 创建UserDao

```java
@Repository
public class UserDao {

    @Autowired
    private JdbcTemplate jdbcTemplate;

    public void insert() {
        String sql = "INSERT INTO `tbl_user` (username,age)VALUES(?,?)";
        String username = UUID.randomUUID().toString().substring(0, 5);
        jdbcTemplate.update(sql, username, 19);
    }

}

```

1. 我们来测试一下：

```java
    @Test
    public void test01() {
        AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(TxConfig.class);
        UserService userService = applicationContext.getBean(UserService.class);
        userService.insertUser();
    }

```

测试结果为：

> 插入完成…

此时数据也进来了：
![在这里插入图片描述](Spring%E6%B3%A8%E8%A7%A3%E9%A9%B1%E5%8A%A8%E5%BC%80%E5%8F%91.assets/20190116192920523.png)

------

此时默认是没有事务的：这里int i = 10/0;会出现异常，此时没有事务，插入的方法也不会回滚；

```java
@Service
public class UserService {

    @Autowired
    private UserDao userDao;

    public void insertUser() {
        userDao.insert();
        System.out.println("插入完成...");
        int i = 10 / 0;
    }

}

```

我们重新来运行一下测试方法：
![在这里插入图片描述](Spring%E6%B3%A8%E8%A7%A3%E9%A9%B1%E5%8A%A8%E5%BC%80%E5%8F%91.assets/20190116193217194.png)
虽然出现了异常，但是数据还是插入进来了：
![在这里插入图片描述](Spring%E6%B3%A8%E8%A7%A3%E9%A9%B1%E5%8A%A8%E5%BC%80%E5%8F%91.assets/20190116193252881.png)

------

### 声明式事务-测试成功

我们可以给方法上面加入`@Transactional`这个注解：

```java
@Service
public class UserService {

    @Autowired
    private UserDao userDao;

    @Transactional
    public void insertUser() {
        userDao.insert();
        System.out.println("插入完成...");
        int i = 10 / 0;
    }

}

```

但是只是加上这样的一个注解还是不行的，我们还需要：
4.@EnableTransactionManagement 开启基于注解的事务管理功能；
5.配置事务管理器来控制事务

```java
/**
     声明式事务：

    环境搭建：
        1.导入相关依赖：数据源、数据库驱动、Spring-jdbc模块
        2.配置数据源、JdbcTemplate（Spring提供简化数据库操作的工具）操作数据
        3.给方法上标注@Transactional注解表示当前的方法是一个事务方法；
        4.@EnableTransactionManagement 开启基于注解的事务管理功能；
        5.配置事务管理器来控制事务
 */
@ComponentScan({"com.ldc.tx"})
@PropertySource({"classpath:dbconfig.properties"})
@Configuration
@EnableTransactionManagement
public class TxConfig {

    //数据源
    @Bean
    public DataSource dataSource() throws PropertyVetoException {
        ComboPooledDataSource dataSource = new ComboPooledDataSource();
        dataSource.setUser("root");
        dataSource.setPassword("12358");
        dataSource.setDriverClass("com.mysql.jdbc.Driver");
        dataSource.setJdbcUrl("jdbc:mysql://localhost:3306/test");
        return dataSource;
    }

    @Bean
    public JdbcTemplate jdbcTemplate() throws PropertyVetoException {
        //Spring会的@Configuration类会做特殊的处理：给容器中添加组件，多次调用都是从容器中找组件
        JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource());
        return jdbcTemplate;
    }

    //注册事务管理器在容器中
    @Bean
    public PlatformTransactionManager transactionManager() throws PropertyVetoException {
        return new DataSourceTransactionManager(dataSource());
    }

}

```

这个时候，我们再来运行测试方法，数据就没有插入成功了，事务就已经生效了；

------

```java
     声明式事务：

    环境搭建：
        1.导入相关依赖：数据源、数据库驱动、Spring-jdbc模块
        2.配置数据源、JdbcTemplate（Spring提供简化数据库操作的工具）操作数据
        3.给方法上标注@Transactional注解表示当前的方法是一个事务方法；
        4.@EnableTransactionManagement 开启基于注解的事务管理功能；
        5.配置事务管理器来控制事务（必须要有这一步）
         //注册事务管理器在容器中
         @Bean
         public PlatformTransactionManager transactionManager() throws PropertyVetoException {
         return new DataSourceTransactionManager(dataSource());
         }

```

------

### [源码]-声明式事务-源码分析

```java
  声明式事务：
 
  环境搭建：
  1、导入相关依赖
  		数据源、数据库驱动、Spring-jdbc模块
  2、配置数据源、JdbcTemplate（Spring提供的简化数据库操作的工具）操作数据
  3、给方法上标注 @Transactional 表示当前方法是一个事务方法；
  4、 @EnableTransactionManagement 开启基于注解的事务管理功能；
  		@EnableXXX
  5、配置事务管理器来控制事务;
  		@Bean
  		public PlatformTransactionManager transactionManager()
 
 
  原理：
  1）、@EnableTransactionManagement
  			利用TransactionManagementConfigurationSelector给容器中会导入组件
  			导入两个组件
  			AutoProxyRegistrar
  			ProxyTransactionManagementConfiguration
  2）、AutoProxyRegistrar：
  			给容器中注册一个 InfrastructureAdvisorAutoProxyCreator 组件；
  			InfrastructureAdvisorAutoProxyCreator：？
  			利用后置处理器机制在对象创建以后，包装对象，返回一个代理对象（增强器），代理对象执行方法利用拦截器链进行调用；
 
  3）、ProxyTransactionManagementConfiguration 做了什么？
  			1、给容器中注册事务增强器；
  				1）、事务增强器要用事务注解的信息，AnnotationTransactionAttributeSource解析事务注解
  				2）、事务拦截器：
  					TransactionInterceptor；保存了事务属性信息，事务管理器；
  					他是一个 MethodInterceptor；
  					在目标方法执行的时候；
  						执行拦截器链；
  						事务拦截器：
  							1）、先获取事务相关的属性
  							2）、再获取PlatformTransactionManager，
            					如果事先没有添加指定任何transactionmanger
  								最终会从容器中按照类型获取一个PlatformTransactionManager；
            
  							3）、执行目标方法
  								如果异常，获取到事务管理器，利用事务管理回滚操作；
  								如果正常，利用事务管理器，提交事务

```

------





## Spring原理

### 扩展原理

#### 扩展原理-BeanFactoryPostProcessor

扩展原理:
BeanPostProcessor：bean的后置处理器，bean创建对象初始化前后进行拦截工作的

BeanFactoryPostProcessor：BeanFactory的后置处理器，在BeanFactory的标准初始化之后调用
所有bean的定义已经保存加载到BeanFactory，但是bean的实例还未创建

```java
/**
 * 扩展原理:
 * BeanPostProcessor：bean的后置处理器，bean创建对象初始化前后进行拦截工作的
 *
 * BeanFactoryPostProcessor：BeanFactory的后置处理器，在BeanFactory的标准初始化之后调用
 * 所有bean的定义已经保存加载到BeanFactory，但是bean的实例还未创建
 */
@Configuration
@ComponentScan("com.ldc.ext")
public class ExtConfig {

    @Bean
    public Blue blue() {
        return new Blue();
    }

}

```

我们来手动写上一个：MyBeanFactoryPostProcessor

```java
@Component
public class MyBeanFactoryPostProcessor implements BeanFactoryPostProcessor {
    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {
        System.out.println("MyBeanFactoryPostProcessor...PostProcessorBeanFactory...");
        int count = beanFactory.getBeanDefinitionCount();
        String[] beanDefinitionNames = beanFactory.getBeanDefinitionNames();
        System.out.println("当前的BeanFactory中有"+count+"个Bean");
        Stream.of(beanDefinitionNames).forEach(System.out::println);
    }
}

```

我们再来测试：

```java
    @Test
    public void test01() {
        AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(ExtConfig.class);
    }

```

运行结果：

> MyBeanFactoryPostProcessor…PostProcessorBeanFactory…
> 当前的BeanFactory中有9个Bean
> org.springframework.context.annotation.internalConfigurationAnnotationProcessor
> org.springframework.context.annotation.internalAutowiredAnnotationProcessor
> org.springframework.context.annotation.internalRequiredAnnotationProcessor
> org.springframework.context.annotation.internalCommonAnnotationProcessor
> org.springframework.context.event.internalEventListenerProcessor
> org.springframework.context.event.internalEventListenerFactory
> extConfig
> myBeanFactoryPostProcessor
> blue

我们可以看到MyBeanFactoryPostProcessor是在所有的bean创建之前执行的；

------

我们以debug来运行：
![在这里插入图片描述](Spring%E6%B3%A8%E8%A7%A3%E9%A9%B1%E5%8A%A8%E5%BC%80%E5%8F%91.assets/20190116202439832.png)

------

```java
  BeanFactoryPostProcessor原理:
  1)、ioc容器创建对象
  2)、invokeBeanFactoryPostProcessors(beanFactory);
  		如何找到所有的BeanFactoryPostProcessor并执行他们的方法；
  			1）、直接在BeanFactory中找到所有类型是BeanFactoryPostProcessor的组件，并执行他们的方法
  			2）、在初始化创建其他组件前面执行

```

------

#### 扩展原理-BeanDefinitionRegistryPostProcessor

```java
  2、BeanDefinitionRegistryPostProcessor extends BeanFactoryPostProcessor
  		postProcessBeanDefinitionRegistry();
  		在所有bean定义信息将要被加载，bean实例还未创建的；

  		优先于BeanFactoryPostProcessor执行；
  		利用BeanDefinitionRegistryPostProcessor给容器中再额外添加一些组件；

  	原理：
  		1）、ioc创建对象
  		2）、refresh()-》invokeBeanFactoryPostProcessors(beanFactory);
  		3）、从容器中获取到所有的BeanDefinitionRegistryPostProcessor组件。
  			1、依次触发所有的postProcessBeanDefinitionRegistry()方法
  			2、再来触发postProcessBeanFactory()方法BeanFactoryPostProcessor；

  		4）、再来从容器中找到BeanFactoryPostProcessor组件；然后依次触发postProcessBeanFactory()方法

```

我们自己来创建一个：

```java
@Component
public class MyBeanDefinitionRegistryPostProcessor implements BeanDefinitionRegistryPostProcessor {

    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {
        System.out.println("MyBeanDefinitionRegistryPostProcessor...bean的数量"+beanFactory.getBeanDefinitionCount());
    }

    //BeanDefinitionRegistry是bean定义信息的保存中心，以后BeanFactory就是按照BeanDefinitionRegistry里面保存的每一个bean的定义信息创建bean的实例
    @Override
    public void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry) throws BeansException {
        System.out.println("MyBeanDefinitionRegistryPostProcessor...bean的数量"+registry.getBeanDefinitionCount());
        RootBeanDefinition beanDefinition = new RootBeanDefinition(Blue.class);
        registry.registerBeanDefinition("hello",beanDefinition);
    }
}

```

运行测试方法，运行结果为：

> MyBeanDefinitionRegistryPostProcessor…bean的数量10
> MyBeanDefinitionRegistryPostProcessor…bean的数量11
> MyBeanFactoryPostProcessor…PostProcessorBeanFactory…
> 当前的BeanFactory中有11个Bean
> org.springframework.context.annotation.internalConfigurationAnnotationProcessor
> org.springframework.context.annotation.internalAutowiredAnnotationProcessor
> org.springframework.context.annotation.internalRequiredAnnotationProcessor
> org.springframework.context.annotation.internalCommonAnnotationProcessor
> org.springframework.context.event.internalEventListenerProcessor
> org.springframework.context.event.internalEventListenerFactory
> extConfig
> myBeanDefinitionRegistryPostProcessor
> myBeanFactoryPostProcessor
> blue
> hello

------

#### 扩展原理-ApplicationListener用法

```java
 3、ApplicationListener：监听容器中发布的事件。事件驱动模型开发；
  	  public interface ApplicationListener<E extends ApplicationEvent>
  		监听 ApplicationEvent 及其下面的子事件；

  	 步骤：
  		1）、写一个监听器（ApplicationListener实现类）来监听某个事件（ApplicationEvent及其子类）
  			@EventListener;
  			原理：使用EventListenerMethodProcessor处理器来解析方法上的@EventListener；

  		2）、把监听器加入到容器；
  		3）、只要容器中有相关事件的发布，我们就能监听到这个事件；
  				ContextRefreshedEvent：容器刷新完成（所有bean都完全创建）会发布这个事件；
  				ContextClosedEvent：关闭容器会发布这个事件；
  		4）、发布一个事件：
  				applicationContext.publishEvent()；

```

------

```java
@Component
public class MyApplicationListener implements ApplicationListener<ApplicationEvent> {
    //当容器中发布次事件，方法触发
    @Override
    public void onApplicationEvent(ApplicationEvent event) {
        System.out.println("收到的事件"+event);
    }
}

```

我们再来运行测试方法：

```java
    @Test
    public void test01() {
        AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(ExtConfig.class);
        applicationContext.close();
    }

```

运行结果：

> MyBeanDefinitionRegistryPostProcessor…bean的数量11
> MyBeanDefinitionRegistryPostProcessor…bean的数量12
> MyBeanFactoryPostProcessor…PostProcessorBeanFactory…
> 当前的BeanFactory中有12个Bean
> org.springframework.context.annotation.internalConfigurationAnnotationProcessor
> org.springframework.context.annotation.internalAutowiredAnnotationProcessor
> org.springframework.context.annotation.internalRequiredAnnotationProcessor
> org.springframework.context.annotation.internalCommonAnnotationProcessor
> org.springframework.context.event.internalEventListenerProcessor
> org.springframework.context.event.internalEventListenerFactory
> extConfig
> myApplicationListener
> myBeanDefinitionRegistryPostProcessor
> myBeanFactoryPostProcessor
> blue
> hello
> 收到的事件org.springframework.context.event.ContextRefreshedEvent[source=org.springframework.context.annotation.AnnotationConfigApplicationContext@3a4afd8d: startup date [Wed Jan 16 20:51:59 CST 2019]; root of context hierarchy]
> 收到的事件org.springframework.context.event.ContextClosedEvent[source=org.springframework.context.annotation.AnnotationConfigApplicationContext@3a4afd8d: startup date [Wed Jan 16 20:51:59 CST 2019]; root of context hierarchy]

#### 扩展原理-ApplicationListener原理

```java
   原理：
   	ContextRefreshedEvent、IOCTest_Ext$1[source=我发布的时间]、ContextClosedEvent；
   1）、ContextRefreshedEvent事件：
   	1）、容器创建对象：refresh()；
   	2）、finishRefresh();容器刷新完成会发布ContextRefreshedEvent事件
   2）、自己发布事件；
   3）、容器关闭会发布ContextClosedEvent；

   【事件发布流程】：
   	3）、publishEvent(new ContextRefreshedEvent(this));
   			1）、获取事件的多播器（派发器）：getApplicationEventMulticaster()
   			2）、multicastEvent派发事件：
   			3）、获取到所有的ApplicationListener；
   				for (final ApplicationListener<?> listener : getApplicationListeners(event, type)) {
   				1）、如果有Executor，可以支持使用Executor进行异步派发；
   					Executor executor = getTaskExecutor();
   				2）、否则，同步的方式直接执行listener方法；invokeListener(listener, event);
   				 拿到listener回调onApplicationEvent方法；

   【事件多播器（派发器）】
   	1）、容器创建对象：refresh();
   	2）、initApplicationEventMulticaster();初始化ApplicationEventMulticaster；
   		1）、先去容器中找有没有id=“applicationEventMulticaster”的组件；
   		2）、如果没有this.applicationEventMulticaster = new SimpleApplicationEventMulticaster(beanFactory);
   			并且加入到容器中，我们就可以在其他组件要派发事件，自动注入这个applicationEventMulticaster；

   【容器中有哪些监听器】
   	1）、容器创建对象：refresh();
   	2）、registerListeners();
   		从容器中拿到所有的监听器，把他们注册到applicationEventMulticaster中；
   		String[] listenerBeanNames = getBeanNamesForType(ApplicationListener.class, true, false);
   		//将listener注册到ApplicationEventMulticaster中
   		getApplicationEventMulticaster().addApplicationListenerBean(listenerBeanName);

```

------

#### 扩展原理-@EventListener与SmartInitializingSingleton

```java
@Service
public class UserService {
    @EventListener(classes = {ApplicationEvent.class})
    public void listen(ApplicationEvent event) {
        System.out.println("UserService监听的事件"+event);
    }
}

```

```markdown
EventListener是通过EventListenerMethodProcessor来解析的，它本质上是个SmartInitializingSingleton。SmartInitializingSingleton 的 afterSingletonsInstantiated()会在所有单实例bean创建完成之后触发。
```

![1606291392021](Spring%E6%B3%A8%E8%A7%A3%E9%A9%B1%E5%8A%A8%E5%BC%80%E5%8F%91.assets/1606291392021.png)

运行结果：

> MyBeanDefinitionRegistryPostProcessor…bean的数量12
> MyBeanDefinitionRegistryPostProcessor…bean的数量13
> MyBeanFactoryPostProcessor…PostProcessorBeanFactory…
> 当前的BeanFactory中有13个Bean
> org.springframework.context.annotation.internalConfigurationAnnotationProcessor
> org.springframework.context.annotation.internalAutowiredAnnotationProcessor
> org.springframework.context.annotation.internalRequiredAnnotationProcessor
> org.springframework.context.annotation.internalCommonAnnotationProcessor
> org.springframework.context.event.internalEventListenerProcessor
> org.springframework.context.event.internalEventListenerFactory
> extConfig
> myApplicationListener
> myBeanDefinitionRegistryPostProcessor
> myBeanFactoryPostProcessor
> userService
> blue
> hello
> UserService监听的事件org.springframework.context.event.ContextRefreshedEvent[source=org.springframework.context.annotation.AnnotationConfigApplicationContext@3a4afd8d: startup date [Wed Jan 16 21:11:12 CST 2019]; root of context hierarchy]
> 收到的事件org.springframework.context.event.ContextRefreshedEvent[source=org.springframework.context.annotation.AnnotationConfigApplicationContext@3a4afd8d: startup date [Wed Jan 16 21:11:12 CST 2019]; root of context hierarchy]
> UserService监听的事件org.springframework.context.event.ContextClosedEvent[source=org.springframework.context.annotation.AnnotationConfigApplicationContext@3a4afd8d: startup date [Wed Jan 16 21:11:12 CST 2019]; root of context hierarchy]
> 收到的事件org.springframework.context.event.ContextClosedEvent[source=org.springframework.context.annotation.AnnotationConfigApplicationContext@3a4afd8d: startup date [Wed Jan 16 21:11:12 CST 2019]; root of context hierarchy]

```java
    SmartInitializingSingleton 原理：->afterSingletonsInstantiated();
    		1）、ioc容器创建对象并refresh()；
    		2）、finishBeanFactoryInitialization(beanFactory);初始化剩下的单实例bean；
    			1）、先创建所有的单实例bean；getBean();
    			2）、获取所有创建好的单实例bean，判断是否是SmartInitializingSingleton类型的；
    				如果是就调用afterSingletonsInstantiated();
```

### 源码

#### [源码]-Spring容器创建-BeanFactory预准备

```java
Spring容器的refresh()【创建刷新】;
1、prepareRefresh()刷新前的预处理;
	1）、initPropertySources()初始化一些属性设置;子类自定义个性化的属性设置方法；
	2）、getEnvironment().validateRequiredProperties();检验属性的合法等
	3）、earlyApplicationEvents= new LinkedHashSet<ApplicationEvent>();保存容器中的一些早期的事件；
        
2、obtainFreshBeanFactory();获取BeanFactory；
	1）、refreshBeanFactory();刷新【创建】BeanFactory；
			创建了一个this.beanFactory = new DefaultListableBeanFactory();
			设置id；
	2）、getBeanFactory();返回刚才GenericApplicationContext创建的BeanFactory对象；
	3）、将创建的BeanFactory【DefaultListableBeanFactory】返回；
                
3、prepareBeanFactory(beanFactory);BeanFactory的预准备工作（BeanFactory进行一些设置）；
	1）、设置BeanFactory的类加载器、支持表达式解析器...
	2）、添加部分BeanPostProcessor【ApplicationContextAwareProcessor】
	3）、设置忽略的自动装配的接口EnvironmentAware、EmbeddedValueResolverAware、xxx；
	4）、注册可以解析的自动装配；我们能直接在任何组件中自动注入：
			BeanFactory、ResourceLoader、ApplicationEventPublisher、ApplicationContext
	5）、添加BeanPostProcessor【ApplicationListenerDetector】
	6）、添加编译时的AspectJ；
	7）、给BeanFactory中注册一些能用的组件；
		environment【ConfigurableEnvironment】、
		systemProperties【Map<String, Object>】、
		systemEnvironment【Map<String, Object>】
                
4、postProcessBeanFactory(beanFactory);BeanFactory准备工作完成后进行的后置处理工作；
	1）、子类通过重写这个方法来在BeanFactory创建并预准备完成以后做进一步的设置
======================以上是BeanFactory的创建及预准备工作==================================

```

------

#### [源码]-Spring容器创建-执行BeanFactoryPostProcessor

```java
5、invokeBeanFactoryPostProcessors(beanFactory);执行BeanFactoryPostProcessor的方法；
	BeanFactoryPostProcessor：BeanFactory的后置处理器。在BeanFactory标准初始化之后执行的；
	两个接口：BeanFactoryPostProcessor、BeanDefinitionRegistryPostProcessor
	1）、执行BeanFactoryPostProcessor的方法；
		先执行BeanDefinitionRegistryPostProcessor
		1）、获取所有的BeanDefinitionRegistryPostProcessor；
		2）、看先执行实现了PriorityOrdered优先级接口的BeanDefinitionRegistryPostProcessor、
			postProcessor.postProcessBeanDefinitionRegistry(registry)
		3）、在执行实现了Ordered顺序接口的BeanDefinitionRegistryPostProcessor；
			postProcessor.postProcessBeanDefinitionRegistry(registry)
		4）、最后执行没有实现任何优先级或者是顺序接口的BeanDefinitionRegistryPostProcessors；
			postProcessor.postProcessBeanDefinitionRegistry(registry)
			
		
		再执行BeanFactoryPostProcessor的方法
		1）、获取所有的BeanFactoryPostProcessor
		2）、看先执行实现了PriorityOrdered优先级接口的BeanFactoryPostProcessor、
			postProcessor.postProcessBeanFactory()
		3）、在执行实现了Ordered顺序接口的BeanFactoryPostProcessor；
			postProcessor.postProcessBeanFactory()
		4）、最后执行没有实现任何优先级或者是顺序接口的BeanFactoryPostProcessor；
			postProcessor.postProcessBeanFactory()

```

------

#### [源码]-Spring容器创建-注册BeanPostProcessors

```java
6、registerBeanPostProcessors(beanFactory);注册BeanPostProcessor（Bean的后置处理器）【 intercept bean creation】
		不同接口类型的BeanPostProcessor；在Bean创建前后的执行时机是不一样的
		BeanPostProcessor、
		DestructionAwareBeanPostProcessor、
		InstantiationAwareBeanPostProcessor、
		SmartInstantiationAwareBeanPostProcessor、
		MergedBeanDefinitionPostProcessor【internalPostProcessors】、
		
		1）、获取所有的 BeanPostProcessor;后置处理器都默认可以通过PriorityOrdered、Ordered接口来指定优先级
		2）、先注册PriorityOrdered优先级接口的BeanPostProcessor；
			把每一个BeanPostProcessor；添加到BeanFactory中
			beanFactory.addBeanPostProcessor(postProcessor);
		3）、再注册Ordered接口的
		4）、最后注册没有实现任何优先级接口的
		5）、最终注册MergedBeanDefinitionPostProcessor；
		6）、注册一个ApplicationListenerDetector；来在Bean创建完成后检查是否是ApplicationListener，如果是
			applicationContext.addApplicationListener((ApplicationListener<?>) bean);

```

------

#### [源码]-Spring容器创建-初始化MessageSource

```java
7、initMessageSource();初始化MessageSource组件（做国际化功能；消息绑定，消息解析）；
		1）、获取BeanFactory
		2）、看容器中是否有id为messageSource的，类型是MessageSource的组件
			如果有赋值给messageSource，如果没有自己创建一个DelegatingMessageSource；
				MessageSource：取出国际化配置文件中的某个key的值；能按照区域信息获取；
		3）、把创建好的MessageSource注册在容器中，以后获取国际化配置文件的值的时候，可以自动注入MessageSource；
			beanFactory.registerSingleton(MESSAGE_SOURCE_BEAN_NAME, this.messageSource);	
			MessageSource.getMessage(String code, Object[] args, String defaultMessage, Locale locale);
12345678
```

------

#### [源码]-Spring容器创建-初始化事件派发器、监听器等

```java
8、initApplicationEventMulticaster();初始化事件派发器；
		1）、获取BeanFactory
		2）、从BeanFactory中获取applicationEventMulticaster的ApplicationEventMulticaster；
		3）、如果上一步没有配置；创建一个SimpleApplicationEventMulticaster
		4）、将创建的ApplicationEventMulticaster添加到BeanFactory中，以后其他组件直接自动注入
9、onRefresh();留给子容器（子类）
		1、子类重写这个方法，在容器刷新的时候可以自定义逻辑；
10、registerListeners();给容器中将所有项目里面的ApplicationListener注册进来；
		1、从容器中拿到所有的ApplicationListener
		2、将每个监听器添加到事件派发器中；
			getApplicationEventMulticaster().addApplicationListenerBean(listenerBeanName);
		3、派发之前步骤产生的事件；

```

------

#### [源码]-Spring容器创建-Bean创建完成

```java
11、finishBeanFactoryInitialization(beanFactory);初始化所有剩下的单实例bean；
	1、beanFactory.preInstantiateSingletons();初始化后剩下的单实例bean
		1）、获取容器中的所有Bean，依次进行初始化和创建对象
		2）、获取Bean的定义信息；RootBeanDefinition
		3）、Bean不是抽象的，是单实例的，不是懒加载；
			1）、判断是否是FactoryBean；是否是实现FactoryBean接口的Bean；
			2）、不是工厂Bean。利用getBean(beanName);创建对象
				0、getBean(beanName)； ioc.getBean();
				1、doGetBean(name, null, null, false);
				2、先获取缓存中保存的单实例Bean。如果能获取到说明这个Bean之前被创建过（所有创建过的单实例Bean都会被缓存起来）
					从private final Map<String, Object> singletonObjects = new ConcurrentHashMap<String, Object>(256);获取的
				3、缓存中获取不到，开始Bean的创建对象流程；
				4、标记当前bean已经被创建
				5、获取Bean的定义信息；
				6、【获取当前Bean依赖的其他Bean;如果有按照getBean()把依赖的Bean先创建出来；】
				7、启动单实例Bean的创建流程；
					1）、createBean(beanName, mbd, args);
					2）、Object bean = resolveBeforeInstantiation(beanName, mbdToUse);让BeanPostProcessor先拦截返回代理对象；
						【InstantiationAwareBeanPostProcessor】：提前执行；
						先触发：postProcessBeforeInstantiation()；
						如果有返回值：触发postProcessAfterInitialization()；
					3）、如果前面的InstantiationAwareBeanPostProcessor没有返回代理对象；调用4）
					4）、Object beanInstance = doCreateBean(beanName, mbdToUse, args);创建Bean
						 1）、【创建Bean实例】；createBeanInstance(beanName, mbd, args);
						 	利用工厂方法或者对象的构造器创建出Bean实例；
						 2）、applyMergedBeanDefinitionPostProcessors(mbd, beanType, beanName);
						 	调用MergedBeanDefinitionPostProcessor的postProcessMergedBeanDefinition(mbd, beanType, beanName);
						 3）、【Bean属性赋值】populateBean(beanName, mbd, instanceWrapper);
						 	赋值之前：
						 	1）、拿到InstantiationAwareBeanPostProcessor后置处理器；
						 		postProcessAfterInstantiation()；
						 	2）、拿到InstantiationAwareBeanPostProcessor后置处理器；
						 		postProcessPropertyValues()；
						 	=====赋值之前：===
						 	3）、应用Bean属性的值；为属性利用setter方法等进行赋值；
						 		applyPropertyValues(beanName, mbd, bw, pvs);
						 4）、【Bean初始化】initializeBean(beanName, exposedObject, mbd);
						 	1）、【执行Aware接口方法】invokeAwareMethods(beanName, bean);执行xxxAware接口的方法
						 		BeanNameAware\BeanClassLoaderAware\BeanFactoryAware
						 	2）、【执行后置处理器初始化之前】applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);
						 		BeanPostProcessor.postProcessBeforeInitialization（）;
						 	3）、【执行初始化方法】invokeInitMethods(beanName, wrappedBean, mbd);
						 		1）、是否是InitializingBean接口的实现；执行接口规定的初始化；
						 		2）、是否自定义初始化方法；
						 	4）、【执行后置处理器初始化之后】applyBeanPostProcessorsAfterInitialization
						 		BeanPostProcessor.postProcessAfterInitialization()；
						 5）、注册Bean的销毁方法；
					5）、将创建的Bean添加到缓存中singletonObjects；
				ioc容器就是这些Map；很多的Map里面保存了单实例Bean，环境信息。。。。；
		所有Bean都利用getBean创建完成以后；
			检查所有的Bean是否是SmartInitializingSingleton接口的；如果是；就执行afterSingletonsInstantiated()；

```

------

#### [源码]-Spring容器创建-容器创建完成

```java
12、finishRefresh();完成BeanFactory的初始化创建工作；IOC容器就创建完成；
		1）、initLifecycleProcessor();初始化和生命周期有关的后置处理器；LifecycleProcessor
			默认从容器中找是否有lifecycleProcessor的组件【LifecycleProcessor】；如果没有new DefaultLifecycleProcessor();
			加入到容器；
			
			写一个LifecycleProcessor的实现类，可以在BeanFactory
				void onRefresh();
				void onClose();	
		2）、	getLifecycleProcessor().onRefresh();
			拿到前面定义的生命周期处理器（BeanFactory）；回调onRefresh()；
		3）、publishEvent(new ContextRefreshedEvent(this));发布容器刷新完成事件；
		4）、LiveBeansView.registerApplicationContext(this);

```

------

#### [源码]-Spring源码总结

```java
	======总结===========
	1）、Spring容器在启动的时候，先会保存所有注册进来的Bean的定义信息；
		1）、xml注册bean；<bean>
		2）、注解注册Bean；@Service、@Component、@Bean、xxx
	2）、Spring容器会合适的时机创建这些Bean
		1）、用到这个bean的时候；利用getBean创建bean；创建好以后保存在容器中；
		2）、统一创建剩下所有的bean的时候；finishBeanFactoryInitialization()；
	3）、后置处理器；BeanPostProcessor
		1）、每一个bean创建完成，都会使用各种后置处理器进行处理；来增强bean的功能；
			AutowiredAnnotationBeanPostProcessor:处理自动注入
			AnnotationAwareAspectJAutoProxyCreator:来做AOP功能；
			xxx....
			增强的功能注解：
			AsyncAnnotationBeanPostProcessor
			....
	4）、事件驱动模型；
		ApplicationListener；事件监听；
		ApplicationEventMulticaster；事件派发：

```



## Servlet

### servlet3.0-简介&测试

现在，我们来说说注解版的web，我们以前来写web的三大组件：Servlet、Filter、Listener，包括SpringMVC的前端控制器DispatcherServlet都需要在web.xml文件中来进行注册；而在Servlet3.0标准以后，就给我们提供了方便的注解的方式来完成我们这些组件的注册以及添加，提供了运行时的可插拔的插件能力；

说明：Servlet3.0及以上的标准是需要Tomcat7及以上的支持；

------

1. 创建一个动态的web工程：
   ![在这里插入图片描述](Spring%E6%B3%A8%E8%A7%A3%E9%A9%B1%E5%8A%A8%E5%BC%80%E5%8F%91.assets/20190117191510495.png)

------

1. 我们来写上一个jsp页面：

```jsp
<%--
  Created by IntelliJ IDEA.
  User: WH1803054
  Date: 2019/1/17
  Time: 19:07
  To change this template use File | Settings | File Templates.
--%>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
  <head>
    <title>$Title$</title>
  </head>
  <body>
    <a href="hello">hello</a>
  </body>
</html>

```

1. 我们再来写上一个servlet，并且用`@WebServlet("/hello")`来标注，并且指明要拦截哪些路径：

```java
package com.ldc.servlet;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

@WebServlet("/hello")
public class HelloServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        resp.getWriter().write("hello...");
    }
}

```

1. 启动Tomcat服务器：运行成功
   ![在这里插入图片描述](Spring%E6%B3%A8%E8%A7%A3%E9%A9%B1%E5%8A%A8%E5%BC%80%E5%8F%91.assets/201901171926112.png)
   点进这个链接也会打印字符串：
   ![在这里插入图片描述](Spring%E6%B3%A8%E8%A7%A3%E9%A9%B1%E5%8A%A8%E5%BC%80%E5%8F%91.assets/20190117192708317.png)

------

同样，要注册Filter用`@WebFilter`注解、注册Listener用`@WebListener`注解；如果在注册的时候，需要一些初始化参数，我们就可以用`@WebInitParam`注解；

------

### servlet3.0-ServletContainerInitializer

```java
Shared libraries（共享库） / runtimes pluggability（运行时插件能力）

1、Servlet容器启动会扫描，当前应用里面每一个jar包的ServletContainerInitializer的实现
2、提供ServletContainerInitializer的实现类；
	必须绑定在，META-INF/services/javax.servlet.ServletContainerInitializer
	文件的内容就是ServletContainerInitializer实现类的全类名；

总结：容器在启动应用的时候，会扫描当前应用每一个jar包里面
META-INF/services/javax.servlet.ServletContainerInitializer
指定的实现类，启动并运行这个实现类的方法；传入感兴趣的类型；


ServletContainerInitializer；
@HandlesTypes；



```

------

第一步：我们写一个MyServletContainerInitializer 类实现ServletContainerInitializer 接口

```java
//容器启动的时候会将@HandlesTypes指定的这个类型下面的子类（实现类或者子接口等）传递过来
//传入感兴趣的类型
@HandlesTypes(value = {HelloService.class})
public class MyServletContainerInitializer implements ServletContainerInitializer {
    /**
     * 在应用启动的时候，会运行onStartup方法；
     * Set<Class<?>> ：感兴趣的类型的所有子类型；
     * ServletContext 代表当前的web应用的ServletContext对象，一个web应用相当于是一个ServletContext
     * @throws ServletException
     */
    @Override
    public void onStartup(Set<Class<?>> set, ServletContext servletContext) throws ServletException {
        System.out.println("感兴趣的类型：");
        set.forEach(System.out::println);
    }
}

```

第二步：在这个路径下新建一个文件

![在这里插入图片描述](Spring%E6%B3%A8%E8%A7%A3%E9%A9%B1%E5%8A%A8%E5%BC%80%E5%8F%91.assets/20190117195242717.png)
文件的内容就写我们实现ServletContainerInitializer 这个接口的类MyServletContainerInitializer 的全类名：
![在这里插入图片描述](Spring%E6%B3%A8%E8%A7%A3%E9%A9%B1%E5%8A%A8%E5%BC%80%E5%8F%91.assets/20190117195358421.png)

------

![在这里插入图片描述](Spring%E6%B3%A8%E8%A7%A3%E9%A9%B1%E5%8A%A8%E5%BC%80%E5%8F%91.assets/20190117195451885.png)

------

最后，我们运行起来，运行结果为：

> 感兴趣的类型：
> class com.ldc.service.HelloServiceExt
> class com.ldc.service.AbstractHelloService
> class com.ldc.service.HelloServiceImpl

------

### servlet3.0-ServletContext注册三大组件

首先我们写一个Servlet：

```java
public class UserServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        resp.getWriter().write("tomcat...");
        System.out.println("UserServlet...doGet...");
    }
}

```

------

再来一个Filter：

```java
public class UserFilter implements Filter {
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {

    }

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        //过滤请求
        System.out.println("UserFilter...doFilter...");
        //放行
        filterChain.doFilter(servletRequest,servletResponse);
    }

    @Override
    public void destroy() {

    }
}

```

最后，再写一个Listener：

```java
/**
 * 监听项目的启动和停止
 */
public class UserListener implements ServletContextListener {
    //监听ServletContextEvent的启动初始化
    @Override
    public void contextInitialized(ServletContextEvent servletContextEvent) {
        System.out.println("UserListener...contextInitialized");
    }
    //监听ServletContextEvent销毁
    @Override
    public void contextDestroyed(ServletContextEvent servletContextEvent) {
        System.out.println("UserListener...contextDestroyed");
    }
}

```

最后，我们来用ServletContext来进行注册三大组件：

```java
//容器启动的时候会将@HandlesTypes指定的这个类型下面的子类（实现类或者子接口等）传递过来
//传入感兴趣的类型
@HandlesTypes(value = {HelloService.class})
public class MyServletContainerInitializer implements ServletContainerInitializer {
    /**
     * 在应用启动的时候，会运行onStartup方法；
     * Set<Class<?>> ：感兴趣的类型的所有子类型；
     * ServletContext 代表当前的web应用的ServletContext对象，一个web应用相当于是一个ServletContext
     * 1）、使用ServletContext注册Web组件（Servlet、Filter、Listener）
     * 2）、使用编码的方式，在项目启动的时候给ServletContext添加组件
     * 必须在项目启动的时候来添加
     *  （1）ServletContainerInitializer得到ServletContext对象来注册；
     *  （2）ServletContextListener的方法的参数里面的ServletContextEvent对象可以获取ServletContext对象
     */
    @Override
    public void onStartup(Set<Class<?>> set, ServletContext servletContext) throws ServletException {
        System.out.println("感兴趣的类型：");
        set.forEach(System.out::println);

        //注册组件
        Dynamic servlet = servletContext.addServlet("userServlet", new UserServlet());
        //配置servlet的映射信息
        servlet.addMapping("/user");

        //注册Listener
        servletContext.addListener(UserListener.class);

        //注册Filter
        FilterRegistration.Dynamic filter = servletContext.addFilter("userFilter", UserFilter.class);
        filter.addMappingForUrlPatterns(EnumSet.of(DispatcherType.REQUEST),true,"/*");
    }
}

```

我们启动，然后再浏览器上进行访问：

> 感兴趣的类型：
> class com.ldc.service.AbstractHelloService
> class com.ldc.service.HelloServiceImpl
> class com.ldc.service.HelloServiceExt
> UserListener…contextInitialized
> UserFilter…doFilter…
> UserFilter…doFilter…
> UserFilter…doFilter…
> UserServlet…doGet…

如果服务器停止了，监听器也是能监听得到：

> UserListener…contextDestroyed

------

### servlet3.0-与SpringMVC整合分析

[Spring5.2.0 WebServlet官方文档](https://docs.spring.io/spring/docs/5.2.0.BUILD-SNAPSHOT/spring-framework-reference/web.html#spring-web)

------

```java
1、web容器在启动的时候，会扫描每个jar包下的META-INF/services/javax.servlet.ServletContainerInitializer
2、加载这个文件指定的类SpringServletContainerInitializer
3、spring的应用一启动会加载感兴趣的WebApplicationInitializer接口的下的所有组件；
4、并且为WebApplicationInitializer组件创建对象（组件不是接口，不是抽象类）
	1）、AbstractContextLoaderInitializer：创建根容器；createRootApplicationContext()；
	2）、AbstractDispatcherServletInitializer：
			创建一个web的ioc容器；createServletApplicationContext();
			创建了DispatcherServlet；createDispatcherServlet()；
			将创建的DispatcherServlet添加到ServletContext中；
				getServletMappings();
	3）、AbstractAnnotationConfigDispatcherServletInitializer：注解方式配置的DispatcherServlet初始化器
			创建根容器：createRootApplicationContext()
					getRootConfigClasses();传入一个配置类
			创建web的ioc容器： createServletApplicationContext();
					获取配置类；getServletConfigClasses();
	
总结：
	以注解方式来启动SpringMVC；继承AbstractAnnotationConfigDispatcherServletInitializer；
实现抽象方法指定DispatcherServlet的配置信息；



```

------

### springmvc-整合

1. 导入jar包：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.ldc</groupId>
    <artifactId>springmvc-annotation</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>war</packaging>

    <dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
            <version>4.3.11.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>servlet-api</artifactId>
            <version>3.0-alpha-1</version>
            <!--因为tomcat也有servlet-api，要是项目打成war包的时候，就不要带上这个jar包,否则就会引起冲突-->
            <scope>provided</scope>
        </dependency>
    </dependencies>


    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-war-plugin</artifactId>
                <version>2.4</version>
                <configuration>
                    <failOnMissingWebXml>false</failOnMissingWebXml>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>

```

1. 创建web初始化器，以及父子配置类：

```java
//Web容器启动的时候创建对象；调用方法来初始化容器以及前端控制器
public class MyWebInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {

    //获取父容器的配置类:（Spring的配置文件） --->作为父容器
    @Override
    protected Class<?>[] getRootConfigClasses() {
        return new Class<?>[]{RootConfig.class};
    }

    //获取web容器的配置类（SpringMVC配置文件） --->作为一个子容器
    @Override
    protected Class<?>[] getServletConfigClasses() {
        return new Class<?>[]{AppConfig.class};
    }

    //获取DispatcherServlet的映射信息
    //  /:拦截所有请求（包括静态资源（xx.js,xx.png），但是不包括*.jsp）
    //  /*:拦截所有请求，连*.jsp页面都拦截；jsp页面是tomcat引擎解析的
    @Override
    protected String[] getServletMappings() {
        return new String[]{"/"};
    }
}

```

------

```java
//Spring的容器不扫描Controller，父容器
@ComponentScan(value = {"com.ldc."},excludeFilters = {
        @Filter(type = FilterType.ANNOTATION,classes = {Controller.class})
})
public class RootConfig {

}

```

------

```java
//SpringMVC只扫描Controller，子容器
//useDefaultFilters = false 禁用默认的过滤规则
@ComponentScan(value = {"com.ldc"},includeFilters = {
        @Filter(type = FilterType.ANNOTATION,classes = {Controller.class})
},useDefaultFilters = false)
public class AppConfig {

}

```

------

1. 我们再来写一个Controller和Service：

```java
@Controller
public class HelloController {

    @Autowired
    private HelloService helloService;

    @ResponseBody
    @RequestMapping("hello")
    public String hello() {
        String hello = helloService.sayHello("tomcat");
        return hello;
    }

}

```

------

```java
@Service
public class HelloService {

    public String sayHello(String name) {
        return "Hello," + name;
    }

}


```

------

1. 测试：我们启动tomcat服务器来运行测试：
   ![在这里插入图片描述](Spring%E6%B3%A8%E8%A7%A3%E9%A9%B1%E5%8A%A8%E5%BC%80%E5%8F%91.assets/20190117214610747.png)

------

### springmvc-定制与接管SpringMVC

[SpringMVC的其他相关的注解的配置参考官方文档](https://docs.spring.io/spring/docs/5.2.0.BUILD-SNAPSHOT/spring-framework-reference/web.html#mvc-config)

```java
定制SpringMVC；
1）、@EnableWebMvc:开启SpringMVC定制配置功能；
	<mvc:annotation-driven/>；

2）、配置组件（视图解析器、视图映射、静态资源映射、拦截器。。。）
	extends WebMvcConfigurerAdapter

```

------

[SpringMVC具体配置的参考文档](https://docs.spring.io/spring/docs/5.2.0.BUILD-SNAPSHOT/spring-framework-reference/web.html#mvc-config-customize)

------

我们就可以这样来写：

```java
//SpringMVC只扫描Controller，子容器
//useDefaultFilters = false 禁用默认的过滤规则
@ComponentScan(value = {"com.ldc"},includeFilters = {
        @Filter(type = FilterType.ANNOTATION,classes = {Controller.class})
},useDefaultFilters = false)
@EnableWebMvc
public class AppConfig implements WebMvcConfigurer {

    @Override
    public void configurePathMatch(PathMatchConfigurer pathMatchConfigurer) {

    }

    @Override
    public void configureContentNegotiation(ContentNegotiationConfigurer contentNegotiationConfigurer) {

    }

    @Override
    public void configureAsyncSupport(AsyncSupportConfigurer asyncSupportConfigurer) {

    }

    @Override
    public void configureDefaultServletHandling(DefaultServletHandlerConfigurer defaultServletHandlerConfigurer) {
        //将SpringMVC处理不了的请求交给tomcat，专门针对于静态资源的，这个时候，静态资源就是可以访问的
        defaultServletHandlerConfigurer.enable();
    }

    @Override
    public void addFormatters(FormatterRegistry formatterRegistry) {
        //添加自定义的类型转换器
    }

    @Override
    public void addInterceptors(InterceptorRegistry interceptorRegistry) {

    }

    @Override
    public void addResourceHandlers(ResourceHandlerRegistry resourceHandlerRegistry) {

    }

    @Override
    public void addCorsMappings(CorsRegistry corsRegistry) {

    }

    @Override
    public void addViewControllers(ViewControllerRegistry viewControllerRegistry) {

    }

    @Override
    public void configureViewResolvers(ViewResolverRegistry viewResolverRegistry) {

    }

    @Override
    public void addArgumentResolvers(List<HandlerMethodArgumentResolver> list) {

    }

    @Override
    public void addReturnValueHandlers(List<HandlerMethodReturnValueHandler> list) {

    }

    @Override
    public void configureMessageConverters(List<HttpMessageConverter<?>> list) {

    }

    @Override
    public void extendMessageConverters(List<HttpMessageConverter<?>> list) {

    }

    @Override
    public void configureHandlerExceptionResolvers(List<HandlerExceptionResolver> list) {

    }

    @Override
    public void extendHandlerExceptionResolvers(List<HandlerExceptionResolver> list) {

    }

    @Override
    public Validator getValidator() {
        return null;
    }

    @Override
    public MessageCodesResolver getMessageCodesResolver() {
        return null;
    }
}

```

但是上面直接实现WebMvcConfigurer接口的方式，有很多的方法用不到，我们可以用这个适配器WebMvcConfigurerAdapter来实现，它实现了WebMvcConfigurer接口：

我们可以来定义一个视图解析器：

```java
//SpringMVC只扫描Controller，子容器
//useDefaultFilters = false 禁用默认的过滤规则
@ComponentScan(value = {"com.ldc"},includeFilters = {
        @Filter(type = FilterType.ANNOTATION,classes = {Controller.class})
},useDefaultFilters = false)
@EnableWebMvc
public class AppConfig extends WebMvcConfigurerAdapter {
    //定制

    //视图解析器

    @Override
    public void configureViewResolvers(ViewResolverRegistry registry) {
        //默认所有页面都是从/WEB-INF/xxx.jsp
        //registry.jsp();
        //我们也可以自己来写规则
        registry.jsp("/WEB-INF/views/", ".jsp");
    }
}

```

我们在/WEB-INF/views/这个路径下新建一个success.jsp

```jsp
<%--
  Created by IntelliJ IDEA.
  User: WH1803054
  Date: 2019/1/17
  Time: 22:27
  To change this template use File | Settings | File Templates.
--%>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
<h1>success</h1>
</body>
</html>


```

------

![在这里插入图片描述](Spring%E6%B3%A8%E8%A7%A3%E9%A9%B1%E5%8A%A8%E5%BC%80%E5%8F%91.assets/20190117223358354.png)

------

现在，我们来运行项目，测试：这个时候，就表示视图解析器配置成功了
![在这里插入图片描述](Spring%E6%B3%A8%E8%A7%A3%E9%A9%B1%E5%8A%A8%E5%BC%80%E5%8F%91.assets/20190117223435680.png)

------

我们在这个路径下放一个图片，然后再写一个jsp来访问它：
![在这里插入图片描述](Spring%E6%B3%A8%E8%A7%A3%E9%A9%B1%E5%8A%A8%E5%BC%80%E5%8F%91.assets/20190117224220443.png)

------

这个时候，图片是显示不出来的：
![在这里插入图片描述](Spring%E6%B3%A8%E8%A7%A3%E9%A9%B1%E5%8A%A8%E5%BC%80%E5%8F%91.assets/2019011722425988.png)

------

现在，我们来配置允许静态资源的访问：

```java
//SpringMVC只扫描Controller，子容器
//useDefaultFilters = false 禁用默认的过滤规则
@ComponentScan(value = {"com.ldc"},includeFilters = {
        @Filter(type = FilterType.ANNOTATION,classes = {Controller.class})
},useDefaultFilters = false)
@EnableWebMvc
public class AppConfig extends WebMvcConfigurerAdapter {
    //定制

    //视图解析器

    @Override
    public void configureViewResolvers(ViewResolverRegistry registry) {
        //默认所有页面都是从/WEB-INF/xxx.jsp
        //registry.jsp();
        //我们也可以自己来写规则
        registry.jsp("/WEB-INF/views/", ".jsp");
    }

    //静态资源的访问
    @Override
    public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
        configurer.enable();
    }
}

```

------

这个时候，我们的图片就是可以访问了：
![在这里插入图片描述](Spring%E6%B3%A8%E8%A7%A3%E9%A9%B1%E5%8A%A8%E5%BC%80%E5%8F%91.assets/20190117224607807.png)

------

我们写上一个拦截器：

```java
public class MyFirstInterceptor implements HandlerInterceptor {
    //在目标方法执行之前执行
    @Override
    public boolean preHandle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o) throws Exception {
        System.out.println("preHandle...");
        return true;
    }

    //在目标方法执行之后执行
    @Override
    public void postHandle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, ModelAndView modelAndView) throws Exception {
        System.out.println("postHandle...");
    }

    //页面响应以后执行
    @Override
    public void afterCompletion(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, Exception e) throws Exception {
        System.out.println("afterCompletion...");
    }
}

```

我们在配置类里面添加拦截器：

```java
//SpringMVC只扫描Controller，子容器
//useDefaultFilters = false 禁用默认的过滤规则
@ComponentScan(value = {"com.ldc"},includeFilters = {
        @Filter(type = FilterType.ANNOTATION,classes = {Controller.class})
},useDefaultFilters = false)
@EnableWebMvc
public class AppConfig extends WebMvcConfigurerAdapter {
    //定制

    //视图解析器

    @Override
    public void configureViewResolvers(ViewResolverRegistry registry) {
        //默认所有页面都是从/WEB-INF/xxx.jsp
        //registry.jsp();
        //我们也可以自己来写规则
        registry.jsp("/WEB-INF/views/", ".jsp");
    }

    //静态资源的访问
    @Override
    public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
        configurer.enable();
    }

    //配置拦截器
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        //拦截任意的路径
        registry.addInterceptor(new MyFirstInterceptor()).addPathPatterns("/**");
    }
}

```

我们重新启动，并且访问这个路径：
![在这里插入图片描述](Spring%E6%B3%A8%E8%A7%A3%E9%A9%B1%E5%8A%A8%E5%BC%80%E5%8F%91.assets/20190117225942280.png)

------

我们发现控制台已经打印了，说明拦截器已经起作用了：

> preHandle…
> postHandle…
> afterCompletion…
> preHandle…
> postHandle…
> afterCompletion…

------

### servlet3.0-异步请求

**servlet3.0异步处理**：
在Servlet 3.0之前，Servlet采用Thread-Per-Request的方式处理请求。
即每一次Http请求都由某一个线程从头到尾负责处理。
![在这里插入图片描述](Spring%E6%B3%A8%E8%A7%A3%E9%A9%B1%E5%8A%A8%E5%BC%80%E5%8F%91.assets/20190118152057841.png)
如果一个请求需要进行IO操作，比如访问数据库、调用第三方服务接口等，那么其所对应的线程将同步地等待IO操作完成， 而IO操作是非常慢的，所以此时的线程并不能及时地释放回线程池以供后续使用，在并发量越来越大的情况下，这将带来严重的性能问题。即便是像Spring、Struts这样的高层框架也脱离不了这样的桎梏，因为他们都是建立在Servlet之上的。为了解决这样的问题，Servlet 3.0引入了异步处理，然后在Servlet 3.1中又引入了非阻塞IO来进一步增强异步处理的性能。

------

我们可以在之前写的Servlet加上当前的线程：

```java
@WebServlet("/hello")
public class HelloServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        System.out.println(Thread.currentThread()+" start...");
        try {
            sayHello();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        resp.getWriter().write("hello...");
        System.out.println(Thread.currentThread()+" end...");
    }

    private void sayHello() throws InterruptedException {
        System.out.println(Thread.currentThread()+ " processing...");
        Thread.sleep(3000);
    }

}

```

启动服务之后，控制台打印结果为：我们可以发现从线程开始、处理请求到执行结束从始至终都是Thread[http-nio-8081-exec-3,5,main]这个线程，主线程得不到释放，当下一个请求进来就得不到处理；

> UserFilter…doFilter…
> Thread[http-nio-8081-exec-3,5,main] start…
> Thread[http-nio-8081-exec-3,5,main] processing…
> Thread[http-nio-8081-exec-3,5,main] end…

------

我们再来加上是哪些线程处理的：

```java
@WebServlet(value = "/async",asyncSupported = true)
public class HelloAsyncServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        //1.支持异步处理:asyncSupported = true

        //2.开启异步模式
        System.out.println("主线程开始..."+Thread.currentThread()+"==>"+ Instant.now().toEpochMilli());
        AsyncContext startAsync = req.startAsync();
        //3.业务逻辑进行异步处理，开始异步处理
        startAsync.start(()-> {
            try {
                System.out.println("副线程开始..."+Thread.currentThread()+"==>"+ Instant.now().toEpochMilli());
                sayHello();
                startAsync.complete();
                //获取异步上下文
                //AsyncContext asyncContext = req.getAsyncContext();
                //4.获取响应
                ServletResponse response = startAsync.getResponse();
                response.getWriter().write("hello async...");
                System.out.println("副线程结束..."+Thread.currentThread()+"==>"+ Instant.now().toEpochMilli());
            } catch (Exception e) {
                e.printStackTrace();
            }
        });
        System.out.println("主线程结束..."+Thread.currentThread()+"==>"+ Instant.now().toEpochMilli());
    }

    private void sayHello() throws InterruptedException {
        System.out.println(Thread.currentThread()+ " processing..."+"==>"+ Instant.now().toEpochMilli());
        Thread.sleep(3000);
    }
}

```

------

执行结果如下：

> 感兴趣的类型：
> class com.ldc.service.HelloServiceImpl
> class com.ldc.service.HelloServiceExt
> class com.ldc.service.AbstractHelloService
> UserListener…contextInitialized
> [2019-01-18 04:47:06,991] Artifact servlet3.0:war exploded: Artifact is deployed successfully
> [2019-01-18 04:47:06,992] Artifact servlet3.0:war exploded: Deploy took 447 milliseconds
> UserFilter…doFilter…
> UserFilter…doFilter…
> UserFilter…doFilter…
> 主线程开始…Thread[http-nio-8081-exec-7,5,main]>1547801232248
> 主线程结束…Thread[http-nio-8081-exec-7,5,main]>1547801232253
> 副线程开始…Thread[http-nio-8081-exec-8,5,main]>1547801232253
> Thread[http-nio-8081-exec-8,5,main] processing…>1547801232253
> 副线程结束…Thread[http-nio-8081-exec-8,5,main]==>1547801235253

------

现在的处理就是可以表示成下图：
![在这里插入图片描述](Spring%E6%B3%A8%E8%A7%A3%E9%A9%B1%E5%8A%A8%E5%BC%80%E5%8F%91.assets/20190118165116681.png)

------

### springmvc-异步请求-返回Callable

```java
@Controller
public class AsyncController {
	 /**
     * 1、控制器返回Callable
     * 2、Spring异步处理，将Callable 提交到 TaskExecutor 使用一个隔离的线程进行执行
     * 3、DispatcherServlet和所有的Filter退出web容器的线程，但是response 保持打开状态；
     * 4、Callable返回结果，SpringMVC将请求重新派发给容器，恢复之前的处理；
     * 5、根据Callable返回的结果。SpringMVC继续进行视图渲染流程等（从收请求-视图渲染）。
     *
     * preHandle.../springmvc-annotation/async01
     主线程开始...Thread[http-bio-8081-exec-3,5,main]==>1513932494700
     主线程结束...Thread[http-bio-8081-exec-3,5,main]==>1513932494700
     =========DispatcherServlet及所有的Filter退出线程============================

     ================等待Callable执行==========
     副线程开始...Thread[MvcAsync1,5,main]==>1513932494707
     副线程开始...Thread[MvcAsync1,5,main]==>1513932496708
     ================Callable执行完成==========

     ================再次收到之前重发过来的请求========
     preHandle.../springmvc-annotation/async01
     postHandle...（Callable的之前的返回值就是目标方法的返回值）
     afterCompletion...

     异步的拦截器:
     1）、原生API的AsyncListener
     2）、SpringMVC：实现AsyncHandlerInterceptor；
     * @return
     */
    @ResponseBody
    @RequestMapping("/async01")
    public Callable<String> async01() {
        System.out.println("主线程开始..." + Thread.currentThread() + "==>" + Instant.now().getEpochSecond());
        Callable<String> callable = new Callable<String>() {
            @Override
            public String call() throws Exception {
                System.out.println("副线程开始..." + Thread.currentThread() + "==>" + Instant.now().getEpochSecond());
                Thread.sleep(2000);
                System.out.println("副线程结束..." + Thread.currentThread() + "==>" + Instant.now().getEpochSecond());
                return "Callable<String> async01()";
            }
        };
        System.out.println("主线程结束..." + Thread.currentThread() + "==>" + Instant.now().getEpochSecond());
        return callable;
    }

}

```

此时运行起来的测试结果如下：

> preHandle…
> 主线程开始…Thread[http-nio-8081-exec-7,5,main]>1547802269
> 主线程结束…Thread[http-nio-8081-exec-7,5,main]>1547802269
> 副线程开始…Thread[MvcAsync1,5,main]>1547802269
> 副线程结束…Thread[MvcAsync1,5,main]>1547802271
> preHandle…
> postHandle…
> afterCompletion…

------

### springmvc-异步请求-返回DeferredResult

我们来看一个实际的应用场景：
![在这里插入图片描述](Spring%E6%B3%A8%E8%A7%A3%E9%A9%B1%E5%8A%A8%E5%BC%80%E5%8F%91.assets/20190118172926766.png)

------

```java
public class DeferredResultQueue {

    private static Queue<DeferredResult<Object>> queue = new ConcurrentLinkedDeque<>();

    public static void save(DeferredResult<Object> deferredResult) {
        queue.add(deferredResult);
    }
    public static DeferredResult<Object> get() {
        return queue.poll();
    }
}

```

------

```java
@Controller
public class AsyncController {


    @ResponseBody
    @RequestMapping("/createOrder")
    public DeferredResult<Object> createOrder(){
        DeferredResult<Object> deferredResult = new DeferredResult<>((long)3000, "create fail...");

        DeferredResultQueue.save(deferredResult);

        return deferredResult;
    }


    @ResponseBody
    @RequestMapping("/create")
    public String create(){
        //创建订单
        String order = UUID.randomUUID().toString();
        DeferredResult<Object> deferredResult = DeferredResultQueue.get();
        deferredResult.setResult(order);
        return "success===>"+order;
    }
}

```

------

我们先访问这个创建订单createOrder接口：
![在这里插入图片描述](Spring%E6%B3%A8%E8%A7%A3%E9%A9%B1%E5%8A%A8%E5%BC%80%E5%8F%91.assets/20190118174029853.png)

------

我们再来访问这个create接口，此时的结果如图所示：
![在这里插入图片描述](Spring%E6%B3%A8%E8%A7%A3%E9%A9%B1%E5%8A%A8%E5%BC%80%E5%8F%91.assets/20190118174213537.png)