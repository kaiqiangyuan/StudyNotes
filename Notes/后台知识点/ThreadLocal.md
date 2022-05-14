**ThreadLocal在数据库连接中的应用**

------

常用实现方式

```java
public class C3P0Utils {
 
	private static DataSource source;//数据源
	static{
		source = new ComboPooledDataSource("mysql");//根据配置文件初始化数据源
	}
	
	/**
	 * 获取数据库连接
	 * @return
	 * @throws SQLException
	 */
	public static Connection getConnection() throws SQLException{
		return source.getConnection();
	}
}
```

ThreadLocal实现

```java
public class TransactionThreadLocal {
 
	private static ThreadLocal<Connection> tc = new ThreadLocal<>();
	/**
	 * 返回当前线程的数据库连接
	 * @return
	 * @throws SQLException
	 */
	public static Connection getConnection() throws SQLException{
		Connection connection = tc.get();
		if(connection == null){
			connection = C3P0Utils.getConnection();
			tc.set(connection);
		}
		return connection;
	}
	
	/**
	 * 开启事务
	 * @throws SQLException
	 */
	public static void startTransaction() throws SQLException{
		getConnection().setAutoCommit(false);
	}
	/**
	 * 提交事务
	 * @throws SQLException
	 */
	public static void commit() throws SQLException{
		getConnection().commit();
	}
	
	/**
	 * 事务回滚
	 * @throws SQLException
	 */
	public static void rollback() throws SQLException{
		getConnection().rollback();
	}
	
	/**
	 * 关闭数据库
	 * @throws SQLException
	 */
	public static void close() throws SQLException{
		getConnection().close();//关闭数据库
		tc.remove();//将该线程的connection对象移除
	}
}
```

getConnection()方法是获取当前线程Connection对象的一个方法。ThreadLocal类中有一个get()方法，该方法用来获取当前线程的保存的ThreadLocal中的对象，每个线程保存的是该对象的一个副本，不同线程之间不会互相影响，一个线程对该对象的修改不会影响其他线程中该对象的值。set(T value)方法用来设置当前线程的该对象的值。

使用ThreadLocal对象来保存Connection对象有一个好处，它可以保证当前线程中任何地方的Connection数据库连接都是相同的。我们知道数据库操作中事务是使用比较频繁的一个东西，一个事务中包含多个数据库操作，我们必须保证一个事务中所有操作所使用的Connection对象是相同的，通常情况下我们会选择将Connection对象作为参数传入相应的数据库操作方法中，这样做比较麻烦。

```java
public class BookDaoImp implements BookDao{
 @Override
	public void edit(Book book) throws Exception {
		Connection conn = TransactionThreadLocal.getConnection();
		String sql = "update book set name=? where id=?";
		PreparedStatement pst = conn.prepareStatement(sql);
		pst.setString(1, "name");
		pst.setString(2, id);
		pst.execute();
	}
}
```

上面的类中的edit方法使用的 Connection对象是使用TransactionThreadLocal类的getConnection()方法获取的，而不需要将Connection对象传入，这样可以使代码更加健壮。









