FROM eclipse-temurin:11-jdk

WORKDIR /app
COPY . .

RUN ./gradlew clean build --no-daemon -x test

EXPOSE 8080

ENTRYPOINT ["java", "-jar", "build/libs/spring-boot-FiiPractic-1.0.jar"]
