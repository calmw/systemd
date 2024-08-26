## 简介

- Centos7.4开机第一个程序从init完全换成了systemd这种启动方式，同centos 5 6已经是实质差别。systemd是靠管理unit的方式来控制开机服务，开机级别等功能。
  在/usr/lib/systemd/system目录下包含了各种unit文件，有service后缀的服务unit，有target后缀的开机级别unit等，这里介绍关于service后缀的文件。因为systemd在开机要想执行自启动，都是通过这些*
  .service 的unit控制的，服务又分为系统服务（system）和用户服务（user）。
  系统服务：开机不登陆就能运行的程序（常用于开机自启）。
  用户服务：需要登陆以后才能运行的程序。

## 配置文件说明：

- ini 代码解读复制代码[UNIT]

```shell
#服务描述
Description=Media wanager Service
#指定了在systemd在执行完那些target之后再启动该服务
After=network.target
[Service]
#定义Service的运行类型，一般是forking(后台运行)   
Type=forking

#定义systemctl start|stop|reload *.service 的执行方法（具体命令需要写绝对路径）
#注：ExecStartPre为启动前执行的命令
ExecStart=/opt/nginx/sbin/nginx
ExecReload=/opt/nginx/sbin/nginx -s reload
ExecStop=/opt/nginx/sbin/nginx -s stop

#创建私有的内存临时空间
PrivateTmp=True

[Install]
#多用户
WantedBy=multi-user.target
```

- ini 代码解读复制代码

```shell
[Unit] 区块：启动顺序与依赖关系

Description字段：给出当前服务的简单描述。
Documentation 字段：给出文档位置。
After 字段：如果network.target或sshd-keygen.service需要启动，那么sshd.service应该在它们之后启动。
Before字段：定义sshd.service应该在哪些服务之前启动。
注：After和Before字段只涉及启动顺序，不涉及依赖关系。

[Service] 区块：启动行为

启动命令
Type=forking是后台运行的形式

ExecStart字段：定义启动进程时执行的命令
ExecReload字段：重启服务时执行的命令
ExecStop字段：停止服务时执行的命令

ExecStartPre字段：启动服务之前执行的命令
ExecStartPost字段：启动服务之后执行的命令
ExecStopPost字段：停止服务之后执行的命令

#注：所有的启动设置之前，都可以加上一个连词号（-），表示"抑制错误"
，即发生错误的时候，不影响其他命令的执行。比如EnvironmentFile=-/etc/sysconfig/sshd（注意等号后面的那个连词号），就表示即使/etc/sysconfig/sshd文件不存在，也不会抛出错误。
#注意：[Service]中的启动、重启、停止命令全部要求使用绝对路径！

启动类型
Type字段定义启动类型。它可以设置的值如下：
simple（默认值）：ExecStart字段启动的进程为主进程
forking：ExecStart字段将以fork()方式启动，此时父进程将会退出，子进程将成为主进程（后台运行）
oneshot：类似于simple，但只执行一次，Systemd 会等它执行完，才启动其他服务
dbus：类似于simple，但会等待 D-Bus 信号后启动
notify：类似于simple，启动结束后会发出通知信号，然后 Systemd 再启动其他服务
idle：类似于simple，但是要等到其他任务都执行完，才会启动该服务。一种使用场合是为让该服务的输出，不与其他服务的输出相混合

PrivateTmp=True表示给服务分配独立的临时空间

[Install] 服务安装的相关设置，可设置为多用户

[Install] 区块
Install 区块，定义如何安装这个配置文件，即怎样做到开机启动。
WantedBy字段：表示该服务所在的 Target。
Target的含义是服务组，表示一组服务。
WantedBy=multi-user.target指的是：sshd 所在的 Target 是multi-user.target。

#这个设置非常重要，因为执行systemctl enable
sshd.service命令时，sshd.service的一个符号链接，就会放在/etc/systemd/system目录下面的multi-user.target.wants子目录之中。
Systemd 有默认的启动 Target。
```

## 常用命令

```shell
# 开机启动
systemctl enable mysqld

# 关闭开机启动
systemctl disable mysqld

# 启动服务
systemctl start mysqld

# 停止服务
systemctl stop mysqld

# 重启服务
systemctl restart mysqld

# 查看服务状态
systemctl status mysqld
systemctl is-active sshd.service

# 结束服务进程(服务无法停止时)
systemctl kill mysqld

#查看全部log
journalctl -u service-name.service

#查看最新的10行log
journalctl -u service-name.service | tail -n 10
```