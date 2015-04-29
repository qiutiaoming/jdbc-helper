Inspired by _Spring Jdbctemplate_ and _Commons Dbutils_ projects, _JdbcHelper_ is a very small library for helping the developers code common jdbc operations. _JdbcHelper_ is very lightweight. It is only ~70K and it has no external dependencies.

A usage example:
```
DataSource dataSource = new SimpleDataSource("com.mysql.jdbc.Driver",
         "jdbc:mysql://localhost/test?autoReconnect=true",
         "root", null);

JdbcHelper jdbc = new JdbcHelper(dataSource);

jdbc.execute("INSERT INTO `jdbctest` (id, name) VALUES(?, ?)", 10, "test");

Test t = jdbc.queryForObject("select * from jdbctest where id = ?", new BeanCreator<Test>() {
         @Override
         public Test createBean(ResultSet rs) throws SQLException {
            Test t = new Test();
            t.id = rs.getInt(1);
            t.name = rs.getString(2);
            return t;
         }
      }, 10);
```

  * [Getting started with JdbcHelper](GettingStarted.md)
  * [Examples](Examples.md)
  * [Built-in connection pooling](ConnectionPooling.md)
  * [Load balancing read-only data-sources](LoadBalancingDataSource.md)
  * [Using JdbcHelper with Maven](Maven.md)
  * [Change Log](ChangeLog.md)