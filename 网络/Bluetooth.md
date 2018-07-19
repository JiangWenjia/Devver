# 资料
[简书蓝牙基本开发](https://www.jianshu.com/p/38a4c6451d93)


# 用语

> 中心设备(Central) : 用来扫描周围蓝牙设备，比如手机扫描周围的蓝牙设备，比如手环，并连接手环，这时候手机就是中心设备

>外设（Peripheral）：被扫描的设备，比如智能手环

>  peripheral 英 [pəˈrɪfərəl]: n.外部设备 central：英 [ˈsentrəl] adj.
中央的，中心的




--

> 服务（Service）：外设广播和运行的时候，会有服务，理解为功能模块，中心设备可以读取服务，可以有多个服务
特征(Characteristic) ： 服务中的单位，特种会有有一个value（一般读写就是value）
UUID：用于区分不同的服务和特征

# CoreBluetooth



