# Introduction #

JdbcHelper ships a slightly modified version of MiniConnectionPoolManager created by Christian d'Heureuse. http://www.source-code.biz/snippets/java/8.htm.

MiniConnectionPoolManager class is in JdbcHelper source tree and named as jdbchelper.ConnectionPool. You can use this class to make a pool of database connections to increase your application performance without a dependency to an external connection pooling library such as C3P0 or Commons DBCP

# Example Usage #

```
MysqlConnectionPoolDataSource ds = new MysqlConnectionPoolDataSource();
ds.setServerName("localhost");
ds.setUser("root");
ds.setPassword("");
ds.setDatabaseName("test");

ConnectionPool pool = new ConnectionPool(ds, 50); // Create a pool of maximum size 50
PooledDataSource pooledDs = new PooledDataSource(pool);
JdbcHelper jdbc = new JdbcHelper(pooledDs);
```