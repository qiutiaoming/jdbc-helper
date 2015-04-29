# Introduction #
JdbcHelper is a simple library for handling common Jdbc tasks such as running queries or executing statements. Although it is a simple API, using Jdbc is doing lots of coding. (Exception handling, properly closing resources etc.) JdbcHelper tries to help the developers with a much simpler API.

# Installing JdbcHelper #
JdbcHelper is a single jar file which is less than 70K. All you need to do is put it in your classpath.  There are no external dependencies in the latest version. (0.3.1).

Before version 0.3.1, the only external dependency was the SLF4J logging library (Compiled against SLF4J api) which is available at http://www.slf4j.org. To setup logging for your application if you are using version 0.3, please refer to the SLF4J documentation. In short, you should add slf4j api jar and the corresponding jar file for your actual logging implementation to your classpath.

# Compiling JdbcHelper #
JdbcHelper is built using Apache Maven. To compile it, change to the directory where you checked out the source code (The directory where you'll find pom.xml file) and type `mvn package` on your console. The jdbc-helper-(version).jar will be built in the target directory.

# Using JdbcHelper #
Users of Spring JdbcTemplate will probably find JdbcHelper familier since they both have a similar API. Please refer to the [Examples](Examples.md) page for different examples on how to use JdbcHelper.