# Ansible 使用 ssh 通过 user/pass 访问远程主机

本文属于 Ansible 基础文章，本文示例代码地址：https://github.com/penglongli/ansible-sample/tree/master/sample-1

## 背景

Ansible 是使用 ssh 的方式来访问远程主机的，ssh 一般情况下有两种：

- 用户密码的方式（user/pass）
- 公钥登录

本文介绍的是通过“用户密码”的方式来访问远程主机，但是生产环境推荐使用“公钥”的方式。关于公钥的方式会在后文中做详细阐述。

### 环境

- 操作系统：Ubuntu 16.04 64bit
- Ansible 版本：2.4.0.0

### 准备

 在阿里云开了两台临时的机器，分别以 A、B 代替

- A 机器：172.31.16.67，Ubuntu 16.04，此机器作为主机器
- B 机器：172.31.16.68，Ubuntu 16.04

A、B 两台机器都需要预先安装上 Ansible，安装文档：[Ansible 安装](http://docs.ansible.com/ansible/latest/intro_installation.html#latest-releases-via-apt-ubuntu)

### 目标

在 A 主机上，使用 ansible 通过 ssh 的 user/pass 模式访问 B 主机

## 过程

登录 A 机器，建立文件夹 ansible-test 作为脚本目录。

```
root@A:~# mkdir ansible-test && cd ansible-test
root@A:~/ansible-test#
```

### 建立 Inventory

按照如下步骤执行：

```
root@A:~/ansible-test# mkdir inventory && cd inventory
root@A:~/ansible-test/inventory# vim inventory
[server]
172.31.16.68

[server:vars]
ansible_ssh_user="{{ remote_user }}"
ansible_ssh_pass="{{ remote_pass }}"
```

- [server]    分组名称，可以随意自定义
- 172.31.16.18    机器 B 的内网 IP 地址
- [server:vars]     定义分组 server 的变量
- {{ remote_user }}    传递来的 remote_user 变量

### 建立 ansible.cfg

```
root@A:~/ansible-test# vim ansible.cfg
[defaults]
host_key_checking=false
```

- host_key_checking=false    当初次链接远程主机，会有一个询问，如果 yes 则会添加记录到 ~/.ssh/known_hosts 文件中。此选项会默认 Yes

  参考：http://docs.ansible.com/ansible/latest/intro_getting_started.html#host-key-checking

### 执行

```
root@A:~/ansible-test# ansible -i inventory/inventory all -m ping  -e "remote_user=root remote_pass=123456"

172.31.16.68 | SUCCESS => {
    "changed": false,
    "failed": false,
    "ping": "pong"
}
```

- -i 参数，指定使用的 inventory 文件地址
- -e 参数，指定传入参数

从上述可以看到，我们访问远程主机是成功了



我们查看下我们的目录结构：

```
root@A:~/ansible-test# tree
.
├── ansible.cfg
└── inventory
    └── inventory

1 directory, 2 files
```



## 知识点

- Ansible 默认配置文件，读取顺序

  ```
  * ANSIBLE_CONFIG (一个环境变量)
  * ansible.cfg (位于当前目录中)
  * .ansible.cfg (位于家目录中)
  * /etc/ansible/ansible.cfg
  ```

  Ansible 将会按以上顺序逐个查询这些文件,直到找到一个为止,并且使用第一个寻找到个配置文件的配置,这些配置将不会被叠加.

  参考文档：http://ansible-tran.readthedocs.io/en/latest/docs/intro_configuration.html