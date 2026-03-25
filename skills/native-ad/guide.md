# 네이티브 광고 연동 가이드

## 사전 조건
- iOS 15.0 이상
- AdFitSDK가 Swift Package Manager(SPM)를 통해 프로젝트에 설치되어 있어야 합니다.

---

## 1. 광고 요청하기

광고 단위 ID로 loader를 생성하고, `loadAd`를 호출합니다. `AdFitNativeAdLoaderDelegate`를 구현하여 응답을 받습니다.

### Swift
```swift
import UIKit
import AdFitSDK

final class MyViewController: UIViewController, AdFitNativeAdLoaderDelegate {
    private var nativeAdLoader: AdFitNativeAdLoader?

    override func viewDidLoad() {
        super.viewDidLoad()

        nativeAdLoader = AdFitNativeAdLoader(clientId: "{clientId}")
        nativeAdLoader?.delegate = self
        nativeAdLoader?.loadAd()
    }

    // MARK: AdFitNativeAdLoaderDelegate
    func nativeAdLoaderDidReceiveAd(_ nativeAd: AdFitNativeAd) {
        // 광고응답 -> nativeAd: AdFitNativeAd
    }

    func nativeAdLoaderDidFailToReceiveAd(_ nativeAdLoader: AdFitNativeAdLoader, error: Error) {
        // 광고응답 실패 -> error 확인하기
    }
}
```

### Objective-C
```objc
#import "MyViewController.h"
#import <AdFitSDK/AdFitSDK-Swift.h>

@implementation MyViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    self.nativeAdLoader = [[AdFitNativeAdLoader alloc] initWithClientId:@"{clientId}" count:1 userObject:nil contentObject:nil];
    self.nativeAdLoader.delegate = self;
    [self.nativeAdLoader loadAdWithKeyword:nil];
}

#pragma mark - AdFitNativeAdLoaderDelegate
- (void)nativeAdLoaderDidReceiveAd:(AdFitNativeAd *)nativeAd {
    // 광고응답 -> nativeAd: AdFitNativeAd
}

- (void)nativeAdLoaderDidFailToReceiveAd:(AdFitNativeAdLoader *)nativeAdLoader error:(NSError *)error {
    // 광고응답 실패 -> error 확인하기
}
@end
```

---

## 2. 광고 뷰 구현하기

- 네이티브 광고의 UI는 서비스의 콘텐츠와 잘 어울리도록 구성되어야 하므로, 뷰 클래스를 개별 서비스에서 직접 구현해야 합니다.
- 뷰 클래스는 `UIView`를 상속받고, `AdFitNativeAdRenderable` 프로토콜을 추가로 구현합니다.

### 2.1 네이티브 광고 뷰 구성

AdFit의 네이티브 광고는 아래 7가지 요소로 구성됩니다.

| 번호 | 설명 | UI 클래스 | AdFitNativeAdRenderable |
|-----|------|-----------|-------------------------|
| 1 | 제목 | UILabel | adTitleLabel() |
| 2 | 행동유도버튼 | UIButton | adCallToActionButton() |
| 3 | 프로필명 | UILabel | adProfileNameLabel() |
| 4 | 프로필이미지 | UIImageView | adProfileIconView() |
| 5 | 미디어 (이미지, 동영상 등) | AdFitMediaView | adMediaView() |
| 6 | 광고 아이콘 | | |
| 7 | 홍보문구 | UILabel | adBodyLabel() |

- `5. 미디어` 요소는 SDK에 포함된 `AdFitMediaView` 클래스를 사용하여 표시합니다.

### 2.2 AdFitNativeAdRenderable protocol

```swift
@objc public protocol AdFitNativeAdRenderable {
    func adTitleLabel() -> UILabel?
    @objc optional func adBodyLabel() -> UILabel?
    func adCallToActionButton() -> UIButton?
    func adProfileNameLabel() -> UILabel?
    func adProfileIconView() -> UIImageView?
    func adMediaView() -> AdFitMediaView?
}
```

### Swift 구현 예시
```swift
import UIKit
import AdFitSDK

class MyNativeAdView: UIView, AdFitNativeAdRenderable {

    @IBOutlet weak var titleLabel: UILabel?
    @IBOutlet weak var bodyLabel: UILabel?
    @IBOutlet weak var callToActionButton: UIButton?
    @IBOutlet weak var profileImageView: UIImageView?
    @IBOutlet weak var profileNameLabel: UILabel?
    @IBOutlet weak var mediaView: AdFitMediaView?

    // MARK: - AdFitNativeAdRenderable
    func adTitleLabel() -> UILabel? {
        return titleLabel
    }

    func adBodyLabel() -> UILabel? {
        return bodyLabel
    }

    func adCallToActionButton() -> UIButton? {
        return callToActionButton
    }

    func adProfileNameLabel() -> UILabel? {
        return profileNameLabel
    }

    func adProfileIconView() -> UIImageView? {
        return profileImageView
    }

    func adMediaView() -> AdFitMediaView? {
        return mediaView
    }
}
```

### Objective-C 구현 예시
```objc
// MyNativeAdView.h
#import <UIKit/UIKit.h>
#import <AdFitSDK/AdFitSDK-Swift.h>

@interface MyNativeAdView : UIView <AdFitNativeAdRenderable>
@property (nonatomic, weak) IBOutlet UILabel *titleLabel;
@property (nonatomic, weak) IBOutlet UILabel *bodyLabel;
@property (nonatomic, weak) IBOutlet UIButton *callToActionButton;
@property (nonatomic, weak) IBOutlet UILabel *profileNameLabel;
@property (nonatomic, weak) IBOutlet UIImageView *profileIconView;
@property (nonatomic, weak) IBOutlet AdFitMediaView *mediaView;
@end

// MyNativeAdView.m
#import "MyNativeAdView.h"

@implementation MyNativeAdView

#pragma mark - AdFitNativeAdRenderable
- (UILabel *)adTitleLabel { return self.titleLabel; }
- (UILabel *)adBodyLabel { return self.bodyLabel; }
- (UIButton *)adCallToActionButton { return self.callToActionButton; }
- (UILabel *)adProfileNameLabel { return self.profileNameLabel; }
- (UIImageView *)adProfileIconView { return self.profileIconView; }
- (AdFitMediaView *)adMediaView { return self.mediaView; }

@end
```

---

## 3. 네이티브 광고 뷰의 높이 최적화

네이티브 광고의 미디어 요소는 각기 다른 비율의 이미지 또는 비디오가 수신될 수 있습니다.
- 이미지: 2:1 or 1:1
- 비디오: 16:9

`mediaAspectRatio` 속성을 사용하여 광고 뷰의 사이즈를 최적화할 수 있습니다.
비율에 따라 사이즈를 조정하지 않으면, 미디어 요소는 `scaleAspectFit` 로직에 따라 그려집니다.

### Swift
```swift
func nativeAdLoaderDidReceiveAd(_ nativeAd: AdFitNativeAd) {
    let mediaAspectRatio = nativeAd.mediaAspectRatio
    if mediaAspectRatio > 0 {
        let mediaWidth = nativeAdView.frame.width
        let mediaHeight = mediaWidth / mediaAspectRatio
        // AdFitMediaView의 height를 조정할 수 있는 코드 구현
    }
    nativeAd.bind(nativeAdView)
}
```

### Objective-C
```objc
- (void)nativeAdLoaderDidReceiveAd:(AdFitNativeAd *)nativeAd {
    CGFloat mediaAspectRatio = nativeAd.mediaAspectRatio;
    if (mediaAspectRatio > 0) {
        CGFloat mediaWidth = nativeAdView.frame.size.width;
        CGFloat mediaHeight = mediaWidth / mediaAspectRatio;
        // AdFitMediaView의 height를 조정할 수 있는 코드 구현
    }
    [nativeAd bind:nativeAdView];
}
```

---

## 4. NativeAd 추가 속성

### rootViewController
광고 클릭 시, SDK 내부 웹뷰 컨트롤러가 modal 방식으로 나타나며 광고주 페이지가 노출됩니다.
광고주 페이지를 노출시킬 뷰 컨트롤러를 직접 지정하려면 `rootViewController` 프로퍼티에 할당합니다.

```swift
func nativeAdLoaderDidReceiveAd(_ nativeAd: AdFitNativeAd) {
    nativeAd.rootViewController = landingViewController
}
```

### mediaAspectRatio
미디어 요소의 가로/세로 비율입니다. 로드 전에는 0, 로드 후 실제 비율로 변경됩니다.

---

## 5. 광고 클릭 이벤트 감지

`AdFitNativeAdDelegate` 구현을 통해 광고 클릭 이벤트를 감지할 수 있습니다.

### Swift
```swift
import UIKit
import AdFitSDK

final class MyViewController: UIViewController, AdFitNativeAdLoaderDelegate, AdFitNativeAdDelegate {

    // ... 구현 생략 ...

    // MARK: AdFitNativeAdLoaderDelegate
    func nativeAdLoaderDidReceiveAd(_ nativeAd: AdFitNativeAd) {
        nativeAd.delegate = self
    }

    // MARK: - AdFitNativeAdDelegate
    func nativeAdDidClickAd(_ nativeAd: AdFitNativeAd) {
        // 광고 클릭 시, 호출됨.
    }
}
```

### Objective-C
```objc
#import "MyViewController.h"
#import <AdFitSDK/AdFitSDK-Swift.h>

@implementation MyViewController

// ... 구현 생략 ...

#pragma mark - AdFitNativeAdLoaderDelegate
- (void)nativeAdLoaderDidReceiveAd:(AdFitNativeAd *)nativeAd {
    nativeAd.delegate = self;
}

#pragma mark - AdFitNativeAdDelegate
- (void)nativeAdDidClickAd:(AdFitNativeAd *)nativeAd {
    // 광고 클릭 시, 호출됨.
}

@end
```

---

## 6. 에러 코드

광고 수신에 실패한 경우, `nativeAdLoaderDidFailToReceiveAd` delegate를 통해 에러 객체를 전달받습니다.

| 코드 | 메시지 | 설명 |
|:----:|--------|------|
| 1 | clientId property is nil | clientId가 세팅되지 않은 경우 |
| 2 | no ad to show | 노출 가능한 광고가 없는 경우. 잠시 후 재요청하세요. |
| 3 | invalid ad received | 유효하지 않은 광고를 받은 경우. AdFit에 문의하세요. |
| 5 | attempted to load ad too frequently | 과도하게 짧은 시간 간격으로 재요청한 경우 |
| 6 | HTTP failed | HTTP 에러. 반복 시 AdFit에 문의하세요. |

---

## 코드 생성 규칙

사용자에게 코드를 생성할 때 아래 규칙을 따릅니다:

1. `{clientId}`는 사용자가 제공한 clientId로 치환합니다.
2. UIKit의 경우 Auto Layout을 우선적으로 사용하여 레이아웃을 구성합니다.
3. UIKit의 경우 반드시 `view.safeAreaLayoutGuide`를 기준으로 제약조건을 설정하여 Safe Area를 준수합니다.
4. `AdFitNativeAdLoaderDelegate`와 `AdFitNativeAdDelegate` 구현을 포함합니다.
5. `mediaAspectRatio`를 활용한 미디어 높이 최적화 코드를 포함합니다.
6. 사용자의 클래스명이나 뷰 이름이 주어진 경우 해당 이름을 사용합니다.
7. 네이티브 광고 뷰는 모든 `AdFitNativeAdRenderable` 프로토콜 메서드를 구현합니다.
