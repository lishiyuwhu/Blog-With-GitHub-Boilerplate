---
layout: post
title: TNN源码阅读笔记_1
date: 2021-02-04 17:30
status: publish
author: Plams
categories: 
  - 端侧
tags:
  - TNN
excerpt: API与tnn/core/abstract_device
---



TNN相关核心源码阅读.



# API接口

```shell
include
└── tnn
    ├── core
    │   ├── blob.h
    │   ├── common.h
    │   ├── instance.h
    │   ├── macro.h
    │   ├── mat.h
    │   ├── status.h
    │   └── tnn.h
    ├── utils
    │   ├── bfp16_utils.h
    │   ├── blob_converter.h
    │   ├── cpu_utils.h
    │   ├── data_type_utils.h
    │   ├── dims_vector_utils.h
    │   ├── half.hpp
    │   ├── half_utils.h
    │   ├── mat_utils.h
    │   └── string_utils.h
    └── version.h
```



tnn对外的接口, 包括核心推理相关以及辅助的半精度/矩阵/字符串函数. 略过



TNN的大版本0.0.2, 第一次大更新后的版本

> ```
> static char *branch_name_tnn = "master";
> static char *commit_date_tnn = "2020-10-20";
> static char *commit_hash_tnn = "0dc11ef";
> ```



# tnn/core/abstract_device



包含抽象设备接口以及对应的注册方法



```c++
std::map<DeviceType, std::shared_ptr<AbstractDevice>>& GetGlobalDeviceMap() {
    static std::once_flag once;
    static std::shared_ptr<std::map<DeviceType, std::shared_ptr<AbstractDevice>>> device_map;
    std::call_once(once, []() { device_map.reset(new std::map<DeviceType, std::shared_ptr<AbstractDevice>>); });
    return *device_map;
}
```

GetGlobalDeviceMap中使用了std::call_once来保证只初始化一次device_map,  同时使用了静态局部变量device_map来管理注册的device, 有点类似于一个单例工厂的模板注册形式.

```c++
// @brief TypeDeviceRegister contruct register device
template <typename T>
class TypeDeviceRegister {
public:
    explicit TypeDeviceRegister(DeviceType type) {
        auto &device_map = GetGlobalDeviceMap();
        if (device_map.find(type) == device_map.end()) {
            device_map[type] = std::shared_ptr<T>(new T(type));
        }
    }
};
```

是一个注册类, 在runtime的时候添加对应的设备. 





# tnn/core/abstract_layer_acc

整体结构类似于bastract_device.

其中Status AbstractLayerAcc::ResolveBlobDataFormat(Blob *blob) 根据不同的backend device 来设置各blob的data_format, 例如对于ARM:

```c++
std::vector<DataFormat> ArmLayerAcc::SupportDataFormat(DataType data_type, int dims_size) {
    std::vector<DataFormat> support_list;
    if (dims_size == 4) {
        if (data_type == DATA_TYPE_FLOAT || data_type == DATA_TYPE_BFP16)
            support_list.push_back(DATA_FORMAT_NC4HW4);
        else if (data_type == DATA_TYPE_INT8)
            support_list.push_back(DATA_FORMAT_NHWC4);
        else if (data_type == DATA_TYPE_HALF) {
            support_list.push_back(DATA_FORMAT_NC8HW8);
        }
    }
    return support_list;
}
```

就以这个顺序来设定data_format. (可能其他backend的support format可以有多种, 所以这里扔回来了一个vec)



# tnn/core/abstract_network

一个标准的 产品注册模板类(NetworkImplFactoryRegister)+单例工厂模板类(NetworkImplManager).



