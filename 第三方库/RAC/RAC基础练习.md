# 基础练习部分代码1

先练习代码，之后总结：

```objc
	RACSignal *singal = self.userNameTextField.rac_textSignal;
	
	//map：转换（映射成return的类型）
	singal = [singal map:^id(NSString *value) {
		return @(value.length);
	}];
	
	
	//filter：过滤 (return YES 的值才会传递)
	RACSignal *enableSingal = [singal filter:^BOOL(NSNumber *value) {
		return value.integerValue >= 3;
	}];
	
	
	//skip： 跳过一个信号
	[[enableSingal skip:1] subscribeNext:^(id x) {
		NSLog(@"rac:%@",x);
	}];
	
	RACSignal *colorSiganl  = [enableSingal map:^(NSNumber *value) {
		return value.integerValue > 8 ? [UIColor greenColor] : [UIColor yellowColor];
	}];
	
	//RAC宏直接把信号输出的值，设置到对象的属性上
	RAC(self.userNameTextField,backgroundColor) = colorSiganl;
	
	RACSignal *passwordSignal = [[self.passwordTextField.rac_textSignal filter:^BOOL(NSString *value) {
		return value.length > 3;
	}] map:^id(NSString *value) {
		return @(value.length);
	}];
	
	//combineLatest(聚合信号) reduce（每当聚合的信号发送时，都会取最新的值，调用reduce） -> 聚合信号的值
	RAC(self.loginButton,backgroundColor) = [[RACSignal combineLatest:@[enableSingal, passwordSignal] reduce:^id(NSNumber *u, NSNumber *p){
		NSLog(@"来信号啦");
		return @(u.integerValue == 5 && p.integerValue == 6);
	}] map:^id(NSNumber *value) {
		return value.boolValue ? [UIColor greenColor] : [UIColor grayColor];
	}];
	
	
	
	RACSignal *loginsignal = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
		
		dispatch_async(dispatch_get_global_queue(0, 0), ^{
			sleep(2);
			[subscriber sendNext:@(11)];
		});
		return nil;
	}];
	
	
	//在信号 sendNext之前 先做 block里的事
	[[[[[self.loginButton  rac_signalForControlEvents:UIControlEventTouchUpInside] doNext:^(id x) {
		self.view.backgroundColor = [UIColor redColor];
		self.loginButton.userInteractionEnabled = NO;
	}]
	//flattenMap 将信号 映射成 其他信号
    flattenMap:^RACStream *(id value) {
		return loginsignal;
	}]
	  // deliverOn : 切换线程
    deliverOn:[RACScheduler mainThreadScheduler]]
	subscribeNext:^(NSNumber *x) {
		if(x.integerValue == 11){
			NSLog(@"登录成功");
			self.view.backgroundColor = [UIColor whiteColor];
			self.loginButton.userInteractionEnabled = YES;
		}
	}
	 completed:^{
		//测试发现这里不会执行，实际上是flattenMap之前信号的订阅
	}];
	
```

