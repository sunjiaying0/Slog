# 配置同步功能（sync）

* 若现有cmdb还在运行 需要先停掉<br>
    /Users/sunjiaying/go/src/configcenter/src/bin/build/dev 该目录下会有多个脚本：<br>
    start.sh stop.sh restart.sh ...<br>
    stop.sh 掉即可

* 添加sync的配置文件<br>
    /Users/sunjiaying/go/src/configcenter/src/bin/build/dev/cmdb_adminserver/configures<br>
    在该目录下编写 vim sync.conf
    ```
    [synchronize]
    // 多个同步项目使用英文逗号分隔
    name = biz

    [trigger]
    type = interval
    // type=timing,表示每天第几分钟触发
    // type=interval,表示每经过几分钟触发
    role = 3
    //如果上面设置为timing，表示每天的第一分钟触发，如果值为65，则表示凌晨1点5分触发；
    //如果上面设置为interval，则表示每分钟触发一次；

    [redis]
    host = 127.0.0.1
    usr =
    pwd = test123
    database = 0
    port = 6379
    maxOpenConns = 3000
    maxIdleConns = 1000
    //本机redis相关信息可参考其他配置文件中的配置；
    
    [biz]
    model = biz
    trigger_type = interval
    trigger_role = 3
    ```

* 更新zk配置文件<br>
    ```
  ./admin_server --addrport=127.0.0.1:60004 logtostderr=false --log-dir=./logs --v=3  --config=configures/migrate.conf
    ```
    admin进程会将配置文件都同步到zk服务里

* 启动sync进程<br>
    有多种启动方法，这里列出最常用的两种<br>

    1. 在Terminal里启动<br>
    ```
    go build
    ./synchronize_server --addrport=127.0.0.1:60010 --logtostderr=false --log-dir=./logs --v=3 --regdiscv=192.168.23.39:21811
    ```
    2. 使用debug<br>
    在goland 运行debug 'go build XXX' <br>
    添加参数【Program arguments】<br>
    ```
  --addrport=192.168.236.41:60010 --logtostderr=false --log-dir=./logs --v=3 --regdiscv=192.168.23.39:21811
    ```

     随后在需要调试的代码前打上断点即可
