# Spring 6 源码学习完整指南

## 目录
1. [项目概述](#项目概述)
2. [学习策略](#学习策略)
3. [环境准备](#环境准备)
4. [学习路径](#学习路径)
5. [第一阶段：IoC容器原理](#第一阶段ioc容器原理)
6. [第二阶段：依赖注入机制](#第二阶段依赖注入机制)
7. [第三阶段：AOP实现原理](#第三阶段aop实现原理)
8. [第四阶段：配置管理](#第四阶段配置管理)
9. [第五阶段：Web框架](#第五阶段web框架)
10. [面试重点](#面试重点)
11. [学习技巧](#学习技巧)

## 项目概述

### 基本信息
- **项目名称**: Spring Framework 6
- **当前版本**: 7.0.0-SNAPSHOT (开发中)
- **Java版本要求**: Java 17+
- **构建工具**: Gradle
- **核心特性**: 支持Jakarta EE 9+, 原生镜像支持, 响应式编程

### 项目结构
```
spring-framework/
├── spring-core/          # 核心工具类和基础设施
├── spring-beans/         # Bean容器和依赖注入
├── spring-context/       # 应用上下文和配置
├── spring-aop/           # 面向切面编程
├── spring-web/           # Web基础功能
├── spring-webmvc/        # MVC框架
├── spring-webflux/       # 响应式Web框架
├── spring-jdbc/          # JDBC支持
├── spring-orm/           # ORM集成
├── spring-tx/            # 事务管理
└── spring-test/          # 测试支持
```

## 学习策略

### 1. 测试用例驱动学习
- **优势**: 官方测试用例覆盖了所有核心功能
- **方法**: 先运行测试，再深入源码分析
- **目标**: 通过测试用例理解API的使用和内部实现

### 2. 调试技巧
- 在测试方法中设置断点
- 使用IDE的调试功能跟踪执行流程
- 观察变量值和调用栈信息

### 3. 学习顺序
1. **从简单到复杂**: 先理解基础概念，再深入高级特性
2. **理论与实践结合**: 理论学习与代码实践相结合
3. **问题导向**: 带着问题阅读源码，提高学习效率

## 环境准备

### 1. 开发环境
- **JDK**: 17+ (Spring 6要求)
- **IDE**: IntelliJ IDEA (推荐) 或 Eclipse
- **构建工具**: Gradle (项目自带)

### 2. 项目导入
```bash
# 克隆项目 (如果还没有)
git clone https://github.com/spring-projects/spring-framework.git
cd spring-framework

# 导入到IDE
# 在IntelliJ IDEA中: File -> Open -> 选择spring-framework目录
```

### 3. 构建项目
```bash
# 首次构建 (可能需要较长时间)
./gradlew build -x test

# 或者只构建特定模块
./gradlew :spring-core:build -x test
./gradlew :spring-beans:build -x test
./gradlew :spring-context:build -x test
```

## 学习路径

### 总体时间安排 (10周)
- **第1-2周**: IoC容器原理
- **第3-4周**: 依赖注入机制
- **第5-6周**: AOP实现原理
- **第7-8周**: 配置管理
- **第9-10周**: Web框架

## 第一阶段：IoC容器原理 (第1-2周)

### 1.1 BeanFactory 基础功能

#### 核心测试用例
[DefaultListableBeanFactoryTests.java](spring-beans/src/test/java/org/springframework/beans/factory/DefaultListableBeanFactoryTests.java)

#### BeanDefinition 体系详解

Spring中的BeanDefinition是描述Bean配置信息的核心接口，有多种实现类，每种都有特定的用途：

##### 1. BeanDefinition 继承体系
```
BeanDefinition (接口)
├── AbstractBeanDefinition (抽象基类)
│   ├── RootBeanDefinition (根Bean定义)
│   ├── ChildBeanDefinition (子Bean定义)
│   └── GenericBeanDefinition (通用Bean定义)
└── AnnotatedBeanDefinition (注解Bean定义接口)
    ├── AnnotatedGenericBeanDefinition (注解通用Bean定义)
    └── ScannedGenericBeanDefinition (扫描通用Bean定义)
```

##### 2. 各种BeanDefinition的作用和区别

###### **RootBeanDefinition (根Bean定义)**
- **用途**: 表示运行时合并后的Bean定义，是最终的、统一的Bean定义视图
- **特点**: 
  - 没有父Bean定义，是独立的
  - 包含完整的Bean配置信息
  - 支持工厂方法和实例提供者
  - 适合程序化配置
- **使用场景**:
  - `@Bean`方法生成的Bean定义
  - 工厂方法创建的Bean
  - 程序化注册的Bean

###### **ChildBeanDefinition (子Bean定义)**
- **用途**: 继承父Bean定义的配置，可以覆盖或添加新的配置
- **特点**:
  - 必须有父Bean定义
  - 继承父Bean的构造函数参数、属性值、方法重写
  - 可以覆盖初始化方法、销毁方法、静态工厂方法
  - 依赖、自动装配模式、作用域等设置来自子定义
- **使用场景**:
  - XML配置中的父子Bean关系
  - 需要继承父Bean配置的场景

###### **GenericBeanDefinition (通用Bean定义)**
- **用途**: 声明式Bean定义的一站式解决方案
- **特点**:
  - 支持动态设置父Bean名称
  - 灵活性最高，可以动态改变父Bean关系
  - 适合XML配置和Bean后处理器操作
  - Spring 2.5+推荐使用
- **使用场景**:
  - XML配置文件中的Bean定义
  - 需要动态调整父Bean关系的场景

###### **AnnotatedBeanDefinition (注解Bean定义)**
- **用途**: 支持注解元数据的Bean定义
- **特点**:
  - 暴露注解元数据，无需加载类
  - 支持工厂方法元数据
  - 用于注解驱动的配置
- **实现类**:
  - `AnnotatedGenericBeanDefinition`: 通用注解Bean定义
  - `ScannedGenericBeanDefinition`: 扫描得到的注解Bean定义

##### 4. BeanDefinition在Spring Boot中的应用

在Spring Boot中，BeanDefinition的使用更加自动化：

- **@Component扫描**: 自动创建`ScannedGenericBeanDefinition`
- **@Bean方法**: 自动创建`ConfigurationClassBeanDefinition`(继承自`RootBeanDefinition`)
- **@Configuration**: 使用`ConfigurationClassBeanDefinition`
- **自动配置**: 使用`GenericBeanDefinition`进行动态配置

##### 5. 实际项目中的选择建议

- **XML配置**: 使用`GenericBeanDefinition`
- **注解配置**: 使用`AnnotatedGenericBeanDefinition`
- **程序化配置**: 使用`RootBeanDefinition`
- **需要继承**: 使用`ChildBeanDefinition`或`GenericBeanDefinition`
- **Spring Boot**: 主要使用自动生成的Bean定义

##### 6. BeanDefinition的核心属性

每个BeanDefinition都包含以下核心属性：

```java
// 基本信息
String getBeanClassName();           // Bean的类名
Class<?> getBeanClass();            // Bean的Class对象
String getScope();                  // 作用域 (singleton/prototype)
boolean isSingleton();              // 是否单例
boolean isPrototype();              // 是否原型

// 依赖关系
String[] getDependsOn();            // 依赖的Bean名称
String getParentName();             // 父Bean名称

// 生命周期
String getInitMethodName();         // 初始化方法名
String getDestroyMethodName();      // 销毁方法名
boolean isLazyInit();               // 是否懒加载

// 配置信息
ConstructorArgumentValues getConstructorArgumentValues();  // 构造函数参数
MutablePropertyValues getPropertyValues();                // 属性值
MethodOverrides getMethodOverrides();                     // 方法重写

// 自动装配
int getAutowireMode();              // 自动装配模式
boolean isAutowireCandidate();      // 是否作为自动装配候选
```
#### 重点测试方法
1. **简单引用测试**
   ```java
   @Test
   void simpleReference() {
       String PREFIX = "beans.";
       Properties p = new Properties();

       p.setProperty(PREFIX + "rod.(class)", TestBean.class.getName());
       p.setProperty(PREFIX + "rod.name", "Rod");

       p.setProperty(PREFIX + "kerry.(class)", TestBean.class.getName());
       p.setProperty(PREFIX + "kerry.name", "Kerry");
       p.setProperty(PREFIX + "kerry.age", "35");
       p.setProperty(PREFIX + "kerry.spouse(ref)", "rod");

       int count = registerBeanDefinitions(p, PREFIX);
       assertThat(count).as("2 beans registered, not " + count).isEqualTo(2);

       TestBean kerry = lbf.getBean("kerry", TestBean.class);
       assertThat(kerry.getName()).as("Kerry name is Kerry").isEqualTo("Kerry");
       ITestBean spouse = kerry.getSpouse();
       assertThat(spouse).as("Kerry spouse is non null").isNotNull();
       assertThat(spouse.getName()).as("Kerry spouse name is Rod").isEqualTo("Rod");
   }
   ```

2. **自引用测试**
   ```java
   @Test
   void selfReference() {
       MutablePropertyValues pvs = new MutablePropertyValues();
       pvs.add("spouse", new RuntimeBeanReference("self"));
       RootBeanDefinition bd = new RootBeanDefinition(TestBean.class);
       bd.setPropertyValues(pvs);
       lbf.registerBeanDefinition("self", bd);

       TestBean self = (TestBean) lbf.getBean("self");
       assertThat(self.getSpouse()).isEqualTo(self);
   }
   ```

#### 学习步骤
1. 运行 `simpleReference()` 测试
2. 在 `lbf.getBean("kerry", TestBean.class)` 设置断点
3. 跟踪Bean实例化过程
4. 理解BeanDefinition的作用

#### 关键源码位置
- [DefaultListableBeanFactory.getBean(String name)](spring-beans/src/main/java/org/springframework/beans/factory/support/DefaultListableBeanFactory.java)
- [AbstractBeanFactory.doGetBean(String name, Class<T> requiredType, Object[] args, boolean typeCheckOnly)](spring-beans/src/main/java/org/springframework/beans/factory/support/AbstractBeanFactory.java)
- [AbstractAutowireCapableBeanFactory.createBean(String beanName, RootBeanDefinition mbd, Object[] args)](spring-beans/src/main/java/org/springframework/beans/factory/support/AbstractAutowireCapableBeanFactory.java)

### 1.2 ApplicationContext 应用上下文

#### 核心测试用例
[AnnotationConfigApplicationContextTests.java](spring-context/src/test/java/org/springframework/context/annotation/AnnotationConfigApplicationContextTests.java)

#### 重点测试方法
1. **注解配置测试**
   ```java
   @Test
   void scanAndRefresh() {
       AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext();
       context.scan("org.springframework.context.annotation6");
       context.refresh();
       
       context.getBean(uncapitalize(ConfigForScanning.class.getSimpleName()));
       context.getBean("testBean"); // contributed by ConfigForScanning
       context.getBean(uncapitalize(ComponentForScanning.class.getSimpleName()));
       context.getBean(uncapitalize(Jsr330NamedForScanning.class.getSimpleName()));
       Map<String, Object> beans = context.getBeansWithAnnotation(Configuration.class);
       assertThat(beans).hasSize(1);
   }
   ```

### 1.3 Bean生命周期管理

#### 核心测试用例
[DefaultListableBeanFactoryTests.java](spring-beans/src/test/java/org/springframework/beans/factory/DefaultListableBeanFactoryTests.java)

#### 重点测试方法
1. **生命周期回调测试**
   ```java
   @Test
   void testInitializingBeanAndDisposableBean() {
       // 测试InitializingBean和DisposableBean接口
   }
   ```

2. **BeanPostProcessor测试**
   ```java
   @Test
   void testBeanPostProcessor() {
       // 测试Bean后处理器机制
   }
   ```
   
#### 生命周期总结

## 第二阶段：依赖注入机制 (第3-4周)

### 2.1 自动装配

#### 核心测试用例
[AutowiredAnnotationBeanPostProcessorTests.java](spring-beans/src/test/java/org/springframework/beans/factory/annotation/AutowiredAnnotationBeanPostProcessorTests.java)

#### 重点测试方法
1. **字段注入测试**
   ```java
   @Test
   void resourceInjection() {
       RootBeanDefinition bd = new RootBeanDefinition(ResourceInjectionBean.class);
       bd.setScope(BeanDefinition.SCOPE_PROTOTYPE);
       bf.registerBeanDefinition("annotatedBean", bd);
       TestBean tb = new TestBean();
       bf.registerSingleton("testBean", tb);

       ResourceInjectionBean bean = bf.getBean("annotatedBean", ResourceInjectionBean.class);
       assertThat(bean.getTestBean()).isSameAs(tb);
       assertThat(bean.getTestBean2()).isSameAs(tb);

       bean = bf.getBean("annotatedBean", ResourceInjectionBean.class);
       assertThat(bean.getTestBean()).isSameAs(tb);
       assertThat(bean.getTestBean2()).isSameAs(tb);

       assertThat(bf.getDependenciesForBean("annotatedBean")).isEqualTo(new String[] {"testBean"});
   }
   ```

2. **构造器注入测试**
   ```java
   @Test
   void extendedResourceInjection() {
       RootBeanDefinition bd = new RootBeanDefinition(TypedExtendedResourceInjectionBean.class);
       bd.setScope(BeanDefinition.SCOPE_PROTOTYPE);
       bf.registerBeanDefinition("annotatedBean", bd);
       TestBean tb = new TestBean();
       bf.registerSingleton("testBean", tb);
       NestedTestBean ntb = new NestedTestBean();
       bf.registerSingleton("nestedTestBean", ntb);

       TypedExtendedResourceInjectionBean bean = bf.getBean("annotatedBean", TypedExtendedResourceInjectionBean.class);
       assertThat(bean.getTestBean()).isSameAs(tb);
       assertThat(bean.getTestBean2()).isSameAs(tb);
       assertThat(bean.getTestBean3()).isSameAs(tb);
       assertThat(bean.getTestBean4()).isSameAs(tb);
       assertThat(bean.getNestedTestBean()).isSameAs(ntb);
       assertThat(bean.getBeanFactory()).isSameAs(bf);

       bean = bf.getBean("annotatedBean", TypedExtendedResourceInjectionBean.class);
       assertThat(bean.getTestBean()).isSameAs(tb);
       assertThat(bean.getTestBean2()).isSameAs(tb);
       assertThat(bean.getTestBean3()).isSameAs(tb);
       assertThat(bean.getTestBean4()).isSameAs(tb);
       assertThat(bean.getNestedTestBean()).isSameAs(ntb);
       assertThat(bean.getBeanFactory()).isSameAs(bf);

       assertThat(bf.getDependenciesForBean("annotatedBean")).isEqualTo(new String[] {"testBean", "nestedTestBean"});
   }
   ```

3. **可选依赖注入测试**
   ```java
   @Test
   void optionalResourceInjection() {
       bf.registerBeanDefinition("annotatedBean", new RootBeanDefinition(OptionalResourceInjectionBean.class));
       TestBean tb = new TestBean();
       bf.registerSingleton("testBean", tb);
       NestedTestBean ntb = new NestedTestBean();
       bf.registerSingleton("nestedTestBean", ntb);

       OptionalResourceInjectionBean bean = bf.getBean("annotatedBean", OptionalResourceInjectionBean.class);
       assertThat(bean.getTestBean()).isSameAs(tb);
       assertThat(bean.getTestBean2()).isSameAs(tb);
       assertThat(bean.getTestBean3()).isSameAs(tb);
       assertThat(bean.getNestedTestBean()).isSameAs(ntb);
       assertThat(bean.getBeanFactory()).isSameAs(bf);

       bean = bf.getBean("annotatedBean", OptionalResourceInjectionBean.class);
       assertThat(bean.getTestBean()).isSameAs(tb);
       assertThat(bean.getTestBean2()).isSameAs(tb);
       assertThat(bean.getTestBean3()).isSameAs(tb);
       assertThat(bean.getNestedTestBean()).isSameAs(ntb);
       assertThat(bean.getBeanFactory()).isSameAs(bf);

       assertThat(bf.getDependenciesForBean("annotatedBean")).isEqualTo(new String[] {"testBean", "nestedTestBean"});
   }
   ```

4. **ObjectProvider注入测试**
   ```java
   @Test
   void objectProviderInjectionWithSingletonTarget() {
       bf.registerBeanDefinition("annotatedBean", new RootBeanDefinition(ObjectProviderInjectionBean.class));
       bf.registerBeanDefinition("testBean", new RootBeanDefinition(TestBean.class));

       ObjectProviderInjectionBean bean = bf.getBean("annotatedBean", ObjectProviderInjectionBean.class);
       assertThat(bean.getTestBean()).isSameAs(bf.getBean("testBean"));
       assertThat(bean.getOptionalTestBean()).isSameAs(bf.getBean("testBean"));
       assertThat(bean.getOptionalTestBeanWithDefault()).isSameAs(bf.getBean("testBean"));
       assertThat(bean.consumeOptionalTestBean()).isEqualTo(bf.getBean("testBean"));
       assertThat(bean.getUniqueTestBean()).isSameAs(bf.getBean("testBean"));
       assertThat(bean.getUniqueTestBeanWithDefault()).isSameAs(bf.getBean("testBean"));
       assertThat(bean.consumeUniqueTestBean()).isEqualTo(bf.getBean("testBean"));

       List<TestBean> testBeans = bean.iterateTestBeans();
       assertThat(testBeans).hasSize(1);
       assertThat(testBeans).contains(bf.getBean("testBean", TestBean.class));
   }
   ```

#### 学习步骤
1. 运行 `resourceInjection()` 测试，理解字段注入机制
2. 运行 `extendedResourceInjection()` 测试，理解构造器注入
3. 运行 `optionalResourceInjection()` 测试，理解可选依赖
4. 运行 `objectProviderInjectionWithSingletonTarget()` 测试，理解ObjectProvider注入

#### 关键源码位置
- [AutowiredAnnotationBeanPostProcessor.postProcessProperties()](spring-beans/src/main/java/org/springframework/beans/factory/annotation/AutowiredAnnotationBeanPostProcessor.java)
- [AutowiredAnnotationBeanPostProcessor.findInjectionMetadata()](spring-beans/src/main/java/org/springframework/beans/factory/annotation/AutowiredAnnotationBeanPostProcessor.java)
- [InjectionMetadata.inject()](spring-beans/src/main/java/org/springframework/beans/factory/annotation/InjectionMetadata.java)

### 2.2 循环依赖解决

#### 核心测试用例
[DefaultListableBeanFactoryTests.java](spring-beans/src/test/java/org/springframework/beans/factory/DefaultListableBeanFactoryTests.java)

#### 重点测试方法
```java
@Test
void extensiveCircularReference() {
    for (int i = 0; i < 1000; i++) {
        MutablePropertyValues pvs = new MutablePropertyValues();
        pvs.addPropertyValue(new PropertyValue("spouse", new RuntimeBeanReference("bean" + (i < 99 ? i + 1 : 0))));
        RootBeanDefinition bd = new RootBeanDefinition(TestBean.class);
        bd.setPropertyValues(pvs);
        lbf.registerBeanDefinition("bean" + i, bd);
    }
    lbf.preInstantiateSingletons();

    for (int i = 0; i < 1000; i++) {
        TestBean bean = (TestBean) lbf.getBean("bean" + i);
        TestBean otherBean = (TestBean) lbf.getBean("bean" + (i < 99 ? i + 1 : 0));
        assertThat(bean.getSpouse()).isSameAs(otherBean);
    }
}
```

#### 学习步骤
1. 运行 `extensiveCircularReference()` 测试
2. 在 `lbf.getBean("bean" + i)` 设置断点
3. 观察循环依赖的解决过程
4. 理解三级缓存的作用

#### 关键源码位置
- [DefaultSingletonBeanRegistry.getSingleton(String beanName, boolean allowEarlyReference)](spring-beans/src/main/java/org/springframework/beans/factory/support/DefaultSingletonBeanRegistry.java)
- [AbstractAutowireCapableBeanFactory.doCreateBean(String beanName, RootBeanDefinition mbd, Object[] args)](spring-beans/src/main/java/org/springframework/beans/factory/support/AbstractAutowireCapableBeanFactory.java)
- [DefaultSingletonBeanRegistry.addSingletonFactory(String beanName, ObjectFactory<?> singletonFactory)](spring-beans/src/main/java/org/springframework/beans/factory/support/DefaultSingletonBeanRegistry.java)

### 2.3 类型转换和属性编辑器

#### 核心测试用例
- [DefaultConversionServiceTests.java](spring-core/src/test/java/org/springframework/core/convert/converter/DefaultConversionServiceTests.java)
- [CustomEditorTests.java](spring-beans/src/test/java/org/springframework/beans/propertyeditors/CustomEditorTests.java)

#### 重点测试方法
1. **基础类型转换测试**
   ```java
   @Test
   void stringToCharacter() {
       assertThat(conversionService.convert("1", Character.class)).isEqualTo(Character.valueOf('1'));
   }

   @Test
   void stringToCharacterEmptyString() {
       assertThat(conversionService.convert("", Character.class)).isNull();
   }

   @Test
   void stringToCharacterInvalidString() {
       assertThatExceptionOfType(ConversionFailedException.class).isThrownBy(() ->
               conversionService.convert("invalid", Character.class));
   }
   ```

2. **集合类型转换测试**
   ```java
   @Test
   void stringToCollection() {
       String source = "1,2,3,4,5";
       List<Integer> result = conversionService.convert(source, 
           TypeDescriptor.valueOf(String.class), 
           TypeDescriptor.collection(List.class, TypeDescriptor.valueOf(Integer.class)));
       assertThat(result).containsExactly(1, 2, 3, 4, 5);
   }
   ```

3. **自定义属性编辑器测试**
   ```java
   @Test
   void testCustomEditor() {
       DefaultListableBeanFactory bf = new DefaultListableBeanFactory();
       bf.registerCustomEditor(TestBean.class, "name", new PropertyEditorSupport() {
           @Override
           public void setAsText(String text) {
               setValue(new TestBean(text.toUpperCase()));
           }
       });
       
       TestBean tb = new TestBean();
       bf.registerSingleton("testBean", tb);
       
       // 测试自定义编辑器
       PropertyValues pvs = new MutablePropertyValues();
       pvs.add("name", "test");
       bf.findPropertyEditor(TestBean.class, "name").setAsText("test");
   }
   ```

#### 学习步骤
1. 运行 `stringToCharacter()` 测试，理解基础类型转换
2. 运行 `stringToCollection()` 测试，理解集合类型转换
3. 运行 `testCustomEditor()` 测试，理解自定义属性编辑器

#### 关键源码位置
- [DefaultConversionService.convert()](spring-core/src/main/java/org/springframework/core/convert/support/DefaultConversionService.java)
- [PropertyEditorRegistry.registerCustomEditor()](spring-beans/src/main/java/org/springframework/beans/PropertyEditorRegistry.java)
- [BeanWrapperImpl.setPropertyValue()](spring-beans/src/main/java/org/springframework/beans/BeanWrapperImpl.java)

## 第三阶段：AOP实现原理 (第5-6周)

### 3.1 代理创建

#### 核心测试用例
[ProxyFactoryTests.java](spring-aop/src/test/java/org/springframework/aop/framework/ProxyFactoryTests.java)

#### 重点测试方法
1. **代理工厂基础测试**
   ```java
   @Test
   void indexOfMethods() {
       TestBean target = new TestBean();
       ProxyFactory pf = new ProxyFactory(target);
       NopInterceptor nop = new NopInterceptor();
       Advisor advisor = new DefaultPointcutAdvisor(new CountingBeforeAdvice());
       Advised advised = (Advised) pf.getProxy();
       // Can use advised and ProxyFactory interchangeably
       advised.addAdvice(nop);
       pf.addAdvisor(advisor);
       assertThat(pf.indexOf(new NopInterceptor())).isEqualTo(-1);
       assertThat(pf.indexOf(nop)).isEqualTo(0);
       assertThat(pf.indexOf(advisor)).isEqualTo(1);
       assertThat(advised.indexOf(new DefaultPointcutAdvisor(null))).isEqualTo(-1);
   }
   ```

2. **通知器移除测试**
   ```java
   @Test
   void removeAdvisorByReference() {
       TestBean target = new TestBean();
       ProxyFactory pf = new ProxyFactory(target);
       NopInterceptor nop = new NopInterceptor();
       CountingBeforeAdvice cba = new CountingBeforeAdvice();
       Advisor advisor = new DefaultPointcutAdvisor(cba);
       pf.addAdvice(nop);
       pf.addAdvisor(advisor);
       ITestBean proxied = (ITestBean) pf.getProxy();
       proxied.setAge(5);
       assertThat(cba.getCalls()).isEqualTo(1);
       assertThat(nop.getCount()).isEqualTo(1);
       assertThat(pf.removeAdvisor(advisor)).isTrue();
       assertThat(proxied.getAge()).isEqualTo(5);
       assertThat(cba.getCalls()).isEqualTo(1);
       assertThat(nop.getCount()).isEqualTo(2);
       assertThat(pf.removeAdvisor(new DefaultPointcutAdvisor(null))).isFalse();
   }
   ```

3. **引入通知测试**
   ```java
   @Test
   void introductionInterceptor() {
       TestBean target = new TestBean();
       ProxyFactory pf = new ProxyFactory(target);
       pf.setInterfaces(ITestBean.class, IOther.class);
       long t = 555555L;
       TimestampIntroductionInterceptor ti = new TimestampIntroductionInterceptor(t);
       pf.addAdvisor(new DefaultIntroductionAdvisor(ti, TimeStamped.class));

       Class<?>[] newProxiedInterfaces = pf.getProxiedInterfaces();
       assertThat(newProxiedInterfaces).as("Advisor proxies one more interface after introduction").hasSize(oldProxiedInterfaces.length + 1);

       TimeStamped ts = (TimeStamped) pf.getProxy();
       assertThat(ts.getTimeStamp()).isEqualTo(t);
       // Shouldn't fail;
       ((IOther) ts).absquatulate();
   }
   ```

#### 代理选择策略
```java
// DefaultAopProxyFactory.createAopProxy()
public AopProxy createAopProxy(AdvisedSupport config) throws AopConfigException {
    if (config.isOptimize() || config.isProxyTargetClass() || 
        hasNoUserSuppliedProxyInterfaces(config)) {
        Class<?> targetClass = config.getTargetClass();
        if (targetClass == null) {
            throw new AopConfigException("TargetSource cannot determine target class");
        }
        if (targetClass.isInterface() || Proxy.isProxyClass(targetClass)) {
            return new JdkDynamicAopProxy(config);
        }
        return new CglibAopProxy(config);
    }
    else {
        return new JdkDynamicAopProxy(config);
    }
}
```

#### 学习步骤
1. 运行 `indexOfMethods()` 测试，理解代理工厂的基本操作
2. 运行 `removeAdvisorByReference()` 测试，理解通知器的管理
3. 运行 `introductionInterceptor()` 测试，理解引入通知机制

#### 关键源码位置
- [ProxyFactory.getProxy()](spring-aop/src/main/java/org/springframework/aop/framework/ProxyFactory.java)
- [DefaultAopProxyFactory.createAopProxy()](spring-aop/src/main/java/org/springframework/aop/framework/DefaultAopProxyFactory.java)
- [JdkDynamicAopProxy.invoke()](spring-aop/src/main/java/org/springframework/aop/framework/JdkDynamicAopProxy.java)

### 3.2 切面织入

#### 核心测试用例
[AspectJExpressionPointcutTests.java](spring-aop/src/test/java/org/springframework/aop/aspectj/AspectJExpressionPointcutTests.java)

#### 重点测试方法
1. **精确匹配测试**
   ```java
   @Test
   void testMatchExplicit() {
       String expression = "execution(int org.springframework.beans.testfixture.beans.TestBean.getAge())";

       Pointcut pointcut = getPointcut(expression);
       ClassFilter classFilter = pointcut.getClassFilter();
       MethodMatcher methodMatcher = pointcut.getMethodMatcher();

       assertMatchesTestBeanClass(classFilter);

       assertThat(methodMatcher.isRuntime()).as("Should not be a runtime match").isFalse();
       assertMatchesGetAge(methodMatcher);
       assertThat(methodMatcher.matches(setAge, TestBean.class)).as("Expression should match setAge() method").isFalse();
   }
   ```

2. **类型模式匹配测试**
   ```java
   @Test
   void testMatchWithTypePattern() {
       String expression = "execution(* *..TestBean.*Age(..))";

       Pointcut pointcut = getPointcut(expression);
       ClassFilter classFilter = pointcut.getClassFilter();
       MethodMatcher methodMatcher = pointcut.getMethodMatcher();

       assertMatchesTestBeanClass(classFilter);

       assertThat(methodMatcher.isRuntime()).as("Should not be a runtime match").isFalse();
       assertMatchesGetAge(methodMatcher);
       assertThat(methodMatcher.matches(setAge, TestBean.class)).as("Expression should match setAge(int) method").isTrue();
   }
   ```

3. **this和target测试**
   ```java
   @Test
   void testThis() throws SecurityException, NoSuchMethodException{
       testThisOrTarget("this");
   }

   @Test
   void testTarget() throws SecurityException, NoSuchMethodException {
       testThisOrTarget("target");
   }

   private void testThisOrTarget(String which) throws SecurityException, NoSuchMethodException {
       String matchesTestBean = which + "(org.springframework.beans.testfixture.beans.TestBean)";
       String matchesIOther = which + "(org.springframework.beans.testfixture.beans.IOther)";
       AspectJExpressionPointcut testBeanPc = new AspectJExpressionPointcut();
       testBeanPc.setExpression(matchesTestBean);

       AspectJExpressionPointcut iOtherPc = new AspectJExpressionPointcut();
       iOtherPc.setExpression(matchesIOther);

       assertThat(testBeanPc.matches(TestBean.class)).isTrue();
       assertThat(testBeanPc.matches(getAge, TestBean.class)).isTrue();
       assertThat(iOtherPc.matches(OtherIOther.class.getMethod("absquatulate"), OtherIOther.class)).isTrue();
       assertThat(testBeanPc.matches(OtherIOther.class.getMethod("absquatulate"), OtherIOther.class)).isFalse();
   }
   ```

#### 学习步骤
1. 运行 `testMatchExplicit()` 测试，理解精确匹配机制
2. 运行 `testMatchWithTypePattern()` 测试，理解模式匹配
3. 运行 `testThis()` 和 `testTarget()` 测试，理解this和target表达式

#### 关键源码位置
- [AspectJExpressionPointcut.matches()](spring-aop/src/main/java/org/springframework/aop/aspectj/AspectJExpressionPointcut.java)
- [AspectJExpressionPointcut.buildPointcutExpression()](spring-aop/src/main/java/org/springframework/aop/aspectj/AspectJExpressionPointcut.java)
- [AspectJExpressionPointcut.buildPointcutExpression()](spring-aop/src/main/java/org/springframework/aop/aspectj/AspectJExpressionPointcut.java)

## 第四阶段：配置管理 (第7-8周)

### 4.1 注解配置解析

#### 核心测试用例
[ConfigurationClassPostProcessorTests.java](spring-context/src/test/java/org/springframework/context/annotation/ConfigurationClassPostProcessorTests.java)

#### 重点测试方法
1. **配置类增强测试**
   ```java
   @Test
   void enhancementIsPresentBecauseSingletonSemanticsAreRespected() {
       beanFactory.registerBeanDefinition("config", new RootBeanDefinition(SingletonBeanConfig.class));
       ConfigurationClassPostProcessor pp = new ConfigurationClassPostProcessor();
       pp.postProcessBeanFactory(beanFactory);
       assertThat(((RootBeanDefinition) beanFactory.getBeanDefinition("config")).hasBeanClass()).isTrue();
       assertThat(((RootBeanDefinition) beanFactory.getBeanDefinition("config")).getBeanClass().getName()).contains(ClassUtils.CGLIB_CLASS_SEPARATOR);
       Foo foo = beanFactory.getBean("foo", Foo.class);
       Bar bar = beanFactory.getBean("bar", Bar.class);
       assertThat(bar.foo).isSameAs(foo);
       assertThat(beanFactory.getDependentBeans("foo")).contains("bar");
       assertThat(beanFactory.getDependentBeans("config")).contains("foo");
       assertThat(beanFactory.getDependentBeans("config")).contains("bar");
   }
   ```

2. **非增强配置类测试**
   ```java
   @Test
   void enhancementIsNotPresentForProxyBeanMethodsFlagSetToFalse() {
       beanFactory.registerBeanDefinition("config", new RootBeanDefinition(NonEnhancedSingletonBeanConfig.class));
       ConfigurationClassPostProcessor pp = new ConfigurationClassPostProcessor();
       pp.postProcessBeanFactory(beanFactory);
       assertThat(((RootBeanDefinition) beanFactory.getBeanDefinition("config")).hasBeanClass()).isTrue();
       assertThat(((RootBeanDefinition) beanFactory.getBeanDefinition("config")).getBeanClass().getName()).doesNotContain(ClassUtils.CGLIB_CLASS_SEPARATOR);
       Foo foo = beanFactory.getBean("foo", Foo.class);
       Bar bar = beanFactory.getBean("bar", Bar.class);
       assertThat(bar.foo).isNotSameAs(foo);
   }
   ```

3. **静态方法配置测试**
   ```java
   @Test
   void enhancementIsNotPresentForStaticMethods() {
       beanFactory.registerBeanDefinition("config", new RootBeanDefinition(StaticSingletonBeanConfig.class));
       ConfigurationClassPostProcessor pp = new ConfigurationClassPostProcessor();
       pp.postProcessBeanFactory(beanFactory);
       assertThat(((RootBeanDefinition) beanFactory.getBeanDefinition("config")).hasBeanClass()).isTrue();
       assertThat(((RootBeanDefinition) beanFactory.getBeanDefinition("config")).getBeanClass().getName()).doesNotContain(ClassUtils.CGLIB_CLASS_SEPARATOR);
       assertThat(((RootBeanDefinition) beanFactory.getBeanDefinition("foo")).hasBeanClass()).isTrue();
       assertThat(((RootBeanDefinition) beanFactory.getBeanDefinition("bar")).hasBeanClass()).isTrue();
       Foo foo = beanFactory.getBean("foo", Foo.class);
       Bar bar = beanFactory.getBean("bar", Bar.class);
       assertThat(bar.foo).isNotSameAs(foo);
   }
   ```

#### 学习步骤
1. 运行 `enhancementIsPresentBecauseSingletonSemanticsAreRespected()` 测试，理解配置类增强机制
2. 运行 `enhancementIsNotPresentForProxyBeanMethodsFlagSetToFalse()` 测试，理解非增强配置类
3. 运行 `enhancementIsNotPresentForStaticMethods()` 测试，理解静态方法配置

#### 关键源码位置
- [ConfigurationClassPostProcessor.postProcessBeanFactory()](spring-context/src/main/java/org/springframework/context/annotation/ConfigurationClassPostProcessor.java)
- [ConfigurationClassEnhancer.enhance()](spring-context/src/main/java/org/springframework/context/annotation/ConfigurationClassEnhancer.java)
- [ConfigurationClassParser.parse()](spring-context/src/main/java/org/springframework/context/annotation/ConfigurationClassParser.java)

### 4.2 条件装配

#### 核心测试用例
[ConfigurationClassWithConditionTests.java](spring-context/src/test/java/org/springframework/context/annotation/ConfigurationClassWithConditionTests.java)

#### 重点测试方法
1. **条件Bean匹配测试**
   ```java
   @Test
   void conditionalOnMissingBeanMatch() {
       AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
       ctx.register(BeanOneConfiguration.class, BeanTwoConfiguration.class);
       ctx.refresh();
       assertThat(ctx.containsBean("bean1")).isTrue();
       assertThat(ctx.containsBean("bean2")).isFalse();
       assertThat(ctx.containsBean("configurationClassWithConditionTests.BeanTwoConfiguration")).isFalse();
   }
   ```

2. **条件Bean不匹配测试**
   ```java
   @Test
   void conditionalOnMissingBeanNoMatch() {
       AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
       ctx.register(BeanTwoConfiguration.class);
       ctx.refresh();
       assertThat(ctx.containsBean("bean1")).isFalse();
       assertThat(ctx.containsBean("bean2")).isTrue();
       assertThat(ctx.containsBean("configurationClassWithConditionTests.BeanTwoConfiguration")).isTrue();
   }
   ```

3. **条件Bean存在匹配测试**
   ```java
   @Test
   void conditionalOnBeanMatch() {
       AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
       ctx.register(BeanOneConfiguration.class, BeanThreeConfiguration.class);
       ctx.refresh();
       assertThat(ctx.containsBean("bean1")).isTrue();
       assertThat(ctx.containsBean("bean3")).isTrue();
   }
   ```

#### 学习步骤
1. 运行 `conditionalOnMissingBeanMatch()` 测试，理解条件Bean匹配机制
2. 运行 `conditionalOnMissingBeanNoMatch()` 测试，理解条件Bean不匹配情况
3. 运行 `conditionalOnBeanMatch()` 测试，理解条件Bean存在匹配

#### 关键源码位置
- [ConditionEvaluator.shouldSkip()](spring-context/src/main/java/org/springframework/context/annotation/ConditionEvaluator.java)
- [ConfigurationClassParser.processConfigurationClass()](spring-context/src/main/java/org/springframework/context/annotation/ConfigurationClassParser.java)
- [Condition.matches()](spring-context/src/main/java/org/springframework/context/annotation/Condition.java)

## 第五阶段：Web框架 (第9-10周)

### 5.1 Spring MVC

#### 核心测试用例
[RequestMappingHandlerMappingTests.java](spring-webmvc/src/test/java/org/springframework/web/servlet/mvc/method/annotation/RequestMappingHandlerMappingTests.java)

#### 重点测试方法
1. **请求映射构建器配置测试**
   ```java
   @Test
   void builderConfiguration() {
       RequestMappingHandlerMapping mapping = createMapping();

       RequestMappingInfo.BuilderConfiguration config = mapping.getBuilderConfiguration();
       assertThat(config).isNotNull();

       mapping.afterPropertiesSet();
   }
   ```

2. **组合注解映射测试**
   ```java
   @Test // SPR-14988
   void postMappingOverridesConsumesFromTypeLevelAnnotation() {
       RequestMappingInfo requestMappingInfo = assertComposedAnnotationMapping(RequestMethod.POST);

       ConsumesRequestCondition condition = requestMappingInfo.getConsumesCondition();
       assertThat(condition.getConsumableMediaTypes()).containsOnly(MediaType.APPLICATION_XML);
   }
   ```

3. **HTTP方法映射测试**
   ```java
   @Test
   void getMapping() {
       assertComposedAnnotationMapping(RequestMethod.GET);
   }

   @Test
   void postMapping() {
       assertComposedAnnotationMapping(RequestMethod.POST);
   }

   @Test
   void putMapping() {
       assertComposedAnnotationMapping(RequestMethod.PUT);
   }

   @Test
   void deleteMapping() {
       assertComposedAnnotationMapping(RequestMethod.DELETE);
   }

   @Test
   void patchMapping() {
       assertComposedAnnotationMapping(RequestMethod.PATCH);
   }
   ```

#### 学习步骤
1. 运行 `builderConfiguration()` 测试，理解请求映射构建器配置
2. 运行 `postMappingOverridesConsumesFromTypeLevelAnnotation()` 测试，理解组合注解映射
3. 运行各种HTTP方法映射测试，理解不同HTTP方法的处理

#### 关键源码位置
- [RequestMappingHandlerMapping.getMappingForMethod()](spring-webmvc/src/main/java/org/springframework/web/servlet/mvc/method/annotation/RequestMappingHandlerMapping.java)
- [RequestMappingInfo.paths()](spring-webmvc/src/main/java/org/springframework/web/servlet/mvc/method/RequestMappingInfo.java)
- [RequestMappingHandlerAdapter.handleInternal()](spring-webmvc/src/main/java/org/springframework/web/servlet/mvc/method/annotation/RequestMappingHandlerAdapter.java)

### 5.2 Spring WebFlux

#### 核心测试用例
[RouterFunctionTests.java](spring-webflux/src/test/java/org/springframework/web/reactive/function/server/RouterFunctionTests.java)

#### 重点测试方法
1. **路由函数组合测试**
   ```java
   @Test
   void and() {
       HandlerFunction<ServerResponse> handlerFunction = request -> ServerResponse.ok().build();
       RouterFunction<ServerResponse> routerFunction1 = request -> Mono.empty();
       RouterFunction<ServerResponse> routerFunction2 = request -> Mono.just(handlerFunction);

       RouterFunction<ServerResponse> result = routerFunction1.and(routerFunction2);
       assertThat(result).isNotNull();

       MockServerHttpRequest mockRequest = MockServerHttpRequest.get("https://example.com").build();
       ServerRequest request = new DefaultServerRequest(MockServerWebExchange.from(mockRequest), Collections.emptyList());
       Mono<HandlerFunction<ServerResponse>> resultHandlerFunction = result.route(request);

       StepVerifier.create(resultHandlerFunction)
               .expectNext(handlerFunction)
               .expectComplete()
               .verify();
   }
   ```

2. **路由函数过滤测试**
   ```java
   @Test
   void filter() {
       Mono<String> stringMono = Mono.just("42");
       HandlerFunction<EntityResponse<Mono<String>>> handlerFunction =
               request -> EntityResponse.fromPublisher(stringMono, String.class).build();
       RouterFunction<EntityResponse<Mono<String>>> routerFunction =
               request -> Mono.just(handlerFunction);

       HandlerFilterFunction<EntityResponse<Mono<String>>, EntityResponse<Mono<Integer>>> filterFunction =
               (request, next) -> next.handle(request).flatMap(
                       response -> {
                           Mono<Integer> intMono = response.entity()
                                   .map(Integer::parseInt);
                           return EntityResponse.fromPublisher(intMono, Integer.class).build();
                       });

       RouterFunction<EntityResponse<Mono<Integer>>> result = routerFunction.filter(filterFunction);
       assertThat(result).isNotNull();

       MockServerHttpRequest mockRequest = MockServerHttpRequest.get("https://example.com").build();
       ServerRequest request = new DefaultServerRequest(MockServerWebExchange.from(mockRequest), Collections.emptyList());
       Mono<EntityResponse<Mono<Integer>>> responseMono =
               result.route(request).flatMap(hf -> hf.handle(request));

       StepVerifier.create(responseMono)
               .consumeNextWith(
                       serverResponse ->
                           StepVerifier.create(serverResponse.entity())
                                   .expectNext(42)
                                   .expectComplete()
                                   .verify()
                       )
               .expectComplete()
               .verify();
   }
   ```

3. **路由函数属性测试**
   ```java
   @Test
   void attributes() {
       RouterFunction<ServerResponse> route = RouterFunctions.route(
               GET("/atts/1"), request -> ServerResponse.ok().build())
               .withAttribute("foo", "bar")
               .withAttribute("baz", "qux")
               .and(RouterFunctions.route(GET("/atts/2"), request -> ServerResponse.ok().build())
               .withAttributes(atts -> {
                   atts.put("foo", "bar");
                   atts.put("baz", "qux");
               }))
               .and(RouterFunctions.nest(path("/atts"),
                       RouterFunctions.route(GET("/3"), request -> ServerResponse.ok().build())
                       .withAttribute("foo", "bar")
                       .and(RouterFunctions.route(GET("/4"), request -> ServerResponse.ok().build())
                       .withAttribute("baz", "qux"))
                       .and(RouterFunctions.nest(path("/5"),
                               RouterFunctions.route(method(GET), request -> ServerResponse.ok().build())
                               .withAttribute("foo", "n3"))
                       .withAttribute("foo", "n2")))
               .withAttribute("foo", "n1"));

       AttributesTestVisitor visitor = new AttributesTestVisitor();
       route.accept(visitor);
       assertThat(visitor.routerFunctionsAttributes()).containsExactly(
               List.of(Map.of("foo", "bar", "baz", "qux")),
               List.of(Map.of("foo", "bar", "baz", "qux")),
               List.of(Map.of("foo", "bar"), Map.of("foo", "n1")),
               List.of(Map.of("baz", "qux"), Map.of("foo", "n1")),
               List.of(Map.of("foo", "n3"), Map.of("foo", "n2"), Map.of("foo", "n1"))
       );
       assertThat(visitor.visitCount()).isEqualTo(7);
   }
   ```

#### 学习步骤
1. 运行 `and()` 测试，理解路由函数组合机制
2. 运行 `filter()` 测试，理解路由函数过滤机制
3. 运行 `attributes()` 测试，理解路由函数属性机制

#### 关键源码位置
- [RouterFunction.route()](spring-webflux/src/main/java/org/springframework/web/reactive/function/server/RouterFunction.java)
- [RouterFunctions.route()](spring-webflux/src/main/java/org/springframework/web/reactive/function/server/RouterFunctions.java)
- [HandlerFunction.handle()](spring-webflux/src/main/java/org/springframework/web/reactive/function/server/HandlerFunction.java)

## 面试重点

### 1. IoC容器
- **BeanFactory vs ApplicationContext**: 理解两者的关系和区别
- **Bean生命周期**: 掌握完整的Bean生命周期流程
- **循环依赖解决**: 理解三级缓存机制

### 2. 依赖注入
- **三种注入方式**: 构造器注入、setter注入、字段注入
- **自动装配原理**: 理解@Autowired的工作原理
- **类型转换机制**: 了解ConversionService的作用

### 3. AOP
- **代理模式选择**: JDK动态代理 vs CGLIB
- **切面织入过程**: 理解AOP的执行流程
- **通知执行顺序**: 掌握@Order和@Priority的作用

### 4. 配置管理
- **注解配置解析**: 理解@Configuration的处理机制
- **条件装配**: 掌握@Conditional的工作原理
- **组件扫描**: 了解@ComponentScan的实现

### 5. Web框架
- **Spring MVC请求处理**: 理解DispatcherServlet的工作流程
- **WebFlux响应式编程**: 掌握Reactor编程模型
- **参数解析和返回值处理**: 了解HandlerMethodArgumentResolver

## 学习技巧

### 1. 调试技巧
- **条件断点**: 在断点上右键设置条件
- **表达式求值**: 在调试时使用Evaluate Expression
- **调用栈分析**: 观察方法调用链

### 2. 源码阅读技巧
- **从接口开始**: 先看接口定义，再看实现
- **关注注释**: Spring源码注释很详细
- **画图理解**: 画出类图和时序图

### 3. 测试用例分析
- **阅读测试方法名**: 了解测试的目的
- **查看测试数据**: 理解测试场景
- **分析断言**: 了解期望的行为
- **跟踪源码**: 理解实现细节

### 4. 学习顺序建议
1. 先运行测试，确保测试通过
2. 在关键方法设置断点
3. 单步调试，观察执行流程
4. 阅读相关源码，理解实现原理
5. 修改测试数据，验证理解

## 实践建议

### 1. 每日学习计划
- **上午**: 运行测试用例，理解功能
- **下午**: 调试源码，深入原理
- **晚上**: 总结笔记，整理知识点

### 2. 学习工具
- **IDE**: IntelliJ IDEA (推荐)
- **调试工具**: IDE内置调试器
- **文档工具**: Markdown笔记

### 3. 进度跟踪
- **每周完成一个模块的学习**
- **记录关键测试用例和源码位置**
- **总结面试重点知识点**

### 4. 常见问题解决
```bash
# 构建失败
./gradlew clean
./gradlew build -x test

# 运行特定测试
./gradlew :spring-beans:test --tests DefaultListableBeanFactoryTests.simpleReference

# IDE导入问题
# 确保使用JDK 17+
# 重新导入Gradle项目
# 清理IDE缓存
```

---

*本指南将根据学习进度和实际需求持续更新优化* 