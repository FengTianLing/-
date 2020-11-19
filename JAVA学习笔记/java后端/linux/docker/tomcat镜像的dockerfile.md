```shell
FROM centos
MAINTAINER JJK<23424@QQ.COM>
COPY readme.txt /usr/local/readme.txt
ADD apache-tomcat-9.0.35.tar.gz /usr/local/
ADD openjdk-11+28_linux-x64_bin.tar.gz /usr/local/
RUN yum -y install vim
ENV MYPATH /usr/local
WORKDIR $MYPATH
ENV JAVA_HOME /usr/local/jdk-11
ENV CLASSPATH $JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
ENV CATALINA_HOME /usr/local/apache-tomcat-9.0.35
ENV CATALINA_BASH /usr/local/apache-tomcat-9.0.35
ENV PATH $PATH:$JAVA_HOME/bin:$CATALINA_HOME/lib:$CATALINA_HOME/bin
EXPOSE 8080
CMD /usr/local/apache-tomcat-9.0.35/bin/startup.sh && tail -F /usr/local/apache-tomcat-9.0.35/bin/logs/catalina.out
```

