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


## 中心设备例子

```objc
#import "ViewController.h"
#import <CoreBluetooth/CoreBluetooth.h>


@interface ViewController () <CBCentralManagerDelegate, CBPeripheralDelegate>
@property (nonatomic, strong) CBCentralManager *centeralManager;
@property (nonatomic, strong) CBPeripheral *peripheral;
@property (nonatomic, strong) CBCharacteristic *writeCharacteristic;
@end

@implementation ViewController

- (void)viewDidLoad {
	[super viewDidLoad];
	self.centeralManager = [[CBCentralManager alloc] initWithDelegate:self queue:dispatch_get_main_queue()];
}

#pragma CBCentralManagerDelegate

/*
 1.生成CBCentralManager并设置代理（表示手机的蓝牙设备）
 2.蓝牙设备可用的时候，开始连接外设
 3.连接外设成功后设置外设代理，开始去寻找外设的服务
 注意：对应的队列，是代理回调的队列
 相关对象：
 CBCentralManager : 表示蓝牙设备
 CBPeripheral：     代表一个外设
 */

//蓝牙设备的状态更新了
- (void)centralManagerDidUpdateState:(CBCentralManager *)central {
	switch (central.state) {
		case CBManagerStatePoweredOn:
		{
			//蓝牙可用
			//根据UUID扫描外设
//			[self.centeralManager scanForPeripheralsWithServices:@[[CBUUID UUIDWithString:@"CDD1"]] options:nil];
			//不设置扫描所有的外设
			[self.centeralManager scanForPeripheralsWithServices:nil options:nil];
			break;
		}
		case CBManagerStateUnsupported:{
			//设备不支持蓝牙
			break;
		}
		case CBManagerStatePoweredOff:
		{
			//设备已经关闭
			break;
		}
		default:{
			break;
		}
	}
}

//发现外设
- (void)centralManager:(CBCentralManager *)central didDiscoverPeripheral:(CBPeripheral *)peripheral advertisementData:(NSDictionary<NSString *,id> *)advertisementData RSSI:(NSNumber *)RSSI {
	
	//连接设备
	[central connectPeripheral:peripheral options:nil];
}


//连接上外设成功
- (void)centralManager:(CBCentralManager *)central didConnectPeripheral:(CBPeripheral *)peripheral {
	//CBPeripheral 代表外设
	[self.centeralManager stopScan];//停止扫描
	//设置外设的代理
	peripheral.delegate = self;
	
//	[peripheral discoverServices:@[[CBUUID UUIDWithString:@"CDD1"]]];
	[peripheral discoverServices:nil];
}

//连接外设失败
- (void)centralManager:(CBCentralManager *)central didFailToConnectPeripheral:(CBPeripheral *)peripheral error:(NSError *)error {
	
}

//与外设断开连接
- (void)centralManager:(CBCentralManager *)central didDisconnectPeripheral:(CBPeripheral *)peripheral error:(NSError *)error {
	
}

#pragma CBPeripheralDelegate


/**
 发现 外设设备的服务回调

 @param peripheral 外设
 @param error 错误
 
 peripheral.services 外设的提供的蓝牙服务
 
 */
- (void)peripheral:(CBPeripheral *)peripheral didDiscoverServices:(NSError *)error {
	
	//查看外设所提供的服务
	[peripheral.services enumerateObjectsUsingBlock:^(CBService * _Nonnull service, NSUInteger idx, BOOL * _Nonnull stop) {
		//CBService 对应的服务
		[peripheral discoverCharacteristics:nil forService:service];
	}];
	
}



/**
 发现服务的特征

 @param peripheral 外设
 @param service 服务
 @param error 错误
 */
- (void)peripheral:(CBPeripheral *)peripheral didDiscoverCharacteristicsForService:(nonnull CBService *)service error:(nullable NSError *)error {
	
	//遍历所有的特征
	[service.characteristics enumerateObjectsUsingBlock:^(CBCharacteristic * _Nonnull characteristic, NSUInteger idx, BOOL * _Nonnull stop) {
		
		//不同的特征作用不同， 有些是读取数据的，有些是写入数据的
		
		//读取特种的数据比如 读取温度特征的值
		[peripheral readValueForCharacteristic:characteristic];
		
		if(characteristic.properties & CBCharacteristicPropertyWrite || characteristic.properties & CBCharacteristicPropertyWriteWithoutResponse) {
			self.writeCharacteristic = characteristic;
		}
		
		//订阅特征的变化
		[peripheral setNotifyValue:YES forCharacteristic:characteristic];
	}];
}


/**
 订阅的特征变化的状态发生了改变

 @param peripheral 外设
 @param characteristic 特征
 @param error 错误
 */
- (void)peripheral:(CBPeripheral *)peripheral didUpdateNotificationStateForCharacteristic:(CBCharacteristic *)characteristic error:(NSError *)error {
	
	if(error) { //订阅错误
		
		return;
	}
	
	if(characteristic.isNotifying) { //订阅成功
		
	}else { //订阅失败
		
	}
}


/**
 特征的值发生了更新

 @param peripheral 外设
 @param characteristic 特征
 @param error 错误
 */
- (void)peripheral:(CBPeripheral *)peripheral didUpdateValueForCharacteristic:(CBCharacteristic *)characteristic error:(NSError *)error {
	
	NSData *data = characteristic.value;
	NSString *dataString = [[NSString alloc] initWithData:data encoding:NSUTF8StringEncoding];
	NSLog(@"%@",dataString);
}

/**
 向外设写入数据
 */
- (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event {
	NSData *data = [@"0x00010" dataUsingEncoding:NSUTF8StringEncoding];
	//向外设写入数据 CBCharacteristicWriteWithResponse 会有响应
	[self.peripheral writeValue:data forCharacteristic:self.writeCharacteristic type:CBCharacteristicWriteWithResponse];
}


/**
 向外设写入数据之后会有回调

 @param peripheral 外设
 @param characteristic 特征
 @param error 错误
 */
- (void)peripheral:(CBPeripheral *)peripheral didWriteValueForCharacteristic:(CBCharacteristic *)characteristic error:(NSError *)error {
	if(error) {
		//写入时，发生错误
		return;
	}
	
	//写入成功
}

```


