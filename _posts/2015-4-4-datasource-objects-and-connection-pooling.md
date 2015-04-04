---
layout: post
title: DataSource objects and Connection pooling
published: true
---

Applications access a data source by using a connection, and a DataSource object can be thought of as a factory for connections to the particular data source that the DataSource instance represents. In a basic DataSource implementation, a call to the getConnection method returns a connection object that is a physical connection to the data source.
Connection pooling lets you reuse an existing connection instead of creating a new connection to the database. This is highly efficient in terms of memory allocation and speed of the request to the database.
A DataSource object may be registered with a JNDI naming service. If so, an application can use the JNDI API to access that DataSource object, which can then be used to connect to the data source it represents.
Pooling options for production environment.
1.	Apache Commons DBCP/Tomcat JDBC Connection Pool
2.	c3p0
3.	BoneCP
Container-managed pool VS Application-managed pool
The only difference between having container-managed pool and application-managed pool is that in first case you have an ability to monitor the workload on the pool using the standard interfaces (e.g. JBoss console). Then, administrator of the application server manages the decision about increasing the pool size, if necessary. Admin may also switch applications to another DB server (e.g. planned migration from MySQL to Oracle). The disadvantage is that you need slightly more efforts to setup JNDI test data source for your unit tests.
And in the second case, you have to package either DBCP or c3p0 plus the JDBC driver together with you application. In this case it is not so easy to collect the statistics about all pools for all application running in Tomcat. Also migration to newer JDBC driver (MySQL 4 to MySQL 5) cannot be done for all applications at once. And connection properties are wired to your application, even if you use a .property file (so changing that needs reassembling and redeployment of the project).

http://vigilbose.blogspot.in/2009/03/apache-commons-dbcp-and-tomcat-jdbc.html
http://tomcat.apache.org/tomcat-7.0-doc/jdbc-pool.html
