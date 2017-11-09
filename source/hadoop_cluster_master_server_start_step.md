#Hadoop Cluster Master Server Start Step

## 1.Start Rinetd
```
rinetd -c /etc/rinetd.conf
```

## 2.Start HBase Thrift Server
```
/usr/hdp/2.6.1.0-129/hbase/bin/hbase-daemon.sh start thrift
```

## 3.Start Nginx
```
nginx
```

## 4.Start Supervisor
```
supervisord -c /data/www/supervisor.conf
supervisorctl
#username:supervisor
#password:supervisor
start bigdata
```
