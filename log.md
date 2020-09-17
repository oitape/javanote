- spring4.x和spring5.x不一样

- 4.x,按照以下顺序查找实例化
```java
    private static final String[] classesToDiscover = {
            "org.apache.commons.logging.impl.Log4JLogger",
            "org.apache.commons.logging.impl.Jdk14Logger",
            "org.apache.commons.logging.impl.Jdk13LumberjackLogger",
            "org.apache.commons.logging.impl.SimpleLog"
    }
```
        - 默认优先使用log4j
        - 然后jul log
        
        - 可以使用sl4j，通过绑定器进行动态绑定不同的log实现，参考sl4j官网

- 5.x 的log

```java

	private static final String LOG4J_SPI = "org.apache.logging.log4j.spi.ExtendedLogger";
	private static final String LOG4J_SLF4J_PROVIDER = "org.apache.logging.slf4j.SLF4JProvider";
	private static final String SLF4J_SPI = "org.slf4j.spi.LocationAwareLogger";
	private static final String SLF4J_API = "org.slf4j.Logger";
	private static final LogApi logApi;
	static {
		if (isPresent(LOG4J_SPI)) {
			if (isPresent(LOG4J_SLF4J_PROVIDER) && isPresent(SLF4J_SPI)) {
				// log4j-to-slf4j bridge -> we'll rather go with the SLF4J SPI;
				// however, we still prefer Log4j over the plain SLF4J API since
				// the latter does not have location awareness support.
				logApi = LogApi.SLF4J_LAL;
			}
			else {
				// Use Log4j 2.x directly, including location awareness support
				logApi = LogApi.LOG4J;
			}
		}
		else if (isPresent(SLF4J_SPI)) {
			// Full SLF4J SPI including location awareness support
			logApi = LogApi.SLF4J_LAL;
		}
		else if (isPresent(SLF4J_API)) {
			// Minimal SLF4J API without location awareness support
			logApi = LogApi.SLF4J;
		}
		else {
			// java.util.logging as default
			logApi = LogApi.JUL;
		}
	}

public static Log createLog(String name) {
	switch (logApi) { // logApi = jcl, 所以走的default，生成了JUL log
		case LOG4J:
			return Log4jAdapter.createLog(name);
		case SLF4J_LAL:
			return Slf4jAdapter.createLocationAwareLog(name);
		case SLF4J:
			return Slf4jAdapter.createLog(name);
		default:
			return JavaUtilAdapter.createLog(name);
	}
}

```


- mybaits自定义log

```java
@Configuration
@ComponentScan("top.oitm")
@MapperScan("top.oitm.dao")
public class AppConfig {
    @Bean
    public DataSource dataSource() {
        DriverManagerDataSource driverManagerDataSource = new DriverManagerDataSource();
        driverManagerDataSource.setDriverClassName("com.mysql.jdbc.Driver");
        driverManagerDataSource.setUrl("jdbc:mysql://172.16.158.139:3306/oitm?useUnicode=true&characterEncoding=utf8&serverTimezone=GMT%2B8&useSSL=false");
        driverManagerDataSource.setUsername("root");
        driverManagerDataSource.setPassword("root");
        return driverManagerDataSource;
    }


    @Bean
    public SqlSessionFactoryBean sqlSessionFactoryBean() {
        SqlSessionFactoryBean sqlSessionFactoryBean = new SqlSessionFactoryBean();
        org.apache.ibatis.session.Configuration configuration = new org.apache.ibatis.session.Configuration();
        configuration.setLogImpl(Log4jImpl.class);
        sqlSessionFactoryBean.setConfiguration(configuration);
        sqlSessionFactoryBean.setDataSource(dataSource());
        return sqlSessionFactoryBean;
    }

}
```


        