```yaml
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://101.201.150.92:3306/jdbc?useUnicode=true&characterEncoding=utf-8&serverTimezone=UTC
    username: root
    password: 101399jjy..
    type: com.alibaba.druid.pool.DruidDataSource
    initial-size: 1
    min-idle: 1
    max-active: 20
    max-wait: 60000
    time-between-eviction-runs-millis: 60000
    min-evictable-idle-time-millis: 300000
    validation-query: SELECT 'X'
    test-while-idle: true
    test-on-borrow: false
    test-on-return: false
    pool-prepared-statements: false
    max-pool-prepared-statement-per-connection-size: 20
```

