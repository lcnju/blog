---
title: 使用springmvc和hibernate搭建restful服务
date: 2017-01-17 15:48:22
tags:
  - springmvc
  - hibernate
  - maven
---

本文介绍使用springmvc加hibernate来搭建restful服务，项目目录如下。
本文全部采用**注解**的方式来进行配置

```
demo/
├── pom.xml
└── src
    └── main
        ├── java
        │   └── demo
        │       ├── controller
        │       │   └── DemoController.java
        │       ├── dao
        │       │   ├── BaseDao.java
        │       │   ├── DemoDao.java
        │       │   └── impl
        │       │       ├── BaseDaoImpl.java
        │       │       └── DemoDaoImpl.java
        │       ├── model
        │       │   └── Demo.java
        │       ├── service
        │       │   ├── DemoService.java
        │       │   └── impl
        │       │       └── DemoServiceImpl.java
        │       ├── until
        │       │   ├── FormatUtil.java
        │       │   ├── StringUtil.java
        │       │   └── UploadUtil.java
        │       └── vo
        │           ├── DemoVo.java
        │           ├── Page.java
        │           └── Result.java
        ├── resources
        │   ├── ApplicationContext.xml
        │   ├── demo.sql
        │   └── SpringMVC.xml
        └── webapp
            ├── fileUpload
            │   └── temp
            └── WEB-INF
                └── web.xml
```

# 使用maven来构建项目

maven的配置文件`pom.xml`内容如下

``` xml pom.xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>lc</groupId>
    <artifactId>demo</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>war</packaging>
    <name>demo</name>

    <properties>
        <spring.version>4.3.2.RELEASE</spring.version>
        <hibernate.version>5.1.2.Final</hibernate.version>
        <jackson.version>2.8.3</jackson.version>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <dependencies>
        <!--springmvc-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
            <version>${spring.version}</version>
        </dependency>

        <!--spring-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-orm</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-tx</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-aspects</artifactId>
            <version>${spring.version}</version>
        </dependency>

        <!--hibernate-->
        <dependency>
            <groupId>org.hibernate</groupId>
            <artifactId>hibernate-core</artifactId>
            <version>${hibernate.version}</version>
        </dependency>

        <!--mysql-->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.39</version>
        </dependency>

        <!--json to object-->
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-core</artifactId>
            <version>${jackson.version}</version>
        </dependency>
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
            <version>${jackson.version}</version>
        </dependency>

        <!--everything base-->
        <dependency>
            <groupId>commons-codec</groupId>
            <artifactId>commons-codec</artifactId>
            <version>1.10</version>
        </dependency>
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>javax.servlet-api</artifactId>
            <version>3.1.0</version>
            <scope>provided</scope>
        </dependency>

        <!--upload file-->
        <dependency>
            <groupId>commons-fileupload</groupId>
            <artifactId>commons-fileupload</artifactId>
            <version>1.3.1</version>
        </dependency>

        <!--test-->
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-test</artifactId>
            <version>${spring.version}</version>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <finalName>demo</finalName>
        <resources>
            <resource>
                <directory>src\main\java</directory>
            </resource>
            <resource>
                <directory>src\main\resources</directory>
            </resource>
        </resources>
        <testResources>
            <testResource>
                <directory>src\main\webapp</directory>
            </testResource>
        </testResources>

        <plugins>
            <!--编译项目-->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.5.1</version>
                <configuration>
                    <source>1.7</source>
                    <target>1.7</target>
                    <encoding>UTF-8</encoding>
                </configuration>
            </plugin>

            <!--tomcat服务器插件-->
            <plugin>
                <groupId>org.apache.tomcat.maven</groupId>
                <artifactId>tomcat7-maven-plugin</artifactId>
                <version>2.2</version>
                <configuration>
                    <port>8080</port>
                    <path>/</path>
                    <uriEncoding>UTF-8</uriEncoding>
                    <server>tomcat7</server>
                </configuration>
            </plugin>

            <!--生成javadoc-->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-javadoc-plugin</artifactId>
                <version>2.10.4</version>
                <configuration>
                    <useStandardDocletOptions>false</useStandardDocletOptions>
                    <show>private</show>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

# 引入框架

修改web.xml，将请求全部交由springmvc，具体代码如下

```xml web.xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://java.sun.com/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
         version="3.0">
    <!--设置spring框架配置文件路径-->
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:ApplicationContext.xml</param-value>
    </context-param>
    <!--字符集过滤 统一使用UTF-8-->
    <filter>
        <filter-name>CharacterEncodingFilter</filter-name>
        <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
        <init-param>
            <param-name>encoding</param-name>
            <param-value>UTF-8</param-value>
        </init-param>
        <init-param>
            <param-name>forceEncoding</param-name>
            <param-value>true</param-value>
        </init-param>
    </filter>
    <filter-mapping>
        <filter-name>CharacterEncodingFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
    <!--添加springMVC-->
    <servlet>
        <servlet-name>dispatcher</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:SpringMVC.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>dispatcher</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>
    <!--添加spring监听 启动容器时自动装配配置文件中的信息-->
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>
</web-app>
```

# spring及hibernate配置文件

```xml ApplicationContext.xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="
       http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.3.xsd
       http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-4.3.xsd
       http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

    <context:component-scan base-package="demo" />

    <!--配置数据库数据-->
    <bean id="dataSource"
          class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="driverClassName" value="com.mysql.jdbc.Driver" />
        <property name="url" value="jdbc:mysql://localhost:3306/demodb?useUnicode=true&amp;characterEncoding=UTF-8&amp;autoReconnect=true" />
        <property name="username" value="demo" />
        <property name="password" value="ustc2017demo" />
    </bean>

    <!-- 配置sessionFactory -->
    <bean id="sessionFactory"
          class="org.springframework.orm.hibernate5.LocalSessionFactoryBean">
        <property name="dataSource" ref="dataSource" />
        <property name="packagesToScan" value="demo.model" />
        <property name="hibernateProperties">
            <props>
                <prop key="hibernate.dialect">org.hibernate.dialect.MySQL5Dialect</prop>
                <prop key="hibernate.show_sql">true</prop>
                <prop key="hibernate.format_sql">true</prop>
                <prop key="hibernate.hbm2ddl.auto">update</prop>
                <prop key="hibernate.connection.autocommit">true</prop>
            </props>
        </property>
    </bean>

    <bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
        <property name="defaultEncoding" value="UTF-8"/>
        <property name="maxUploadSize" value="5400000"/>
        <property name="uploadTempDir" value="fileUpload/temp"/>
    </bean>

    <!-- 定义事务管理器（声明式的事务）-->
    <bean id="transactionManager"
          class="org.springframework.orm.hibernate5.HibernateTransactionManager">
        <property name="sessionFactory" ref="sessionFactory" />
    </bean>
    <tx:annotation-driven transaction-manager="transactionManager" />

</beans>
```

上文中，配置了mysql数据库，数据库名为`demodb`，账号为`demo`，密码为`ustc2017demo`，故需要执行以下数据库命令

```sql demo.sql
-- 创建用户
CREATE USER demo IDENTIFIED BY 'ustc2017demo';
-- 赋予数据库操作权限
GRANT ALL PRIVILEGES ON demodb.* TO 'demo'@'localhost' IDENTIFIED BY 'ustc2017demo';
-- 新建数据库
DROP DATABASE IF EXISTS `demodb`;
CREATE DATABASE IF NOT EXISTS `demodb` DEFAULT CHARACTER SET utf8 COLLATE utf8_general_ci;
```

# springmvc配置文件

```xml SpringMVC.xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.3.xsd
       http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc-4.3.xsd
       http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">
    <!--对项目中的所有注解进行扫描注入-->
    <context:component-scan base-package="demo.controller" />
    <!--对controller中MVC相关注解进行注入-->
    <mvc:annotation-driven>
        <mvc:message-converters>
            <bean class="org.springframework.http.converter.StringHttpMessageConverter"/>
            <bean class="org.springframework.http.converter.json.MappingJackson2HttpMessageConverter"/>
        </mvc:message-converters>
    </mvc:annotation-driven>
    <!--文件上传目录-->
    <mvc:resources location="/fileUpload/" mapping="/fileUpload/**" />
</beans>
```

# controller层代码编写

```java DemoController.java
package demo.controller;

import java.io.File;

import javax.servlet.http.HttpServletRequest;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.DeleteMapping;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.PutMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.multipart.MultipartFile;

import demo.service.DemoService;
import demo.until.UploadUtil;
import demo.vo.DemoVo;
import demo.vo.Page;
import demo.vo.Result;

@RestController
@RequestMapping("/demo")
public class DemoController {

    @Autowired
    private DemoService demoService;

    @GetMapping("/{id}")
    public DemoVo get(@PathVariable("id") int id) {
        return demoService.findDemo(id);
    }

    @GetMapping("/search")
    public Page<DemoVo> get(@RequestParam String something,
            @RequestParam(required = false, defaultValue = "1") int page) {
        return demoService.searchDemo(page, something);
    }

    @PostMapping()
    public Result save(@RequestParam String something) {
        return demoService.addDemo(something);
    }

    @PostMapping("/img")
    public Result upload(HttpServletRequest request, @RequestParam MultipartFile file) {

        String filePath = request.getSession().getServletContext().getRealPath("/");
        filePath += "fileUpload" + File.separator;
        Result uploadResult = UploadUtil.upload(filePath, file);
        if (uploadResult.isSuccess()) {
            return demoService.addDemo(uploadResult.getResult());
        }
        return uploadResult;
    }

    @PutMapping("/{id}")
    public Result update(@PathVariable("id") int id, @RequestParam String something) {
        DemoVo demoVo = new DemoVo();
        demoVo.setDemoId(id);
        demoVo.setSomething(something);
        return demoService.modifyDemo(demoVo);
    }

    @DeleteMapping("/{id}")
    public Result delete(@PathVariable("id") int id) {
        return demoService.deleteDemo(id);
    }

}
```

# service层代码编写

以下为demo模块的接口

```java DemoService.java
package demo.service;

import demo.vo.DemoVo;
import demo.vo.Page;
import demo.vo.Result;

public interface DemoService {
	
	Result addDemo(String something);
	
	Result deleteDemo(int id);
	
	Result modifyDemo(DemoVo demoVo);
	
	DemoVo findDemo(int id);
	
	Page<DemoVo> searchDemo(int page, String s);

}
```

接口对应的实现为


```java DemoServiceImpl.java
package demo.service.impl;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import demo.dao.BaseDao;
import demo.dao.DemoDao;
import demo.model.Demo;
import demo.service.DemoService;
import demo.vo.DemoVo;
import demo.vo.Page;
import demo.vo.Result;

@Service
public class DemoServiceImpl implements DemoService {

	@Autowired
	private BaseDao<Demo> demoBaseDao;

	@Autowired
	private DemoDao demoDao;

	@Override
	public Result addDemo(String something) {
		Demo demo = new Demo();
		demo.setSomething(something);
		demoBaseDao.save(demo);
		return Result.getSuccessInstance("保存成功");
	}

	@Override
	public Result deleteDemo(int id) {
		Demo demo = demoBaseDao.load(Demo.class, id);
		if (demo == null) {
			return Result.getFailureInstance("demo不存在");
		}
		demoBaseDao.delete(demo);
		return Result.getSuccessInstance("删除成功");
	}

	@Override
	public Result modifyDemo(DemoVo demoVo) {
		Demo demo = demoBaseDao.load(Demo.class, demoVo.getDemoId());
		if (demo == null) {
			return Result.getFailureInstance("demo不存在");
		}
		demo.setSomething(demoVo.getSomething());
		demoBaseDao.update(demo);
		return Result.getSuccessInstance("保存成功");
	}

	@Override
	public DemoVo findDemo(int id) {
		DemoVo demoVo = new DemoVo();
		Demo demo = demoBaseDao.load(Demo.class, id);
		if (demo != null) {
			demoVo.setDemoId(demo.getDemoId());
			demoVo.setSomething(demo.getSomething());
		}
		return demoVo;
	}

	@Override
	public Page<DemoVo> searchDemo(int page, String s) {
		return demoDao.getSimilarDemo(page, s);
	}

}
```

# dao层实现

dao层接口定义如下

```java BaseDao.java
package demo.dao;

import org.hibernate.Session;

import java.util.List;

/**
 * 基本数据包接口.
 * Created by 丞 on 2016/9/22.
 */
public interface BaseDao<T> {

    /**
     * 获取可自动关闭的session.
     * @return session
     */
    Session getSession();

    /**
     * 必须手动关闭的session.
     * @return session
     */
    Session getNewSession();

    /**
     * flush.
     */
    void flush();

    /**
     * clear().
     */
    void clear();

    /**
     * 根据主键id查找数据库中的一条.
     * @param templateClass class name
     * @param id 主键id
     * @return 对应model类
     */
    T load(Class<T> templateClass, int id);

    /**
     * 获取某张表所有行.
     * @param templateClass T.CLASS
     * @return 列表
     */
    List<T> getAllList(Class<T> templateClass);

    /**
     * 获取总行数.
     * @param templateClass T.CLASS
     * @return 行数
     */
    Long getTotalCount(Class<T> templateClass);

    /**
     * 保存model.
     * @param bean model
     */
    void save(T bean);

    /**
     * 更新model.
     * @param bean model
     */
    void update(T bean);

    /**
     * 删除model.
     * @param bean model
     */
    void delete(T bean);

    /**
     * 根据主键id删除model.
     * @param templateClass T.CLASS
     * @param id ID
     */
    void delete(Class<T> templateClass, String id);

    /**
     * 删除多个.
     * @param templateClass T.CLASS
     * @param ids IDs
     */
    void delete(Class<T> templateClass, String[] ids);

    /**
     * 根据列名查找一行.
     * @param templateClass T.CLASS
     * @param coloum 列头
     * @param value 值
     * @return 单行
     */
    T find(Class<T> templateClass, String coloum, String value);

    /**
     * 根据列名查找一行.
     * @param templateClass T.CLASS
     * @param coloum 列头
     * @param value 值
     * @return 列表
     */
    List<T> findlist(Class<T> templateClass, String coloum, String value);

}
```

```java DemoDao.java
package demo.dao;

import demo.vo.DemoVo;
import demo.vo.Page;

public interface DemoDao {
	
	Page<DemoVo> getSimilarDemo(int page, String s);
	
}
```

对应接口实现如下

```java BaseDaoImpl.java
package demo.dao.impl;

import org.hibernate.Query;
import org.hibernate.Session;
import org.hibernate.SessionFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Repository;
import org.springframework.transaction.annotation.Transactional;

import demo.dao.BaseDao;

import java.util.List;

/**
 * 基本sql.
 * @author 丞
 *
 * @param <T> 需要进行数据操作的类
 */
@Repository
@Transactional
public class BaseDaoImpl<T> implements BaseDao<T> {

    @Autowired
    private SessionFactory sessionFactory;

    @Override
    public final Session getSession() {
        return sessionFactory.getCurrentSession();
    }

    @Override
    public final Session getNewSession() {
        return sessionFactory.openSession();
    }

    @Override
    public final void flush() {
        getSession().flush();
    }

    @Override
    public final void clear() {
        getSession().clear();
    }

    @Override
    public final T load(final Class<T> templateClass, final int id) {
        Session session = getSession();
        return session.get(templateClass, id);
    }

    @Override
    @SuppressWarnings("unchecked")
    public final List<T> getAllList(final Class<T> templateClass) {
        String hql = "from " + templateClass.getName();
        Session session = getSession();
        return session.createQuery(hql).list();
    }

    @Override
    public final Long getTotalCount(final Class<T> templateClass) {
        Session session = getSession();
        String hql = "select count(*) from " + templateClass.getName();
        Long count = (Long) session.createQuery(hql).uniqueResult();
        session.close();
        if (count != null) {
            return count;
        } else {
            return (long) 0;
        }
    }

    @Override
    public final void save(final T bean) {
        try {
            Session session = getSession();
            session.save(bean);
            session.flush();
        } catch (Exception exception) {
            exception.printStackTrace();
        }
    }

    @Override
    public final void update(final T bean) {
        Session session = getSession();
        session.update(bean);
        session.flush();
    }

    @Override
    public final void delete(final T bean) {

        Session session = getSession();
        session.delete(bean);
        session.flush();
    }

    @Override
    public final void delete(final Class<T> templateClass, final String id) {
        Session session = getSession();
        T obj = session.get(templateClass, Integer.parseInt(id));
        session.delete(obj);
        session.flush();
    }

    @Override
    public final void delete(final Class<T> templateClass, final String[] ids) {
        for (String id : ids) {
            T obj = getSession().get(templateClass, id);
            if (obj != null) {
                getSession().delete(obj);
            }
        }
        flush();
    }

    @Override
    @SuppressWarnings("unchecked")
    public final T find(final Class<T> templateClass,
                        final String column, final String value) {
        Session session = getSession();
        try {
            StringBuilder stringBuilder = new StringBuilder();
            stringBuilder.append("FROM ");
            stringBuilder.append(templateClass.getName());
            stringBuilder.append(" AS u WHERE u.");
            stringBuilder.append(column);
            stringBuilder.append(" = :value");
            String hql = stringBuilder.toString();
            Query query = session.createQuery(hql);
            query.setParameter("value", value);
            List<T> list = query.list();
            if (list.size() == 0) {
                return null;
            } else {
                return list.get(0);
            }
        } catch (Exception exception) {
            exception.printStackTrace();
            return null;
        }
    }

    @Override
    @SuppressWarnings("unchecked")
    public final List<T> findlist(final Class<T> templateClass,
                                  final String coloum, final String value) {
        Session session = getSession();
        try {
            StringBuilder stringBuilder = new StringBuilder();
            stringBuilder.append("FROM ");
            stringBuilder.append(templateClass.getName());
            stringBuilder.append(" AS u WHERE u.");
            stringBuilder.append(coloum);
            stringBuilder.append(" = ?1");
            String hql = stringBuilder.toString();
            Query query = session.createQuery(hql);
            query.setParameter(1, value);
            return query.list();
        } catch (Exception exception) {
            exception.printStackTrace();
            return null;
        }
    }

}
```

```java DemoDaoImpl.java
package demo.dao.impl;

import java.util.ArrayList;

import org.hibernate.Query;
import org.hibernate.Session;
import org.hibernate.SessionFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Repository;
import org.springframework.transaction.annotation.Transactional;

import demo.dao.DemoDao;
import demo.model.Demo;
import demo.until.FormatUtil;
import demo.vo.DemoVo;
import demo.vo.Page;

@Repository
@Transactional
public class DemoDaoImpl implements DemoDao {

    private static final int PAGE_SHOW = 10;

    @Autowired
    private SessionFactory sessionFactory;

	@Override
	public Page<DemoVo> getSimilarDemo(int page, String s) {
        Session session = sessionFactory.getCurrentSession();
        String countHql = "select count(d.demoId) from Demo as d where d.something LIKE :s";
        Long count = (Long) session.createQuery(countHql)
                .setParameter("s", "%" + s + "%")
                .uniqueResult();
        Page<DemoVo> demoVoPage = new Page<>();
        demoVoPage.setPageShow(PAGE_SHOW);
        demoVoPage.setTotalCount(count.intValue());
        demoVoPage.setNowPage(page);

        String selectHql = "from Demo as d where d.something LIKE :s";
        Query query = session.createQuery(selectHql)
                .setParameter("s", "%" + s + "%")
                .setFirstResult(demoVoPage.getStart())
                .setMaxResults(demoVoPage.getPageShow());

        ArrayList<DemoVo> demoVos = new ArrayList<>();
        for (Object o:query.list()) {
        	Demo demo = (Demo) o;
        	DemoVo demoVo = FormatUtil.change(demo);
        	demoVos.add(demoVo);
        }
        demoVoPage.setResult(demoVos);

        return demoVoPage;
	}

}
```

# OR关系映射类

```java Demo.java
package demo.model;

import java.io.Serializable;

import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;
import javax.persistence.Table;

@Entity
@Table(name = "demo")
public class Demo implements Serializable {

    private static final long serialVersionUID = 5769337720793242214L;

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(length = 32, name = "demo_id")
    private int demoId;

    @Column(length = 1024)
    private String something;

    public int getDemoId() {
        return demoId;
    }

    public void setDemoId(int demoId) {
        this.demoId = demoId;
    }

    public String getSomething() {
        return something;
    }

    public void setSomething(String something) {
        this.something = something;
    }

}
```

# vo类

vo类为对外提供json数据的包装类

```java DemoVo.java
package demo.vo;

public class DemoVo {

	private int demoId;
	private String something;

	public int getDemoId() {
		return demoId;
	}

	public void setDemoId(int demoId) {
		this.demoId = demoId;
	}

	public String getSomething() {
		return something;
	}

	public void setSomething(String something) {
		this.something = something;
	}

}
```

```java Page.java
package demo.vo;

import java.util.Collections;
import java.util.List;

/**
 * 分页vo.
 * Created by 丞 on 2016/10/23.
 */
public class Page<E> {
    private int pageShow;
    private int totalCount;
    private int nowPage;
    private List<E> result = Collections.emptyList();

    public int getPageShow() {
        return pageShow;
    }

    public void setPageShow(int pageShow) {
        this.pageShow = pageShow;
    }

    public int getTotalPage() {
        return (int) Math.ceil(totalCount * 1.0 / pageShow);
    }

    public int getTotalCount() {
        return totalCount;
    }

    public void setTotalCount(int totalCount) {
        this.totalCount = totalCount;
    }

    /**
     * 获取当前列表起始序号.
     * @return 序号
     */
    public int getStart() {
        int start = (getNowPage() - 1) * getPageShow();
        if (start < 0) {
            start = 0;
        }
        return start;
    }

    /**
     * 获取当前页码.
     * @return 页码从1开始
     */
    public int getNowPage() {
        if (nowPage <= 0) {
            nowPage = 1;
        }
        if (nowPage > getTotalPage()) {
            nowPage = getTotalPage();
        }
        return nowPage;
    }

    public void setNowPage(int nowPage) {
        this.nowPage = nowPage;
    }

    public List<E> getResult() {
        return result;
    }

    public void setResult(List<E> result) {
        this.result = result;
    }
}
```

```java Result.java
package demo.vo;

/**
 * 返回信息.
 * Created by 丞 on 2016/9/23.
 */
public class Result {

    private static final String TITLE_SUCCESS = "Success";
    private static final String TITLE_FAILURE = "Error";

    private String title;
    private String result;

    public Result(){}

    private Result(String title, String result) {
        this.title = title;
        this.result = result;
    }

    public static Result getSuccessInstance(String message) {
        return new Result(TITLE_SUCCESS, message);
    }

    public static Result getFailureInstance(String message) {
        return new Result(TITLE_FAILURE, message);
    }

    public String getTitle() {
        return title;
    }

    public void setTitle(String title) {
        this.title = title;
    }

    public String getResult() {
        return result;
    }

    public void setResult(String result) {
        this.result = result;
    }

    public boolean isSuccess() {
        return this.title.equals(TITLE_SUCCESS);
    }
}
```

# 工具类

```java FormatUtil.java
package demo.until;

import demo.model.Demo;
import demo.vo.DemoVo;

/**
 * model vo change. 
 * Created by 丞 on 2016/12/20.
 */
public class FormatUtil {

	public static DemoVo change(Demo demo) {
		DemoVo demoVo = new DemoVo();
		demoVo.setDemoId(demo.getDemoId());
		demoVo.setSomething(demo.getSomething());
		return demoVo;
	}

}
```

```java StringUtil.java
package demo.until;

import java.util.Random;

/**
 * 字符串工具类
 * Created by 丞 on 2016/9/23.
 */
public class StringUtil {

    /**
     * 判断字符串是否为空.
     * @param string 字符串
     * @return 判断结果
     */
    public static boolean isNull(final String string) {
        return string == null || string.length() == 0;
    }

    /**
     * 获取特定长度随机字符串.
     * @param length 随机串长度
     * @return 随机字符串
     */
    public static String randomString(final int length) {
        String str = "abcdefghijklmnopqrstuvwxyz";
        str += "ABCDEFGHIJKLMNOPQRSTUVWXYZ";
        str += "0123456789";
        Random random = new Random();
        StringBuilder buf = new StringBuilder();
        for (int i = 0; i < length; i++) {
            int num = random.nextInt(str.length());
            buf.append(str.charAt(num));
        }
        return buf.toString();
    }

}
```

```java UploadUtil.java
package demo.until;

import org.springframework.web.multipart.MultipartFile;

import demo.vo.Result;

import static demo.until.StringUtil.randomString;

import java.io.File;

public class UploadUtil {
    
    private static final int FILE_NAME_LENGTH = 5;

    /**
     * 保存文件到指定目录.
     * @param filePath 保存目录
     * @param file 上传的文件
     * @return 保存结果
     */
    public static Result upload(final String filePath, final MultipartFile file) {
        if (!file.isEmpty()) {
            try {
                String realName = file.getOriginalFilename();
                String fileName = randomString(FILE_NAME_LENGTH);
                fileName += System.currentTimeMillis();
                fileName += ".";
                fileName += realName.substring(realName.indexOf(".") + 1);
                file.transferTo(new File(filePath + fileName));
                return Result.getSuccessInstance("/fileUpload/" + fileName);
            } catch (Exception exception) {
                exception.printStackTrace();
                return Result.getFailureInstance("IO异常");
            }
        }
        return Result.getFailureInstance("文件为空");
    }
}
```

# 启动项目

maven构建目标为`tomcat7:run`，即可在本机8080端口上启动本工程

# 测试项目

使用chrome浏览器的应用postman来测试接口

# 打包项目

maven构建目标为`clean install`，即可在target文件夹中生成war文件，用于发布到服务器的tomcat中

# 生成文档

maven构建目标为`clean javadoc:javadoc`，即可在site文件夹中生成文档网页
