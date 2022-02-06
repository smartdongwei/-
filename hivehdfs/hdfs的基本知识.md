## 1: HDFS的三个核心组件是什么

**1: NameNode** 集群的核心, 是整个文件系统的管理节点. 维护着
    a) 文件系统的文件目录结构和元数据信息
    b) 文件与数据块列表的对应关系
**2 :DataNode** 存放具体数据块的节点, 主要负责数据的读写, 定期向NameNode发送心跳
**3 :SecondaryNameNode** 辅助节点, 同步NameNode中的元数据信息, 辅助NameNode对fsimage和editsLog进行合并。



## 2: fsimage和editlogs是做什么用的?

fsimage文件存储的是Hadoop的元数据文件, 如果namenode发生故障, 最近的fsimage文件会被载入到内存中, 用来重构元数据的最近状态, 再从相关点开始向前执行edit logs文件中记录的每个事务.

文件系统客户端执行写操作时, 这些事务会首先记录到日志文件中.

在namenode运行期间, 客户端对hdfs的写操作都保存到edit文件中, 久而久之就会造成edit文件变得很大, 这对namenode的运行没有影响, 但是如果namenode重启, 它会将fsimage中的内容映射到内存中, 然后再一条一条执行edit文件中的操作, 所以日志文件太大会导致重启速度很慢. 所以在namenode运行的时候就要将edit logs和fsimage定期合并.
