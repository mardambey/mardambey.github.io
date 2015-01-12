---
layout: post
title: On Tomcat 6.x, Hibernate 3.x, MySQL, and JNDI
date: '2008-01-06T00:12:00-05:00'
tags:
- Config
- hibernate
- apache
- tomcat
- jndi
tumblr_url: http://www.hisham.cc/post/30203524981/on-tomcat-6-x-hibernate-3-x-mysql-and-jndi
---
After lots of googling and documentation reading, I got a setup of mine to work properly with Tomcat 6.x, Hibernate 3.x, MySQL, and JNDI datasources. For what its worth, most of the resources on the web regarding setting this up with the versions I specified are worthless. To save myself the future hassle, and the lot of you who might be having a hard time setting this up, here are the configuration files and directives required.

yourapp/web/META-INF/context.xml:


  <Context>
  <Resource name="jdbc/my_app" 
            global="jdbc/my_app"
            auth="Container"
            type="javax.sql.DataSource" 
            username="user"
            password="XXXX"
            driverClassName="com.mysql.jdbc.Driver"
url="jdbc:mysql://localhost:3306/my_db_name?autoReconnect=true"
            maxActive="8" 
            maxIdle="4"/>
</Context>



yourapp/web/WEB-INF/web.xml:

...
<resource-ref>
  <description>DB Connection</description>
  <res-ref-name>jdbc/my_app</res-ref-name>
  <res-type>javax.sql.DataSource</res-type>
  <res-auth>Container</res-auth>
</resource-ref>
...


hibernate.cfg.xml:

...
<hibernate-configuration>
  <session-factory>
    <!-- Database connection settings -->
    <property name="connection.datasource">java:comp/env/jdbc/my_app</property>
    <property name="dialect">org.hibernate.dialect.MySQLDialect</property>
    <property name="current_session_context_class">thread</property>
    <mappings here>    
  </session-factory>  
</hibernate-configuration>
...


You can get your session factory as usual:

sessionFactory = new Configuration().configure().buildSessionFactory();

or whichever way you prefer.
