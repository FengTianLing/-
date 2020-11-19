~~~java
package com.java.config;

import com.alibaba.druid.pool.DruidDataSource;
import com.alibaba.druid.support.http.StatViewServlet;
import com.alibaba.druid.support.http.WebStatFilter;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.web.servlet.FilterRegistrationBean;
import org.springframework.boot.web.servlet.ServletRegistrationBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class DruidConfiguration {
    @Bean
    @ConfigurationProperties(prefix = "spring.datasource")
    public DruidDataSource druidDataSource(){
        return new DruidDataSource();
    }
    @Bean
    public ServletRegistrationBean druidServlet() {
        ServletRegistrationBean<StatViewServlet> servletRegistrationBean = new ServletRegistrationBean<>(new StatViewServlet(), "/druid/*");
        //IP������
        servletRegistrationBean.addInitParameter("allow", "");
        //IP������
        servletRegistrationBean.addInitParameter("deny", "192.168.1.100");
        //����̨�����û�
        servletRegistrationBean.addInitParameter("loginUsername", "admin");
        servletRegistrationBean.addInitParameter("loginPassword", "123456");

    //�Ƿ��ܹ��������� ����HTMLҳ���ϵġ�Reset All������
    servletRegistrationBean.addInitParameter("resetEnable", "false");
    return servletRegistrationBean;
}

@Bean
public FilterRegistrationBean filterRegistrationBean() {
    FilterRegistrationBean<WebStatFilter> filterRegistrationBean = new FilterRegistrationBean<>(new WebStatFilter());
    filterRegistrationBean.addUrlPatterns("/*");
    filterRegistrationBean.addInitParameter("exclusions", "*.js,*.gif,*.jpg,*png,*.css,*.ico,/druid/*");
    return filterRegistrationBean;
}

}
~~~


