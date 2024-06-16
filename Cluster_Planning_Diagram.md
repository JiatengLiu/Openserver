```
@startuml
!define RECTANGLE class

RECTANGLE "核心交换机" as CoreSwitch {
  H3C WS5820-28X
}

RECTANGLE "防火墙" as Firewall {
}

RECTANGLE "校园网" as CampusNet {
}

RECTANGLE "堡垒机" as BastionHost {
}

RECTANGLE "海外学术资源加速" as OverseasAccelerator {
}

RECTANGLE "workerA" as WorkerA {
  agents
  docker nfs
  Ubuntu Server22
  CPU 内存 10G网卡 GPUs
}

RECTANGLE "workerB" as WorkerB {
  agents
  docker nfs
  Ubuntu Server22
  CPU 内存 10G网卡 GPUs
}

RECTANGLE "proxy & storage" as ProxyStorage {
  files
  nfs-kernel-server
  Ubuntu Server22
  CPU 10G网卡 内存 存储盘
}

CoreSwitch -up-> Firewall
Firewall -up-> CampusNet
CoreSwitch -down-> BastionHost
CoreSwitch -down-> OverseasAccelerator
CoreSwitch -left-> WorkerA
CoreSwitch -left-> WorkerB
CoreSwitch -right-> ProxyStorage

@enduml

```

[PlantText UML Editor](https://www.planttext.com/)