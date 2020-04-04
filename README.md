# [Look up data from Datebase]
## Goals of this project
* Look up data from DB using Spring, MariaDB, Mybatis
### Development environment
* JDK 1.8u_241
* apache-tomcat-9.0.31
* Spring 5.2.5 RELEASE
* MariaDB 10.4.12
* MySQL Workbench 8.0.19 (SQL Developer)
* Mybatis 3.4.4
* Eclipse IDE version 2019-12(4.14.0)

### Before starting
* Before starting development, change the version of Spring, JDK, Maven-compiler at the **[pom.xml]** file. And, change the source and target about Maven-complier.
```xml
<properties>
		<java-version>1.8</java-version>
		<org.springframework-version>5.2.5.RELEASE</org.springframework-version>
		<org.aspectj-version>1.6.10</org.aspectj-version>
		<org.slf4j-version>1.6.6</org.slf4j-version>
</properties>
```
```xml
<build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.7.0</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                    <compilerArgument>-Xlint:all</compilerArgument>
                    <showWarnings>true</showWarnings>
                    <showDeprecation>true</showDeprecation>
                </configuration>
            </plugin> 
        </plugins>
</build>
```
* For setting non-web part, change contents of the root-context.xml file 
```xml
<bean id="dataSource" 
        class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <!-- For checking log, I added 'log4jdbc' on the property  -->
        <property name="driverClassName"
            value="net.sf.log4jdbc.sql.jdbcapi.DriverSpy"></property>
        <property name="url"
            value="jdbc:log4jdbc:mariadb://127.0.0.1:3306/theater" />
        <property name="username" value="root" />
        <property name="password" value="password" />
</bean>
 
 
<bean id="sqlSessionFactory"
        class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource"></property>
        <property name="configLocation"
            value="classpath:/mybatis/mybatis-config.xml"></property>
        <property name="mapperLocations"
            value="classpath*:/mybatis/sql/*.xml"></property>
</bean>
 
<!--sqlSession is a scope for managing transactions, processing threads and managing DB connection. -->   
<bean id="sqlSession"
        class="org.mybatis.spring.SqlSessionTemplate">
        <constructor-arg name="sqlSessionFactory"
            ref="sqlSessionFactory"></constructor-arg>
</bean>
 
<!-- <mybatis-spring:scan base-package="com.devfun.dao" /> -->
<context:component-scan
        base-package="com.devfun.dao"></context:component-scan>
<context:component-scan
        base-package="com.devfun.service"></context:component-scan>
```
### Classes organization
* MovieDAO.java
	+ It is a interface made up of a function contains List\<MoiveVO\>.
	```java
	public interface MovieDAO {   
    	public List<MovieVO> selectMovie() throws Exception;
	}
	```
* MovieDAOImpl.java
	+ Excute SQL query registered by SqlSession. And return List\<MoiveVO\>.
	```java
	public List<MovieVO> selectMovie() throws Exception {
		return sqlSession.selectList(Namespace+".selectMovie");
    }
	```
	+ After writing **MovieDAOImpl.java**, register dao in **root-context.xml** file.
	```xml
	<context:component-scan
        	base-package="com.devfun.dao"></context:component-scan>
	```
* MovieService.java
	+ You can use this interface when you process DB and write some business logic
	+ It is also a interface made up of a function contains List\<MoiveVO\>.
	```java
	public interface MovieService {   
    	public List<MovieVO> selectMovie() throws Exception;
	}
	```
* MovieServiceImpl.java
	+ After writing **MovieServiceImpl.java**, register dao in **root-context.xml** file.
	```xml
	<context:component-scan
        	base-package="com.devfun.service"></context:component-scan>
	```
* HomeController.java
	+ 'value' property of RequestMapping is a path of url to invoke.
	```java
	@RequestMapping(value = "/", method = RequestMethod.GET)
	```
	+ By calling selectMovie function from 'service', you can get contents. And It is sent to 'model' object (it is Model class object). By using the return statement, you can send contents stored in 'model'.
	```java
	public String home(Locale locale, Model model) throws Exception{
 
        logger.info("home");
        
        List<MovieVO> movieList = service.selectMovie();
        
        model.addAttribute("movieList", movieList);
 
        return "home";
    }
	```
	+ '**return "home";**' means sending 'model' stored contents to the view called **home.jsp**. If you want to set this calling information, you shuold change **servlet-context.xml** file like below. 
	(servlet-context.xml is located in "src/main/webapp/WEB-INF/spring/appServlet/servlet-context.xml".)
	```xml
	 <beans:bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <beans:property name="prefix" value="/WEB-INF/views/" />
        <beans:property name="suffix" value=".jsp" />
    </beans:bean>
	```

### JSP file
* home.jsp
	+ By using \<c:forEach\>, you can look up data from DB. For using  \<c:forEach\>, you should use JSTL.
