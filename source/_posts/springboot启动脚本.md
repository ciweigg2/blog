title: springboot启动脚本
date: 2019-08-24 10:22:58
tags: [springboot]
categories: [综合]
---
参考：https://blog.csdn.net/qq_40962416/article/details/99069446

SERVER_NAME是不带.jar的名称 脚本应该放在和jar同级目录

```
sudo chmod +x ./app.sh
```

<!--more-->

启动示例 ./app.sh start

vi app.sh

```shell
ENV=dev  
RUNNING_USER=root  
ADATE=`date +%Y%m%d%H%M%S`  
SERVER_NAME=springboot-demo  
  
APP_HOME=`pwd`  
  
dirname $0|grep "^/" >/dev/null  
if [ $? -eq 0 ];then  
   APP_HOME=`dirname $0`  
else  
    dirname $0|grep "^\." >/dev/null  
    retval=$?  
    if [ $retval -eq 0 ];then  
        APP_HOME=`dirname $0|sed "s#^.#$APP_HOME#"`  
    else  
        APP_HOME=`dirname $0|sed "s#^#$APP_HOME/#"`  
    fi  
fi  
  
if [ ! -d "$APP_HOME/logs" ];then  
  mkdir $APP_HOME/logs  
fi  
  
LOG_PATH=$APP_HOME/logs/$SERVER_NAME.out  
GC_LOG_PATH=$APP_HOME/logs/gc-$SERVER_NAME-$ADATE.log  
#JMX监控需用到  
JMX="-Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.port=1091 -Dcom.sun.management.jmxremote.ssl=false -Dcom.sun.management.jmxremote.authenticate=false"  
#JVM参数  
JVM_OPTS="-Dname=$SERVER_NAME -Djeesuite.configcenter.profile=$ENV -Djava.awt.headless=true -Djava.net.preferIPv4Stack=true -Duser.timezone=Asia/Shanghai -Xms512M -Xmx512M -XX:PermSize=256M -XX:MaxPermSize=512M -XX:+HeapDumpOnOutOfMemoryError -XX:+PrintGCDateStamps -Xloggc:$GC_LOG_PATH -XX:+PrintGCDetails -XX:NewRatio=1 -XX:SurvivorRatio=30 -XX:+UseParallelGC -XX:+UseParallelOldGC"  
  
JAR_FILE=$SERVER_NAME.jar  
pid=0  
  
  
start(){  
  checkpid  
  if [ ! -n "$pid" ]; then  
    #JAVA_CMD="nohup java -server -jar $JVM_OPTS $JAR_FILE > $LOG_PATH 2>&1 &"  
    #su - $RUNNING_USER -c "$JAVA_CMD"  
    nohup java -server -jar $JVM_OPTS $JAR_FILE > $LOG_PATH 2>&1 &  
    echo "---------------------------------"  
    echo "启动完成，按CTRL+C退出日志界面即可>>>>>"  
    echo "---------------------------------"  
    sleep 3s  
    tail -f $LOG_PATH  
  else  
      echo "$SERVER_NAME is runing PID: $pid"     
  fi  
  
}  
  
  
status(){  
   checkpid  
   if [ ! -n "$pid" ]; then  
     echo "$SERVER_NAME not runing"  
   else  
     echo "$SERVER_NAME runing PID: $pid"  
      sleep 4s  
     tail -f $LOG_PATH  
   fi   
}  
  
checkpid(){  
    pid=`ps -ef |grep $JAR_FILE |grep -v grep |awk '{print $2}'`  
}  
  
stop(){  
    checkpid  
    if [ ! -n "$pid" ]; then  
     echo "$SERVER_NAME not runing"  
    else  
      dump  
      echo "$SERVER_NAME stop..."  
      kill $pid  
    fi   
}  
  
restart(){  
    stop   
    sleep 1s  
    start  
}  
dump(){   
    LOGS_DIR=$APP_HOME/logs/  
    DUMP_DIR=$LOGS_DIR/dump  
    if [ ! -d $DUMP_DIR ]; then  
        mkdir $DUMP_DIR  
    fi  
    DUMP_DATE=`date +%Y%m%d%H%M%S`  
    DATE_DIR=$DUMP_DIR/$DUMP_DATE  
    if [ ! -d $DATE_DIR ]; then  
        mkdir $DATE_DIR  
    fi  
      
    echo  "Dumping the $SERVER_NAME ...\c"  
      
    PIDS=`ps -ef | grep java | grep $JAR_FILE |awk '{print $2}'`  
    for PID in $PIDS ; do  
        jstack $PID > $DATE_DIR/jstack-$PID.dump 2>&1  
        echo -e  "PID=$PID .\c"  
        jinfo $PID > $DATE_DIR/jinfo-$PID.dump 2>&1  
        echo -e  ".\c"  
        jstat -gcutil $PID > $DATE_DIR/jstat-gcutil-$PID.dump 2>&1  
        echo -e  ".\c"  
        jstat -gccapacity $PID > $DATE_DIR/jstat-gccapacity-$PID.dump 2>&1  
        echo -e  ".\c"  
        jmap $PID > $DATE_DIR/jmap-$PID.dump 2>&1  
        echo -e  ".\c"  
        jmap -heap $PID > $DATE_DIR/jmap-heap-$PID.dump 2>&1  
        echo -e  ".\c"  
        jmap -histo $PID > $DATE_DIR/jmap-histo-$PID.dump 2>&1  
        echo -e  ".\c"  
        if [ -r /usr/sbin/lsof ]; then  
        /usr/sbin/lsof -p $PID > $DATE_DIR/lsof-$PID.dump  
        echo -e  ".\c"  
        fi  
    done  
      
    if [ -r /bin/netstat ]; then  
    /bin/netstat -an > $DATE_DIR/netstat.dump 2>&1  
    echo -e  "netstat.dump ..."  
    fi  
    if [ -r /usr/bin/iostat ]; then  
    /usr/bin/iostat > $DATE_DIR/iostat.dump 2>&1  
    echo -e  "iostat.dump ..."  
    fi  
    if [ -r /usr/bin/mpstat ]; then  
    /usr/bin/mpstat > $DATE_DIR/mpstat.dump 2>&1  
    echo -e  "mpstat.dump ..."  
    fi  
    if [ -r /usr/bin/vmstat ]; then  
    /usr/bin/vmstat > $DATE_DIR/vmstat.dump 2>&1  
    echo -e  "vmstat.dump ..."  
    fi  
    if [ -r /usr/bin/free ]; then  
    /usr/bin/free -t > $DATE_DIR/free.dump 2>&1  
    echo -e  "free.dump ..."  
    fi  
    if [ -r /usr/bin/sar ]; then  
    /usr/bin/sar > $DATE_DIR/sar.dump 2>&1  
    echo -e  ".\c"  
    fi  
    if [ -r /usr/bin/uptime ]; then  
    /usr/bin/uptime > $DATE_DIR/uptime.dump 2>&1  
    echo -e  ".\c"  
    fi  
      
    echo "OK!"  
    echo "DUMP: $DATE_DIR"  
}  
  
case $1 in    
          start) start;;    
          stop)  stop;;   
          restart)  restart;;    
          status)  status;;     
          dump)  dump;;     
              *)  echo "require start|stop|restart|status|dump"  ;;    
esac
```