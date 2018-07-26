# 描述
- 支持iOS8及以上版本
- 内置手势
- 官方称滚动刷新帧率60fps
- 与Safari相同的javaScript引擎
- 占用内存更少


# delegate

- WKUIDelegate ： 处理js脚本，确认框，警告框
- WKNavigationDelegate ： 加在处理， 跳转

先做个记录
```objc
@import WebKit;


@interface ViewController () <WKUIDelegate, WKNavigationDelegate,WKScriptMessageHandler> {
    
}
@property (weak, nonatomic) IBOutlet WKWebView *webView;
@property (weak, nonatomic) IBOutlet UIProgressView *progressView;

@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
   
    //delegate
    self.webView.UIDelegate = self;
    self.webView.navigationDelegate = self;
  
    [self.webView loadRequest:[NSURLRequest requestWithURL:[NSURL URLWithString:@"https://www.baidu.com"]]];
    
    __weak ViewController  *weakSelf = self;
    
    //约定对象 会循环引用，应该再起一个对象，然后对象回调
    [self.webView.configuration.userContentController addScriptMessageHandler:weakSelf name:@"myweb"];
    //js调用： window.webkit.messageHandlers.myweb.postMessage({body: 'hello world!'});
    
    
    //返回手势
    self.webView.allowsBackForwardNavigationGestures = YES;
    //加载进度
    @weakify(self)
    [RACObserve(self.webView, estimatedProgress) subscribeNext:^(NSNumber *x) {
        @strongify(self)
        self.progressView.progress = x.floatValue;
        self.progressView.hidden = x.floatValue == 1.0f;
    }];
    
    //标题
    [RACObserve(self.webView, title) subscribeNext:^(NSString *x) {
        @strongify(self)
        self.title = x;
    }];
}

- (void)dealloc {
     [self.webView.configuration.userContentController removeScriptMessageHandlerForName:@"myweb"];
}


- (void)userContentController:(WKUserContentController *)userContentController didReceiveScriptMessage:(WKScriptMessage *)message {
    NSLog(@"%@", message);
}



#pragma mark - WKNavigationDelegate


//在发起请求之前，请求是否继续执行
- (void)webView:(WKWebView *)webView decidePolicyForNavigationAction:(WKNavigationAction *)navigationAction decisionHandler:(void (^)(WKNavigationActionPolicy))decisionHandler {
    NSLog(@"1:%@",navigationAction.request.URL.absoluteString);
    decisionHandler(WKNavigationActionPolicyAllow);
}


- (void)webView:(WKWebView *)webView didStartProvisionalNavigation:(WKNavigation *)navigation {
    NSLog(@"页面开始加载");
}

//收到响应后
- (void)webView:(WKWebView *)webView decidePolicyForNavigationResponse:(WKNavigationResponse *)navigationResponse decisionHandler:(void (^)(WKNavigationResponsePolicy))decisionHandler {
    //跳转的路径
    NSLog(@"2:%@", navigationResponse.response.URL.absoluteString);
    
    //允许跳转
    decisionHandler(WKNavigationResponsePolicyAllow);
}

- (void)webView:(WKWebView *)webView didCommitNavigation:(WKNavigation *)navigation {
    NSLog(@"页面开始提交");
}

- (void)webView:(WKWebView *)webView didFinishNavigation:(WKNavigation *)navigation {
    NSLog(@"页面完成加载");
}


- (void)webView:(WKWebView *)webView didFailProvisionalNavigation:(WKNavigation *)navigation withError:(NSError *)error {
    NSLog(@"页面完成失败");
}

- (void)webView:(WKWebView *)webView didReceiveServerRedirectForProvisionalNavigation:(WKNavigation *)navigation {
    NSLog(@"页面请求跳转");
}






#pragma mark WKUIDelegate 不确定作用

- (WKWebView *)webView:(WKWebView *)webView createWebViewWithConfiguration:(WKWebViewConfiguration *)configuration forNavigationAction:(WKNavigationAction *)navigationAction windowFeatures:(WKWindowFeatures *)windowFeatures {
    return [WKWebView new];
}



/**
 弹出输入框

 @param webView webView
 @param prompt 提示信息
 @param defaultText 默认提示文本
 @param frame 窗口
 @param completionHandler 回调JS，输入内容
 */
- (void)webView:(WKWebView *)webView runJavaScriptTextInputPanelWithPrompt:(NSString *)prompt defaultText:(NSString *)defaultText initiatedByFrame:(WKFrameInfo *)frame completionHandler:(void (^)(NSString * _Nullable))completionHandler {
    NSLog(@"输入框");
    
    
    UIAlertController *alert = [UIAlertController alertControllerWithTitle:@"请输入" message:prompt preferredStyle:(UIAlertControllerStyleAlert)];
    
    
    [alert addTextFieldWithConfigurationHandler:^(UITextField * _Nonnull textField) {
        textField.placeholder = @"请输入";
    }];
    
    UIAlertAction *ok = [UIAlertAction actionWithTitle:@"确定" style:(UIAlertActionStyleDefault) handler:^(UIAlertAction * _Nonnull action) {
        
        UITextField *tf = [alert.textFields firstObject];
        
        completionHandler(tf.text);
    }];
    
    UIAlertAction *cancel = [UIAlertAction actionWithTitle:@"取消" style:(UIAlertActionStyleCancel) handler:^(UIAlertAction * _Nonnull action) {
        completionHandler(defaultText);
    }];
    
    [alert addAction:ok];
    [alert addAction:cancel];
    [self presentViewController:alert animated:YES completion:nil];
}


/**
 webView 弹出警告框

 @param webView webView
 @param message 信息
 @param frame 区分窗口
 @param completionHandler 警告框消失的时候，回调js
 */
- (void)webView:(WKWebView *)webView runJavaScriptAlertPanelWithMessage:(NSString *)message initiatedByFrame:(WKFrameInfo *)frame completionHandler:(void (^)(void))completionHandler {
     NSLog(@"警告框");
    UIAlertController *alert = [UIAlertController alertControllerWithTitle:@"警告" message:message preferredStyle:(UIAlertControllerStyleAlert)];
    UIAlertAction *ok = [UIAlertAction actionWithTitle:@"我知道了" style:(UIAlertActionStyleDefault) handler:^(UIAlertAction * _Nonnull action) {
        completionHandler();
    }];
    
    [alert addAction:ok];
    [self presentViewController:alert animated:YES completion:nil];
}


/**
 提示框

 @param webView webView
 @param message 信息
 @param frame 窗口
 @param completionHandler 回掉（YES/NO）
 */
- (void)webView:(WKWebView *)webView runJavaScriptConfirmPanelWithMessage:(NSString *)message initiatedByFrame:(WKFrameInfo *)frame completionHandler:(void (^)(BOOL))completionHandler {
    NSLog(@"确认框");
    completionHandler(YES);
}

```