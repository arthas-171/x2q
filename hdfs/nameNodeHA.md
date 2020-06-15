# hadoop 命名空间的维护机制 
### [go back](/hdfs.md)      
### [go home](../README.md)     
##hadoop 1.x secondarynamenode合并edits 和 fsimage 流程
  
![合并日志](/static/img/c91504c7f4e344750da4f9d03e6dc897edf.jpg)  
##如何触发 secondaryNameNode合并edits 和 fsimage
+ fs.checkpoint.period:3600s 默认一小时触发一次(core-site.xml)
+ fs.checkpoint.size:64MB 或者 edits 大小达到 64MB(core-site.xml)
##hadoop 2.x HA 主备同步namespace流程(Journal 模式 )
![HA图解](/static/img/b025a133a1e1fdf71210fe5cc60f9bbde74.jpg)

 active namenode 每次有文件有变化会写入本地的edits log并且会向 journal node的edits也写入一份, 而standy namenode 会监听journal node上edits文件的变化,如果有变化将会同步变化到自己本地edits,journal是一组相互独立的进程,当多数journal响应active namenode成功的时候,就认为写入成功了,当 active namenode 挂掉的时候 ,通过zk standy会感知到这一变化,然后再确保同步了全部 journal的edits之后,会把自己状态提升到active  
    
参考: https://blog.csdn.net/u012736748/article/details/79534019

#### 联系邮箱 xxx_xxx@aliyun.com   