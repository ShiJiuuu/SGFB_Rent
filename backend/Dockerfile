# ===== 构建阶段 =====
FROM maven:3.8-openjdk-11-slim AS builder

WORKDIR /app
COPY pom.xml .
# 预先下载依赖（利用 Docker 缓存层）
RUN mvn dependency:go-offline -B

COPY src ./src
RUN mvn clean package -DskipTests -B

# ===== 运行阶段 =====
FROM openjdk:11-jre-slim

WORKDIR /app

# 创建非 root 用户运行应用
RUN groupadd -r sgfb && useradd -r -g sgfb sgfb

COPY --from=builder /app/target/*.jar app.jar

# JVM 内存限制：最大 512M，初始 256M
ENV JAVA_OPTS="-Xmx512m -Xms256m"

EXPOSE 8080

USER sgfb

ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS -Dspring.profiles.active=docker -jar app.jar"]
