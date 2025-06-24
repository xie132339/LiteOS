FROM openjdk:21

ENV WORK_PATH /home/app
ENV APP_NAME LiteOS-0.0.1-SNAPSHOT.jar
ENV APP_VERSION 0.0.1
ENV JAVA_OPTS="\
-server \
-XX:+UseG1GC"
ENV TZ=Asia/Shanghai
ENV active env

VOLUME ["/home/app","/opt/data"]
VOLUME ["/usr/liteos/logs"]

ADD target/$APP_NAME app.jar

EXPOSE 8080

RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS -Djava.security.egd=file:/dev/./urandom -jar app.jar"]