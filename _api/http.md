---
title: HTTP接口
layout: page
show_sidebar: false
menubar: api_menu
---

## 功能目标

Pegasus为MetaServer和ReplicaServer提供了HTTP接口，用于查看集群相关信息，查询服务器信息等。

## 接口介绍

接口共分为服务器信息查询、表管理和节点管理三类，其中服务器信息查询类中的指令同时支持对ReplicaServer和MetaServer的访问，其它指令支持MetaServer。

当访问备用MetaServer的表管理和节点管理类接口时，由于相关信息保存在主MetaServer中，会自动重定向至主MetaServer对应的接口。

所有接口均返回JSON格式。

### `/version`

**功能：** 获取应用的版本

**示例：**`127.0.0.1:34801/version`

**返回：**

```json
{
    "GitCommit": "cf428dc0ef995b961809df860e783417de7757fc",
    "Version": "1.12.SNAPSHOT"
}
```

### `/recentStartTime`

**功能：** 获取进程启动时间

**示例：**`127.0.0.1:34801/recentStartTime`

**返回：**

```json
{
    "RecentStartTime": "2019-08-14 00:22:00"
}
```

### `/pprof/profile`

**功能：** 获取server的cpu采样

**示例1：**`127.0.0.1:34801/pprof/profile?seconds=10`

**返回：** profile文件

**示例2：** 推荐使用google/pprof Web UI

`pprof --http=:8080 http://127.0.0.1:34801/pprof/heap`

### `/pprof/symbol`

**功能：** 获取symbol信息

**示例1：**`127.0.0.1:34801/pprof/symbol`

**返回：** 非post请求，返回symbol个数
```
num_symbols: 46225
```

**示例2：**`curl 127.0.0.1:34801/pprof/symbol -X POST -d "123455"`

**返回：** post请求，返回输入地址对应的symbol
```
0x00123455      boost::asio::detail::keyword_tss_ptr
```

### `/pprof/heap`

**功能：** 获取server的内存采样

**示例1：**`127.0.0.1:34801/pprof/heap`

**返回：** heap profile文本

**示例2：** 推荐使用google/pprof Web UI

`pprof --http=:8080 http://127.0.0.1:34801/pprof/heap`

**注意：** 需在被采样server所在机器设置环境变量`TCMALLOC_SAMPLE_PARAMETER`

### `/pprof/cmdline`

**功能：** 获取/proc/self/cmdline内容

**示例：**`127.0.0.1:34801/pprof/cmdline`

**返回：** 
```json
/somepath/pegasus/onebox/replica1/pegasus_server
config.ini
-app_list
replica
```
### `/pprof/growth`

**功能：** 获取growth profile

**示例：**`127.0.0.1:34801/pprof/growth`

**返回：** growth profile文本

### `/meta/app?name=<app_name>`

**功能：** 获取某个表的基本信息。

**参数：**

- name： 待查的表名
- detail：是否打印详细信息

**示例：**`127.0.0.1:34601/meta/app?name=stat&detail`

**返回：**

```json
{
    "general": {
        "app_id": "1",
        "app_name": "stat",
        "max_replica_count": "3",
        "partition_count": "4"
    },
    "healthy": {
        "fully_healthy_partition_count": "0",
        "read_unhealthy_partition_count": "0",
        "unhealthy_partition_count": "4",
        "write_unhealthy_partition_count": "0"
    },
    "nodes": {
        "127.0.0.1:34801": {
            "node": "127.0.0.1:34801",
            "primary": "2",
            "secondary": "2",
            "total": "4"
        },
        "127.0.0.1:34802": {
            "node": "127.0.0.1:34802",
            "primary": "2",
            "secondary": "2",
            "total": "4"
        },
        "total": {
            "node": "total",
            "primary": "4",
            "secondary": "4",
            "total": "8"
        }
    },
    "replicas": {
        "0": {
            "ballot": "2",
            "pidx": "0",
            "primary": "127.0.0.1:34801",
            "replica_count": "2/3",
            "secondaries": "[127.0.0.1:34802]"
        },
        "1": {
            "ballot": "2",
            "pidx": "1",
            "primary": "127.0.0.1:34802",
            "replica_count": "2/3",
            "secondaries": "[127.0.0.1:34801]"
        },
        "2": {
            "ballot": "2",
            "pidx": "2",
            "primary": "127.0.0.1:34801",
            "replica_count": "2/3",
            "secondaries": "[127.0.0.1:34802]"
        },
        "3": {
            "ballot": "2",
            "pidx": "3",
            "primary": "127.0.0.1:34802",
            "replica_count": "2/3",
            "secondaries": "[127.0.0.1:34801]"
        }
    }
}
```

### `/meta/apps`

**功能：** 获取所有表的列表。

**参数：**

- detail：是否打印详细信息
  
**示例：**`127.0.0.1:34601/meta/apps?detail`

**返回：**
  
```json
{
    "general_info": {
        "1": {
            "app_id": "1",
            "app_name": "stat",
            "app_type": "pegasus",
            "create_time": "2019-08-15 12:34:15.000",
            "drop_expire": "-",
            "drop_time": "-",
            "envs_count": "0",
            "is_stateful": "true",
            "partition_count": "4",
            "replica_count": "3",
            "status": "AVAILABLE"
        },
        "2": {
            "app_id": "2",
            "app_name": "temp",
            "app_type": "pegasus",
            "create_time": "2019-08-15 12:34:15.000",
            "drop_expire": "-",
            "drop_time": "-",
            "envs_count": "0",
            "is_stateful": "true",
            "partition_count": "8",
            "replica_count": "3",
            "status": "AVAILABLE"
        }
    },
    "healthy_info": {
        "1": {
            "app_id": "1",
            "app_name": "stat",
            "fully_healthy": "0",
            "partition_count": "4",
            "read_unhealthy": "0",
            "unhealthy": "4",
            "write_unhealthy": "4"
        },
        "2": {
            "app_id": "2",
            "app_name": "temp",
            "fully_healthy": "0",
            "partition_count": "8",
            "read_unhealthy": "0",
            "unhealthy": "8",
            "write_unhealthy": "8"
        }
    },
    "summary": {
        "fully_healthy_app_count": "0",
        "read_unhealthy_app_count": "0",
        "total_app_count": "2",
        "unhealthy_app_count": "2",
        "write_unhealthy_app_count": "2"
    }
}
```

### `/meta/cluster`

**功能：** 获取集群基本信息。

**示例：**`127.0.0.1:34601/meta/cluster`

**返回：**

```json
{
    "balance_operation_count": "move_pri=0,copy_pri=0,copy_sec=0,total=0",
    "meta_function_level": "steady",
    "meta_servers": "127.0.0.1:34601,127.0.0.1:34602,127.0.0.1:34603",
    "primary_meta_server": "127.0.0.1:34601",
    "primary_replica_count_stddev": "0.00",
    "total_replica_count_stddev": "0.00",
    "zookeeper_hosts": "127.0.0.1:22181",
    "zookeeper_root": "/pegasus/onebox/127.0.0.1"
}
```

### `/meta/nodes`

**功能：** 获取replica节点列表，以IP地址表示各个节点，并输出基本信息。

**参数：**

- detail：是否打印详细信息

**示例：**`127.0.0.1:34601/meta/nodes?detail`

**返回：**

```json
{
    "details": {
        "127.0.0.1:34801": {
            "address": "127.0.0.1:34801",
            "primary_count": "6",
            "replica_count": "12",
            "secondary_count": "6",
            "status": "ALIVE"
        },
        "127.0.0.1:34802": {
            "address": "127.0.0.1:34802",
            "primary_count": "6",
            "replica_count": "12",
            "secondary_count": "6",
            "status": "ALIVE"
        }
    },
    "summary": {
        "alive_node_count": "2",
        "total_node_count": "2",
        "unalive_node_count": "0"
    }
}
```

### `/meta/app_envs?name=<app_name>`

**功能：** 获取某个表的所有环境变量。

**参数：**

- name： 待查的表名
  
**示例：**`127.0.0.1:34601/meta/app_envs?name=temp`

**返回：**
  
```json
{
    "replica.enable_slow_query_log": "true",
    "replica.slow_query_threshold": "20"
}
```

### `/perfCounter?name=<perf_counter_name>`

**功能：** 获取某个perf counter的详细信息。如果perf counter名字中含有特殊字符时，需要先对其进行uri编码。

**参数：**

- name： 待查的perf counter名字
  
**示例：**`127.0.0.1:34101/perfCounter?name=collector*app.pegasus*app.stat.read_qps%23_all_`

**返回：**
  
```json
{
    "name": "collector*app.pegasus*app.stat.read_qps#_all_",
    "value": "0.00",
    "type": "NUMBER",
    "description": "statistic the read_qps of app _all_"
}
```
