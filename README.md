# 人大金仓数据库管理系统 (KingbaseES V8 R6) Docker 镜像


## 拉取镜像
- [kingbase-es-v8-r6-docker](https://hub.docker.com/repository/docker/chyiyaqing/kingbase)

```
$ docker pull chyiyaqing/kingbase:v8r6
```

## 构建镜像
```
$ git clone https://github.com/chyidl/kingbase-es-v8-r6-docker.git
$ cd kingbase-es-v8-r6-docker
$ docker build -t kingbase:v8r6 .
```

## 运行
```
$ docker run -d --name kingbasev8r6 -p 54322:54321 -e SYSTEM_USER=kingbasees -e SYSTEM_PWD=kingbasees -v /home/hyperchain/kingbase-es-v8-r6-docker/data:/opt/kingbase/data -v /home/hyperchain/kingbase-es-v8-r6-docker/license.dat:/opt/kingbase/Server/bin/license.dat  kingbase:v8r6
```
- --name: 容器名称
- -p: 端口映射
- -e: 通过环境变量SYSTEM_USER, SYSTEM_PWD指定初始化数据库时的默认用户名和密码
- -v: 挂载宿主机的，挂载数据存储目录

## 启动日志
```
[hyperchain@mini ~]$ docker ps | grep kingbase
046d11fd774c        kingbase:v8r6                                                    "/opt/kingbase/docke…"   26 minutes ago      Up 26 minutes                   0.0.0.0:54322->54321/tcp                                                                kingbasev8r6
[hyperchain@mini ~]$ docker logs 046d11fd774c
The files belonging to this database system will be owned by user "kingbase".
This user must also own the server process.

The database cluster will be initialized with locale "C".
The default text search configuration will be set to "english".

The comparision of strings is case-insensitive.
Data page checksums are disabled.

fixing permissions on existing directory /opt/kingbase/data ... ok
creating subdirectories ... ok
selecting dynamic shared memory implementation ... posix
selecting default max_connections ... 100
selecting default shared_buffers ... 128MB
selecting default time zone ... UTC
creating configuration files ... ok
Begin setup encrypt device
initializing the encrypt device ... ok
running bootstrap script ... ok
performing post-bootstrap initialization ... ok
create security database ... ok
load security database ... ok
create initial audit rules ... ok
syncing data to disk ... ok


Success. You can now start the database server using:

    ./sys_ctl -D /opt/kingbase/data -l logfile start

initdb: warning: enabling "trust" authentication for local connections
You can change this by editing sys_hba.conf or using the option -A, or
--auth-local and --auth-host, the next time you run initdb.
waiting for server to start.... done
server started
2021-11-01 10:45:04.107 UTC [32] LOG:  sepapower extension initialized
2021-11-01 10:45:04.108 UTC [32] LOG:  starting KingbaseES V008R006C005B0013 on x86_64-pc-linux-gnu, compiled by gcc (GCC) 4.1.2 20080704 (Red Hat 4.1.2-46), 64-bit
2021-11-01 10:45:04.108 UTC [32] LOG:  listening on IPv4 address "0.0.0.0", port 54321
2021-11-01 10:45:04.108 UTC [32] LOG:  listening on IPv6 address "::", port 54321
2021-11-01 10:45:04.111 UTC [32] LOG:  listening on Unix socket "/tmp/.s.KINGBASE.54321"
2021-11-01 10:45:04.156 UTC [32] LOG:  redirecting log output to logging collector process
2021-11-01 10:45:04.156 UTC [32] HINT:  Future log output will appear in directory "sys_log".
```

## 目录结构
```
[hyperchain@mini ~]$ tree kingbase-es-v8-r6-docker -L 1
kingbase-es-v8-r6-docker
├── data
├── docker-entrypoint.sh
├── Dockerfile
├── kingbase.tar.gz
├── license.dat
├── password
└── README.md

1 directory, 6 files
```
- kingbase.tar.gz: 通过压缩安装后Kingbase的Server文件夹(该目录下存放了服务器二进制文件、链接文件等, tar -czvf kingbase.tar.gz Server/)

## 常见问题
* 启动失败
- 启动失败: 日志报kingbase: superuser_reserved_connections must be less than max_connections
- 原因: 本仓库中的license.dat 文件是开发测试版，限制最大连接数为10,而人大金仓配置文件默认连接数为100，导致启动失败.
- 解决: 修改数据目录data下的kingbase.conf 配置文件
```
port = 54321                            # (change requires restart)
max_connections = 100                   # (change requires restart)
#superuser_reserved_connections = 3     # (change requires restart)
#unix_socket_directories = '/tmp'       # comma-separated list of directories
```

* 测试License文件替换
```
# 使用企业license版本测试

https://github.com/chyidl/kingbase-es-v8-r6-docker
kingbase-es-v8-r6-docker on  main on 🐳 v20.10.8
➜ pgbench -i  -h 127.0.0.1 -p 54322 -U kingbasees test
Password:
dropping old tables...
NOTICE:  table "pgbench_accounts" does not exist, skipping
NOTICE:  table "pgbench_branches" does not exist, skipping
NOTICE:  table "pgbench_history" does not exist, skipping
NOTICE:  table "pgbench_tellers" does not exist, skipping
creating tables...
generating data (client-side)...
100000 of 100000 tuples (100%) done (elapsed 0.07 s, remaining 0.00 s)
vacuuming...
creating primary keys...
done in 5.95 s (drop tables 0.03 s, create tables 0.13 s, client-side generate 4.59 s, vacuum 0.38 s, primary keys 0.83 s).

kingbase-es-v8-r6-docker on  main on 🐳 v20.10.8 took 8s
➜ pgbench  -c 20 -h 127.0.0.1 -p 54322 -U kingbasees test
Password:
pgbench (14.0, server 12.1)
starting vacuum...end.
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 20
number of threads: 1
number of transactions per client: 10
number of transactions actually processed: 200/200
latency average = 248.896 ms
initial connection time = 842.381 ms
tps = 80.354912 (without initial connection time)

## 使用测试license.dat 进行测试 (测试版license最大连接数10)
kingbase-es-v8-r6-docker on  main on 🐳 v20.10.8 took 5s
➜ pgbench -i  -h 127.0.0.1 -p 54322 -U kingbasees test
Password:
dropping old tables...
NOTICE:  table "pgbench_accounts" does not exist, skipping
NOTICE:  table "pgbench_branches" does not exist, skipping
NOTICE:  table "pgbench_history" does not exist, skipping
NOTICE:  table "pgbench_tellers" does not exist, skipping
creating tables...
generating data (client-side)...
100000 of 100000 tuples (100%) done (elapsed 0.33 s, remaining 0.00 s)
vacuuming...
creating primary keys...
done in 5.68 s (drop tables 0.02 s, create tables 0.12 s, client-side generate 4.40 s, vacuum 0.39 s, primary keys 0.75 s).

kingbase-es-v8-r6-docker on  main [!?] on 🐳 v20.10.8 took 8s
➜ pgbench  -c 20 -h 127.0.0.1 -p 54322 -U kingbasees test
Password:
pgbench (14.0, server 12.1)
starting vacuum...end.
pgbench: error: connection to server at "127.0.0.1", port 54322 failed: FATAL:  sorry, too many clients already
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 20
number of threads: 1
number of transactions per client: 10
number of transactions actually processed: 0/200
```
