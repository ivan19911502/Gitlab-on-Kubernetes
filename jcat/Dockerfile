# Stage 1: Сборка фронтенда
FROM node:16-alpine AS frontend-build

WORKDIR /app/frontend

COPY frontend/package*.json ./

RUN npm cache clean --force

RUN npm install --legacy-peer-deps

COPY frontend/public ./public
COPY frontend/src ./src

# Копируем другие необходимые файлы для сборки фронтенда
COPY frontend/*.json ./

# Собираем фронтенд
RUN npm run build

# Stage 2: Сборка бэкенда
FROM maven:3.8-openjdk-17-slim AS backend-build

WORKDIR /app

# Копируем корневой pom.xml
COPY pom.xml .

# Копируем модуль frontend (для структуры Maven)
COPY frontend frontend/

# Копируем модуль backend
COPY backend backend/

# Копируем собранный фронтенд в нужную директорию
COPY --from=frontend-build /app/frontend/build /app/backend/src/main/resources/static

# Собираем только бэкенд
RUN mvn clean package -pl backend -am

# Stage 3: Финальный образ
FROM eclipse-temurin:17-jre-alpine

WORKDIR /app

COPY --from=backend-build /app/backend/target/*.jar app.jar

ENV SPRING_DATASOURCE_URL=jdbc:postgresql://postgres:5432/postgres?ApplicationName=account%20service
ENV SPRING_DATASOURCE_USERNAME=postgres
ENV SPRING_DATASOURCE_PASSWORD=postgres

EXPOSE 8080

ENTRYPOINT ["java", "-jar", "app.jar"]
