# master
curl http://172.17.19.15:60010/jmx>./master.json

# regionserver
curl http://172.17.19.2:60030/jmx?description=true > rs1.json

# 主要指标

| 监控指标 | 范围 | 指标含义 |
| :--------: | :--------: | :--------------------------------: |
| OpenFileDescriptorCount | Regionserver本机 | 当前机器打开文件数 |
| FreePhysicalMemorySize | Regionserver本机 | 空虚物理内存大小 |
| AvailableProcessors | Regionserver本机 | 可用cpu个数 |
| Region前缀--storeCount | 单个region | Store个数 |
| Region前缀--storeFileCount | 单个region | Storefile个数 |
| Region前缀--memStoreSize | 单个region | Memstore大小 |
| Region前缀--storeFileSize | 单个region | Storefile大小 |
| Region前缀--compactionsCompletedCount | 单个region | 合并完成次数 |
| Region前缀--numBytesCompactedCount | 单个region | 合并文件总大小 |
| Region前缀-- numFilesCompactedCount | 单个region | 合并完成文件个数 |
| totalRequestCount | Regionserver | 总请求数 |
| readRequestCount | Regionserver | 读请求数 |
| writeRequestCount | Regionserver | 写请求数 |
| compactedCellsCount | Regionserver | 合并cell个数 |
| majorCompactedCellsCount | Regionserver | 大合并cell个数 |
| flushedCellsSize | Regionserver | flush到磁盘的大小 |
| blockedRequestCount | Regionserver | 因memstore大于阈值而引发flush的次数 |
| splitRequestCount | Regionserver | region分裂请求次数 |
| splitSuccessCounnt | Regionserver | region分裂成功次数 |
| slowGetCount | Regionserver | 请求完成时间超过1000ms的次数 |
| numOpenConnections | Regionserver | 该regionserver打开的连接数 |
| numActiveHandler | Regionserver | rpc handler数 |
| receivedBytes | Regionserver | 收到数据量 |
| sentBytes | Regionserver | 发出数据量 |
| HeapMemoryUsage—>>used | Regionserver | 堆内存使用量 |
| SyncTime_mean | Regionserver | WAL写hdfs的平均时间 |
| regionCount | Regionserver | Regionserver管理region数量 |
| memStoreSize | Regionserver | Regionserver管理的总memstoresize |
| storeFileSize | Regionserver | 该Regionserver管理的storefile大小 |
| staticIndexSize | Regionserver | 该regionserver所管理的表索引大小 |
| storeFileCount | Regionserver | 该regionserver所管理的storefile个数 |
| hlogFileSize | Regionserver | WAL文件大小 |
| hlogFileCount | Regionserver | WAL文件个数 |
| storeCount | Regionserver | 该regionserver所管理的store个数 |
| Name: java.lang:type=MemoryPool,name=Par Eden Space <br> CollectionUsage—>>used | Regionserver | Eden区使用空间大小 |
| Name: java.lang:type=MemoryPool,name=CMS Old Gen <br> CollectionUsage—>>used | Regionserver | 老年代内存大小 |
| Name: java.lang:type=MemoryPool,name=Par Survivor Space <br> CollectionUsageà> used | Regionserver | Survivor内存大小 |
| GcTimeMillis | Regionserver | GC总时间 |
| GcTimeMillisParNew | Regionserver | ParNew GC时间 |
| GcCount | Regionserver | GC总次数 |
| GcCountConcurrentMarkSweep | Regionserver | ConcurrentMarkSweep总次数 |
| GcTimeMillisConcurrentMarkSweep | Regionserver | ConcurrentMarkSweep GC时间 |
| ThreadsBlocked | Regionserver | Block线程数 |
| ThreadsWaiting | Regionserver | 等待线程数 |

