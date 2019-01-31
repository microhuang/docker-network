#深度理解Docker的网络模型和任意组网需求

##Docker有四种网络模型：
* bridge
* host
* container
* none

本质上，每一种网络模型就是Docker的默认网络配置。而none模型则是一张白纸，为我们提供了最大自由度的组网可能。

![Bridge](pic.jpg)
![Host](pic2.jpg)
![Container](pic3.jpg)
![None](pic4.jpg)

Bridge模式下，Docker服务启动时，会在主机上创建名为docker0的虚拟网桥，默认分配172.17.0.1/16。同时，创建一对虚拟网卡veth pair设备，并将一端放在容器中，并命名为eth0，另一端则在主机中，以veth***命名，并入docker0网桥绑定。

之于以上理解，我们完全可以从none模式定制出双网桥双网卡等特殊组网模型。

##主要包括：
```
建立容器网络隔离空间
ln -sf /proc/<PID>/ns/net /var/run/netns/<container id>

创建veth peer虚拟网卡设备
ip link add veth1_container type veth peer name veth1_root

添加网桥
brctl addbr br0

为网桥分配IP
ifconfig br0 172.18.0.0 netmask 255.255.0.0

将veth peer设备对一端绑定到网桥
brctl addif br0 veth1_root

将veth peer设备对另一端绑定到容器
ip link set veth1_container netns <container id>

可选，修改容器内网卡命名

为容器内网卡分配IP
ip netns exec <container id> ifconfig veth1_container 172.18.0.2 netmask 255.255.0.0

启动veth peer设备
ifconfig veth1_root up
ip netns exec <container id> ifconfig veth1_container up
```

重复以上过程，构造多网桥、多网卡独立网络。
