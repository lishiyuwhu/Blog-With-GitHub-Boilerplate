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



# tnn/core/blob_manager

```c++
NetworkConfig config_;
NetStructure *net_structure_;
BlobMemoryPool *blob_memory_pool_;
AbstractDevice *device_;
BlobMap input_blobs_;
BlobMap output_blobs_;
std::shared_ptr<MemoryAssignStrategy> strategy_;
std::map<std::string, Blob *> blobs_;
std::map<Blob *, BlobMemory *> blob_memory_mapping_;

std::thread::id init_thread_id_;
MemoryModeState *memory_mode_state_;
```

负责持有相关资源, 内存池等.

init的时候会把相关layer_blob, input_blob, out_blob都分配到各自的blob map中, 但并没有分配内存. 调用AllocateBlobMemory()的时候才会真正的根据device的情况分配内存. 同时也会给各个blob添加引用计数, 方便空间重用



# tnn/core/context

context接口类.  不同的device自己实现不同的方法来管理context. 



# tnn/core/default_network

AbstractNetwork的实现类. 网络前向相关的操作就在这里.



DefaultNetwork::optimize_mtx_ 是一个静态变量实现的互斥锁, 防止在算子融合的时候, 改变network structure等相关资源



DefaultNetwork::InitLayers 中, 会判断下网络是否含有量化层, 后续设置UpdateBlobPrecision时候有不同的实现. 

如果layer中的LayerParam是量化的, 且对应的desc是非量化的, 则会将其转换一个一个int8Blob.



forward分为两种带Async和不带Async的, 区别在于一个加了"同步锁"一个是直接跑的



# tnn/core/instance

Instance是真正对外暴露执行推理操作的单元, 内部保存了network相关资源和配置

