
# Round 1 介绍
方案介绍：
        UV统计：
             利用正则表达式将数据拆分，每一行保留用户的ip地址、访问时间、存放到set集合中进行去重
             map中 key值为时间 value为ip进行输出
             reduce 中处理去重的ip
        Top10 show统计
             利用正则表达式将数据拆分，需要统计每一个ip和每个ip的访问数量
             map中 key为访问数量 value为ip进行输出
             reduce 进行排序 输出前十
        次日留存统计：
             通过MapReduce得到前一天访问的不同ip
             在通过MapReduce得到当天访问的不同ip
             通过以上得到的数据做一个MapReduce 统计每个ip出现的次数 
             ip次数不为1就是留下的用户
             
环境搭建：
安装虚拟机，将网卡改成桥接网卡
安装openssh-client与lrzsz
配置Java环境 vim /home/hadoop/.bashrc 
最后一行添加 export JAVA_HOME=/home/hadoop/jdk1.6.0_45/
export CLASSPATH=.:$CLASSPATH:$JAVA_HOME/lib:$JRE_HOME/lib
export PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin
输入source ~/.bashrc
测试Java配置是否成功 Java-version
安装Hadoop 配置相应的文件（修改master文件、slaves文件、core-site.xml、mapred-site.xml、hdfs-site.xml、hadoop-env.sh、sudo vi /etc/hosts）
  
  以下是详细的配置： 
      a) hadoop-env.sh 、yarn-env.sh，mapred_env.sh
       这二个文件主要是修改JAVA_HOME后的目录，改成实际本机jdk所在目录位置 
          vi etc/hadoop/hadoop-env.sh （及 vi etc/hadoop/yarn-env.sh）
            在这一行上添加： 
        export JAVA_HOME=/home/hadoop/jdk1.8(按具体情况修改) 
      b) core-site.xml 参考下面的内容修改： 
         <?xml version="1.0" encoding="UTF-8"?> 
         <?xml-stylesheet type="text/xsl" href="configuration.xsl"?> 
           <configuration> 
             <property> 
               <name>fs.defaultFS</name> 
               <value>hdfs://master:9000</value> 
            </property> 
                <property> 
                <name>hadoop.tmp.dir</name> 
                <value>/home/hadoop/tmp</value> 
            </property> 
           </configuration>    
        注：/home/hadoop/tmp 目录如不存在，则先mkdir手动创建      
      c) hdfs-site.xml 
        <?xml version="1.0" encoding="UTF-8"?> 
        <?xml-stylesheet type="text/xsl" href="configuration.xsl"?> 
          <configuration> 
               <property> 
                 <name>dfs.namenode.secondary.http-address</name> 
                 <value>slave1:50090</value> 
               </property> 
              <property> 
                 <name>dfs.replication</name> 
                 <value>3</value> 
             </property> 
                <property> 
                   <name>dfs.namenode.name.dir</name> 
                   <value>file:/home/hadoop/hadoop-2.6.5/tmp/dfs/name</value> 
                 </property> 
                <property> 
                  <name>dfs.datanode.data.dir</name> 
                  <value>file:/home/hadoop/hadoop-2.6.5/tmp/dfs/data</value> 
               </property> 
           </configuration> 
       d) mapred-site.xml 
         <configuration> 
             <property> 
                  <name>mapreduce.framework.name</name> 
                 <value>yarn</value> 
             </property> 
              <property> 
                    <name>mapreduce.jobhistory.address</name> 
                  <value>master:10020</value> 
               </property> 
              <property> 
                   <name>mapreduce.jobhistory.webapp.address</name> 
                 <value>master:19888</value> 
              </property> 
           </configuration>                   
        e)yarn-site.xml 
         <configuration> 
               <property> 
                   <name>yarn.resourcemanager.hostname</name> 
                   <value>master</value> 
                </property> 
             <property> 
                   <name>yarn.nodemanager.aux-services</name> 
                    <value>mapreduce_shuffle</value> 
            </property> 
         </configuration> 
        f) 修改slaves 
           vim slaves 编辑该文件，输入 
              slave01 
              slave02 
              slave03
              
配置成功后复制该虚拟机两到三台
 建立互信关系：在master生成公私钥ssh-keygen 复制公钥cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
              同样在slave机器上输入ssh-keygen  从master机器上复制公钥scp master:~/.ssh/authorized_keys      /home/hadoop/.ssh/
              测试连接 
启动Hadoop bin目录下./hadoop namenode –format 启动执行./start-all.sh
查看jps 看进程是否正常启动。
