# write a dockefile for current java project 
# Stage 1: Build the project using Maven
FROM maven:3.8.4-openjdk-11 AS build
WORKDIR /vhr
COPY . .
RUN echo '<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0" \
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" \
    xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd"> \
    <mirrors> \
        <mirror> \
            <id>aliyunmaven</id> \
            <mirrorOf>*</mirrorOf> \
            <name>阿里云公共仓库</name> \
            <url>https://maven.aliyun.com/repository/public</url> \
        </mirror> \
    </mirrors> \
</settings>' > /usr/share/maven/conf/settings.xml
# Build the project, resolve all dependencies and cache them locally 


RUN cd vhrserver  && mvn clean package -DskipTests

 



# Stage 2: Create a lightweight image to run the application
FROM openjdk:11-jre-slim 
WORKDIR /app


RUN echo > /etc/apt/sources.list && \
    echo "deb http://mirrors.aliyun.com/debian bullseye main contrib non-free" > /etc/apt/sources.list && \
    echo "deb-src http://mirrors.aliyun.com/debian bullseye main contrib non-free" >> /etc/apt/sources.list && \
    echo "deb http://mirrors.aliyun.com/debian-security bullseye-security main contrib non-free" >> /etc/apt/sources.list && \
    echo "deb-src http://mirrors.aliyun.com/debian-security bullseye-security main contrib non-free" >> /etc/apt/sources.list && \
    echo "deb http://mirrors.aliyun.com/debian bullseye-updates main contrib non-free" >> /etc/apt/sources.list && \
    echo "deb-src http://mirrors.aliyun.com/debian bullseye-updates main contrib non-free" >> /etc/apt/sources.list && \
    echo "deb http://mirrors.aliyun.com/debian bullseye-backports main contrib non-free" >> /etc/apt/sources.list && \
    echo "deb-src http://mirrors.aliyun.com/debian bullseye-backports main contrib non-free" >> /etc/apt/sources.list

# Install font libraries
RUN apt-get update && \
    apt-get install -y --no-install-recommends libfreetype6 && \
    apt-get install -y fontconfig fonts-dejavu && \
    rm -rf /var/lib/apt/lists/*


COPY --from=build /vhr/vhrserver/vhr-web/target/*.jar app.jar


# Database Configuration
ENV DB_USERNAME="" \
 DB_PASSWORD="" \
 DB_HOSTNAME="" \
 RABBITMQ_USERNAME="" \
 RABBITMQ_PASSWORD="" \
 RABBITMQ_HOST="" \
 REDIS_HOST="" \
 REDIS_DATABASE=0 \
 REDIS_PORT=6379 \
 REDIS_PASSWORD="" 

EXPOSE 8081
CMD ["java", "-jar", "app.jar"]

