# 배너 광고 연동 가이드

## 사전 조건
- iOS 15.0 이상
- AdFitSDK가 Swift Package Manager(SPM)를 통해 프로젝트에 설치되어 있어야 합니다.

---

## UIKit 연동

### 1. 광고 요청하기

#### Swift
```swift
import UIKit
import AdFitSDK

class MyViewController: UIViewController {

    override func viewDidLoad() {
        super.viewDidLoad()
        let bannerAdView = AdFitBannerAdView(clientId: "{clientId}", adUnitSize: "{adUnitSize}")
        bannerAdView.translatesAutoresizingMaskIntoConstraints = false
        bannerAdView.rootViewController = self
        view.addSubview(bannerAdView)

        NSLayoutConstraint.activate([
            bannerAdView.leadingAnchor.constraint(equalTo: view.safeAreaLayoutGuide.leadingAnchor),
            bannerAdView.trailingAnchor.constraint(equalTo: view.safeAreaLayoutGuide.trailingAnchor),
            bannerAdView.bottomAnchor.constraint(equalTo: view.safeAreaLayoutGuide.bottomAnchor),
            bannerAdView.heightAnchor.constraint(equalToConstant: {adHeight})
        ])

        bannerAdView.loadAd()
    }

}
```

#### Objective-C
```objc
#import "MyViewController.h"
#import <AdFit/AdFit-Swift.h>

@implementation MyViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    AdFitBannerAdView *bannerAdView = [[AdFitBannerAdView alloc] initWithClientId:@"{clientId}"
                                                                       adUnitSize:@"{adUnitSize}"];
    bannerAdView.translatesAutoresizingMaskIntoConstraints = NO;
    bannerAdView.rootViewController = self;
    [self.view addSubview:bannerAdView];

    [NSLayoutConstraint activateConstraints:@[
        [bannerAdView.leadingAnchor constraintEqualToAnchor:self.view.safeAreaLayoutGuide.leadingAnchor],
        [bannerAdView.trailingAnchor constraintEqualToAnchor:self.view.safeAreaLayoutGuide.trailingAnchor],
        [bannerAdView.bottomAnchor constraintEqualToAnchor:self.view.safeAreaLayoutGuide.bottomAnchor],
        [bannerAdView.heightAnchor constraintEqualToConstant:{adHeight}]
    ]];

    [bannerAdView loadAd];
}

@end
```

### 2. 배너 속성 설정

#### rootViewController
- 배너 광고 클릭 시, SDK 내부 웹뷰 컨트롤러가 modal 방식으로 나타나며 광고주 페이지가 노출됩니다.
- 광고주 페이지를 노출시킬 뷰 컨트롤러를 직접 지정하려면 `rootViewController` 프로퍼티에 할당하세요.

#### delegate
- `AdFitBannerAdViewDelegate` 프로토콜을 구현한 객체를 `delegate` 프로퍼티에 할당하면 광고 이벤트를 감지할 수 있습니다.

### 3. delegate 메서드 구현

#### Swift
```swift
// 광고를 정상적으로 받은 경우
func adViewDidReceiveAd(_ bannerAdView: AdFitBannerAdView) {
    print("didReceiveAd")
}

// 광고를 받지 못한 경우
func adViewDidFailToReceiveAd(_ bannerAdView: AdFitBannerAdView, error: Error) {
    print("didFailToReceiveAd - error: \(error.localizedDescription)")
}

// 광고가 클릭된 경우
func adViewDidClickAd(_ bannerAdView: AdFitBannerAdView) {
    print("didClickAd")
}
```

#### Objective-C
```objc
- (void)adViewDidReceiveAd:(AdFitBannerAdView *)bannerAdView {
    NSLog(@"didReceiveAd");
}

- (void)adViewDidFailToReceiveAd:(AdFitBannerAdView *)bannerAdView error:(NSError *)error {
    NSLog(@"didFailToReceiveAd - error = %@", [error localizedDescription]);
}

- (void)adViewDidClickAd:(AdFitBannerAdView *)bannerAdView {
    NSLog(@"didClickAd");
}
```

---

## SwiftUI 연동

### 1. 광고 생성하기

```swift
import SwiftUI
import AdFitSDK

struct ContentView: View {
    var body: some View {
        AdFitBannerPresentableView(
            clientId: "{clientId}",
            adUnitSize: "{adUnitSize}"
        )
    }
}
```

### 2. operator 설정하기 (선택사항)

광고 이벤트를 감지하기 위한 operator를 제공합니다. 구현은 필수가 아닙니다.

#### onDidReceiveAd
```swift
.onDidReceiveAd { bannerAdView, error in
    // 배너 광고 응답이 왔을 때 호출
    // 정상 응답 시, error는 nil
}
```

#### onDidClickAd
```swift
.onDidClickAd { bannerAdView in
    // 광고 클릭 시 호출
}
```

#### onSizeThatFits
```swift
.onSizeThatFits(orientation: $orientation, width: customWidth)
```
- 배너 광고의 unitSize에 맞춰서 사이즈를 자동 설정합니다.
- `orientation`: 가로/세로 모드 전환 감지를 위한 `Binding<UIDeviceOrientation>`
- `width`: AdFitBannerPresentableView의 width (미입력 시 디바이스 width로 설정)

### 3. 전체 사용 예시

```swift
import SwiftUI
import AdFitSDK

struct ContentView: View {
    @State private var orientation: UIDeviceOrientation = .unknown

    var body: some View {
        AdFitBannerPresentableView(
            clientId: "{clientId}",
            adUnitSize: "{adUnitSize}"
        )
        .onDidClickAd { _ in
            print("광고를 클릭했습니다.")
        }
        .onDidReceiveAd { _, error in
            print("광고 응답을 받았습니다.")
            if let error = error {
                print("광고 응답 에러: \(error)")
            }
        }
        .onSizeThatFits(orientation: $orientation)
    }
}
```

---

## 에러 코드

광고 수신 실패 시 전달되는 에러 코드입니다.

| 코드 | 메시지 | 설명 |
|:----:|--------|------|
| 1 | clientId property is nil | clientId가 세팅되지 않은 경우 |
| 2 | no ad to show | 노출 가능한 광고가 없는 경우. 잠시 후 재요청하세요. |
| 3 | invalid ad received | 유효하지 않은 광고를 받은 경우. AdFit에 문의하세요. |
| 4 | failed to render ad | AdFitBannerAdView 사이즈가 기준보다 작은 경우 |
| 5 | attempted to load ad too frequently | 30초 이내 과도하게 재요청한 경우 |
| 6 | HTTP failed | HTTP 에러. 반복 시 AdFit에 문의하세요. |
| 14 | failed to load webview | 인앱 브라우저 내 광고 로딩 실패 (UIKit만 해당) |

---

## 코드 생성 규칙

사용자에게 코드를 생성할 때 아래 규칙을 따릅니다:

1. `{clientId}`는 사용자가 제공한 clientId로 치환합니다.
2. `{adUnitSize}`는 사용자가 제공한 adUnitSize로 치환합니다. (기본값: "320x50")
3. `{adHeight}`는 adUnitSize에서 높이를 추출하여 CGFloat 값으로 치환합니다. (예: "320x50" → 50)
4. UIKit의 경우 Auto Layout을 우선적으로 사용하여 레이아웃을 구성합니다. (frame 기반 레이아웃은 사용자가 명시적으로 요청한 경우에만 사용합니다.)
5. UIKit의 경우 반드시 `view.safeAreaLayoutGuide`를 기준으로 제약조건을 설정하여 Safe Area를 준수합니다.
6. UIKit의 경우 delegate 메서드 구현을 포함합니다.
7. SwiftUI의 경우 operator 설정 예시를 포함합니다.
8. 사용자의 클래스명이나 뷰 이름이 주어진 경우 해당 이름을 사용합니다.
