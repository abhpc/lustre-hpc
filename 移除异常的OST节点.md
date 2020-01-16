#### 1. 一些基本的概念
##### 1.1 osc_name
在MDS节点(一般而言是disk01)上通过以下命令获得：
```
#  lctl get_param -NF osc.*.*
```

会输出很多行类似如下的内容：
```
osc.amefs-OST0003-osc-MDT0000.import=
```
两个小数点之间的就是osc_name（上面例子osc_name为amefs-OST0003-osc-MDT0000）

##### 1.2 ost_name
在客户端（一般而言，集群的主节点即可）上通过以下命令获得：
```
# lfs osts
OBDS:
1: amefs-OST0001_UUID ACTIVE
2: amefs-OST0002_UUID ACTIVE
3: amefs-OST0003_UUID INACTIVE
4: amefs-OST0004_UUID ACTIVE
5: amefs-OST0005_UUID ACTIVE
6: amefs-OST0006_UUID ACTIVE

```
在"\_UUID"之前的内容即为ost_name。

#### 2. 暂停该OST节点的写入
在MDS节点上(disk01)执行以下命令：
```
# lctl set_param osp.[osc_name].active=0
```
这里的osc_name可通过1.1方法获得。例如：
```
# lctl set_param osp.amefs-OST0003-osc-MDT0000.active=0
```

#### 3. 将该OST节点上的数据迁移到其他节点上去

**注意：这一步的执行时间很长，千万不可中断，否则可能产生奇怪的错误**

在客户端(集群主节点)上执行以下命令：
```
# lfs find --ost ost_name /mount/point | lfs_migrate -y
```
假设移除的是OST是amefs-OST0003，挂在主节点上的目录是/home-ame，则执行命令为：
```
# lfs find -ost amefs-OST0003 /home-ame | lfs_migrate -y
```
等全部执行完毕后，即完成数据迁移。

#### 4. 下线该OST节点
在MDS节点上执行以下命令：
```
lctl conf_param ost_name.osc.active=0
```
恢复的话，将0改成1即可恢复。
