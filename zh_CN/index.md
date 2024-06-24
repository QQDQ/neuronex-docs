# 产品概览

NeuronEX 是一款工业边缘网关软件，提供设备数据采集和边缘智能分析服务，主要部署在工业现场，实现工业设备通信及工业总线协议采集、工业系统数据集成、边端数据过滤分析及AI算法集成，以及工业互联网平台对接集成等功能，为工业场景提供低延迟的数据接入管理及智能分析服务，帮助用户快速洞悉业务趋势，提升运营效率和可持续性。


## 产品优势

- 丰富的协议接入

    丰富的协议插件满足工业各类场景下，PLC、CNC、机器人、Scada以及智能仪表等设备数据的实时采集及统一接入。内置多种插件模块，例如 Modbus，OPC UA，Ethernet/IP，IEC104，BACnet，Siemens，Mitsubishi 等。这些插件某块被广泛应用于楼宇自动化、数控机床、机器人、电力以及各种 PLC 通信中。

- 低延迟数据处理

    专为工业现场设计，提供低延迟的数据接入和处理，能够更快速地将数据在多系统间传递，实现实时监控和决策。

- 轻量及灵活部署

    NeuronEX具有轻量化、低内存占用，支持多种CPU架构部署，并且支持 Docker、Kubernetes容器化部署。

- 完整的数据分析能力

    内置数据抽取、转换、过滤、排序、分组、聚合、连接等功能，内置 160+ 各类函数，覆盖数学运算、字符串处理、聚合运算和哈希运算等，通过强大的流式计算分析能力，实现数据过滤清洗、数据标准化、分析监测及实时报警。

- AI/ML分析

    支持用户自定义函数扩展和 AI 算法集成，提供智能数据分析能力。

- 平台集成

    通过对接 MQTT、SparkplugB、HTTP 等方式，将数据集成到本地数据中心、工业互联网平台或云服务中。

## 功能一览

| <div style="width:40pt">功能</div> | 描述     | <div style="width:80pt">功能清单</div>   |
| ---------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 数据采集                           | NeuronEX 支持数十种工业协议的一站式设备连接、数据采集、设备反控、MQTT 协议转换、南向数采监控，为工业设备赋予工业 4.0 时代关键的互联互通能力。| [创建南向驱动](./configuration/south-devices/south-devices.md)<br /><br />[连接南向设备](./configuration/south-devices/south-devices.md) <br /><br />[南向数采监控](./admin/monitoring.md)|
| 数据传递                           | 完成设备数据的采集后，NeuronEX 支持用户通过北向应用将数据转发到云平台或外部处理引擎 | [创建北向应用](./configuration/north-apps/north-apps.md)<br /><br />[订阅南向数据](./configuration/subscription.md) |
| 数据处理                         | NeuronEX 集成了边缘数据处理引擎，提供低延迟的数据处理分析，能够更快速地将数据在多系统间传递，结合 AI/ML 算法，可以实现智能决策与控制。此外，边缘端分析还可以对数据做预处理和边缘计算，减少云边通讯负载及后端存储压力。 | [数据源](./streaming-processing/source.md)<br /><br />[规则](./streaming-processing/rules.md)<br /><br />[Sink 连接](./streaming-processing/sink/sink.md)<br /><br />[扩展功能](./streaming-processing/extension.md) |
| 系统管理                           | NeuronEX提供了一站式的运维与管理平台，您可通过 Web 页面进行系统配置管理、日志下载、查看系统信息等操作。 | [日志管理](./admin/log-management.md)<br /><br />[数据统计](./admin/data-statistics.md)<br /><br />[配置管理](./admin/conf-management.md) |
