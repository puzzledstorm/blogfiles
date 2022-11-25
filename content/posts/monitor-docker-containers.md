---
title: "监控机器上的docker容器"
date: 2022-11-25T15:44:06+08:00
description: 用grafana展示容器内存使用情况
draft: false
hideToc: false
enableToc: true
enableTocContent: false
author: puzzled
authorEmoji: 🐶
tocFolding: false
tocPosition: inner
tocLevels: ["h2", "h3", "h4"]
tags:
- docker
- grafana 
- cadvisor 
- prometheus 
- node-exporter
series:
-
categories:
-
image: images/feature1/5.gif
---

## 监控环境搭建

需要以下组件：

- docker
- prometheus
- node-exporter
- cadvisor
- grafana

## docker-compose.yml

```yaml
version: "3.3"
networks:
    monitor:
        driver: bridge
services:
  node-exporter:
    image: prom/node-exporter:latest
    container_name: "node-exporter0"
    ports:
      - "9130:9100"
    restart: always
    networks:
      - monitor
  
  cadvisor:
    image: google/cadvisor:latest
    container_name: "cadvisor0"
    restart: always
    volumes:
      - /:/rootfs:ro
      - /var/run:/var/run:rw
      - /sys:/sys:ro
      - /var/lib/docker/:/var/lib/docker:ro
    ports:
      - "8081:8080"
    networks:
      - monitor

  prometheus:
    image: prom/prometheus:latest
    container_name: "prometheus0"
    restart: always
    ports:
      - "9097:9090"
    volumes:
      - "./prometheus.yml:/etc/prometheus/prometheus.yml"
      - "./prometheus_data:/prometheus"
    networks:
      - monitor

  grafana:
    image: grafana/grafana
    container_name: "grafana0"
    ports:
      - "3005:3000"
    restart: always
    volumes:
      - "./grafana_data:/var/lib/grafana"
      - "./grafana_log:/var/log/grafana"
      - "./grafana_data/crypto_data:/crypto_data" 
    networks:
      - monitor
```

## prometheus.yml

```yaml
global:
  scrape_interval:     15s # 默认抓取周期
  external_labels:
    monitor: 'codelab-monitor'  
scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['172.16.139.24:9097', '172.16.139.24:8081'] 
      
  - job_name: 'node-exporter' #服务的名称
    scrape_interval: 5s
    metrics_path: /metrics  #获取指标的url
    static_configs:
      - targets: ['172.16.139.24:9130'] # 这个为监听指定服务服务的ip和port，需要修改为自己的ip,不能使用localhost和127.0.0.1
 
  - job_name: "cadvisor"
    static_configs:
    - targets:
      - '172.16.139.24:8081'
```

## 启动

```bash
docker-compose up -d
```

结果如下：

![1](/images/posts/1.png)

## 检查

浏览器访问

### node-exporter

http://172.16.139.24:9130/metrics

### cadvisor

http://172.16.139.24:8081/containers/

### prometheus

http://172.16.139.24:9097/targets

![2](/images/posts/2.png)

### grafana

http://172.16.139.24:3005/login   

用户名/密码 admin/admin

操作：

1. 添加数据源

   Configuration下

   Add data source

   url填上：http://172.16.139.24:9097

   Save & test

2. 导入面板

   id： 10619
   
   [Docker Container & Host Metrics](https://grafana.com/grafana/dashboards/10619-docker-host-container-overview/)

3.  结果

   ![3](/images/posts/3.png)

## 参考

```
https://grafana.com/grafana/dashboards/893-main/
https://www.jianshu.com/p/cb50ffe0b6b0
https://www.jianshu.com/p/bd64a114aab0
https://www.cnblogs.com/augus007/articles/9225431.html
https://juejin.cn/post/6969764486701924383
https://blog.csdn.net/An1090239782/article/details/102999721
https://grafana.com/dashboards?search=docker
https://www.gbmb.org/mib-to-mb
```

