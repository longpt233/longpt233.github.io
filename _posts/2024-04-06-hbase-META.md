---
layout: post
title: Hbase -META-
tags:
  - hadoop
---

hbase architecture

# Overview logic architecture

```
Table                    
    Region               
        Store            ứng với một column family. bao gồm 01 memstore và >=0 storefile
            MemStore     memtable - đọc ghi trong mem - NOT a cache. khi đầy sẽ đẩy xuống thành Hfile
            StoreFile    HFile -  cấu trúc dữ liệu: SSTable
                Block    storefile bao gồm các khối. đây là đơn vị compession
          
```

thao tác: 

- với memstore: hbase.hregion.memstore.flush.size: khi đầy sẽ đẩy thành hfile
- với hfile: hfile sẽ tăng lên theo thời gian. hfile là imutable. Compaction nhằm mục đích gộp lại số lượng storefile của 1 store. tăng perform đọc. Compaction: Minor, Major. 

block cache là cache đọc. ghi đồng thời vào WALs và Memstore. 

![](../images/2024-04-08%2015-59-33.png)


# Cấu trúc bảng

- -ROOT- table is removed since HBase 0.96.0 (https://issues.apache.org/jira/browse/HBASE-3171)
- the **location** of .META table is currently stored in the Zookeeper and its name become hbase:meta. lưu trên zoo này để mục đích HA.(bthg client sẽ cache cái này tới khi k connect được nó mới gọi tới zoo lại ?)

```
Client -> ZooKeeper: Where's hbase:meta?
ZooKeeper -> Client : It's at RegionServer RS1.
```

- bảng hbase:meta này cũng như những bảng bthg (có thể split). chứa thông tin key đó ở server nào cf:info:server -> value=master:16020 . 

The hbase:meta table structure 

| Key | values |
|-|-|
|Region key of the format ([table],[region start key],[region id])|info:regioninfo (serialized HRegionInfo instance for this region)|
||info:server (server:port of the RegionServer containing this region)|
||info:serverstartcode (start-time of the RegionServer process containing this region)|

```
hbase:001:0> scan 'hbase:meta',{LIMIT =>2};
ROW                                                       COLUMN+CELL                                                                                                                                                                                                                                             
 SINHVIEN,,long.uuid column=info:regioninfo, timestamp=2024-03-28T11:49:47.317, value={ENCODED => uuid, NAME => 'SINHVIEN,,long.uuid', STARTKEY => '', ENDKEY => ''}                                                                                                              
 SINHVIEN,,long.uuid column=info:seqnumDuringOpen, timestamp=2024-03-28T11:49:47.317, value=\x00\x00\x00\x00\x00\x00\x00u                                                                                                                                                                                                             
 SINHVIEN,,long.uuid column=info:server, timestamp=2024-03-28T11:49:47.317, value=master:16020                                                                                             
                                               
```

The hbase:namespace, hbase:acl

# Split region

When a table is in the process of splitting, two other columns will be created, called info:splitA and info:splitB. These columns represent the two daughter regions. The values for these columns are also serialized HRegionInfo instances. After the region has been split, eventually this row will be deleted. [chi tiết](https://blog.cloudera.com/apache-hbase-region-splitting-and-merging/) 


```
users,,1335466383956.4a15eba  column=info:splitA,
 38d58db711e1c7693581af7f1.    timestamp=1335466889942, value=
                               {NAME =>
                               'users,,1335466889926.9fd558ed44a63f016
                               c0a99c4cf141eb5.', STARTKEY => '',
                               ENDKEY => '}7\
                               x8E\xC3\xD1\xE3\x0F\x0D\xE9\xFE'fIK\xB7\
                               xD6', ENCODED =>
                               9fd558ed44a63f016c0a99c4cf141eb5,}

 users,,1335466383956.4a15eba  column=info:splitB,
 38d58db711e1c7693581af7f1.    timestamp=1335466889942, value={NAME =>
                               'users,}7\x8E\xC3\xD1\xE3\x0F\
                               x0D\xE9\xFE'fIK\xB7\xD6,1335466889926.a3
                               c3a9162eeeb8abc0358e9e31b892e6.',
                               STARTKEY => '}7\x8E\
                               xC3\xD1\xE3\x0F\x0D\xE9\xFE'fIK\xB7\xD6'
                               , ENDKEY => '', ENCODED =>
                               a3c3a9162eeeb8abc0358
                               e9e31b892e6,}
```


# Cấu trúc znode trên zoo 



|znode | config | desciption | 
|-|-| -|
|/hbase/table | zookeeper.znode.masterTableEnableDisable |	Used by the master to track the table state during assignments (disabling/enabling states, for example).|
|/hbase/master |(zookeeper.znode.master) 	|The “active” master will register its own address in this znode at startup, making this znode the source of truth for identifying which server is the Master.|
|/hbase/rs |(zookeeper.znode.rs) |	On startup each RegionServer will create a sub-znode (e.g. /hbase/rs/m1.host) that is supposed to describe the “online” state of the RegionServer. The master monitors this znode to get the “online” RegionServer list and use that during Assignment/Balancing.|
|/hbase |(zookeeper.znode.parent) 	|The root znode that will contain all the znodes created/used by HBase|
| /hbase/hbaseid |(zookeeper.znode.clusterId) |	Initialized by the Master with the UUID that identifies the cluster. The ID is also stored on HDFS in hdfs:/<namenode>:<port>/hbase/hbase.id.|
|/hbase/root-region-server |(zookeeper.znode.rootserver) -> zookeeper.znode.metaserver (HBASE-3171) 	|that contains the location of the server hosting META.|
|/hbase/replication |(zookeeper.znode.replication) 	|Root znode that contains all HBase replication state information|

```
[zk: localhost:2181(CONNECTED) 2] get /hbase/master
�master:16000'???
[zk: localhost:2181(CONNECTED) 1] get /hbase/meta-region-server 
�master:16000????
[zk: localhost:2181(CONNECTED) 6] get /hbase/table/hbase:meta 
�master:16000O�W1�aPBUF
[zk: localhost:2181(CONNECTED) 7] get /hbase/table/SINHVIEN
�master:160009)����PBUF
```


# Ref 

[cấu trúc znode hbase](https://blog.cloudera.com/what-are-hbase-znodes/)
[ví dụ split table ](https://livebook.manning.com/concept/hbase/meta)



