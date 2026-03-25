# 하이브리드 광고 연동 가이드

## 사전 조건
- iOS 15.0 이상
- AdFitSDK가 Swift Package Manager(SPM)를 통해 프로젝트에 설치되어 있어야 합니다.
- Web Page에서 AdFit Web SDK를 연동한 사용자를 대상으로 합니다.

---

## UIKit 연동

### 1. WKWebView 설정

웹뷰 내에서의 동영상 자동 재생을 위해 아래의 옵션을 설정합니다.

#### Swift
```swift
let configuration = WKWebViewConfiguration()
configuration.mediaTypesRequiringUserActionForPlayback = []
configuration.allowsInlineMediaPlayback = true
```

#### Objective-C
```objc
WKWebViewConfiguration *configuration = [[WKWebViewConfiguration alloc] init];
configuration.mediaTypesRequiringUserActionForPlayback = [NSArray array];
configuration.allowsInlineMediaPlayback = YES;
```

### 2. WKWebView 등록하기

AdFit Hybrid SDK 연동을 위해, 사용 중인 WKWebView를 등록합니다.

#### Swift
```swift
import UIKit
import AdFitSDK

class MyViewController: UIViewController {
    private var webView: WKWebView!

    override func viewDidLoad() {
        super.viewDidLoad()

        let configuration = WKWebViewConfiguration()
        configuration.mediaTypesRequiringUserActionForPlayback = []
        configuration.allowsInlineMediaPlayback = true

        webView = WKWebView(frame: .zero, configuration: configuration)
        webView.translatesAutoresizingMaskIntoConstraints = false
        view.addSubview(webView)

        NSLayoutConstraint.activate([
            webView.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor),
            webView.leadingAnchor.constraint(equalTo: view.safeAreaLayoutGuide.leadingAnchor),
            webView.trailingAnchor.constraint(equalTo: view.safeAreaLayoutGuide.trailingAnchor),
            webView.bottomAnchor.constraint(equalTo: view.safeAreaLayoutGuide.bottomAnchor)
        ])

        AdFit.register(webView: webView)

        // 웹 페이지 로드
        if let url = URL(string: "{webPageURL}") {
            webView.load(URLRequest(url: url))
        }
    }

    deinit {
        AdFit.unRegister(webView: webView)
    }
}
```

#### Objective-C
```objc
#import "MyViewController.h"
#import <AdFit/AdFit-Swift.h>

@interface MyViewController ()
@property (nonatomic, strong) WKWebView *webView;
@end

@implementation MyViewController

- (void)viewDidLoad {
    [super viewDidLoad];

    WKWebViewConfiguration *configuration = [[WKWebViewConfiguration alloc] init];
    configuration.mediaTypesRequiringUserActionForPlayback = [NSArray array];
    configuration.allowsInlineMediaPlayback = YES;

    self.webView = [[WKWebView alloc] initWithFrame:CGRectZero configuration:configuration];
    self.webView.translatesAutoresizingMaskIntoConstraints = NO;
    [self.view addSubview:self.webView];

    [NSLayoutConstraint activateConstraints:@[
        [self.webView.topAnchor constraintEqualToAnchor:self.view.safeAreaLayoutGuide.topAnchor],
        [self.webView.leadingAnchor constraintEqualToAnchor:self.view.safeAreaLayoutGuide.leadingAnchor],
        [self.webView.trailingAnchor constraintEqualToAnchor:self.view.safeAreaLayoutGuide.trailingAnchor],
        [self.webView.bottomAnchor constraintEqualToAnchor:self.view.safeAreaLayoutGuide.bottomAnchor]
    ]];

    [AdFit registerWithWebView:self.webView];

    // 웹 페이지 로드
    NSURL *url = [NSURL URLWithString:@"{webPageURL}"];
    [self.webView loadRequest:[NSURLRequest requestWithURL:url]];
}

- (void)dealloc {
    [AdFit unRegisterWithWebView:self.webView];
}

@end
```

### 3. 웹 페이지에 광고 스크립트 추가

설정을 마친 WKWebView의 웹 페이지 내에서 AdFit Web SDK를 사용하여 광고를 송출합니다.

```html
<ins class="kakao_ad_area" style="display:none;"
data-ad-unit="{adUnitId}"></ins>
<script type="text/javascript" src="//t1.kakaocdn.net/kas/static/ba.min.js" async></script>
```

자세한 사항은 AdFit Web SDK 가이드 문서를 참고하세요.

### 4. WKWebView 등록 해제하기

메모리에서 해제시키길 원할 경우, WKWebView의 등록을 반드시 해제해야 합니다.
위 전체 예시의 `deinit` / `dealloc`에서 `AdFit.unRegister(webView)`를 호출합니다.

---

## 코드 생성 규칙

사용자에게 코드를 생성할 때 아래 규칙을 따릅니다:

1. `{adUnitId}`는 사용자가 제공한 광고 단위 ID로 치환합니다.
2. `{webPageURL}`는 사용자가 제공한 웹 페이지 URL로 치환합니다. 제공하지 않은 경우 placeholder로 남겨둡니다.
3. UIKit의 경우 Auto Layout을 우선적으로 사용하여 레이아웃을 구성합니다. (frame 기반 레이아웃은 사용자가 명시적으로 요청한 경우에만 사용합니다.)
4. UIKit의 경우 반드시 `view.safeAreaLayoutGuide`를 기준으로 제약조건을 설정하여 Safe Area를 준수합니다.
5. WKWebView 생성 시 `WKWebViewConfiguration`의 동영상 자동 재생 설정을 반드시 포함합니다.
6. `AdFit.register(webView: webView)` 호출을 반드시 포함합니다.
7. `AdFit.unRegister(webView: webView)` 호출을 `deinit` 또는 `dealloc`에 반드시 포함합니다.
8. 사용자의 클래스명이나 뷰 이름이 주어진 경우 해당 이름을 사용합니다.
