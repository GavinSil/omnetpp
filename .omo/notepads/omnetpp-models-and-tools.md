# OMNeT++ 仿真模型与工具完整目录

> 从 OMNeT++ 官方 Models and Tools 页面提取，共 152 个项目，按类别分组。

---

## 一、互联网协议与路由 (27项)

### 1. INET Framework
- **描述**: OMNeT++ 最核心的开源协议模型库，包含 TCP/UDP/IP、以太网、WiFi、MANET 协议、移动性、DiffServ、MPLS 等
- **关键字**: tcp/ip, internet, lan, wan, manet, wireless, routing, ipv6, mpls, voip, ospf, bgp, ethernet, vlan, wifi, 802.11, 802.15.4
- **兼容**: omnetpp3, 4, 5, 6 / inet1, 2, 3, 4
- **类型**: framework + model
- **精选**: ★ 是
- **安装**: `opp_env install inet-latest`
- **链接**: https://github.com/inet-framework/inet/releases

### 2. ANSA - Automated Network Simulation and Analysis
- **描述**: INET 扩展，提供 HSRP、VRRP、ISIS、RIP、EIGRP、Babel、LISP、PIM、STP、TRILL、VLAN 等多种路由/交换协议模型
- **关键字**: hsrp, vrrpv2, glbp, isis, rip, eigrp, babel, ripv2, ripng, cdp, lldp, stp, trill, lisp, pim-dm, pim-sm, igmpv2, igmpv3, vlan, rbridge, clns
- **兼容**: omnetpp4, 5 / inet3
- **类型**: framework + model
- **精选**: ★ 是
- **安装**: `opp_env install ansa-latest`
- **链接**: https://github.com/kvetak/ANSA

### 3. AQTmodel - Adversarial Queueing Framework
- **描述**: 模拟计算机网络中对抗性排队（高丢包）场景，研究网络不稳定性下的丢包行为
- **关键字**: traffic, queueing, latency, packet loss
- **兼容**: omnetpp4
- **类型**: model
- **精选**: 否
- **链接**: https://github.com/dasebe/AQTmodel

### 4. Antnet
- **描述**: 基于蚁群优化的路由算法（AntNet-CL 和 AntNet-CO）实现
- **关键字**: ad hoc, ant, routing, antnet
- **兼容**: omnetpp3
- **类型**: model
- **精选**: 否
- **链接**: https://github.com/omnetpp-models/archive/releases/download/archive/antnet-4.0-src.zip

### 5. ECMP - Equal-cost multi-path routing for Clos networks
- **描述**: INET 扩展，实现 Clos 网络中二层等价多路径（ECMP）路由
- **关键字**: routing
- **兼容**: omnetpp6 / inet4
- **类型**: model
- **精选**: 否
- **安装**: `opp_env install ecmp_allinone-latest`
- **链接**: https://github.com/inet-framework/inet-clos-ecmp

### 6. Fisheye State Routing
- **描述**: 实现 IETF 鱼眼状态路由草案（draft-ietf-manet-fsr-01/02）
- **关键字**: (无明确关键字)
- **兼容**: omnetpp2
- **类型**: model
- **精选**: 否
- **链接**: https://github.com/omnetpp-models/archive/releases/download/archive/FSR-20021124-src.tgz

### 7. PASER - Position-Aware Secure and Efficient Mesh Routing
- **描述**: 面向无线 mesh 网络的高效安全路由协议，在安全性和性能之间取得折中
- **关键字**: mesh, ad hoc, routing
- **兼容**: omnetpp4 / inet2
- **类型**: model
- **精选**: 否
- **链接**: https://github.com/PASER/Simulation-Linux

### 8. OppBSD
- **描述**: 将 FreeBSD 网络栈集成到 OMNeT++，模拟主机运行真实 FreeBSD 内核网络协议栈，高度逼真
- **关键字**: ipv6, tcp, ipv4, udp, icmpv6, arp, nd, ethernet
- **兼容**: omnetpp3, 4
- **类型**: model
- **精选**: 否
- **安装**: `opp_env install oppbsd-latest`

### 9. Quagga for INET Framework
- **描述**: 将 Quagga 路由守护进程移植到 INET，可用标准 Quagga 配置文件配置模拟路由器
- **关键字**: routing, ospfv2, ospfv3, ripv1, ripv2, ripng, bgpv4
- **兼容**: omnetpp3, 4 / inet1
- **类型**: model
- **精选**: 否
- **安装**: `opp_env install quagga-latest`
- **链接**: https://github.com/inet-framework/inet-quagga

### 10. OpenFlow Extension for INET Framework
- **描述**: 基于 OpenFlow 1.0 规范的 SDN 仿真模型，实现数据平面与控制平面分离
- **关键字**: ethernet, sdn
- **兼容**: omnetpp4, 5, 6 / inet2, 3, 4
- **类型**: model
- **精选**: 否
- **安装**: `opp_env install openflow4core-latest`
- **链接**: https://github.com/inet-framework/openflow

### 11. DNS / mDNS model for INET
- **描述**: INET 的 DNS 和组播 DNS（mDNS）流量仿真扩展
- **关键字**: mdns
- **兼容**: omnetpp4 / inet3
- **类型**: model
- **精选**: 否
- **安装**: `opp_env install dns-latest`
- **链接**: https://github.com/saenridanra/inet-dns-extension

### 12. HIPSim++
- **描述**: 主机标识协议（HIP）的 OMNeT++/INET 仿真实现，兼顾 xMIPv6 扩展
- **关键字**: routing, xmipv6
- **兼容**: omnetpp4 / inet1
- **类型**: model
- **精选**: 否

### 13. xMIPv6
- **描述**: 可扩展的移动 IPv6 仿真模型，已合并至 INET 框架
- **关键字**: xmipv6, ipv6
- **兼容**: omnetpp4 / inet2（已合并）
- **类型**: model（已合并）
- **精选**: 否
- **链接**: https://github.com/zarrar/xMIPv6/

### 14. mCoA++ - Multiple Care of Address Registration for xMIPv6
- **描述**: 扩展 xMIPv6，支持移动 IPv6 节点注册多个转交地址
- **关键字**: (无明确关键字)
- **兼容**: omnetpp4 / inet2
- **类型**: model
- **精选**: 否
- **链接**: https://github.com/bmsousa/mCoAplus

### 15. TCP-Fit and TCP-Illinois models for OMNeT++ and INET
- **描述**: TCP-Fit 和 TCP-Illinois 拥塞控制机制的 OMNeT++/INET 实现
- **关键字**: (无明确关键字)
- **兼容**: omnetpp5 / inet2
- **类型**: model
- **精选**: 否
- **安装**: `opp_env install tcp_fit_illinois-latest`
- **链接**: https://github.com/SpyrosMArtel/TCP-Fit-Illinois

### 16. INETMANET 3.x
- **描述**: INET 3.x 的分支版本，包含社区贡献的额外自组网路由协议和模型
- **关键字**: mobility, wireless, manet, ad hoc
- **兼容**: omnetpp4, 5 / inet3
- **类型**: framework + model
- **精选**: 否
- **安装**: `opp_env install inetmanet3-latest`
- **链接**: https://github.com/aarizaq/inetmanet-3.x

### 17. INETMANET 4.x
- **描述**: INET 4.x 的分支版本，扩展了移动自组网相关协议和功能
- **关键字**: mobility, wireless, manet, ad hoc
- **兼容**: omnetpp5, 6 / inet4
- **类型**: framework + model
- **精选**: 否
- **安装**: `opp_env install inetmanet4-latest`
- **链接**: https://github.com/aarizaq/inetmanet-4.x

### 18. INET-HNRL
- **描述**: INET 的混合网络研究分支，新增光网络和无线网络混合模型（TDM/WDM-PON、NGOA 架构等）
- **关键字**: optical, wireless, hybrid, access network
- **兼容**: omnetpp4, 5 / inet1, 2, 3
- **类型**: model
- **精选**: 否
- **安装**: `opp_env install inet_hnrl-latest`
- **链接**: https://github.com/kyeongsoo/inet-hnrl

### 19. IP-Suite (20040812)
- **描述**: INET 框架的前身，包含 IPv4、TCP、UDP 模型和 QoS 支持
- **关键字**: ipv4, tcp, udp
- **兼容**: omnetpp2, 3（已弃用）
- **类型**: framework + model（已弃用）
- **精选**: 否

### 20. ReaSE
- **描述**: 提供图形界面生成真实拓扑 NED 文件，扩展 OMNeT++/INET 支持层次化路由和流量生成
- **关键字**: traffic generation, topology generation
- **兼容**: omnetpp3, 4 / inet1, 2
- **类型**: model
- **精选**: 否
- **安装**: `opp_env install rease-latest`
- **链接**: https://github.com/ToGaKIT/ReaSE

### 21. EtherDemo
- **描述**: 简单以太网仿真演示，已合并到 INET 框架
- **关键字**: ethernet
- **兼容**: omnetpp2（已合并）
- **类型**: model（已合并）
- **精选**: 否

### 22. Ethernet
- **描述**: 以太网/快速以太网/千兆以太网模型（MAC、LLC、交换机、集线器、总线），已合并至 INET
- **关键字**: ethernet
- **兼容**: omnetpp2, 3（已合并）
- **类型**: model（已合并）
- **精选**: 否

### 23. HttpTools
- **描述**: HTTP 流量仿真组件集，已合并至 INET 框架
- **关键字**: (无明确关键字)
- **兼容**: omnetpp4 / inet1（已合并）
- **类型**: model（已合并）
- **精选**: 否

### 24. RTP (Real-time Transport Protocol) model
- **描述**: RTP v2 (RFC 1889) 模型，已合并至 INET 框架
- **关键字**: rtpv2
- **兼容**: omnetpp2（已合并）
- **类型**: model（已合并）
- **精选**: 否

### 25. SimpleBus
- **描述**: 通用总线模型（传播延迟、数据速率、碰撞检测），已被以太网包中的总线模型替代
- **关键字**: ethernet
- **兼容**: omnetpp3（已合并）
- **类型**: model（已合并）
- **精选**: 否

### 26. VoIPTool for INET
- **描述**: 逼真的 VoIP 流量生成和评估工具，已合并至 INET 3.x
- **关键字**: voip
- **兼容**: omnetpp4 / inet1（已合并）
- **类型**: model（已合并）
- **精选**: 否
- **链接**: https://github.com/kirankishore/voiptool

### 27. libARA - Ant Colony Optimization routing algorithms
- **描述**: 基于蚁群优化（ACO）元启发式的路由算法框架，构建于 INETMANET
- **关键字**: antnet, aco, routing
- **兼容**: omnetpp4 / inet1
- **类型**: model
- **精选**: 否
- **安装**: `opp_env install libara_allinone-latest`
- **链接**: https://github.com/des-testbed/libara

---

## 二、无线/传感器/物联网网络 (34项)

### 1. 6LoWPAN Model for OMNeT++
- **描述**: 将 Contiki 的 6LoWPAN 实现集成到 OMNeT++ 的自集成版模型
- **关键字**: wpan, ipv6, 802.15.4, contiki, wsn, sensor, wireless
- **兼容**: omnetpp4 / inet2
- **类型**: model
- **精选**: 否
- **链接**: https://github.com/michaelkirsche/6lowpan4omnet-diy

### 2. AdHocSim
- **描述**: 自组网模拟器，实现 AODV 协议和多种移动性模型
- **关键字**: aodv, ad hoc, manet, mobility, wireless
- **兼容**: omnetpp3
- **类型**: model
- **精选**: 否

### 3. BLE - Bluetooth Low Energy model
- **描述**: 蓝牙低功耗协议底层（PHY 和 LL）仿真模型，基于 MiXiM
- **关键字**: bluetooth, ble, wpan, mesh, wsn, sensor, wireless
- **兼容**: omnetpp4
- **类型**: model
- **精选**: 否

### 4. CMM and ORBIT (SOLAR) mobility models
- **描述**: 社区移动性模型和 ORBIT 太阳移动性模型，用于机会网络研究
- **关键字**: mobility, cmm, orbit, opportunistic networking
- **兼容**: omnetpp5 / inet4
- **类型**: model
- **精选**: 否
- **链接**: https://github.com/ComNets-Bremen/Mobility-Models

### 5. Castalia
- **描述**: 无线传感器网络和体域网仿真器，提供时间变化路径损耗、精细干扰/RSSI 计算、物理过程建模、时钟漂移等
- **关键字**: 802.15.4, wsn, sensor, wireless, wpan, tmac
- **兼容**: omnetpp4
- **类型**: framework + model（已弃用，推荐用 INET）
- **精选**: 否
- **安装**: `opp_env install castalia-latest`
- **链接**: https://github.com/badapplexx/Castalia

### 6. ChSim - Channel Simulator
- **描述**: 无线信道模拟器，生成单蜂窝上下行链路信道状态值，包含多种移动性和信道模型
- **关键字**: wireless, channel
- **兼容**: omnetpp3（已弃用）
- **类型**: model
- **精选**: 否

### 7. CometOS
- **描述**: 面向无线网络的组件化微小操作系统，可在 OMNeT++ 和实际传感器平台上运行同一协议代码
- **关键字**: wsn, sensor, wireless, routing, 802.15.4, ipv6, 6lowpan, rpl, uart, i2c, rs-485, spi, dsme, http, mis, aodv, gpsr
- **兼容**: omnetpp4, 5
- **类型**: model
- **精选**: 否
- **链接**: https://github.com/CometOS/CometOS

### 8. ComNets RPL
- **描述**: INET 框架的 RPL 路由协议（RFC 6550）模型
- **关键字**: routing, wireless, wsn
- **兼容**: omnetpp5 / inet4
- **类型**: model
- **精选**: 否
- **安装**: `opp_env install rpl_allinone-latest`
- **链接**: https://github.com/ComNetsHH/omnetpp-rpl

### 9. ComNets TDMA
- **描述**: INET 框架的抽象 TDMA MAC 协议模型，按帧/时隙调度传输
- **关键字**: tdma
- **兼容**: omnetpp5 / inet4
- **类型**: model
- **精选**: 否
- **安装**: `opp_env install omnet_tdma-latest`
- **链接**: https://github.com/ComNetsHH/omnet-tdma

### 10. ComNets TSCH
- **描述**: IEEE 802.15.4e TSCH 及 6TiSCH 协议栈仿真模型，结合 RPL
- **关键字**: tsch
- **兼容**: omnetpp5 / inet4
- **类型**: model
- **精选**: 否
- **安装**: `opp_env install tsch_allinone-latest`
- **链接**: https://github.com/ComNetsHH/omnetpp-tsch

### 11. Directional Radio Models
- **描述**: INET 框架的方向性天线辐射模式（Cardioid、Circular、Folium、Rose 等）
- **关键字**: wireless, 802.11
- **兼容**: omnetpp4 / inet1
- **类型**: model
- **精选**: 否
- **链接**: https://github.com/jmaureir/DirectionalRadio

### 12. FLoRa - Framework for LoRa
- **描述**: LoRa 网络端到端仿真框架，支持自适应数据速率（ADR）和能耗统计
- **关键字**: iot, lora, wsn, sensor, wireless
- **兼容**: omnetpp5, 6 / inet3, 4
- **类型**: model + framework
- **精选**: 否
- **安装**: `opp_env install flora-latest`
- **链接**: https://github.com/florasim/flora

### 13. IEEE802154INET-Standalone
- **描述**: IEEE 802.15.4-2006 仿真模型，独立于 INET
- **关键字**: 802.15.4, wsn, sensor, wireless, wpan
- **兼容**: omnetpp4 / inet2
- **类型**: model
- **精选**: 否
- **安装**: `opp_env install ieee802154standalone-latest`
- **链接**: https://github.com/michaelkirsche/IEEE802154INET-Standalone

### 14. LEACH (Low-Energy Adaptive Clustering Hierarchy)
- **描述**: 基于 IEEE 802.15.4 的 LEACH 分层自组织路由协议 OMNeT++ 实现
- **关键字**: wireless, sensor, wsn, power
- **兼容**: omnetpp5 / inet4
- **类型**: model
- **精选**: 否
- **链接**: https://github.com/Agr-IoT/LEACH

### 15. LEACH Protocol Wireless Sensor Network Simulation (InetLeach)
- **描述**: LEACH 协议 OMNeT++/INET 实现，含动态分簇选举、TDMA 数据传输、能耗跟踪
- **关键字**: leach
- **兼容**: omnetpp6 / inet4
- **类型**: model
- **精选**: 否
- **链接**: https://github.com/xcodebn/PiLeachProtocol

### 16. MiXiM
- **描述**: 面向无线传感器/体域/自组网/车载网的建模框架，含详细无线电波传播、干扰估算、功耗模型（已弃用，代码合并至 INET 3.x）
- **关键字**: manet, ad hoc, mobility, wireless, wsn, sensor, wpan, 802.15.4, power
- **兼容**: omnetpp3, 4（已弃用）
- **类型**: framework + model
- **精选**: 否
- **安装**: `opp_env install mixim-latest`

### 17. Mobility Framework for OMNeT++ 3.x
- **描述**: 支持无线和移动仿真的框架，已合并至 INET
- **关键字**: wireless, mobility
- **兼容**: omnetpp3（已弃用）
- **类型**: framework + model（已弃用）
- **精选**: 否

### 18. Mobility Framework for OMNeT++ 4.x
- **描述**: 移动性框架 OMNeT++ 4.0 移植版，已合并至 INET
- **关键字**: mobility
- **兼容**: omnetpp4（已弃用）
- **类型**: framework + model（已弃用）
- **精选**: 否
- **链接**: https://github.com/lidongming/mf-opp4

### 19. NesCT
- **描述**: 将 TinyOS 的 NesC 应用程序翻译为 C++ 仿真代码，使 OMNeT++ 可仿真 TinyOS 传感器网络
- **关键字**: wsn, wpan, tinyos, nesc, sensor, wireless
- **兼容**: omnetpp3, 4
- **类型**: model
- **精选**: 否

### 20. openDSME - IEEE 802.15.4 DSME
- **描述**: IEEE 802.15.4 DSME 确定性同步多信道扩展的开源可移植实现，集成至 INET
- **关键字**: wsn, sensor, wpan, dsme, 802.15.4, wireless
- **兼容**: omnetpp5 / inet4
- **类型**: model
- **精选**: 否
- **安装**: `opp_env install opendsme_allinone-latest`
- **链接**: https://github.com/openDSME/inet-dsme

### 21. OPS - Opportunistic Protocol Simulator
- **描述**: 机会网络仿真模型集，模块化架构可插入不同协议
- **关键字**: manet, mobility, opportunistic networking
- **兼容**: omnetpp5 / inet4
- **类型**: model
- **精选**: ★ 是
- **安装**: `opp_env install ops_allinone-latest`
- **链接**: https://github.com/ComNets-Bremen/OPS

### 22. OPSLite
- **描述**: 轻量级机会网络仿真器
- **关键字**: manet, mobility, opportunistic networking
- **兼容**: omnetpp5 / inet4
- **类型**: model
- **精选**: 否
- **安装**: `opp_env install opslite-latest`
- **链接**: https://github.com/ComNets-Bremen/OPSLite

### 23. PAWiS
- **描述**: 面向无线传感器网络优化的仿真框架，支持跨层/跨模块优化
- **关键字**: sensor, wsn, wireless, power
- **兼容**: omnetpp3
- **类型**: framework + model
- **精选**: 否

### 24. RIMFading - Radio Irregularity Model
- **描述**: INET 框架的 2D/3D 无线电不规则衰落模型实现
- **关键字**: wireless, pathloss, propagation
- **兼容**: omnetpp5 / inet3
- **类型**: model
- **精选**: 否
- **链接**: https://github.com/ComNets-Bremen/RIMFading

### 25. SWIMMobility
- **描述**: Small Worlds in Motion 移动性模型的 OMNeT++/INET 实现
- **关键字**: opportunistic networking, mobility, manet, ad hoc
- **兼容**: omnetpp4, 5 / inet3
- **类型**: model
- **精选**: 否
- **安装**: `opp_env install swim_allinone-latest`
- **链接**: https://github.com/ComNets-Bremen/SWIMMobility

### 26. SolarLEACH
- **描述**: LEACH 协议的太阳感知扩展，使太阳能节点在分簇中获更高优先级
- **关键字**: wireless, sensor, wsn, power
- **兼容**: omnetpp3
- **类型**: model
- **精选**: 否
- **安装**: `opp_env install solarleach-latest`

### 27. STEAM-Sim
- **描述**: 无线传感器网络的硬件/软件/网络协同仿真，支持 Contiki OS 代码标注运行时序
- **关键字**: wireless, mobility, sensor
- **兼容**: omnetpp3, 4
- **类型**: framework + model
- **精选**: 否

### 28. Stochastic Battery
- **描述**: 随机电池行为仿真模型，实现 Chiasserini 和 Rao 的随机电池模型
- **关键字**: power
- **兼容**: omnetpp5
- **类型**: model
- **精选**: 否
- **安装**: `opp_env install stochasticbattery-latest`
- **链接**: https://github.com/brandte/stochastic_battery

### 29. StreetlightSim
- **描述**: 基于无线传感器网络的路灯自主/自适应照明方案评估仿真
- **关键字**: iot, sumo, wsn, sensor, wireless, power, cosimulation
- **兼容**: omnetpp4
- **类型**: model
- **精选**: 否
- **安装**: `opp_env install streetlightsim-latest`

### 30. WiFi Direct for INET
- **描述**: 修改版 INET 3.5，增加 WiFi Direct 功能
- **关键字**: 802.11, wifi direct, wireless
- **兼容**: omnetpp5 / inet3
- **类型**: model
- **精选**: 否
- **安装**: `opp_env install wifidirect_allinone-latest`
- **链接**: https://github.com/ashahin1/inet

### 31. WiFi-MLO - Wi-Fi 7 Multi-link Operation
- **描述**: Wi-Fi 7 多链路操作（MLO）的开源实现，基于 INET 框架
- **关键字**: wireless, wifi7
- **兼容**: omnetpp6 / inet4
- **类型**: model
- **精选**: 否
- **链接**: https://github.com/tkn-tub/wifi-mlo-omnet

### 32. WSN Chaos Manager
- **描述**: 受混沌工程启发、面向 MANET 的自动化硬件故障注入工具
- **关键字**: wireless, sensor, wsn, power
- **兼容**: omnetpp5 / inet4
- **类型**: model
- **精选**: 否
- **安装**: `opp_env install chaosmanager-latest`
- **链接**: https://github.com/Agr-IoT/WSN-Chaos-Manager

### 33. oTWLAN - Tactical ad hoc networks
- **描述**: 战术自组网模型，支持 DSSS 无线电上的多级优先与抢占（MLPP）
- **关键字**: sensor, wsn, wpan, wireless, mlpp
- **兼容**: omnetpp3 / inet1
- **类型**: model
- **精选**: 否

### 34. crSimulator - Cognitive Radio Ad hoc Network
- **描述**: 认知无线电自组网仿真模型
- **关键字**: cognitive radio, wireless
- **兼容**: omnetpp4
- **类型**: model
- **精选**: 否
- **链接**: https://github.com/s2r2010/crSimulator

---

## 三、车载网络 (V2X/VANET) (13项)

### 1. Veins - Vehicles in Network Simulation
- **描述**: 开源车载通信仿真框架，基于 OMNeT++ 和 SUMO 协同仿真
- **关键字**: vanet, vehicular, mobility, ad hoc, ivc, 802.11p, 1609.4, wireless, cosimulation
- **兼容**: omnetpp3, 4, 5, 6
- **类型**: framework + model
- **精选**: ★ 是
- **安装**: `opp_env install veins-latest`
- **链接**: https://veins.car2x.org

### 2. Artery - V2X simulation framework for ETSI ITS-G5
- **描述**: 基于 ETSI ITS-G5 协议（GeoNetworking、BTP）的 V2X 仿真框架
- **关键字**: veins, vanetza, vanet, car2x, v2x, vehicular, routing, its-g5
- **兼容**: omnetpp5 / inet3
- **类型**: framework + model
- **精选**: ★ 是
- **安装**: `opp_env install artery_allinone-latest`
- **链接**: https://github.com/riebl/artery

### 3. Plexe - the Platooning Extension for Veins
- **描述**: Veins 的车辆编队扩展，提供逼真车辆动力学和多种巡航控制模型
- **关键字**: vanet, vehicular, mobility, ad hoc, ivc, 802.11p, 1609.4, wireless, cosimulation
- **兼容**: omnetpp4, 5, 6
- **类型**: framework + model
- **精选**: 否
- **链接**: http://plexe.car2x.org

### 4. VENTOS - VEhicular NeTwork Open Simulator
- **描述**: 开源集成 VANET 仿真器，用于研究车辆交通流、协同驾驶和 DSRC 通信
- **关键字**: vanet, vehicular, platoon, mobility, ad hoc, ivc, v2x, dsrc, wave, 802.11p, 1609.4, wireless, cosimulation
- **兼容**: omnetpp4, 5
- **类型**: framework + model
- **精选**: 否
- **链接**: https://github.com/ManiAm/VENTOS_Public

### 5. VANETProject
- **描述**: 基于 Veins 的 VANET 通信解决方案
- **关键字**: vanet, vehicular, ad hoc, aodv, gpsr, georouting
- **兼容**: omnetpp5 / veins
- **类型**: model
- **精选**: 否
- **链接**: https://github.com/chaotictoejam/VANETProject

### 6. OpenCV2X - Open Cellular Vehicle To Everything Mode 4
- **描述**: 3GPP C-V2X Rel 14 Mode 4 开源实现，基于 SimuLTE
- **关键字**: vanet, vehicular, mobility, ad hoc, v2x, wireless, cosimulation
- **兼容**: omnetpp5 / simulte, veins, artery
- **类型**: framework + model
- **精选**: 否
- **安装**: `opp_env install opencv2x_veins-latest`
- **链接**: https://github.com/brianmc95/OpenCV2X

### 7. Eclipse Mosaic (formerly VSimRTI)
- **描述**: 多尺度多领域协同仿真框架，用于评估联网自动驾驶出行方案
- **关键字**: mobility, v2x, vehicular, vanet, hla, rti, cosimulation
- **兼容**: omnetpp5
- **类型**: framework + model
- **精选**: 否
- **安装**: `opp_env install eclipse_mosaic_allinone-latest`
- **链接**: https://github.com/eclipse-mosaic/mosaic

### 8. LimoSim - Lightweight ICT-centric Vehicle Mobility Simulation
- **描述**: 轻量级 ICT 车辆移动性仿真，直接在 INET 中仿真车辆移动，无需外部交通仿真器
- **关键字**: vanet, vehicular, mobility
- **兼容**: omnetpp5 / inet3
- **类型**: model
- **精选**: 否
- **链接**: https://github.com/BenSliwa/LIMoSim

### 9. CrowNet - Crowds in Networks
- **描述**: 面向新型联网移动和智能交通系统的开源仿真框架，整合行人移动性建模
- **关键字**: crowd, mobility
- **兼容**: omnetpp5 / inet4
- **类型**: framework + model
- **精选**: 否
- **链接**: https://github.com/roVer-HM/crownet

### 10. space_Veins - Satellite supported vehicular networks
- **描述**: Veins 扩展，将卫星作为车载网络的额外通信伙伴，卫星移动基于 SGP4 轨道模型
- **关键字**: vanet, vehicular, mobility, ad hoc, ivc, 802.11p, 1609.4, aerospace, wireless, satellite
- **兼容**: omnetpp6 / veins
- **类型**: framework + model
- **精选**: 否
- **安装**: `opp_env install space_veins_allinone-latest`
- **链接**: https://github.com/veins/space_veins/

### 11. Veins VLC - Vehicular Visible Light Communication
- **描述**: Veins 扩展，增加车载可见光通信（V-VLC）信道模型
- **关键字**: vlc, visual light, vanet, vehicular, mobility, ad hoc, ivc, 802.11p, 1609.4, wireless
- **兼容**: omnetpp5 / veins
- **类型**: framework + model
- **精选**: 否
- **安装**: `opp_env install veins_vlc-latest`
- **链接**: https://github.com/veins/veins_vlc

### 12. VNS - Vehicular Networks Simulator
- **描述**: 完全整合移动性和网络组件的交通/网络协同仿真框架
- **关键字**: mobility, vehicular
- **兼容**: omnetpp4
- **类型**: framework + model
- **精选**: 否
- **链接**: https://github.com/enriquefynn/libvns

### 13. TBUS - Trace-based UMTS Simulation
- **描述**: 基于轨迹的 UMTS 仿真，支持 OMNeT++ 和 VSimRTI
- **关键字**: vanet, vehicular, mobility, ad hoc, wireless, sumo, cosimulation
- **兼容**: omnetpp4
- **类型**: framework + model
- **精选**: 否
- **链接**: https://github.com/hhucn/tbus-vsimrti

---

## 四、蜂窝/移动通信 (3项)

### 1. Simu5G
- **描述**: 5G NR 和 LTE/LTE-A 网络仿真器，支持 FDD/TDD 模式、D2D 和双连接
- **关键字**: 3gpp, wireless, gsm, voip, 5g
- **兼容**: omnetpp5, 6 / inet4
- **类型**: framework + model
- **精选**: ★ 是
- **安装**: `opp_env install simu5g-latest`
- **链接**: https://github.com/Unipisa/Simu5G

### 2. SimuLTE
- **描述**: LTE/LTE-A 网络用户面仿真工具（已弃用，由 Simu5G 取代）
- **关键字**: 3gpp, wireless, gsm, voip
- **兼容**: omnetpp4, 5, 6 / inet3, 4（已弃用）
- **类型**: framework + model
- **精选**: ★ 是
- **安装**: `opp_env install simulte-latest`
- **链接**: https://github.com/inet-framework/simulte

### 3. Numbat
- **描述**: 移动 WiMAX、IPv6 自动配置（DHCPv6）和移动性机制的实现
- **关键字**: 802.16e 2005, xmipv6, wimax, ipv6, dhcpv6, mobility
- **兼容**: omnetpp3, 4
- **类型**: framework + model
- **精选**: 否
- **链接**: https://github.com/tomaszmrugalski/numbat

### 4. Simple GSM simulation
- **描述**: 极简 GSM 网络仿真（文档仅匈牙利语）
- **关键字**: gsm
- **兼容**: omnetpp2
- **类型**: model
- **精选**: 否

---

## 五、数据中心/云计算/HPC (10项)

### 1. DCTrafficGen (DCTG)
- **描述**: 数据中心网络流量生成库，基于统计特征生成流量
- **关键字**: datacenter, cloud, traffic
- **兼容**: omnetpp5
- **类型**: framework + model
- **精选**: 否
- **安装**: `opp_env install dctrafficgen-latest`
- **链接**: https://github.com/Mellanox/DCTrafficGen

### 2. DataCenter
- **描述**: 轻量级分组级 FatTree 数据中心网络仿真器
- **关键字**: hpc, interconnection, fat tree
- **兼容**: omnetpp4 / inet2
- **类型**: model
- **精选**: 否
- **链接**: https://github.com/mhaitjema/DataCenter

### 3. CloudNetSim++
- **描述**: 分布式数据中心仿真工具包，支持架构、能耗模型和高速通信网络
- **关键字**: cloud, performance, data center
- **兼容**: omnetpp4 / inet2
- **类型**: model
- **精选**: 否

### 4. MimicNet
- **描述**: 基于机器学习的数据中心网络快速性能估计，用分组级仿真和 ML 加速大规模仿真
- **关键字**: data center, machine learning, mi, performance estimation
- **兼容**: omnetpp4 / inet2
- **类型**: model
- **精选**: 否
- **链接**: https://github.com/eniac/MimicNet

### 5. SIMCAN
- **描述**: 分布式架构和应用仿真平台，支持 MPI 应用仿真和大容量存储网络
- **关键字**: hpc, cloud, data center, storage network, mpi, performance
- **兼容**: omnetpp4 / inet2
- **类型**: framework + model
- **精选**: 否
- **安装**: `opp_env install simcan-latest`

### 6. SimSANs - Simulating Storage Area Networks
- **描述**: 数据中心存储网络设计仿真工具，支持 SCSI over FC 和 FCoE
- **关键字**: scsi, fcoe, san
- **兼容**: omnetpp4
- **类型**: model
- **精选**: 否

### 7. HECIOS - Parallel Virtual File System Simulator
- **描述**: 面向高性能计算 I/O 的可扩展仿真包
- **关键字**: mpi, ethernet
- **兼容**: omnetpp3, 4 / inet1
- **类型**: model
- **精选**: 否
- **链接**: https://github.com/bws/HECIOS

### 8. InfiniBand
- **描述**: InfiniBand 模型，支持 IB 流控制、多 VL 仲裁和线性转发表路由
- **关键字**: routing
- **兼容**: omnetpp3
- **类型**: model
- **精选**: 否

### 9. InfiniBand Flit Level Model
- **描述**: Mellanox 贡献的 InfiniBand 切片级数据通路仿真模型
- **关键字**: routing
- **兼容**: omnetpp4
- **类型**: model
- **精选**: 否

### 10. iCanCloud
- **描述**: 云计算系统仿真平台，预测应用在特定硬件上的成本/性能折中
- **关键字**: cloud, performance
- **兼容**: omnetpp4 / inet2
- **类型**: model
- **精选**: 否
- **安装**: `opp_env install icancloud-latest`

---

## 六、工业/实时网络/现场总线 (10项)

### 1. AFDX - Avionics Full-Duplex Switched Ethernet
- **描述**: 航空电子全双工交换以太网模型，支持 BAG 调控、令牌桶流量监管、VL 路由等
- **关键字**: afdx, ethernet, fieldbus, avionics
- **兼容**: omnetpp6
- **类型**: model
- **精选**: 否
- **安装**: `opp_env install afdx-latest`
- **链接**: https://github.com/badapplexx/AFDX

### 2. CAN (Controller Area Network) model
- **描述**: CAN 总线仿真模型，支持 CAN 消息路由器和 CAN-CAN 网关仿真
- **关键字**: canbus, fieldbus, vehicular, automotive
- **兼容**: omnetpp4 / inet2
- **类型**: model
- **精选**: 否
- **安装**: `opp_env install can_allinone-latest`

### 3. Core4INET - Real-Time Ethernet protocols for INET
- **描述**: INET 的实时以太网扩展，支持 TTEthernet (AS6802)、IEEE 802.1 AVB/TSN、VLAN（已弃用，推荐用 INET 4.4+ TSN 支持）
- **关键字**: vehicular, automotive, ethernet, as6802, ttethernet, 802.1q, avb, srp, tsn, vlan, 802.1p
- **兼容**: omnetpp4, 5 / inet3（已弃用）
- **类型**: model
- **精选**: ★ 是
- **安装**: `opp_env install core4inet-latest`
- **链接**: https://github.com/CoRE-RG/CoRE4INET

### 4. FiCo4OMNeT - Fieldbus Communication (CAN, FlexRay)
- **描述**: CAN 和 FlexRay 现场总线通信模型（已进入维护模式）
- **关键字**: vehicular, automotive, canbus, fieldbus, flexray
- **兼容**: omnetpp4, 5, 6
- **类型**: framework + model
- **精选**: ★ 是
- **安装**: `opp_env install fico4omnet-latest`
- **链接**: https://github.com/CoRE-RG/FiCo4OMNeT

### 5. FIELDBUS
- **描述**: 工业控制网络仿真框架，包含 Ethernet、ControlNet 和 DeviceNet 模型
- **关键字**: fieldbus, ethernet, controlnet, devicenet
- **兼容**: omnetpp3
- **类型**: model
- **精选**: 否

### 6. IEEE 802.1AS gPTP for Clock Synchronization
- **描述**: IEEE 802.1AS 精确时间协议（gPTP）仿真（已弃用，代码已合并至 INET）
- **关键字**: ethernet, lan, timing, gptp, 802.1as
- **兼容**: omnetpp5 / inet3（已弃用）
- **类型**: model
- **精选**: 否
- **安装**: `opp_env install gptp-latest`

### 7. NeSTiNg - Network Simulator for TSN
- **描述**: 时间敏感网络（TSN）仿真模型（已弃用，推荐用 INET 4.4+）
- **关键字**: vehicular, automotive, ethernet, 802.1q, avb, srp, tsn, vlan
- **兼容**: omnetpp5 / inet3, 4（已弃用）
- **类型**: model
- **精选**: 否
- **安装**: `opp_env install nesting-latest`
- **链接**: https://gitlab.com/ipvs/nesting

### 8. ProcessBusIec61850 - IEC61850 process bus communication
- **描述**: IEC 61850 过程总线通信（GOOSE 和 SV）模型，用于变电站智能电子设备
- **关键字**: electrical, grid, process, bus
- **兼容**: omnetpp5 / inet3
- **类型**: model
- **精选**: 否
- **安装**: `opp_env install processbus_allinone-latest`
- **链接**: https://github.com/hectordelahoz/ProcessBusIec61850/

### 9. libPTP - Precision Time Protocol (IEEE 1588)
- **描述**: 精确时间协议（PTP, IEEE 1588-2008）的 OMNeT++ 仿真库
- **关键字**: timing, libptp
- **兼容**: omnetpp4 / inet2
- **类型**: model
- **精选**: 否
- **安装**: `opp_env install libptp-latest`
- **链接**: https://github.com/w-wallner/libPTP

### 10. Precision Time Protocol for INET
- **描述**: INET 2.6 的 PTP 模块，作为 UDP 应用实现
- **关键字**: timing, ptp
- **兼容**: omnetpp4 / inet2
- **类型**: model
- **精选**: 否
- **链接**: https://github.com/martinlevesque/ptp-plusplus

---

## 七、光网络 (5项)

### 1. CAROBS - Car Optical Burst Switching simulator
- **描述**: 光突发交换（OBS）新范式仿真器，串联多突发以提高链路利用率
- **关键字**: optical, obs, sle, rwa
- **兼容**: omnetpp4
- **类型**: model
- **精选**: 否
- **链接**: https://github.com/lejmr/carobs

### 2. EPON - Ethernet Passive Optical Network
- **描述**: 1G EPON 基本实现，含 OLT/ONU 模块、MPCP 协议、802.1Q VLAN、DBA 轮询
- **关键字**: pon, passive, optical, network
- **兼容**: omnetpp4
- **类型**: model
- **精选**: 否

### 3. OBS Modules - Optical Burst Switching
- **描述**: OMNeT++ 光突发交换网络仿真模块集
- **关键字**: obs, optical
- **兼容**: omnetpp4 / inet2
- **类型**: model
- **精选**: 否
- **安装**: `opp_env install obs-latest`
- **链接**: https://github.com/mikelizal/OBSmodules

### 4. PhoenixSim - Photonic and Electronic Network Integration and Execution Simulator
- **描述**: 光子/电子互连网络设计和性能分析仿真环境
- **关键字**: photonic, hybrid, nobs, optical, network-on-chip
- **兼容**: omnetpp4
- **类型**: framework + model
- **精选**: 否
- **链接**: https://github.com/lebiednik/PhoenixSim

### 5. WDM Simulator
- **描述**: WDM 网络仿真器，基于 HORNET（混合光电环网）架构
- **关键字**: wdm, optical, hornet
- **兼容**: omnetpp3
- **类型**: model
- **精选**: 否

---

## 八、P2P/覆盖网络 (5项)

### 1. OverSim - Overlay Network Simulation Framework
- **描述**: 开源覆盖层和对等网络仿真框架，包含 Chord、Kademlia、Pastry、GIA 等协议
- **关键字**: p2p, chord, kademlia, pastry, gia
- **兼容**: omnetpp4, 5 / inet1, 3
- **类型**: framework + model
- **精选**: ★ 是
- **安装**: `opp_env install oversim-latest`
- **链接**: https://github.com/inet-framework/oversim

### 2. DenaCast
- **描述**: 开源 P2P 视频流框架，基于 OverSim
- **关键字**: p2p, overlay, multimedia, video, streaming
- **兼容**: omnetpp4 / inet2
- **类型**: model
- **精选**: 否
- **链接**: https://github.com/aarizaq/DenaCast

### 3. EbitSim
- **描述**: 增强型 BitTorrent 仿真，支持多并发 Swarm、多 Tracker、时间片处理模型
- **关键字**: bittorrent, p2p
- **兼容**: omnetpp4 / inet1
- **类型**: model
- **精选**: 否
- **链接**: https://github.com/pedromanoel/EbitSim

### 4. PITHOS - P2P MMVE OMNeT++/Oversim simulation
- **描述**: 面向 P2P 大规模多用户虚拟环境的可靠/安全/公平分布式存储系统
- **关键字**: p2p, overlay, mmve, distributed, storage
- **兼容**: omnetpp4 / oversim
- **类型**: model
- **精选**: 否
- **链接**: https://github.com/jsgilmore/pithos

### 5. RSPSIM - Reliable Server Pooling Simulation
- **描述**: IETF RSerPool 架构的开源仿真模型，轻量级服务器池化和会话故障转移框架
- **关键字**: rserpool, asap, enrp, calcapp
- **兼容**: omnetpp5, 6
- **类型**: framework + model
- **精选**: ★ 是
- **安装**: `opp_env install rspsim-latest`
- **链接**: https://github.com/dreibh/rspsim

---

## 九、内容中心/信息中心网络 (ICN/CCN) (4项)

### 1. ccnSim - Content Centric Networks Simulation
- **描述**: 高度可扩展的 ICN/CCN 块级仿真器，支持事件驱动、混合建模和并行仿真引擎
- **关键字**: web, caching
- **兼容**: omnetpp4
- **类型**: model
- **精选**: 否
- **链接**: https://github.com/TeamRossi/ccnSim-0.4

### 2. CDNSim - Content Distribution Networks Simulator
- **描述**: 内容分发网络仿真模型，含重定向策略、缓存策略、TCP/IP 等
- **关键字**: web, caching
- **兼容**: omnetpp3 / inet1
- **类型**: model
- **精选**: 否

### 3. inbaverSim - Content Centric Networks Simulation Framework
- **描述**: 内容中心网络仿真框架，遵循 RFC 8569/8609，支持多种节点类型（无线、DTN、有线、核心路由器、IoT 网关等）
- **关键字**: ccn, icn, ndn
- **兼容**: omnetpp6 / inet4
- **类型**: model
- **精选**: ★ 是
- **安装**: `opp_env install inbaversim-latest`
- **链接**: https://github.com/ComNets-Bremen/inbaverSim

### 4. NDNOMNeT - Named Data Networking framework
- **描述**: 面向 IoT 系统的命名数据网络（NDN）仿真扩展
- **关键字**: ndn, iot
- **兼容**: omnetpp5 / inet3
- **类型**: model
- **精选**: 否
- **安装**: `opp_env install ndnomnet-latest`
- **链接**: https://github.com/amar-ox/NDNOMNeT

---

## 十、SDN 与安全 (3项)

### 1. sEden Controller
- **描述**: SDN 控制器，通过 OpenFlow v1.0.0 消息控制交换机，可与真实 SDN 网络（Mininet）协作
- **关键字**: sdn, openflow
- **兼容**: omnetpp5 / inet4
- **类型**: model
- **精选**: 否
- **安装**: `opp_env install sedencontroller_allinone-latest`
- **链接**: https://github.com/swiru95/omnetpp_sdncontroller

### 2. NETA - NETwork Attacks Framework
- **描述**: 面向异构网络的攻击仿真框架，用于评估网络安全防御技术
- **关键字**: security, attack
- **兼容**: omnetpp4, 5 / inet2
- **类型**: model
- **精选**: 否
- **安装**: `opp_env install neta_allinone-latest`
- **链接**: https://github.com/robertomagan/neta_v1/releases

### 3. SEA++ - Simulating Security Attacks
- **描述**: 安全攻击仿真器，定量评估安全攻击影响，兼容传统和 SDN 架构
- **关键字**: security, attack, sdn
- **兼容**: omnetpp4 / inet2
- **类型**: model
- **精选**: 否
- **安装**: `opp_env install seapp-latest`
- **链接**: https://github.com/seapp/seapp_stable

---

## 十一、卫星/航空航天 (2项)

### 1. OS³ - Open Source Satellite Simulator
- **描述**: 基于卫星的通信仿真框架，可导入真实卫星轨道和气象数据
- **关键字**: aerospace, wireless, mobility, satellite
- **兼容**: omnetpp4, 5 / inet3
- **类型**: model
- **精选**: 否
- **安装**: `opp_env install os3-latest`
- **链接**: https://github.com/inet-framework/cni-os3

### 2. AVENS - Aerial VEhicle Network Simulator
- **描述**: OMNeT++/INET 与 XPlane 飞行仿真器的协同仿真，用于 UAV 通信
- **关键字**: mobility, vehicular, cosimulation
- **兼容**: omnetpp4 / inet3
- **类型**: model + framework
- **精选**: 否
- **链接**: https://github.com/lsecicmc/AVENS

---

## 十二、量子/新型架构 (2项)

### 1. QuISP - Quantum Internet Simulation Package
- **描述**: 量子中继器网络事件驱动仿真，面向量子互联网协议设计和大规模行为研究
- **关键字**: quantum, quantum-internet, quantum-computing
- **兼容**: omnetpp5
- **类型**: framework + model
- **精选**: 否
- **安装**: `opp_env install quisp-latest`
- **链接**: https://github.com/sfc-aqua/quisp

### 2. RINASim - Recursive InterNetwork Architecture Simulator
- **描述**: 递归互联网络架构（RINA）仿真框架，独立于 INET
- **关键字**: rina, network, architecture
- **兼容**: omnetpp4, 5
- **类型**: framework + model
- **精选**: ★ 是
- **安装**: `opp_env install rinasim-latest`
- **链接**: https://github.com/kvetak/RINA

---

## 十三、协同仿真/全系统仿真 (4项)

### 1. COSSIM - Cyber-Physical Systems Simulator Framework
- **描述**: 首个开源高性能全系统仿真器，联合 GEM5（处理器）、OMNeT++（网络）、McPAT（能耗）
- **关键字**: full system, simulator, hla, cosimulation
- **兼容**: omnetpp5 / inet3
- **类型**: model
- **精选**: 否
- **链接**: https://github.com/H2020-COSSIM

### 2. NoSSim - Network-of-Systems Simulator
- **描述**: 网络/系统协同仿真框架，结合宿主编译全系统仿真器和 OMNeT++ 网络仿真
- **关键字**: full system, os, iot, systemc, cosimulation, lwip
- **兼容**: omnetpp5 / inet3
- **类型**: framework + model
- **精选**: 否
- **链接**: https://github.com/SLAM-Lab/NoSSim

### 3. Cell Communication Signaling Project
- **描述**: 细胞间分子级通信信号研究项目，评估基于钙离子的扩散通信信道
- **关键字**: biology, cell, signaling
- **兼容**: omnetpp4
- **类型**: model
- **精选**: 否
- **安装**: `opp_env install cell-latest`
- **链接**: https://github.com/dhuertas/cell-signaling

### 4. NIST TESIM-OMNeT++
- **描述**: NIST 开发的无线网络与物理系统 CPS 协同仿真
- **关键字**: cps, wireless, 802.15.4, sensor, wsn, wpan, 802.11
- **兼容**: omnetpp4 / inet2
- **类型**: model
- **精选**: 否
- **链接**: https://github.com/usnistgov/tesim_omnetpp

---

## 十四、片上网络 (1项)

### 1. HNOCS - Network on Chip Simulation Framework
- **描述**: 开源片上网络仿真框架，支持虫孔交换、多种仲裁器和 VOQ
- **关键字**: network-on-chip
- **兼容**: omnetpp4, 5
- **类型**: framework + model
- **精选**: 否
- **安装**: `opp_env install hnocs-latest`
- **链接**: https://github.com/yanivbi/HNOCS

---

## 十五、工具与语言绑定 (17项)

### 1. SimProcTC - Simulation Processing Tool-Chain
- **描述**: 面向 OMNeT++ 的模型无关工具链，用于设置、并行运行、结果聚合和数据分析
- **关键字**: cluster computing, simulation campaign, batch run, statistics
- **兼容**: omnetpp3, 4, 5, 6
- **类型**: tool
- **精选**: ★ 是
- **安装**: `opp_env install simproctc-latest`
- **链接**: https://github.com/dreibh/simproctc

### 2. JResultWriter
- **描述**: Java 库，用于以 OMNeT++ 向量/标量文件格式记录仿真结果
- **关键字**: java, statistics
- **兼容**: omnetpp4
- **类型**: tool
- **精选**: 否

### 3. JSimpleModule
- **描述**: 用 Java 编写 OMNeT++ 简单模块的扩展，可混合 Java 和 C++ 模块
- **关键字**: java, language, binding
- **兼容**: omnetpp3, 4, 5
- **类型**: tool
- **精选**: 否
- **链接**: https://github.com/omnetpp/jsimplemodule

### 4. CSharpSimpleModule
- **描述**: 用 C# 编写 OMNeT++ 简单模块的扩展
- **关键字**: csharp, c#, language, binding
- **兼容**: omnetpp3
- **类型**: tool
- **精选**: 否

### 5. Java Extensions for OMNeT++
- **描述**: 基于 JSimpleModule 的增强版，用 Java 编写 OMNeT++ 仿真模块，提供 VM 镜像
- **关键字**: java, language, binding
- **兼容**: omnetpp5 / inet3
- **类型**: tool
- **精选**: 否
- **链接**: https://gitlab.amd.e-technik.uni-rostock.de/henning.puttnies/javaextensions4omnet

### 6. omnetpy - OMNeT++ meets Python
- **描述**: 用 Python 编写 OMNeT++ 简单模块的扩展，支持 Docker 容器
- **关键字**: python, language, binding
- **兼容**: omnetpp5
- **类型**: tool
- **精选**: 否
- **链接**: https://github.com/mmodenesi/omnetpy/

### 7. LRE-OMNeT++ - Limited Relative Error
- **描述**: 将有限相对误差（LRE）算法集成到 OMNeT++，用于统计置信度评估
- **关键字**: statistics, confidence, lre
- **兼容**: omnetpp5
- **类型**: model
- **精选**: 否
- **安装**: `opp_env install lre_omnet-latest`
- **链接**: https://github.com/ComNetsHH/LRE-OMNeT

### 8. Machine Learning in OMNeT++
- **描述**: 在 OMNeT++ 中使用机器学习框架的资料和示例
- **关键字**: tool, machine learning
- **兼容**: omnetpp5
- **类型**: tool
- **精选**: 否
- **链接**: https://github.com/ComNetsHH/omnetpp-ml

### 9. Neurogenesis - Distributed OMNeT++ Simulation Toolchain
- **描述**: 将大量 OMNeT++ 仿真作业分发到多个集群计算机的框架，基于 mpi4py
- **关键字**: cluster computing, simulation campaign, batch run, mpi
- **兼容**: omnetpp5
- **类型**: tool
- **精选**: 否
- **链接**: https://github.com/juliusf/Neurogenesis

### 10. OMNeT++ export for BRITE 2.1
- **描述**: 为 BRITE 拓扑生成器添加 NED 导出功能
- **关键字**: brite, topology generation
- **兼容**: omnetpp4 / inet1
- **类型**: tool
- **精选**: 否

### 11. OMPCM - Palladio Component Model for OMNeT++
- **描述**: 将 Palladio 软件架构模型转换为 OMNeT++ 网络定义文件，用于性能/可扩展性分析
- **关键字**: uml, software architecture simulation, performance
- **兼容**: omnetpp4 / inet2
- **类型**: tool
- **精选**: 否

### 12. R plugin for reading OMNeT++ result files
- **描述**: R 语言插件，加载 OMNeT++ 结果文件
- **关键字**: statistics
- **兼容**: omnetpp4
- **类型**: tool
- **精选**: 否
- **链接**: https://github.com/omnetpp/omnetpp-resultfiles

### 13. SimDistribution
- **描述**: OMNeT++ 3.x 仿真的分布式运行 GUI 管理工具
- **关键字**: cluster computing, simulation campaign, batch run
- **兼容**: omnetpp3
- **类型**: tool
- **精选**: 否

### 14. U2Q - UML to Queueing Network
- **描述**: 从 UML 模型生成排队网络性能模型的工具
- **关键字**: uml, queueing, performance
- **兼容**: omnetpp4
- **类型**: tool
- **精选**: 否

### 15. Veins Gym
- **描述**: 将 Veins 仿真导出为 OpenAI Gym 接口，用于强化学习算法
- **关键字**: tool, ml, machine learning
- **兼容**: omnetpp5, 6
- **类型**: tool
- **精选**: 否
- **链接**: https://github.com/tkn-tub/veins-gym

### 16. WAMPInterface - Live Monitoring and Remote Control
- **描述**: 基于 Web 技术的 OMNeT++ 仿真实时监控和远程控制接口
- **关键字**: web, interface, gui, live, monitoring
- **兼容**: omnetpp5
- **类型**: tool
- **精选**: 否
- **链接**: https://github.com/WAMPInterfaceForOmnetpp

### 17. Generic OMNeT++ Utilities
- **描述**: OMNeT++ 工具集：Callable、Channels、DynamicSignals、InitBase、ParameterParser
- **关键字**: utility, library
- **兼容**: omnetpp4
- **类型**: model
- **精选**: 否
- **链接**: https://github.com/w-wallner/OMNeT_Utils

### 18. oProbe
- **描述**: 为 OMNeT++ 提供可控随机采样技术的统计探头模块，确保结果置信度和相关性控制
- **关键字**: statistics
- **兼容**: omnetpp3
- **类型**: tool
- **精选**: 否

### 19. Queues - Queueing library and tutorial
- **描述**: 排队网络教程和基础排队库
- **关键字**: queue
- **兼容**: omnetpp3
- **类型**: model
- **精选**: 否
- **链接**: https://github.com/ndvanforeest/omnet_queues

### 20. GT-ITM Topologies for OMNeT++ and OverSim
- **描述**: 将 Georgia Tech Internet Topology Model (GT-ITM) 拓扑转换为 OverSim InetUnderlay 格式
- **关键字**: (无明确关键字)
- **兼容**: omnetpp4 / inet1
- **类型**: tool
- **精选**: 否

---

## 十六、其他/杂项 (6项)

### 1. GrADyS-SIM - Simulations from the GrADyS project
- **描述**: 面向 UAV 蜂群和传感器协调策略的仿真框架
- **关键字**: sensor network, uav, autonomous, swarm
- **兼容**: omnetpp5 / inet4
- **类型**: framework + model
- **精选**: 否
- **安装**: `opp_env install gradys-latest`
- **链接**: https://github.com/brunoolivieri/gradys-simulations

### 2. File System Simulation
- **描述**: 文件系统组件仿真（OMNeT++ 最早期的模型之一）
- **关键字**: filesystem, performance
- **兼容**: omnetpp2
- **类型**: model
- **精选**: 否

### 3. SCSI Bus
- **描述**: SCSI 总线模型
- **关键字**: (无)
- **兼容**: omnetpp2
- **类型**: model
- **精选**: 否

### 4. Personal Communication Services (PCS)
- **描述**: 个人通信服务模型，实现 Lin & Fishwick 1995 论文中的异步并行离散事件仿真
- **关键字**: wireless
- **兼容**: omnetpp2
- **类型**: model
- **精选**: 否

### 5. VideoInterface
- **描述**: 视频接口代码（源自 trace.eas.asu.edu），已更新至 OMNeT++ 3.0
- **关键字**: video, multimedia
- **兼容**: omnetpp3
- **类型**: model
- **精选**: 否

### 6. Google Earth Demo
- **描述**: 用 Google Earth 可视化仿真（已弃用，新版用 OSG/osgEarth 替代）
- **关键字**: 3d, visualization, terrain, mobility
- **兼容**: omnetpp4（已弃用）
- **类型**: model
- **精选**: 否

### 7. SelfSimMGI
- **描述**: 自相似流量模型 "M/G/∞ Input" 的修改版，用于分析 WAN 流量特征
- **关键字**: traffic generation
- **兼容**: omnetpp4
- **类型**: model
- **精选**: 否

---

## 十七、电力/能源系统 (1项)

### 1. ProcessBusIec61850
（见第六类）

---

## 统计

| 分类 | 数量 |
|------|------|
| 互联网协议与路由 | 27 |
| 无线/传感器/物联网 | 34 |
| 车载网络 | 13 |
| 蜂窝/移动通信 | 4 |
| 数据中心/云计算 | 10 |
| 工业/实时网络 | 10 |
| 光网络 | 5 |
| P2P/覆盖网络 | 5 |
| 内容中心/ICN | 4 |
| SDN 与安全 | 3 |
| 卫星/航空航天 | 2 |
| 量子/新型架构 | 2 |
| 协同仿真 | 4 |
| 片上网络 | 1 |
| 工具与语言绑定 | 20 |
| 其他 | 7 |
| **总计** | **~152** |

---

> 注：部分项目横跨多个类别，上述分组以其主要应用领域为准。"已合并"表示代码已合入 INET 框架；"已弃用"表示不再维护；★ 标记为精选项目。