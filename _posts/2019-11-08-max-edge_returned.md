---
layout: post
title: Max edge returned from one vertex
---

# 加 gflags 参数控制全局大点

## 待确认
gflags 的值设为多少? 1000 是不是太小?

## 需求

interface/storage.thrift

## 提交
### 修改列表
max_edge_returned_per_vertex

```shell
src/storage/QueryBaseProcessor.cpp
src/storage/QueryBaseProcessor.h
src/storage/QueryBaseProcessor.inl
src/storage/QueryBoundProcessor.cpp
src/storage/test/QueryBoundTest.cpp
```

## 调用顺序
storage/client/StorageClient.cpp 
StorageClient::getNeighbors
future_getBound

storage/StorageServiceHandler.cpp
auto* processor = QueryBoundProcessor::instance(kvstore_,
                                                    schemaMan_,
                                                    &getBoundQpsStat_,
                                                    getThreadManager());
                                                    
## 测试
src/storage/test/_build/query_bound_test
### UT
test/QueryBoundTest.cpp 
中新增 FlagsBoundVertexTest 
(基本 copy 自 OutBoundSimpleTest, 只是设置一个 FLAGS_max_sub_vertices,
并验证, 最后checkResponse时, )
                                                    
## 沟通 四王 2019-11-05

1. storage/QueryBaseProcessor.inl 
    asyncProcessBucket

2. storage/QueryBoundProcessor.cpp
    processVertex
    这个是放最终结果的地方
    vertices_.emplace_back(std::move(vResp));
    
1. 验证可以通过设置小一点的 bucket , 小一点的 gflags, 看看效果
    
### 调用堆栈


```c
13. 
12. QueryBaseProcessor<REQ, RESP>::asyncProcessBucket(Bucket bucket)
11. QueryBaseProcessor<REQ, RESP>::process
...
1. QueryBoundProcessor::instance
0. StorageServiceHandler::future_getBound
```


```c
using OneVertexResp = std::tuple<PartitionID, VertexID, kvstore::ResultCode>;
```