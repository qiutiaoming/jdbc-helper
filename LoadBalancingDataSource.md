# Introduction #
When you have a set of database servers which contain the same data, sometimes you may need to load balance these servers by forwarding the queries to a different server each time. For this type of a goal, JdbcHelper provides a custom DataSource implementation named LoadBalancingDataSource. LoadBalancingDataSource tries to use a different DataSource by randomly selecting one. It can also detect failed data sources and stop using them.

```
      MysqlConnectionPoolDataSource ds = new MysqlConnectionPoolDataSource();
      ds.setServerName("server1");
      ds.setUser("root");
      ds.setPassword("");
      ds.setDatabaseName("test");

      LoadBalancingDataSource lbDs = new LoadBalancingDataSource();
      lbDs.addDataSource("first", ds, 10);

      ds = new MysqlConnectionPoolDataSource();
      ds.setServerName("server2");
      ds.setUser("root");
      ds.setPassword("");
      ds.setDatabaseName("test");
      lbDs.addDataSource("second", ds, 10);

      JdbcHelper jdbc = new JdbcHelper(lbDs);
```

In the example above, the LoadBalancingDataSource creates a connection pool for each added MysqlConnectionPoolDataSource. Each call made to the jdbc-helper api is forwarded to a randomly selected pool.

# Eviction of idle connections and failed data sources #
The LoadBalancingDataSource has an internal thread for its maintenance purposes. To run this thread you should call the `startEvictionThread()` method on the LoadBalancingDataSource instance. To stop it you can call the `stopEvictionThread()` method.

If you don't want to start a custom thread for this purpose since you already have a custom thread pool, then you can get a Runnable for the maintenance job using the `getMaintenanceJob()` method. The maintenance job should be run every few minutes.