# 各个进程的启动命令

```
【web】
./web_server --addrport=127.0.0.1:8090 --logtostderr=false --log-dir=./logs --v=3 --regdiscv=192.168.23.39:21811

【admin】
./admin_server --addrport=127.0.0.1:60004 logtostderr=false --log-dir=./logs --v=3  --config=configures/migrate.conf

【core】
./coreservice --addrport=127.0.0.1:50009 --logtostderr=false --log-dir=./logs --v=3 --regdiscv=192.168.23.39:21811

【api】
./apiserver --addrport=127.0.0.1:8080 --logtostderr=false --log-dir=./logs --v=3 --regdiscv=192.168.23.39:21811 --enable-auth=false

【topo】
./topo_server --addrport=127.0.0.1:60002 --logtostderr=false --log-dir=./logs --v=3 --regdiscv=192.168.23.39:21811 --enable-auth=false

【host】
./host_server --addrport=127.0.0.1:60001 --logtostderr=false --log-dir=./logs --v=3 --regdiscv=192.168.23.39:21811 --enable-auth=false

【task】
./task_server --addrport=127.0.0.1:60012 --logtostderr=false --log-dir=./logs --v=3 --regdiscv=192.168.23.39:21811

【sync】
./synchronize_server --addrport=127.0.0.1:60010 --logtostderr=false --log-dir=./logs --v=3 --regdiscv=192.168.23.39:21811

【datacollection】
--addrport=127.0.0.1:60005 --logtostderr=false --log-dir=./logs --v=3 --regdiscv=192.168.23.39:21811

```