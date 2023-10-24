FROM maven:3.8.2-jdk-11-slim AS builder

RUN mkdir -p /opt/datagen;
COPY user_behavior.log /opt/datagen
COPY flink-sql-demo.jar /opt/datagen

WORKDIR /opt/datagen
ENV DATEGEN_HOME /opt/datagen