# 🔴 Master Slave 동기화 문제

## 🟠 Master와 Slave 구조

트래픽이 어느정도 있다면 DB를 구성할때 Master와 Slave 구조로 나누게 된다.

### 🟢 Replication Lag

이때 발생할 수 있는 문제로는 Master에서 데이터를 insert, update, delete와 같은 데이터 추가 수정이 일어날때 Replication 구조에서 데이터가 복제되기 전에 조회가 일어나버리는 것이다. 만약 데이터가 insert하는 경우라면 slave에서 조회했을때 null 값이 발생할 수 있어 시스템 문제로 발생할 수 있다.

### 🟢 Master 강제 조회

가장 간단한 방법으로 DataSource에서 Master를 강제로 조회해서 가져오도록 처리할 수 있다.

```java
// DynamicRoutingDataSource Override
public class DynamicRoutingDataSource extends AbstractRoutingDataSource {

    @Override
    protected Object determineCurrentLookupKey() {
        return isCurrentTransactionReadOnly() ? SLAVE : MASTER;
    }
}

// Config
@RequiredArgsConstructor
@Configuration
@EnableTransactionManagement
public class RoutingDataSourceConfig {

    private final Environment env;

    @ConfigurationProperties(prefix = "spring.datasource.hikari.master")
    @Bean
    public DataSource masterDataSource() {
        return DataSourceBuilder.create().type(HikariDataSource.class).build();
    }

    @ConfigurationProperties(prefix = "spring.datasource.hikari.slave")
    @Bean
    public DataSource slaveDataSource() {
        return DataSourceBuilder.create().type(HikariDataSource.class).build();
    }

    @DependsOn({"masterDataSource", "slaveDataSource"})
    @Bean
    public DataSource routingDataSource(
        @Qualifier("masterDataSource") DataSource master,
        @Qualifier("slaveDataSource") DataSource slave) {
        DynamicRoutingDataSource dynamicRoutingDataSource = new DynamicRoutingDataSource();

        Map<Object, Object> dataSourceMap = new HashMap<>();

        dataSourceMap.put(MASTER, master);
        dataSourceMap.put(SLAVE, slave);

        dynamicRoutingDataSource.setTargetDataSources(dataSourceMap);
        dynamicRoutingDataSource.setDefaultTargetDataSource(master);

        return dynamicRoutingDataSource;
    }

    @DependsOn({"routingDataSource"})
    @Bean
    public DataSource dataSource(DataSource routingDataSource) {
        return new LazyConnectionDataSourceProxy(routingDataSource);
    }
}
```

@Transaction 값이 readOnly가 true이,false에 따라 Slave와 Master를 구분하여 조회하도록 자동으로 처리할 수 있다. 만약 select에서 문제가 발생하는 경우가 있다면 readOnly값을 false로 설정하여 master를 강제로 바라보게 할 수 있다.