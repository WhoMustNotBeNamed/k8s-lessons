FROM amazoncorretto:23-alpine3.19

WORKDIR /app
COPY muffin-wallet-server/build/libs/muffin-wallet-server-1.0.0.jar app.jar

CMD ["java", "-jar", "app.jar"]