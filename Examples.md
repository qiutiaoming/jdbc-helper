# Introduction #
JdbcHelper has a simple API which consists of only a few number of classes and interfaces. The main class that you will be using is the jdbchelper.JdbcHelper class. This class is well documented using javadocs, so checking that documentation for each method of the class is a good idea.

# Creating a JdbcHelper Instance #

JdbcHelper class has a single constructor which accepts a javax.sql.DataSource implementation. A DataSource instance can be created in several ways.
  * It can be obtained from your application server by making a JNDI lookup or by asking the appserver to inject it in one of your classes
  * You may create an instance of your Database vendors implementations such as `com.mysql.jdbc.jdbc2.optional.MysqlDataSource` or `oracle.jdbc.pool.OracleDataSource`
  * You can use `jdbchelper.SimpleDataSource` class
  * You write your own implementation :)

An example of a JNDI lookup for an Oracle DataSource in an application server environment is as follows:

```
javax.naming.InitialContext ic               = new javax.naming.InitialContext();
oracle.jdbc.pool.OracleDataSource dataSource = (oracle.jdbc.pool.OracleDataSource)ic.lookup("jdbc/pool/OracleDS");
```

Once you have an instance of DataSource, than you can create a JdbcHelper:
```
JdbcHelper jdbc = new JdbcHelper(dataSource);
```

JdbcHelper class holds a reference to the dataSource implementation and manages getting a connection from the datasource or closing it when needed for you.

JdbcHelper class is thread safe. That means you can share the same JdbcHelper instance with many different threads.

# Running Queries #
You can use JdbcHelper to run queries with a single line of code. You can select simple values, create Java objects from resultsets, create list of objects etc. In the examples below we will use a simple java class named User.

```
public class User {
   int userId;
   String userName;
   String email;

   public static BeanCreator<User> beanCreator = new BeanCreator<User>() {
       public User createBean(ResultSet rs) throws SQLException {
           User u = new User();
           u.userId = rs.getInt("user_id");
           u.userName = rs.getString("user_name");
           u.email = rs.getString("email");
           return u;
       }
   };

   public static StatementMapper<User> statementMapper = new StatementMapper<User>() {
       public void mapStatement(PreparedStatement stmt, User u) throws SQLException {
          stmt.setInt(1, u.userId);
          stmt.setString(2, u.userName);
          stmt.setString(3, u.email);
       }
   };
}
```

## Querying a simple Object ##
```
User user = jdbc.queryForObject("select * from users where user_id = ?", User.beanCreator, 100);
```

## Querying a list of Objects ##
```
List<User> users = jdbc.queryForList("select * from users", User.beanCreator);
```

## Querying with a custom ResultSetHandler ##
```
// Process each user record with a ResultSetHandler which has a fetch size of 100
jdbc.query("select * from users", new ResultSetHandler(100) {
    public void processRow(ResultSet rs) throws SQLException {
         // do something with the resultset
    }
});
```

Alternatively you can use a ResultSetBeanHandler
```
jdbc.query("select * from users", new ResultsetBeanHandler<User>(User.beanCreator) {
    public void processBean(User u) {
         // do something with the User object
    }
});
```

## Querying simple values ##

There are many methods in JdbcHelper for selecting single rows from the database.

### Query for string ###
```
String userName = jdbc.queryForString("select user_name from users where user_id = ?", 100);
```
### Query for int ###
```
int userId = jdbc.queryForInt("select user_id from users where user_name = ?", test);
```
### Query for tuple ###
```
Tuple<Integer, String> t = jdbc.queryForTuple("select user_id, user_name from users where email = ?", new IntegerBeanCreator(1), new StringBeanCreator(2), "test@test.com");

int userId = t.getFirst();
String userName = t.getSecond();
```

## Custom iteration over ResultSet ##
```
QueryResult result = jdbc.query("select * from users");
result.setFetchSize(100);
while(result.next()) {
   int userId = result.getInt("user_id");
   String userName = result.getString("user_name");
}
result.close();
```

# Executing Statements #
## Executing a simple statement ##
```
int rows = jdbc.execute("delete from users where user_id < ?", 1000);
```

## Inserting a mapped java object ##
```
User user = new User();
user.userId = 100;
user.userName = "erdinc";
user.email = "test@example.com";
jdbc.execute("insert into users (user_id, user_name, email) values (?, ?, ?)", user, User.statementMapper);
```

## Batch Statements ##
### Using an anonymous BatchFeeder ###
```
List<User> users = getUsers();
final Iterator<User> iter = users.iterator();
jdbc.executeBatch("insert into users (user_id, user_name, email) values (?, ?, ?)", new BatchFeeder() {
    public boolean hasNext() {
       return iter.hasNext();
    }

    public boolean feedStatement(PreparedStatement stmt) throws SQLException {
         User u = iter.next();
         stmt.setInt(1, u.userId);
         stmt.setString(2, u.userName);
         stmt.setEmail(3, u.email);
         return true;
    }
});
```

The example code above inserts all the elements in the users list using a batch insert. Although it is a good example of writing a BatchFeeder, it is still lots of coding for this type of a work. The example below has the same exact bahaviour with much fewer lines of code:

```
List<User> users = getUsers();
jdbc.executeBatch("insert into users (user_id, user_name, email) values (?, ?, ?)", new MappingBatchFeeder<User>(users.iterator(), User.beanCreator));
```

# Using the same Connection on consecutive API calls #
By default JdbcHelper closes the connection that it uses after each call to its API. (This generally returns the connection to the connection pool to be reused.) Sometimes you may need to use the same physical connection to the database in consecutive queries or statements that are run. This is especially needed in running transactions.
There are two methods for holding and releasing a reference to the connection object in JdbcHelper which are `holdConnection()` and `releaseConnection()` methods.

If you are not using a connection pool, opening and closing connections may take a long time and using the same connection in consecutive db operations becomes a must.

Consider the following example:
```
try {
   jdbc.holdConnection();
   jdbc.query("...");
   jdbc.execute("...");
} finally {
   jdbc.releaseConnection();
}
```

In the example above, JdbcHelper binds the Connection instance that it pulled from the data source to the thread that is currently using that JdbcHelper instance. Untill the user calls the releaseConnection method, jdbchelper keeps the connection open and uses it for all the db operations. Holding a connection does not make the operations transactional. By default all the db calls are automatically commited.

Holding the connection is also needed if you want to get db generated primary key values from the database such as the auto\_increment key values from a MySQL database. There is a custom method for this is JdbcHelper:

```
try {
   jdbc.holdConnection();
   jdbc.execute("insert into users (user_name, email) values (?, ?)", "erdinc", "test@example.com");
   int userId = (int) jdbc.getLastInsertId();
} finally {
   jdbc.releaseConnection();
}
```

To query the auto\_increment value from the mysql database, you have to use the same connection in which you inserted the value to the database.
# Transactions #
Transaction behavior in JdbcHelper is very similar to holding the connections in the previous section with one difference. Beginning a transaction set the auto commit flag on the connection to false.
```
jdbc.beginTransaction();
try {
    int balance = jdbc.queryForInt("select balance from accounts where accountId = ? for update", 3242);
    jdbc.execute("update accounts set balance = ? where accountId = ?", balance + 100, 3242);
    jdbc.commitTransaction();
} catch(JdbcException e) {
   jdbc.rollbackTransaction();
}
```