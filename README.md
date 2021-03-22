# master-slave mysql 双击热备---目前只在windows10 64位下使用成功 linux-暂未使用
dnsmasq 容器已默认将 .test 泛域名解析至 127.0.0.1, 主机首选 dns 设置为 127.0.0.1 即可

如需自定义域名解析请在 [dnsmasq.conf](./conf/dnsmasq.conf) 自行配置

```
master-slave
├── conf
│   ├── dnsmasq.conf
│   └── nginx.conf
├── docker
│   ├── dns
│   │   └── Dockerfile
│   ├── master
│   │   └── Dockerfile
│   └── slave
│       └── Dockerfile
├── docker-compose.yml
└── README.md
```
### 项目说明
此镜像实现mysql 双击热备

| images        | versions   | container name |
| ------------  | ---------- | -------------- |
| mysql-master  | latest     | mysql-master   |
| mysql-slave   | latest     | mysql-slave    |
| dns           | latest     | dns            |

### 使用说明

1. `git clone https://github.com/892349689/mysql-master-slave.git`
2. `cd master-slave`
3. `docker-compose up -d`

### 常用命令

| command                           | description       |
| --------------------------------- | ----------------- |
| docker-compose up -d              | 构建镜像、容器及网络  |
| docker-compose start              | 启动容器            |
| docker-compose stop               | 停止容器            |
| docker-compose restart            | 重启容器            |
| docker-compose down               | 删除所有容器及网络    |
| docker exec -it (容器名) /bin/bash | 进入 (容器名) 容器   |

### 容器构建查看状态:
1. `mysql-master容器`
2. `登录mysql`
3. `执行:ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY 'root';`
4. `执行:FLUSH PRIVILEGES;`
5. `进入mysql-slave容器`
6. `登录mysql`
7. `执行:ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY 'root';`
8. `执行:FLUSH PRIVILEGES;`
9. `重启docker服务`
10. `mysql-master容器登录mysql 查看mysql状态:show master status; 例:`

| File                           | Position       |
| -------------------------------| -------------- |
| replicas-mysql-bin.000004      | 156            |

1. `进入mysql-slave容器`
2. `登录mysql`
3. `停止mysql slave:stop slave; 服务`
4. `运行:CHANGE MASTER TO MASTER_HOST='mysql-master',MASTER_USER='root',MASTER_PASSWORD='root',MASTER_LOG_FILE='replicas-mysql-bin.000003',MASTER_LOG_POS=154;`
5. `注:MASTER_HOST 为mysql-master容器名,MASTER_USER-MASTER_PASSWORD为mysql-master数据库的用户名密码,MASTER_LOG_FILE为mysql-master查出状态的File值,MASTER_LOG_POS为mysql-master查出状态的Position值`
6. `启动mysql slave:start slave; 服务`
7. `查看配置状态:show slave status\G;`
8. `以下图为配置成功:`
[mysql-slave.png](./mysql-slave.png)