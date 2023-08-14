---
title: "docker compose depends on : wait for postgres ready"
date: 2023-08-14T14:34+08:00
draft: false
description: "docker compose depends on : wait for postgres ready"
categories:

tags:
    - docker
---
> docker compose如何等待其他container运行后再启动，如：app等待数据库准备就绪后才能启动
```yaml
version: '3'
services:
  database:
    container_name: db
    image: postgres:12.15-alpine
    volumes:
      - erp_db_vol:/var/lib/postgresql/data
    healthcheck:
      test: [ "CMD-SHELL", "sh -c 'pg_isready -U ${DB_USERNAME} -d ${DB_NAME}'" ]
      interval: 10s
      timeout: 5s
      retries: 5
  backend:
    container_name: server
    image: server
    ports:
      - "8000:8000"
    networks:
      - erp
    depends_on:
      database:
        condition: service_healthy
```