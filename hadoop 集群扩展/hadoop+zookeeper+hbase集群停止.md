#!bin/bash

user=eagle
host55=eagle55
host56=eagle56
host57=eagle57
host68=eagle68

JAVA_HOME=/usr/local/java/jdk1.8.0_131/bin

stop-hbase.sh
#zkServer.sh stop
#ssh -t -p39876 ${user}@${host55} 'zkServer.sh stop'
#ssh -t -p39876 ${user}@${host56} 'zkServer.sh stop'
#ssh -t -p39876 ${user}@${host57} 'zkServer.sh stop'
#ssh -t -p39876 ${user}@${host68} 'zkServer.sh stop'

sleep 1

stop-dfs.sh
stop-yarn.sh
mr-jobhistory-daemon.sh stop histroyserver

echo "****67 jps"
jps
echo "****55 jps"
ssh -t -p39876 ${user}@${host55} '/'${JAVA_HOME}'/jps'
echo "****56 jps"
ssh -t -p39876 ${user}@${host56} '/'${JAVA_HOME}'/jps'
echo "****67 jps"
ssh -t -p39876 ${user}@${host57} '/'${JAVA_HOME}'/jps'
echo "****68 jps"
ssh -t -p39876 ${user}@${host68} '/'${JAVA_HOME}'/jps'
