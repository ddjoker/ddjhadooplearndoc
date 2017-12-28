#!bin/bash
user=eagle
host55=eagle55
host56=eagle56
host57=eagle57
host68=eagle68

JAVA_HOME=/usr/local/java/jdk1.8.0_131/bin

#zkServer.sh start
#ssh -p39876 eagle@eagle68 'zkServer.sh start'
#ssh -p39876 eagle@eagle55 'zkServer.sh start'
#ssh -t -p39876 eagle@eagle56 'zkServer.sh start'
#ssh -t -p39876 eagle@eagle57 'zkServer.sh start'

#sleep 1

echo "****67 jps"
#jps
echo "****55 jps"
#ssh -t -p39876 ${user}@${host55} '/'${JAVA_HOME}'/jps'
#echo "****56 jps"
#ssh -t -p39876 ${user}@${host56} '/'${JAVA_HOME}'/jps'
#echo "****67 jps"
#ssh -t -p39876 ${user}@${host57} '/'${JAVA_HOME}'/jps'
echo "****68 jps"
#ssh -t -p39876 ${user}@${host68} '/'${JAVA_HOME}'/jps'


##hadoop-daemon.sh start zkfc
##ssh -t -p39876 eagle@eagle68 'hadoop-daemon.sh start zkfc'

#exit


#hdfs namenode -format

#hadoop-daemon.sh start journalnode
#hadoop-daemon.sh start zkfc

start-dfs.sh
start-yarn.sh
mr-jobhistory-daemon.sh start historyserver

#zkServer.sh start
#ssh -t -p39876 eagle@eagle55 'zkServer.sh start'
#ssh -t -p39876 eagle@eagle56 'zkServer.sh start'
#ssh -t -p39876 eagle@eagle57 'zkServer.sh start'

#start-hbase.sh


#bei yong
#hadoop-daemon.sh start namenode
#hadoop-daemon.sh stop namenode
#hadoop-daemon.sh start datanode
#hadoop-daemon.sh stop datanode
