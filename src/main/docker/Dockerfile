FROM eclipse-temurin:21-jre-alpine

ENV WORK_PATH /home/app
ENV APP_NAME @project.build.finalName@.@project.packaging@
ENV APP_VERSION @project.version@
ENV JAVA_OPTS="\
-server \
-XX:+UseG1GC"
ENV TZ=Asia/Shanghai
ENV active env

# 安装时区数据（Alpine 需要单独安装）
RUN apk add --no-cache tzdata && \
    ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && \
    echo $TZ > /etc/timezone && \
    # 清理缓存减小镜像层大小
    rm -rf /var/cache/apk/* \

VOLUME ["/home/app","/opt/data"]
VOLUME ["/usr/liteos/logs"]

ADD target/$APP_NAME app.jar

EXPOSE 8080

RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS -Djava.security.egd=file:/dev/./urandom -jar app.jar"]