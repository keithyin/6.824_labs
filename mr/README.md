## master 
流程：
原始文件split，然后给相对应的 woker 执行 mapper 【文件切分成几份？】
mapper执行完之后，会将文件路径重新传给 master。【mapper输出的文件有几个？是按照reducer的数量来的，reducer多，自然写的份数就多】
master将该信息给 reducer，然后reducer使用 rpc 从worker中读取文件，然后对这些文件执行 reduce，然后写到 hdfs上

fault tolerance:
master 会对worker 进行心跳检测，如果worker挂了，会将该worker的任务交给其它worker执行。

## worker

mapper: 处理文件，然后根据key不同，写不同的文件，写完之后，文件返回给 master管理
reducer：master将对应key的文件名给 reducer，然后reducer会通过rpc去读该文件，然后执行reduce。计算得到的结果放到hdfs上


具体流程整理：

1. 任务提交
2. master根据mapper的个数进行文件切分，然后指定 <文件part，mapper>执行，并实时监控该mapper状态
3. master通过rpc调用worker来执行对应的操作，worker操作的结果通过rpc返回给master。似乎很合理？感觉也不对，rpc调用岂不是  
4. mapper执行完毕后，会根据hash(key)%reducerNum 来拆分文件，写在本机磁盘上，路径会上报给master
5. master会告诉reducer这些磁盘路径，reducer会用prc去读这些数据，然后进行处理，处理结果放到hdfs
6. mapper个数事先指定（相当于master切分文件的个数已经定了），reducer个数事先指定。
