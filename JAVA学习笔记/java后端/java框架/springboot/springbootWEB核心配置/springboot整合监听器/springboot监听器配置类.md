```java
public class MyListener implements ServletContextListener {
    @Override
    public void contextInitialized(ServletContextEvent sce) {
        System.out.println("servlet容器已经初始化啦");
    }

    @Override
    public void contextDestroyed(ServletContextEvent sce) {
        System.out.println("servlet容器已经被销毁");
    }
}
```

