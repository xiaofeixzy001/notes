[TOC]

**LVS持久连接的意义：**

  在固定时间内将来自于同一个客户端发往VIP的所有请求统统定向至同一个RS；在指定时长内不再根据调度算法进行调度，会根据内存的ipvs的连接模版里的记录信息将同一个客户端的请求定向至同一个后端RS；

持久连接的类型：

  1.**PCC**：持久客户端连接，将来自于同一个客户端发往VIP的所有请求统统定向至同一个RS （只是根据Vip，所有服务都是集群服务，不同的服务都会被定向至同一个RS）；

  2.**PPC**：持久端口连接，将来自于同一个客户端发往某VIP的某端口的所有请求统统定向至同一个RS中  （根据Vip和端口号，不同的服务不会再被定向至同一个RS）；

  3.**PFMC**：持久防火墙标记连接，基于防火墙标记，将两个或两个以上的端口绑定为同一个服务（应用场景是http服务和https服务需要定向至同一个RS）,这种防火墙标记仅在数据包在分发器上时有影响，数据包一旦离开director，就不再被标记.

案例：在电商网站中提供http服务和https服务时如何将同一客户端的VIP请求定向至同一个RS中？

1.第一种解决方案： 使用PCC，但是这种方法有一个缺点就是会将同一个客户端请求的所有集群服务定向至同一个RS；

2.第二种解决方案： 使用PFMC，可以将两个集群服务绑定为同一个服务，对于两个不同端口的请求定向至同一后端RS；

1，**PPC的演示**（这里是基于LVS-DR模型架构的）

添加一个集群服务，VIP地址为192.168.100.7，RS为10.0.0.8，10.0.0.9访问VIP的任意端口或80端口，都开启持久连接功能，超时时间为120秒

 

```
# ipvsadm -C
# ipvsadm -A -t 192.168.100.7:80 -s rr -p 120            //-p 持久连接超时时间，单位秒

# ipvsadm -a -t 192.168.100.7:80 -r 10.0.0.8 -g
# ipvsadm -a -t 192.168.100.7:80 -r 10.0.0.9 -g
# ipvsadm -L -n
```

测试：

在客户端浏览器访问http://192.168.100.7，多次刷新查看页面显示内容。

在xshell中通过ssh访问Director，发现连接到的主机已经不再是Director，而是后端的RS。

说明：这样只要用户是在长连接的时间限制内访问集群服务的80端口，都会被绑定到同一个RS上，如果超时后发现用户还是处于活跃状态，则会在访问模版上为该用户默认添加两分钟的连接时长，直到用户不再响应，且2分钟内不再活跃，就会断开连接；

2，**PCC的演示**（这里是基于LVS-DR模型架构的，关于其配置不再赘述）  

 

```
# ipvsadm -C
# ipvsadm -A -t 192.168.100.7:0 -s rr -p 120        
# ipvsadm -a -t 192.168.100.7:0 -r 10.0.0.8 -g
# ipvsadm -a -t 192.168.100.7:0 -r 10.0.0.9 -g
# ipvsadm -L -n
```

测试：

在客户端浏览器访问http://192.168.100.7，多次刷新查看页面显示内容。

在xshell中通过ssh访问Director，发现连接到的主机已经不再是Director，而是后端的RS。

说明：当我们将LVS服务的端口设置为0时，就是指所有端口，表示将来自同一个客户端的任何请求（端口不限）都转给后端的某一RS，基于持久连接时，来自于同一个客户端的所有请求统统被转发至同一个RS。

3，PFMC的演示（这里是基于LVS-DR模型架构的）

 

```
# ipvsadm -C
# ipvsadm -t mangle -A PREROUTING -d 192.168.100.7 -p tcp -dport 80 -j MARK --set-mark 10     //MARK的值是在0--99之间的任意一个整数值
# ipvsadm -t mangle -A PREROUTING -d 192.168.100.7 -p tcp -dport 443 -j MARK --set-mark 10
# service ipvsadm save
# ipvsadm -A -f 10 -s rr -p 120
# ipvsadm -a -f 10 -r 10.0.0.8 -g 
# ipvsadm -a -f 10 -r 10.0.0.9 -g
# ipvsadm -L -n
# service ipvsadm save
```

RS的健康状态检测：

1，自动上下线各RS

​    如果RS状态发生了改变，比如：online--->down，一般需要探测3次以上才能确认；若offline--->OK，一般探测2次以上；

2，所有RS故障时，应该提供一个back server

如何检测RS的健康状态：

   利用集群服务所依赖的协议进行检测；

   可利用端口扫描或探测类工具对指定协议的端口进行探测；

  在网络层上进行探测，如ping

RS健康状态检查脚本示例第一版：

```
 RSTATUS=("1" "1") RW=("2" "1") RPORT=80 TYPE=g  add() {   ipvsadm -a -t $VIP:$CPORT -r $1:$RPORT -$TYPE -w $2   [ $? -eq 0 ] && return 0 || return 1 }  del() {   ipvsadm -d -t $VIP:$CPORT -r $1:$RPORT   [ $? -eq 0 ] && return 0 || return 1 }  while :; do   let COUNT=0   for I in ${RS[*]}; do     if curl --connect-timeout 1 http://$I &> /dev/null; th       if [ ${RSTATUS[$COUNT]} -eq 0 ]; then          add $I ${RW[$COUNT]}          [ $? -eq 0 ] && RSTATUS[$COUNT]=1       fi     else       if [ ${RSTATUS[$COUNT]} -eq 1 ]; then          del $I          [ $? -eq 0 ] && RSTATUS[$COUNT]=0       fi     fi     let COUNT++   done   sleep 5 done
```

RS健康状态检查脚本示例第二版：

 

```
#!/bin/bash
        #
        VIP=192.168.10.3
        CPORT=80
        FAIL_BACK=127.0.0.1
        RS=("192.168.10.7" "192.168.10.8")
        declare -a RSSTATUS
        RW=("2" "1")
        RPORT=80
        TYPE=g
        CHKLOOP=3
        LOG=/var/log/ipvsmonitor.log


        addrs() {
          ipvsadm -a -t $VIP:$CPORT -r $1:$RPORT -$TYPE -w $2
          [ $? -eq 0 ] && return 0 || return 1
        }


        delrs() {
          ipvsadm -d -t $VIP:$CPORT -r $1:$RPORT 
          [ $? -eq 0 ] && return 0 || return 1
        }


        checkrs() {
          local I=1
          while [ $I -le $CHKLOOP ]; do 
            if curl --connect-timeout 1 http://$1 &> /dev/null; then
              return 0
            fi
            let I++
          done
          return 1
        }


        initstatus() {
          local I
          local COUNT=0;
          for I in ${RS[*]}; do
            if ipvsadm -L -n | grep "$I:$RPORT" && > /dev/null ; then
              RSSTATUS[$COUNT]=1
            else 
              RSSTATUS[$COUNT]=0
            fi
          let COUNT++
          done
        }


        initstatus
        while :; do
          let COUNT=0
          for I in ${RS[*]}; do
            if checkrs $I; then
              if [ ${RSSTATUS[$COUNT]} -eq 0 ]; then
                 addrs $I ${RW[$COUNT]}
                 [ $? -eq 0 ] && RSSTATUS[$COUNT]=1 && echo "`date +'%F %H:%M:%S'`, $I is back." >> $LOG
              fi
            else
              if [ ${RSSTATUS[$COUNT]} -eq 1 ]; then
                 delrs $I
                 [ $? -eq 0 ] && RSSTATUS[$COUNT]=0 && echo "`date +'%F %H:%M:%S'`, $I is gone." >> $LOG
              fi
            fi
            let COUNT++
          done 
          sleep 5
        done
```