title: 【解决多数据源动态切换事务问题】代码实战结合mybatis-plus
date: 2019-08-13 21:10:54
tags: [mybatis]
categories: [综合]
---
### 介绍

参考：http://www.yanglajiao.com/article/gaoshili001/79378902

<!--more-->

原理请参考另外一篇文章：解决多数据源动态切换事务问题】Spring Mybatis 事务管理器源码

网上好多的springboot的事务都是瞎扯，根本不起作用，后来通过各种渠道查证，springboot的声明式事务需要重写Transaction，那么我们来解决多数据源动态切换事务的问题

核心方法：

```java
private void openMainConnection() throws SQLException { 
   this.mainConnection = DataSourceUtils.getConnection(this.dataSource)
}
```

### 直接上代码

CiweiSqlSessionFactory

```java
/**
 * 配置sqlSessionFactory
 */
@Configuration
@Slf4j
public class CiweiSqlSessionFactory {

    @Autowired
    DataSourceConfiguration.DynamicDataSource dataSource;

    @Bean
    public SqlSessionFactory sqlSessionFactory() {
        log.info("--------------------  sqlSessionFactory init ---------------------");
        try {
//            SqlSessionFactoryBean sessionFactoryBean = new SqlSessionFactoryBean();
            MybatisSqlSessionFactoryBean sessionFactoryB= new MybatisSqlSessionFactoryBean();
            sessionFactorttoryBean.setDataSource(dataSource);
            sessionFactoryBean.setTransactionFactory(new MultiDataSourceTransactionFactory());
            // 读取配置
            sessionFactoryBean.setTypeAliasesPackage("com.example.apollodynamicdatasource.entity");

            //设置mapper.xml文件所在位置
            Resource[] resources = new PathMatchingResourcePatternResolver().getResources("classpath*:/mapper/*.xml");
            sessionFactoryBean.setMapperLocations(resources);
            return sessionFactoryBean.getObject();
        } catch (IOException e) {
            log.error("mybatis resolver mapper*xml is error",e);
            return null;
        } catch (Exception e) {
            log.error("mybatis sqlSessionFactoryBean create error",e);
            return null;
        }
    }

}
```

MultiDataSourceTransaction

```java
/**
 * 多数据源切换的支持事务
 */
public class MultiDataSourceTransaction implements Transaction {

	private static final Log LOGGER = LogFactory.getLog(MultiDataSourceTransaction.class);

	private final DataSource dataSource;

	private Connection mainConnection;

	private String mainDatabaseIdentification;

	private ConcurrentMap<String, Connection> otherConnectionMap;

	private boolean isConnectionTransactional;

	private boolean autoCommit;

	public MultiDataSourceTransaction(DataSource dataSource) {
		notNull(dataSource, "No DataSource specified");
		this.dataSource = dataSource;
		otherConnectionMap = new ConcurrentHashMap<>();
		mainDatabaseIdentification = DynamicDataSourceContextHolder.getDataSourceKey();
	}

	/**
	 * {@inheritDoc}
	 */
	@Override
	public Connection getConnection() throws SQLException {
		String databaseIdentification = DynamicDataSourceContextHolder.getDataSourceKey();
		if (databaseIdentification.equals(mainDatabaseIdentification)) {
			if (mainConnection != null) return mainConnection;
			else {
				openMainConnection();
				mainDatabaseIdentification = databaseIdentification;
				return mainConnection;
			}
		} else {
			if (!otherConnectionMap.containsKey(databaseIdentification)) {
				try {
					Connection conn = dataSource.getConnection();
					otherConnectionMap.put(databaseIdentification, conn);
				} catch (SQLException ex) {
					throw new CannotGetJdbcCoot get JDBC Connection", ex);
				}
			}
			retu	}
			r	}
			}
		}
			return otherConnectionMap.get(databaseIdentification);
		}

	}


	private void openMainConnection() throws SQLException {
		this.mainConnection = DataSourceUtils.getConnection(this.dataSource);
		this.autoCommit = this.mainConnection.getAutoCommit();
//		this.isConnectionTransactional = DataSourceUtils.isConnectionTransactional(this.mainConnection, this.dataSource);
		this.isConnectionTransactional = false;

		if (LOGGER.isDebugEnabled()) {
			LOGGER.debug(
					"JDBC Connection ["
							+ this.mainConnection
							+ "] will"
							+ (this.isConnectionTransactional ? " " : " not ")
							+ "be managed by Spring");
		}
	}

	/**
	 * {@inheritDoc}
	 */
	@Override
	public void commit() throws SQLException {
		if (this.mainConnection != null && !this.isConnectionTransactional && !this.autoCommit) {
			if (LOGGER.isDebugEnabled()) {
				LOGGER.debug("Committing JDBC Connection [" + this.mainConnection + "]");
			}
			this.mainConnection.commit();
			for (Connection connection : otherConnectionMap.values()) {
				connection.commit();
			}
		}
	}

	/**
	 * {@inheritDoc}
	 */
	@Override
	public void rollback() throws SQLException {
		if (this.mainConnection != null && !this.isConnectionTransactional && !this.autoCommit) {
			if (LOGGER.isDebugEnabled()) {
				LOGGER.debug("Rolling back JDBC Connection [" + this.mainConnection + "]");
			}
			this.mainConnection.rollback();
			for (Connection connection : otherConnectionMap.values()) {
				connection.rollback();
			}
		}
	}

	/**
	 * {@inheritDoc}
	 */
	@Override
	public void close() throws SQLException {
		DataSourceUtils.releaseConnection(this.mainConnection, this.dataSource);
		for (Connection connection : otherConnectionMap.values()) {
			DataSourceUtils.releaseConnection(connection, this.dataSource);
		}
	}

	@Override
	public Integer getTimeout() throws SQLException {
		return null;
	}

}
```

MultiDataSourceTransactionFactory

```java
/**
 * 支持Service内多数据源切换的Factory
 */
public class MultiDataSourceTransactionFactory extends SpringManagedTransactionFactory {

    @Override
    public Transaction newTransaction(DataSource dataSource, TransactionIsolationLevel level, boolean autoCommit) {
        return new MultiDataSourceTransaction(dataSource);
    }

}
```

最后在代码中引入

```java
@EnableTransactionManagement
@Transactional(propagation = Propagation.REQUIRED, rollbackFor = Throwable.class)
```

需要注意的是：

在spring的文档中说道，spring声明式事务管理默认对非检查型异常和运行时异常进行事务回滚，而对检查型异常（try catch）则不进行回滚操作：

故而如果异常被try｛｝catch｛｝了，事务就不回滚了，如果想让事务回滚必须再往外抛try｛｝catch｛throw Exception｝

### 总结

代码必须关闭自动提交 关闭自动提交会根据代码手动控制提交 回滚的话也是代码中控制

```java
dataSource.setAutoCommit(false);
```

demo：https://github.com/ciweigg2/apollo-dynamic-datasource