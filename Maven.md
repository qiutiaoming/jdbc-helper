# Introduction #

JdbcHelper is currently not included in the central maven repository. If you want to use it in your maven projects, you need to install it to your local maven repository.

# Installing JdbcHelper to your local maven repository #

First you need to checkout or export the version 0.3.1 source code:
```
svn checkout http://jdbc-helper.googlecode.com/svn/tags/0.3.1/ jdbc-helper-0.3.1
```

Use export if you don't want .svn directories created:
```
svn export http://jdbc-helper.googlecode.com/svn/tags/0.3.1/ jdbc-helper-0.3.1
```

Then change to the jdbc-helper-0.3.1 directory and type:
```
mvn clean source:jar javadoc:jar install
```

This will install the version 0.3.1 to your local maven repository along with the sources and the javadocs.

# Depending on JdbcHelper #
You can add a dependency to the library by adding the following xml to the `<dependencies>` section of your pom.xml file:

```
<dependency>
   <groupId>com.googlecode.jdbc-helper</groupId>
   <artifactId>jdbc-helper</artifactId>
   <version>0.3.1</version>
</dependency>
```