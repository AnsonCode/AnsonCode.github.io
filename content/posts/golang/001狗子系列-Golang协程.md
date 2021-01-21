---
title: "001狗子系列 Golang结构体"
date: 2021-01-20T23:43:40+08:00
draft: true
---

# go协程



线程处理

线程池

# GPM模型

goRoutine Processor Machine

M=>线程

##  协程比线程一定快吗？
- 部分情况快
- goroutine快的原因，节省了线程切换的资源消耗


##  和线程池中的任务等同？


chanel的本质 一个先入先出的队列

​```mermaid
graph TD
    A[Christmas] -->|Get money| B(Go shopping)
    B --> C{Let me think}
    C -->|One| D[Laptop]
    C -->|Two| E[iPhone]
    C -->|Three| F[fa:fa-car Car]
            
​```
 
