---
title: spring项目xml及static方法读取配置文件
layout: post
---
今天在项目中配置webservice，想要将webservice.xml中publish的地址公共部分配置到配置文件中。这个想法来源于MyBatis的配置文件可以通过配置<properties />属性来加载配置文件，并访问配置文件中的KV值。通过尝试，在借助spring的配置完成了这个功能。在web.xml中配置

```
 <bean class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
     <property name="locations"> <!-- PropertyPlaceholderConfigurer类中有个locations属性，接收的是一个数组，即我们可以在下面配好多个properties文件 -->
         <array>
             <value>classpath:config.properties</value>
         </array>
     </property>
 </bean>
```

以上代码来源于网络。相同的是使用"org.springframework.beans.factory.config.PropertyPlaceholderConfigurer"来加载。这样在webservice.xml就可以通过${KEY}来访问配置文件中的配置项。
另外，在今天的工作中还遇到了需要Java中的static方法读取配置文件的情况。由于想将读取配置作为一个静态方法供其他类调用，这里使用了static方法来加载配置。以往非静态方法的加载方式如下：

```
InputStream inStream = this.getClass().getClassLoader().getResourceAsStream("config.properties");
```

在静态方法中使用this会报错。后来使用了如下方法：

```
InputStream inStream = Thread.currentThread().getContextClassLoader().getResourceAsStream("config.properties");
```

然后使用如下方法即可加载配置文件：

```
Properties p = new Properties();
p.load(inStream);
String username = p.getProperty("username");
```

经过查阅，Java中的static方法不属于类的实例而属于类，所以不能访问this。
另外，读取配置文件的方式还有：

```
InputStream inStream = TestProperties.class.getClassLoader().getResourceAsStream("config.properties");
```

等等。


