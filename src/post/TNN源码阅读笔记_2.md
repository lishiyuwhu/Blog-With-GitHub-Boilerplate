---
layout: post
title: TNN源码阅读笔记_2
date: 2021-02-04 21:37
status: publish
author: Plams
categories: 
  - 端侧
tags:
  - TNN
excerpt: tnn/core/blob相关
---



# tnn/core/blob

数据存储的相关接口.   blob中分为两块 BlobDesc保存着 设备/精度/格式/维度/别名 这些信息, 真正的数据存储在BlobHandle中的指针中



涉及到内存相关的 allocate/set/free 的操作, 都是用 auto device = GetDevice(desc_.device_type); 拿到对应的设备, 再调用设备各自的方法实现



# tnn/core/blob_int8

blob_int8是blob的子类, 中间加上了int8 layer的resources保存, 